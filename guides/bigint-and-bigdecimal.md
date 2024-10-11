# BigInt and BigDecimal

Checkpoint provides support for `BigInt`, `Decimal`, `BigDecimal`. It's also possible to define custom (or define existing) decimal types.

### MySQL mappings

BigInt and BigDecimal types are mapped to following MySQL types by default:

* `BigInt` -> `BIGINT`
* `Decimal` -> `DECIMAL(10, 2)`
* `BigDecimal` -> `DECIMAL(20, 8)`

### Using BigInt and BigDecimal types

To use default (or custom) BigInt and BigDecimal type it needs to be predeclared in GraphQL schema using `scalar` keyword. Once it's declared it can be used as field's type.

```graphql
scalar BigInt

type User {
  votes_count: BigInt
}  
```

{% hint style="warning" %}
When writing to those fields make sure that you are using JavaScript type that `mysql` package understands and won't result in loss of precision. `BigInt` and `String` should be preferred.
{% endhint %}

### Custom decimal types

It's possible to define new or to redefine existing decimal types using [checkpoint configuration](../core-concepts/checkpoint-configuration.md).

```typescript
import { CheckpointConfig } from "@snapshot-labs/checkpoint";

const config: CheckpointConfig = {
  network_node_url:
    "https://starknet-goerli.infura.io/v3/46a5dd9727bf48d4a132672d3f376146",
  sources: [
    {
      contract:
        "0x04d10712e72b971262f5df09506bbdbdd7f729724030fa909e8c8e7ac2fd0012",
      start: 185778,
      deploy_fn: "handleDeploy",
      events: [
        {
          name: "new_post",
          fn: "handleNewPost",
        },
      ],
      decimal_types: {
        Decimal: {
          p: 14,
          d: 10,
        },
        BigDecimal: {
          p: 20,
          d: 8,
        },
        EvenBiggerDecimal: {
          p: 40,
          d: 16,
        },
      },
    },
  ],
};
```
