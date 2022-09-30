# Entity schema

Checkpoint requires a set of defined GraphQL Schema Objects. The schema objects will be used to create a database structure for indexing this data and also generating GraphQL Queries for accessing the indexed data.

In Checkpoint terms, these schema objects are called `Entities`, an Entity can be defined as a GraphQL Object with a unique name and an `id` field.

For example:

```graphql
""" Vote is a valid entity """
type Vote {
  id: String!
  voter: User
  space: String
  proposal: Int
  choice: Int
  vp: Int
  created: Int
}

type User {
  id: String!
  vote_count: Int
  created: Int
}
```

`Vote` and `User` are valid entities because their names are unique (within the schema) and contain an `id` field.

### Query generation

Checkpoint generates two Query fields for each of the defined entities. One for querying a single entity record by its id and the second for querying multiple records of an entity.

For example, using the earlier defined `User` entity, Checkpoint will generate two query fields like:

```graphql
type Query {
  user(id: ID): User
  users(
    first: Int
    skip: Int
    orderBy: String
    orderDirection: OrderDirection
    where: WhereUser
  ): [User]
}

type WhereUser {
  id: String
  id_in: [String]
  vote_count_gt: Int
  vote_count_gte: Int
  vote_count_lt: Int
  vote_count_lte: Int
  vote_count: Int
  vote_count_in: [Int]
  created_gt: Int
  created_gte: Int
  created_lt: Int
  created_lte: Int
  created: Int
  created_in: [Int]
}
```

Things to note:

* The name of the single record entity query is derived from the entity's name lowercased.
* The name of the multi-record entity query is derived from the entity's name lowercased with an `s` suffix.
* The generated `Where*` types fields for multi-record are derived based on original fields in the entity, and non-null values are treated a `AND` where filters when being executed against the database.

