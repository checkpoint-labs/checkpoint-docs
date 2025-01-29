# Indexing Ethereum contracts

Checkpoint initially supported indexing Starknet contracts only, but it can also index Ethereum contracts now.

{% hint style="info" %}
Checkpoint can work with any network and chain as long as there is [a provider](https://github.com/checkpoint-labs/checkpoint/tree/master/src/providers) implemented for it. Currently there is official support for Starknet and Ethereum (or any other EVM network), but external providers are supported as well.
{% endhint %}

Usage with Ethereum is very similar to usage with Starknet, differences are:

* Your writers should be using `evm.Writer` instead of `starknet.Writer`.
* You should create indexer using `new evm.EvmIndexer` instead of `new starknet.StarknetIndexer`.

```typescript
import Checkpoint, { evm } from '@snapshot-labs/checkpoint';

const handleProxyDeployed: evm.Writer = async ({ blockNumber, event }) => {};
const writers = { handleProxyDeployed };

const ethIndexer = new evm.EvmIndexer(writers);

const checkpoint = new Checkpoint(...);
checkpoint.addIndexer('eth', config, ethIndexer);
checkpoint.start();
```
