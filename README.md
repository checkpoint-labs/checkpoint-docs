# Introduction

Checkpoint is a library for indexing data from Starknet events and making it accessible through GraphQL. Checkpoint is inspired by The Graph and focused on providing similar functionality for Starknet.

## How it works

Checkpoint learns what and how to index Starknet data based on configuration parameters, known as the [Checkpoint configuration](core-concepts/checkpoint-configuration.md). Checkpoint configuration defines the smart contracts of interest and the relevant events to be tracked. The logic for how to map events to data that Checkpoint will store in its database is defined in user-defined Javascript functions called [Checkpoint data writers](core-concepts/data-writers.md). Checkpoint then exposes the stored data to the public through a GraphQL API.

This diagram gives more detail about the flow of data once a Checkpoint instance has started processing Starknet transactions:

<div align="left">

<img src=".gitbook/assets/image.png" alt="Checkpoint Flow Diagram">

</div>

As highlighted in the flow diagram above:

1. A decentralized application adds data to Starknet through a transaction on a smart contract.
2. The smart contract emits one or more events while processing the transaction. Checkpoint continually scans Starknet for new blocks and checks if it contains events for your configured contracts they may contain.
3. Checkpoint calls the data writer function for the respective event. [Writers](core-concepts/data-writers.md) are responsible for writing entity objects to the database.
4. The decentralized application queries the Checkpoint GraphQL API. Checkpoint translates the GraphQL queries into SQL queries to fetch this entity data from the database. The decentralized application displays this data in a rich UI for end-users, which they can also use to issue new transactions on Starknet, and the cycle repeats.

For each block where Checkpoint encounters relevant events, it creates a record in the database to keep track of it. These records can be queried through the GraphQL API using the `_checkpoints` query. You can read more about how to do that [here](core-concepts/internal-data-query.md#2.-\_checkpoint-and-\_checkpoints-query-fields).&#x20;

## Installation

Checkpoint is an NPM package that can be installed through the following command:

```bash
npm install @snapshot-labs/checkpoint@beta
```

## Guides: jump right in

Follow our handy guides to get started with Checkpoint as quickly as possible:

{% content-ref url="guides/quickstart.md" %}
[quickstart.md](guides/quickstart.md)
{% endcontent-ref %}

{% content-ref url="guides/step-by-step-setup.md" %}
[step-by-step-setup.md](guides/step-by-step-setup.md)
{% endcontent-ref %}

{% content-ref url="guides/advanced-options.md" %}
[advanced-options.md](guides/advanced-options.md)
{% endcontent-ref %}

## Core concepts: dive a little deeper

Learn the fundamentals of Checkpoint to get a deeper understanding of how it works:

{% content-ref url="core-concepts/checkpoint-configuration.md" %}
[checkpoint-configuration.md](core-concepts/checkpoint-configuration.md)
{% endcontent-ref %}

{% content-ref url="core-concepts/entity-schema.md" %}
[entity-schema.md](core-concepts/entity-schema.md)
{% endcontent-ref %}

{% content-ref url="core-concepts/data-writers.md" %}
[data-writers.md](core-concepts/data-writers.md)
{% endcontent-ref %}
