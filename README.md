<div align="center">

# Awesome Multisig [![Awesome](https://awesome.re/badge.svg)](https://awesome.re)

*A curated list of wallets, standards, libraries, incidents and research for **multi-signature** and **threshold** custody across Bitcoin, Ethereum/EVM, Solana and MPC.*

[BIP-174 (PSBT)](https://github.com/bitcoin/bips/blob/master/bip-0174.mediawiki) · [Safe docs](https://docs.safe.global) · [Squads docs](https://docs.squads.so/main) · [RFC 9591 (FROST)](https://datatracker.ietf.org/doc/rfc9591/) · [SEAL Multisig Framework](https://frameworks.securityalliance.org/multisig-for-protocols/overview/)

</div>

---

Multisig ("M-of-N") removes the single point of failure of a lone private key: a transaction is only valid once *M* of *N* authorized signers approve it. This list covers on-chain script multisig (Bitcoin), smart-contract multisig (EVM and Solana) and off-chain threshold cryptography (MuSig2, FROST, MPC/TSS), plus the standards, tooling and security lessons around them.

## Contents

- [Concepts & Introductions](#concepts--introductions)
  - [Three ways to require M-of-N](#three-ways-to-require-m-of-n)
- [Why Multisig Fails — Notable Incidents](#why-multisig-fails--notable-incidents)
- [Wallets & Platforms](#wallets--platforms)
  - [Bitcoin](#bitcoin)
  - [Bitcoin hardware signers](#bitcoin-hardware-signers)
  - [Ethereum / EVM](#ethereum--evm)
  - [Solana](#solana)
  - [MPC / Threshold Custody](#mpc--threshold-custody)
- [Standards & Specifications](#standards--specifications)
  - [Bitcoin (BIPs)](#bitcoin-bips)
  - [Ethereum (EIPs / ERCs)](#ethereum-eips--ercs)
- [Libraries & SDKs](#libraries--sdks)
  - [Bitcoin libraries](#bitcoin-libraries)
  - [EVM libraries](#evm-libraries)
  - [Solana libraries](#solana-libraries)
  - [MPC / TSS libraries](#mpc--tss-libraries)
- [Smart Contracts & Modules](#smart-contracts--modules)
- [Tooling & Verification](#tooling--verification)
- [Security Guides & Frameworks](#security-guides--frameworks)
- [Research & Papers](#research--papers)
- [Signer's Checklist](#signers-checklist)
- [Related Lists](#related-lists)
- [Contributing](#contributing)

---

## Concepts & Introductions

- **[Multisignature (Bitcoin Wiki)](https://en.bitcoin.it/wiki/Multisignature)** — The original on-chain M-of-N model and its script primitives. `reference`
- **[What is multisig? (Unchained)](https://www.unchained.com/blog/what-is-multisig)** — Bitcoin multisig and collaborative custody, explained by a vendor that sells it. `article`
- **[Bitcoin Optech: MuSig](https://bitcoinops.org/en/topics/musig/)** — Topic page tracking MuSig2 from paper to BIPs, libraries and wallets. `reference`
- **[Bitcoin Optech: Threshold signatures](https://bitcoinops.org/en/topics/threshold-signature/)** — Topic page on FROST and other threshold schemes for Bitcoin. `reference`
- **[Bitcoin Optech: Miniscript](https://bitcoinops.org/en/topics/miniscript/)** — Topic page on Miniscript, the language behind timelocked and decaying multisig policies. `reference`
- **[Miniscript (Pieter Wuille)](https://bitcoin.sipa.be/miniscript/)** — Interactive reference and policy compiler for Miniscript. `reference`
- **[Bitcoin Core multisig tutorial](https://github.com/bitcoin/bitcoin/blob/master/doc/multisig-tutorial.md)** — Set up a descriptor-based multisig wallet and sign with PSBTs using only Bitcoin Core. `guide`
- **[What is a multisig wallet? (Safe)](https://safe.global/wallet)** — Smart-contract multisig on EVM chains, from Safe. `article`
- **[Safe smart account concepts](https://docs.safe.global/advanced/smart-account-concepts)** — Owners, threshold, modules, guards and fallback handlers in Safe. `reference`
- **[Which is more secure: MPC or multi-sig? (Fireblocks)](https://www.fireblocks.com/blog/mpc-vs-multi-sig/)** — How threshold signatures differ from on-chain multisig; vendor-authored, read critically. `article`
- **[Why we need wide adoption of social recovery wallets (Vitalik Buterin)](https://vitalik.eth.limo/general/2021/01/11/recovery.html)** — The case for guardians and threshold recovery over seed phrases. `article`

### Three ways to require M-of-N

| | Script multisig (Bitcoin) | Smart-contract multisig (EVM, Solana) | Threshold signatures (MuSig2, FROST, MPC/TSS) |
|---|---|---|---|
| **Where M-of-N is enforced** | Consensus rules: `OP_CHECKMULTISIG`, or `OP_CHECKSIGADD` in Tapscript | Account contract or program logic | Off-chain protocol; the chain sees one public key and one signature |
| **On-chain footprint** | Policy is revealed when spent (Taproot key-path spends hide it) | Owners and threshold are readable on-chain | Looks like a single-key account |
| **Changing signers** | New script and address; funds must move | A transaction from the account itself (add/remove owner, change threshold) | Resharing keeps the same public key (FROST, MPC); MuSig2 is N-of-N, so a new set means a new key |
| **Typical failure mode** | Lost descriptor or xpub backups; tampered PSBTs or change addresses | Signers approving a malicious payload behind a spoofed UI; contract or upgrade bugs | Implementation bugs that leak key shares; policy hidden inside a vendor service |

---

## Why Multisig Fails — Notable Incidents

A threshold only helps if each signer independently checks what they sign. Most large multisig losses came from signers approving a payload they couldn't read, or from several "independent" keys sharing one weak point. The same failures drive the work on [clear signing](https://github.com/Wozacosta/awesome-clear-signing#why-blind-signing-is-dangerous--notable-incidents).

- **[Drift Protocol — ~$285M (Apr 2026)](https://blocksec.com/blog/drift-protocol-incident-multisig-governance-compromise-via-durable-nonce-exploitation)** — Two of five signers on Drift's Squads Security Council (no timelock) pre-signed malicious governance transactions; Solana durable nonces kept the signatures valid until the attacker submitted them and took over admin. `incident`
- **[Humanity Protocol — ~$36M (Jun 2026)](https://rekt.news/humanity-protocol-rekt)** — Enough keys to reach the 3-of-6 and 3-of-5 Safe thresholds had been backed up on one director's laptop, which was compromised through spear-phishing; no timelock stood between the Safes and the proxy admin. `incident`
- **[UXLINK — ~$18M+ (Sep 2025)](https://rekt.news/uxlink-rekt)** — An attacker with multisig access used `delegatecall` to remove the existing owners and install themselves, then minted tokens. `incident`
- **[Bybit — ~$1.5B (Feb 2025)](https://www.nccgroup.com/research/in-depth-technical-analysis-of-the-bybit-hack/)** — Malicious JavaScript injected into the Safe{Wallet} front end showed signers a routine transfer while their hardware wallets signed a `delegatecall` that replaced the Safe's implementation. NCC Group's technical reconstruction. `incident`
- **[Sygnia's investigation of the Bybit hack](https://www.sygnia.co/blog/sygnia-investigation-bybit-hack/)** — Forensic findings that the malicious JavaScript was served from Safe{Wallet}'s AWS infrastructure. `article`
- **[FBI PSA: North Korea responsible for the Bybit hack](https://www.ic3.gov/psa/2025/psa250226)** — Official attribution to the DPRK group "TraderTraitor". `reference`
- **[Radiant Capital — ~$50M (Oct 2024)](https://www.halborn.com/blog/post/explained-the-radiant-capital-hack-october-2024)** — Malware on three signers' devices showed legitimate data in the Safe front end while hardware wallets blind-signed malicious transactions against a 3-of-11 Safe. `incident`
- **[WazirX — ~$235M (Jul 2024)](https://www.halborn.com/blog/post/explained-the-wazirx-hack-july-2024)** — Signers of a 4-of-6 Safe co-managed with Liminal approved a transaction that differed from what the custody interface displayed, handing the wallet to an attacker contract. `incident`
- **[Harmony Horizon bridge — ~$100M (Jun 2022)](https://rekt.news/harmony-rekt)** — The bridge was secured by a 2-of-5 multisig; two compromised keys were enough. `incident`
- **[Ronin bridge — ~$625M (Mar 2022)](https://rekt.news/ronin-rekt)** — Attackers reached the 5-of-9 validator threshold with four Sky Mavis keys plus a stale Axie DAO signing allowance. `incident`
- **[The Parity multisig freeze (Nov 2017)](https://www.openzeppelin.com/news/parity-wallet-hack-reloaded)** — Anyone could initialize and kill the shared wallet library, freezing an estimated 500,000+ ETH; the canonical "your multisig is only as safe as its code" lesson. `incident`
- **[The Parity multisig hack (Jul 2017)](https://www.openzeppelin.com/news/on-the-parity-wallet-multisig-hack-405a8c12e8f7)** — An unprotected `initWallet` reachable through the fallback `delegatecall` let an attacker take over three wallets and drain ~153,000 ETH. `incident`
- **[TSSHOCK (Verichains, 2023)](https://www.verichains.io/tsshock/)** — Key-extraction attacks against audited Go and Rust implementations of GG18, GG20 and CGGMP21 threshold ECDSA. `vulnerability`
- **[BitForge (Fireblocks, 2023)](https://www.fireblocks.com/blog/bitforge-fireblocks-researchers-uncover-vulnerabilities-in-over-15-major-wallet-providers)** — Key-extraction flaws in GG18, GG20 and Lindell17 implementations used by more than 15 wallet providers, blockchains and open-source projects. `vulnerability`
- **[Rekt News](https://rekt.news)** — Running catalogue of major exploits, many of them multisig and key-management failures. `reference`

---

## Wallets & Platforms

### Bitcoin

- **[Sparrow Wallet](https://sparrowwallet.com)** — Desktop wallet with first-class multisig, PSBT and hardware-wallet support. ([source](https://github.com/sparrowwallet/sparrow)) `wallet`
- **[Specter Desktop](https://github.com/cryptoadvance/specter-desktop)** — Multisig coordinator built around Bitcoin Core and air-gapped signers. `wallet`
- **[Caravan](https://www.caravanmultisig.com)** — Stateless, browser-based multisig coordinator. ([source](https://github.com/caravan-bitcoin/caravan)) `wallet`
- **[Electrum](https://electrum.org)** — Long-standing wallet supporting M-of-N multisig setups. `wallet`
- **[Liana](https://lianawallet.com)** — Miniscript wallet with timelocked recovery paths, by Wizardsardine. ([source](https://github.com/wizardsardine/liana)) `wallet`
- **[Nunchuk](https://nunchuk.io)** — Mobile and desktop multisig with shared group wallets, Miniscript templates (such as decaying multisig) and Taproot MuSig2 multisig. `wallet`
- **[Bitcoin Keeper](https://www.bitcoinkeeper.app)** — Open-source mobile wallet for multi-key vaults (2-of-3, 3-of-5 or custom) mixing hardware and software signers, with inheritance tooling. ([source](https://github.com/KeeperCommunity/bitcoin-keeper)) `wallet`
- **[Bitcoin Core](https://github.com/bitcoin/bitcoin/blob/master/doc/descriptors.md)** — Descriptor wallets support `multi`, `sortedmulti`, `multi_a`, `musig()` and Miniscript in `wsh` and `tr`. `wallet`
- **[Unchained](https://www.unchained.com)** — Collaborative custody: 2-of-3 bitcoin vaults in which the client holds two keys and Unchained one. `custody`
- **[Casa](https://casa.io)** — Multi-key bitcoin vaults (3-key and 5-key) where Casa holds one recovery and co-signing key; also offers an Ethereum and stablecoin vault. `custody`

### Bitcoin hardware signers

- **[COLDCARD multisig](https://coldcard.com/docs/multisig/)** — Air-gapped PSBT signing for M-of-N wallets with up to 15 co-signers. `hardware`
- **[Ledger Bitcoin app](https://github.com/LedgerHQ/app-bitcoin)** — Registers multisig and Miniscript wallets as BIP-388 wallet policies; supports `musig()` in Taproot policies since v2.4.0. `hardware`
- **[Frostsnap](https://frostsnap.com)** — Daisy-chained hardware devices that hold FROST key shares, so a Taproot wallet needs any *t* of *n* devices to sign. ([source](https://github.com/frostsnap/frostsnap)) `hardware`

### Ethereum / EVM

- **[Safe](https://safe.global)** — The most widely used smart-contract multisig and smart account on EVM chains, stewarded by the Safe Ecosystem Foundation. ([source](https://github.com/safe-global)) `wallet`
- **[Safe{Wallet}](https://app.safe.global)** — The hosted interface to Safe accounts, operated by Safe Labs since October 2025. ([source](https://github.com/safe-global/safe-wallet-monorepo)) `wallet`
- **[Den](https://onchainden.com)** — Treasury and operations wallet for teams, with approval policies, mobile and API signing, and onchain enforcement. `wallet`
- **[Candide](https://candide.dev)** — ERC-4337 bundler, paymaster and AbstractionKit SDK for Safe accounts, plus a Safe recovery module service. ([source](https://github.com/candidelabs)) `infra`

### Solana

- **[Squads](https://squads.xyz)** — The standard multisig and smart-account platform on Solana. ([docs](https://docs.squads.so/main)) `wallet`
- **[Squads Protocol v4](https://github.com/Squads-Protocol/v4)** — The v4 multisig program: time locks, spending limits, roles and sub-accounts; audited by OtterSec, Neodyme, Certora (with formal verification) and Trail of Bits. ([audits](https://github.com/Squads-Protocol/v4/tree/main/audits)) `contract`
- **[Squads Smart Account Program](https://github.com/Squads-Protocol/smart-account-program)** — Newer program for rent-free smart accounts with policy-based execution. `contract`
- **[Realms / SPL Governance](https://github.com/Mythic-Project/solana-program-library/tree/master/governance)** — DAO governance and treasury control, now maintained by the Mythic Project after the SPL monorepo was archived. ([app](https://realms.today)) `wallet`
- **[SPL Token multisig](https://github.com/solana-program/token)** — The Token program's native `Multisig` account: M-of-N authority over mints and token accounts, up to 11 signers. `contract`

### MPC / Threshold Custody

- **[Fireblocks](https://www.fireblocks.com)** — Institutional MPC-CMP custody with a transaction policy engine. `custody`
- **[Dfns](https://www.dfns.co)** — MPC key management and wallet infrastructure as an API. `custody`
- **[Web3Auth](https://web3auth.io)** — Threshold key management (tKey, MPC) for consumer onboarding; acquired by Consensys in 2025. `infra`
- **[ZenGo](https://zengo.com)** — Consumer two-party MPC wallet with no seed phrase. `wallet`

---

## Standards & Specifications

### Bitcoin (BIPs)

**Script multisig and derivation**

- **[BIP-11 — M-of-N Standard Transactions](https://github.com/bitcoin/bips/blob/master/bip-0011.mediawiki)** — Bare `OP_CHECKMULTISIG`. `spec`
- **[BIP-16 — Pay to Script Hash (P2SH)](https://github.com/bitcoin/bips/blob/master/bip-0016.mediawiki)** — The redeem-script pattern most legacy multisig relies on. `spec`
- **[BIP-342 — Validation of Taproot Scripts](https://github.com/bitcoin/bips/blob/master/bip-0342.mediawiki)** — Replaces `OP_CHECKMULTISIG` in Tapscript with `OP_CHECKSIGADD`, enabling batch-verifiable script multisig. `spec`
- **[BIP-48 — Multi-Script Hierarchy for Multi-Sig Wallets](https://github.com/bitcoin/bips/blob/master/bip-0048.mediawiki)** — The derivation paths most multisig wallets use, per script type. `spec`
- **[BIP-87 — Hierarchy for Deterministic Multisig Wallets](https://github.com/bitcoin/bips/blob/master/bip-0087.mediawiki)** — A script-type-agnostic alternative to BIP-48 that relies on descriptors. `spec`
- **[BIP-67 — Deterministic key sorting](https://github.com/bitcoin/bips/blob/master/bip-0067.mediawiki)** — Canonical ordering of public keys in P2SH multisig scripts. `spec`
- **[BIP-89 — Chain Code Delegation](https://github.com/bitcoin/bips/blob/master/bip-0089.mediawiki)** — Lets a collaborative-custody co-signer sign without holding the chain code, so it can't see the wallet's balances or activity. `spec`

**Descriptors and wallet policies**

- **[BIP-383 — Multisig Output Script Descriptors](https://github.com/bitcoin/bips/blob/master/bip-0383.mediawiki)** — The `multi()` and `sortedmulti()` descriptors. `spec`
- **[BIP-379 — Miniscript](https://github.com/bitcoin/bips/blob/master/bip-0379.md)** — A structured language for spending policies (thresholds, timelocks, recovery paths). Draft. `spec`
- **[BIP-388 — Wallet Policies for Descriptor Wallets](https://github.com/bitcoin/bips/blob/master/bip-0388.mediawiki)** — A compact descriptor format hardware signers can register and display, used for multisig and Miniscript. `spec`
- **[BIP-129 — Bitcoin Secure Multisig Setup (BSMS)](https://github.com/bitcoin/bips/blob/master/bip-0129.mediawiki)** — A protocol for signers and a coordinator to set up a multisig wallet and verify each other's keys. `spec`

**PSBT**

- **[BIP-174 — Partially Signed Bitcoin Transactions (PSBT)](https://github.com/bitcoin/bips/blob/master/bip-0174.mediawiki)** — The interchange format for collecting signatures. `spec`
- **[BIP-370 — PSBT Version 2](https://github.com/bitcoin/bips/blob/master/bip-0370.mediawiki)** — Lets inputs and outputs be added after the PSBT is created, which helps interactive multi-party transactions. `spec`
- **[BIP-373 — MuSig2 PSBT Fields](https://github.com/bitcoin/bips/blob/master/bip-0373.mediawiki)** — PSBT fields for exchanging MuSig2 nonces and partial signatures. `spec`

**Schnorr multisig and threshold signatures**

- **[BIP-327 — MuSig2](https://github.com/bitcoin/bips/blob/master/bip-0327.mediawiki)** — Two-round N-of-N Schnorr multi-signatures compatible with BIP-340. `spec`
- **[BIP-328 — Derivation Scheme for MuSig2 Aggregate Keys](https://github.com/bitcoin/bips/blob/master/bip-0328.mediawiki)** — BIP-32 derivation from a MuSig2 aggregate key. `spec`
- **[BIP-390 — musig() Descriptor Key Expression](https://github.com/bitcoin/bips/blob/master/bip-0390.mediawiki)** — Describing MuSig2 aggregate keys in output descriptors. Draft. `spec`
- **[FROST signing for BIP-340 (BIP-445 proposal)](https://github.com/siv2r/bip-frost-signing)** — A BIP340-compatible variant of FROST threshold signing; under review in [bitcoin/bips#2070](https://github.com/bitcoin/bips/pull/2070). `proposal`
- **[ChillDKG](https://github.com/BlockstreamResearch/bip-frost-dkg)** — Proposed BIP for distributed key generation for FROST, by Blockstream Research; under review in [bitcoin/bips#2227](https://github.com/bitcoin/bips/pull/2227). `proposal`

### Ethereum (EIPs / ERCs)

- **[EIP-712 — Typed structured data hashing and signing](https://eips.ethereum.org/EIPS/eip-712)** — How a Safe transaction hash is built; the hash signers should compare on their device. `spec`
- **[ERC-1271 — Standard Signature Validation for Contracts](https://eips.ethereum.org/EIPS/eip-1271)** — How a multisig contract "signs" a message. `spec`
- **[ERC-4337 — Account Abstraction Using Alt Mempool](https://eips.ethereum.org/EIPS/eip-4337)** — Smart accounts (including multisig) via an EntryPoint contract and bundlers, without protocol changes. `spec`
- **[ERC-6900 — Modular Smart Contract Accounts and Plugins](https://eips.ethereum.org/EIPS/eip-6900)** — Pluggable validation, including multisig modules. `spec`
- **[ERC-7579 — Minimal Modular Smart Accounts](https://eips.ethereum.org/EIPS/eip-7579)** — A leaner module standard; Safe supports it through the Safe7579 adapter. `spec`
- **[ERC-7484 — Registry Extension for ERC-7579](https://eips.ethereum.org/EIPS/eip-7484)** — Lets accounts check module attestations before installing a module. `spec`
- **[ERC-7913 — Signature Verifiers](https://eips.ethereum.org/EIPS/eip-7913)** — Verifies signatures from keys without an Ethereum address (P256, RSA, WebAuthn), so they can be multisig signers. `spec`
- **[EIP-7702 — Set Code for EOAs](https://eips.ethereum.org/EIPS/eip-7702)** — Lets an EOA delegate to contract code. The EOA's own key keeps full control, so delegating to multisig code doesn't put a threshold on that key. `spec`
- **[Safe and EIP-7702](https://docs.safe.global/advanced/eip-7702/7702-safe)** — Why the stock Safe singleton can't be used for 7702 delegation, and Safe's experimental, unaudited alternatives. `reference`

---

## Libraries & SDKs

### Bitcoin libraries

- **[libsecp256k1](https://github.com/bitcoin-core/secp256k1)** — Bitcoin Core's curve library; its `musig` module (since v0.6.0) implements BIP-327 MuSig2. `library`
- **[secp256k1-zkp](https://github.com/BlockstreamResearch/secp256k1-zkp)** — Blockstream's experimental fork of libsecp256k1 with MuSig2, adaptor signatures and more. `library`
- **[rust-miniscript](https://git.rust-bitcoin.org/rust-bitcoin/rust-miniscript/)** — Rust Miniscript and descriptor library; its GitHub repository is archived and development moved to this host. `library`
- **[BDK Wallet](https://github.com/bitcoindevkit/bdk_wallet)** — Descriptor-based Rust wallet library that handles multisig and Miniscript policies. `library`
- **[bitcoinjs-lib](https://github.com/bitcoinjs/bitcoinjs-lib)** — Build multisig scripts and PSBTs in JavaScript and TypeScript. `library`
- **[@caravan/bitcoin](https://github.com/caravan-bitcoin/caravan/tree/main/packages/caravan-bitcoin)** — Multisig and PSBT primitives from the Caravan monorepo. `library`
- **[FROST-BIP340](https://github.com/jesseposner/FROST-BIP340)** — Educational reference implementation of FROST for secp256k1 and BIP-340, with no external crypto dependencies. `library`

### EVM libraries

- **[Safe{Core} SDK](https://github.com/safe-global/safe-core-sdk)** — TypeScript SDK (Protocol Kit, API Kit, Relay Kit) to build, sign and execute Safe transactions. ([docs](https://docs.safe.global/sdk/overview)) `library`
- **[safe-eth-py](https://github.com/safe-global/safe-eth-py)** — Python library for Safe and Ethereum utilities. `library`
- **[micro-eth-signer](https://github.com/paulmillr/micro-eth-signer)** — Minimal, auditable Ethereum transaction and message signer; useful for checking what a multisig will execute. `library`

### Solana libraries

- **[@sqds/multisig](https://github.com/Squads-Protocol/v4)** — TypeScript SDK for Squads v4, alongside the `squads-multisig` Rust crate. `library`

### MPC / TSS libraries

- **[ZF FROST](https://github.com/ZcashFoundation/frost)** — Zcash Foundation's Rust FROST implementation (RFC 9591), including a `frost-secp256k1-tr` ciphersuite for Taproot. `library`
- **[cggmp21](https://github.com/LFDT-Lockness/cggmp21)** — Rust threshold ECDSA based on CGGMP, audited by Kudelski; originally by Dfns, now under LF Decentralized Trust. `library`
- **[cb-mpc](https://github.com/coinbase/cb-mpc)** — Coinbase's open-source MPC library: key generation, refresh, signing and verifiable key backup. `library`
- **[mpc-lib](https://github.com/fireblocks/mpc-lib)** — Fireblocks' C++ implementation of MPC-CMP for ECDSA and EdDSA. `library`
- **[multi-party-sig](https://github.com/taurushq-io/multi-party-sig)** — Taurus' Go implementation of CGGMP threshold ECDSA and FROST for Taproot. `library`
- **[dkls23](https://github.com/silence-laboratories/dkls23)** — Silence Laboratories' Rust DKLs23 threshold ECDSA, audited by Trail of Bits. `library`
- **[tss-lib](https://github.com/bnb-chain/tss-lib)** — BNB Chain's Go threshold ECDSA (GG18) and EdDSA. `library`
- **[multi-party-ecdsa](https://github.com/ZenGo-X/multi-party-ecdsa)** — ZenGo's Rust threshold ECDSA (GG18/GG20); no longer maintained, kept for reference. `library`

---

## Smart Contracts & Modules

- **[safe-smart-account](https://github.com/safe-fndn/safe-smart-account)** — The audited Safe contracts (formerly `safe-contracts`), now under the Safe Foundation's `safe-fndn` organization. `contract`
- **[safe-modules](https://github.com/safe-fndn/safe-modules)** — Official Safe modules: ERC-4337, passkeys, allowances and recovery. `contract`
- **[Safe7579](https://github.com/rhinestonewtf/safe7579)** — ERC-7579 adapter for Safe accounts, built by Rhinestone and Safe. `contract`
- **[Zodiac](https://github.com/zodiaceco/zodiac)** — Standard for modules, modifiers and guards on Safe-style accounts (delays, roles, exits). `contract`
- **[Zodiac Roles Modifier](https://github.com/zodiaceco/zodiac-modifier-roles)** — Role-based permissions that scope what a Safe's operators can call. `contract`
- **[OpenZeppelin multisig accounts](https://docs.openzeppelin.com/contracts/5.x/multisig)** — `MultiSignerERC7913` and its weighted variant: multisig smart accounts whose signers can be any ERC-7913 key. ([source](https://github.com/OpenZeppelin/openzeppelin-contracts)) `contract`
- **[Solady](https://github.com/Vectorized/solady)** — Gas-optimized building blocks, including ERC-1271 and ERC-4337 helpers. `contract`

---

## Tooling & Verification

Tools that let each signer verify a transaction independently of the interface that proposed it.

- **[safe-tx-hashes-util](https://github.com/pcaversaccio/safe-tx-hashes-util)** — Bash script that recomputes a Safe transaction's domain, message and EIP-712 hashes so they can be compared with the hardware wallet. `tool`
- **[safe-hash](https://github.com/Cyfrin/safe-hash-rs)** — Cyfrin's Rust CLI to verify Safe transaction data and EIP-712 messages before signing, with an offline mode. `tool`
- **[Safe Utils (OpenZeppelin)](https://safeutils.openzeppelin.com)** — Web UI that computes Safe transaction hashes. ([source](https://github.com/OpenZeppelin/safe-utils)) `tool`
- **[Safe Transaction Service](https://github.com/safe-global/safe-transaction-service)** — Indexes Safe transactions and collects off-chain confirmations. ([API docs](https://docs.safe.global/core-api/transaction-service-overview)) `tool`
- **[Safe CLI](https://github.com/safe-global/safe-cli)** — Command-line management of Safe accounts. `tool`
- **[Tenderly Simulator](https://tenderly.co/transaction-simulator)** — Simulate a multisig transaction before signers approve it. A simulation only helps if it's bound to the exact bytes being signed. `tool`
- **[solana-verify](https://github.com/Ellipsis-Labs/solana-verifiable-build)** — Verifiable builds for Solana programs, used to check that the deployed Squads program matches its source. `tool`

---

## Security Guides & Frameworks

- **[SEAL Multisig Security Framework](https://frameworks.securityalliance.org/multisig-for-protocols/overview/)** — Security Alliance's guide to designing, operating and exiting protocol multisigs: roles, timelocks, signer operations and runbooks. `guide`
- **[SEAL: Secure multisig signing process](https://frameworks.securityalliance.org/wallet-security/signing-and-verification/secure-multisig-signing-process/)** — Step-by-step verification each signer should do before approving. `guide`
- **[How to perform basic transaction checks on Safe{Wallet}](https://help.safe.global/articles/2485383995-how-to-perform-basic-transactions-checks-on-safewallet)** — Safe's own guide to checking a transaction before signing. `guide`
- **[Safe Guards](https://docs.safe.global/advanced/smart-account-guards)** — Contracts that check every Safe transaction before and after execution, for example to block `delegatecall`. `reference`
- **[Bybit hack: lessons for multisig wallet security (Certora)](https://certora.com/blog/bybit-hack-multisig-wallet-security)** — Why a threshold doesn't help when every signer sees the same manipulated UI. `article`

---

## Research & Papers

- **[MuSig2: Simple Two-Round Schnorr Multi-Signatures](https://eprint.iacr.org/2020/1261)** — Nick, Ruffing, Seurin. `paper`
- **[FROST: Flexible Round-Optimized Schnorr Threshold Signatures](https://eprint.iacr.org/2020/852)** — Komlo, Goldberg. `paper`
- **[RFC 9591 — The FROST Threshold Signature Scheme](https://datatracker.ietf.org/doc/rfc9591/)** — The IETF standardization of FROST. `spec`
- **[ROAST: Robust Asynchronous Schnorr Threshold Signatures](https://eprint.iacr.org/2022/550)** — Ruffing, Ronge, Jin, Schneider-Bensch, Schröder; wraps FROST so signing completes even when some signers don't respond. `paper`
- **[Practical Schnorr Threshold Signatures Without the Algebraic Group Model](https://eprint.iacr.org/2023/899)** — Chu, Gerhart, Ruffing, Schröder; security proof for the FROST3 variant the Bitcoin FROST BIP builds on. `paper`
- **[Fast Secure Two-Party ECDSA Signing (Lindell17)](https://eprint.iacr.org/2017/552)** — Lindell. `paper`
- **[Fast Multiparty Threshold ECDSA with Fast Trustless Setup (GG18)](https://eprint.iacr.org/2019/114)** — Gennaro, Goldfeder. `paper`
- **[One Round Threshold ECDSA with Identifiable Abort (GG20)](https://eprint.iacr.org/2020/540)** — Gennaro, Goldfeder. `paper`
- **[UC Non-Interactive, Proactive, Threshold ECDSA (CMP)](https://eprint.iacr.org/2020/492)** — Canetti, Makriyannis, Peled; the basis of MPC-CMP. `paper`
- **[UC Non-Interactive, Proactive, Threshold ECDSA with Identifiable Aborts (CGGMP)](https://eprint.iacr.org/2021/060)** — Canetti, Gennaro, Goldfeder, Makriyannis, Peled. `paper`
- **[Threshold ECDSA in Three Rounds (DKLs23)](https://eprint.iacr.org/2023/765)** — Doerner, Kondi, Lee, shelat. `paper`
- **[Practical Key-Extraction Attacks in Leading MPC Wallets](https://eprint.iacr.org/2023/1234)** — Makriyannis, Yomtov, Galansky; the paper behind BitForge. `paper`

---

## Signer's Checklist

- **Compare hashes, not screens.** Recompute the Safe transaction hash on a separate machine ([safe-tx-hashes-util](https://github.com/pcaversaccio/safe-tx-hashes-util), [safe-hash](https://github.com/Cyfrin/safe-hash-rs)) and check it matches the hardware wallet.
- **Treat `delegatecall` as an upgrade.** On Safe, `operation = 1` runs foreign code with the Safe's storage. Owner, threshold, module, guard and fallback-handler changes deserve the same scrutiny.
- **On Solana, know whether a transaction uses a durable nonce.** A signed durable-nonce transaction doesn't expire, so it can be submitted weeks later.
- **Keep keys independent.** Separate people, devices and backups; one laptop holding several keys turns M-of-N into 1-of-1.
- **Put a timelock on admin powers** so a compromised quorum can be noticed and stopped.
- **For Bitcoin, back up the wallet descriptor, not only the seeds.** Spending a multisig needs every co-signer's xpub. Register the policy on each hardware signer so it can verify change addresses.

---

## Related Lists

- **[Awesome Clear Signing](https://github.com/Wozacosta/awesome-clear-signing)** — Standards, tools and wallets for human-readable transaction signing on EVM (ERC-7730), the defense against the blind signing behind most of the incidents above.

---

## Contributing

Contributions welcome! Please read [CONTRIBUTING.md](CONTRIBUTING.md) first. In short: one link per entry, a concise neutral description with a trailing category tag, the right section, and no dead links or affiliate URLs.

## License

[![CC0](https://licensebuttons.net/p/zero/1.0/88x31.png)](LICENSE)

To the extent possible under law, the contributors have waived all copyright and related or neighboring rights to this work. See [LICENSE](LICENSE) (CC0 1.0 Universal).
