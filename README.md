# RDTS (BIP-110)

## What Is the Essence of the Bitcoin Community Governance Debate?

> From the definition of the block chain and `OP_RETURN` to arbitrary-data
> input analysis and a critical review of the BIP-110 implementation.

**Author:** [@1amhesus](https://github.com/1amhesus)  
**Status:** Work in progress  
**Original presentation:** [Korean PDF](presentation/bip110-presentation-ko.pdf)

This repository contains the structured English edition of my Korean
presentation on BIP-110. It distinguishes observations confirmed through
code review from the Poison Block hypotheses that still require controlled
regtest experiments.

## Research Question

How are Bitcoin consensus rules chosen and propagated across the ecosystem?

This research approaches that question through the RDTS/BIP-110
implementation, its UTXO-grandfathering mechanism, per-input verification
flags, script-cache behavior, and the resulting hypothesis of
validation-cost amplification.
