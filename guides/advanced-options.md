# Advanced options

When initializing Checkpoint library, there is an optional parameter that can be passed to configure some extra behavior of the library.

The type definition for the option object is:

```typescript
export interface CheckpointOptions {
  // Set the log output levels for checkpoint. Defaults to Error.
  logLevel?: LogLevel;
  // optionally format logs to pretty output.
  // will require installing pino-pretty. Not recommended for production.
  prettifyLogs?: boolean;
  // Optional database connection string. For now only accepts mysql database
  // connection string. If no provided will default to looking up a value in
  // the DATABASE_URL environment.
  dbConnection?: string;
}
```

This option can be provided as the final argument to the Checkpoint constructor like this:

```typescript
import Checkpoint, { LogLevel } from '@snapshot-labs/checkpoint';

const checkpoint = new Checkpoint(..., {
    logLevel: LogLevel.Info;
})
```

See more about the configuration options below.

### Database options

By default, when a `Checkpoint` object is started, it looks up the `DATABASE_URL` environment variable, but with `dbConnection` option parameter, you can specify a different connection string within the codebase itself and this value will override the `DATABASE_URL` value in the environment when connecting to the database.

### Logging options

There are six (6) log levels currently supported by Checkpoint, these are:

```typescript
// LogLevel to control what levels of logs are output.
export enum LogLevel {
  // silent to disable all logging
  Silent = 'silent',
  // fatal to log unrecoverable errors
  Fatal = 'fatal',
  // error to log general errors
  Error = 'error',
  // warn to log alerts or notices
  Warn = 'warn',
  // info to log useful information
  Info = 'info',
  // debug to log debug and trace information
  Debug = 'debug'
}
```

In a non-production environment, you can set the `prettifyLogs` option to `true` and this will output a pretty version
