# BigInt and BigDecimal

Checkpoint provides support for `BigInt`, `Decimal`, `BigDecimal`. It's also possible to define custom (or define existing) decimal types.

### PostgreSQL mappings

BigInt and BigDecimal types are mapped to following PostgreSQL types by default:

* `BigInt` -> `bigint`
* `Decimal` -> `decimal(10, 2)`
* `BigDecimal` -> decimal`(20, 8)`

### Using BigInt and BigDecimal types

To use default (or custom) BigInt and BigDecimal type it needs to be predeclared in GraphQL schema using `scalar` keyword. Once it's declared it can be used as field's type.

```graphql
scalar BigInt

type User {
  votes_count: BigInt
}  
```

### Custom decimal types

It's possible to define new or to redefine existing decimal types using overrides. This file should be called overrides.json and stored in your source directory.

```json
{
  "decimal_types": {
    "BigDecimalVP": {
      "p": 60,
      "d": 0
    }
  }
}

```

{% hint style="info" %}
When you use custom decimal types your decimal types need to be static and stored in overrides.json file. This is because it's used in codegen process of ORM.
{% endhint %}

You can then pass those to Checkpoint via options:

```typescript
import overridesConfig from './overrides.json';

// ...

const checkpoint = new Checkpoint(schema, {
  overridesConfig
});
```
