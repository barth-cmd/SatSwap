# SatSwap Protocol

**SatSwap** is a decentralized exchange (DEX) protocol built on the [Stacks](https://www.stacks.co/) Layer 2 blockchain, enabling **trustless token swaps** for Bitcoin-compatible assets. Using the proven **constant product AMM model (x \* y = k)**, SatSwap supports on-chain liquidity provisioning, seamless token exchanges, and protocol-level administrative controls.

## Overview

SatSwap allows users to:

* **Create liquidity pools** for pairs of fungible tokens.
* **Add or remove liquidity** in exchange for pool shares.
* **Swap tokens** via the constant product formula with configurable slippage protection.
* **Collect protocol fees** from swap operations.
* **Pause or resume pools** as part of administrative safety controls.

It is designed to support any SIP-010-compliant fungible token on Stacks.

## Key Features

### AMM Engine

* Implements the constant product formula `x * y = k` for price discovery.
* Ensures minimal slippage for swaps through real-time calculations.
* Fee-adjusted output accounting for protocol fees.

### Liquidity Provision

* Liquidity providers earn proportional shares of trading activity.
* LP tokens (tracked internally) represent pool ownership.
* Supports initial pool bootstrapping and dynamic share calculation.

### Token Swaps

* Trustless swaps between SIP-010 tokens.
* Price slippage checks with user-defined thresholds.
* Automatic reserve updates and fee collection.

### Admin Controls

* Protocol owner can:

  * Set protocol-wide swap fee (`0.3%` default).
  * Pause or resume pools.
* Built-in checks prevent unauthorized or invalid operations.

## Contract Interface

### Traits

* **`ft-trait`**: Standard interface for fungible tokens.

### Constants

* `PRECISION`: `1_000_000` (used for decimal accuracy)
* Error codes: Ranging from `ERR-NOT-AUTHORIZED (100)` to `ERR-ZERO-LIQUIDITY (106)`

## Usage

### 1. Create a New Pool

```clarity
(create-pool token-x token-y)
```

Only the contract owner can call this to initiate a new trading pair.

### 2. Add Liquidity

```clarity
(add-liquidity pool-id token-x token-y amount-x amount-y min-shares)
```

Users deposit equal value of both tokens and receive LP shares.

### 3. Swap Tokens

```clarity
(swap-exact-tokens pool-id token-in token-out amount-in min-amount-out x-to-y)
```

Performs a swap from `token-in` to `token-out`. `x-to-y` determines swap direction.

### 4. Remove Liquidity

```clarity
(remove-liquidity pool-id token-x token-y shares min-amount-x min-amount-y)
```

Burns LP shares to withdraw proportional amounts of the underlying tokens.

## Read-Only Functions

| Function              | Description                                    |
| --------------------- | ---------------------------------------------- |
| `get-pool-info`       | Returns the full pool state by ID              |
| `get-provider-shares` | Fetches LP shares owned by a provider          |
| `get-exchange-rate`   | Calculates current token price ratio (Y per X) |

## Administrative Functions

| Function                     | Purpose                                      |
| ---------------------------- | -------------------------------------------- |
| `set-protocol-fee`           | Update protocol fee rate (up to `PRECISION`) |
| `pause-pool` / `resume-pool` | Temporarily disable/enable a specific pool   |

Only callable by the contract owner (`CONTRACT-OWNER`).

## Storage Overview

* **`pools`**: Tracks all liquidity pools and their reserves.
* **`liquidity-providers`**: Maps users to their LP token share per pool.
* **`accumulated-fees`** *(optional extension)*: Could be used for fee withdrawal (currently tracked but unused).

## Security & Design Considerations

* **Slippage Protection**: User-defined minimum output guards against front-running or volatility.
* **Permissioned Actions**: Only the contract deployer can change fees or pause pools.
* **State Consistency**: Strict type checks and reserve synchronization are enforced after each swap or liquidity update.

## Future Enhancements

* Fee distribution to LPs or protocol treasury.
* Multi-hop swaps (e.g. token A → B → C).
* LP token standardization (e.g. fungible LP tokens).
* Oracle integration for real-world asset pricing.

## Requirements

* **Stacks 2.1+**
* SIP-010 compliant tokens for pool participation
* Contract must be deployed by a wallet intending to act as the protocol admin
