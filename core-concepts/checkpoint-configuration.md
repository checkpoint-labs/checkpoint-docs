# Checkpoint configuration

Checkpoint uses a configuration object to determine which networks and contract information it will be indexing.

```typescript
// Configuration used to initialize Checkpoint
export interface CheckpointConfig {
  network_node_url: string;
  optimistic_indexing?: boolean;
  fetch_interval?: number; 
  tx_fn?: string;
  global_events?: ContractSourceConfig[];
  sources?: ContractSourceConfig[];
  templates?: { [key: string]: ContractTemplate };
  // mapping from ABI name to contract ABI
  abis?: { [key: string]: any };
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
  // name of ABI (if used it Checkpoint will parse events using provided ABI)
  abi?: string;
  // start block number
  start: number;
  // callback function in writer to handle deployment
  deploy_fn?: string;
  events: ContractEventConfig[];
}

export interface ContractTemplate {
  // name of ABI (if used it Checkpoint will parse events using provided ABI)
  abi?: string;
  events: ContractEventConfig[];
};
```
