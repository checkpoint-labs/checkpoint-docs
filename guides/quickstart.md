# Quickstart

In this guide, we will set up a project that uses Checkpoint to expose data.

### Setup the starter template project

First, clone the Checkpoint starter template project that can be found at: [https://github.com/snapshot-labs/checkpoint-template](https://github.com/snapshot-labs/checkpoint-template).&#x20;

To successfully, follow this guide and run the template project, you'll need the following dependencies set up on your computer:

* Node.js (>=14.x.x)
* Yarn
* PostgreSQL (or Docker to quickly run an image).

#### 1. Setup dependencies

First, install Node dependencies by running the following command in the project's directory:

```bash
yarn
```

#### 2. Run project

Checkpoint projects (and by extension this template) require a PostgreSQL database connection to store indexed data. If you have a PostgreSQL server running, then create a copy of the `.env.example` file and name `.env`. Then update the `DATABASE_URL` value in the `.env` file to match the connection string to your database.

{% hint style="info" %}
If you have Docker on your computer, you can quickly startup a MySQL server by running the following command in a separate terminal:

```bash
docker run --name checkpoint-postgres \
    -p "5432:5432" \
    -e POSTGRES_PASSWORD=default_password -d postgres
```

And then you can use `postgres://postgres:default_password@localhost:5432/postgres` as your .env files `DATABASE_URL` value .
{% endhint %}

Next, start the server by running:

```bash
yarn dev
```

This will start the indexing process and also startup the GraphQL server for querying indexed information.

### Querying the API data

This starter project defines a `Post` entity in its `src/schema.gql` schema file and is setup to write data based on that structure. Read more on how Entity schemas are used to generate GraphQL queries [here](../core-concepts/entity-schema.md).

Now that the server is running, open up your browser and visit http://localhost:3000. This will show a GraphiQL interface to explore the exposed data.&#x20;

Try executing the following query:

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

You should get a response similar to the one in this image:

![Screenshot of sample query and response](<../.gitbook/assets/Screenshot 2022-06-01 at 1.38.04 am.png>)

The response to your queries depends on how much on-chain data Checkpoint has indexed. If the server has run long enough, you'll get more results. Checkpoint exposes a `_metadata` query, which can be used to track the latest block it has indexed. Example of this is:

```graphql
query {
  _metadata(id: "last_indexed_block") {
    value
  }
}
```



### Template project structure

Here is a quick description of important files within the template project and what each does.

```
├── src
│   ├── checkpoints.json
│   ├── config.ts
│   ├── index.ts
│   ├── schema.gql
│   ├── utils.ts
│   └── writers.ts
...
```

* [**src/schema.gql**](https://github.com/snapshot-labs/checkpoint-template/blob/master/src/schema.gql): Defines the `Post` entity Checkpoint uses to generate API queries.
* [**src/config.ts**](https://github.com/checkpoint-labs/checkpoint-template/blob/master/src/config.ts): Defines the [CheckpointConfig](../core-concepts/checkpoint-configuration.md) object used to initialize Checkpoint and it contains the details of a Poster contracts deployed on Starknet mainnet and Starknet sepolia networks.
* [**src/writers.ts**](https://github.com/snapshot-labs/checkpoint-template/blob/master/src/writers.ts): Defines the [Writers](../core-concepts/data-writers.md) responsible for writing `Post` data whenever Checkpoint encounters a `new_post` event.
* [**src/index.ts**](https://github.com/snapshot-labs/checkpoint-template/blob/master/src/index.ts): This is the entrypoint, that initializes Checkpoint, configures indexers for each network and exposes the GraphQL API.
* [**src/checkpoints.json**](https://github.com/snapshot-labs/checkpoint-template/blob/master/src/checkpoints.json): Defines blocks where we are sure the contracts events exists. This is used to seed checkpoint inside the index.ts file to speed up Checkpoints indexing.

You are encouraged to try modifying the template project to understand how Checkpoint works and quickly get started using it to index your contracts data on Starknet.



