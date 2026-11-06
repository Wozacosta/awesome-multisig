# Awesome Multisig [![Awesome](https://awesome.re/badge.svg)](https://awesome.re)

> A curated list of wallets, standards, libraries, and research for **multi‑signature** and **threshold** custody across Bitcoin, Ethereum/EVM, Solana, and beyond.

Multisig ("M‑of‑N") removes the single point of failure of a lone private key: a transaction is only valid once *M* of *N* authorized signers approve it. This list covers on‑chain script multisig (Bitcoin), smart‑contract multisig (EVM/Solana), and off‑chain threshold cryptography (MPC/TSS, MuSig2, FROST) — plus the standards, tooling, and hard‑won security lessons around them.

## Contents

- [Concepts & Introductions](#concepts--introductions)
- [Wallets & Platforms](#wallets--platforms)
  - [Bitcoin](#bitcoin)
  - [Ethereum / EVM](#ethereum--evm)
  - [Solana](#solana)
  - [MPC / Threshold Custody](#mpc--threshold-custody)
- [Standards & Specifications](#standards--specifications)
  - [Bitcoin (BIPs)](#bitcoin-bips)
  - [Ethereum (EIPs / ERCs)](#ethereum-eips--ercs)
- [Libraries & SDKs](#libraries--sdks)
- [Smart Contracts](#smart-contracts)
- [Tooling & Infrastructure](#tooling--infrastructure)
- [Security & Post‑Mortems](#security--post-mortems)
- [Research & Papers](#research--papers)
- [Contributing](#contributing)

## Concepts & Introductions

- [Multisignature (Bitcoin Wiki)](https://en.bitcoin.it/wiki/Multisignature) — the original on‑chain M‑of‑N model and its script primitives.
- [What is a multisig wallet? (Safe)](https://safe.global/wallet) — smart‑contract multisig, explained for EVM.
- [MPC vs. Multisig](https://www.fireblocks.com/blog/mpc-vs-multi-sig/) — how threshold signatures differ from on‑chain multisig (vendor‑authored; read critically).
- [Vitalik Buterin — Why we need wide adoption of social recovery wallets](https://vitalik.eth.limo/general/2021/01/11/recovery.html) — the case for guardians/threshold recovery over seed phrases.
- [Unchained — What is a bitcoin multisig wallet?](https://www.unchained.com/blog/what-is-multisig) — collaborative custody framing.

## Wallets & Platforms

### Bitcoin

- [Sparrow Wallet](https://sparrowwallet.com) — desktop wallet with first‑class multisig, PSBT, and hardware‑wallet support. ([source](https://github.com/sparrowwallet/sparrow))
- [Specter Desktop](https://github.com/cryptoadvance/specter-desktop) — multisig coordinator built around Bitcoin Core and airgapped signers.
- [Nunchuk](https://nunchuk.io) — mobile/desktop collaborative multisig with key‑sharing and inheritance.
- [Caravan](https://www.caravanmultisig.com) — stateless, browser‑based multisig coordinator. ([source](https://github.com/caravan-bitcoin/caravan))
- [Electrum](https://electrum.org) — long‑standing wallet supporting M‑of‑N multisig setups.
- [Liana](https://lianawallet.com) — multisig with timelocked recovery paths (Miniscript‑based).

### Ethereum / EVM

- [Safe](https://safe.global) — the most widely used smart‑contract multisig / smart account on EVM chains. ([source](https://github.com/safe-global))
- [Den](https://onchainden.com) — multisig operations and treasury management built on Safe.
- [Candide](https://candide.dev) — ERC‑4337 smart accounts with multisig modules.
- [Safe{Wallet}](https://app.safe.global) — the hosted interface to Safe.

### Solana

- [Squads](https://squads.xyz) — the standard multisig protocol and smart‑account platform for Solana. ([source](https://github.com/Squads-Protocol/v4))
- [Realms / SPL Governance](https://github.com/Mythic-Project/solana-program-library/tree/master/governance) — multisig‑style DAO treasury control.

### MPC / Threshold Custody

- [Fireblocks](https://www.fireblocks.com) — institutional MPC‑CMP custody and transaction policy engine.
- [DFNS](https://www.dfns.co) — MPC key management as an API.
- [Web3Auth](https://web3auth.io) — threshold key management (tKey / MPC) for consumer onboarding.
- [ZenGo](https://zengo.com) — consumer MPC wallet (no seed phrase).

## Standards & Specifications

### Bitcoin (BIPs)

- [BIP‑11 — M‑of‑N Standard Transactions](https://github.com/bitcoin/bips/blob/master/bip-0011.mediawiki) — bare `OP_CHECKMULTISIG`.
- [BIP‑16 — Pay to Script Hash (P2SH)](https://github.com/bitcoin/bips/blob/master/bip-0016.mediawiki) — the redeem‑script pattern most multisig relies on.
- [BIP‑48 — Multi‑Script HD wallet derivation](https://github.com/bitcoin/bips/blob/master/bip-0048.mediawiki) — standardized derivation paths for multisig accounts.
- [BIP‑67 — Deterministic key sorting](https://github.com/bitcoin/bips/blob/master/bip-0067.mediawiki) — canonical ordering of pubkeys in multisig scripts.
- [BIP‑174 — Partially Signed Bitcoin Transactions (PSBT)](https://github.com/bitcoin/bips/blob/master/bip-0174.mediawiki) — the interchange format for collecting signatures.
- [BIP‑327 — MuSig2](https://github.com/bitcoin/bips/blob/master/bip-0327.mediawiki) — two‑round Schnorr multi‑signatures for Taproot.

### Ethereum (EIPs / ERCs)

- [ERC‑1271 — Standard Signature Validation for Contracts](https://eips.ethereum.org/EIPS/eip-1271) — how a multisig contract "signs" a message.
- [ERC‑4337 — Account Abstraction via Entry Point](https://eips.ethereum.org/EIPS/eip-4337) — smart accounts (including multisig) without protocol changes.
- [ERC‑6900 — Modular Smart Contract Accounts and Plugins](https://eips.ethereum.org/EIPS/eip-6900) — pluggable validation, incl. multisig modules.
- [ERC‑7579 — Minimal Modular Smart Accounts](https://eips.ethereum.org/EIPS/eip-7579) — a leaner module standard adopted by Safe and others.

## Libraries & SDKs

- [Safe{Core} SDK](https://github.com/safe-global/safe-core-sdk) — TypeScript SDK to build, sign, and execute Safe transactions.
- [safe-eth-py](https://github.com/safe-global/safe-eth-py) — Python library for Safe and Ethereum utilities.
- [@sqds/multisig](https://github.com/Squads-Protocol/v4) — TypeScript SDK for Squads v4 on Solana.
- [bitcoinjs-lib](https://github.com/bitcoinjs/bitcoinjs-lib) — build multisig scripts and PSBTs in JS/TS.
- [@caravan/bitcoin](https://github.com/caravan-bitcoin/caravan) — multisig and PSBT primitives extracted from Caravan.
- [tss-lib](https://github.com/bnb-chain/tss-lib) — Go implementation of GG18/GG20 threshold ECDSA.
- [multi-party-ecdsa](https://github.com/ZenGo-X/multi-party-ecdsa) — Rust threshold ECDSA (GG18/GG20); no longer maintained.
- [ZF FROST](https://github.com/ZcashFoundation/frost) — Rust implementation of the FROST threshold Schnorr scheme.
- [micro-eth-signer](https://github.com/paulmillr/micro-eth-signer) — minimal, auditable Ethereum tx/message signer (useful for verifying what a multisig will execute).

## Smart Contracts

- [safe-smart-account](https://github.com/safe-fndn/safe-smart-account) — the audited Safe contracts (formerly `safe-contracts`).
- [Squads Protocol v4](https://github.com/Squads-Protocol/v4) — the on‑chain Solana multisig program.
- [Solady](https://github.com/Vectorized/solady) — gas‑optimized building blocks, incl. ERC‑1271 / ERC‑4337 helpers.
- [OpenZeppelin Contracts](https://github.com/OpenZeppelin/openzeppelin-contracts) — `Governor` and access‑control primitives often paired with multisig.

## Tooling & Infrastructure

- [Safe Transaction Service](https://github.com/safe-global/safe-transaction-service) — indexes and relays pending multisig transactions.
- [Safe CLI](https://github.com/safe-global/safe-cli) — command‑line management of Safe accounts.
- [safe-tx-hashes-util](https://github.com/pcaversaccio/safe-tx-hashes-util) — independently compute a Safe transaction hash so you can verify what you're signing.
- [Tenderly](https://tenderly.co) — simulate a multisig transaction before signers approve it.

## Security & Post‑Mortems

Multisig removes single‑key risk but adds *approval‑UI* and *signer‑hygiene* risk. Read these before trusting any setup.

- [Rekt News](https://rekt.news) — running catalog of major exploits, including multisig/Safe incidents.
- [The Parity multisig freeze (2017)](https://www.openzeppelin.com/news/parity-wallet-hack-reloaded) — a library `selfdestruct` locked ~$150M+; the canonical "your multisig is only as safe as its code" lesson.
- [Bybit / Safe UI compromise (2025)](https://rekt.news/bybit-rekt/) — signers approved a malicious payload shown a benign UI; birthed the "verify the tx hash, not the screen" practice.

## Research & Papers

- [MuSig2: Simple Two‑Round Schnorr Multi‑Signatures](https://eprint.iacr.org/2020/1261) — Nick, Ruffing, Seurin.
- [FROST: Flexible Round‑Optimized Schnorr Threshold Signatures](https://eprint.iacr.org/2020/852) — Komlo, Goldberg.
- [RFC 9591 — The FROST Threshold Signature Scheme](https://datatracker.ietf.org/doc/rfc9591/) — the IETF standardization.
- [Fast Multiparty Threshold ECDSA (GG18)](https://eprint.iacr.org/2019/114) — Gennaro, Goldfeder.
- [UC Non‑Interactive, Proactive, Threshold ECDSA (CMP/CGGMP)](https://eprint.iacr.org/2020/492) — the basis of many production MPC systems.

## Contributing

Contributions welcome! Please read [CONTRIBUTING.md](CONTRIBUTING.md) first. In short: one link per entry, a concise neutral description, correct category, and no dead links or affiliate URLs.

## License

[![CC0](https://licensebuttons.net/p/zero/1.0/88x31.png)](LICENSE)

To the extent possible under law, the contributors have waived all copyright and related or neighboring rights to this work. See [LICENSE](LICENSE) (CC0 1.0 Universal).
