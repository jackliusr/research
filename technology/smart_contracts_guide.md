# Smart Contracts: The Self-Executing Promise

**The Code-Level and Standards-Level Deep-Dive — from Nick Szabo's 1994 Coinage and the 2015 Ethereum Launch to the EIP/ERC Standards, the Proxy Upgrade Patterns, the SWC Security Classification, the Major Incident Record, and the Banking Tokenization Application — with a Cymbal Bank Tokenized Money-Market Fund Worked Example**

> **Author:** Jack Liu Shurui, Solution Architect
> **Context:** Banking/Technology Domain · Deep-Dive — the standards, the security, and the banking applications of smart contracts; the complementary companion to the blockchain-guide overview of ledger mechanics
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Last Updated:** September 2026
> **Companion guides (technology/, same folder):** [blockchain_technology_guide.md](blockchain_technology_guide.md) (the ledger/platform mechanics — consensus, blocks, the EVM, L2s, cross-chain bridges; its §5 is only a ~70-line smart-contract overview that this guide expands to full depth) · [defi_guide.md](defi_guide.md) (the DeFi application layer built on contracts — AMMs, lending) · [cybersecurity_guide.md](cybersecurity_guide.md) (audit practice and secure-development context) · [enterprise_middleware_integration_platform_guide.md](enterprise_middleware_integration_platform_guide.md) (integration of contract platforms into bank middleware) · [market_data_consumption_guide.md](market_data_consumption_guide.md) (oracle and market-data feeds into contracts)
> **Companion guides (banking/, prefix `../banking/`):** [tokenized_assets_guide.md](../banking/tokenized_assets_guide.md) (the institutional tokenization twin) · [jpmorgan_chase_software_systems_guide.md](../banking/jpmorgan_chase_software_systems_guide.md) (JPM Coin / Onyx blockchain themes) · [reap_global_guide.md](../banking/reap_global_guide.md) (stablecoin-collateralised cards) · [mas_regulations_guidelines_guide.md](../banking/mas_regulations_guidelines_guide.md) (the MAS regulatory posture and the Cymbal Bank persona conventions) · [enterprise_risk_management_guide.md](../banking/enterprise_risk_management_guide.md) (ERM context for the risk sections) · [cayman_bvi_master_feeder_guide.md](../banking/cayman_bvi_master_feeder_guide.md) (master-feeder fund structures, relevant to the fund-tokenization worked example)


**How to use this guide:** Section 1 is the overview — the short answer, the key-facts table, why a bank should care, and the explicit statement of how this guide relates to the blockchain umbrella guide. Section 2 is the concept, verified: Szabo's 1994 coinage and his 1996 Extropy essay, the definition, the legal-contract analogy and its limits, the Ricardian contract idea, oracles, and the self-executing versus self-enforcing distinction. Section 3 is the platforms — Ethereum's 2015 launch and founders, the EVM, Solidity's origin, other chains and permissioned platforms (cross-referencing the blockchain guide for ledger mechanics rather than re-deriving them). Section 4 is the standards process — EIP-1 and the Draft→Final lifecycle. Section 5 is the token standards deep-dive — ERC-20, ERC-721, ERC-1155, ERC-4626 — with the integration pitfalls that matter. Section 6 is the development lifecycle — frameworks, testing, deployment, the EIP-170 size limit, oracle integration. Section 7 is upgradeability — the transparent proxy, UUPS and beacon patterns, and their risks. Section 8 is the security deep-dive — the reentrancy class (SWC-107) and the DAO case, the other SWC-classified weakness classes, MEV and price manipulation, signature attacks, and audit practice (cross-referencing the cybersecurity guide, condensed). Section 9 is the incident history table. Section 10 is the banking application layer — tokenized deposits and funds, stablecoins, the JPMorgan Onyx context and the MAS posture (cross-referencing the sibling guides, condensed). Section 11 is the Cymbal Bank worked example — an illustrative tokenized money-market fund design, clearly marked as design fiction built on the verified facts of the guide. Section 12 is the claims audit (✅/⚠/❌), with §12.4 "What Could Not Be Verified" collecting every item that could not be confirmed this pass. Section 13 is the glossary. Section 14 is cross-references and the closing summary. **Reading paths:** *Banking/architecture:* §1 → §10 → §11 → §8 → §12. *Engineer/developer:* §2 → §3 → §5 → §6 → §7 → §8. *Risk/compliance:* §8 → §9 → §10 → §12. *In a hurry:* §1, §5.5, §9, §12.

**Integrity convention.** Every factual claim in this guide carries one of three marks: **✅** verified this pass against a primary or authoritative source (named in the claims audit, §12); **⚠** flagged — reported, approximate, single-sourced, fast-moving, or contested (USD figures for crypto incidents are estimates at the time and move with the market); **❌** refuted or not found. Unmarked statements are domain-stable technical knowledge (what `delegatecall` does, what an ABI is) rather than research claims. Where a figure is inherently time-varying — hack totals, market values — treat it as an order of magnitude, not a precise value.


## Table of Contents

