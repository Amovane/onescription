# All-in-One inscription tool

![https://npm.im/@scriptione/one](https://img.shields.io/npm/v/@scriptione/one)
![](https://snyk.io/test/github/amovane/onescription/badge.svg)

A multi-chain inscription tool that can function as a standalone inscription bot or seamlessly integrate into web applications. The tool also boasts concurrent request handling, secure wallet generation, and a range of other valuable features.

- [Packages](#packages)
- [Features](#features)
- [Examples](#examples)
  - [use as an inscription bot](#to-use-as-an-inscription-bot)
  - [use in web application](#to-use-in-web-application)

## Packages

| Package                                                                                 | Version                                              | Security                                                        | Installation                         |
| --------------------------------------------------------------------------------------- | ---------------------------------------------------- | --------------------------------------------------------------- | ------------------------------------ |
| [@scriptione/evm](https://github.com/Amovane/onescription/tree/main/packages/evm)       | ![](https://img.shields.io/npm/v/@scriptione/evm)    | ![](https://snyk.io/test/github/amovane/onescription/badge.svg) | `yarn add @scriptione/evm@latest`    |
| [@scriptione/cosmos](https://github.com/Amovane/onescription/tree/main/packages/cosmos) | ![](https://img.shields.io/npm/v/@scriptione/cosmos) | ![](https://snyk.io/test/github/amovane/onescription/badge.svg) | `yarn add @scriptione/cosmos@latest` |

## Features

- Utility

  - [x] inscription bot
  - [x] can be integrated into web application

- Multi-chain support

  - [x] Evm-compatible chains
  - [x] Cosmos Hub

- Highly customizable

  - [x] configurable gas options, with automatic estimation as the default.
  - [x] selectively executed according to customized logic, such as writing execution logic based on block height or unix timestamp. [INJS demo](#cosmos)
  - [ ] configurable transaction type, including contract call or token transfer.

- Concurrent requests

  - [x] correct way to handle concurrent requests on nodejs, based on [async-mutext](https://github.com/DirtyHairy/async-mutex) / semaphore.
  - [ ] wait for each request until the user-defined status is reached.

- Wallet
  - [x] connect an existing signer from private key / mnemonic / secret csv file.
  - [x] create a new wallet and export it to secret file (CSV format).
  - [ ] may be a better practice to encrypt the generated secret file using [age encryption](https://github.com/FiloSottile/typage).

## Examples

### To use as an inscription bot

#### **Evm:**

opbrc

```typescript
import { EvmConfig, Inscriber, Onescription, Strategy } from "@scriptione/evm";
const configuration: EvmConfig = {
  os: "evm",
  chainId: 204,
  isSelfTransaction: false,
  value: 0,
  recipient: "0x83b978Cf73ee1D571b1a2550c5570861285AF337",
};
const inscriber = Inscriber.from(configuration);
inscriber.connectSignerFromPrivateKey("YOUR PRIVATE");
// or
// inscriber.connectSignerFromMnemonic("YOUR MNEMONIC");
const strategy: Strategy = {
  maxConcurrentRequests: 3,
  statusToWait: "submitted",
};
const onescription = new Onescription(inscriber, strategy);
// const obrcInsc: InscriptionText = data:application/json,{"p":"opbrc","op":"mint","tick":"obrc"}
const opbnInsc: InscriptionText = `data:application/json,{"p":"opbrc","op":"mint","tick":"opbn"}`;
const concurrCb = ({ hash, err }: Tx) => console.log(hash || err);
for (; ;) {
  await onescription.inscribe(inscription, concurrCb);
}
```

#### **Cosmos:**

Injective

```typescript
import {
  ChainInfoProvider,
  CosmosConfig,
  Inscriber,
  Onescription,
  Strategy,
} from "@scriptione/cosmos";

const configuration: CosmosConfig = {
  os: "cosmos",
  prefix: "inj",
  isSelfTransaction: true,
};
const inscriber = Inscriber.from(configuration);
inscriber.connectSignerFromMnemonic("YOUR MNEMONIC");
const strategy: Strategy = {
  maxConcurrentRequests: 2,
  statusToWait: "submitted",
  // The $INJS introduction is available in this link
  // https://docs.injs.ink/mint-injs
  predicate: async (provider: ChainInfoProvider) => {
    const blockHeight = await provider.getBlockHeight();
    console.log("current block height:", blockHeight);
    const rounds = [
      [55051600, 55053100],
      [55094800, 55096300],
      [55138000, 55139500],
      [55181200, 55182700],
      [55224400, 55225900],
      [55267600, 55269100],
      [55310800, 55312300],
      [55354000, 55355500],
    ];
    return (
      undefined !==
      rounds.find(([start, end]) => start <= blockHeight && blockHeight <= end)
    );
  },
};
const onescription = new Onescription(inscriber, strategy);
const inscription = { p: "injrc-20", op: "mint", tick: "INJS", amt: "1000" };
for (;;) {
  await onescription.inscribe(inscription);
}
```

### To use in web application

```typescript
import { EvmConfig, Inscriber } from "@scriptione/evm";
const configuration: EvmConfig = {
  os: "evm",
  chainId: 56,
  isSelfTransaction: true,
};
const inscriber = Inscriber.from(configuration);
const signer = new ethers.providers.Web3Provider(window.ethereum).getSigner();
inscriber.connectSigner(signer);
const inp: Inscription = { p: "bsc-20", op: "mint", tick: "bnbs", amt: "1000" };
await inscriber.inscribe(inp);
```
