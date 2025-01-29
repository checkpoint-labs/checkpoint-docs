# Data writers (Evm)

Data writers are callback functions that Checkpoint invokes when the event of a contract is discovered at a particular block.

```typescript
type EvmWriter = (args: {
  blockNumber: number;
  eventIndex?: number;
  source: ContractSourceConfig;
  helpers: { executeTemplate };
  tx: Transaction;
  block: BlockWithTransactions | null;
  rawEvent?: Log;
  event?: LogDescription;
}) => Promise<void>;
```

