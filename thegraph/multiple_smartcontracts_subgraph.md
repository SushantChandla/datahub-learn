# Introduction

Most dapps span across multiple smart contracts with many of them being created dynamically. This tutorial demonstrates how to build a subgraph for a complex multi smart contract dapp with contracts deployed at runtime.

It uses the Uniswap AMM market contracts as an example as the reader might be familiar with it. We will start with the factory contract which deploys Uniswap pairs and then use events to be able to add the newly created pairs to our subgraph.

![Subgraph Studio](https://github.com/figment-networks/datahub-learn/raw/master/assets/graph.png)


# Prerequisites

To successfully complete this tutorial, you will need to have a basic understanding of Ethereum, Solidity, the Graph and Uniswap ecosystem.

# Requirements

As the previous tutorial, to complete this tutorial, you will need the Graph CLI installed: https://www.npmjs.com/package/@graphprotocol/graph-cli

# Creating a Graph Project

After you have graph-cli installed, you can create a new subgraph using the `graph init command`

```text
graph init uni-subgraph
```

For newer Macbooks you might need `yarn exec graph init uni-subgraph` depending upon your system permissions.

![Init](https://github.com/figment-networks/datahub-learn/raw/master/assets/graphinit.png)

For the network, select mainnet. The contract address for Uniswap Factory Contract is "0x5C69bEe701ef814a2B6a3EDD4B1652CB9cc5aA6f" and it remains the same across all test networks as well.

Graph will fetch the ABI of this contract for you and also initialize some of the files.

# Uniswap Pair Creation and Volume Tracking

In this tutorial, we are going to keep track of the reserves of tokens in Uniswap pairs.
As you know, pairs in Uniswap are created dynamically by the factory contract. The factory contract deploys the pair contract and then users can add liquidity to these contracts. Since the pair contract address is not known in advance, we will be dynamically taking the address of the pairs from the events emitted by the Factory contract. Whenever a new pair is created PairCreated event is emitted. We will be listening to this event and add the new pair contracts as data sources when they are created.

![Uniswap Diagram](https://github.com/figment-networks/datahub-learn/raw/master/assets/graphuniswap.png)

# Downloading the ABIs

For this tutorial you will also need the ABI (Application Binary Interface) of the pair contract, as we will be listening to events in the pair contracts as well as the factory contract. The official ABIs are aviailable [here](https://github.com/Uniswap/v2-subgraph/blob/master/abis/pair.json).

# Defining the Data Schema

We must define the schema of the data that we want to index. This is done in the `schema.graphql` file

```graphql
type Pair @entity {
  id: ID!
  count: BigInt!
  token0: Bytes! # address
  token1: Bytes! # address
  reserve0: BigInt!
  reserve1: BigInt!
}
```

The pair entity represents one token pair and maps directly to a single smart contract on ethereum.
`ID` - The address of the pair contract
`count` - The number of times this pair had liquidity added to it (uses the PairCreated event)
`token0` - The address of first token
`token1` - The address of second token
`reserve0` - The total reserves of first token in the pair contract
`reserve1` - The total reserves of second token in the pair contract


# Creating the Subgraph Config File

We will start with the `subgraph.yaml` file. The Graph CLI will automatically initialize some settings for you in this file. This information will be taken from the ABI that it will download from Etherscan. We will add the factory contract as a datasource. This is how it will look:

```yaml
specVersion: 0.0.2
schema:
  file: ./schema.graphql
dataSources:
  - kind: ethereum/contract
    name: UniswapFactory
    network: mainnet
    source:
      address: "0x5C69bEe701ef814a2B6a3EDD4B1652CB9cc5aA6f"
      abi: UniswapFactory
      startBlock: 13081816
    mapping:
      kind: ethereum/events
      apiVersion: 0.0.5
      language: wasm/assemblyscript
      entities:
        - PairCreated
      abis:
        - name: UniswapFactory
          file: ./abis/UniswapFactory.json
      eventHandlers:
        - event: PairCreated(indexed address,indexed address,address,uint256)
          handler: handlePairCreated
      file: ./src/mapping.ts
```

For this tutorial we will set the start block to a recent value. This is so that we can quickly test our graph in the graph studio. It can take many hours to index all the pairs in the entire ethereum chain.

For this tutorial we will set the start block to a recent value. This is so that we can quickly test our graph in the graph studio. Without specifying a recent block height to start from, it can take a very long time to index all of the token pairs in the entire Ethereum chain.

Put the address of the smart contract, the startBlock and the ABI in the source field.

In the mapping section, we will be listening to the PairCreated event, so we need to mention it in the entities section as well as the ABIs we will be using.

In the eventHandlers, add the event as in the ABI and then the handler. This is the AssemblyScript function which will create new pairs and add them as data source.

Since we will only know the address of the pair when the PairCreated event is emitted, we will have to create a datasource Template. This template is like a skeleton for each pair and it will be initialized dynamically using AssemblyScript.

```yaml
templates:
  - kind: ethereum/contract
    name: Pair
    network: mainnet
    source:
      abi: Pair
    mapping:
      kind: ethereum/events
      apiVersion: 0.0.5
      language: wasm/assemblyscript
      file: ./src/mapping.ts
      entities:
        - Pair
      abis:
        - name: Pair
          file: ./abis/Pair.json
        - name: UniswapFactory
          file: ./abis/UniswapFactory.json
      eventHandlers:
        - event: Sync(uint112,uint112)
          handler: handleSync
```

Notice that the format for template is exactly the same as a normal datasource but it does not have the source contract and address which binds it to a specific instance.

For the purposes of this tutorial, we will be handling the `Sync` event which provides the reserves of both tokens in the pair contract. It is fired every time a swap or other operation happens in the pair contract.

# Generating Types

Before writing the assemblyscript mapping file, we will need to generate the types for the contracts and data sources. You can do this using:-

```text
yarn codegen
```

```text
$ graph codegen
  Skip migration: Bump mapping apiVersion from 0.0.1 to 0.0.2
  Skip migration: Bump mapping apiVersion from 0.0.2 to 0.0.3
  Skip migration: Bump mapping apiVersion from 0.0.3 to 0.0.4
  Skip migration: Bump mapping apiVersion from 0.0.4 to 0.0.5
  Skip migration: Bump mapping specVersion from 0.0.1 to 0.0.2
✔ Apply migrations
✔ Load subgraph from subgraph.yaml
  Load contract ABI from abis/UniswapFactory.json
✔ Load contract ABIs
  Generate types for contract ABI: UniswapFactory (abis/UniswapFactory.json)
  Write types to generated/UniswapFactory/UniswapFactory.ts
✔ Generate types for contract ABIs
  Generate types for data source template Pair
  Write types for templates to generated/templates.ts
✔ Generate types for data source templates
  Load data source template ABI from abis/Pair.json
  Load data source template ABI from abis/UniswapFactory.json
✔ Load data source template ABIs
  Generate types for data source template ABI: Pair > Pair (abis/Pair.json)
  Write types to generated/templates/Pair/Pair.ts
  Generate types for data source template ABI: Pair > UniswapFactory (abis/UniswapFactory.json)
  Write types to generated/templates/Pair/UniswapFactory.ts
✔ Generate types for data source template ABIs
✔ Load GraphQL schema from schema.graphql
  Write types to generated/schema.ts
✔ Generate types for GraphQL schema

Types generated successfully

✨  Done in 2.09s.
```

This should create a generated folder with all the generated types.


# Write AssemblyScript Mapping

In the mapping file, we need to write two functions. One for handling new Pair and one for handling the `Sync` event of each pair. Let's start with the `handleNewPair` Event. Let's import the types from the generated folder as well the `BigInt` type:

```typescript
import { BigInt } from "@graphprotocol/graph-ts"
import { Pair } from "../generated/schema"
import {
  PairCreated
} from "../generated/UniswapFactory/UniswapFactory"
import {
  Pair as PairTemplate
} from "../generated/templates"
```

Notice that the datasource template is also called Pair, the same as our graphql schema type. So we import it as PairTemplate.

```typescript
export function handlePairCreated(event: PairCreated): void {
  // Entities can be loaded from the store using a string ID; this ID
  // needs to be unique across all entities of the same type
  let pair = Pair.load(event.transaction.from.toHex())

  // Entities only exist after they have been saved to the store;
  // `null` checks allow to create entities on demand
  if (!pair) {
    pair = new Pair(event.transaction.from.toHex())

    // Entity fields can be set using simple assignments
    pair.count = BigInt.fromI32(0)

    //set reserves to 0
    pair.reserve0 = BigInt.fromI32(0)
    pair.reserve1 = BigInt.fromI32(0)
  }

  pair.count = pair.count + BigInt.fromI32(1)

  //  fields can be set based on event parameters
  pair.token0 = event.params.token0
  pair.token1 = event.params.token1

  PairTemplate.create(event.params.pair)
  // Entities can be written to the store with `.save()`
  pair.save()

}
```

Let's look at each line of our function. The first line tries to load the Pair from the subgraph. If the Pair is null, it will then create a new Pair with the address taken from the event. Since our datatype is an `ID`, we need to use the `toHex()` function to convert it into an `ID` type. The `BigInt.fromI32(0)` initializes `count` to 0.

Next we simply store the address of the tokens taken from the event params into our token values.

For now we simply set the reserves to 0, as it is not possible to get the reserves from the PairCreated Event.

The line `PairTemplate.create(event.params.pair)` is crucial. Without this, the subgraph will not know that it has to start listening to events from this smart contract. It adds the Pair at `event.params.pair` as a datasource.

For the `handleSync` Event, we first import the required types.

```typescript
import {
  Sync
} from "../generated/templates/Pair/Pair"
```

Then in the `handleSync` function, we simply update the reserves from the Sync event. If you notice in our ABI the event has uint112 as the type of the reserve, so we have to typecast it to BigInt.

```typescript
export function handleSync(event: Sync): void {
    
    let pair = Pair.load(event.address.toHex())
     
    if(pair) {
      pair.reserve0 = BigInt.fromI32(event.params.reserve0.toI32())
      pair.reserve1 = BigInt.fromI32(event.params.reserve1.toI32())
      pair.save()
    }
}

```

# Deploying the Subgraph

Now we are ready to deploy our subgraph. To deploy the subgraph to the graph, we will first deploy it to graph studio and test it.

For this we will need to go to the studio https://thegraph.com/studio/ and get the deploy key.

From the subgraph studio, copy the deploy key from the dashboard.

In the cli, now you can authenticate using the following command.

```text
graph auth --product subgraph-studio uni-subgraph
```

After authentication is done, you can deploy your graph to subgraph studio.

```text
yarn deploy
```

You can now enter any version you like in the `vmajor.minor.patch` format. You should see the subgraph deployment success on your terminal.

```text
yarn run v1.22.10
$ graph deploy --node https://api.studio.thegraph.com/deploy/ uni-subgraph
✔ Version Label (e.g. v0.0.1) · v0.1.0
  Skip migration: Bump mapping apiVersion from 0.0.1 to 0.0.2
  Skip migration: Bump mapping apiVersion from 0.0.2 to 0.0.3
  Skip migration: Bump mapping apiVersion from 0.0.3 to 0.0.4
  Skip migration: Bump mapping apiVersion from 0.0.4 to 0.0.5
  Skip migration: Bump mapping specVersion from 0.0.1 to 0.0.2
✔ Apply migrations
✔ Load subgraph from subgraph.yaml
  Compile data source: UniswapFactory => build/UniswapFactory/UniswapFactory.wasm
  Compile data source template: Pair => build/UniswapFactory/UniswapFactory.wasm (already compiled)
✔ Compile subgraph
  Copy schema file build/schema.graphql
  Write subgraph file build/UniswapFactory/abis/UniswapFactory.json
  Write subgraph file build/Pair/abis/Pair.json
  Write subgraph file build/Pair/abis/UniswapFactory.json
  Write subgraph manifest build/subgraph.yaml
✔ Write compiled subgraph to build/
  Add file to IPFS build/schema.graphql
                .. QmNYFqNjWcJBskJXf5uaKjfWLyWv6dHoGign8zY38Hxp4p
  Add file to IPFS build/UniswapFactory/abis/UniswapFactory.json
                .. QmZ55G1yYFzde8Vcq4cpLfNgPSEibpLi9aYCqS1jEvCKQ9
  Add file to IPFS build/UniswapFactory/UniswapFactory.wasm
                .. QmRQ5iRV3JqkY6W9L16pANppNbhEa19KK2p5KC7LHnPjD7
  Add file to IPFS build/Pair/abis/Pair.json
                .. QmbPLMADBP8L6LBVP3ZBQ8RgG7ghamD8DvbdUxHAjZrLgm
  Add file to IPFS build/Pair/abis/UniswapFactory.json
                .. QmZ55G1yYFzde8Vcq4cpLfNgPSEibpLi9aYCqS1jEvCKQ9 (already uploaded)
  Add file to IPFS build/UniswapFactory/UniswapFactory.wasm
                .. QmRQ5iRV3JqkY6W9L16pANppNbhEa19KK2p5KC7LHnPjD7 (already uploaded)
✔ Upload subgraph to IPFS

Build completed: QmZHBED1WzxvEeCNySwJ8FoQVwGsb3CABByLNbRPyiPGMo

Deployed to https://thegraph.com/studio/subgraph/uni-subgraph

Subgraph endpoints:
Queries (HTTP):     https://api.studio.thegraph.com/query/4570/uni-subgraph/v0.1.0
Subscriptions (WS): https://api.studio.thegraph.com/query/4570/uni-subgraph/v0.1.0
```

You can now go to https://thegraph.com/studio/subgraph/uni-subgraph/ to play with your subgraph. You should see a `syncing` status and you can start making queries once the progress reaches 100. 

![Playground](https://github.com/figment-networks/datahub-learn/raw/master/assets/graphplayground.png)


# Conclusion

Congratulations on finishing this tutorial! You learned how to create a subgraph for a dapp with multiple smartcontracts and how to add smartcontracts to your subgraph that are dynamically created.

# About the Author

I'm Mohit Jandwani, a crypto investor and software engineer. I teach about how to generate passive income from crypto on my [Youtube Channel](https://www.youtube.com/c/DefiMasterClass). Feel free to connect with me on [Twitter](https://twitter.com/mohitjandwani).

