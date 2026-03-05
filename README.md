# ILiquid Interface

> Solidity interface for the Liquid protocol — a hub-and-spoke AMM that wraps ERC-20 tokens with built-in constant-product liquidity.

## Overview

ILiquid is the canonical interface for the Liquid protocol, where:

- **Solid** tokens are wrapped into **Liquid** tokens with built-in AMM liquidity
- A single **Hub** instance serves as the intermediary for all cross-pool swaps
- **Spoke** instances created via `make` each wrap a different ERC-20 backing token
- Contracts are deployed **deterministically** using CREATE2
- Uses EIP-1167 minimal proxy pattern (OpenZeppelin Clones)

## Installation

### Foundry

```bash
forge install uniteum/iliquid
```

Add to your `remappings.txt`:

```
iliquid/=lib/iliquid/
```

### Usage in Solidity

```solidity
import {ILiquid} from "iliquid/ILiquid.sol";

contract MyContract {
    ILiquid public immutable HUB;

    constructor(ILiquid hub) {
        HUB = hub;
    }

    function createLiquid(IERC20Metadata backing) external returns (ILiquid liquid) {
        liquid = HUB.make(backing);
    }
}
```

## Core Functions

### Liquidity

#### `heat(uint256 m) → (uint256 u, uint256 p)`

Deposit backing tokens (solid), mint liquid tokens split between caller and pool.

#### `heat(uint256 m, uint256 e) → (uint256 u, uint256 p)`

Deposit backing tokens and hub tokens, mint liquid tokens.

#### `cool(uint256 u) → (uint256 m, uint256 p)`

Burn liquid tokens, redeem proportional backing tokens.

#### `cool(uint256 u, uint256 e) → (uint256 m, uint256 p)`

Burn liquid tokens, redeem backing tokens plus hub tokens.

### Trading

#### `sell(uint256 s) → (uint256 e)`

Sell spoke tokens for hub tokens via the constant-product AMM.

#### `buy(uint256 e) → (uint256 s)`

Buy spoke tokens with hub tokens.

#### `sellFor(ILiquid that, uint256 s) → (uint256 e, uint256 thats)`

Atomic cross-spoke swap: sell this spoke's tokens, buy another's, routing through the hub.

### Quoting

#### `heats(uint256 m) → (uint256 u, uint256 p)`
#### `heats(uint256 m, uint256 e) → (uint256 u, uint256 p)`
#### `cools(uint256 u) → (uint256 m, uint256 p)`
#### `cools(uint256 u, uint256 e) → (uint256 m, uint256 p)`
#### `sells(uint256 s) → (uint256 e)`
#### `buys(uint256 e) → (uint256 s)`
#### `sellsFor(ILiquid that, uint256 s) → (uint256 e, uint256 thats)`

View functions that return expected outputs without executing.

### Factory

#### `make(IERC20Metadata backing) → (ILiquid liquid)`

Deploy a new spoke for the given backing token via deterministic CREATE2.

#### `made(IERC20Metadata backing) → (bool cloned, address home, bytes32 salt)`

Check whether a spoke exists and compute its deterministic address.

### Queries

#### `pool() → (uint256 S, uint256 E)`

Current pool reserves: spoke tokens (S) and hub tokens (E).

#### `mass() → uint256`

Backing token balance held by the contract.

#### `solid() → IERC20Metadata`

The backing ERC-20 token this instance wraps.

## Events

- `Heat(ILiquid indexed liquid, uint256 solids, uint256 pools, uint256 senders)`
- `Cool(ILiquid indexed liquid, uint256 liquids, uint256 hubs, uint256 solids)`
- `Buy(ILiquid indexed liquid, uint256 liquids, uint256 hubs)`
- `Sell(ILiquid indexed liquid, uint256 liquids, uint256 hubs)`
- `Make(ILiquid indexed liquid, IERC20Metadata indexed solid)`

## Errors

- `HubNotPool()` — Operation not available on the hub
- `Nothing()` — Zero-output operation
- `Unauthorized()` — Caller is not a registered Liquid instance

## Links

- [Liquid Protocol](https://github.com/uniteum/liquid)

## Version

Solidity ^0.8.30 (EIP-1153 transient storage support).
