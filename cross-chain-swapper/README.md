# Cross-Chain Swapper

A trustless protocol for atomic swaps between Stacks assets (STX and SIP-010 tokens) and Bitcoin.

## Overview

Cross-Chain Swapper enables secure peer-to-peer trading between Stacks blockchain assets and Bitcoin without requiring trusted intermediaries. It uses Hash Time-Locked Contracts (HTLCs) and cross-chain verification to ensure that either both parties receive their assets or both parties keep their original assets.

## How It Works

1. **Initiation**: A Stacks user locks their STX or SIP-010 tokens in the contract, specifying:
   - Amount of Stacks assets to swap
   - Expected Bitcoin amount
   - Bitcoin receiver address (where they want to receive BTC)
   - Expected Bitcoin sender address
   - Expiration time in blocks

2. **Execution**: The Bitcoin counterparty sends the specified BTC amount to the Bitcoin receiver address.

3. **Verification**: The Stacks transaction is proven on the Stacks blockchain using Bitcoin transaction proofs.

4. **Settlement**: Upon verification, the Stacks assets are released to the Bitcoin sender, completing the atomic swap.

5. **Expiration**: If the swap is not completed before the specified expiration time, the initiator can cancel the swap and recover their assets.

## Features

- **Trustless Operation**: No intermediaries or custodians required
- **Atomic Swaps**: Either both trades execute or neither does
- **Support for STX and Tokens**: Trade both STX and any SIP-010 compliant token
- **Configurable Parameters**: Adjustable protocol fees and expiration windows
- **Bitcoin Transaction Verification**: Uses Stacks' Bitcoin verification capabilities
- **Fee Generation**: Protocol fees support sustainable development

## Contract Functions

### Swap Creation

| Function | Description |
|----------|-------------|
| `create-stx-to-btc-swap` | Initiate a swap from STX to Bitcoin |
| `create-token-to-btc-swap` | Initiate a swap from a SIP-010 token to Bitcoin |

### Swap Execution and Management

| Function | Description |
|----------|-------------|
| `complete-swap` | Complete a swap by proving Bitcoin transaction execution |
| `cancel-swap` | Cancel an expired swap and recover assets |

### Administrative Functions

| Function | Description |
|----------|-------------|
| `set-protocol-fee-rate` | Update the protocol fee percentage (owner only) |
| `set-expiration-limits` | Set minimum and maximum expiration time limits (owner only) |
| `withdraw-protocol-fees-stx` | Withdraw accumulated STX protocol fees (owner only) |
| `withdraw-protocol-fees-token` | Withdraw accumulated token protocol fees (owner only) |

### Read-Only Functions

| Function | Description |
|----------|-------------|
| `get-swap` | Get details of a specific swap |
| `get-initiator-swaps` | Get all swaps initiated by a specific address |
| `get-protocol-fee-rate` | Get the current protocol fee rate |
| `get-expiration-limits` | Get the current expiration time limits |
| `get-swap-count` | Get the total number of swaps created |
| `is-swap-active` | Check if a swap is active |
| `is-swap-completed` | Check if a swap is completed |
| `is-swap-canceled` | Check if a swap is canceled |
| `is-swap-expired` | Check if a swap is expired |

## Default Configuration

- **Protocol Fee Rate**: 0.5% (50 basis points)
- **Minimum Expiration**: 100 blocks (~16.7 hours on Stacks)
- **Maximum Expiration**: 10,000 blocks (~69.4 days on Stacks)

## Error Codes

| Code | Description |
|------|-------------|
| `u100` | Owner-only function called by non-owner |
| `u101` | Swap not found |
| `u102` | Unauthorized caller |
| `u103` | Resource already exists |
| `u104` | Invalid parameters |
| `u105` | Swap has expired |
| `u106` | Swap has not expired yet |
| `u107` | Swap already completed |
| `u108` | Swap already canceled |
| `u109` | Invalid Bitcoin transaction |
| `u110` | Insufficient Bitcoin amount |
| `u111` | Invalid Bitcoin receiver |

## Project Structure

```
cross-chain-swapper/
├── contracts/
│   ├── cross-chain-swapper.clar     # Main contract
│   └── sip-010-trait.clar           # SIP-010 trait definition
├── tests/
│   └── cross-chain-swapper_test.ts  # Contract tests
└── README.md                        # This file
```

## Development Setup

### Prerequisites

- [Clarinet](https://github.com/hirosystems/clarinet) for local development and testing
- [Stacks CLI](https://docs.stacks.co/stacks-blockchain/cli) for deployment to testnet/mainnet

### Getting Started

1. Clone the repository
```bash
git clone https://github.com/idumachika/cross-chain-swapper.git
cd cross-chain-swapper
```

2. Install dependencies
```bash
npm install
```

3. Run Clarity tests
```bash
clarinet test
```

## Contract Deployment

### Testnet Deployment

```bash
clarinet publish --testnet
```

### Mainnet Deployment

```bash
clarinet publish --mainnet
```

## Usage Example

### Creating a Swap (STX to BTC)

```clarity
(contract-call? 'ST1PQHQKV0RJXZFY1DGX8MNSNYVE3VGZJSRTPGZGM.cross-chain-swapper create-stx-to-btc-swap
  u10000000 ;; 10 STX
  u500000   ;; 0.005 BTC (500,000 satoshis)
  0x76a914bc3b654b9633a68cd34d6e650dffd8f19319e2208ac ;; P2PKH script
  0x76a914a6b8fe09c950fc7cefd7672cd581a262d952ecf088ac ;; Expected Bitcoin sender
  u200      ;; Expire after 200 blocks
)
```

### Completing a Swap

```clarity
(contract-call? 'ST1PQHQKV0RJXZFY1DGX8MNSNYVE3VGZJSRTPGZGM.cross-chain-swapper complete-swap
  u5        ;; Swap ID
  0x0200... ;; Bitcoin transaction bytes
  u0        ;; Transaction output index
  {         ;; Merkle proof
    tx-index: u12,
    hashes: (list 0x1234...),
    tree-depth: u3
  }
  0x0200... ;; Bitcoin block header
  u700000   ;; Bitcoin block height
)
```

## Security Considerations

- The contract relies on the security of Stacks blockchain's Bitcoin transaction verification
- Users should carefully verify transaction parameters before initiating swaps
- The contract implements timeouts to ensure funds aren't locked indefinitely
- Administrative functions are protected by owner-only access controls

## Implementation Notes

The current implementation includes:
- Full support for STX and SIP-010 tokens
- Simplified Bitcoin transaction verification as a placeholder
- Protocol fee collection mechanism

In a production environment:
- The Bitcoin transaction verification would be implemented using Clarity's native Bitcoin verification functions
- Additional security measures and validations would be added
- More sophisticated fee structures might be implemented

## Fixing the Contract Error

To fix the error mentioned in the issue, you need to:

1. Create a separate contract file called `sip-010-trait.clar` with the trait definition
2. Modify the main contract to import the trait:

```clarity
;; In cross-chain-swapper.clar
(use-trait sip-010-token .sip-010-trait.sip-010-trait)
```

Or alternatively, keep it in the same file but change the order of definitions:

```clarity
;; First define the trait
(define-trait sip-010-trait
  (
    ;; Transfer from the caller to a new principal
    (transfer (uint principal principal (optional (buff 34))) (response bool uint))
    ;; ... other methods ...
  )
)

;; Then use the trait
(use-trait sip-010-token sip-010-trait)
```

## License

[MIT License](LICENSE)

## Contributi