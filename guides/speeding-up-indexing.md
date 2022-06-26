---
description: Seeding Checkpoint Blocks
---

# Speeding up Indexing

By default, Checkpoint scans each block for relevant events and invokes the appropriate data writer to handle the event. This sequential scanning can be pretty slow, so Checkpoint provides a way to skip through irrelevant blocks.

You can seed Checkpoint with a list of blocks, and Checkpoint will start by scanning this list of blocks first and invokes the appropriate data writers for any events founds. Once Checkpoint checks through the list of seeded blocks, it continues sequential scanning from the next block.

There is a `seedCheckpoints` method defined on Checkpoint instances:

```typescript
seedCheckpoints(Array<{ contract: string; blocks: number[] }>): Promise<void>
```

This `seedCheckpoints` method should be called before starting the indexer, and the method can be called with as many contracts and blocks you already have.\


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

checkpoint.seedCheckpoint(checkpointBlocks)
.then(() => checkpoint.start());
```

### Exporting Checkpoint Blocks

For each block where Checkpoint encounters relevant events, it creates a record in the database to keep track of it. These records can be queried through the GraphQL API using the `_checkpoints` query. You can read more about how to do that [here](../core-concepts/internal-data-query.md#2.-\_checkpoint-and-\_checkpoints-query-fields).

You can set up a process (or external service) that periodically uses the `_checkpoints` query to fetch the latest blocks and export them for archiving or sharing with another instance of Checkpoint.

Using our Poster contract examples from above, to fetch all blocks where Checkpoint has encountered an event, you can run the following query:

```graphql
query {
    _checkpoints(
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
