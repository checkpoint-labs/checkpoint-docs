# Data writers (Starknet)

Data writers are callback functions that Checkpoint invokes when the event of a contract is discovered at a particular block.

```typescript
type StarknetWriter = (args: {
  blockNumber: number;
  eventIndex?: number;
  source: ContractSourceConfig;
  helpers: { executeTemplate };
  tx: Transaction;
  block: FullBlock | null;
  rawEvent?: Event;
  event?: ParsedEvent;
}) => Promise<void>;
```

