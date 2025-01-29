# Internal data query

Checkpoint also exposes some queries to inspect the internal state of how the indexer is running. Typically, internal queries are usually prefixed with an underscore (`_`) and are structured in a similar way to how [Entity Schema queries](entity-schema.md#query-generation) are generated.

Currently, Checkpoint exposes the following internal data queries:

#### 1. \`\_metadata\` and \`\_metadatas\` query fields.

These are used to query single or multiple metadata records. Metadata records are key-value pairs of data used by Checkpoint internally to describe the state of its process. These queries are  defined as:

```graphql
type Query {
    """ queries a single metadata value by it's id (key) """
    _metadata(id: ID!, indexer: String): _Metadata

    """ queries multiple metadata values """
    _metadatas(
    indexer: String
    first: Int
    skip: Int
    orderBy: String
    orderDirection: OrderDirection
    where: Where_Metadata
    ): [_Metadata]
}

""" Core metadata values used internally by Checkpoint """
type _Metadata {
    """ example id: last_indexed_block """
    id: ID!
    indexer: String!
    value: String
}
```

For starters, you can execute the following query to see a list of all metadata values exposed by Checkpoint:

```graphql
query {
    _metadatas {
        id
        _indexer
        value
    }
}
```

#### 2. \`\_checkpoint\` and \`\_checkpoints\` query fields

Internally, Checkpoint keeps track of blocks where contracts event are found. These records are usually used by Checkpoint to speed up re-indexing when restarted. The `_checkpoint(s)` queries provide a way to query these blocks. The results can be exported and used to seed another Checkpoint instance running on another machine.

These queries are defined as:

```graphql
type Query {
    """ queries a single _checkpoint entry by it's id"""
    _checkpoint(id: ID!, indexer: String): _Checkpoint
    
    """ queries multiple _checkpoint entires """
    _checkpoints(
    indexer: String
    first: Int
    skip: Int
    orderBy: String
    orderDirection: OrderDirection
    where: Where_Checkpoint
    ): [_Checkpoint]
}

""" Contract and Block where its event is found. """
type _Checkpoint {
    """ id computed as last 5 bytes of sha256(contract+block) """
    id: ID!
    block_number: Int!
    contract_address: String!
}
```

For example, you can run the following query to fetch all blocks where the event of a particular contract can be found:

```graphql
query {
    _checkpoints(indexer: "mainnet", where: {
        contract_address: "0x<contract-address>"
    }) {
        block_number
    }
}
```
