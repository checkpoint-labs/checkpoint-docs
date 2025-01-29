# Advanced options

When initializing Checkpoint library there is an optional parameter that can be passed to configure some extra behavior of the library.

The type definition for the option object is:

```typescript
export interface CheckpointOptions {
  // Setting to true will trigger reset of database on config changes.
  resetOnConfigChange?: boolean;
  // Set the log output levels for checkpoint. Defaults to Error.
  // Note, this does not affect the log outputs in writers.
  logLevel?: LogLevel;
  // optionally format logs to pretty output.
  // Not recommended for production.
  prettifyLogs?: boolean;
  // Optional database connection string. For now only accepts PostgreSQL and MySQL/MariaDB
  // connection string. If no provided will default to looking up a value in
  // the DATABASE_URL environment.
  dbConnection?: string;
  overridesConfig?: OverridesConfig;
}
```

This option can be provided as the final argument to the Checkpoint constructor like this:

```typescript
import Checkpoint, { LogLevel } from '@snapshot-labs/checkpoint';

const checkpoint = new Checkpoint(..., {
    logLevel: LogLevel.Info;
})
```

See more about the configuration options below.

### Database options

By default, when a `Checkpoint` object is started, it looks up the `DATABASE_URL` environment variable, but with `dbConnection` option parameter, you can specify a different connection string within the codebase itself and this value will override the `DATABASE_URL` value in the environment when connecting to the database.

### Logging options

There are six (6) log levels currently supported by Checkpoint, these are:

```typescript
// LogLevel to control what levels of logs are output.
export enum LogLevel {
  // silent to disable all logging
  Silent = 'silent',
  // fatal to log unrecoverable errors
  Fatal = 'fatal',
  // error to log general errors
  Error = 'error',
  // warn to log alerts or notices
  Warn = 'warn',
  // info to log useful information
  Info = 'info',
  // debug to log debug and trace information
  Debug = 'debug'
}
```

In a non-production environment, you can set the `prettifyLogs` option to `true` and this will output a pretty version

### Overrides config

Used to redefine [decimal types](bigint-and-bigdecimal.md#custom-decimal-types) available in the schema.

### Seed checkpoints manually

You can seed Checkpoint with a list of blocks, and Checkpoint will start by scanning this list of blocks first and invokes the appropriate data writers for any events founds. Once Checkpoint checks through the list of seeded blocks, it continues sequential scanning from the next block.

There is a `seedCheckpoints` method defined on Checkpoint instances:

```typescript
seedCheckpoints(Array<{ contract: string; blocks: number[] }>): Promise<void>
```

This `seedCheckpoints` method should be called before starting the indexer, and the method can be called with as many contracts and blocks you already have.\\

For example, in the checkpoint template [project](https://github.com/snapshot-labs/checkpoint-template), seed blocks for the Poster contracts are defined in the \`checkpoints.json\` file as:

```json
[
  {
    "contract": "0x04d10712e72b971262f5df09506bbdbdd7f729724030fa909e8c8e7ac2fd0012",
    "blocks": [185778, 185780, 220016, 220138, 221984]
  }
]
```

And this array is used as an argument when [initializing checkpoint](https://github.com/snapshot-labs/checkpoint-template/blob/master/src/index.ts#L27) like:

```typescript
import checkpointBlocks from './checkpoints.json';

//...
// after initialising a checkpoint object

checkpoint.seedCheckpoint('mainnet', checkpointBlocks)
    .then(() => checkpoint.start());
```

### Exporting checkpoint blocks

For each block where Checkpoint encounters relevant events, it creates a record in the database to keep track of it. These records can be queried through the GraphQL API using the `_checkpoints` query. You can read more about how to do that [here](../core-concepts/internal-data-query.md#2.-_checkpoint-and-_checkpoints-query-fields).

You can set up a process (or external service) that periodically uses the `_checkpoints` query to fetch the latest blocks and export them for archiving or sharing with another instance of Checkpoint.

Using our Poster contract examples from above, to fetch all blocks where Checkpoint has encountered an event, you can run the following query:

```graphql
query {
    _checkpoints(
        indexer: "mainnet"
        first: 100,
        where: {
        contract_address: "0x04d10712e72b971262f5df09506bbdbdd7f729724030fa909e8c8e7ac2fd0012",
block_number_gt: 221984 
    }) {
        block_number
    }
}
```

The above query will fetch the subsequent 100 blocks after the \`221984\` block where Checkpoint encounters an events for the [Poster](https://github.com/snapshot-labs/starknet-poster) contracts address (`0x04d10712e72b971262f5df09506bbdbdd7f729724030fa909e8c8e7ac2fd0012`).
