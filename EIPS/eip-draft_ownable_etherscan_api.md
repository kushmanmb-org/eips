---
title: Ownable Contract Etherscan API Interface Standard
description: A standard interface for ownable smart contracts discoverable via the Etherscan API
author: KUSHMANMB (@KUSHMANMB)
discussions-to: https://ethereum-magicians.org/t/eip-draft-ownable-etherscan-api-interface-standard
status: Draft
type: Standards Track
category: ERC
created: 2026-03-16
requires: 173
---

## Abstract

This EIP defines a standard interface for ownable smart contracts that are discoverable and queryable via the Etherscan API. It extends [EIP-173](./eip-173.md) by specifying how ownership information should be exposed in a way that is compatible with the Etherscan API's contract verification and metadata endpoints, enabling consistent discovery and interaction with contract owners.

## Motivation

The Etherscan API is widely used to retrieve ownership and metadata information about deployed smart contracts. Currently, there is no standard specifying how ownable contracts should expose their data to be uniformly accessible via the Etherscan API. Different implementations handle ownership exposure differently, leading to fragmentation across tooling and developer experience.

By standardizing a minimal ownable interface that pairs with Etherscan's existing contract retrieval mechanisms, developers can rely on a consistent pattern when building tools that need to query and interact with contract owners on-chain.

## Specification

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119) and [RFC 8174](https://www.rfc-editor.org/rfc/rfc8174).

### Interface

A contract compliant with this EIP MUST implement the following interface, which extends [EIP-173](./eip-173.md):

```solidity
// SPDX-License-Identifier: CC0-1.0
pragma solidity >=0.8.25 <0.9.0;

/// @title IEIP_OwnableEtherscanAPI
/// @notice Interface for ownable contracts discoverable via the Etherscan API
interface IEIP_OwnableEtherscanAPI {
    /// @notice Emitted when ownership is transferred
    /// @param previousOwner The address of the previous owner
    /// @param newOwner The address of the new owner
    event OwnershipTransferred(address indexed previousOwner, address indexed newOwner);

    /// @notice Returns the address of the current owner
    /// @return The address of the current owner
    function owner() external view returns (address);

    /// @notice Transfers ownership of the contract to a new account
    /// @param newOwner The address of the new owner
    function transferOwnership(address newOwner) external;

    /// @notice Renounces ownership of the contract, leaving it without an owner
    function renounceOwnership() external;
}
```

### Required Behaviour

- The `owner()` function MUST return the current owner address.
- `transferOwnership(address newOwner)` MUST revert if called by any account other than the current owner.
- `transferOwnership(address newOwner)` MUST revert if `newOwner` is the zero address.
- `renounceOwnership()` MUST revert if called by any account other than the current owner.
- An `OwnershipTransferred` event MUST be emitted whenever ownership changes, including during contract initialization.

### Etherscan API Compatibility

Contracts implementing this EIP SHOULD be verified on Etherscan so that the `owner()` function is accessible via the Etherscan API read contract interface. This allows off-chain tools to query ownership without requiring an ABI.

The recommended Etherscan API call to retrieve the owner of a compliant contract is:

```
GET https://api.etherscan.io/api
   ?module=contract
   &action=getsourcecode
   &address=<CONTRACT_ADDRESS>
   &apikey=<APIKEY>
```

## Rationale

OpenZeppelin's `Ownable` contract is the de-facto standard implementation for ownership management in Solidity. This EIP formalizes the interface already in widespread use and adds the requirement for Etherscan verification to enable consistent API discoverability.

The pragma range `>=0.8.25 <0.9.0` is chosen to align with modern Solidity releases that include important safety improvements while maintaining a well-defined, stable compiler version range.

## Backwards Compatibility

This EIP is fully backwards compatible with [EIP-173](./eip-173.md) and with contracts that already use OpenZeppelin's `Ownable` module. Existing ownable contracts that are already verified on Etherscan are compliant without any changes.

## Reference Implementation

```solidity
// SPDX-License-Identifier: CC0-1.0
pragma solidity >=0.8.25 <0.9.0;

import { Ownable } from "@openzeppelin/contracts/access/Ownable.sol";

/// @title OwnableEtherscanAPI
/// @notice Reference implementation of the Ownable Etherscan API Interface Standard
/// @dev Deployed reference: 0x6fb9e80dDd0f5DC99D7cB38b07e8b298A57bF253
contract OwnableEtherscanAPI is Ownable {
    constructor(address initialOwner) Ownable(initialOwner) {}
}
```

## Security Considerations

- Ownership confers significant privileges. Owners of compliant contracts SHOULD use multi-signature wallets or time-locks where appropriate.
- The `renounceOwnership()` function permanently removes the ability to call owner-only functions; callers MUST understand this is irreversible.
- Etherscan API keys SHOULD be kept confidential and rotated regularly.
- Off-chain tools querying ownership via the Etherscan API SHOULD also verify on-chain state, as Etherscan data may be delayed or cached.

## Copyright

Copyright and related rights waived via [CC0](../LICENSE.md).
