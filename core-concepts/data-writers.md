# Data writers

Data writers are callback functions that Checkpoint invokes when the event of a contract is discovered at a particular block.

```typescript
type CheckpointWriter = (args: {
  tx: Transaction;
  block: FullBlock;
  blockNumber: number;
  eventIndex?: number;
  rawEvent?: Event;
  event?: ParsedEvent;
  source: ContractSourceConfig;
  instance: Checkpoint;
}) => Promise<void>;
```

