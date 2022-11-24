# One-to-one relations

It's possible to create one-to-one relations in schema. Those relations will be then available for querying via GraphQL.

### Define relations in schema

To define one-to-one relation nested entity must have `id` field that is either `String!` or `ID!`. In parent entity create field (with any name) that is of nested entity's type (can be nullable or required).

```graphql
type Space {
  id: String!
  name: String
}

type Proposal {
  id: String!
  name: String
  space: Space!
}
```

### Create entities in writer&#x20;

When creating entities in writer set `Proposal`'s `space` field to the value of `Space`'s `id` field.

```typescript
const proposal = {
  id: 'proposal_id',
  name: 'Proposal name',
  space: 'space_id'
}

await mysql.queryAsync('INSERT INTO proposals SET ?', [proposal]);
```

### Query data via GraphQL

It's now possible to nest space entity when querying proposals. You can still filter proposals by space (limited to `Space`'s `id` currently).

```graphql
{
  proposals(first: 10, where: {space: "0x00b60f2a154b9aaec8e4bec8e04f86d6cd92a9c993871e904bd815962603492d"}) {
    id
    space {
      id
      name
    }
  }
}
```
