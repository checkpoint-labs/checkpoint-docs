# Checkpoint configuration

Checkpoint uses a configuration JSON object to determine which networks and contract information it will be indexing.

```typescript
// Configuration used to initialize Checkpoint
export interface CheckpointConfig {
  network: 'mainnet-alpha' | 'goerli-alpha';
  sources: ContractSourceConfig[];
}

export interface ContractEventConfig {
  // name of event in the contract
  name: string;
  // callback function in writer
  fn: string;
}

export interface ContractSourceConfig {
  // contract address
  contract: string;
  // start block number
  start: number;
  // callback function in writer to handle deployment
  deploy_fn: string;
  events: ContractEventConfig[];
}
```

