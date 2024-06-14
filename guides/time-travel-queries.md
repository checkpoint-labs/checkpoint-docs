# Time-travel queries

Checkpoint's time travel queries feature allows to access historical data at a specific block number. By specifying a block, you can retrieve data from that exact point in the past.

```graphql
{
  proposals (block: 123456) {
    id
    start
  }
}
```
