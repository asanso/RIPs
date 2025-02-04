---
rip: ????
title: Precompile for Falcon support
description: Proposal to add a precompiled contract that performs signature verifications using the Falcon signature scheme.
author: Antonio Sanso (@asanso), Marius Van Der Wijden, Kevaundray Wedderburn, Zhenfei Zhang
discussions-to: ...
status: Draft
type: Standards Track
category: Core
created: ????-??-??
---

## Abstract
This proposal creates a precompiled contract that performs signature verifications using the Falcon signature scheme by given parameters of the message hash, signature, and public key. This allows any EVM chain—principally Ethereum rollups—to integrate this precompiled contract easily.

## Motivation
Falcon is a post-quantum digital signature scheme based on lattice-based cryptography, providing resistance against attacks from both classical and quantum computers. Unlike traditional elliptic curve cryptography, Falcon ensures long-term security in a future where quantum computing could break existing cryptographic standards.

Adding a precompiled contract for Falcon signature verification allows smart contracts to leverage quantum-resistant authentication mechanisms. This ensures Ethereum rollups and other EVM-based chains remain secure as quantum advancements progress. The introduction of this precompiled contract would enable crucial improvements in account abstraction, allowing efficient and flexible key management while maintaining security against quantum threats.

Many emerging cryptographic standards, including NIST’s post-quantum cryptography (PQC) initiative, highlight the importance of transitioning to quantum-resistant schemes. By integrating Falcon at the EVM level, this proposal future-proofs blockchain security, ensuring that decentralized applications and smart contracts remain resistant to quantum attacks.

This precompiled contract enables secure signing mechanisms that can be utilized for key management and transaction verification. Since Falcon is designed for high efficiency and small signature sizes, it is well-suited for blockchain environments, ensuring that quantum security does not come at the cost of excessive computational overhead. Implementing Falcon at the precompiled contract level ensures that Ethereum and rollups remain at the forefront of cryptographic security in the post-quantum era.

## Specification

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in RFC 2119 and RFC 8174.

### Falcon Information

### Falcon Verification Steps


### Required Checks in Verification

### Precompiled Contract Specification


### Precompiled Contract Gas Usage

## Rationale

## Backwards Compatibility



## Test Cases

## Reference Implementation



## Security Considerations



## Copyright

Copyright and related rights waived via [CC0](../LICENSE.md).
