# Step-by-step setup

Checkpoint is a Node.js library and running the following NPM commands will install it:

```
npm install @snapshot-labs/checkpoint@beta

# or if using yarn
yarn add @snapshot-labs/checkpoint@beta
```

Three arguments are required to initialize a Checkpoint instance. These are:

* Checkpoint Configuration,
* GraphQL entity Schemas,
* Data Writers.

{% hint style="info" %}
There is a fourth optional argument for configuring options like log levels and database connection. Read more about this parameter [here](advanced-options.md).
{% endhint %}

### Creating a Checkpoint Configuration

A checkpoint configuration is an object that defines the smart contract addresses and their respective events. For this guide, we will be indexing the list of posts and authors for this [Poster Contract](https://github.com/snapshot-labs/starknet-poster/blob/master/contracts/Poster.cairo). A copy of this contract is deployed to the `goerli-alpha` network at the address: `0x04d10712e72b971262f5df09506bbdbdd7f729724030fa909e8c8e7ac2fd0012`.

To successfully track the addresses of authors, you'll need to listen to [new\_post](https://github.com/snapshot-labs/starknet-poster/blob/master/contracts/Poster.cairo#L7) events from the contract. Therefore, a valid checkpoint configuration for the above requirements will be:

```typescript
import { CheckpointConfig } from "@snapshot-labs/checkpoint";

const config: CheckpointConfig = {
  network_node_url: "https://starknet-goerli.infura.io/v3/46a5dd9727bf48d4a132672d3f376146",
  sources: [
    {
      contract: "0x04d10712e72b971262f5df09506bbdbdd7f729724030fa909e8c8e7ac2fd0012",
      start: 185778,
      deploy_fn: "handleDeploy",
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

The `start` block number is set to `185778` because the contract was deployed at that block. This will mean Checkpoint starts scanning from that block as opposed to starting at block 0.

The `deploy_fn` and `fn` values are the names of the data writer functions to be invoked when the contract deployment and new\_post events are encountered respectively. Read more about Checkpoint configuration [here](../core-concepts/checkpoint-configuration.md).

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

Checkpoint will use the above entity (`Post)` to generate a MySQL database table named `posts` with columns matching the defined fields. It will also generate a list of graphql queries to enable querying indexed data. Read more about how queries are generated [here](../core-concepts/entity-schema.md#query-generation).

When updating your schema you should run following script to generate ORM models:

```sh
yarn checkpoint generate
```

### Creating Data Writers

Data writers are Javascript functions that get invoked by Checkpoint when it discovers a block containing this event. A Data writer is responsible for writing records to the database. These records will eventually be exposed via Checkpoint's GraphQL endpoint.

We have defined two data writer functions in our Checkpoint configuration above. These are:

* handleDeploy
* handleNewPost

Let's create these data writer functions:

{% code title="src/writers.ts" %}
```typescript
import { starknet } from "@snapshot-labs/checkpoint";
import { getAddress } from '@ethersproject/address';
import { Post } from '../.checkpoint/models';

// handleDeploy will get invoked when a contract deployment
// is found at a block
export const handleDeploy: starknet.Writer = async (args) => {
    // we won't do anything at this time.
};

// handleNewPost will get invoked when a `new_post` event
// is found at a block
export const handleNewPost: starknet.Writer = async ({ event, block }) => {
    if (!event) return;

    // extract posters address from events data
    const author = getAddress(BigNumber.from(event.data[0]).toHexString());
    
    // store Post in database
    const post = new Post(`${author}/${tx.transaction_hash}`);
    post.author = author;
    post.created_at_block = block.blockNumber;
    await post.save();
};
```
{% endcode %}

With the above code snippet, we have a data writer that writes new posts to the MySQL database.

{% hint style="info" %}
You can view a more comprehensive data writer example in our checkpoint-template codebase [here](https://github.com/snapshot-labs/checkpoint-template/blob/master/src/writers.ts).
{% endhint %}

### Starting Checkpoint with Arguments

Finally, we can initialize a checkpoint instance with our arguments like these:

{% code title="src/index.ts" %}
```typescript
import fs from 'fs/promises';
import Checkpoint from "@snapshot-labs/checkpoint";
import * as writers from './writers.ts';

...

const schemaFile = path.join(__dirname, `${dir}../src/schema.gql`);
const schema = fs.readFileSync(schemaFile, 'utf8');

...

const indexer = new starknet.StarknetIndexer(writers);
const checkpoint = new Checkpoint(config, indexer, schema);
```
{% endcode %}

Next, we start up checkpoint's indexer like this:

```typescript
checkpoint.start();
```

The above code will start the checkpoint indexer, and it will begin processing each StarkNet block. When a relevant contract event is found, it gets passed to the data writer for that event.

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

This will mount a GraphQL endpoint that can be accessible at http://localhost:3000/graphql (and a GraphiQL interface at http://localhost:3000/graphql when visited from the browser).&#x20;

Checkpoint exposes two types of queries:&#x20;

1. [Entity Queries](../core-concepts/entity-schema.md#query-generation)
2. [Internal data Queries](../core-concepts/internal-data-query.md)

These enable public users to fetch information about being indexed.

For starters, you can visit http://localhost:3000/graphql in your browser and try running the sample query generated in the graphiql UI.

### Conclusion

At this juncture, you should have Checkpoint running, indexing your contracts data and serving this indexed data via graphql.&#x20;

Next up, you can explore our template repository [here](https://github.com/snapshot-labs/checkpoint-template) to get up and running quickly.

You can also explore some Checkpoint [core concepts](broken-reference) or move on to the next guide.

