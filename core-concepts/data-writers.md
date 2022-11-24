# Data writers

Data writers are callback functions that Checkpoint invokes when the event of a contract is discovered at a particular block.



````typescript
/**
 * Callback function invoked by checkpoint when a contract event
 * is encountered. A writer function should use the `mysql`
 * object to write to the database entities based on the require logic.
 *
 * For example, if a graphql Entity is defined in the schema:
 *
 * ```graphql
 * type Vote {
 *  id: ID!
 *  voter: String!
 * }
 * ```
 *
 * Then you can insert into the entity into the database like:
 * ```typescript
 * await args.mysql.queryAsync('INSERT INTO votes VALUES(?, ?);', ['voteId', 'voters-address']);
 * ```
 *
 * Note, Graphql Entity names are lowercased with an 's' suffix when
 * interacting with them in the database.
 *e
 */
type CheckpointWriter = (args: {
  tx: Transaction;
  block: FullBlock;
  event?: Event;
  source: ContractSourceConfig;
  mysql: AsyncMySqlPool;
  instance: Checkpoint;
}) => Promise<void>;

/**
 * Object map of events to CheckpointWriters.
 *
 * The CheckpointWriter function will be invoked when an
 * event matching a key is found.
 *
 */
interface CheckpointWriters {
  [event: string]: CheckpointWriter;
}
````

