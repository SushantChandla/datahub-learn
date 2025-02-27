# Introduction 

Often times you will need to pull information about the blockchain and its state such as: 

* Most recent synced block 
* Node information 
* Validator set status 
* Token prices

Let’s see how this can be done by using DataHub to connect to a Terra node.

In this tutorial, we will explore different queries that we can run again our [**DataHub** ](https://figment.io/datahub-waitlist/)Terra node. More specifically, we will take a look at retrieving data about:

* Blockchain details
* Account details
* Luna exchange rates
* Governance proposals

## **Prerequisites**

1. Please make sure that you completed tutorials \#1 \("Connecting to a Terra node with DataHub"\) and \#2 \("Creating your first Terra account with the Terra SDK"\) before moving forward. We will be building on top of the Nodejs application created in these tutorials. 
2. Make sure that you connect to an archive Tequila node available on DataHub.

![](https://github.com/figment-networks/datahub-learn/raw/master/assets/download.png)

{% hint style="info" %}
Archive nodes allow you to query historical data, no matter how old it is.
{% endhint %}

## **Query blockchain details**

In your root directory, create a new file `query.js` and place in it below the following code snippet:

```javascript
const { LCDClient, MnemonicKey } = require('@terra-money/terra.js');
require('dotenv').config();

const main = async () => {
  const terra = new LCDClient({
    URL: process.env.TERRA_NODE_URL,
    chainID: process.env.TERRA_CHAIN_ID,
  });

  // Use key created in tutorial #2
  const mk = new MnemonicKey({
    mnemonic: process.env.MNEMONIC,
  });

  // 1. Query state of chain

  // 2. Query account information

  // 3. Query exchange rates

}

main().then(resp => {
  console.log(resp);
}).catch(err => {
  console.log(err);
})
```

This might look familiar to you. Starting from the top you can see that we import Terra.js, dotenv, and then create a new terra instance using `LCDClient`. This establishes the connection to our Terra node. Then we use `MnemonicKey`. You’ve seen this class before in tutorial \#2 where we created your account. 

However, there is a subtle but very important difference in how we instantiate this class. When you want to create a random account and mnemonic, you instantiate `MnemonicKey` without any arguments. On the other hand, if you already have a mnemonic \(and we do have it in our `.env` file\) we can create a `MnemonicKey` instance from that mnemonic which will load our existing account instead of creating a new one.

Now let’s stop for a second. Do you remember we explained in tutorial \#2 that you should always keep your mnemonic secure? I hope that now you can see why this is important. The mnemonic that we stored in `.env` file allows you to recover your account using `MnemonicKey` constructor. **You should keep it private and secure at all times!**

Ok, now that we have this out of our way, let’s move on to writing our first query methods!

### Chain state 

Let's start by getting more information about the Terra blockchain.

In the `query.js` file under the comment `// 1. Query state of chain` add the following code snippet below:

```javascript
const blockInfo = await terra.tendermint.blockInfo();
console.log('blockInfo: ', blockInfo);
```

This should return a response similar to:

```javascript
{
  "block_id": {
    "hash": "43D4D65924D7B10C947C385271B69B31176F9CA58BAD88E617843B1D9273D5EB",
    "parts": {
      "total": "1",
      "hash": "C11DCDC431D7B963CD435234EDD60A42084B5D9B38058854E49B6BEEBE4B17C6"
    }
  },
  "block": {
    "header": {
      "version": {
        "block": "10",
        "app": "0"
      },
      "chain_id": "tequila-0004",
      "height": "728883",
      "time": "2020-10-20T08:58:12.692826541Z",
      "last_block_id": {
        "hash": "3B7329611690F222AC7007B33ACF267B8F806ED1FDBD7D238B463398B0405949",
        "parts": {
          "total": "1",
          "hash": "32087D47AA51B7E5A142316BB63756728EB47E9A869DEA58AF37F9C85CF05796"
        }
      },
      "last_commit_hash": "46A1AF4834ECC50561414FF3ACA103815E0700E07476CC3DB5919EB4FB647208",
      "data_hash": "C42B204B512F492EF58D236D515A3CF289D31F2E0F818B28F1CD88198CF8845F",
      "validators_hash": "78FC097F4304B0D2C745B14E9FA547722881A97807BA16E03BFB0149B0758453",
      "next_validators_hash": "78FC097F4304B0D2C745B14E9FA547722881A97807BA16E03BFB0149B0758453",
      "consensus_hash": "048091BC7DDC283F77BFBF91D73C44DA58C3DF8A9CBC867405D8B7F3DAADA22F",
      "app_hash": "D1D105BB03966FF4EE800F32D2C30BCBCDE3A6066D66CD2E0CEE820AAB1D0FE3",
      "last_results_hash": "FE43D66AFA4A9A5C4F9C9DA89F4FFB52635C8F342E7FFB731D68E36C5982072A",
      "evidence_hash": "",
      "proposer_address": "090B0F80019F8BBC71FCE4C7D40C8353FB9C40E0"
    },
    "data": {
      "txs": [
        "jgbGwQI/ClLeU+EsChQtMTAwMDAwMDAwMDAwMDAwMDAwMBIEZDBlZRoEdXVzZCIUnhQdxPKyQIYbkzgMbtAR8eo0cmkqFJ4UHcTyskCGG5M4DG7QEfHqNHJpClLeU+EsChQtMTAwMDAwMDAwMDAwMDAwMDAwMBIEZjBhYxoEdWtydyIUnhQdxPKyQIYbkzgMbtAR8eo0cmkqFJ4UHcTyskCGG5M4DG7QEfHqNHJpClLeU+EsChQtMTAwMDAwMDAwMDAwMDAwMDAwMBIEY2U4YRoEdXNkciIUnhQdxPKyQIYbkzgMbtAR8eo0cmkqFJ4UHcTyskCGG5M4DG7QEfHqNHJpClLeU+EsChQtMTAwMDAwMDAwMDAwMDAwMDAwMBIEY2IyZhoEdW1udCIUnhQdxPKyQIYbkzgMbtAR8eo0cmkqFJ4UHcTyskCGG5M4DG7QEfHqNHJpCkxxWMHeChQpqZJNSwemnef35gQ6c/W750NgJBIEdXVzZBoUnhQdxPKyQIYbkzgMbtAR8eo0cmkiFJ4UHcTyskCGG5M4DG7QEfHqNHJpCkxxWMHeChTHW8UOf5FFBuw5UOPE9ygt1RRzTxIEdWtydxoUnhQdxPKyQIYbkzgMbtAR8eo0cmkiFJ4UHcTyskCGG5M4DG7QEfHqNHJpCkxxWMHeChT/g+qVp3CUH8tsMDhMKvmg/kW3lxIEdXNkchoUnhQdxPKyQIYbkzgMbtAR8eo0cmkiFJ4UHcTyskCGG5M4DG7QEfHqNHJpCkxxWMHeChSecF0kR0hoHOT/Pag2rrx484fyNxIEdW1udBoUnhQdxPKyQIYbkzgMbtAR8eo0cmkiFJ4UHcTyskCGG5M4DG7QEfHqNHJpEhQKDgoEdWtydxIGMzYwMDAwEMCaDBpqCibrWumHIQJG64Ud+sxi4ABTpaSKSjzvoCCc74+jwmM5xDJXTbjT0xJAhbwlBvcF9ehO0hUXsiKleeD88XPo5ckaAXdZ+Zqc2V8aRniDWqKyYzXmLCKdpNUmtICAQJI07H4rwuqT5EdmsA=="
      ]
    },
    "evidence": {
      "evidence": null
    },
    "last_commit": {
      "height": "728882",
      "round": "0",
      "block_id": {
        "hash": "3B7329611690F222AC7007B33ACF267B8F806ED1FDBD7D238B463398B0405949",
        "parts": {
          "total": "1",
          "hash": "32087D47AA51B7E5A142316BB63756728EB47E9A869DEA58AF37F9C85CF05796"
        }
      },
      "signatures": [
        {
          "block_id_flag": 2,
          "validator_address": "090B0F80019F8BBC71FCE4C7D40C8353FB9C40E0",
          "timestamp": "2020-10-20T08:58:12.778075272Z",
          "signature": "R5pEt8mQXy8Y1BXB3oxoma495+/R46UvVdeI/XkAOczSZJU+mMbfgoeN5lbvEaMEpZdqcfExdqLse28LFmCECA=="
        },
        {
          "block_id_flag": 2,
          "validator_address": "118BC1F7C001FEA994E356A414CB6751E3F21D82",
          "timestamp": "2020-10-20T08:58:12.692826541Z",
          "signature": "75w0zu48MBNyq/2pAWG5OirIJrcHswEr3SPV/SjGolGqHusjCDpEBufMj+HosFUIwmxmcjCFSy4c2vZpXnmMDw=="
        },
        {
          "block_id_flag": 2,
          "validator_address": "42823B42D99BF70FCAF54CFA419D03789368EE88",
          "timestamp": "2020-10-20T08:58:12.693091303Z",
          "signature": "sRrG8eQruD5jEhVwBzZJuVzkNDZDWNNqeFYaNADopy4+4OmHkmZs3mB7v5gBYsS+65rpryLENdTXayMi0HDvAQ=="
        },
        {
          "block_id_flag": 2,
          "validator_address": "47124BD026F7344E94E9A5C4700E24E3911469B1",
          "timestamp": "2020-10-20T08:58:12.849743683Z",
          "signature": "LtqHapzq8xFAR3/gJN7YAf4YQWChvbDeizbvQ6bph3LkVU2lGde+FlCDlms9erBObVG6WrkFTKxgvuPuz2VgBQ=="
        },
        {
          "block_id_flag": 2,
          "validator_address": "4AFD390FDA4DF4DD1DDA38470EC034989AC4276E",
          "timestamp": "2020-10-20T08:58:12.633376636Z",
          "signature": "+Aj6byteePsbl5RscHf/kT163vQhWec+EP8mfajyZNWhHLcf4Zp1LXJxbF6lGG+j3+DI/rq8FCaa79x/BYKABA=="
        },
        {
          "block_id_flag": 2,
          "validator_address": "5240489B9CA45F90E5DCC4F5F4398C6730DC7823",
          "timestamp": "2020-10-20T08:58:12.790334776Z",
          "signature": "fyZ+/ktI+sEpGP74NZNs49kTYf4f9d+Yft9a2nP6XnIDxcPp7qOBg7v2TgCHS6oqEgZMVV2reN4uFJr6BX8iAw=="
        },
        {
          "block_id_flag": 2,
          "validator_address": "5FFD09EC6F8511A997EA5530DB401D9B5E34FEA5",
          "timestamp": "2020-10-20T08:58:12.971736777Z",
          "signature": "Lm4+XUDgZ3HDSfpA3TiMSaUC6Lk8gYRW29xo7osn9ictqaXzrzi+lsA9ySZcjHEH0tueM2ll1vmcyDZgg1uiBg=="
        },
        {
          "block_id_flag": 2,
          "validator_address": "A38D8527D109261F1833FCEA07323B260F169A4B",
          "timestamp": "2020-10-20T08:58:12.925312277Z",
          "signature": "SsofYbJldqfQVVrvnHEWavorgVrnu1h3UpnktdbvSTGEd/XViwKQc6C1toiu09/Yd4djA1C7+DGU1T0DxICXBQ=="
        },
        {
          "block_id_flag": 2,
          "validator_address": "A4F1AE87B3C04C8A807B6D00F1D73D1CD727099C",
          "timestamp": "2020-10-20T08:58:12.973650896Z",
          "signature": "mvQZWT5asCanfmGYYq1gAsJDGttOvrkhwEDb/cnr/UY5IcOsQTtxXjsyzY7LFaGjDrWnHuwkbG4nQVG3SeRJBw=="
        },
        {
          "block_id_flag": 2,
          "validator_address": "B0DF524982C7F03C451135D0D6D53C9A8ECB0696",
          "timestamp": "2020-10-20T08:58:12.792765723Z",
          "signature": "qkazmo9qLlizWGstlLoxDhrSraA1lCzucodxOM9pR3ookGYYHQX2z0GABlIInLPiWdNFxwK8MacjxC9PenqBBw=="
        }
      ]
    }
  }
}
```

In this response, you can see that the last synced block was at height `728883` with the timestamp of `2020-10-20T08:58:12.692826541Z`. Other interesting things that can be found in the response are: `txs` \(transactions in the block\) and `signatures` \(signatures of validators who validated the block\).

### Block information 

Great! Now it’s time to get some more information about the node we are connecting to. Add this snippet below the `blockInfo` call:

```javascript
const nodeInfo = await terra.tendermint.nodeInfo();
console.log('nodeInfo: ', nodeInfo);
```

You should receive a response similar to:

```javascript
{
  "node_info": {
    "protocol_version": {
      "p2p": "7",
      "block": "10",
      "app": "0"
    },
    "id": "a19b1664f34056a2c8370742468fd12115e3f95e",
    "listen_addr": "tcp://0.0.0.0:26656",
    "network": "tequila-0004",
    "version": "0.33.7",
    "channels": "4020212223303800",
    "moniker": "XRA4GiZecK",
    "other": {
      "tx_index": "on",
      "rpc_address": "tcp://0.0.0.0:26657"
    }
  },
  "application_version": {
    "name": "terra",
    "server_name": "terrad",
    "client_name": "terracli",
    "version": "0.4.0",
    "commit": "0ccb97e4427007f940b611fb0b5fb67d6a7d69f5",
    "build_tags": "netgo ledger muslc,",
    "go": "go version go1.14.7 linux/amd64"
  }
}
```

In the response, you can find information about the network name, its version, protocol version, and application version.

{% hint style="info" %}
This is very useful information to make sure that the SDK you use is compatible with the node you are connecting to. 
{% endhint %}

### Validator set

Good job! Now that you have more information about the blockchain and the node, we can check what validators are in the validator set for the current height. In order to do this, add this snippet of code and run the `query.js` script:

```javascript
const validatorSet = await terra.tendermint.validatorSet();
console.log('validatorSet: ', validatorSet);
```

You should see an output similar to:

```javascript
{
  "block_height": "728915",
  "validators": [
    {
      "address": "terravalcons1py9slqqpn79mcu0uunragryr20aecs8q2va0ke",
      "pub_key": "terravalconspub1zcjduepqwgwyky5375uk0llhwf0ya5lmwy4up838jevfh3pyzf5s3hd96xjslnexul",
      "proposer_priority": "35872",
      "voting_power": "220343"
    },
    {
      "address": "terravalcons1zx9ura7qq8l2n98r26jpfjm8283ly8vzr5300n",
      "pub_key": "terravalconspub1zcjduepq7cspfkyg8wrs4yyp6w75a70p40gs22g8fe22w4uyz2mly4k72yvs5y0tlp",
      "proposer_priority": "88745",
      "voting_power": "239910"
    },
    {
      "address": "terravalcons1g2prksken0msljh4fnayr8gr0zfk3m5g0gh25z",
      "pub_key": "terravalconspub1zcjduepq85lttdmt2ela3plau0qj4ytvtxndht7dg3l58ngkg0p93am7uyqsph96vw",
      "proposer_priority": "235796",
      "voting_power": "236449"
    },
    {
      "address": "terravalcons1gufyh5px7u6ya98f5hz8qr3yuwg3g6d34ar2ys",
      "pub_key": "terravalconspub1zcjduepqh7jx8jkach39qsw0kcp9jztzwwca28tvh4s9wccypvl79x80v96qpl9lst",
      "proposer_priority": "-459891",
      "voting_power": "100"
    },
    {
      "address": "terravalcons1ft7njr76fh6d68w68prsasp5nzdvgfmwhveejd",
      "pub_key": "terravalconspub1zcjduepqnzrrsmy6dq995vnehuu63286q9nltnw6sxzq93kl8a0etgxggnms6vmu33",
      "proposer_priority": "364282",
      "voting_power": "239713"
    },
    {
      "address": "terravalcons12fqy3xuu530epewucn6lgwvvvucdc7prynxa60",
      "pub_key": "terravalconspub1zcjduepqkwv5ltsk762q2a0eaqrj4plnvjx9hkesgr47rsqphc59c9srtjnqr4y89x",
      "proposer_priority": "232828",
      "voting_power": "10"
    },
    {
      "address": "terravalcons1tl7snmr0s5g6n9l225cdksqand0rfl497ejhj6",
      "pub_key": "terravalconspub1zcjduepqsac476x64pp5pvmsrx7huag0sm96sgxqarxndscuxnjwzzr3ha2qllrlza",
      "proposer_priority": "-349080",
      "voting_power": "1904"
    },
    {
      "address": "terravalcons15wxc2f73pynp7xpnln4qwv3myc83dxjtnkue6t",
      "pub_key": "terravalconspub1zcjduepq2c7yhl7ztphp63m4k259grf86h9u7cudkd5lvgcp799px9wzkupq55z4k9",
      "proposer_priority": "-43020",
      "voting_power": "12"
    },
    {
      "address": "terravalcons15nc6apancpxg4qrmd5q0r4earntjwzvupuhd9y",
      "pub_key": "terravalconspub1zcjduepquytugqld2y8zsvk20uud9gl0ug6lhcw5vzrqequ7pysj9hacpawqrr4rkf",
      "proposer_priority": "192729",
      "voting_power": "559"
    },
    {
      "address": "terravalcons1kr04yjvzclcrc3g3xhgdd4fun28vkp5kuu0ec5",
      "pub_key": "terravalconspub1zcjduepqvyc0lm4umk8ertkgjgaldwqktkx94mtx0dcddwc0upttplgmznrs7a7n5w",
      "proposer_priority": "-298253",
      "voting_power": "11008"
    }
  ]
}
```

In this output, you can see a list of validator who are in the validator set at the most recent block, which in this case is `728915` . For every validator, most notably, you get `address` and `voting_power`. The value for the address is somehow similar to your account’s address because it also starts with `terra`. However, it is a unique address that identifies that particular validator. The value for voting power identifies how much voting power a validator has when producing blocks or voting on governance proposals \(you will find out more about proposals later in this tutorial\).

Now that we had a chance to look at validators, it's time to get more details about our own account.

In `query.js`, add this snippet below the comment `// 2. Query account information` and run the following script:

```javascript
const accountInfo = await terra.auth.accountInfo(mk.accAddress);
console.log('accountInfo: ', accountInfo);
```

This call should return an output similar to:

```javascript
{
  "type": "core/Account",
  "value": {
    "account_number": "1456",
    "address": "terra1jmn8kywcyuvg92ljvv68vmddprz2tflg33pu7g",
    "coins": [
      {
        "amount": "999971891",
        "denom": "uluna"
      }
    ],
    "public_key": {
      "type": "tendermint/PubKeySecp256k1",
      "value": "A1qIZpJ9zYJy3opUA0yjhdj20o4hK9J3QiOkr+V2CUkm"
    },
    "sequence": "2"
  }
}
```

In this response, you can see your account address and how many LUNA tokens you own. Hopefully, you see `1,000` LUNAs that you received from the testnet faucet. If your account balance is 0, please go to tutorial \#2 under “Getting testnet tokens” to request some free tokens. You will need them in tutorial \#4.

### Exchange rates

Now that we know that we own some tokens, let’s explore a call to gives more information about exchange rates to different denominations. Add this snippet to your `query.js` under `// 3. Query exchange rates`

```javascript
const exchangeRates = await terra.oracle.exchangeRates();
console.log('exchangeRates: ', JSON.stringify(exchangeRates));
```

This should return:

```javascript
[
  {
    "amount": "357.437740549741630308",
    "denom": "ukrw"
  },
  {
    "amount": "889.657532099939221897",
    "denom": "umnt"
  },
  {
    "amount": "0.222326274621939294",
    "denom": "usdr"
  },
  {
    "amount": "0.313830336202673152",
    "denom": "uusd"
  }
]
```

In this response, you see exchange rates for 4 different denominations. Get yourself familiar with those exchange rates, because in the next tutorial we will be exchanging LUNAs to different denominations.

### Governance proposals

Governance is the process by which Terra network participants can effect change for the protocol, by collectively demonstrating consensus support for proposals. In order to participate, you need to know what those proposals are. Let’s examine how we can do just that using the Terra SDK.

In `query.js` add this snippet under `// 4. Query proposals`:

```javascript
const proposals = await terra.gov.proposals();
console.log('proposals: ', JSON.stringify(proposals));
```

This should yield a response similar to:

```javascript
[
  {
    "content": {
      "type": "gov/TextProposal",
      "value": {
        "description": "Drop the power of loki.",
        "title": "test"
      }
    },
    "deposit_end_time": "2020-09-08T10:40:04.126Z",
    "final_tally_result": {
      "abstain": "0",
      "no": "0",
      "no_with_veto": "0",
      "yes": "0"
    },
    "id": "1",
    "proposal_status": "Rejected",
    "submit_time": "2020-09-07T10:40:04.126Z",
    "total_deposit": [
      {
        "amount": "11000000",
        "denom": "uluna"
      }
    ],
    "voting_end_time": "2020-09-08T10:40:04.126Z",
    "voting_start_time": "2020-09-07T10:40:04.126Z"
  },
  {
    "content": {
      "type": "gov/TextProposal",
      "value": {
        "description": "flex",
        "title": "just flexing"
      }
    },
    "deposit_end_time": "2020-10-06T11:13:30.668Z",
    "final_tally_result": {
      "abstain": "0",
      "no": "0",
      "no_with_veto": "0",
      "yes": "1009999999"
    },
    "id": "2",
    "proposal_status": "Rejected",
    "submit_time": "2020-10-05T11:13:30.668Z",
    "total_deposit": [
      {
        "amount": "512000000",
        "denom": "uluna"
      }
    ],
    "voting_end_time": "2020-10-06T11:13:30.668Z",
    "voting_start_time": "2020-10-05T11:13:30.668Z"
  },
  {
    "content": {
      "type": "gov/TextProposal",
      "value": {
        "description": "Hello?",
        "title": "Test Hello"
      }
    },
    "deposit_end_time": "2020-10-07T05:04:43.988Z",
    "final_tally_result": {
      "abstain": "0",
      "no": "0",
      "no_with_veto": "0",
      "yes": "10000000"
    },
    "id": "3",
    "proposal_status": "Rejected",
    "submit_time": "2020-10-06T05:04:43.988Z",
    "total_deposit": [
      {
        "amount": "256000000",
        "denom": "uluna"
      }
    ],
    "voting_end_time": "2020-10-07T05:04:43.988Z",
    "voting_start_time": "2020-10-06T05:04:43.988Z"
  },
  {
    "content": {
      "type": "params/ParameterChangeProposal",
      "value": {
        "changes": [
          {
            "key": "KeyMaxEntries",
            "subspace": "staking",
            "value": "100"
          }
        ],
        "description": "Test Parameter-change Proposal",
        "title": "Test"
      }
    },
    "deposit_end_time": "2020-10-14T03:13:35.869Z",
    "final_tally_result": {
      "abstain": "0",
      "no": "0",
      "no_with_veto": "0",
      "yes": "0"
    },
    "id": "4",
    "proposal_status": "Rejected",
    "submit_time": "2020-10-13T03:13:35.869Z",
    "total_deposit": [
      {
        "amount": "10000000",
        "denom": "uluna"
      }
    ],
    "voting_end_time": "2020-10-14T03:13:35.869Z",
    "voting_start_time": "2020-10-13T03:13:35.869Z"
  },
  {
    "content": {
      "type": "params/ParameterChangeProposal",
      "value": {
        "changes": [
          {
            "key": "KeyMaxEntries",
            "subspace": "staking",
            "value": "100"
          }
        ],
        "description": "Increase max entries to 100",
        "title": "Undelegation Max Entries"
      }
    },
    "deposit_end_time": "2020-10-15T12:49:21.321Z",
    "final_tally_result": {
      "abstain": "0",
      "no": "0",
      "no_with_veto": "0",
      "yes": "238449256585"
    },
    "id": "5",
    "proposal_status": "Rejected",
    "submit_time": "2020-10-14T12:49:21.321Z",
    "total_deposit": [
      {
        "amount": "100000000",
        "denom": "uluna"
      }
    ],
    "voting_end_time": "2020-10-15T12:49:21.321Z",
    "voting_start_time": "2020-10-14T12:49:21.321Z"
  },
  {
    "content": {
      "type": "gov/TextProposal",
      "value": {
        "description": "Deposit Test...",
        "title": "Deposit Test..."
      }
    },
    "deposit_end_time": "2020-10-16T07:53:04.395Z",
    "final_tally_result": {
      "abstain": "9999999",
      "no": "0",
      "no_with_veto": "0",
      "yes": "0"
    },
    "id": "6",
    "proposal_status": "Rejected",
    "submit_time": "2020-10-15T07:53:04.395Z",
    "total_deposit": [
      {
        "amount": "100000000",
        "denom": "uluna"
      }
    ],
    "voting_end_time": "2020-10-16T07:53:31.803Z",
    "voting_start_time": "2020-10-15T07:53:31.803Z"
  },
  {
    "content": {
      "type": "gov/TextProposal",
      "value": {
        "description": "Deposit!",
        "title": "Depost Test.. retry..."
      }
    },
    "deposit_end_time": "2020-10-19T05:09:58.460Z",
    "final_tally_result": {
      "abstain": "0",
      "no": "0",
      "no_with_veto": "0",
      "yes": "0"
    },
    "id": "9",
    "proposal_status": "Rejected",
    "submit_time": "2020-10-18T05:09:58.460Z",
    "total_deposit": [
      {
        "amount": "14999999",
        "denom": "uluna"
      }
    ],
    "voting_end_time": "2020-10-19T05:27:22.768Z",
    "voting_start_time": "2020-10-18T05:27:22.768Z"
  }
]
```

In this response, you can find a list of proposals. Each one comes with an ID, title, description, status, deposit, and final tally result. You can read more about governance on Terra [**here**](https://docs.terra.money/governance.html#proposals).

## **Conclusion**

Congratulations! Now you know how to query a Terra node with [**DataHub**](https://figment.io/datahub-waitlist/) to get the information you need.

For more queries please refer to the Terra documentation for the [**Tendermint RPC**](https://learn.datahub.figment.io/network-documentation/terra/rpc-and-rest-api/tendermint-rpc-1) and [**Terra LCD**](https://learn.datahub.figment.io/network-documentation/terra/rpc-and-rest-api/terra-lcd). 

The complete code for this tutorial can be found [**here**](https://github.com/figment-networks/tutorials/blob/main/terra/3_query_node/query.js). 

## **Next steps**

Being able to query a Terra node is fun, but wouldn’t it be great if you could not only read but also write to the blockchain? In the next tutorial, we will explore the use of transactions in order to write to the blockchain state.

If you had any difficulties following this tutorial or simply want to discuss Terra tech with us you can [**join our community today**](https://discord.gg/fszyM7K)!

