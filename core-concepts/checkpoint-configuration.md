# Checkpoint configuration

Checkpoint uses a configuration object to determine which networks and contract information it will be indexing.

```typescript
// Configuration used to initialize Checkpoint
export interface CheckpointConfig {
  network_node_url: string;
  optimistic_indexing?: boolean;
  sources?: ContractSourceConfig[];
  templates?: { [key: string]: ContractTemplate };
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

export interface ContractTemplate {
  events: ContractEventConfig[];
};
```

