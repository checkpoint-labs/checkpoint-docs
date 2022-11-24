# ApolloServer usage

Checkpoint supports Express.js by default, but it's also possible to use Checkpoint with ApolloServer. Assuming ApolloServer is already installed in your Checkpoint project you can start ApolloServer using Checkpoint's schema with following code:

```typescript
import Checkpoint, { LogLevel, createGetLoader } from '@snapshot-labs/checkpoint';
import { ApolloServer } from '@apollo/server';
import { startStandaloneServer } from '@apollo/server/standalone';

// Checkpoint initialization

async function run() {
  const server = new ApolloServer({
    schema: checkpoint.getSchema()
  });

  const { url } = await startStandaloneServer(server, {
    listen: { port: 3000 },
    context: async () => {
      const baseContext = checkpoint.getBaseContext();
      return {
        ...baseContext,
        getLoader: createGetLoader(baseContext)
      };
    }
  });

  console.log(`🚀  Server ready at: ${url}`);
}

run();
```

{% hint style="warning" %}
Currently only Express.js is officially supported. ApolloServer compatibility is only provided as alternative for user's convenience, but it's not guaranteed that it will support all ApolloServer's features.
{% endhint %}
