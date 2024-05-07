---
description: >-
  Checkpoint introduces a robust ORM to streamline your interactions with the
  database.
---

# Models

Checkpoint enables data manipulation in your blockchain database without having to construct SQL queries directly. It features a capability to generate model objects from a GraphQL schema, and this guide will lead you through the process of creating and saving a model object into your database.

### Generating Models

The first step is to generate the models from your GraphQL schema. This is done with the `checkpoint generate` command. Run it in your project environment like this:

```bash
yarn checkpoint generate
```

After running this command, your models will be created based on your GraphQL schema.

### Importing Models

To use the generated models in your scripts, you need to import them. This is done with an `import` statement at the top of your file. For instance, if you've generated a `Post` model, you can import it like this:

```javascript
import { Post } from '../.checkpoint/models';
```

This allows you to use the `Post` class to create new posts and save them to the database.

### Defining Your Model

Each model is defined in a GraphQL schema file, such as `schema.gql`. Every model object will have a unique `id` along with other properties. Here's an example of a `Post` model:

```graphql
scalar Text

type Post {
  id: String!
  author: String!
  content: Text!
  tag: String
  tx_hash: String!
  created_at: Int!
  created_at_block: Int!
}
```

Please note, this `Post` model is merely an example. Your actual model will have properties specific to your needs.

### Creating a model Instance

To create a new instance of your model, you instantiate it with the `id`. Here's how you'd create a new `Post`:

```javascript
const post = new Post(`${author}/${tx.transaction_hash}`);
```

You then set the other properties:

```javascript
post.author = author;
post.content = content;
post.tag = tag;
post.tx_hash = tx.transaction_hash;
post.created_at = timestamp;
post.created_at_block = blockNumber;
```

### Edit a model instance

Once the instance is defined, you can load an entity and edit it using `loadEntity` like that :&#x20;

```typescript
const post = await Post.loadEntity(id);
post.author = '0x123...';
await post.save();
```

### Saving a model Instance

Once the instance is fully defined, you can save it into your database using the `save` method. This operation is asynchronous, so you should use `await`:

```javascript
await post.save();
```
