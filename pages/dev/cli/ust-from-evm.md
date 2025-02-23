# Redeem UST from an EVM chain

import Callout from 'nextra-theme-docs/callout'
import Markdown from 'markdown-to-jsx'
import Tabs from '../../../components/tabs'
import CodeBlock from '../../../components/code-block'

Redeem UST tokens from an EVM chain to Terra using the terminal.

<Callout type="error" emoji="🔥">
  Important: The Axelar network is under active development. Use at your own risk with funds you're comfortable using. See [Terms of use](/terms-of-use).
</Callout>

## Prerequisites

- Skill level: intermediate
- Prerequisites for [Send UST to an EVM chain](./ust-to-evm)
- `{TERRA_DEST_ADDR}` is an address controlled by you on the Terra chain. This is where your UST tokens will be redeemed.

## Redeem UST tokens from an EVM chain

Link your `{TERRA_DEST_ADDR}` address to a new temporary deposit address on the EVM chain:

<Tabs tabs={[
  {
    title: "Mainnet",
    content: <CodeBlock language="bash">
      {"echo my-secret-password | ~/.axelar/bin/axelard tx evm link {EVM_CHAIN} terra {TERRA_DEST_ADDR} uusd --from validator --gas auto --gas-adjustment 1.5 --chain-id axelar-dojo-1 --home ~/.axelar/.core"}
    </CodeBlock>
  },
  {
    title: "Testnet",
    content: <CodeBlock language="bash">
      {"echo my-secret-password | ~/.axelar_testnet/bin/axelard tx evm link {EVM_CHAIN} terra {TERRA_DEST_ADDR} uusd --from validator --gas auto --gas-adjustment 1.5 --chain-id axelar-testnet-lisbon-3 --home ~/.axelar_testnet/.core"}
    </CodeBlock>
  }
]} />

Output should contain

```
successfully linked {EVM_TEMP_ADDR} and {TERRA_DEST_ADDR}
```

Use Metamask to send some wrapped AXL tokens on `{EVM_CHAIN}` to the new temporary deposit address `{EVM_TEMP_ADDR}`. Save the transaction hash `{EVM_TX_HASH}` for later.

<Callout type="error" emoji="🔥">
  Danger: Send only `Axelar Wrapped UST` ERC20 tokens to `{EVM_TEMP_ADDR}`. Any other token sent to `{EVM_TEMP_ADDR}` will be lost.
</Callout>

<Callout emoji="📝">
  Note: Third-party monitoring tools will automatically complete the remaining steps of this process.

  Wait a few minutes then check your Terra `{TERRA_DEST_ADDR}` account UST token balance.
</Callout>

<Callout type="warning" emoji="⚠️">
  Caution: If you attempt the remaining steps while third-party monitoring tools are active then your commands are likely to conflict with third-party commands. In this case you are likely to observe errors. Deeper investigation might be needed to resolve conflicts and complete the transfer.

  The remaining steps are needed only if there are no active third-party monitoring tools and you wish to complete the process manually.
</Callout>

Do not proceed to the next step until you have waited for sufficiently many block confirmations on the EVM chain. Block confirmation minimums can be found at [Testnet resources](/resources/testnet), [Mainnet resources](/resources/mainnet).

Confirm the EVM chain transaction on Axelar.

<Tabs tabs={[
  {
    title: "Mainnet",
    content: <CodeBlock language="bash">
      {"echo my-secret-password | ~/.axelar/bin/axelard tx evm confirm-erc20-deposit {EVM_CHAIN} {EVM_TX_HASH} {AMOUNT} {EVM_TEMP_ADDR} --from validator --gas auto --gas-adjustment 1.5 --chain-id axelar-dojo-1 --home ~/.axelar/.core"}
    </CodeBlock>
  },
  {
    title: "Testnet",
    content: <CodeBlock language="bash">
      {"echo my-secret-password | ~/.axelar_testnet/bin/axelard tx evm confirm-erc20-deposit {EVM_CHAIN} {EVM_TX_HASH} {AMOUNT} {EVM_TEMP_ADDR} --from validator --gas auto --gas-adjustment 1.5 --chain-id axelar-testnet-lisbon-3 --home ~/.axelar_testnet/.core"}
    </CodeBlock>
  }
]} />

Wait for confirmation on Axelar.

Create and sign pending transfers for `{EVM_CHAIN}`.

