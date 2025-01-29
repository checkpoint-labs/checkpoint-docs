# Step-by-step setup

Checkpoint is a Node.js library and running the following NPM commands will install it:

```
npm install @snapshot-labs/checkpoint@beta

# or if using yarn
yarn add @snapshot-labs/checkpoint@beta
```

Two arguments can be used to initialize a Checkpoint instance. These are:

* GraphQL schema (required)
* &#x20;[`CheckpointOptions`](advanced-options.md) (optional)

### Creating a Checkpoint Configuration

A Checkpoint configuration is an object that defines the smart contract addresses and their respective events. For this guide, we will be indexing the list of posts and authors for this [Poster Contract](https://github.com/snapshot-labs/starknet-poster/blob/master/contracts/Poster.cairo). A copy of this contract is deployed on following Starknet networks:

* Mainnet: 0x0654e9232d5f402829755029901f69c32b423ded0f8c081e416e3b24f5a7a46e
* Sepolia: 0x03aa7630a4f9c5108bf3cd1910c7d45404cba865fc0fc0756bf9eedc073a98a9

With new versions of Checkpoint contracts on different networks can be indexed and queried from single Checkpoint instance.

To successfully track the addresses of authors, you'll need to listen to [new\_post](https://github.com/snapshot-labs/starknet-poster/blob/master/contracts/Poster.cairo#L7) events from the contract. Therefore, a valid checkpoint configuration for the above requirements will be:

```typescript
import { CheckpointConfig } from "@snapshot-labs/checkpoint";

export const mainnetConfig: CheckpointConfig = {
  network_node_url: "https://starknet-mainnet.infura.io/v3/46a5dd9727bf48d4a132672d3f376146",
  sources: [
    {
      contract: "0x0654e9232d5f402829755029901f69c32b423ded0f8c081e416e3b24f5a7a46e",
      start: 639485,
      events: [
        {
          name: "new_post",
          fn: "handleNewPost",
        }
      ],
    }
  ]
};

export const sepoliaConfig: CheckpointConfig = {
  network_node_url: "https://starknet-sepolia.infura.io/v3/46a5dd9727bf48d4a132672d3f376146",
  sources: [
    {
      contract: "0x03aa7630a4f9c5108bf3cd1910c7d45404cba865fc0fc0756bf9eedc073a98a9",
      start: 65137,
      events: [
        {
          name: "new_post",
          fn: "handleNewPost",
        }
      ],
    }
  ]
};
```

In this case we want to index Poster contract on both Starknet mainnet and Starknet sepolia so we define two configurations.

The `start` block number is set to `639485` because our mainnet contract was deployed at that block. This will mean Checkpoint starts scanning from that block as opposed to starting at block 0.

`fn` value is the name of the data writer function to be invoked when `new_post` events are encountered. Read more about Checkpoint configuration [here](../core-concepts/checkpoint-configuration.md).

### Defining GraphQL entity schemas

Checkpoint requires a set of defined GraphQL Schema Objects. These schema objects will be used to create the database tables for indexing records and also generate graphql queries for accessing the indexed data.

For this guide, we will want to track a `Post` entity and have it exposed via the graphql API. This entity can be defined as the following schema file:

<pre class="language-graphql" data-title="src/schema.gql"><code class="lang-graphql"><strong>""" Entity named Post """
</strong>type Post {
  id: String!
  author: String!
  created_at_block: Int!
}
</code></pre>

Checkpoint will use the above entity (`Post)` to generate a PostgreSQL database table named `posts` with columns matching the defined fields. It will also generate a list of GraphQL queries to enable querying indexed data. Read more about how queries are generated [here](../core-concepts/entity-schema.md#query-generation).

When updating your schema you should run following script to generate ORM models:

```sh
yarn checkpoint generate
```

### Creating Data Writers

Data writers are JavaScript functions that get invoked by Checkpoint when it discovers a block containing this event. A Data writer is responsible for writing records to the database. These records will eventually be exposed via Checkpoint's GraphQL endpoint.

We have defined data writer function in our Checkpoint configuration for `new_post` event called `handleNewPost`.

Let's create these data writer functions:

{% code title="src/writers.ts" %}
```typescript
import { starknet } from "@snapshot-labs/checkpoint";
import { getAddress } from '@ethersproject/address';
import { Post } from '../.checkpoint/models';

// We create createWriters function that accepts indexerName.
// This means we can reuse those for different networks (for example
// "mainnet" and "sepolia".
export const createWriters(indexerName: string) {
    // handleNewPost will get invoked when a `new_post` event
    // is found at a block
    const handleNewPost: starknet.Writer = async ({ event, block }) => {
        if (!event) return;
    
        // extract posters address from events data
        const author = getAddress(BigNumber.from(event.data[0]).toHexString());
        
        // store Post in database (note we pass indexerName here to be able
        // to tell apart Posts on each network)
        const post = new Post(`${author}/${tx.transaction_hash}`, indexerName);
        post.author = author;
        post.created_at_block = block.blockNumber;
        await post.save();
    };
    
    return { handleNewPost };
}
```
{% endcode %}

With the above code snippet, we have a data writer that writes new posts to the PostgreSQL database.

{% hint style="info" %}
You can view a more comprehensive data writer example in our checkpoint-template codebase [here](https://github.com/snapshot-labs/checkpoint-template/blob/master/src/writers.ts).
{% endhint %}

### Starting Checkpoint

Finally, we can initialize a checkpoint instance with our arguments like these:

{% code title="src/index.ts" %}
```typescript
import fs from 'fs/promises';
import Checkpoint from "@snapshot-labs/checkpoint";
import { createWriters } from './writers.ts';
import { mainnetConfig, sepoliaConfig } from './config.ts';

...

const schemaFile = path.join(__dirname, `${dir}../src/schema.gql`);
const schema = fs.readFileSync(schemaFile, 'utf8');

...

const checkpoint = new Checkpoint(schema);

const mainnetIndexer = new starknet.StarknetIndexer(createWriters('mainnet'));
checkpoint.addIndexer('mainnet', mainnetConfig, mainnetIndexer);

const sepoliaIndexer = new starknet.StarknetIndexer(createWriters('sepolia'));
checkpoint.addIndexer('sepolia', sepoliaConfig, sepoliaIndexer);
```
{% endcode %}

Next, we start up checkpoint's indexer like this:

```typescript
checkpoint.start();
```

The above code will start the checkpoint indexer, and it will begin processing each Starknet block. When a relevant contract event is found, it gets passed to the data writer for that event.

### Querying data

Once Checkpoint has run for a while and has written some data to its database, you can query this data using the generated GraphQL API.

You can mount the query endpoint on any port you like using the `graphql` handler exported by Checkpoint's object. Like this:

```typescript
import express from 'express';

const checkpoint = new Checkpoint(...);

const app = express();
app.use('/graphql', checkpoint.graphql);

const PORT = 3000;
app.listen(PORT, () => console.log(`Listening at http://localhost:${PORT}`));
```

This will mount a GraphQL endpoint that can be accessed at http://localhost:3000/graphql (and a GraphiQL interface at http://localhost:3000/graphql when visited from the browser).&#x20;

Checkpoint exposes two types of queries:&#x20;

1. [Entity Queries](../core-concepts/entity-schema.md#query-generation)
2. [Internal data Queries](../core-concepts/internal-data-query.md)

These enables users to fetch information about indexer.

For starters, you can visit http://localhost:3000/graphql in your browser and try running the sample query generated in the graphiql UI.

### Conclusion

At this juncture, you should have Checkpoint running, indexing your contracts data (on multiple networks) and serving this indexed data via graphql.&#x20;

Next up, you can explore our template repository [here](https://github.com/snapshot-labs/checkpoint-template) to get up and running quickly.

You can also explore some Checkpoint [core concepts](broken-reference) or move on to the next guide.

