---
VIP: pending
Title: Support Ethereum-format transaction signing
Description: outlines an upgrade to the core validation of transactions to support transactions signed by Ethereum-based client.
Author: Oliver Chalk <oliver.chalk@proxima.capital>, Ian McLeod <ian.mcleod@proxima.capital>
Discussions: <URL>
Category:  Core
Status: Draft
CreatedAt: 2023-07-27

---

# Overview

The below VIP-215 proposal outlines an upgrade to the core validation of transactions on the VeChainThor blockchain, in order to support transactions signed by Ethereum-based client systems. (such as Ethereum-supporting wallets and other JSON-RPC clients)

  
# Rationale

<!--

  The rationale fleshes out the specification by describing what motivated the design and why particular design decisions were made. It should describe alternate designs that were considered and related work, e.g. how the feature is supported in other languages.
  The current placeholder is acceptable for a draft.
  TODO: Remove this comment before submitting
-->
  
# Motivation

VeChain, from its outset, has offered a unique evolutionary path via features such as multi-clause transactions and alternate gas models; however, the possibility of native interoperability with the broader Ethereum transaction ecosystem presents a significant additional upside in end-user participation and engagement.

We would like to propose a modification to VeChain that would allow an end-user to use Ethereum tooling, such as Ethereum compatible wallets like MetaMask, to sign VeChain transactions and thus reduce friction for VeChain's reach into the wider crypto user-base.
  
# Specification
  
## Concept

Support for accepting Ethereum-format transactions will be accomplished by:
  1. Representing an Ethereum-format signed transaction within a VeChain transaction;
     (In doing so, we ensure that all required data to verify the signing hash is included in the VeChain transaction, and hence atomically within the same block.)
  2. Ensure transactions are distinguishable as standard or Ethereum-format transactions; 
  3. Implement an additional alternate Ethereum-compatible transaction hash validation mechanism, in order to verify the externally signed alternate transaction format for the Ethereum-format transactions.
  4. A "gateway" component will receive the Ethereum-format transactions and 'wrap' them in VeChain transactions by applying the field mapping described below. This gateway could be internal (API modification/extension) or external (gateway services) to the Node depending on implementation requirements and/or constraints.

### Transaction Types

