# Ownable Smart Contract

An access-control smart contract implementing a **two-step ownership transfer** pattern in Solidity `0.8.34`. The contract establishes a single owner at deployment and provides a secure mechanism for transferring ownership to a new account, as well as renouncing ownership entirely.

## Contract Overview

**`Ownable`** provides role-based access control with a single privileged owner. It uses a two-step transfer pattern (`transferOwnership` → `acceptOwnership`) to prevent accidental or malicious ownership transfers to wrong addresses. Custom errors are used for gas-efficient reverts.

### State

| Variable | Visibility | Description |
|---|---|---|
| `_owner` | `private` | Current contract owner |
| `_pendingOwner` | `private` | Address that has been proposed but not yet accepted ownership |

### Events

| Event | Description |
|---|---|
| `OwnershipTransferred(previousOwner, newOwner)` | Emitted when ownership is successfully transferred or renounced |
| `OwnershipTransferStarted(previousOwner, newOwner)` | Emitted when a new pending owner is proposed |

### Custom Errors

| Error | Description |
|---|---|
| `Ownable__UnauthorizedAccount(address)` | Reverted when a non-owner calls an `onlyOwner` function |
| `Ownable__OwnerCannotBeBurnAddress()` | Reverted when `transferOwnership` is called with the zero address |
| `Ownable__NotPendingOwner()` | Reverted when `acceptOwnership` is called by an address other than the pending owner |

### Functions

| Function | Visibility | Description |
|---|---|---|
| `constructor()` | — | Sets the deployer (`msg.sender`) as the initial owner |
| `owner()` | `view` | Returns the current owner's address |
| `pendingOwner()` | `view` | Returns the pending owner's address (if a transfer is in progress) |
| `transferOwnership(address newOwner)` | `onlyOwner` | Proposes a new owner; starts the two-step transfer process |
| `acceptOwnership()` | `public` | Called by the pending owner to complete the transfer |
| `renounceOwnership()` | `onlyOwner` | Renounces ownership, leaving the contract ownerless |

### Two-Step Transfer Flow

```
Current Owner                     New Owner
     |                                |
     |--- transferOwnership(addr) --->|
     |    (emits TransferStarted)     |
     |                                |
     |<--- acceptOwnership() ---------|
     |    (emits TransferTransferred) |
```

This pattern prevents accidental loss of ownership — if the wrong address is passed to `transferOwnership`, the current owner can call `transferOwnership` again to overwrite the pending owner before `acceptOwnership` is called.

## Getting Started

### Prerequisites
- [Remix IDE](https://remix.ethereum.org/) (no local setup required) or a local toolchain like [Foundry](https://book.getfoundry.sh/) / [Hardhat](https://hardhat.org/)

### Compile
1. Open `contracts/Ownable.sol` in Remix IDE.
2. In the **Solidity Compiler** plugin, select compiler version `0.8.34` (or compatible).
3. Click **Compile Ownable.sol**.

### Deploy
1. Navigate to the **Deploy & Run Transactions** plugin.
2. Select your desired environment (Remix VM, Injected Provider for MetaMask, etc.).
3. Click **Deploy** — the deployer automatically becomes the initial owner.

### Interact
- Call `owner()` to check the current owner.
- Call `transferOwnership(newOwnerAddress)` as the current owner to propose a transfer.
- Switch to the new owner account and call `acceptOwnership()` to complete the transfer.
- Call `renounceOwnership()` as the current owner to make the contract ownerless.

## Usage as a Base Contract

`Ownable` is designed to be inherited by other contracts to add access control:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity 0.8.34;

import "./Ownable.sol";

contract MyContract is Ownable {
    function privilegedAction() external onlyOwner {
        // Only the owner can call this
    }
}
```

## Project Structure

```
.
├── contracts/
│   └── Ownable.sol          # Ownable access-control contract
├── artifacts/
│   ├── Ownable.json          # Compilation artifact
│   └── Ownable_metadata.json # Contract metadata
├── remix.config.json         # Remix IDE workspace configuration
└── README.md
```

## Security Considerations

- **Two-step transfer** mitigates the risk of transferring ownership to a wrong or typo'd address.
- `transferOwnership` rejects the zero address to prevent accidental lockout.
- `renounceOwnership` permanently removes the owner — there is no recovery path. Use with caution.
- Consider adding a timelock or multi-sig as the owner for production deployments.

## License

This project is licensed under the **MIT License** (SPDX-License-Identifier: MIT).

## Resources

- [Remix IDE Documentation](https://remix-ide.readthedocs.io/)
- [Solidity Language Documentation](https://docs.soliditylang.org/)
- [OpenZeppelin Ownable (reference implementation)](https://docs.openzeppelin.com/contracts/5.x/access-control#ownership)
