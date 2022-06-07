# Querying data

Once Checkpoint has run for a while and has written some data to its database, you can query this data using the generated GraphQL API.

You can mount the query endpoint on any port you like using the `graphql` handler exported by Checkpoint's object. Like this:

```typescript
import express from 'express';

const checkpoint = new Checkpoint(...);

const app = express();
app.use('/', checkpoint.graphql);

const PORT = 3000;
app.listen(PORT, () => console.log(`Listening at http://localhost:${PORT}`));
```

This will mount a GraphQL endpoint that can be accessible at http://localhost:3000/graphql (and a GraphiQL interface at http://localhost:3000/graphiql).&#x20;



Checkpoint exposes two types of queries:&#x20;

1. [Entity Queries](../core-concepts/entity-schema.md#query-generation)
2. [Internal data Queries](../core-concepts/internal-data-query.md)

These  enable users to fetch information about  being indexed:

