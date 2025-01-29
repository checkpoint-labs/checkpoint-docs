# Templates

In many cases there are contracts that should be tracked but their addresses are not known right away so they can't be configured in static config. Templates make it possible to define templates for such contracts that can be dynamically turned into sources whenever writer becomes aware of new contract's existence - for example when new contract has been deployed through factory.

### Defining templates

Templates are defined in [config](../core-concepts/checkpoint-configuration.md) and each template has a name that is later used when executing it.

Sample config might look like this:

```typescript
const config = {
  network_node_url: 'RPC_NODE',
  sources: [
    {
      contract: '0x2121c175922cef8870b6b6b956d01a7ac75af034dfbb9135698f88644b17ea3',
      start: 345862,
      events: [
        {
          name: 'space_deployed',
          fn: 'handleSpaceCreated'
        }
      ]
    }
  ],
  templates: {
    Space: {
      events: [
        {
          name: 'proposal_created',
          fn: 'handlePropose'
        },
        {
          name: 'vote_created',
          fn: 'handleVote'
        }
      ]
    }
  }
};
```

### Executing templates

After template is defined it can be executed in any writer to start tracking specific contract address for events defined in that template:

<pre class="language-typescript"><code class="lang-typescript"><strong>const handleSpaceCreated: starknet.Writer = async ({ block, helpers }) => {
</strong><strong>  await helpers.executeTemplate('Space', {
</strong>    contract: 'some_new_contract_address',
    start: block.block_number
  });
}
</code></pre>

Once template has been executed all events defined in that template will be tracked for given contract address.
