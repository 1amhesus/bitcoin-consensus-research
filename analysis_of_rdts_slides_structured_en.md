
# RDTS (BIP-110): What Is the Essence of the Bitcoin Community Governance Debate?

> From the definition of a block chain and `OP_RETURN` to arbitrary-data input analysis and the BIP-110 code review

**Independent Bitcoin Research Note** · [@1amhesus](https://github.com/1amhesus)  
Structured English edition of the 8th Bitcoin Research Team @ Orakle final presentation

> **Research status:** This note distinguishes observations supported by code review from hypotheses that still require controlled regtest validation.

## Table of Contents

1.  [Introduction: Background and Motivation](#introduction-background-and-motivation)
2.  [Phase 0: BIP-110 Motivation, Restrictions, Rationale, and History](#phase-0-bip-110-motivation-restrictions-rationale-and-history)
3.  [Phase 1: Where Can Arbitrary Data Enter the Bitcoin Block Chain?](#phase-1-where-can-arbitrary-data-enter-the-bitcoin-block-chain)
4.  [Phase 2: Why Restricting Only OP_RETURN Does Not Solve the Problem](#phase-2-why-restricting-only-op_return-does-not-solve-the-problem)
5.  [Phase 3: A Critical Reading of the BIP-110 Implementation](#phase-3-a-critical-reading-of-the-bip-110-implementation)
6.  [Conclusion and Open Questions](#conclusion-and-open-questions)

## Introduction: Background and Motivation

### What Is Happening with Bitcoin?

Bitcoin is increasingly being discussed not only as a payment network but also as a strategic reserve asset and a foundation for native financial infrastructure.

- Lightning Network payment experiments are being observed and measured at the user level.
- Bitcoin-native finance projects such as RGB and Taro attempt to reduce reliance on global on-chain publication by using technical and data-based verification models.
- Institutional demand has expanded through spot Bitcoin ETFs.
- Several countries have discussed or reported public-sector Bitcoin holdings, reserves, or mining strategies.
- The United States has also discussed Bitcoin within the language of strategic national assets.

The larger research context is therefore not merely whether Bitcoin can store or relay data. It is how a monetary protocol, increasingly connected to national and market infrastructure, chooses and propagates its consensus rules.

### Cypherpunk Is Dead...?

### A Personal Reflection on the Meme

The recent phrase “Cypherpunk is dead?” and the controversy surrounding BIP-110 led me to revisit a question that had remained with me for some time: what is cypherpunk?

Is it a fixed doctrine with a single correct interpretation? Is it the freedom to develop and use privacy technologies? In today's Bitcoin ecosystem, should the cypherpunk spirit mean allowing users to write or transmit any data they choose? Or should it mean preserving a monetary protocol that users can continue to run and verify independently?

The disagreement itself may be the important evidence. People who share a cypherpunk lineage can still disagree about freedom, responsibility, protocol neutrality, resource consumption, and the long-term purpose of Bitcoin.

### Opening Question

> If cypherpunk is “dead”, what exactly has died? Who inherits the authority, responsibility, and freedom once attributed to that spirit? Or are these questions simply being retried inside a new technical dispute?

## Phase 0: BIP-110 Motivation, Restrictions, Rationale, and History

Abstract · Specification · Motivation · Rationale · Other

### Luke Dashjr (Luke-Jr)

- A Bitcoin Core contributor since 2011 and an editor of Bitcoin Improvement Proposals.
- Author of BIP-22 and BIP-23, which helped define the `getblocktemplate` interface used by mining software and pools.
- A long-running advocate for tighter limits on non-monetary or arbitrary data in Bitcoin.
- Founder of OCEAN, a mining-pool project that emphasized decentralization and miner control.
- In August 2026, after the closure of BIP-110 and disagreements over the project's future, he announced his resignation from OCEAN leadership and indicated a new project direction.

### Abstract: The Proposal's Core Aim

> "Temporarily limit the size of data fields at the consensus level in order to counter distorted incentives created by standardized support for arbitrary data, and to refocus priorities on improving Bitcoin as money."

The proposal frames data restrictions not merely as an efficiency measure but as a statement about Bitcoin's primary function.

### Specification: Seven Categories of Restrictions

1.  An output `scriptPubKey` may not exceed 34 bytes, except that a script whose first opcode is `OP_RETURN` may be up to 83 bytes.
2.  Data carried by `OP_PUSHDATA*`, script arguments, or witness stack items may not exceed 256 bytes, subject to specified compatibility exceptions such as the BIP16 redeemScript case.
3.  Undefined witness versions and undefined Taproot or tapleaf versions may not be spent under the new rules.
4.  Taproot annex use is restricted under the proposal's reduced-data rules.
5.  Script constructions that concatenate or rebuild larger payloads are constrained.
6.  Previously reserved or upgrade-oriented success opcodes are restricted so they cannot be used as alternate data-bearing routes.
7.  Additional tapscript constructions that could bypass the intended limits are restricted.

**Summary comment:** The rules are designed to close multiple paths for inscription-style arbitrary-data insertion, not only the `OP_RETURN` path.

### UTXO Grandfathering and GetBlockTemplate

#### UTXO grandfathering

Outputs created below the activation height are exempt when later spent. This preserves the spendability of outputs that were valid before the new restrictions took effect. Outputs created at or above the activation height are evaluated under the new rules.

#### GetBlockTemplate

The proposal adds a `reduced_data` rule identifier to the mining template interface. During the mandatory signaling period, the relevant signaling bit is included in the version requirements exposed to miners.

### Motivation: The Expansion of Inscriptions

Beginning in 2022, inscription techniques were used to place arbitrary content into Bitcoin transaction data. The proposal argues that this trend imposed unnecessary burdens on node operators and application developers while producing avoidable conflict over the use of block space.

### Motivation: Eliminate Large Contiguous Arbitrary Data

The proposal attempts to invalidate methods that embed contiguous arbitrary data larger than 256 bytes. It also invalidates large `scriptPubKey` and Tapleaf constructions used almost exclusively for data embedding, while restoring the historical 83-byte OP_RETURN policy boundary as a consensus rule.

### Motivation: Censorship Resistance

According to the proposal's rationale, Bitcoin's role as a censorship-resistant payment system is strengthened when users do not need to rely on third-party payment processors. By contrast, generalized data storage competes with payment use for block space and can increase the cost of running and using the monetary network.

### Motivation: Bitcoin Should Do One Thing Well

> Bitcoin should “do one thing, and do it well.” Rejecting data-storage use is presented as a way to reduce scope creep and allow developers to focus on Bitcoin's success as money.

### Change Log

- October 2025: initial temporary-soft-fork proposal for limiting data.
- November 2025: structural revision introducing UTXO grandfathering and revising the activation approach.
- November 2025-February 2026: deployment revisions, including BIP9-style signaling, a 55% threshold, mandatory signaling, and an expiration state.
- June 2026: the specification and implementation were treated as substantially complete.
- August 9, 2026: the proposal was closed after insufficient adoption and continued disagreement.

### Events After the Proposal Was Merged

- BIP-110 nodes began mandatory bit-4 signaling at height 961,632 and rejected blocks that did not satisfy the proposal's signaling requirements.
- OCEAN produced blocks around heights 961,632-961,633, but the broader network did not build on the resulting branch.
- The BIP-110-compatible branch remained a non-active chain tip while the ordinary Bitcoin main chain continued.
- Because the minority branch was not treated as the active chain, the associated coinbase rewards were not economically usable on the main chain.
- OCEAN announced compensation to miners affected by the stale blocks.
- After BIP-110 was closed, Luke Dashjr was removed as a BIP editor and later announced his departure from OCEAN leadership.

### Phase 0 Summary

In response to inscription-style arbitrary-data use, BIP-110 attempted to restore what its proponents regarded as Bitcoin's monetary focus by moving seven categories of data restriction into consensus.

## Phase 1: Where Can Arbitrary Data Enter the Bitcoin Block Chain?

`scriptPubKey` · `scriptSig` · witness · coinbase · Taproot script path · `OP_RETURN`

### Transaction Data Surfaces

    Bitcoin block
    └── transactions
        ├── coinbase transaction
        │   └── coinbase input script
        │       └── miner message / extra nonce / arbitrary data
        └── normal transactions
            ├── inputs
            │   ├── scriptSig
            │   │   └── legacy unlocking data / redeemScript / pushed data
            │   └── witness
            │       ├── signatures / public keys / scripts / stack items
            │       └── Taproot script path
            │           └── tapscript / control block / inscription-style data
            └── outputs
                └── scriptPubKey
                    ├── P2PKH / P2SH / P2WPKH / P2WSH / P2TR locking data
                    ├── custom scripts
                    └── OP_RETURN data

Data is not placed in one undifferentiated “block data” field. It enters through transaction structures whose validation and policy treatment differ.

### Block Chain

A block chain is the history of transaction groups connected by cryptographic references between block headers. Inside each block, transactions consume previous outputs and create new outputs. Temporary double-spending candidates may exist in the network, but consensus selects a valid active history.

### Data Locations and Their Current Practical Relevance

| Location               | Implementation keyword | Description                                                                                | Current relevance |
|------------------------|------------------------|--------------------------------------------------------------------------------------------|-------------------|
| Output locking script  | `scriptPubKey`         | The script that locks a UTXO; arbitrary byte scripts are possible within consensus limits. | High              |
| Null-data output       | `OP_RETURN`            | A special `scriptPubKey` pattern that marks an output as provably unspendable.             | High              |
| Input unlocking script | `scriptSig`            | Legacy unlocking data, including signatures, keys, redeemScripts, and pushed data.         | Medium            |
| SegWit witness         | `witness`              | Signatures, keys, scripts, and stack items stored in the witness serialization.            | Very high         |
| Taproot script path    | `tapscript`            | A spending path revealed inside witness data, including tapscript and a control block.     | Very high         |
| Coinbase input script  | `coinbase`             | Miner-controlled input data such as height, extra nonce, or a message.                     | Low               |

### Containment Relationships

`OP_RETURN` is not a separate transaction field. It is an opcode used inside an output's `scriptPubKey`.

    OP_RETURN ⊂ scriptPubKey
    Taproot script path ⊂ witness

At the raw-transaction level, an input may contain a `scriptSig` and witness data, while an output contains a `scriptPubKey`. Consensus size limits and node relay or mining policy must be distinguished.

### Example: Inspecting an OP_RETURN Output

    bitcoin-cli getrawtransaction <txid> true | jq '.vout'

    {
    "value": 0.00000000,
    "scriptPubKey": {
    "asm": "OP_RETURN ...",
    "type": "nulldata"
    }
    }

The important observation is that the null-data output is represented as one transaction output whose locking script begins with `OP_RETURN`.

### Why OP_RETURN Is Called Null Data

A null-data output records data in a provably unspendable output. Because the output can never be spent, a node does not need to keep it as a live coin in the UTXO set.

    if (coin.out.scriptPubKey.IsUnspendable()) return;

The underlying consensus rules historically allowed much larger scripts than ordinary relay policy accepted. Bitcoin Core policy evolved from a small, single OP_RETURN output toward an 83-byte default boundary, and later toward broader configurable data-carrier behavior. The key distinction is:

- **Consensus:** determines whether a block is valid.
- **Policy:** determines whether an individual node will ordinarily relay or mine a transaction.

### The First Bitcoin Transaction and the Coinbase Field

A coinbase input does not spend a previous UTXO. Its input script is created by the miner and is constrained differently from a normal transaction input. Since BIP34, the block height is committed in this field; miners also use it for an extra nonce or a message.

### SegWit Witness

Witness is the input-proof area introduced by Segregated Witness. Under BIP141, each transaction input can carry a witness field composed of witness stack items. The witness is not itself a script; it is an ordered collection of data items interpreted according to the witness program and spending conditions.

### Taproot Script Path

In a Taproot script-path spend, executable tapscript, a control block, and any required stack data are revealed inside the witness. Ordinals and inscription techniques use a valid tapscript construction while placing envelope-like data in script elements that do not need to alter the intended spending result.

BIP341 defines the Taproot commitment and spending structure; BIP342 defines tapscript validation rules.

### Putting the Data Surfaces Together

    Transaction
    ├── inputs
    │   ├── scriptSig
    │   └── witness
    │       └── Taproot script path
    │           ├── tapscript
    │           └── control block
    └── outputs
        └── scriptPubKey
            ├── standard locking script
            └── OP_RETURN <data>

### Bitcoin Core's Block Representation

In Bitcoin Core, `CBlock` extends the block-header representation and stores the block's transactions as a vector:

    class CBlock : public CBlockHeader
    {
    public:
        std::vector<CTransactionRef> vtx;
    };

The first transaction in `vtx` is the coinbase transaction; the remaining entries are ordinary transactions.

### Code-to-Concept Mapping

| Concept       | Bitcoin Core type or field         | Meaning                                                                               |
|---------------|------------------------------------|---------------------------------------------------------------------------------------|
| Bitcoin block | `class CBlock`                     | Combines a block header with the transaction list.                                    |
| Transactions  | `std::vector<CTransactionRef> vtx` | The list of transactions contained in the block; index 0 is the coinbase transaction. |

### Phase 1 Summary

OP_RETURN is a policy-recognized, provably unspendable output form, but it is not the only route for arbitrary data. Important alternative surfaces include SegWit witness and the Taproot script path.

## Phase 2: Why Restricting Only OP_RETURN Does Not Solve the Problem

OP_RETURN as provably unspendable null data · inscription techniques using witness and tapscript · policy expansion · BIP-110

### OP_RETURN and Inscriptions Are Different Mechanisms

An OP_RETURN output records data in an output script that is provably unspendable. The output is visible as a null-data output and does not remain in the UTXO set.

An inscription commonly places content in witness data revealed during a Taproot script-path spend. The committed output can look like an ordinary P2TR output before the script path is revealed.

The economically relevant distinction is not simply whether data exists, but where it is committed, when it is revealed, whether it produces a spendable UTXO, and which policy or consensus rules govern it.

### Example: Inscription \#8,719,943

- The relevant content is placed in a witness or tapscript construction rather than in an OP_RETURN output.
- Opcodes such as `OP_FALSE OP_IF` create a non-executed envelope branch for the data.
- Content-type and body markers identify the embedded media or payload.
- The witness data is part of a valid spend and is published when the Taproot script path is used.

    OP_FALSE
    OP_IF
      "ord"
      <content-type>
      OP_0
      <body>
    OP_ENDIF

### From Policy Debate to Consensus Restrictions

After Bitcoin Core 30 broadened the default policy treatment of OP_RETURN and data-carrier outputs, BIP-110 proposed a consensus response. It retained the 83-byte OP_RETURN boundary and extended restrictions to ordinary `scriptPubKey` size, pushed data, witness stack items, and alternative script routes.

The central point is that this is not only a proposal to change a local relay policy. It asks all participating nodes to share a common validity rule for data size and structure.

### Phase 2 Summary

Because OP_RETURN is only one data surface, restricting it alone cannot prevent inscription-style data publication. BIP-110 therefore expands from an OP_RETURN controversy into a broader consensus-level attempt to restrict data-bearing transaction structures.

## Phase 3: A Critical Reading of the BIP-110 Implementation

### Code Review Scope

This review does not attempt to summarize the entire political history of BIP-110. It focuses on Peter Todd's concern that the implementation may enable a poison-block condition or repeated verification work.

The review traces the connection between:

- grandfathering,
- `flags_per_input`,
- the full-script execution cache,
- repeated script execution, and
- possible validation-cost amplification.

The objective is to distinguish what can be confirmed by code inspection from what still requires experiment.

### Why Is Grandfathering Necessary?

If the reduced-data rules were applied retroactively, an output that was valid when created could become impossible to spend after activation. BIP-110 therefore preserves the spendability of UTXOs created before the activation boundary.

However, validation can no longer apply one uniform transaction-wide rule set. The validator must know whether each spent UTXO was created before or after activation.

    rules_per_input[i] = rule set derived from the spent UTXO's context

### Code Changes Introduced by the Compatibility Requirement

- **Grandfathering is required:** otherwise previously valid coins may become unspendable.
- **Rules must be selected per input:** one transaction may spend both old and new UTXOs.
- **`flags_per_input` is introduced:** each input can be checked under a different combination of verification flags.
- **The full-script cache condition changes:** a cache keyed around one uniform flag set cannot always represent a mixed per-input validation question.

### Full-Script Cache and Signature Cache

Two related but distinct caches are relevant:

**Signature cache**  
Reuses the result of a signature-verification operation identified by the signature, public key, and message or sighash context.

**Script-execution cache**  
Reuses the result of a previously successful input or transaction script validation under a particular set of flags.

If the full-script cache cannot be used, signature verification may still benefit from the signature cache. But other work - script parsing, witness-program processing, stack execution, tapscript evaluation, opcode processing, and cache lookup logic - may be repeated.

### Core Function Call Path

    ConnectBlock()
      └── determine grandfathering / per-input rule state
          └── CheckInputScripts(...)
              └── script interpreter
                  └── OP_CHECKSIG / ECDSA / Schnorr
                      └── libsecp256k1

Peter Todd's criticism is that the full-script cache relies on a uniform rule question. If grandfathering creates transactions whose inputs use different rule sets, cached results may not be reusable in the normal way. An attacker could then attempt to construct blocks that maximize repeated validation work.

### Experimental Hypotheses

| Hypothesis | Claim                                                                                                                        | Current evidence                                  | Next validation                                                                |
|------------|------------------------------------------------------------------------------------------------------------------------------|---------------------------------------------------|--------------------------------------------------------------------------------|
| A          | Use of `flags_per_input` creates cases in which the existing full-script cache cannot be reused.                             | Code-review evidence and Peter Todd's criticism.  | Instrument cache-use conditions and verify hit/miss behavior.                  |
| B          | If cached full-script results cannot be reused, the same input scripts can be executed again during later validation stages. | Call-path reasoning; not yet fully measured.      | Count script executions across mempool, block, reconnect, and reorg paths.     |
| C          | An attacker can combine cache non-reuse with high-cost valid scripts to amplify block-validation CPU cost.                   | Architecturally plausible; attack scale unproven. | Construct controlled blocks and compare time, CPU, and memory with a baseline. |

### Hypothesis A: Cache-Use Conditions Change

BIP-110's grandfathering requires per-input verification rules. This creates an execution path in which the ordinary full-script cache cannot always be used. The claim is not that all caching disappears, but that full-script result reuse is restricted under specific mixed-rule conditions.

### Hypothesis B: Scripts Are Executed Again

If an earlier successful validation result cannot be reused from the full-script cache, the corresponding scripts may be executed again in later validation contexts.

Questions for measurement:

- Is the repetition tied to mempool admission followed by block validation?
- Does it appear during block disconnect and reconnect?
- Does a reorganization cause additional executions?
- Is the effect identical for transactions with uniform and mixed input histories?
- Which script types and validation paths exhibit the effect?

### Hypothesis C: Verification-Cost Amplification

If cache non-use and repeated script execution can be combined, an attacker may attempt to fill a valid block with workloads that are unusually expensive to verify.

Questions that remain open:

- Can the attacker make execution sufficiently expensive under existing consensus limits?
- Does the attacker pay an economic cost proportional to the verifier's burden?
- Can the effect cause meaningful delay or only a small performance regression?
- Does the Bitcoin Core or Knots implementation contain another defense that dominates first?

### What Phase 3 Confirms and What Remains to Be Tested

| Item                        | Why it matters                                                          | Validation method                              |
|-----------------------------|-------------------------------------------------------------------------|------------------------------------------------|
| Grandfathering              | Determines whether an input is evaluated under old or new restrictions. | Consume pre- and post-activation UTXOs.        |
| `flags_per_input`           | Represents different rules inside a single transaction.                 | Trace per-input flag construction.             |
| Repeated script execution   | Establishes whether cache loss becomes real work.                       | Instrument and count executions.               |
| Full-script cache condition | Identifies the exact reuse boundary.                                    | Compare cache-hit and cache-miss paths.        |
| Time and CPU difference     | Measures amplification, not merely logical possibility.                 | Controlled benchmark with repeated runs.       |
| Attack construction         | Tests whether “poison block” is an operationally meaningful label.      | Build maximum-cost valid blocks under regtest. |

### Phase 3 Conclusion

BIP-110's grandfathering preserves the spendability of historical UTXOs, but makes validation rules depend on input context. That change leads to `flags_per_input` and alters the condition under which the full-script execution cache can be reused.

Code review can confirm the changed cache condition. It cannot, by itself, establish repeated execution at attack-relevant scale or prove a poison-block attack. The next step is to implement the A-B-C hypotheses in a regtest experiment and measure the actual amplification.

## Conclusion and Open Questions

### The Research Path

1.  **Reflection:** What does “Cypherpunk is dead?” mean?
2.  **Phase 0:** What did BIP-110 attempt to restrict?
3.  **Phase 1:** Where can arbitrary data actually enter?
4.  **Phase 2:** Why does restricting only OP_RETURN fail to address witness- and tapscript-based data insertion?
5.  **Phase 3:** What complexity and validation cost emerge when a broad data restriction is implemented with grandfathering?

### Final Summary

1.  **Cypherpunk:** the BIP-110 dispute reopens the question of whose interpretation of freedom, neutrality, responsibility, and protocol purpose should govern.
2.  **BIP-110:** after inscription-style data use expanded, the proposal attempted to refocus Bitcoin on money by defining seven categories of consensus-level restrictions.
3.  **Data surfaces:** OP_RETURN is only one route. Data can also appear in witness programs and the Taproot script path, so any broad restriction must account for multiple transaction structures.
4.  **Policy and consensus:** restoring the 83-byte OP_RETURN boundary is not merely a local relay preference when embedded in BIP-110; it becomes part of a shared block-validity rule.
5.  **Implementation risk:** grandfathering protects historical UTXO spendability, but introduces input-context-dependent rules, `flags_per_input`, and a changed full-script cache condition. A poison-block possibility remains an experimental hypothesis requiring measurement.

## References Shown or Discussed in the Presentation

- [BIP-110: Reduced Data Temporary Softfork](https://github.com/bitcoin/bips/blob/master/bip-0110.mediawiki)
- [Peter Todd: BIP-110 Code Review](https://petertodd.org/2026/bip-110-code-review)
- [Bitcoin Developer Guide: Block Chain](https://developer.bitcoin.org/devguide/block_chain.html)
- [Bitcoin Developer Guide: Transactions](https://developer.bitcoin.org/devguide/transactions.html)
- [BIP141: Segregated Witness](https://github.com/bitcoin/bips/blob/master/bip-0141.mediawiki)
- [BIP341: Taproot](https://github.com/bitcoin/bips/blob/master/bip-0341.mediawiki)
- [BIP342: Validation of Taproot Scripts](https://github.com/bitcoin/bips/blob/master/bip-0342.mediawiki)

**Editorial note:** This file follows the original 53-slide order and preserves the deck's distinction between code-level observations and hypotheses that still require regtest validation. Repeated transition slides and purely visual dividers have been represented as section headers rather than duplicated verbatim.

