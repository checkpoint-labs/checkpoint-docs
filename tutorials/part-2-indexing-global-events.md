---
description: >-
  In this tutorial, we'll be exploring how to index and query global events on
  Starknet using Checkpoint.
---

# Part 2 - Indexing global events

We'll start from scratch so if you read episode you may want to skip some parts. We’ll go over the basics of setting up a Checkpoint project, including defining a Checkpoint configuration, a GraphQL entity schema, and data writers. We'll also cover how to start the Checkpoint indexer and query the indexed data using the generated GraphQL API.

By the end of this tutorial, you'll have a good understanding of how to use Checkpoint to index global events and query data on Starknet.

We’ll be using [https://github.com/checkpoint-labs/token-api-checkpoint](https://github.com/checkpoint-labs/token-api-checkpoint) as an example. Feel free to follow along with the repository

**Step 1: Installing Checkpoint**

To get started with Checkpoint, you'll need to install the module using either npm or yarn. Open your terminal and navigate to your project directory, then run the following command:

`npm install @snapshot-labs/checkpoint`

Or, if you prefer using yarn: `yarn add @snapshot-labs/checkpoint`

**Step 2: Creating the Project Structure**

Next, you'll need to create a project structure for your Checkpoint application. In this tutorial, we'll be creating the following structure conatining the abi of the contract type we’d like to import, it will be needed later on to query the tokens metadatas:

```jsx
project/
├── src/
|   ├── abis/erc20.json
│   ├── config.json
│   ├── index.ts
│   ├── schema.gql
│   └── writers.ts
└── package.json
```

The **`src`** directory will contain all the source files for your application, while **`package.json`** will be used to manage your application's dependencies.

**Step 3: Checkpoint Configuration for global events** Checkpoint uses a simple process to index data. It traces the blockchain block by block and at each of these blocks, it checks if the smart contract we want to track has emitted events and if so, do these events correspond to those we want?

To do this, we need to create a configuration file for Checkpoint. In the **`src`** directory, create a file named **`config.json`** and define the following configuration:

```json
{
  "network_node_url": "https://starknet-goerli.infura.io/v3/46a5dd9727bf48d4a132672d3f376146",
  "start": 10000,
  "global_events": [
    {
      "name": "Transfer",
      "fn": "handleTransfer"
    }
  ]
}

```

The **`network_node_url`** property specifies the URL of the Starknet node we want to connect to. The `**global_events`\*\* property is an array of objects that define the events we want to index and it’s associated handle function. Here compared to the previous tutorial you will notice that we have not filled any contract but only an event. In this example, we're tracking a list of erc20 and holders, and listening to the `**transfer`\*\* event emitted by any smart-contract. The **`start`** property specifies the block number from which Checkpoint starts scanning.

**Step 4: Defining GraphQL Entity Schemas**

Checkpoint requires a set of defined GraphQL Schema Objects. These schema objects will be used to create the database tables for indexing records and also generate GraphQL queries for accessing the indexed data.

In the **`src`** directory, open the file named **`schema.gql`** and define the schema for the tokens and account holders entity we'll be tracking:

```graphql
scalar BigInt

type AccountToken {
  id: String! # Equal to <tokenAddress>-<accountAddress>
  account: String
  token: Token
  balance: Float # Parsed balance based on token decimals
  rawBalance: String # Raw balance without decimals
  modified: Int # Last modified timestamp in seconds
  tx: String # Last transaction that modified balance
}

type Token {
  id: String! # Token address
  decimals: Int
  name: String
  symbol: String
  totalSupply: BigInt
}
```

Checkpoint will use the above entities to generate a MySQL (or postgress) database table named **`tokens`** and **`accounttokens`** with columns matching the defined fields. It will also generate a list of GraphQL queries to enable querying indexed data.

**💡** Note that entities are converted to lower case and pluralized

**Step 5: Creating Data Writers**

Data writers are typescript functions that get invoked by Checkpoint when it discovers a block containing a relevant event. A data writer is responsible for writing records to the database. These records will eventually be exposed via Checkpoint's GraphQL endpoint.

In the **`src`** directory, create a file named **`writers.ts`** and define the data writer functions for the **`handleTransfer`** event:

```tsx
import { convertToDecimal, getEvent } from './utils/utils';
import type { CheckpointWriter } from '@snapshot-labs/checkpoint';
import { createToken, isErc20, loadToken, newToken, Token } from './utils/token';
import { createAccount, newAccount, Account, loadAccount } from './utils/account';

export async function handleTransfer({
  block,
  tx,
  rawEvent,
  mysql
}: Parameters<CheckpointWriter>[0]) {
  // Start manipulating your data here
}
```

At this point you can launch Checkpoint and get the data corresponding to the events you’d like and it’s associated block (block\_hash, block\_number, timestamp…) and tx (contract\_address, transaction\_hash,…) objects. But we want to manipulate it in order to index the entity fields we created earlier

**Step 6: Filter contracts interface**

First of all, you may want to filter the emitting contracts to avoid unwanted events. Here for exemple we want to index only erc20 contracts, but the erc721 contracts also contains a **`transfer`** event so what we’ll do is that we’ll create a function called **`isErc20()`** in writers.ts to filter contracts based on desired and undesired functions. We will simply retrieve the abi of the emitting contract and check if it respects the conditions we’ve set in `**desired**` and `**undesired**` function:

```tsx
export async function isErc20(address: string, block_number: number) {
  const desiredFunctions = [
    'name',
    'decimals',
    'totalSupply',
    'balanceOf',
    'transfer',
    'transferFrom',
    'approve',
    'allowance'
  ];
  const undesiredFunctions = ['tokenURI'];

  const classHash = await provider.getClassHashAt(address, block_number);
  const contractClass = await provider.getClassByHash(classHash);

  const hasFunctions = desiredFunctions.every(func =>
    contractClass.abi?.find(token => token.name === func && token.type === 'function')
  );
  const hasNoFunctions = undesiredFunctions.every(
    func => !contractClass.abi?.find(token => token.name === func && token.type === 'function')
  );

  const result = hasFunctions && hasNoFunctions;
  console.log(result, `Smart contract ${result ? 'matches' : "doesn't match"} desired functions`);
  return result;
}
```

**Step 7: Handle sender and receiver values**

Now that we are sure we have the right event we can manipulate its data as we wish. Here we first check that the erc20 in question is already known in our database. If it is the case we call it otherwise we create it. And we do the same process for the sender and the receiver of the token. Then we subtract from the sender the value of the transfer and add it to the receiver. It looks something like this:

```tsx
// If token isn't indexed yet we add it, else we load it
if (await newToken(rawEvent.from_address, mysql)) {
  token = await createToken(rawEvent.from_address);
  await mysql.queryAsync(`INSERT IGNORE INTO tokens SET ?`, [token]);
} else {
  token = await loadToken(rawEvent.from_address, mysql);
}

// If accounts aren't indexed yet we add them, else we load them
// First with fromAccount
const fromId: string = token.id.slice(2) + '-' + data.from.slice(2);
if (await newAccount(fromId, mysql)) {
  fromAccount = await createAccount(token, fromId, tx, block);
  await mysql.queryAsync(`INSERT IGNORE INTO accounttokens SET ?`, [fromAccount]);
} else {
  fromAccount = await loadAccount(fromId, mysql);
}

// Then with toAccount
const toId: string = token.id.slice(2) + '-' + data.to.slice(2);
if (await newAccount(toId, mysql)) {
  toAccount = await createAccount(token, toId, tx, block);
  await mysql.queryAsync(`INSERT IGNORE INTO accounttokens SET ?`, [toAccount]);
} else {
  toAccount = await loadAccount(toId, mysql);
}

// Updating balances
fromAccount.balance -= convertToDecimal(data.value, token.decimals);
toAccount.balance += convertToDecimal(data.value, token.decimals);
// Updating raw balances
fromAccount.rawBalance = BigInt(fromAccount.rawBalance) - BigInt(data.value);
toAccount.rawBalance = BigInt(toAccount.rawBalance) + BigInt(data.value);
// Updating modified field
fromAccount.modified = block.timestamp;
toAccount.modified = block.timestamp;
// Updating tx field
fromAccount.tx = tx.transaction_hash!;
toAccount.tx = tx.transaction_hash!;
```

**Step 8: Store the updated values**

Now we need to store the updated values in our database. To do this you just have to make a call to your database by executing the sql request you want via your sql instance (**`mysql`** here). Here we want to update **`fromAccount`** and **`toAccount`** so we will proceed like this:

```tsx
// Indexing accounts
await mysql.queryAsync(
  `UPDATE accounttokens SET balance=${
    fromAccount.balance
  }, rawBalance=${fromAccount.rawBalance.toString()}, modified=${fromAccount.modified}, tx='${
    fromAccount.tx
  }' WHERE id='${fromAccount.id}'`
);
await mysql.queryAsync(
  `UPDATE accounttokens SET balance=${
    toAccount.balance
  }, rawBalance=${toAccount.rawBalance.toString()}, modified=${toAccount.modified}, tx='${
    toAccount.tx
  }' WHERE id='${toAccount.id}'`
);
```

**Step 9: Run and test your indexer**

Lastly to test your indexer you need to run a database instance (mysql or postgres) locally and set the **`database_url`** environment variable in your .env (refer to the .env.exemple file). Then you just have to `yarn` to install the dependencies and then `yarn dev` to run Checkpoint. To check if everything is working well you may want to query your indexed data by running GraphQL queries on **`http://localhost:3000`**. For this project we’ll be using :

```graphql
query {
  accounttokens {
    id
    account
    token {
      id
      name
      symbol
      decimals
      totalSupply
    }
    balance
    rawBalance
    modified
    tx
  }
}
```

**Conclusion**

That’s it! You should have Checkpoint running, indexing your global\_events data and serving this indexed data via graphql.
