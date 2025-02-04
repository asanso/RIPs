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

As of `FORK_TIMESTAMP` in the integrated EVM chain, add precompiled contract `P256VERIFY` for signature verifications in the “secp256r1” elliptic curve at address `PRECOMPILED_ADDRESS` in `0x100` (indicates 0x0000000000000000000000000000000000000100).

### Elliptic Curve Information

“secp256r1” is a specific elliptic curve, also known as “P-256” and “prime256v1” curves. The curve is defined with the following equation and domain parameters:

```
# curve: short weierstrass form
y^2 ≡ x^3 + ax + b

# p: curve prime field modulus
0xffffffff00000001000000000000000000000000ffffffffffffffffffffffff

# a: elliptic curve short weierstrass first coefficient
0xffffffff00000001000000000000000000000000fffffffffffffffffffffffc

# b: elliptic curve short weierstrass second coefficient
0x5ac635d8aa3a93e7b3ebbd55769886bc651d06b0cc53b0f63bce3c3e27d2604b

# G: base point of the subgroup
(0x6b17d1f2e12c4247f8bce6e563a440f277037d812deb33a0f4a13945d898c296,
 0x4fe342e2fe1a7f9b8ee7eb4a7c0f9e162bce33576b315ececbb6406837bf51f5)

# n: subgroup order (number of points)
0xffffffff00000000ffffffffffffffffbce6faada7179e84f3b9cac2fc632551

# h: cofactor of the subgroup
0x1

```

### Elliptic Curve Signature Verification Steps

The signature verifying algorithm takes the signed message hash, the signature components provided by the “secp256r1” curve algorithm, and the public key derived from the signer private key. The verification can be done with the following steps:

```
# h (message hash)
# pubKey = (public key of the signer private key)

# Calculate the modular inverse of the signature proof:
s1 = s^(−1) (mod n)

# Recover the random point used during the signing:
R' = (h * s1) * G + (r * s1) * pubKey

# Take from R' its x-coordinate:
r' = R'.x

# Calculate the signature validation result by comparing whether:
r' == r

```

### Required Checks in Verification

The following requirements **MUST** be checked by the precompiled contract to verify signature components are valid:

- Verify that the `r` and `s` values are in `(0, n)` (exclusive) where `n` is the order of the subgroup.
- Verify that the point formed by `(x, y)` is on the curve and that both `x` and `y` are in `[0, p)` (inclusive 0, exclusive p) where `p` is the prime field modulus. Note that many implementations use `(0, 0)` as the reference point at infinity, which is not on the curve and should therefore be rejected.

### Precompiled Contract Specification

The `P256VERIFY` precompiled contract is proposed with the following input and outputs, which are big-endian values:

- **Input data:** 160 bytes of data including:
    - 32 bytes of the signed data `hash`
    - 32 bytes of the `r` component of the signature
    - 32 bytes of the `s` component of the signature
    - 32 bytes of the `x` coordinate of the public key
    - 32 bytes of the `y` coordinate of the public key

- **Output data:**
    - If the signature verification process succeeds, it returns 1 in 32 bytes format.
    - If the signature verification process fails, it does not return any output data.

### Precompiled Contract Gas Usage

