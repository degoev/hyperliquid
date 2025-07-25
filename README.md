# Hyperliquid API TypeScript SDK

[![NPM](https://img.shields.io/npm/v/@nktkas/hyperliquid?style=flat-square&color=blue)](https://www.npmjs.com/package/@nktkas/hyperliquid)
[![JSR](https://img.shields.io/jsr/v/@nktkas/hyperliquid?style=flat-square&color=blue)](https://jsr.io/@nktkas/hyperliquid)
[![Coveralls](https://img.shields.io/coverallsCoverage/github/nktkas/hyperliquid?style=flat-square)](https://coveralls.io/github/nktkas/hyperliquid)
[![bundlejs](https://img.shields.io/bundlejs/size/@nktkas/hyperliquid?style=flat-square)](https://bundlejs.com/?q=@nktkas/hyperliquid)

Unofficial [Hyperliquid API](https://hyperliquid.gitbook.io/hyperliquid-docs/for-developers/api) SDK for all major JS
runtimes, written in TypeScript and provided with tests.

## Features

- 🖋️ **Typed**: Source code is 100% TypeScript.
- 🧪 **Tested**: Good code coverage and type-safe API responses.
- 📦 **Minimal dependencies**: A few small trusted dependencies.
- 🌐 **Cross-Environment Support**: Compatible with all major JS runtimes.
- 🔧 **Integratable**: Easy to use with [viem](https://github.com/wevm/viem),
  [ethers](https://github.com/ethers-io/ethers.js) and other wallet libraries.
- 📚 **Documented**: JSDoc annotations with usage examples in source code.

## Installation

> [!NOTE]
> While this library is in TypeScript, it can also be used in JavaScript and supports ESM/CommonJS.

### Node.js (choose your package manager)

```
npm i @nktkas/hyperliquid

pnpm add @nktkas/hyperliquid

yarn add @nktkas/hyperliquid
```

### Deno

```
deno add jsr:@nktkas/hyperliquid
```

### Web

```html
<script type="module">
    import * as hl from "https://esm.sh/jsr/@nktkas/hyperliquid";
    // Use hl.InfoClient, hl.ExchangeClient, etc.
</script>
```

### React Native

<details>
<summary>For React Native, you need to import several polyfills before importing the SDK:</summary>

```js
// React Native 0.76.3
import { Event, EventTarget } from "event-target-shim";

if (!globalThis.EventTarget || !globalThis.Event) {
    globalThis.EventTarget = EventTarget;
    globalThis.Event = Event;
}

if (!globalThis.CustomEvent) {
    globalThis.CustomEvent = function (type, params) {
        params = params || {};
        const event = new Event(type, params);
        event.detail = params.detail || null;
        return event;
    };
}

if (!AbortSignal.timeout) {
    AbortSignal.timeout = function (delay) {
        const controller = new AbortController();
        setTimeout(() => controller.abort(), delay);
        return controller.signal;
    };
}

if (!Promise.withResolvers) {
    Promise.withResolvers = function () {
        let resolve, reject;
        const promise = new Promise((res, rej) => {
            resolve = res;
            reject = rej;
        });
        return { promise, resolve, reject };
    };
}

if (!ArrayBuffer.prototype.transfer) {
    ArrayBuffer.prototype.transfer = function (newByteLength) {
        const length = newByteLength ?? this.byteLength;
        const newBuffer = new ArrayBuffer(length);
        const oldView = new Uint8Array(this);
        const newView = new Uint8Array(newBuffer);

        newView.set(oldView.subarray(0, Math.min(oldView.length, length)));

        Object.defineProperty(this, "byteLength", { value: 0 });

        return newBuffer;
    };
}
```

</details>

## Quick Start

#### Info endpoint

```ts
import * as hl from "@nktkas/hyperliquid";

const transport = new hl.HttpTransport();
const infoClient = new hl.InfoClient({ transport });

const openOrders = await infoClient.openOrders({ user: "0x..." });
```

#### Exchange endpoint

```ts
import * as hl from "@nktkas/hyperliquid";
import { privateKeyToAccount } from "viem/accounts"; // or other wallet libraries

const wallet = privateKeyToAccount("0x...");

const transport = new hl.HttpTransport();
const exchClient = new hl.ExchangeClient({ wallet, transport });

const result = await exchClient.order({
    orders: [{
        a: 0,
        b: true,
        p: "30000",
        s: "0.1",
        r: false,
        t: {
            limit: {
                tif: "Gtc",
            },
        },
    }],
    grouping: "na",
});
```

#### Subscription

```ts
import * as hl from "@nktkas/hyperliquid";

const transport = new hl.WebSocketTransport();
const subsClient = new hl.SubscriptionClient({ transport });

const sub = await subsClient.allMids((event) => {
    console.log(event);
});

await sub.unsubscribe(); // unsubscribe from the event
```

#### Multi-Sign

```ts
import * as hl from "@nktkas/hyperliquid";
import { privateKeyToAccount } from "viem/accounts"; // or other wallet libraries

const multiSignAddress = "0x...";
const signers = [
    privateKeyToAccount("0x..."), // first is leader
    privateKeyToAccount("0x..."), // can be a custom async wallet
    // ...
    privateKeyToAccount("0x..."),
];

const transport = new hl.HttpTransport();
const multiSignClient = new hl.MultiSignClient({ transport, multiSignAddress, signers }); // extends ExchangeClient

const data = await multiSignClient.approveAgent({ // same API as ExchangeClient
    agentAddress: "0x...",
    agentName: "agentName",
});
```

<details>
<summary><h2>Usage</h2></summary>

### 1) Initialize Transport

First, choose and configure your transport layer (more details in the [API Reference](#transports)):

```ts
import * as hl from "@nktkas/hyperliquid";

// HTTP Transport
const httpTransport = new hl.HttpTransport(); // Accepts optional parameters

// WebSocket Transport
const wsTransport = new hl.WebSocketTransport(); // Accepts optional parameters
```

### 2) Initialize Client

Next, initialize a client with the transport layer (more details in the [API Reference](#clients)):

#### Create InfoClient

```ts
import * as hl from "@nktkas/hyperliquid";

const transport = new hl.HttpTransport(); // or WebSocketTransport
const infoClient = new hl.InfoClient({ transport });
```

#### Create ExchangeClient

```ts
import * as hl from "@nktkas/hyperliquid";
import { createWalletClient, custom } from "viem";
import { privateKeyToAccount } from "viem/accounts";
import { ethers } from "ethers";

const transport = new hl.HttpTransport(); // or WebSocketTransport

// 1. Using Viem with private key
const viemAccount = privateKeyToAccount("0x...");
const exchClient_viem = new hl.ExchangeClient({ wallet: viemAccount, transport });

// 2. Using Ethers (or Ethers V5) with private key
const ethersWallet = new ethers.Wallet("0x...");
const exchClient_ethers = new hl.ExchangeClient({ wallet: ethersWallet, transport });

// 3. Using external wallet (e.g. MetaMask) via Viem
const [account] = await window.ethereum.request({ method: "eth_requestAccounts" });
const externalWallet = createWalletClient({ account, transport: custom(window.ethereum) });
const exchClient_viemMetamask = new hl.ExchangeClient({ wallet: externalWallet, transport });

// 4. Using external wallet (e.g. MetaMask) via `window.ethereum` (EIP-1193)
const exchClient_windowMetamask = new hl.ExchangeClient({ wallet: window.ethereum, transport });
```

#### Create SubscriptionClient

```ts
import * as hl from "@nktkas/hyperliquid";

const transport = new hl.WebSocketTransport(); // only WebSocketTransport
const subsClient = new hl.SubscriptionClient({ transport });
```

#### Create MultiSignClient

```ts
import * as hl from "@nktkas/hyperliquid";
import { privateKeyToAccount } from "viem/accounts";
import { ethers } from "ethers";

const multiSignAddress = "0x...";
const signers = [
    privateKeyToAccount("0x..."), // first is leader for multi-sign transaction, must contain own address
    { // can be a custom async wallet
        signTypedData(params: {
            domain: {
                name: string;
                version: string;
                chainId: number;
                verifyingContract: Hex;
            };
            types: {
                [key: string]: {
                    name: string;
                    type: string;
                }[];
            };
            primaryType: string;
            message: Record<string, unknown>;
        }): Promise<Hex> {
            // Custom signer logic
            return "0x..."; // return signature
        },
    },
    // ...
    new ethers.Wallet("0x..."),
];

const transport = new hl.HttpTransport();
const multiSignClient = new hl.MultiSignClient({ transport, multiSignAddress, signers }); // extends ExchangeClient
```

### 3) Use Client

Finally, use client methods to interact with the Hyperliquid API (more details in the [API Reference](#clients)):

#### Example of using an InfoClient

```ts
import * as hl from "@nktkas/hyperliquid";

const transport = new hl.HttpTransport();
const infoClient = new hl.InfoClient({ transport });

// L2 Book
const l2Book = await infoClient.l2Book({ coin: "BTC" });

// Account clearinghouse state
const clearinghouseState = await infoClient.clearinghouseState({ user: "0x..." });

// Open orders
const openOrders = await infoClient.openOrders({ user: "0x..." });
```

#### Example of using an ExchangeClient

```ts
import * as hl from "@nktkas/hyperliquid";
import { privateKeyToAccount } from "viem/accounts";

const account = privateKeyToAccount("0x...");

const transport = new hl.HttpTransport();
const exchClient = new hl.ExchangeClient({ wallet: account, transport });

// Place an orders
const result = await exchClient.order({
    orders: [{
        a: 0,
        b: true,
        p: "30000",
        s: "0.1",
        r: false,
        t: {
            limit: {
                tif: "Gtc",
            },
        },
    }],
    grouping: "na",
});

// Approve an agent
const result = await exchClient.approveAgent({
    agentAddress: "0x...",
    agentName: "agentName",
});

// Withdraw funds
const result = await exchClient.withdraw3({
    destination: account.address,
    amount: "100",
});
```

#### Example of using a SubscriptionClient

```ts
import * as hl from "@nktkas/hyperliquid";

const transport = new hl.WebSocketTransport();
const subsClient = new hl.SubscriptionClient({ transport });

// L2 Book updates
await subsClient.l2Book({ coin: "BTC" }, (data) => {
    console.log(data);
});

// User fills
await subsClient.userFills({ user: "0x..." }, (data) => {
    console.log(data);
});

// Candle updates
const sub = await subsClient.candle({ coin: "BTC", interval: "1h" }, (data) => {
    console.log(data);
});
```

#### Example of using a MultiSignClient

```ts
import * as hl from "@nktkas/hyperliquid";
import { privateKeyToAccount } from "viem/accounts";

const multiSignAddress = "0x...";
const signers = [privateKeyToAccount("0x...")];

const transport = new hl.HttpTransport();
const multiSignClient = new hl.MultiSignClient({ transport, multiSignAddress, signers });

// Interaction is the same as with ExchangeClient

// Place an orders
const result = await multiSignClient.order({
    orders: [{
        a: 0,
        b: true,
        p: "30000",
        s: "0.1",
        r: false,
        t: {
            limit: {
                tif: "Gtc",
            },
        },
    }],
    grouping: "na",
});

// Approve an agent
const result = await multiSignClient.approveAgent({
    agentAddress: "0x...",
    agentName: "agentName",
});

// Withdraw funds
const result = await multiSignClient.withdraw3({
    destination: account.address,
    amount: "100",
});
```

</details>

<details>
<summary><h2>API Reference</h2></summary>

### Clients

A client is an interface through which you can interact with the Hyperliquid API.

The client is responsible for formatting an action, creating a signature correctly, sending a request, and validating a
response.

#### InfoClient

```ts
class InfoClient {
    constructor(args: {
        transport: HttpTransport | WebSocketTransport;
    });

    // Market
    allMids(): Promise<AllMids>;
    candleSnapshot(args: CandleSnapshotParameters): Promise<Candle[]>;
    fundingHistory(args: FundingHistoryParameters): Promise<FundingHistory[]>;
    l2Book(args: L2BookParameters): Promise<Book>;
    meta(): Promise<PerpsMeta>;
    metaAndAssetCtxs(): Promise<PerpsMetaAndAssetCtxs>;
    perpDeployAuctionStatus(): Promise<DeployAuctionStatus>;
    perpDexs(): Promise<(PerpDex | null)[]>;
    perpsAtOpenInterestCap(): Promise<string[]>;
    predictedFundings(): Promise<PredictedFunding[]>;
    spotDeployState(args: SpotDeployStateParameters): Promise<SpotDeployState>;
    spotMeta(): Promise<SpotMeta>;
    spotMetaAndAssetCtxs(): Promise<SpotMetaAndAssetCtxs>;
    tokenDetails(args: TokenDetailsParameters): Promise<TokenDetails>;

    // Account
    clearinghouseState(args: ClearinghouseStateParameters): Promise<PerpsClearinghouseState>;
    extraAgents(args: ExtraAgentsParameters): Promise<ExtraAgent[]>;
    isVip(args: IsVipParameters): Promise<boolean>;
    legalCheck(args: LegalCheckParameters): Promise<LegalCheck>;
    maxBuilderFee(args: MaxBuilderFeeParameters): Promise<number>;
    portfolio(args: PortfolioParameters): Promise<PortfolioPeriods>;
    preTransferCheck(args: PreTransferCheckParameters): Promise<PreTransferCheck>;
    referral(args: ReferralParameters): Promise<Referral>;
    spotClearinghouseState(args: SpotClearinghouseStateParameters): Promise<SpotClearinghouseState>;
    subAccounts(args: SubAccountsParameters): Promise<SubAccount[] | null>;
    userFees(args: UserFeesParameters): Promise<UserFees>;
    userFunding(args: UserFundingParameters): Promise<UserFundingUpdate[]>;
    userNonFundingLedgerUpdates(args: UserNonFundingLedgerUpdatesParameters): Promise<UserNonFundingLedgerUpdate[]>;
    userRateLimit(args: UserRateLimitParameters): Promise<UserRateLimit>;
    userRole(args: UserRoleParameters): Promise<UserRole>;
    userToMultiSigSigners(args: UserToMultiSigSignersParameters): Promise<MultiSigSigners | null>;

    // Order
    frontendOpenOrders(args: FrontendOpenOrdersParameters): Promise<FrontendOrder[]>;
    historicalOrders(args: HistoricalOrdersParameters): Promise<OrderStatus<FrontendOrder>[]>;
    openOrders(args: OpenOrdersParameters): Promise<Order[]>;
    orderStatus(args: OrderStatusParameters): Promise<OrderLookup>;
    twapHistory(args: TwapHistoryParameters): Promise<TwapHistory[]>;
    userFills(args: UserFillsParameters): Promise<Fill[]>;
    userFillsByTime(args: UserFillsByTimeParameters): Promise<Fill[]>;
    userTwapSliceFills(args: UserTwapSliceFillsParameters): Promise<TwapSliceFill[]>;
    userTwapSliceFillsByTime(args: UserTwapSliceFillsByTimeParameters): Promise<TwapSliceFill[]>;

    // Staking
    delegations(args: DelegationsParameters): Promise<Delegation[]>;
    delegatorHistory(args: DelegatorHistoryParameters): Promise<DelegatorUpdate[]>;
    delegatorRewards(args: DelegatorRewardsParameters): Promise<DelegatorReward[]>;
    delegatorSummary(args: DelegatorSummaryParameters): Promise<DelegatorSummary>;
    validatorSummaries(): Promise<ValidatorSummary[]>;

    // Vault
    userVaultEquities(args: UserVaultEquitiesParameters): Promise<VaultEquity[]>;
    vaultDetails(args: VaultDetailsParameters): Promise<VaultDetails | null>;
    vaultSummaries(): Promise<VaultSummary[]>;

    // Explorer (RPC endpoint)
    blockDetails(args: BlockDetailsParameters): Promise<BlockDetails>;
    txDetails(args: TxDetailsParameters): Promise<TxDetails>;
    userDetails(args: UserDetailsParameters): Promise<TxDetails[]>;
}
```

#### ExchangeClient

```ts
class ExchangeClient {
    constructor(args: {
        transport: HttpTransport | WebSocketTransport;
        wallet:
            | AbstractViemWalletClient // viem
            | AbstractEthersSigner // ethers
            | AbstractEthersV5Signer // ethers v5
            | AbstractWindowEthereum; // window.ethereum (EIP-1193)
        isTestnet?: boolean; // Whether to use testnet (default: false)
        defaultVaultAddress?: Hex; // Vault address used by default if not provided in method call
        signatureChainId?: Hex | (() => MaybePromise<Hex>); // Chain ID used for signing (default: trying to guess based on wallet and isTestnet)
        nonceManager?: () => MaybePromise<number>; // Function to get the next nonce (default: auto-incrementing Date.now())
    });

    // Order
    batchModify(args: BatchModifyParameters): Promise<OrderResponseSuccess>;
    cancel(args: CancelParameters): Promise<CancelResponseSuccess>;
    cancelByCloid(args: CancelByCloidParameters): Promise<CancelResponseSuccess>;
    modify(args: ModifyParameters): Promise<SuccessResponse>;
    order(args: OrderParameters): Promise<OrderResponseSuccess>;
    scheduleCancel(args?: ScheduleCancelParameters): Promise<SuccessResponse>;
    twapCancel(args: TwapCancelParameters): Promise<TwapCancelResponseSuccess>;
    twapOrder(args: TwapOrderParameters): Promise<TwapOrderResponseSuccess>;
    updateIsolatedMargin(args: UpdateIsolatedMarginParameters): Promise<SuccessResponse>;
    updateLeverage(args: UpdateLeverageParameters): Promise<SuccessResponse>;

    // Account
    approveAgent(args: ApproveAgentParameters): Promise<SuccessResponse>;
    approveBuilderFee(args: ApproveBuilderFeeParameters): Promise<SuccessResponse>;
    claimRewards(): Promise<SuccessResponse>;
    convertToMultiSigUser(args: ConvertToMultiSigUserParameters): Promise<SuccessResponse>;
    createSubAccount(args: CreateSubAccountParameters): Promise<CreateSubAccountResponse>;
    evmUserModify(args: EvmUserModifyParameters): Promise<SuccessResponse>;
    registerReferrer(args: RegisterReferrerParameters): Promise<SuccessResponse>;
    reserveRequestWeight(args: ReserveRequestWeightParameters): Promise<SuccessResponse>;
    setDisplayName(args: SetDisplayNameParameters): Promise<SuccessResponse>;
    setReferrer(args: SetReferrerParameters): Promise<SuccessResponse>;
    spotUser(args: SpotUserParameters): Promise<SuccessResponse>;

    // Transfer
    perpDexClassTransfer(args: PerpDexClassTransferParameters): Promise<SuccessResponse>;
    spotSend(args: SpotSendParameters): Promise<SuccessResponse>;
    subAccountSpotTransfer(args: SubAccountSpotTransferParameters): Promise<SuccessResponse>;
    subAccountTransfer(args: SubAccountTransferParameters): Promise<SuccessResponse>;
    usdClassTransfer(args: UsdClassTransferParameters): Promise<SuccessResponse>;
    usdSend(args: UsdSendParameters): Promise<SuccessResponse>;
    withdraw3(args: Withdraw3Parameters): Promise<SuccessResponse>;

    // Staking
    cDeposit(args: CDepositParameters): Promise<SuccessResponse>;
    cWithdraw(args: CWithdrawParameters): Promise<SuccessResponse>;
    tokenDelegate(args: TokenDelegateParameters): Promise<SuccessResponse>;

    // Market
    perpDeploy(args: PerpDeployParameters): Promise<SuccessResponse>;
    spotDeploy(args: SpotDeployParameters): Promise<SuccessResponse>;

    // Vault
    createVault(args: CreateVaultParameters): Promise<CreateVaultResponse>;
    vaultDistribute(args: VaultDistributeParameters): Promise<SuccessResponse>;
    vaultModify(args: VaultModifyParameters): Promise<SuccessResponse>;
    vaultTransfer(args: VaultTransferParameters): Promise<SuccessResponse>;

    // Multi-Sign
    multiSig(args: MultiSigParameters): Promise<BaseExchangeResponse>;

    // Validator
    cSignerAction(args: CSignerActionParameters): Promise<SuccessResponse>;
    cValidatorAction(args: CValidatorActionParameters): Promise<SuccessResponse>;
}
```

#### SubscriptionClient

<!-- deno-fmt-ignore-start -->
```ts
class SubscriptionClient {
    constructor(args: {
        transport: WebSocketTransport;
    });

    // Market
    activeAssetCtx(args: EventActiveAssetCtxParameters, listener: (data: WsActiveAssetCtx | WsActiveSpotAssetCtx) => void): Promise<Subscription>;
    activeAssetData(args: EventActiveAssetDataParameters, listener: (data: WsActiveAssetData) => void): Promise<Subscription>;
    allMids(listener: (data: WsAllMids) => void): Promise<Subscription>;
    bbo(args: EventBboParameters, listener: (data: WsBbo) => void): Promise<Subscription>;
    candle(args: EventCandleParameters, listener: (data: Candle) => void): Promise<Subscription>;
    l2Book(args: EventL2BookParameters, listener: (data: Book) => void): Promise<Subscription>;
    trades(args: EventTradesParameters, listener: (data: WsTrade[]) => void): Promise<Subscription>;

    // Account
    notification(args: EventNotificationParameters, listener: (data: WsNotification) => void): Promise<Subscription>;
    userEvents(args: EventUserEventsParameters, listener: (data: WsUserEvent) => void): Promise<Subscription>;
    userFundings(args: EventUserFundingsParameters, listener: (data: WsUserFundings) => void): Promise<Subscription>;
    userNonFundingLedgerUpdates(args: EventUserNonFundingLedgerUpdatesParameters, listener: (data: WsUserNonFundingLedgerUpdates) => void): Promise<Subscription>;
    webData2(args: EventWebData2Parameters, listener: (data: WsWebData2) => void): Promise<Subscription>;

    // Order
    orderUpdates(args: EventOrderUpdatesParameters, listener: (data: OrderStatus<Order>[]) => void): Promise<Subscription>;
    userFills(args: EventUserFillsParameters, listener: (data: WsUserFills) => void): Promise<Subscription>;
    userTwapHistory(args: EventUserTwapHistory, listener: (data: WsUserTwapHistory) => void): Promise<Subscription>;
    userTwapSliceFills(args: EventUserTwapSliceFills, listener: (data: WsUserTwapSliceFills) => void): Promise<Subscription>;

    // Explorer
    explorerBlock(listener: (data: WsBlockDetails[]) => void): Promise<Subscription>;
    explorerTx(listener: (data: TxDetails[]) => void): Promise<Subscription>;
}
```
<!-- deno-fmt-ignore-end -->

#### MultiSignClient

```ts
class MultiSignClient extends ExchangeClient {
    constructor(
        args:
            & Omit<ExchangeClientParameters, "wallet"> // Instead of `wallet`, you should specify the following parameters:
            & {
                multiSignAddress: Hex; // Multi-signature address
                signers: [ // Array of signers
                    AbstractWalletWithAddress, // First signer is the leader of a multi-sign transaction
                    ...AbstractWallet[], // Any number of additional signers
                ];
            },
    );

    // Same methods as ExchangeClient
}
```

### Transports

Transport acts as a layer between the class and Hyperliquid servers.

#### HTTP Transport

```ts
class HttpTransport {
    constructor(options?: {
        isTestnet?: boolean; // Whether to use testnet url (default: false)
        timeout?: number; // Request timeout in ms (default: 10_000)
        server?: { // Custom server URLs
            mainnet?: { api?: string | URL; rpc?: string | URL };
            testnet?: { api?: string | URL; rpc?: string | URL };
        };
        fetchOptions?: RequestInit; // A custom fetch options
        onRequest?: (request: Request) => MaybePromise<Request | void | null | undefined>; // A callback before request is sent
        onResponse?: (response: Response) => MaybePromise<Response | void | null | undefined>; // A callback after response is received
    });
}
```

#### WebSocket Transport

```ts
class WebSocketTransport {
    constructor(options?: {
        url?: string | URL; // WebSocket URL (default: "wss://api.hyperliquid.xyz/ws")
        timeout?: number; // Request timeout in ms (default: 10_000)
        keepAlive?: {
            interval?: number; // Ping interval in ms (default: 30_000)
            timeout?: number; // Pong timeout in ms (default: same as `timeout` for requests)
        };
        reconnect?: {
            maxRetries?: number; // Maximum number of reconnection attempts (default: 3)
            connectionTimeout?: number; // Connection timeout in ms (default: 10_000)
            connectionDelay?: number | ((attempt: number) => number | Promise<number>); // Delay between reconnection (default: Exponential backoff (max 10s))
            shouldReconnect?: (event: CloseEvent) => boolean | Promise<boolean>; // Custom reconnection logic (default: Always reconnect)
            messageBuffer?: MessageBufferStrategy; // Message buffering strategy between reconnection (default: FIFO buffer)
        };
        autoResubscribe?: boolean; // Whether to automatically resubscribe to events after reconnection (default: true)
    });
    ready(signal?: AbortSignal): Promise<void>;
    close(signal?: AbortSignal): Promise<void>;
}
```

</details>

<details>
<summary><h2>Additional Import Points</h2></summary>

### `/types`

The import point gives access to all Hyperliquid-related types, including the base types on which class methods are
based.

### `/signing`

The import point gives access to functions that generate signatures for Hyperliquid transactions.

### Examples

#### Cancel order yourself

```ts
import { actionSorter, signL1Action } from "@nktkas/hyperliquid/signing";
import { privateKeyToAccount } from "viem/accounts"; // or other wallet libraries

const wallet = privateKeyToAccount("0x...");

const action = {
    type: "cancel",
    cancels: [
        { a: 0, o: 12345 },
    ],
};
const nonce = Date.now();

const signature = await signL1Action({
    wallet,
    action: actionSorter[action.type](action), // key order affects signature
    nonce,
    isTestnet: true, // change to `false` for mainnet
});

const response = await fetch("https://api.hyperliquid-testnet.xyz/exchange", {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({ action, signature, nonce }),
});
const body = await response.json();
```

#### Approve agent yourself

```ts
import { signUserSignedAction, userSignedActionEip712Types } from "@nktkas/hyperliquid/signing";
import { privateKeyToAccount } from "viem/accounts"; // or other wallet libraries

const wallet = privateKeyToAccount("0x...");

const action = {
    type: "approveAgent",
    signatureChainId: "0x66eee", // must match the current wallet network
    hyperliquidChain: "Testnet", // Mainnet | Testnet
    agentAddress: "0x...",
    agentName: "Agent",
    nonce: Date.now(),
};

const signature = await signUserSignedAction({
    wallet,
    action,
    types: userSignedActionEip712Types[action.type], // key order affects signature
    chainId: parseInt(action.signatureChainId, 16),
});

const response = await fetch("https://api.hyperliquid-testnet.xyz/exchange", {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({ action, signature, nonce: action.nonce }),
});
const body = await response.json();
```

</details>

## Contributing

We appreciate your help! To contribute, please read the [contributing instructions](CONTRIBUTING.md).