The Ethereum transaction format type `0x01`, i.e. transactions introduced with the transaction envelope [EIP-2718](https://eips.Ethereum.org/EIPS/eip-2718)) will be adopted, which reflects the cleanest subset of VeChain transaction features.

Specifically, this Ethereum-format transaction supports the inclusion of a `chainID` in the transaction, which will be mapped through to the `ChainTag` field in VeChain.

Note that it also includes an 'access list' for the transaction that will not be supported. As such, VeChain nodes (and the gateway component) will simply reject transactions that include the `accessList` field as invalid. Due to the need to reconstruct the Ethereum transactions for signature validation, it is not possible to simply silently drop the `accessList` at the gateway level.

Supporting the type 0x01 Ethereum-format transaction provides the most significant step-change benefit, and further Ethereum transaction types could be considered in future VIPs when relevant.

### Feature Bitset

The `Reserved` transaction field, with `Feature` bitset value `0x02` will be used to indicate the transaction is in Ethereum-format, and thus requires Ethereum-format hash validation.

This is similar to VIP-191, which identifies transactions as supporting an alternate gas-payer when the `Feature` bitset value `0x01` is set.

## Transaction Mapping

The primary proposed mapping includes zero additional fields, using the VeChain transaction signature data to encode both the original Ethereum-format Nonce and signed data.

Mapping fields are as follows:

| VeChain Field | Type     | Ethereum Field(s) | Notes                                                           |
| :------------ | :------- | :---------------  | :-------------------------------------------------------------- |
| ChainTag      | byte     | chainId           | Straight mapping from chainId.                                  |
| BlockRef      | uint64   | *Derived*         | See [block & expiry](#blockref-and-expiration) section below.   |
| Expiration    | uint32   | *Derived*         | No equivalent will not be set.                                  |
| Clauses       | []clause | To, Value, Data   | See clause sub-structure below.                                 |
| GasPriceCoef  | uint8    | gasPrice          | Map from `gasPrice`; see [gasPrice mapping](#Gas-Price-mapping) |
| Gas           | uint64   | Gas               | Straight `Gas` mapping.                                         |
| DependsOn     | ptr      | *Derived*         | See [txn dependency](#Transaction-dependency) section below.    |
| Nonce         | uint64   | Nonce             | See section on [Nonces](#Nonces) below.                         |
| Reserved      | reserved | Set by gateway    | Set to indicate the transaction is in Ethereum-format.          |
| Signature     | []byte   | Signature fields  | Signature data is hashed based on Ethereum format encoding.     |

Clause sub-structure (only a single clause will be used.)

| VeChain Field | Type     | Ethereum Field(s) | Notes                              |
| :------------ | :------- | :---------------  | :--------------------------------- |
| To            | address  | To                | Straight `To` address mapping.     |
| Value         | big-int  | Value             | Straight `Value` mapping.          |
| Data          | []byte   | Data              | Straight `Data` mapping.           |

In order to maintain user-experience of Ethereum-based systems and zero change to the existing VeChain Transaction structure, some fields map into VeCHain fields with restrictions, or special behavior for Ethereum-format transactions. These special cases are described in the following sections.

### `reserved` field update details

Add index 1, isEthereumFormat to the existing bitset.

| Index | Attribute          | Type    | Optional | Added in this change |
| ----- | ------------------ | ------- | -------- | -------------------- |
|    0  | `isGasPaid`        | boolean | yes      | pre-existing         |
|    1  | `isEthereumFormat` | boolean | yes      | new                  |

Adding `isEthereumFormat` to the `reserved` field bitset allows us to signal to the VM that this transaction should be validated as an Ethereum transaction, following the RLP encoding rules described above.

### BlockRef and Expiration

The VeChain fields `BlockRef` and `Expiration` have no equivalent in Ethereum; as such Ethereum-format transactions will take default values - values to be confirmed.

### Nonces

VeChain's transaction uniqueness system, and specifically the use of the `Nonce` field is not fully compatible with the Ethereum-format Nonce and transaction uniqueness; since the VeChain Nonce is completely user-defined and is not required to be in any given order or format, unlike the Ethereum-format Nonce. The Ethereum-format Nonce however, is effectively a sub-set of its possible values so we are able to achieve compatibility via a series of small changes.

In this proposal, we will map the Ethereum-format Nonce into the VeChain Nonce field, but add additional constraints and operations to the transaction processing, in order to respect a transactional behavior that is consistent with an Ethereum-format signing client's expectation, achieving compatible operation.

#### Nonce based transaction rejection

VeChain guarantees protection against invalid replay of transactions via the rejection of transactions having the same TxId as an already-processed transaction, whereas Ethereum enforces strict sequencing of transaction via an incrementing integer Nonce.

In order to support Ethereum-format transactions within VeChain, this proposal introduces a validation to confirm the valid Nonce for the sender account has been set in the transaction. Any other Nonce value will result in the rejection of the transaction.

There is only a single valid Nonce at any point in time; it initializes with the value zero (0) and increments by 1 with every Ethereum-format transaction successfully processed.

This additional validation and rejection of invalid Nonces handles the Ethereum-style use of the Nonce, for Ethereum-format transactions.

#### Transaction Dependency

In addition to rejection of transactions with invalid Nonces, in order to respect the Ethereum approach of using the integer sequence of the Nonce to enforce transaction ordering, an extension to the transaction dependency implementation in VeChain is required. 

Normally, achieved via the `DependsOn` field of the transactions in VeChain, this proposal includes a change to derive an equivalent dependency relationship to the `DependsOn` field, based on the account and Nonce, for Ethereum-format transactions.

To achieve this, conceptually a map of pending Ethereum-format transactions will be maintained, mapping a hash of account & Nonce to the transaction's TxId in order to create a dependency chain of the unconfirmed Ethereum-format transactions, for a given account. 

On processing an Ethereum-format transaction, the transaction's dependent transaction will be determined by subtracting 1 from the Nonce, and looking up the hash of the account and this prior Nonce, to retrieve the prior transaction's TxId. 

If the TxId is present in the map, it will be used equivalently to the `DependsOn` field, as if this TxId was originally set in the `DependsOn` field at the time of transaction creation. 

In implementation, this may be achieved by actually setting the the `DependsOn` field, noting that this is theoretically a valid optimization that does not violate the transaction integrity, as the field itself would never be used in signature validation for Ethereum-format transactions.

#### Optimally Managing the 'Next Valid Nonce'

In order to optimally maintain the net valid Nonce that an Ethereum-format transaction may use, a new field `nextValidEthereumNonce` of type uint64 is proposed to be added to the `cachedObject` data of the `State`, keyed off the VeChain accounts (type `thor.Address`).

This new field would be initialized to zero and represent the Nonce that any Ethereum-format transaction being processed must contain in order for it to be deemed valid. On successful processing of the transaction, the field will be incremented by 1, ready for the next transaction.

#### Exposing the Next Valid Nonce to Transaction Signers

In order for Ethereum-format transaction signers to correctly create and sign a transaction, the `nextValidEthereumNonce` value for a given account must be made accessible via query from the external-facing API.

### Gas Price mapping

In the field `GasPriceCoef`, VeChain supports a range of 0-255 for specifying the gas-price-coefficient, which is incompatible with the Ethereum equivalent `gasPrice` field (of type bignum). In order to maintain a deterministic user experience consistent with the intended use case, the range of `gasPrice` values will be mapped into the range of `GasPriceCoef` values where there is a conversion resulting in a value within range of the 8-bit integer field, without rounding.

A transaction including any unmappable values will result in a transaction rejection.

The following formula will be used:

    `GasPriceCoef` = 256 * ( `gasPrice` - 1 ) - 1

Example conversions:

| `gasPrice` | `GasPriceCoef` | Status      | Description                            |
| :--------: | :------------: | :---------: | :------------------------------------- |
| 1          | 0              | Accepted    | Min GasPriceCoef                       |
| 1.25       | 63             | Accepted    | Valid                                  |
| 1.5        | 127            | Accepted    | Valid                                  | 
| 1.75       | 191            | Accepted    | Valid                                  |
| 2          | 255            | Accepted    | Max GasPriceCoef                       |
| 0.5        | < 0            | Rejected    | Exceeds range of GasPriceCoef          |
| 3          | > 255          | Rejected    | Exceeds range of GasPriceCoef          |
| 1.3        | 75.8           | Rejected    | Fractional result / requires rounding. |

## Transaction Validation

During transaction processing, the system will switch behavior depending on the value of the `isEthereumFormat` bit flag in the `reserve` field. If the field is set, alternate validation will occur.

The alternate validation will consist of emulating the Ethereum signature validation by constructing an Ethereum-format representation of the transaction, via reversing the transaction mapping described above, and verifying the signature hash from that representation.

## 'Native' API support: JSON / RPC gateway

Ultimately, for seamless integration of existing Ethereum based tools and services, support of a compatible JSON/RPC based API will be required. 

This is explicitly not part of this proposal, with the intention that such integration can be achieved at least initially, via separate gateway services that provide external access via JSON/RPC and VeChain-facing access via the existing VeChain REST API. 

## Threat Assessment Discussion

This section discusses potential threats introduced by the change.

### Extension Analysis

Based on the standing assumption that both the current VeChain transaction validation method is safe, and the Ethereum transaction signing method is safe; by extension this change is assumed equivalently safe and not introducing any vulnerabilities as it effectively introduces only a switchable RLP-encoding between the two validations formats.

Switching between validation methods is controlled by setting the reserve bit, implicitly set when creating a transaction in Ethereum-format, but not included in the transaction hash.

### Independent verification that the transaction is in Ethereum-format

Note that in the event that the Ethereum-format transaction is not flagged by the correct reserve bit being set, the signature has will be invalid and the transaction should be rejected as any invalid transaction would be.

Similarly, the same invalid state would occur if a transaction incorrectly sets the reserve bit, and the transaction is not in Ethereum-format.

# Test Cases
  
<!--
  This section is optional for non-Core VIPs.

  The Test Cases section should include expected input/output pairs, but may include a succinct set of executable tests. It should not include project build files. No new requirements may be be introduced here (meaning an implementation following only the Specification section should pass all tests here.)
  If the test suite is too large to reasonably be included inline, then consider adding it as one or more files in `../assets/vip-####/`. External links will not be allowed

  TODO: Remove this comment before submitting
-->
  
# Implementation
  
<!--
  This section is optional.

  The Reference Implementation section should include a minimal implementation that assists in understanding or implementing this specification. It should not include project build files. The reference implementation is not a replacement for the Specification section, and the proposal should still be understandable without it.
  If the reference implementation is too large to reasonably be included inline, then consider adding it as one or more files in `../assets/vip-####/`. External links will not be allowed.

  TODO: Remove this comment before submitting
-->
  
# Security Considerations

<!--
  All VIPs must contain a section that discusses the security implications/considerations relevant to the proposed change. Include information that might be important for security discussions, surfaces risks and can be used throughout the life cycle of the proposal. For example, include security-relevant design decisions, concerns, important discussions, implementation-specific guidance and pitfalls, an outline of threats and risks and how they are being addressed. VIP submissions missing the "Security Considerations" section will be rejected. An VIP cannot proceed to status "Final" without a Security Considerations discussion deemed sufficient by the reviewers.

  The current placeholder is acceptable for a draft.

  TODO: Remove this comment before submitting
-->

# References