The use of signature verification cost by `P256VERIFY` is `3450` gas. Following reasons and calculations are provided in the [Rationale](#rationale) and [Test Cases](#test-cases) sections.

## Rationale

“secp256r1” ECDSA signatures consist of `v`, `r`, and `s` components. While the `v` value makes it possible to recover the public key of the signer, most signers do not generate the `v` component of the signature since `r` and `s` are sufficient for verification. In order to provide an exact and more compatible implementation, verification is preferred over recovery for the precompile.

Existing P256 implementations verify `(x, y, r, s)` directly. We've chosen to match this style here, encoding each argument for the EVM as a `uint256`.

This is different from the `ecrecover` precompiled address specification. The advantage is that it 1. follows the NIST specification (as defined in NIST FIPS 186-5 Digital Signature Standard (DSS)), 2. matches the rest of the (large) P256 ecosystem, and most importantly 3. allows execution clients to use existing well-vetted verifier implementations and test vectors.

Another important difference is that the NIST FIPS 186-5 specification does not include a malleability check. We've matched that here in order to maximize compatibility with the large existing NIST P-256 ecosystem.

Wrapper libraries **SHOULD** add a malleability check by default, with functions wrapping the raw precompile call (exact NIST FIPS 186-5 spec, without malleability check) clearly identified. For example, `P256.verifySignature` and `P256.verifySignatureWithoutMalleabilityCheck`. Adding the malleability check is straightforward and costs minimal gas.

The `PRECOMPILED_ADDRESS` is chosen as `0x100` as `P256VERIFY` is the first precompiled contract presented as an RIP, and the address is the first available address in the precompiled address set that is reserved for the RIP precompiles.

The gas cost is proposed by comparing the performance of the `P256VERIFY` and the `ECRECOVER` precompiled contract which is implemented in the EVM at `0x01` address. It is seen that “secp256r1” signature verification is ~15% slower (elaborated in [test cases](#test-cases)) than “secp256k1” signature recovery, so `3450` gas is proposed by comparison which causes similar “mgas/op” values in both precompiled contracts.

## Backwards Compatibility

No backward compatibility issues found as the precompiled contract will be added to `PRECOMPILED_ADDRESS` at the next available address in the precompiled address set.

## Test Cases

Functional tests are applied for multiple cases in the [reference implementation](#reference-implementation) of `P256VERIFY` precompiled contract and they succeed. Benchmark tests are also applied for both `P256VERIFY` and `ECRECOVER` with some pre-calculated data and signatures in the “go-ethereum”s precompile testing structure to propose a meaningful gas cost for the “secp256r1” signature verifications by the precompiled contract implemented in the [reference implementation](#reference-implementation). The benchmark test results by example data in the assets can be checked:

- [P256Verify Benchmark Test Results](../assets/rip-7212/p256Verify_benchmark_test)
- [Ecrecover Benchmark Test Results](../assets/rip-7212/ecrecover_benchmark_test)

```
# results of geth benchmark tests of
# ECRECOVER and P256VERIFY (reference implementation)
# by benchstat tool

goos: darwin
goarch: arm64
pkg: github.com/ethereum/go-ethereum/core/vm
                                            │ compare_p256Verify │ compare_ecrecover  │
                                            │       sec/op       │   sec/op           │
PrecompiledP256Verify/p256Verify-Gas=3450-8          57.75µ ± 1%
PrecompiledEcrecover/-Gas=3000-8                                   50.48µ ± 1%
geomean                                              57.75µ        50.48µ

                                            │ compare_p256Verify │ compare_ecrecover  │
                                            │       gas/op       │   gas/op           │
PrecompiledP256Verify/p256Verify-Gas=3450-8          3.450k ± 0%
PrecompiledEcrecover/-Gas=3000-8                                   3.000k ± 0%
geomean                                              3.450k        3.000k

                                            │ compare_p256Verify │ compare_ecrecover │
                                            │       mgas/s       │   mgas/s          │
PrecompiledP256Verify/p256Verify-Gas=3450-8           59.73 ± 1%
PrecompiledEcrecover/-Gas=3000-8                                   59.42 ± 1%
geomean                                               59.73        59.42

                                            │ compare_p256Verify │ compare_ecrecover │
                                            │        B/op        │    B/op           │
PrecompiledP256Verify/p256Verify-Gas=3450-8         1.523Ki ± 0%
PrecompiledEcrecover/-Gas=3000-8                                   800.0 ± 0%
geomean                                             1.523Ki        800.0

                                            │ compare_p256Verify │ compare_ecrecover │
                                            │     allocs/op      │ allocs/op         │
PrecompiledP256Verify/p256Verify-Gas=3450-8           33.00 ± 0%
PrecompiledEcrecover/-Gas=3000-8                                   7.000 ± 0%
geomean                                               33.00        7.000

```

## Reference Implementation

Implementation of the `P256VERIFY` precompiled contract is applied to go-ethereum client to create a reference. Also, a “secp256r1” package has already been included in the Besu Native library which is used by Besu client. Other client implementations are in the future roadmap.

## Security Considerations

The changes are not directly affecting the protocol security, it is related with the applications using `P256VERIFY` for the signature verifications. The “secp256r1” curve has been using in many other protocols and services and there is not any security issues in the past.


## Copyright

Copyright and related rights waived via [CC0](../LICENSE.md).
