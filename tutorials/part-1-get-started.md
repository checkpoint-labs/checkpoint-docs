# Part 1 - Get started

In this tutorial, we'll be exploring how to index and query data on Starknet using Checkpoint. We'll cover the basics of setting up a Checkpoint project to index data on Starknet, including defining a Checkpoint configuration, a GraphQL entity schema, and data writers. We'll also cover how to start the Checkpoint indexer and query the indexed data using the generated GraphQL API.

By the end of this tutorial, you'll have a good understanding of how to use Checkpoint to index and query data on Starknet\
\
[💡](https://emojipedia.org/fr/ampoule/) W_e’ll be using_ [_https://github.com/snapshot-labs/checkpoint-template_](https://github.com/snapshot-labs/checkpoint-template) _as an exemple. Feel free to follow along with the repository._\
\
**Step 1: Installing Checkpoint**&#x20;

To get started with Checkpoint, you'll need to install the module using either npm or yarn. Open your terminal and navigate to your project directory, then run the following command:

```bash
npm install @snapshot-labs/checkpoint
```

Or, if you prefer using yarn

```bash
yarn add @snapshot-labs/checkpoint
```

**Step 2: Creating the Project Structure**

Next, you'll need to create a project structure for your Checkpoint application. In this tutorial, we'll be creating the following structure:

```jsx
project/
├── src/
│   ├── config.json
│   ├── index.ts
│   ├── schema.gql
│   └── writers.ts
└── package.json
```

The **`src`** directory will contain all the source files for your application, while **`package.json`** will be used to manage your application's dependencies.

**Step 3: Creating the Checkpoint Configuration**

Checkpoint uses a simple process to index data. It traces the blockchain block by block and at each of these blocks, it checks if the smart contract we want to track has emitted events and if so, do these events correspond to those we want?

To do this, we need to create a configuration file for Checkpoint. In the **`src`** directory, create a file named **`config.json`** and define the following configuration:

```json
{
  "network_node_url": "https://starknet-goerli.infura.io/v3/46a5dd9727bf48d4a132672d3f376146",
  "sources": [
    {
      "contract": "0x04d10712e72b971262f5df09506bbdbdd7f729724030fa909e8c8e7ac2fd0012",
      "start": 185778,
      "deploy_fn": "handleDeploy",
      "events": [
        {
          "name": "new_post",
          "fn": "handleNewPost"
        }
      ]
    }
  ]
}
```

The **`network_node_url`** property specifies the URL of the Starknet node we want to connect to. The **`sources`** property is an array of objects that define the smart contract addresses and their respective events we want to index. In this example, we're tracking a list of posts and authors, and listening to the **`new_post`** event emitted by the smart contract deployed at **`0x04d10712e72b971262f5df09506bbdbdd7f729724030fa909e8c8e7ac2fd0012`**. The **`start`** property specifies the block number from which Checkpoint starts scanning. The **`deploy_fn`** and **`fn`** properties are the names of the data writer functions to be invoked when the contract deployment and **`new_post`** events are encountered respectively.

**Step 4: Defining GraphQL Entity Schemas**

Checkpoint requires a set of defined GraphQL Schema Objects. These schema objects will be used to create the database tables for indexing records and also generate GraphQL queries for accessing the indexed data.

In the **`src`** directory, create a file named **`schema.gql`** and define the schema for the Post entity we'll be tracking:

```graphql
type Post {
  id: String!
  author: String!
  created_at_block: Int!
}
```

Checkpoint will use the above entity (Post) to generate a MySQL database table named **`posts`** with columns matching the defined fields. It will also generate a list of GraphQL queries to enable querying indexed data.

**Step 5: Creating Data Writers**

Data writers are typescript functions that get invoked by Checkpoint when it discovers a block containing a relevant event. A data writer is responsible for writing records to the database. These records will eventually be exposed via Checkpoint's GraphQL endpoint.

In the **`src`** directory, create a file named **`writers.ts`** and define the data writer functions for the **`handleDeploy`** and **`handleNewPost`** events:

```typescript
import { hexStrArrToStr, toAddress } from './utils';
import type { starknet } from '@snapshot-labs/checkpoint';
import { Post } from '../.checkpoint/models';

export async function handleDeploy() {
  // Run logic as at the time Contract was deployed.
}

// This decodes the new_post events data and stores successfully
// decoded information in the `posts` table.
//
// See here for the original logic used to create post transactions:
// https://gist.github.com/perfectmak/417a4dab69243c517654195edf100ef9#file-index-ts
export async function handleNewPost({ block, tx, event }: Parameters<starknet.Writer>[0]) {
  if (!event) return;

  const author = toAddress(event.data[0]);
  let content = '';
  let tag = '';
  const contentLength = BigInt(event.data[1]);
  const tagLength = BigInt(event.data[2 + Number(contentLength)]);
  const timestamp = block!.timestamp;
  const blockNumber = block!.block_number;

  // parse content bytes
  try {
    content = hexStrArrToStr(event.data, 2, contentLength);
  } catch (e) {
    console.error(`failed to decode content on block [${blockNumber}]: ${e}`);
    return;
  }

  // parse tag bytes
  try {
    tag = hexStrArrToStr(event.data, 3 + Number(contentLength), tagLength);
  } catch (e) {
    console.error(`failed to decode tag on block [${blockNumber}]: ${e}`);
    return;
  }

  // Create new Post from generated models
  const post = new Post(`${author}/${tx.transaction_hash}`);
  post.author = author;
  post.content = content;
  post.tag = tag;
  post.tx_hash = tx.transaction_hash!,
  post.created_at = timestamp;
  post.created_at_block = blockNumber;

  // Save Posts into your db
  await post.save();
}
```

**Step 7: Mounting the GraphQL Endpoint**

Checkpoint exposes a GraphQL endpoint to enable querying the indexed data. In the **`index.ts`** file, add the following code to mount the GraphQL endpoint on a port:

```tsx
import 'dotenv/config';
import express from 'express';
import cors from 'cors';
import path from 'path';
import fs from 'fs';
import Checkpoint, { starknet, LogLevel } from '@snapshot-labs/checkpoint';
import config from './config.json';
import * as writers from './writers';
import checkpointBlocks from './checkpoints.json';

const dir = __dirname.endsWith('dist/src') ? '../' : '';
const schemaFile = path.join(__dirname, `${dir}../src/schema.gql`);
const schema = fs.readFileSync(schemaFile, 'utf8');

const checkpointOptions = {
  logLevel: LogLevel.Info
  // prettifyLogs: true, // uncomment in local dev
};

// Initialize checkpoint
const indexer = new starknet.StarknetIndexer(writers);
const checkpoint = new Checkpoint(config, indexer, schema, checkpointOptions);

// resets the entities already created in the database
// ensures data is always fresh on each re-run
checkpoint
  .reset()
  .then(() => checkpoint.seedCheckpoints(checkpointBlocks))
  .then(() => {
    // start the indexer
    checkpoint.start();
  });

const app = express();
app.use(express.json({ limit: '4mb' }));
app.use(express.urlencoded({ limit: '4mb', extended: false }));
app.use(cors({ maxAge: 86400 }));

// mount Checkpoint's GraphQL API on path /
app.use('/', checkpoint.graphql);

const PORT = process.env.PORT || 3000;
app.listen(PORT, () => console.log(`Listening at <http://localhost>:${PORT}`));
```

We're creating a new Express application and mounting the Checkpoint GraphQL endpoint on the **`/graphql`** path. We're then starting Checkpoint with the required parameters and the server on the specified port.

**Step 8: Testing the Checkpoint Application**

Now that we've set up our Checkpoint application, we can test it by running the following command in the terminal:

`yarn dev`

This will start the Checkpoint indexer and mount the GraphQL endpoint. You can now query the indexed data using the generated GraphQL API.\
Checkpoint exposes two types of queries:

1. **`entities`**: This query enables you to fetch information about the entities being indexed.
2. **`records`**: This query enables you to fetch the indexed data.

For example, to fetch all the posts in the database, you can run the following query on[http://localhost:3000](http://localhost:3000):

```graphql
query {
  posts {
    id
    author
    content
    tag
    created_at_block
    created_at
    tx_hash
  }
}
```

**Step 9: Customizing Checkpoint**

Checkpoint provides several options for customizing its behavior. In the **`config.json`** file, you can set the following options:

* **`network_node_url`**: The URL of the StarkNet node to connect to.
* **`sources`**: An array of objects representing the contracts and their events to track.
* **`start_block`**: The block number to start indexing from..
* In the **`writers.ts`** file, you can customize the data writer functions to suit your indexing needs.
* In the **`schema.gql`** file, you can define additional entities to track and expose via the GraphQL API.

**Conclusion**

That’s it! You should have Checkpoint running, indexing your contracts data and serving this indexed data via graphql.