<Tabs tabs={[
  {
    title: "Mainnet",
    content: <CodeBlock language="bash">
{`echo my-secret-password | ~/.axelar/bin/axelard tx evm create-burn-tokens {EVM_CHAIN} --from validator --chain-id axelar-dojo-1 --home ~/.axelar/.core --gas auto --gas-adjustment 1.5
echo my-secret-password | ~/.axelar/bin/axelard tx evm sign-commands {EVM_CHAIN} --from validator --gas auto --gas-adjustment 1.2 --chain-id axelar-dojo-1 --home ~/.axelar/.core`}
    </CodeBlock>
  },
  {
    title: "Testnet",
    content: <CodeBlock language="bash">
{`echo my-secret-password | ~/.axelar_testnet/bin/axelard tx evm create-burn-tokens {EVM_CHAIN} --from validator --chain-id axelar-testnet-lisbon-3 --home ~/.axelar_testnet/.core --gas auto --gas-adjustment 1.5
echo my-secret-password | ~/.axelar_testnet/bin/axelard tx evm sign-commands {EVM_CHAIN} --from validator --gas auto --gas-adjustment 1.2 --chain-id axelar-testnet-lisbon-3 --home ~/.axelar_testnet/.core`}
    </CodeBlock>
  }
]} />

Output should contain

```
successfully started signing batched commands with ID {BATCH_ID}
```

Get the `execute_data`:

<Tabs tabs={[
  {
    title: "Mainnet",
    content: <CodeBlock language="bash">
      {"~/.axelar/bin/axelard q evm batched-commands {EVM_CHAIN} {BATCH_ID}"}
    </CodeBlock>
  },
  {
    title: "Testnet",
    content: <CodeBlock language="bash">
      {"~/.axelar_testnet/bin/axelard q evm batched-commands {EVM_CHAIN} {BATCH_ID}"}
    </CodeBlock>
  }
]} />

Wait for `status: BATCHED_COMMANDS_STATUS_SIGNED` and copy the `execute_data`.

Use Metamask to send a transaction on `{EVM_CHAIN}` with the `execute_data` to the Axelar gateway contract address `{GATEWAY_ADDR}`.

<Callout type="error" emoji="🔥">
  Danger: Post your transaction to the correct chain! Set your Metamask network to `{EVM_CHAIN}`.
</Callout>

<Callout type="warning" emoji="⚠️">
  Caution: Manually increase the gas limit to 5 million gas (5000000). If you don't do this then the transaction will fail due to insufficient gas and you will not receive your tokens.

  Before you click "confirm": select "EDIT", change "Gas Limit" to 5000000, and "Save"
</Callout>

<Callout emoji="💡">
  Tip: Learn the Axelar `{GATEWAY_ADDR}` for `{EVM_CHAIN}` in two ways:

  ### 1. Documentation

  [Testnet resources](/resources/testnet), [Mainnet resources](/resources/mainnet).

  ### 2. Terminal

  <Tabs tabs={[
    {
      title: "Mainnet",
      content: <CodeBlock language="bash">
        {"~/.axelar/bin/axelard q evm gateway-address {EVM_CHAIN}"}
      </CodeBlock>
    },
    {
      title: "Testnet",
      content: <CodeBlock language="bash">
        {"~/.axelar_testnet/bin/axelard q evm gateway-address {EVM_CHAIN}"}
      </CodeBlock>
    }
  ]} />
</Callout>

To send a transaction to `{GATEWAY_ADDR}` using Metamask: paste hex from `execute_data` above into "Hex Data" field. (Do not send tokens!)

Execute the pending IBC transfer:

<Tabs tabs={[
  {
    title: "Mainnet",
    content: <CodeBlock language="bash">
      {"echo my-secret-password | ~/.axelar/bin/axelard tx axelarnet route-ibc-transfers --from validator --gas auto --gas-adjustment 1.5 --chain-id axelar-dojo-1 --home ~/.axelar/.core"}
    </CodeBlock>
  },
  {
    title: "Testnet",
    content: <CodeBlock language="bash">
      {"echo my-secret-password | ~/.axelar_testnet/bin/axelard tx axelarnet route-ibc-transfers --from validator --gas auto --gas-adjustment 1.5 --chain-id axelar-testnet-lisbon-3 --home ~/.axelar_testnet/.core"}
    </CodeBlock>
  }
]} />

Wait a few minutes for the IBC relayer to relay your transaction to Terra.

You should see the redeemed `{AMOUNT}` of UST token (minus transaction fees) in your Terra `{TERRA_DEST_ADDR}` account.

Congratulations! You have redeemed UST tokens from the external EVM chain back to Terra!