1. [The Overview](#1-the-overview)
   - 1.1 [The Short Answer](#11-the-short-answer)
   - 1.2 [The Key-Facts Table](#12-the-key-facts-table)
   - 1.3 [Why a Bank Should Care: The Cymbal Bank Lens](#13-why-a-bank-should-care-the-cymbal-bank-lens)
   - 1.4 [The Relationship to the Blockchain Guide](#14-the-relationship-to-the-blockchain-guide)
2. [The Concept: Szabo's Coinage and the Limits of the Analogy](#2-the-concept-szabos-coinage-and-the-limits-of-the-analogy)
   - 2.1 [The Coinage, Verified: 1994 and 1996](#21-the-coinage-verified-1994-and-1996)
   - 2.2 [The Definition](#22-the-definition)
   - 2.3 [The Legal-Contract Analogy and Its Limits](#23-the-legal-contract-analogy-and-its-limits)
   - 2.4 [The Ricardian Contract Idea](#24-the-ricardian-contract-idea)
   - 2.5 [Oracles: The Bridge to Off-Chain Reality](#25-oracles-the-bridge-to-off-chain-reality)
   - 2.6 [Self-Executing versus Self-Enforcing](#26-self-executing-versus-self-enforcing)
3. [The Platforms: Where Smart Contracts Run](#3-the-platforms-where-smart-contracts-run)
   - 3.1 [Ethereum: Frontier, 30 July 2015](#31-ethereum-frontier-30-july-2015)
   - 3.2 [The EVM and the Contract Account](#32-the-evm-and-the-contract-account)
   - 3.3 [Solidity: Wood's 2014 Proposal, Reitwiessner's Team, 2015 Release](#33-solidity-woods-2014-proposal-reitwiessners-team-2015-release)
   - 3.4 [Other Chains and Permissioned Platforms](#34-other-chains-and-permissioned-platforms)
   - 3.5 [The Cross-Chain Reality: Bridges Are the Hack Surface](#35-the-cross-chain-reality-bridges-are-the-hack-surface)
4. [The Standards Process: EIP-1 and the EIP Lifecycle](#4-the-standards-process-eip-1-and-the-eip-lifecycle)
5. [The Token Standards: ERC-20, ERC-721, ERC-1155, ERC-4626](#5-the-token-standards-erc-20-erc-721-erc-1155-erc-4626)
   - 5.1 [ERC-20: The Fungible Token](#51-erc-20-the-fungible-token)
   - 5.2 [ERC-721: The Non-Fungible Token](#52-erc-721-the-non-fungible-token)
   - 5.3 [ERC-1155: The Multi-Token Standard](#53-erc-1155-the-multi-token-standard)
   - 5.4 [ERC-4626: The Tokenized Vault](#54-erc-4626-the-tokenized-vault)
   - 5.5 [The Standards Table](#55-the-standards-table)
   - 5.6 [Why Standards Matter to a Bank](#56-why-standards-matter-to-a-bank)
6. [The Development Lifecycle: Frameworks, Testing, Deployment](#6-the-development-lifecycle-frameworks-testing-deployment)
   - 6.1 [The Frameworks](#61-the-frameworks)
   - 6.2 [The Lifecycle: Write → Compile → Test → Deploy → Verify](#62-the-lifecycle-write--compile--test--deploy--verify)
   - 6.3 [Testing Practice: Unit, Integration, Fuzz, Formal](#63-testing-practice-unit-integration-fuzz-formal)
   - 6.4 [Deployment: Gas, Addresses, and the EIP-170 Limit](#64-deployment-gas-addresses-and-the-eip-170-limit)
   - 6.5 [Oracle Integration](#65-oracle-integration)
7. [Upgradeability: Proxy Patterns and Their Risks](#7-upgradeability-proxy-patterns-and-their-risks)
   - 7.1 [Immutability versus Upgradeability](#71-immutability-versus-upgradeability)
   - 7.2 [The Proxy Patterns: Transparent, UUPS, Beacon](#72-the-proxy-patterns-transparent-uups-beacon)
   - 7.3 [The Risks: Storage Collision, Function Collision, Initialization](#73-the-risks-storage-collision-function-collision-initialization)
   - 7.4 [The Upgrade-Adjacent Lessons: Poly Network and Curve](#74-the-upgrade-adjacent-lessons-poly-network-and-curve)
8. [The Security Deep-Dive: Reentrancy and the Vulnerability Classes](#8-the-security-deep-dive-reentrancy-and-the-vulnerability-classes)
   - 8.1 [The Reentrancy Class: SWC-107 and the DAO](#81-the-reentrancy-class-swc-107-and-the-dao)
   - 8.2 [The Other Classes, Mapped to SWC](#82-the-other-classes-mapped-to-swc)
   - 8.3 [MEV, Flash Loans, and Price Manipulation](#83-mev-flash-loans-and-price-manipulation)
   - 8.4 [Signature and Approval Attacks](#84-signature-and-approval-attacks)
   - 8.5 [Audit Practice](#85-audit-practice)
9. [The Incident History: What the Hacks Teach](#9-the-incident-history-what-the-hacks-teach)
10. [The Banking Applications: Tokenized Deposits and Funds](#10-the-banking-applications-tokenized-deposits-and-funds)
    - 10.1 [The Tokenization Thesis](#101-the-tokenization-thesis)
    - 10.2 [The Tokenized-Deposit Design](#102-the-tokenized-deposit-design)
    - 10.3 [Stablecoins and the Reap Context](#103-stablecoins-and-the-reap-context)
    - 10.4 [The JPMorgan Onyx Context](#104-the-jpmorgan-onyx-context)
    - 10.5 [The Regulatory Posture: MAS](#105-the-regulatory-posture-mas)
    - 10.6 [Why Standards Matter to a Bank](#106-why-standards-matter-to-a-bank)
11. [The Cymbal Bank Worked Example: A Tokenized Money-Market Fund Vault](#11-the-cymbal-bank-worked-example-a-tokenized-money-market-fund-vault)
    - 11.1 [The Design Fiction Frame](#111-the-design-fiction-frame)
    - 11.2 [The Contract Architecture](#112-the-contract-architecture)
    - 11.3 [The Standards Chosen and Why](#113-the-standards-chosen-and-why)
    - 11.4 [The Security Review: Threats Mapped to SWC](#114-the-security-review-threats-mapped-to-swc)
    - 11.5 [The Audit Trail and Middleware Integration](#115-the-audit-trail-and-middleware-integration)
12. [The Claims Audit](#12-the-claims-audit)
    - 12.1 [The Verified-Facts Table](#121-the-verified-facts-table)
    - 12.2 [The Incident-Figure Table](#122-the-incident-figure-table)
    - 12.3 [Corrections and Discrepancies](#123-corrections-and-discrepancies)
    - 12.4 [What Could Not Be Verified](#124-what-could-not-be-verified)
13. [The Glossary](#13-the-glossary)
14. [Cross-References and the Closing Summary](#14-cross-references-and-the-closing-summary)

---

## 1. The Overview

### 1.1 The Short Answer

A **smart contract** is a program that runs on a blockchain: its code is stored on-chain, its execution is deterministic and replicated by every node in the network, and its state transitions are irreversible once committed. The term was coined by the computer scientist and legal scholar **Nick Szabo in 1994**, and the concept was generalised into a production platform by **Ethereum**, which launched its mainnet on **30 July 2015** with the *Frontier* release. Ethereum's contribution was to make the blockchain a general-purpose computer — a "world computer" — on which any party can deploy a program, pay for its execution in gas, and rely on the network (not on any single counterparty) to run it exactly as written.

Three properties separate a smart contract from an ordinary server-side program:

| Property | Meaning |
|---|---|
| **Deterministic execution** | Every node runs the same bytecode against the same state and reaches the same result; there is no "interpretation" of the terms. |
| **Censorship-resistant state** | Once a transaction is committed, no single party — not even the deployer — can roll it back or alter the contract's code and state without a network-level fork. |
| **Permissionless composability** | Any contract can call any other contract; the deployed interfaces (ERC-20, ERC-4626, …) are public standards, so protocols combine like lego bricks. |

For a bank, the honest summary is more nuanced: a smart contract is a *self-executing* promise — the settlement mechanics run automatically — but **not** a self-enforcing legal contract. It does not know the law, cannot reach off-chain reality without an oracle, and will happily execute a bug. The entire discipline of this guide is the management of that gap: standards so contracts interoperate, security so bugs are found before attackers do, and design so the on-chain promise maps onto the bank's legal and accounting reality.

### 1.2 The Key-Facts Table

| Fact | Value | Status |
|---|---|---|
| Term coined | Nick Szabo, essay "Smart Contracts" (1994); expanded in "Smart Contracts: Building Blocks for Digital Markets", *Extropy* #16 (1996) | ✅ §2.1 |
| First general-purpose platform | Ethereum mainnet, *Frontier* release, 30 July 2015 (whitepaper late 2013; founders incl. Vitalik Buterin, Gavin Wood, Charles Hoskinson, Anthony Di Iorio, Joseph Lubin) | ✅ §3.1 |
| Dominant language | Solidity — proposed by Gavin Wood in August 2014, developed by the Ethereum project's team led by Christian Reitwiessner, first released 2015 | ✅ §3.3 |
| Fungible-token standard | ERC-20 — Fabian Vogelsteller & Vitalik Buterin, created 19 Nov 2015, **Final** | ✅ §5.1 |
| NFT standard | ERC-721 — William Entriken, Dieter Shirley, Jacob Evans, Nastassia Sachs, created 24 Jan 2018, **Final** | ✅ §5.2 |
| Multi-token standard | ERC-1155 — Witek Radomski *et al.* (Enjin), created 17 Jun 2018, **Final** | ✅ §5.3 |
| Tokenized-vault standard | ERC-4626 — Joey Santoro *et al.*, created 22 Dec 2021, **Final** (reported finalised March 2022) | ✅ §5.4 |
| Contract-size limit | EIP-170: 0x6000 = 24,576 bytes (24 KB), Spurious Dragon hard fork (2016) | ✅ §6.4 |
| Canonical hack | The DAO, 17 June 2016 — reentrancy (SWC-107), ~3.6M ETH drained (≈US$60M reported at the time ⚠); hard fork 20 July 2016 | ✅ §8.1 |
| Weakness taxonomy | SWC Registry (swcregistry.io, per EIP-1470) — e.g., SWC-107 reentrancy, SWC-101 overflow, SWC-115 tx.origin, SWC-106 selfdestruct | ✅ §8 |
| Toolchains | Truffle (2015, ConsenSys), Hardhat (Buidler → Hardhat, Nomic Labs/Foundation), Foundry (Dec 2021, Paradigm; forge/cast/anvil) | ✅ §6.1 |
| Largest bridge hacks | Poly Network (Aug 2021, ~US$610M, returned), Ronin (Mar 2022, ~US$625M), Wormhole (Feb 2022, ~US$320M), Nomad (Aug 2022, ~US$190M) | ⚠ §9 |

### 1.3 Why a Bank Should Care: The Cymbal Bank Lens

This guide is written from the vantage of **Cymbal Bank**, the fictional-but-substantiated Singapore-based bank persona used across this repository (its persona conventions are documented in [mas_regulations_guidelines_guide.md](../banking/mas_regulations_guidelines_guide.md)). Cymbal Bank's interest in smart contracts is not speculative: the Monetary Authority of Singapore (MAS) has run the **Project Guardian** asset-tokenization pilot since May 2022 with DBS, JPMorgan and SBI Digital Asset Holdings, and in August 2023 finalised the world's first comprehensive regulatory framework for single-currency stablecoins. The building block underneath all of that — tokenized deposits, tokenized money-market funds, tokenized bonds — is the smart contract. A bank that cannot read a contract, cannot commission an audit, and cannot map a standard to a control is not equipped to be a counterparty in that market.

The bank's angle on every section of this guide:

| Section | The bank question |
|---|---|
| §2 Concept | What exactly is the promise, and where does it end? |
| §3 Platforms | Which rails (public vs permissioned) do we deploy on, and why? |
| §4–5 Standards | Which interfaces will our tokens and vaults implement, and who else speaks them? |
| §6–7 Development | How do we build, test, deploy and upgrade without breaking the promise? |
| §8–9 Security | What are the failure classes, and what did the US$2B+ of incidents teach? |
| §10 Applications | What is the product thesis — deposits, funds, stablecoins? |
| §11 Worked example | What does a production-grade design actually look like? |

The risk-management overlay for all of this is in [enterprise_risk_management_guide.md](../banking/enterprise_risk_management_guide.md); this guide supplies the technical substrate that ERM must assess.

### 1.4 The Relationship to the Blockchain Guide

The umbrella [blockchain_technology_guide.md](blockchain_technology_guide.md) covers the ledger and platform mechanics — consensus algorithms, blocks, the Ethereum Virtual Machine, layer-2s, and cross-chain bridges — and its §5 treats smart contracts in a ~70-line overview. **This guide is the complementary deep-dive**: it assumes the ledger mechanics (consensus, blocks, the EVM's gas model) as given and goes one level deeper into the things the umbrella guide deliberately skims — the standards (EIP/ERC), the security classification (SWC) and incident record, the development and upgrade patterns, and the banking application layer. Where ledger mechanics are needed for context (e.g., the EVM account model in §3.2, bridge mechanics in §3.5), this guide cross-references the umbrella guide instead of re-deriving it. The DeFi layer that is built *on top of* these contracts — AMMs, lending protocols — is covered in [defi_guide.md](defi_guide.md); this guide covers the contracts themselves.

---

## 2. The Concept: Szabo's Coinage and the Limits of the Analogy

### 2.1 The Coinage, Verified: 1994 and 1996

The term "smart contract" was coined by **Nick Szabo**, a computer scientist and legal scholar, in a 1994 essay titled *Smart Contracts*, published on his website (szabo.best.vwh.net). In 1996 he published *Smart Contracts: Building Blocks for Digital Markets* in *Extropy* #16 — explicitly labelled, in the surviving text, as "a partial rewrite of the article which appeared in Extropy #16" and carrying a 1996 copyright notice. ✅ The 1996 essay was verified directly this pass at the University of Amsterdam mirror (fon.hum.uva.nl, a long-standing mirror of szabo.best.vwh.net): *"Smart Contracts: Building Blocks for Digital Markets," Copyright (c) 1996 by Nick Szabo; (This is a partial rewrite of the article which appeared in Extropy #16).* ✅ The 1994 dating of the earlier essay is attested by multiple independent secondary sources (including the blockchain-history literature and the essay's own provenance); the original szabo.best.vwh.net domain is offline and was unreachable this pass, so the 1994 page itself could not be re-read directly — see §12.4.

Why two years matter: many secondary sources conflate the two essays. The precise record: **1994** — the term appears in the essay *Smart Contracts*; **1996** — it is consolidated and circulated via the *Extropy* #16 article. Both are Szabo's, and both predate Bitcoin (2008) and Ethereum (2013–2015) by over a decade — and Ethereum's whitepaper explicitly credits Szabo's framing, describing Bitcoin as a weak form of the smart-contract idea and proposing a stronger, Turing-complete version.

The 1996 essay opens with the example that still anchors every explainer: the **vending machine**. A vending machine is a primitive smart contract — it holds property (goods), enforces the agreed terms (coins in, item out) mechanically, and can be attacked but cannot be cheated in the normal course. Szabo's argument was to generalise that mechanism: embed contracts in all sorts of property "that is valuable and controlled by digital means," with the contract referencing that property "in a dynamic, proactively enforced form."

### 2.2 The Definition

Szabo's definition, from the 1994 essay (repeated in the 1996 version): a smart contract is *"a computerized transaction protocol that executes the terms of a contract."* The general objectives of contract design, in his words, are "to satisfy common contractual conditions (such as payment terms, liens, confidentiality, and even enforcement), minimize exceptions both malicious and accidental, and minimize the need for trusted intermediaries."

In modern, Ethereum-era terms, the operational definition used throughout this guide is:

> A smart contract is a stateful program deployed to a blockchain, executed by the network's virtual machine, whose transitions are triggered by transactions, priced in gas, recorded immutably, and verifiable by any observer against the published source code and interface.

Three elements of that definition do the work: **stateful** (contracts hold balances and data, not just logic), **immutable** (the deployed bytecode cannot change without a proxy indirection or a fork), and **verifiable** (source code, bytecode and interface are all public, so the contract is inspectable in a way a bank's legacy COBOL never was).

### 2.3 The Legal-Contract Analogy and Its Limits

The word "contract" invites a legal reading, and the analogy is useful but dangerous. What a smart contract shares with a legal contract: an agreed set of terms, parties, and a mechanism of performance. What it does not share:

| | Legal contract | Smart contract |
|---|---|---|
| Formation | Offer, acceptance, consideration; legal capacity | Deployment + transactions; anyone can call |
| Interpretation | Courts, statutes, jurisdiction, precedent | Bytecode, exactly; no equity, no good faith |
| Performance | Enforceable by courts (coercion) | Executed by the network, automatically |
| Failure modes | Breach, frustration, mistake, voidness | Bugs, reentrancy, oracle error, stuck state |
| Amendment | Mutual agreement, consideration | New deployment, proxy upgrade, or nothing |
| Remedies | Damages, specific performance, rescission | Reverts, forks, or nothing |

The slogan "code is law" (popularised in the 1990s by Lawrence Lessig in a different context, and later adopted by the crypto ecosystem) is best treated as a *description of what happens* — the code executes as written, regardless of intent — and not as a *normative claim*. The DAO hack of June 2016 is the canonical demonstration: the code executed exactly as written, and the result was ~3.6M ETH drained from a fund that had raised ~US$150M. The Ethereum community then chose to hard-fork to reverse the outcome — proof that even on a blockchain, the community's legal-moral judgment overrode the letter of the executed code. For a bank, the correct mental model is: **the smart contract is the settlement engine of a legal relationship, not a substitute for the legal relationship.** Legal wrappers (term sheets, offering documents, custody agreements) still exist; the contract automates the performance leg. The industry term for the combined construct is the *smart legal contract* — the on-chain code plus the off-chain legal prose that governs it.

### 2.4 The Ricardian Contract Idea

The bridge between legal prose and machine-executable code has a name: the **Ricardian contract**, articulated by Ian Grigg in the 1990s and formalised in his paper *The Ricardian Contract* (canonical version 2004 ⚠ — the exact first-publication date is contested across sources). A Ricardian contract is a single document that is simultaneously (a) a human-readable contract, (b) a machine-readable format, and (c) cryptographically signed and identified, so that a hash of the document *is* its identifier on the ledger. The point, for banking, is that the classic smart-contract stack records *transfers* but not *meaning*; a Ricardian wrapper pins the meaning (the fund's prospectus, the loan's covenants) to the executing code. This is precisely the pattern MAS-aligned tokenization pilots and stablecoin frameworks assume: the token references an off-chain legal and disclosure document, and the contract enforces the mechanics.

### 2.5 Oracles: The Bridge to Off-Chain Reality

A blockchain is a closed system: contracts can only see what is on the ledger. Any contract whose logic depends on the outside world — a fund NAV, an FX rate, a bond price, a weather index — needs an **oracle**: a trusted feed that writes external data on-chain. The oracle is therefore the single largest trust concession in an otherwise trust-minimised design, and oracle manipulation (feeding a false price to a contract that prices liquidations or vault shares) is one of the most exploited vulnerability classes in DeFi history (see §8.3 and §9). The standard industry answers are decentralised oracle networks (Chainlink's aggregated price feeds being the best-known), **TWAPs** (time-weighted average prices computed from on-chain exchange data, which are manipulation-resistant but laggy), and — the banker's answer — **admin-pushed, audited data**: for a regulated fund, the NAV is computed off-chain by the fund administrator and pushed to the contract by a permissioned role, with the feed itself subject to the bank's data-governance controls. The data-feed and market-data theme is covered in [market_data_consumption_guide.md](../banking/market_data_consumption_guide.md); the contract-side integration pattern is in §6.5.

### 2.6 Self-Executing versus Self-Enforcing

The distinction that keeps Szabo's vision honest: a smart contract is **self-executing** — once triggered, the network runs it to completion without human intervention. It is *not* self-enforcing in the full Szabo sense: it cannot compel an off-chain obligation, seize off-chain collateral, or punish a party that walks away; enforcement of the underlying deal still rests on collateralisation, reputation, and the courts. The practical consequence for the banking reader: **on-chain settlement finality (the transfer happened) is not the same as legal finality (the transfer is unchallengeable)** — the two are reconciled by the instrument's design (who can pause, who can claw back, what the dispute process is), which is why the governance and upgrade patterns of §7 and the worked example of §11 matter as much as the code.

---

## 3. The Platforms: Where Smart Contracts Run

### 3.1 Ethereum: Frontier, 30 July 2015

Ethereum was conceived by **Vitalik Buterin**, who published the Ethereum whitepaper in late 2013 (November 2013 is the commonly cited date; ethereum.org describes the network as "founded by Vitalik Buterin in 2013"). The founding team is usually listed as Buterin plus **Gavin Wood** (who wrote the Ethereum Yellow Paper and proposed Solidity), **Charles Hoskinson**, **Anthony Di Iorio**, and **Joseph Lubin** (other early contributors appear in some "co-founder" rosters ⚠). Development was crowdfunded in 2014 (the ether presale raised over US$18M in Bitcoin, per the commonly cited figure ⚠), and the network went live on **30 July 2015** with the mining of the genesis block under the release codename **Frontier**. ✅ Verified against ethereum.org's history page ("The Ethereum network officially launched on July 30, 2015, with the mining of the first block") and consistent across Wikipedia, Ledger Academy, and the blockchain-history literature.

The Frontier launch is the boundary that matters for this guide: from 30 July 2015, anyone could deploy a smart contract to a live, permissionless, economically secured network. The timeline that followed is the subject matter of the rest of this guide: the DAO and its fork (2016, §8.1), the standards wave (ERC-20 in 2015, ERC-721 and ERC-1155 in 2018, ERC-4626 in 2021–22, §5), the tooling (Truffle 2015, Hardhat, Foundry, §6), and the incident record (§9).

### 3.2 The EVM and the Contract Account

On Ethereum, there are two kinds of accounts. An **EOA** (externally owned account) is a public-private keypair — a wallet; it can initiate transactions. A **contract account** is an address that holds code (the compiled contract bytecode) and state (its storage), and can only act when a transaction or a call from another contract activates it. The **Ethereum Virtual Machine (EVM)** executes that bytecode; execution is priced in **gas**, a fee paid in the native asset (ETH) that bounds computation and prevents infinite loops and denial-of-service. The **ABI** (application binary interface) is the JSON description of the contract's functions and events that lets off-chain clients encode calls and decode results. The precise mechanics of blocks, consensus, and the EVM's gas model are ledger-level topics covered in [blockchain_technology_guide.md](blockchain_technology_guide.md) §3–§4; this guide only needs the account model as vocabulary.

Two EVM properties do most of the work in the rest of this guide:

- **`delegatecall`** — a contract can execute another contract's code *in its own storage context*. This is the mechanism behind proxy upgradeability (§7) and behind a large family of storage-collision bugs.
- **Determinism and atomicity** — a transaction either fully commits or fully reverts (state rolls back, gas is still paid). Reentrancy (§8.1) exploits the fact that within a transaction, an external call hands control to another contract *before* the caller's state updates are finalised.

### 3.3 Solidity: Wood's 2014 Proposal, Reitwiessner's Team, 2015 Release

Solidity is the dominant smart-contract language. Its origin, verified: proposed in **August 2014 by Gavin Wood**, then developed by the Ethereum project's Solidity team **led by Christian Reitwiessner**, with the first release in **2015** (v0.1.0, August 2015 per the project's release history as recorded in ecosystem references). ✅ The 2014 proposal and Reitwiessner's leadership are attested by Wikipedia's Solidity article and consistent across Springer, the blockchain-history literature and the Solidity project's own documentation lineage. The exact first-release date is a minor discrepancy across sources (v0.0.1 versioning reportedly began July 2015; v0.1.0 is dated 7 August 2015) ⚠ — the safe statement is "first released in 2015."

Solidity is compiled to EVM bytecode; it is C/JavaScript-flavoured, contract-oriented, and ships with a standard library ecosystem (OpenZeppelin Contracts being the de-facto reference implementations of the ERC standards). Two language-level facts matter for security: **checked arithmetic** — since Solidity 0.8.0 (2020), integer overflow/underflow reverts by default, which eliminated the SWC-101 class for new code (see §8.2); and **revert semantics** — `require` for input validation, `assert` for invariants (misuse of `assert` is itself a weakness class, SWC-110). The other notable EVM language is **Vyper**, a deliberately minimal Python-flavoured language — which made history in July 2023 when a compiler bug in three Vyper versions broke reentrancy locks and led to the ~US$61M Curve exploit (⚠ figure; §8.1, §9).

### 3.4 Other Chains and Permissioned Platforms

Ethereum is the reference platform but not the only one; the ledger mechanics of the alternatives are covered in [blockchain_technology_guide.md](blockchain_technology_guide.md) and are not re-derived here. The one-line map a banker needs:

| Platform | Contract model | Notes |
|---|---|---|
| Ethereum (+ EVM L2s) | Solidity/Vyper → EVM bytecode | The standards centre of gravity; every ERC in §5 originated here |
| Solana | Rust/C → BPF bytecode, "programs" | High throughput; account model differs sharply from EVM |
| Cardano | Plutus (Haskell) — eUTxO model | Scripts validated against UTxO constraints, not account-state |
| Hyperledger Fabric | "Chaincode" (Go/Java/JS) | Permissioned; endorsed and ordered by the network's members |
| R3 Corda | CorDapps (Kotlin/Java) | Permissioned; designed for regulated financial workflows, not a global ledger |
| Quorum | EVM bytecode | Ethereum fork engineered for permissioned consortia |

The permissioned platforms matter to banks because they remove the two features banks are most uncomfortable with — permissionless participation and pseudonymity — while keeping the deterministic-execution core. The honest trade-off, discussed in the umbrella guide, is that permissioned chains sacrifice the censorship-resistance and public-verifiability properties that make public contracts interesting in the first place. Most institutional pilots (including the Project Guardian work of §10.5) run on permissioned or semi-permissioned EVM environments precisely so that whitelisting, KYC/AML screening and regulatory access can be enforced at the platform layer rather than inside every contract.

### 3.5 The Cross-Chain Reality: Bridges Are the Hack Surface

The final platform-level fact is the one the incident record (fully tabulated in §9) makes undeniable: **cross-chain bridges are the largest hack surface in the smart-contract economy.** A bridge is a set of contracts on two (or more) chains plus a relaying mechanism; the value it locks is a standing target, and its security depends on the weakest of many components — signature verification, keeper/validator sets, message authentication, upgrade governance. The four largest DeFi thefts through 2023 were all bridge-adjacent: **Poly Network** (Aug 2021, ~US$610M — later returned), **Ronin** (Mar 2022, ~US$625M — compromised validator keys), **Wormhole** (Feb 2022, ~US$320M — signature-verification bypass), and **Nomad** (Aug 2022, ~US$190M — a botched upgrade that let anyone replay a forged message). The umbrella guide's §8 treats bridge architecture and trust models; this guide's §9 treats the failure classes. The banking takeaway is conservative and simple: *a tokenized-deposit or fund product does not need a bridge*, and every design decision that adds one should be treated as adding an order of magnitude of attack surface.


## 4. The Standards Process: EIP-1 and the EIP Lifecycle

### 4.1 EIP-1, the Process Document

Standards on Ethereum are proposed through the **Ethereum Improvement Proposal (EIP)** process, defined by — appropriately — EIP-1, *EIP Purpose and Guidelines*. Verified directly at eips.ethereum.org: EIP-1's authors are **Martin Becze and Hudson Jameson, et al.**, created **27 October 2015**, and the document is itself a "Living" Meta-track EIP that has been amended repeatedly since. ✅ EIP-1 acknowledges its own lineage: it was derived from Bitcoin's BIP-0001 (Amir Taaki), which was in turn derived from Python's PEP-0001. An EIP that reaches the **ERC** (Ethereum Request for Comments) category is an application-level standard — the token standards of §5 are all ERCs.

The EIP taxonomy, per EIP-1:

| Type | Purpose | Examples |
|---|---|---|
| Standards Track: Core | Consensus-level changes | EIP-170 (contract size limit), EIP-1559 (fee market) |
| Standards Track: Networking | Network protocol | — |
| Standards Track: Interface | Client API/ABI conventions | EIP-712 (typed data signing) |
| Standards Track: ERC | Application-level standards | ERC-20, ERC-721, ERC-1155, ERC-4626, ERC-1967 |
| Meta | Process documents | EIP-1 itself |
| Informational | Design guidelines | EIP-1470 (SWC classification) |

The registry this guide relies on for §8 — the **Smart Contract Weakness Classification (SWC)** — is itself defined by an EIP: **EIP-1470**, *Smart Contract Weakness Classification (SWC)*, described as "a smart contract specific software weakness classification scheme for developers, tool vendors and security practitioners," loosely aligned with MITRE's Common Weakness Enumeration (CWE).

### 4.2 The Lifecycle: Draft → Review → Last Call → Final

EIP-1 defines the status flow for standards-track EIPs. The current lifecycle (simplified from EIP-1's work-flow section):

```
Idea → Draft → Review → Last Call → Final
                ↑         │
                └── Rejected / Withdrawn / Stagnant ──┘
```

| Status | Meaning |
|---|---|
| **Draft** | The proposal is written up and open for discussion (Ethereum Magicians forum, then the EIPs GitHub repo). |
| **Review** | EIP editors and the community review the specification; substantial changes expected. |
| **Last Call** | Final comment period before a decision. |
| **Final** | Accepted and deployed / standardised. For ERCs, "Final" means the standard is ratified and implementations can be certified against it. |
| **Stagnant** | No activity for six months or more; can be resurrected. |
| **Withdrawn / Rejected** | The author or editors ended the process. |

Two subtleties matter for a bank reading a standard's status. First, **Final is not frozen**: EIP-1 is itself "Living", and standards can be superseded (ERC-20's approval model is widely criticised but too entrenched to change — §5.1). Second, **status ≠ adoption**: ERC-1822 (UUPS, §7.2) is a widely deployed pattern whose EIP is officially **Stagnant**, while ERC-20, ERC-721, ERC-1155 and ERC-4626 are all **Final** — verified from their EIP pages' status badges this pass. The practical rule: check the status *and* the implementations, not just the badge.

### 4.3 What the ERC Designation Buys

An ERC is a promise of interface compatibility: any wallet, exchange, custody system or protocol that speaks ERC-20 can hold, transfer, and account for any ERC-20 token without bespoke integration. That is the entire economic point of the standards layer, and it is why the token standards of §5 are the load-bearing wall of institutional tokenization: when a bank issues a tokenized deposit or fund share, the *issuer's* contract is one deployment, but the *ecosystem's* acceptance of it is inherited from the standard. The flip side, developed in §5 and §8, is that standards also inherit *pitfalls* — the ERC-20 infinite-approval hazard, the ERC-4626 share-price manipulation surface — so a bank must implement the standard *and* the hardening around it.

## 5. The Token Standards: ERC-20, ERC-721, ERC-1155, ERC-4626

### 5.1 ERC-20: The Fungible Token

**ERC-20, *Token Standard*** — the interface for fungible tokens (every unit identical, divisible, interchangeable). Verified directly at eips.ethereum.org: authors **Fabian Vogelsteller and Vitalik Buterin**, created **19 November 2015**, status **Final**. ✅ It is the most widely implemented smart-contract interface in existence: virtually every stablecoin, every DeFi governance token, and — critically for this guide — most tokenized-deposit and tokenized-fund pilots are ERC-20s or extensions of one.

The mandatory surface (per the EIP's specification):

| Function | Purpose |
|---|---|
| `totalSupply()` | Total tokens in existence |
| `balanceOf(owner)` | Balance of an account |
| `transfer(to, value)` | Move tokens; MUST fire `Transfer` |
| `transferFrom(from, to, value)` | Move tokens on behalf of `from` (subject to allowance) |
| `approve(spender, value)` | Authorise `spender` to move up to `value` |
| `allowance(owner, spender)` | Read the authorised amount |

Plus two events — `Transfer(from, to, value)` and `Approval(owner, spender, value)` — and optional metadata functions `name()`, `symbol()`, `decimals()`. A minimal illustrative implementation, in the spirit of the EIP's own reference style (not production code):

```solidity
// SPDX-License-Identifier: MIT
// ILLUSTRATIVE ONLY — see OpenZeppelin for a hardened implementation.
pragma solidity ^0.8.0;

contract MinimalToken {
    mapping(address => uint256) public balanceOf;
    mapping(address => mapping(address => uint256)) public allowance;
    uint256 public totalSupply;

    event Transfer(address indexed from, address indexed to, uint256 value);
    event Approval(address indexed owner, address indexed spender, uint256 value);

    function transfer(address to, uint256 value) external returns (bool) {
        balanceOf[msg.sender] -= value;      // underflow reverts on Solidity >=0.8
        balanceOf[to] += value;
        emit Transfer(msg.sender, to, value);
        return true;
    }

    function approve(address spender, uint256 value) external returns (bool) {
        allowance[msg.sender][spender] = value;
        emit Approval(msg.sender, spender, value);
        return true;
    }

    function transferFrom(address from, address to, uint256 value) external returns (bool) {
        allowance[from][msg.sender] -= value;
        balanceOf[from] -= value;
        balanceOf[to] += value;
        emit Transfer(from, to, value);
        return true;
    }
}
```

The **allowance model** is the design feature every integrator must understand. `transferFrom` lets a third party (a DEX, a bridge, a vault) pull tokens from an owner's balance up to the approved allowance — this is what makes DeFi composability possible. Its two canonical pitfalls:

- **The infinite-approval hazard.** User-facing applications habitually request `approve(spender, maxUint256)` ("unlimited" approval) to avoid re-approving on every trade. The user then carries a standing, revocable-but-rarely-revoked authorisation to the spender contract; if that spender is compromised (or is malicious), every approved token is exposed. Banking posture: finite, per-transaction or per-session allowances, and `approve(0)` revocation flows in the product UI.
- **The race on `approve`.** The EIP does not specify behaviour when `approve` is called twice concurrently; the classic mitigation is `increaseAllowance`/`decreaseAllowance` (as in OpenZeppelin's implementation) so that a spender cannot sandwich a re-approval. Modern implementations increasingly bypass the whole model with **EIP-2612 `permit`** — a signed, gasless approval — but EIP-2612 is an extension, not part of ERC-20 itself.

Other ERC-20 integration hazards worth a line each: **missing-return-value tokens** (USDT famously returns nothing on success; a strict `require(token.transfer(...))` will revert against it — use the SafeERC20 wrapper); **fee-on-transfer and rebasing tokens** (balance changes outside the caller's accounting); and **transfer hooks** (ERC-777, which had its own reentrancy incident — the 2018 Uniswap/imBTC exploit — and is now largely deprecated in favour of ERC-20 + ERC-165 receiver patterns).

### 5.2 ERC-721: The Non-Fungible Token

**ERC-721, *Non-Fungible Token Standard*** — the interface for unique, non-interchangeable assets ("deeds" in the EIP's own language). Verified directly at eips.ethereum.org: authors **William Entriken, Dieter Shirley, Jacob Evans, Nastassia Sachs**, created **24 January 2018**, status **Final**, and it **requires EIP-165** (interface detection). ✅ The EIP's motivation is explicit: ERC-20 is insufficient because "each asset is distinct (non-fungible) whereas each of a quantity of tokens is identical (fungible)" — houses, artwork, loans, collectibles. CryptoKitties is listed among the early implementations; Dieter Shirley is a CryptoKitties (Axiom Zen) founder.

The surface: `balanceOf(owner)`, `ownerOf(tokenId)`, `safeTransferFrom(from, to, tokenId, data)`, `transferFrom`, `approve`, `getApproved`, `setApprovalForAll`, `isApprovedForAll`, plus the `Transfer` event. The design differences from ERC-20 that matter:

- **Per-token ownership**, not balances — each NFT has an owner and an operator model (an "operator" can manage all of an owner's tokens, the NFT analogue of an allowance).
- **Safe transfers** — `safeTransferFrom` checks whether the recipient implements the ERC-721 receiver interface (via EIP-165) and reverts otherwise, preventing tokens being stranded in contracts that cannot handle them.
- **Metadata indirection** — `tokenURI` points to off-chain metadata (often an IPFS or HTTPS URL), which is where the NFT's "meaning" lives; this is the classic example of the oracle/off-chain-dependency problem of §2.5.

The banking relevance of ERC-721 is narrower than ERC-20's but real: it is the natural envelope for *unique* instruments — a specific syndicated loan participation, a specific bond ISIN, a specific art-backed collateral asset — where the token must carry identity, provenance and an individual metadata document rather than a fungible balance. Institutional NFT platforms (e.g., tokenized bond or structured-product pilots) typically use ERC-721 for the instrument and ERC-20 for the cash leg.

### 5.3 ERC-1155: The Multi-Token Standard

**ERC-1155, *Multi Token Standard*** — a single contract that manages *any number* of token types, fungible and non-fungible in one deployment. Verified directly at eips.ethereum.org: authors **Witek Radomski, Andrew Cooke, Philippe Castonguay, James Therien, Eric Binet, Ronan Sandford** (the Enjin team), created **17 June 2018**, status **Final**, requires **EIP-165**. ✅ The motivation is economic: where ERC-20 needs one contract per token type and ERC-721 one contract per collection, ERC-1155 packs many token IDs into one contract, with **batch transfers** (`safeBatchTransferFrom`) and per-ID balances (`balanceOf(account, id)`), and the EIP's cited motivation material claims ~80% gas reduction for batch operations versus the alternatives.

Its design commitments worth knowing: **safe-transfer rules** (the receiver must implement the ERC-1155 receiver interface, else revert), **single approval model** (`setApprovalForAll` covers every token ID in the contract), and **guaranteed log trace** (every balance change must be observable from events — a deliberate auditability property). For banking, ERC-1155 is the elegant answer to a *family* of related instruments on one contract — e.g., a fund platform where each share class is a token ID (money-market class, short-bond class, currency-hedged class) sharing one deployment, one approval surface and one audit trail. It is also the standard of choice for tokenized-deposit platforms that want deposit, fee and collateral tokens in a single registry.

### 5.4 ERC-4626: The Tokenized Vault

**ERC-4626, *Tokenized Vaults*** — the standard API for yield-bearing vaults: a contract that holds one underlying asset (an ERC-20) and issues its own ERC-20 **shares** representing a claim on a fraction of the vault's holdings. Verified directly at eips.ethereum.org: authors **Joey Santoro, t11s (transmissions11), Jet Jadeja, Alberto Cuesta Cañada, Señor Doggo (fubuloubu)**, created **22 December 2021**, status **Final**; it **requires EIP-20 and EIP-2612** (the latter optional in practice, for gasless approvals). ✅ The "Final" badge is on the primary page; the specific finalisation date is not shown there — ecosystem trackers date it to **2 March 2022** ⚠ (see §12.4). The motivation, per the EIP: tokenized vaults had "a lack of standardization leading to diverse implementation details," forcing every aggregator to write error-prone adapters per protocol.

The core surface:

| Function | Purpose |
|---|---|
| `deposit(assets, receiver)` | Deposit underlying, mint shares |
| `mint(shares, receiver)` | Mint exact shares, pull underlying |
| `withdraw(assets, receiver, owner)` | Burn shares, return exact underlying |
| `redeem(shares, receiver, owner)` | Burn exact shares, return underlying |
| `previewDeposit/previewMint/previewWithdraw/previewRedeem` | Simulate the result of each operation |
| `convertToShares(assets)` / `convertToAssets(shares)` | Exchange-rate reads (MUST round *down*) |
| `totalAssets()` | Underlying held by the vault |
| `asset()` | The underlying ERC-20 |

An illustrative minimal skeleton (not production code — a real vault needs access control, rounding discipline, pause and usually a proxy):

```solidity
// SPDX-License-Identifier: MIT
// ILLUSTRATIVE ONLY — omits access control, rounding and reentrancy guards.
pragma solidity ^0.8.0;

import {IERC20} from "./IERC20.sol";

contract MinimalVault {
    IERC20 public immutable asset;
    uint256 public totalShares;
    mapping(address => uint256) public sharesOf;

    constructor(address _asset) { asset = IERC20(_asset); }

    function totalAssets() public view returns (uint256) {
        return asset.balanceOf(address(this));
    }

    function convertToShares(uint256 assets) public view returns (uint256) {
        uint256 t = totalAssets();
        return t == 0 ? assets : assets * totalShares / t;   // 1:1 at inception
    }

    function deposit(uint256 assets, address receiver) external returns (uint256 shares) {
        shares = convertToShares(assets);
        asset.transferFrom(msg.sender, address(this), assets);
        sharesOf[receiver] += shares;
        totalShares += shares;
        emit Deposit(msg.sender, receiver, assets, shares);
    }

    event Deposit(address indexed caller, address indexed owner, uint256 assets, uint256 shares);
}
```

The **share-price manipulation surface** is the security story of ERC-4626, and it is the reason the standard's Security Considerations section is long. The vault's share price is `totalAssets / totalShares`; anything that moves either side without the other is an attack vector:

- **First-depositor / inflation attack** — the first depositor can receive 1 wei of shares for a large deposit, then a subsequent deposit rounds down to 0 shares, effectively donating value to the first depositor. Classic mitigations: *virtual shares* (mint a fixed dead amount at inception), minimum-deposit thresholds, or rounding up shares minted (as the EIP's own discussion recommends: integrators "may wish to mimic rounding up versions" of the convert functions).
- **Donation attacks** — anyone can `transfer` underlying directly to the vault, inflating `totalAssets` and diluting existing shareholders; vaults must use an internal accounting balance (assets actually deposited) rather than raw `balanceOf(address(this))`.
- **Preview-function manipulation** — if `previewDeposit` reads a manipulable price (an AMM spot price or a single oracle tick), an attacker can inflate/deflate the preview between the user's check and their transaction (slippage between preview and execution).
- **Reentrancy across the share/asset boundary** — the 2023 Curve/Vyper incident (read-only reentrancy) is the canonical modern case: a view function whose balance read is distorted mid-transaction (see §8.1).

The institutional significance of ERC-4626 is that it is the *first* token standard written for what banks actually do — a vault with deposits, withdrawals and a net asset value — rather than for trading collectibles. A tokenized money-market fund *is* an ERC-4626 vault (underlying: the fund's cash/securities position; shares: the fund units), which is exactly the design used in the Cymbal Bank worked example of §11. The standard's own motivation even names "lending markets, aggregators, and intrinsically interest bearing tokens" as the target.

### 5.5 The Standards Table

| EIP/ERC | Title | Authors (verified) | Created | Status | Banking use |
|---|---|---|---|---|---|
| EIP-1 | EIP Purpose and Guidelines | Martin Becze, Hudson Jameson, et al. | 2015-10-27 | Living | The process itself |
| ERC-20 | Token Standard | Fabian Vogelsteller, Vitalik Buterin | 2015-11-19 | Final | Tokenized deposits, stablecoins, fund shares |
| ERC-721 | Non-Fungible Token | Entriken, Shirley, Evans, Sachs | 2018-01-24 | Final | Unique instruments, art/loan collateral |
| ERC-1155 | Multi Token Standard | Radomski, Cooke, Castonguay, Therien, Binet, Sandford | 2018-06-17 | Final | Share-class families on one contract |
| ERC-4626 | Tokenized Vaults | Santoro, t11s, Jadeja, Cuesta Cañada, Señor Doggo | 2021-12-22 | Final | Money-market funds, yield products |
| ERC-1967 | Proxy Storage Slots | Palladino, Giordano, Croubois | 2019-04-24 | Final | Upgradeable contracts (§7) |
| ERC-1822 | UUPS | Barros, Gallagher | 2019-03-04 | Stagnant | Upgradeable contracts (§7) |
| ERC-712 | Typed Data Signing | Bloemen, Logvinov, Evans | 2017-09-12 | Final | Signature replay protection (§8.4) |
| EIP-170 | Contract Code Size Limit | Vitalik Buterin | 2016-11-04 | Final | Deployment constraint (§6.4) |

### 5.6 Why Standards Matter to a Bank

Three arguments, in increasing order of importance:

1. **Interoperability economics.** A token that speaks ERC-20 is immediately compatible with wallets, exchanges, custodians, and reporting tools that collectively represent years of engineering that the bank does not have to build or certify. Standards are the bank's shortcut to ecosystem liquidity.
2. **Auditability and control mapping.** Every ERC maps to a testable surface (the functions and events above), which means auditors, regulators and the bank's own risk function can reason about the instrument from its interface. An ERC-4626 vault has a *defined* deposit/withdraw/preview contract — the control objectives (correct pricing, no donation, no reentrancy) are expressible against the standard, as §11.4 demonstrates.
3. **The composability thesis.** The value of tokenized deposits and funds is not that they exist in isolation but that they plug into settlement, collateral and liquidity rails — payment systems, money-market interactions, repo-like arrangements. Standards are the plug. This is the institutional tokenization thesis of [tokenized_assets_guide.md](../banking/tokenized_assets_guide.md), and the reason the standards layer is where this guide puts its deepest technical weight.


## 6. The Development Lifecycle: Frameworks, Testing, Deployment

### 6.1 The Frameworks

The toolchain story, verified: **Truffle** — created in 2015 by Tim Coulter, a ConsenSys developer, and the first widely used Ethereum development framework (compile, test, migrate); it ran under the ConsenSys umbrella for its whole life and is now in the ConsenSys archive as "being sunset." ✅ **Hardhat** — born as **Buidler** at EthWorks/Nomic Labs in 2018, renamed Hardhat in a 2019/2020 rebrand (the Nomic Foundation's own announcement, "Buidler has evolved: Introducing Hardhat," documents the rename; the exact announcement date is reported variously ⚠), and maintained by the Nomic Foundation; its local network with stack traces and mainnet forking made it the JavaScript-ecosystem standard. ✅ **Foundry** — released in December 2021 by **Paradigm**, from **Georgios Konstantopoulos** (now a Paradigm partner); a Rust-based toolbox whose components are **Forge** (build/test/fuzz, tests written in Solidity), **Cast** (chain interaction CLI), **Anvil** (local node) and Chisel (REPL). ✅ Verified against Paradigm's own launch post ("Introducing the Foundry Ethereum development toolbox", December 2021) and the Foundry docs.

| Framework | Origin (verified) | Language | Role today |
|---|---|---|---|
| Truffle | 2015, Tim Coulter, ConsenSys | JavaScript | Legacy; sunset/archived; Ganache (its local simulator) still widely used |
| Hardhat | Buidler 2018 → Hardhat (Nomic Labs → Nomic Foundation) | JavaScript/TypeScript | Standard JS toolchain; local node, mainnet forking, Etherscan verification plugins |
| Foundry | Dec 2021, Paradigm (G. Konstantopoulos) | Rust + Solidity tests | Fastest build/test/fuzz; Forge/Cast/Anvil/Chisel |
| Remix IDE | Browser-based (Ethereum Foundation-adjacent) | Solidity | Prototyping and teaching; not CI-grade |

For a bank, the framework choice is less important than the *pipeline* it enables: deterministic builds, reproducible tests, and source verification (publishing the source on the block explorer so the deployed bytecode can be checked against it — the on-chain equivalent of "the code matches the review").

### 6.2 The Lifecycle: Write → Compile → Test → Deploy → Verify

The canonical flow, with the control each step adds:

| Step | What happens | The control it adds |
|---|---|---|
| **Write** | Author the contracts (Solidity/Vyper) against the chosen ERC interfaces | Design review against §5 pitfalls; threat modelling before code |
| **Compile** | `solc` produces bytecode + ABI; build must be deterministic | Reproducible artifacts for audit and verification |
| **Test** | Unit/integration/fuzz suites on a local network | Behavioural proof against the spec (§6.3) |
| **Audit** | Independent review (human + tooling) | §8.5; gate before any real value |
| **Deploy** | Transaction to mainnet; constructor/initializer runs | Gas budgeting (§6.4); admin keys ceremony |
| **Verify** | Publish source to the block explorer; match bytecode | Public verifiability; the audit trail starts here |
| **Operate** | Monitoring, incident response, upgrade governance (§7) | The contract is now a production system |

The two steps banks most often underestimate are **verify** (an unverified contract is a black box that defeats the whole transparency argument) and **operate** (a contract is not a software release; it is a *position* with ongoing risk, monitored and governed like one).

### 6.3 Testing Practice: Unit, Integration, Fuzz, Formal

Testing smart contracts is unusual in one respect: the test environment is a *simulation of the production environment*, and the gap between the two (gas behaviour, MEV conditions, adversarial callers) is where bugs live. The layered practice:

- **Unit tests** — per-function behaviour, including every revert path, on a local EVM (Hardhat Network, Anvil, Ganache).
- **Integration tests** — multi-contract flows (deposit → invest → withdraw), often against a **fork of mainnet** so the contract is tested against real deployed dependencies (real stablecoins, real price feeds).
- **Fuzz / property-based testing** — random inputs against invariants ("the vault never mints shares without assets", "totalAssets never decreases on deposit"); Foundry's fuzzing and invariant testing, plus standalone fuzzers like Echidna, are the industry standard. Invariant testing is where the ERC-4626 attacks of §5.4 are actually caught.
- **Formal verification** — proving properties mathematically (Certora's verification language, symbolic execution tools, KEVM/Halmos for EVM semantics). Expensive, but the right tool for the *math* of a vault: rounding, share pricing, and the deposit/withdraw invariants are provable properties.

The security-audit layer that wraps this practice — who audits, what an audit covers, bug bounties — is covered in the audit section (§8.5) and in depth in [cybersecurity_guide.md](cybersecurity_guide.md), which this guide cross-references rather than re-deriving.

### 6.4 Deployment: Gas, Addresses, and the EIP-170 Limit

Three deployment facts that shape every contract design:

1. **Gas is the pricing of computation.** Every EVM operation costs gas; the deployer pays for the contract creation, users pay for every call. Gas shapes design: storage is expensive, events are cheap, loops are dangerous (SWC-128, §8.2). Gas optimisation is an engineering discipline of its own, and the EIP-170 size limit below is a direct consequence of gas economics.
2. **Contract addresses are deterministic.** An address is derived from the deployer's address and a nonce (`CREATE`), or from a salt and code hash (`CREATE2`, EIP-1014) — which enables counterfactual deployments ("the address exists before the code does"). Address determinism matters for banking because counterparties can be pre-approved against an address that will hold the contract.
3. **EIP-170 caps contract size at 24 KB.** Verified directly from the EIP: *"Contract code size limit"*, author **Vitalik Buterin**, created **4 November 2016**, **Final**, activated in the **Spurious Dragon** hard fork at block 2,675,000. The parameter: `MAX_CODE_SIZE = 0x6000` = 24,576 bytes. If contract-creation initialisation returns more code than that, creation fails. ✅ The practical consequences: large contracts must be split (separate contracts, libraries, or diamond-style facets), and the *implementation* contracts of §7 are the ones that hit the limit — a proxy itself is tiny, but its logic contract can be size-constrained.

### 6.5 Oracle Integration

The oracle is the contract's only window on the world (§2.5), and its integration pattern is a design decision, not a plumbing detail:

| Feed type | How it works | Strengths | Weaknesses |
|---|---|---|---|
| Decentralised network (Chainlink-style) | Aggregated off-chain price feeds pushed on-chain by a node network | Manipulation-resistant, battle-tested | Trust in the node network; update latency |
| TWAP (time-weighted average price) | Computed on-chain from exchange data over a window | Fully on-chain, manipulation-resistant over long windows | Laggy; vulnerable to extreme short-window manipulation |
| Admin-pushed (banking) | Authorised role writes NAV/price after off-chain computation | Matches the regulated data pipeline (fund administrator → bank) | Centralised by design; the role is the trust anchor |

For a regulated fund or deposit product, the third pattern is the correct one: the NAV is computed by the fund administrator under the bank's data governance (see [market_data_consumption_guide.md](../banking/market_data_consumption_guide.md)), pushed on-chain by a permissioned role, and *consumed* by the vault contract via a read-only function — never fetched from a live market price. The oracle-manipulation risk class (feeding a false price to trigger a bad liquidation or a share-price distortion) is a standing item on every audit checklist; the mitigation is "no price, no execution": the contract must refuse to transact at an unvalidated or stale price.

## 7. Upgradeability: Proxy Patterns and Their Risks

### 7.1 Immutability versus Upgradeability

Deployed code is immutable: there is no "patch" on a blockchain. The tension is structural. Immutability is a *feature* — it is the source of the user's assurance that the contract will keep its promise (a tokenized-deposit holder can verify that the bank cannot silently change the withdrawal rules). But it is also a *liability* — bugs cannot be fixed, and the only alternatives are deploying a new contract (breaking continuity: balances, approvals and integrations all reset) or forking the network (absurd for a bank product). The industry answer is the **proxy**: a tiny, permanent contract that users interact with, which delegates execution to a **logic (implementation) contract** via `delegatecall` and can be pointed at a new logic contract by an authorised governance process. Users keep the same address and state; the behaviour is upgradeable.

The governance of the upgrade — who may point the proxy where, under what quorum, with what notice — is the actual control. A proxy with a single admin key is a central point of failure worse than the immutability it solves; the banking pattern is a multi-sig or a timelock-governed upgrade path with mandatory re-audit of each new implementation.

### 7.2 The Proxy Patterns: Transparent, UUPS, Beacon

Three patterns dominate, and their standards are verified:

**Transparent proxy (EIP-1967).** *"Proxy Storage Slots"*, authors **Santiago Palladino, Francisco Giordano, Hadrien Croubois** (OpenZeppelin), created **24 April 2019**, status **Final**. ✅ EIP-1967 standardises *where* a proxy stores its metadata — the logic-contract address slot, the beacon-contract address slot, and the admin address slot — using hash-derived slots that no compiler will ever allocate, so the proxy's own bookkeeping can never collide with the logic contract's storage. The *transparent* pattern (named by OpenZeppelin, not in the EIP) resolves function-selector clashes: when the caller is the admin, the proxy executes its own management functions; everyone else is delegated to the logic contract, so no function selector can be "hidden" from the admin.

**UUPS (EIP-1822).** *"Universal Upgradeable Proxy Standard"*, authors **Gabriel Barros, Patrick Gallagher**, created **4 March 2019**, status **Stagnant** (widely deployed regardless — status ≠ adoption, §4.2). ✅ UUPS moves the upgrade function *into the logic contract* (`upgradeTo` implemented by the implementation itself), so the proxy is a bare `delegatecall` shell with no management surface of its own — smaller attack surface, but the upgrade function must exist and be protected in every implementation, and a broken upgrade can leave the proxy permanently stuck.

**Beacon proxy.** A single **beacon contract** holds the current implementation address; many proxies each store *the beacon's* address (in the EIP-1967 beacon slot) and delegate through it. Upgrading the beacon upgrades every proxy that points at it — the right pattern for many identical instances (a whole family of fund vaults, a fleet of deposit contracts), where upgrading one-by-one would be operationally untenable. The beacon slot is part of EIP-1967's specification. ✅

| Pattern | Upgrade authority lives in | Best for | Main risk |
|---|---|---|---|
| Transparent (EIP-1967) | Proxy (admin role) | Single flagship contracts | Admin surface on the proxy; selector-clash rules add gas |
| UUPS (EIP-1822) | Logic contract | Lean proxies, frequent upgrades | Upgrade function lost/broken in an implementation = stuck |
| Beacon | Beacon contract (EIP-1967 slot) | Families of identical instances | Beacon is a single point of failure — guard it hardest |

### 7.3 The Risks: Storage Collision, Function Collision, Initialization

The proxy design space is where the most expensive smart-contract bugs live, because the proxy *shares storage with its logic contract* (that is the point of `delegatecall`). The canonical risk classes:

- **Storage collision.** The proxy's variables and the logic contract's variables occupy the same storage namespace; a state variable declared in the wrong position silently overwrites another. EIP-1967's unstructured, hash-derived slots exist precisely to move proxy bookkeeping out of the way — but the *logic contracts* themselves must still respect layout discipline: never reorder, rename or retype state variables across upgrades. This is the classic "upgrade broke the vault" class, and it is what the Parity November 2017 disaster (§9) *looked* like at library level: the library's state was claimed and destroyed.
- **Function collision (selector clash).** Two functions with the same 4-byte selector (the first four bytes of `keccak256(signature)`) — one in the proxy, one in the logic — create ambiguity about which runs. The transparent pattern resolves it by admin-vs-user routing; the *diamond* (EIP-2535) and *clone* patterns have their own resolutions. Audit tools specifically fuzz for selector collisions between proxy and implementation.
- **Initialization.** Proxies cannot use constructors (the constructor's code runs in the *implementation's* context, not the proxy's); state must be set by an **initializer** function guarded by an `initialized` flag. The classic failure: a deployed-but-uninitialized implementation contract that an attacker initializes for themselves, turning the "template" into a weaponized contract (this was the root of the November 2017 Parity library takeover — anyone could claim the library because its initializer was unprotected).
- **Missing upgrade authorization.** The upgrade function without a proper access control is a total-loss bug: whoever can call `upgradeTo` owns everything the proxy holds. The Poly Network exploit of 2021 (§7.4, §9) was exactly this class at bridge level.

### 7.4 The Upgrade-Adjacent Lessons: Poly Network and Curve

Two incidents belong in the upgradeability section because they are *governance-of-change* failures as much as code failures:

**Poly Network (10 August 2021).** The cross-chain bridge lost ~US$610M ⚠ (later returned nearly in full). The technical class: an unguarded configuration/upgrade function in the bridge's cross-chain manager let the attacker replace the bridge's trusted keeper public keys, after which they could forge cross-chain messages authorising withdrawals of locked funds. The lesson: **every function that can change trust configuration — keeper sets, implementation addresses, verification logic — is an upgrade function and must carry the same multi-sig/timelock protection as an upgrade.** The DAO-fork-era lesson, restated for bridges: trust anchors are where the money concentrates, and their change-control is the control that matters.

**Curve / Vyper (30 July 2023).** The compiler itself was the failure: three Vyper versions (0.2.15, 0.2.16, 0.3.0) shipped a malfunctioning reentrancy lock, and several Curve pools compiled with them were drained for ~US$61M ⚠ (initial estimates ~US$47M; later tallies vary up to ~US$73M — see §12.4). The lesson: **upgrades are not the only way code changes under you — your dependencies change.** The supply chain (compiler, standard library, dependencies pinned in the build) must be treated as part of the audited surface, with pinned, hash-verified toolchain versions and re-audit triggers whenever a dependency bumps. For a bank: the same discipline as patching a core banking platform — change control, provenance, and regression testing — applied to the contract toolchain.


## 8. The Security Deep-Dive: Reentrancy and the Vulnerability Classes

The taxonomy used throughout this section is the **Smart Contract Weakness Classification (SWC) registry** (swcregistry.io, per EIP-1470, hosted under the SmartContractSecurity GitHub organisation). ✅ The individual entries cited below were verified directly from the registry this pass. Two caveats, verified from the registry's own banner: the SWC registry is **no longer actively maintained** (content "not thoroughly updated since 2020", with the EEA EthTrust Security Levels specification named as its maintained successor), and its original authorship is commonly credited to **ConsenSys Diligence / MythX** — a connection documented across ecosystem sources (MythX's detector documentation maps to SWC IDs) but not stated on the registry's own landing page ⚠ (see §12.4). The SWC IDs remain the *lingua franca* of contract audit findings, so they are used here with that maintenance caveat attached.

### 8.1 The Reentrancy Class: SWC-107 and the DAO

**SWC-107, "Reentrancy"**, is the class that opened the era. The registry's description, verified: *"One of the major dangers of calling external contracts is that they can take over the control flow. In the reentrancy attack (a.k.a. recursive call attack), a malicious contract calls back into the calling contract before the first invocation of the function is finished."* The registry's own remediation is exactly the two defences still taught today: the **Checks-Effects-Interactions pattern** (make all internal state changes *before* any external call) and a **reentrancy lock** (OpenZeppelin's `ReentrancyGuard`). ✅

The mechanism, concretely: a contract that sends value or calls out *before* updating its own balances lets the callee — which may be attacker-controlled — call back into the same function while the caller's state still shows the pre-withdrawal balance, repeating the withdrawal. The canonical fix pattern:

```solidity
// ILLUSTRATIVE ONLY — the pattern that matters.
function withdraw(uint256 amount) external nonReentrant {
    require(balances[msg.sender] >= amount, "insufficient balance");
    // 1) CHECK  — validated above
    // 2) EFFECTS — update state BEFORE any external interaction
    balances[msg.sender] -= amount;
    // 3) INTERACTIONS — external call last
    (bool ok, ) = msg.sender.call{value: amount}("");
    require(ok, "transfer failed");
}
```

**The DAO attack (17 June 2016)** is the canonical case and the reason the entire industry takes the class seriously. Verified facts: on **17 June 2016**, an attacker exploited a reentrancy-style vulnerability in The DAO's `splitDAO` function, draining approximately **3.6 million ETH** — about one-third of The DAO's funds — into a child contract. ✅ (Consistent across Gemini Cryptopedia, the blockchain-history literature, and Wikipedia; the date 17 June 2016 is uniform.) The DAO itself had raised roughly **US$150M in ETH from over 11,000 contributors** in mid-2016 ⚠ (reported figure, sources vary slightly). At the time of the hack the drained ETH was commonly reported as worth **~US$60M** ⚠ — the USD figure depends on the ETH price at the moment of valuation and varies across sources; the *ETH* figure (~3.6M) is the stable fact. The aftermath, verified: after a contentious debate, Ethereum **hard-forked on 20 July 2016** at block 1,920,000, restoring the stolen funds to the original contract; the minority chain that refused the fork became **Ethereum Classic**. ✅ The DAO attack is thus simultaneously: (a) the proof that reentrancy is a real, catastrophic class (SWC-107); and (b) the proof that "code is law" is not the whole story (§2.3) — the community chose to override executed code.

The class has since evolved: **read-only reentrancy** (the 2023 Curve/Vyper variant, §7.4) attacks *view* functions — a contract reads another contract's balance mid-transaction while that balance is transiently distorted, and prices a decision (e.g., a share price, a liquidation check) on the distorted value, without ever re-entering the victim's state-changing functions. Defences: internal accounting balances instead of raw `balanceOf` reads, reentrancy guards on *all* state-changing entry points, and the "no price, no execution" rule of §6.5.

### 8.2 The Other Classes, Mapped to SWC

The weakness classes a bank's audit checklist must cover, each verified against the registry this pass:

| SWC | Title (verified) | The failure | Mitigations |
|---|---|---|---|
| SWC-101 | Integer Overflow and Underflow | Arithmetic wraps around; balances and totals become wrong (the pre-0.8 era's #1 bug; SafeMath existed for this) | Solidity ≥0.8 checked arithmetic (reverts by default); SafeMath for older code; invariants tested by fuzzing |
| SWC-107 | Reentrancy | External call re-enters before state updates (§8.1) | Checks-Effects-Interactions; `nonReentrant` guards; internal accounting |
| SWC-115 | Authorization through tx.origin | `tx.origin` (the original transaction sender) used for auth — a phishing contract can call through the user | Use `msg.sender`; ban `tx.origin` in auth paths |
| SWC-106 | Unprotected SELFDESTRUCT Instruction | Anyone can destroy the contract and force its balance to an address | Remove `selfdestruct`; multi-sig if truly required (the registry's own remediation) |
| SWC-105 | Unprotected Ether Withdrawal | Missing access control lets anyone drain the contract's balance | Role-based access control (RBAC) on every value-moving function |
| SWC-110 | Assert Violation | A reachable `assert()` means a broken invariant or a misused assert (assert used for input validation) | `require` for inputs; `assert` only for true invariants |
| SWC-124 | Write to Arbitrary Storage Location | Attacker-controlled array/mapping index writes over sensitive storage (e.g., the owner field) | Bound all indices; never derive storage keys from user input |
| SWC-113 | DoS with Failed Call | One failing recipient reverts the whole payout loop | Push-over-pull payments (users withdraw, contract doesn't send); isolate failures |
| SWC-128 | DoS With Block Gas Limit | Unbounded loops/arrays exceed the block gas cap and brick the function | Bound iteration; batching; avoid loops over user-extensible arrays |
| SWC-114 | Transaction Order Dependence | Result depends on transaction ordering (the raw form of front-running) | Commit-reveal, or accept MEV exposure consciously (§8.3) |

Three notes on the table. First, the registry title for the selfdestruct class is **SWC-106** ("Unprotected SELFDESTRUCT Instruction") — the *research brief* behind this guide expected it under SWC-105, but SWC-105 is "Unprotected Ether Withdrawal"; the discrepancy is logged in §12.3. Second, several further classes are registry-documented but not tabulated here (SWC-119 shadowing, SWC-120 weak randomness, SWC-121 missing protection against signature replay, SWC-123 requirement violation) — the ten rows above are the ones a banking audit sees most often. Third, **access control** (RBAC) is the meta-class: most of the table's rows are "missing or wrong authorization" in disguise, which is why the permissioning layer of the worked example (§11) is treated as a first-class component.

### 8.3 MEV, Flash Loans, and Price Manipulation

Beyond code-level weaknesses sit the *economic* attacks, which exploit the environment rather than a bug:

- **MEV (maximal extractable value)** — the value a block proposer (or a searcher paying the proposer) can extract by ordering, inserting or censoring transactions. The classic forms are **front-running** (see a user's large trade, trade first, profit from the price move) and **sandwich attacks** (buy before, sell after). SWC-114's transaction-order dependence is the code-level seed of this. For banking products the exposure is limited but real: any contract whose execution price is set by market order flow (an AMM interaction, a token sale) is front-runnable; the mitigations are commit-reveal designs, private order flow, and — the banking answer — permissioned pools where only whitelisted counterparties transact.
- **Flash loans** — uncollateralised loans that must be repaid within the same transaction; they let an attacker command arbitrarily large capital for one atomic operation. Flash loans are not themselves attacks; they are the *amplifier* that turns a small price discrepancy or a small logic flaw into a nine-figure drain (Euler 2023: a ~US$197M exploit powered by flash-loaned capital, §9).
- **Price-manipulation attacks** — an attacker moves the price a contract reads (an AMM spot price, a single oracle, a manipulable TWAP window) and executes against the distorted value: draining a lending market with inflated collateral, or attacking an ERC-4626 vault through its preview functions (§5.4). The defence is the oracle discipline of §6.5 and the invariant tests of §6.3.

### 8.4 Signature and Approval Attacks

The off-chain-meets-on-chain boundary has its own class. The two mechanisms that matter:

- **EIP-712, "Typed structured data hashing and signing"** — verified: authors **Remco Bloemen, Leonid Logvinov, Jacob Evans**, created **12 September 2017**, **Final**. ✅ It standardises *what* is signed: a structured, human-readable, domain-separated message instead of a raw bytestring. Its purpose in the security stack is **replay protection**: the domain separator binds a signature to one chain and one contract (chain ID, verifying contract, version), so a signature cannot be replayed on another chain or against another contract. EIP-2612 `permit` (gasless ERC-20 approvals) is built on it.
- **The signature-replay failure mode** — without domain separation, a signature taken on one deployment is valid on another: the same code deployed on two chains (or two instances of the same fund) will accept the same signature twice. The audit check is "every signature is bound to its domain."
- **The approval attack surface** — the ERC-20 allowance model (§5.1) is itself an attack surface: phishing approvals, infinite approvals, and the approve-race. For banking custody the design rule is: approvals only to audited, whitelisted contracts; finite allowances; and revocation tooling in every wallet.

### 8.5 Audit Practice

The audit is the industry's quality gate, and its practice is documented in [cybersecurity_guide.md](cybersecurity_guide.md) — this guide cross-references rather than re-derives. The condensed version a bank needs:

- **What an audit is**: an independent review combining manual code review, automated static analysis (Slither, Mythril), fuzzing/invariant testing, and — for vaults and token contracts — formal verification, producing a findings report graded by severity (Critical/High/Medium/Low/Informational) with remediation guidance. Notable firms in the space include Trail of Bits, OpenZeppelin, ConsenSys Diligence, CertiK and others — a long tail of smaller specialists exists.
- **What an audit is not**: a guarantee. Audits find known classes well (the SWC table above is their checklist) and miss novel compositions; several of the §9 incidents (Poly Network, Nomad, Euler) passed audits. The industry answer is **defence in depth**: audit + bug bounty + monitoring + upgrade capability + insurance.
- **Bug bounties**: Immunefi is the dominant platform; bounties align attacker incentives with disclosure and have recovered or prevented material losses (the Wormhole "bounty" of US$10M in February 2022 is the famous case — the attacker was offered the bounty as a white-hat reward after the exploit).
- **Formal verification** is increasingly non-optional for the *math* components (vault rounding, share pricing): the property "for all valid inputs, deposit never decreases any existing shareholder's claim" is provable, and the ERC-4626 literature treats it as such.
- **The maintenance caveat**: the SWC registry's own banner says it has not been updated since 2020 and points to the EEA EthTrust Security Levels specification as the maintained successor — a bank's audit RFP should require findings mapped to SWC *and* checked against the newer specifications.

## 9. The Incident History: What the Hacks Teach

The table below consolidates the major incidents verified this pass. Two conventions: **amounts are USD estimates at the time of the incident** (⚠ — they move with crypto prices and vary across sources; the asset-denominated figure is the stable fact), and each row gives the *class* of vulnerability, which is the durable lesson.

| Date | Incident | Class of vulnerability | Amount (⚠ estimates) | Outcome / lesson |
|---|---|---|---|---|
| 17 Jun 2016 | **The DAO** | Reentrancy (SWC-107); recursive call in `splitDAO` | ~3.6M ETH ≈ US$60M ⚠ | Hard fork 20 Jul 2016 → ETH/ETC split; the class that defined the discipline (§8.1) |
| 19 Jul 2017 | **Parity multi-sig (I)** | Function-visibility bug — unprotected `initWallet` on the multi-sig library let the attacker become owner | ~150,000 ETH stolen ≈ US$30M ⚠ (sources cite 150k–153k ETH) | The fix was deployed; the *second* incident below is what froze the ecosystem |
| 6 Nov 2017 | **Parity multi-sig (II)** | Library `selfdestruct` — a user claimed the uninitialised library and killed it, freezing every dependent wallet (SWC-106 class) | ~513,774 ETH frozen ≈ US$150–300M ⚠ (at the time; ~US$1.5B at later ETH peaks) | No theft, but permanent freeze; the canonical "initialization + selfdestruct" double lesson (§7.3) |
| 10 Aug 2021 | **Poly Network** | Cross-chain message forgery via unguarded keeper/key-update function (upgrade-authorization class, §7.4) | ~US$610M ⚠ (reports US$610–613M) | Nearly all returned within days; largest DeFi theft at the time; trust-anchor change-control lesson |
| 2 Feb 2022 | **Wormhole** | Signature-verification bypass on the Solana side of the bridge | ~120,000 wETH ≈ US$320M ⚠ | Parent company Jump restored the funds; US$10M white-hat bounty offered; verification-logic lesson |
| 23/29 Mar 2022 | **Ronin** | Compromised validator keys — 5 of 9 signers; forged withdrawals (not a code bug but a key-management failure) | ~173,600 ETH + 25.5M USDC ≈ US$625M ⚠ | Key management and quorum design as security controls; bridge again |
| 1 Aug 2022 | **Nomad** | Botched upgrade set the trusted root to zero → any message "pre-proven"; crowdsourced replay by ~300 addresses | ~US$190M ⚠ | Upgrade-process failure; partial recoveries; replay amplification lesson (§7.4) |
| 13 Mar 2023 | **Euler Finance** | Flash-loan-amplified logic flaw: missing liquidity-status check in the "donate" feature (price/state inconsistency) | ~US$197M ⚠ | ~US$240M returned after negotiation (recovery exceeded the loss figure due to price moves, per Euler); flash-loan amplifier lesson (§8.3) |
| 30 Jul 2023 | **Curve / Vyper** | Compiler bug in Vyper 0.2.15/0.2.16/0.3.0 broke reentrancy locks → read-only reentrancy on pools | ~US$61M ⚠ (initial ~US$47M; tallies up to ~US$73M vary) | Supply-chain lesson: the compiler is part of the audited surface (§7.4) |

Read across the rows, the pattern is unambiguous and it is worth stating as the section's thesis: **the money is always lost through a trust boundary — a call into untrusted code (DAO), an unprotected initializer (Parity), a changeable trust anchor (Poly), a verification shortcut (Wormhole), a key ceremony failure (Ronin), an upgrade mistake (Nomad), a state-consistency gap (Euler), or a broken dependency (Curve).** None of the nine required a cryptographic break. Every one of them is a *process* failure — of state-update discipline, change control, key management, or dependency management — which is why the security answer for a bank is not "better code" alone but the full operating discipline of §6–§8.


## 10. The Banking Applications: Tokenized Deposits and Funds

### 10.1 The Tokenization Thesis

The institutional thesis, developed in full in [tokenized_assets_guide.md](../banking/tokenized_assets_guide.md), is that the *instrument* layer of banking — deposits, fund units, bonds, repo — can be expressed as tokens on a programmable ledger, with the smart contract supplying what the legacy stack lacks: atomic settlement, 24/7 operability, composable collateral, and a single verifiable record. The four products in order of near-term plausibility for a bank like Cymbal Bank:

| Product | Token shape | Contract core |
|---|---|---|
| **Tokenized deposit** | ERC-20 representing a deposit claim on the bank's balance sheet | Whitelisted ERC-20 with pause/revocation; 1:1 with the core-banking liability ledger |
| **Tokenized money-market fund** | ERC-4626 vault over the fund's portfolio; shares = fund units | Vault + admin-pushed NAV + redemption gating |
| **Tokenized securities** | ERC-721/ERC-1155 per instrument (bond ISIN, structured product) | Instrument registry + coupon/redemption logic |
| **Tokenized fund-of-funds / master-feeder** | Vaults composed of vaults | The [cayman_bvi_master_feeder_guide.md](../banking/cayman_bvi_master_feeder_guide.md) structure expressed as nested ERC-4626s |

The common contract core across all four: a **permissioning layer** (who may hold, transfer, transact), an **emergency-stop** (pause), an **admin/governance layer** (multi-sig upgrades, §7), and a **full event audit trail** (§11.5).

### 10.2 The Tokenized-Deposit Design

A tokenized deposit is the simplest honest token: the bank holds the deposit liability on its balance sheet (unchanged, in the core banking system), and issues an ERC-20 that is *only ever* a claim on that liability, redeemable 1:1 at the bank. The design constraints that make it safe:

- **The token is not the money; the ledger is.** The bank's general ledger remains the system of record; the token is a bearer-style presentation of a liability. Reconciliation between token supply and ledger balance is a daily control (§11.5).
- **The whitelist is the KYC boundary.** Transfer functions check a whitelist (or the platform enforces it); tokens cannot move to unscreened addresses — the on-chain expression of AML/KYC.
- **The pause is the resolution tool.** Regulatory freeze, suspected fraud, or a fork emergency all map to a `pause()` on the token contract, mirroring the bank's existing account-freeze powers.
- **No leverage, no interest on-chain.** Interest accrues off-chain (the bank's deposit system) and is paid as new tokens; the contract never re-prices the deposit — avoiding the entire price-manipulation surface of §5.4.

The Project Guardian pilots (below) demonstrated exactly this: tokenized deposits used as the settlement leg of tokenized-bond trades, with the deposit token moving atomically against the bond token in a single transaction.

### 10.3 Stablecoins and the Reap Context

Stablecoins are the same design family with a different issuer: an ERC-20 redeemable claim, backed by reserves, targeting a stable value. The payment-application layer built on them — stablecoin-collateralised corporate cards and cross-border settlement — is covered in [reap_global_guide.md](../banking/reap_global_guide.md) (cross-referenced, not re-derived here); the contract-side facts this guide adds are the standards and security ones: stablecoin reserves held in contracts that must resist the same attack classes as any vault (donation attacks, share-price distortion, upgrade governance), and the regulatory frameworks (§10.5) that now define what "backed" means. From a contract standpoint, the interesting recent development is **tokenized deposits as the regulated alternative**: same ERC-20 ergonomics as a stablecoin, but the issuer is a bank, the liability is a deposit, and the reserve question disappears — the balance sheet is the backing.

### 10.4 The JPMorgan Onyx Context

The institutional reference point for all of the above is JPMorgan's blockchain work — JPM Coin, the Onyx platform, and the Onyx Digital Assets tokenized-collateral capability — covered in depth in [jpmorgan_chase_software_systems_guide.md](../banking/jpmorgan_chase_software_systems_guide.md) and cross-referenced rather than re-derived here. The two facts this guide needs from that record: (1) JPM Coin (2019) is the longest-running institutional example of a deposit-like token used for internal and interbank settlement, proving the operational case; and (2) JPMorgan's Onyx unit was one of the three institutions in MAS's **Project Guardian** first industry pilot (with DBS and SBI Digital Asset Holdings), executing live FX and government-bond transactions with tokenized deposits and institutional-grade contracts on a permissioned network — the exact design pattern of §10.2 in production.

### 10.5 The Regulatory Posture: MAS

Two verified regulatory anchors frame everything a Singapore bank does in this space (the full regulatory treatment is in [mas_regulations_guidelines_guide.md](../banking/mas_regulations_guidelines_guide.md)):

- **Project Guardian.** MAS's initiative to test asset tokenization and institutional DeFi, **launched in May 2022** ✅ (verified via the SBI Digital Asset Holdings press release of 2 September 2022 announcing its participation, the Straits Times coverage of the first live pilot, and the MAS consultation documents). The first industry pilot — DBS, JPMorgan, SBI Digital Asset Holdings — executed live trades of tokenized bonds and deposits in a permissioned environment, exploring "the institutional trading of tokenised bonds and deposits to improve efficiency and liquidity in wholesale funding markets."
- **The single-currency stablecoin (SCS) framework.** MAS **finalised its stablecoin regulatory framework on 15 August 2023** ✅ (verified from the MAS media release of that date, "MAS Finalises Stablecoin Regulatory Framework", responding to the October 2022 public consultation) — one of the first comprehensive stablecoin-specific regimes globally. The framework's requirements (full backing by low-risk liquid assets, timely redemption at par, capital and disclosure standards) are the compliance spec for any Cymbal Bank stablecoin or tokenized-deposit product. Note on dating: some secondary sources reference an "August 2024" milestone — that refers to the framework's *commencement/legislative-effect* phase rather than the finalisation itself, which the primary release dates to August 2023 ⚠ (see §12.4).

The regulatory direction of travel is unambiguous: MAS treats tokenized deposits and stablecoins as *regulated instruments on a programmable ledger*, not as an unregulated DeFi space — which is precisely why the standards (§5), the security discipline (§8), and the audit trail (§11.5) of this guide are the compliance-critical parts of the build.

### 10.6 Why Standards Matter to a Bank

The argument, stated once and then demonstrated in §11: standards are the bank's *only* way to inherit ecosystem liquidity without inheriting ecosystem risk. A whitelisted ERC-20 deposit token is held by the same wallets, custody systems and reporting tools as every other ERC-20 — but its *transfer* surface is bank-controlled, its *pause* is bank-controlled, and its *issuance* is ledger-reconciled. The standard provides the interface; the bank provides the trust; the security record of §8–§9 provides the checklist. The three layers must never be conflated: **the standard is not the product, the contract is not the bank, and the code is not the law** (§2.3).

## 11. The Cymbal Bank Worked Example: A Tokenized Money-Market Fund Vault

### 11.1 The Design Fiction Frame

The following is an **illustrative design exercise** — design fiction built on the verified facts, standards and incident lessons of this guide. **Cymbal Bank** is the fictional-but-substantiated Singapore-based bank persona used across the sibling guides in this repository (persona conventions documented in [mas_regulations_guidelines_guide.md](../banking/mas_regulations_guidelines_guide.md)); the worked example is on its own books, not a real client, and no contract below is deployed or audited. The point of the exercise is to show how every element of §2–§9 composes into one coherent, bank-shaped design: the standards chosen, the threat model, the controls, and the operating model.

**The product.** Cymbal Bank Tokenized Money Market Fund ("Cymbal MMF"): a Singapore-regulated money-market fund (the master-feeder structure considerations of [cayman_bvi_master_feeder_guide.md](../banking/cayman_bvi_master_feeder_guide.md) apply to the *fund*, not the contract) whose units are tokenized as an ERC-4626 vault. Investors subscribe in Singapore dollars (tokenized as a deposit token, §10.2); the vault invests in short-dated, high-quality instruments; the NAV is computed daily by the fund administrator; units are redeemable on the standard MMF notice cycle. Everything on-chain is the *settlement* layer; the fund's constitution, prospectus and NAV policy are the legal layer (§2.3–§2.4).

### 11.2 The Contract Architecture

```
┌────────────────────────────────────────────────────────────────┐
│  Cymbal MMF — contract topology (all addresses on permissioned  │
│  EVM network; upgradeable via EIP-1967 proxies)                 │
├────────────────────────────────────────────────────────────────┤
│                                                                 │
│  User / Investor (whitelisted EOA or custody wallet)            │
│        │  deposit(SGD) / redeem(shares)                         │
│        ▼                                                        │
│  ┌─────────────────────────┐      ┌──────────────────────────┐  │
│  │  MMF Vault (ERC-4626)   │─────▶│  Deposit Token (ERC-20)  │  │
│  │  shares = fund units    │ pulls│  whitelisted SGD deposit │  │
│  └───────────┬─────────────┘      └──────────────────────────┘  │
│              │ NAV read                                           │
│              ▼                                                  │
│  ┌─────────────────────────┐      ┌──────────────────────────┐  │
│  │  NAV Oracle (admin-     │      │  Permissioning + Pause   │  │
│  │  pushed by administrator│      │  (whitelist, freeze,     │  │
│  │  role; stale-price gate)│      │  emergency stop)         │  │
│  └─────────────────────────┘      └───────────┬──────────────┘  │
│                                              │                  │
│  ┌─────────────────────────┐      ┌──────────▼──────────────┐  │
│  │  Upgrade governance     │◀────▶│  Admin Multi-Sig (3-of-5)│  │
│  │  (EIP-1967 transparent  │      │  + timelock             │  │
│  │  proxy, UUPS-style      │      │                         │  │
│  │  upgradeTo in logic)    │      └──────────────────────────┘  │
└────────────────────────────────────────────────────────────────┘
```

The components, mapped to the guide:

| Component | Standard/pattern | Role | Key § |
|---|---|---|---|
| Deposit token | ERC-20 (+ whitelist + pause extensions) | The cash leg: SGD deposit claims | §5.1, §10.2 |
| MMF vault | ERC-4626 (shares = fund units) | Subscription, redemption, NAV-based pricing | §5.4 |
| NAV oracle | Admin-pushed, stale-price gate, "no price, no execution" | Daily NAV from fund administrator | §2.5, §6.5 |
| Permissioning layer | RBAC + whitelist + freeze | KYC/AML boundary, regulatory freeze | §8.2 |
| Emergency stop | `pause()` on vault and token | Resolution tool, mirrors account-freeze powers | §10.2 |
| Proxies | EIP-1967 transparent proxy; beacon for share classes | Upgradeability with governance | §7.2 |
| Governance | 3-of-5 multi-sig + timelock on upgrades and pause | No single point of failure | §7.1, §7.4 |

### 11.3 The Standards Chosen and Why

- **ERC-4626 for the vault** — because the MMF *is* a tokenized vault: `deposit/redeem/preview*` maps exactly onto subscription/redemption/NAV-quote, and the ecosystem (custodians, reporting tools, aggregators) already speaks it. The share-price manipulation surface of §5.4 is addressed head-on: the vault uses **internal accounting balances** (no raw `balanceOf` reads), **virtual shares** at inception against the first-depositor attack, and **admin-pushed NAV** so `convertToShares/convertToAssets` can never read a manipulable market price.
- **ERC-20 for the deposit leg** — maximal interoperability for the cash side, with the bank's own hardening (whitelist, finite allowances only, no infinite-approval UX; §5.1).
- **EIP-1967 transparent proxy + timelock for upgrades** — the bank's change-control: every implementation change requires the 3-of-5 multi-sig, a timelock period (investor notice), and a fresh audit of the new implementation (§7).
- **EIP-712 for every off-chain signature** (e.g., redemption instructions, KYC attestations) — domain-separated so nothing replays across chains or deployments (§8.4).
- Deliberately **not** chosen: an AMM spot-price oracle (manipulable), ERC-777 (reentrancy history), and any bridge component (§3.5 — the product has no cross-chain leg; if one is ever added, it is a new risk review, not an extension).

### 11.4 The Security Review: Threats Mapped to SWC

The pre-audit threat model, as it would be handed to the audit firm (each row: threat → SWC class → mitigation):

| Threat | SWC class | Mitigation in the design |
|---|---|---|
| Reentrant deposit/withdraw | SWC-107 | `nonReentrant` on all state-changing entry points; internal accounting; effects-before-interactions |
| Share-price distortion via donation | — (vault-specific, §5.4) | Internal `totalDeposited` accounting; donations cannot inflate `totalAssets` used for pricing |
| First-depositor / inflation attack | — (vault-specific, §5.4) | Virtual shares at inception + minimum subscription; rounding discipline (shares round up on mint) |
| NAV oracle manipulation / staleness | SWC-114 family / oracle class | Admin-pushed NAV with stale-price gate: execution reverts if NAV older than the policy window; role separation (administrator ≠ vault operator) |
| Unauthorized upgrade / trust-anchor change | §7.3 "missing upgrade authorization" | Upgrade functions only via 3-of-5 multi-sig + timelock; proxy admin and implementation roles separated; Poly Network lesson applied (§7.4) |
| Uninitialized implementation takeover | §7.3 "initialization"; SWC-106-adjacent | `initialized` flag; implementation contracts deployed with a locked initializer; Parity lesson applied (§7.3) |
| Storage collision across upgrades | §7.3 | EIP-1967 unstructured slots for proxy bookkeeping; immutable storage-layout policy; upgrade checklist includes layout diff |
| Whitelist bypass / wrong auth | SWC-105/115 family | RBAC everywhere; `msg.sender` only (no `tx.origin`); freeze and pause roles distinct from upgrade roles |
| Arithmetic edge cases in NAV math | SWC-101 | Solidity ≥0.8 checked arithmetic + fuzz/invariant tests over the pricing functions; formal verification of share-price invariants |
| Signature replay | SWC-121 family / EIP-712 | Domain-separated EIP-712 signatures bound to chain ID + contract address; nonces; EIP-712 verified §8.4 |
| DoS via large redemption batch | SWC-113/128 | No unbounded loops: per-investor redeem (pull), batch cap on any admin operation |
| Compromised single admin key | — (operational) | 3-of-5 multi-sig with geographically separated key custody; key ceremony per the ERM guide's operational-risk framework ([enterprise_risk_management_guide.md](../banking/enterprise_risk_management_guide.md)) |

### 11.5 The Audit Trail and Middleware Integration

The contract emits events for everything that matters — `Deposit`, `Redeem`, `NavUpdated`, `Paused`, `Unpaused`, `WhitelistChanged`, `RoleChanged`, `ImplementationUpgraded` — and the bank's integration platform consumes them. This is the enterprise-integration pattern of [enterprise_middleware_integration_platform_guide.md](../technology/enterprise_middleware_integration_platform_guide.md): an event-indexer → message-bus → core-banking-adapter pipeline that turns on-chain events into the bank's ledger entries, investor statements, and regulatory reports. The daily controls:

1. **Token-supply reconciliation** — total vault shares + total deposit tokens vs the fund administrator's unit register and the core-banking liability ledger; any mismatch is a production incident, not a reconciliation variance.
2. **NAV event lineage** — every `NavUpdated` event links to the administrator's signed NAV file (off-chain hash referenced on-chain), so the audit trail runs from market data (see [market_data_consumption_guide.md](../banking/market_data_consumption_guide.md)) through the administrator to the executed redemption price.
3. **Upgrade and pause logging** — every proxy upgrade and every pause is timestamped, attributed to the multi-sig, and notified to investors per the product terms (the timelock makes this enforceable on-chain, not just by policy).
4. **Regulatory reporting** — MAS-facing reports (token supply, investor roster, redemption activity) generated from the event stream, giving the regulator the same single verifiable record the bank has.

The operating model completes the design-fiction picture: the contracts are the settlement engine; the middleware is the nervous system; the fund administrator, the multi-sig holders and the audit firm are the humans; and the ERM framework of [enterprise_risk_management_guide.md](../banking/enterprise_risk_management_guide.md) is the governance that decides when pause, freeze, or upgrade is the right response. That is the difference between a DeFi vault and a bank product: not the code, but the operating discipline wrapped around it.

---

## 12. The Claims Audit

This section consolidates every key fact asserted in the guide with its verification status and primary source. Convention: **✅** verified this pass against the named primary/authoritative source; **⚠** reported/approximate/contested; **❌** refuted or not found.

### 12.1 The Verified-Facts Table

| # | Claim | Status | Primary source verified this pass |
|---|---|---|---|
| 1 | Szabo's 1996 essay *Smart Contracts: Building Blocks for Digital Markets* exists, carries a 1996 copyright, and is "a partial rewrite of the article which appeared in Extropy #16" | ✅ | The surviving text at fon.hum.uva.nl (mirror of szabo.best.vwh.net) |
| 2 | The earlier *Smart Contracts* essay dates to 1994 | ✅ (date attested by multiple secondary sources; the original 1994 page is offline — see §12.4) | Blockchain-history literature, Cryptogloss, Blockchain Council |
| 3 | Ethereum launched 30 July 2015 (Frontier; genesis block) | ✅ | ethereum.org history page |
| 4 | Ethereum founders incl. Buterin, Wood, Hoskinson, Di Iorio, Lubin; whitepaper late 2013 | ✅ (exact co-founder roster varies ⚠) | ethereum.org; Wikipedia |
| 5 | Solidity proposed by Gavin Wood (Aug 2014); developed by team led by Christian Reitwiessner; first release 2015 | ✅ (exact 2015 release date varies ⚠) | Wikipedia (Solidity); Springer; project release history |
| 6 | ERC-20: Vogelsteller + Buterin, created 2015-11-19, Final | ✅ | eips.ethereum.org/EIPS/eip-20 |
| 7 | ERC-721: Entriken, Shirley, Evans, Sachs, created 2018-01-24, Final, requires EIP-165 | ✅ | eips.ethereum.org/EIPS/eip-721 |
| 8 | ERC-1155: Radomski, Cooke, Castonguay, Therien, Binet, Sandford, created 2018-06-17, Final | ✅ | eips.ethereum.org/EIPS/eip-1155 |
| 9 | ERC-4626: Santoro, t11s, Jadeja, Cuesta Cañada, Señor Doggo, created 2021-12-22, Final | ✅ (exact Final date ⚠ — see §12.4) | eips.ethereum.org/EIPS/eip-4626 |
| 10 | EIP-1: Becze, Jameson et al., created 2015-10-27, Living | ✅ | eips.ethereum.org/EIPS/eip-1 |
| 11 | DAO hack: 17 June 2016, ~3.6M ETH drained (~one-third of funds), hard fork 20 July 2016 (block 1,920,000) → Ethereum Classic | ✅ | Gemini Cryptopedia; blockchain-history literature; Wikipedia |
| 12 | DAO hack USD value ~US$60M at the time; DAO raised ~US$150M from 11k+ contributors | ⚠ (USD varies with ETH price; raise figure slightly varies) | crypto.news; Block Clarity Hub |
| 13 | SWC registry = implementation of EIP-1470; hosted at swcregistry.io / SmartContractSecurity org | ✅ | swcregistry.io; EIP-1470; GitHub |
| 14 | SWC-107 reentrancy; SWC-101 overflow; SWC-115 tx.origin; SWC-106 selfdestruct; SWC-105 unprotected withdrawal; SWC-110 assert violation; SWC-124 arbitrary storage write; SWC-113/114/128 DoS classes | ✅ (titles verified against registry pages) | swcregistry.io/docs/SWC-107, -101, -115, -106, -105, -110, -124, -113, -114, -128 |
| 15 | SWC registry origin credited to ConsenSys Diligence / MythX | ⚠ (documented in ecosystem sources; not stated on the registry landing page) | MythX/ConsenSys detector docs; see §12.4 |
| 16 | Truffle: 2015, Tim Coulter, ConsenSys | ✅ | ConsenSys/Truffle archives; ConsenSys Media history |
| 17 | Hardhat: born as Buidler (2018, EthWorks/Nomic Labs), renamed Hardhat, Nomic Foundation | ✅ (exact rename date ⚠) | Nomic Foundation blog; EthWorks/buidler repo |
| 18 | Foundry: Dec 2021, Paradigm, Georgios Konstantopoulos; Forge/Cast/Anvil | ✅ | Paradigm launch post; Foundry docs |
| 19 | EIP-1967 Proxy Storage Slots (logic/beacon/admin slots): Palladino, Giordano, Croubois, 2019-04-24, Final | ✅ | eips.ethereum.org/EIPS/eip-1967 |
| 20 | EIP-1822 UUPS: Barros, Gallagher, 2019-03-04, Stagnant | ✅ | eips.ethereum.org/EIPS/eip-1822 |
| 21 | EIP-170: 0x6000 = 24,576-byte code-size cap, Spurious Dragon, block 2,675,000 | ✅ | eips.ethereum.org/EIPS/eip-170 |
| 22 | EIP-712 typed-data signing: Bloemen, Logvinov, Evans, 2017-09-12, Final | ✅ | eips.ethereum.org/EIPS/eip-712 |
| 23 | MAS Project Guardian launched May 2022; first pilot DBS/JPMorgan/SBI DAH (tokenized bonds + deposits) | ✅ | SBI DAH press release (Sep 2022); Straits Times; MAS documents |
| 24 | MAS stablecoin framework finalised 15 Aug 2023 (SCS; response to Oct 2022 consultation) | ✅ (an "Aug 2024" commencement variant exists ⚠) | MAS media release, 15 Aug 2023 |

### 12.2 The Incident-Figure Table

| Incident | Status | Verified basis |
|---|---|---|
| Parity July 2017: ~150k ETH stolen (~US$30M then; sources 150k–153k) | ⚠ | OpenZeppelin postmortem; Zerofriction; SaferICO |
| Parity Nov 2017: ~513,774 ETH frozen (≈US$150–300M at the time; ~US$1.5B at later peaks) | ⚠ (ETH figure ✅ across sources; USD varies) | DF3NDR security literature; TechCrunch; Coin Bureau |
| Poly Network: ~US$610M (reports 610–613M), Aug 2021, returned | ⚠ | Chainalysis; Reuters; Wikipedia |
| Wormhole: ~120k wETH ≈ US$320M, 2 Feb 2022 | ⚠ | Chainalysis; CertiK; CNBC |
| Ronin: ~173,600 ETH + 25.5M USDC ≈ US$625M, Mar 2022 | ⚠ | Halborn; The Verge; Blockworks |
| Nomad: ~US$190M, 1 Aug 2022, crowdsourced replay | ⚠ | Immunefi analysis; TRM Labs |
| Euler: ~US$197M, 13 Mar 2023; ~US$240M recovered per Euler | ⚠ | Chainalysis; Euler blog; Cyfrin |
| Curve/Vyper: ~US$61M (initial ~US$47M; up to ~US$73M reported), 30 Jul 2023 | ⚠ | Quadriga; Cointelegraph; SmartContractAudit |

### 12.3 Corrections and Discrepancies

Three places where this guide's research corrected the brief or the common lore, all stated transparently:

1. **SWC-105 vs SWC-106.** The brief expected "SWC-105 unprotected selfdestruct"; the registry's SWC-105 is *Unprotected Ether Withdrawal*, and unprotected selfdestruct is **SWC-106** (verified from the registry; SWC-106's references even cite the Parity "I accidentally killed it" incident). The guide uses the registry's actual numbering (§8.2).
2. **Parity November 2017 figure.** The brief's "~280k ETH burned" is not supported by the sources; the consistent verified figure is **~513,774 ETH frozen** (permanently locked, not burned — no attacker took it). The USD range (~US$150–300M at the time) is the price-dependent part.
3. **MAS stablecoin framework dating.** The brief's "finalised Aug 2024" conflates finalisation (15 August 2023, per the MAS release of that date) with the later commencement/legislative-effect phase. The guide states finalisation as August 2023 and flags the 2024 variant.

### 12.4 What Could Not Be Verified

The following items could not be fully confirmed this pass and are marked accordingly in the text:

- **The 1994 Szabo essay page itself.** szabo.best.vwh.net is offline and was unreachable from this environment (the domain resolves to a private/internal address and blocks extraction). The 1994 dating rests on multiple independent secondary attestations and the provenance of the 1996 Extropy piece; the primary 1994 page could not be re-read directly. The 1996 essay *was* verified from the surviving mirror text.
- **The exact USD value of the DAO hack.** "~US$60M" is the commonly reported figure at the time; the value depends on the ETH price used, and sources differ. The ETH figure (~3.6M) is the verified stable fact.
- **Exact USD figures for all nine incidents in §9.** Every amount is an at-the-time estimate that varies across sources and with prices; asset-denominated figures (ETH, USDC) are the stable facts. The Curve/Vyper total is reported anywhere from ~US$47M to ~US$73M.
- **The exact date ERC-4626 reached Final.** The primary EIP page shows the Final badge but no date; ecosystem trackers date the finalisation to 2 March 2022 (multiple independent trackers agree, but the primary page does not display it).
- **The exact Hardhat rename date.** Nomic Foundation's "Buidler has evolved: Introducing Hardhat" announcement exists; sources date it 2019 or 2020, and this pass did not pin the exact date.
- **The precise Solidity first-release date.** v0.1.0 is dated 7 August 2015 in some references; others cite v0.0.1 versioning from July 2015. "First released in 2015" is the safe verified statement.
- **The exact Ethereum co-founder roster.** The five core names (Buterin, Wood, Hoskinson, Di Iorio, Lubin) are consistent across sources; the full "8 co-founders" lists vary and were not pinned.
- **Ethereum's 2014 presale total (~US$18M), the DAO's raise (~US$150M, 11k+ contributors), and Wormhole's US$10M bounty.** Reported in reputable sources but not re-verified against primary documents this pass.

## 13. The Glossary

| Term | Definition |
|---|---|
| **ABI** | Application Binary Interface — the JSON spec of a contract's functions/events used to encode calls and decode results |
| **Bytecode** | The compiled EVM machine code deployed at a contract address |
| **Gas** | The fee unit pricing EVM computation; bounded per block (gas limit) and per transaction |
| **EVM** | Ethereum Virtual Machine — the deterministic execution engine every node runs |
| **EOA** | Externally Owned Account — a keypair-controlled wallet account that initiates transactions |
| **Contract account** | An address holding bytecode + state; acts only when called |
| **Transaction** | A signed instruction (call, deployment) that changes state atomically |
| **Reentrancy** | Re-entering a function before its state updates complete (SWC-107) |
| **Checks-Effects-Interactions** | The pattern: validate → update state → then interact externally |
| **Proxy / delegatecall** | A permanent shell contract executing logic from an implementation in its own storage |
| **Upgradeability** | Changing a contract's behaviour by re-pointing a proxy (transparent, UUPS, beacon) |
| **Storage collision** | Two contracts (proxy + logic) overwriting the same storage slots |
| **Oracle** | A feed bringing off-chain data (prices, NAV) on-chain |
| **TWAP** | Time-Weighted Average Price — on-chain, manipulation-resistant price proxy |
| **Slippage** | The difference between the quoted/expected price and the executed price |
| **Whitelist** | A permissioned list of addresses allowed to hold/transact a token |
| **Multi-sig** | A wallet/contract requiring M-of-N signatures to act |
| **Timelock** | A delay between a governance decision and its execution |
| **EIP / ERC** | Ethereum Improvement Proposal / Ethereum Request for Comments (application-level standards) |
| **Token standard** | A ratified interface (ERC-20, ERC-721, ERC-1155, ERC-4626) that implementations must satisfy |
| **NFT** | Non-Fungible Token — a unique, indivisible token (ERC-721/ERC-1155) |
| **Allowance / infinite approval** | ERC-20 spend authorisation; "infinite" = maxUint256 standing approval |
| **Permit (EIP-2612)** | Gasless signed approval built on EIP-712 |
| **DAO** | Decentralised Autonomous Organisation — the 2016 fund that was hacked, and the organisation form |
| **Hard fork** | A network split on new consensus rules (ETH/ETC, 2016) |
| **MEV** | Maximal Extractable Value — value extracted via transaction ordering |
| **Flash loan** | An uncollateralised loan repaid within one transaction |
| **selfdestruct** | The EVM opcode that destroys a contract and sends its balance to an address (SWC-106) |
| **SWC** | Smart Contract Weakness Classification — the weakness taxonomy (per EIP-1470) |

## 14. Cross-References and the Closing Summary

**The sibling guides, and where this guide touches them:**

| Sibling guide | Relationship to this guide |
|---|---|
| [blockchain_technology_guide.md](blockchain_technology_guide.md) | The umbrella: ledger mechanics, consensus, EVM, L2s, bridges (its §5 is the ~70-line overview this guide expands; its §8 covers bridge architecture that §3.5/§9 reference) |
| [defi_guide.md](defi_guide.md) | The application layer (AMMs, lending) built on these contracts |
| [tokenized_assets_guide.md](../banking/tokenized_assets_guide.md) | The institutional tokenization twin — products, pilots, the asset side |
| [jpmorgan_chase_software_systems_guide.md](../banking/jpmorgan_chase_software_systems_guide.md) | JPM Coin / Onyx — the longest-running institutional deposit-token record |
| [reap_global_guide.md](../banking/reap_global_guide.md) | Stablecoin-collateralised cards — the payment layer on stablecoin contracts |
| [mas_regulations_guidelines_guide.md](../banking/mas_regulations_guidelines_guide.md) | The MAS regulatory posture and the Cymbal Bank persona conventions |
| [enterprise_risk_management_guide.md](../banking/enterprise_risk_management_guide.md) | ERM framing for the risk and governance sections (§8, §11.4) |
| [cybersecurity_guide.md](cybersecurity_guide.md) | Audit practice and secure development — cross-referenced, not re-derived (§8.5) |
| [enterprise_middleware_integration_platform_guide.md](enterprise_middleware_integration_platform_guide.md) | Event→bus→core-banking integration pattern (§11.5) |
| [market_data_consumption_guide.md](../banking/market_data_consumption_guide.md) | Oracle/data-feed governance into contracts (§2.5, §6.5, §11.5) |
| [cayman_bvi_master_feeder_guide.md](../banking/cayman_bvi_master_feeder_guide.md) | Fund structures for the fund-tokenization worked example (§10.1, §11.1) |

**Closing summary.** Smart contracts took forty years from Szabo's 1994 coinage to become the settlement layer of institutional finance: Ethereum made them real in 2015, the ERC standards made them composable, the SWC classification and the US$2B+ incident record made them auditable, and the MAS-era pilots made them bankable. The engineering discipline they demand is the discipline of this guide — standards chosen deliberately, change controlled by multi-sig and timelock, prices pushed by administrators rather than pulled from markets, and every promise expressed in code that an auditor can verify and a regulator can reconcile. The legal promise still lives in the prose; the operational promise lives in the controls; and the two meet in the contract, where a bank's word is compiled, deployed, and kept exactly once. Every thread of this guide — the essay, the standards, the hacks, the vault — resolves into the same place: the self-executing promise.

*Part of the jackliusr/research series · Technology/Banking Domain · Author: Jack Liu Shurui, Solution Architect*
