# Ethereum: The World Computer

**The Ethereum-Platform Protocol Monograph — from the 2013 Whitepaper and the 2015 Frontier Launch to the Merge and Proof-of-Stake, the Execution/Consensus Client Landscape, the Roadmap Eras (Surge, Scourge, Verge, Purge, Splurge), Proto-Danksharding Blobs, the Layer-2 Rollup Ecosystem, the Enterprise and Tokenization Context, and the Cymbal Bank Platform-Assessment Worked Example**

> **Author:** Jack Liu Shurui, Solution Architect
> **Context:** Technology Research — the Ethereum *platform/protocol* monograph of the repo's blockchain-crypto cluster; the sibling guides carry the layers this guide only maps (contracts, DeFi/NFT economy, Web3 umbrella)
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Last Updated:** September 2026
> **Companion guides (technology/, same folder):** [blockchain_technology_guide.md](blockchain_technology_guide.md) (the general ledger survey — hash chains, consensus taxonomy, blocks, L2/EVMs/bridges at survey depth; cross-ref for basics, do NOT re-derive) · [smart_contracts_guide.md](smart_contracts_guide.md) (the code layer — Solidity, the EIP/ERC standards process, ERC-20/721/1155/4626, proxy patterns, SWC security classes, the incident history; carries a *verified Ethereum code-level lineage* including origin dates, ERC standards and the DAO hack — inherited by reference here) · [web3_technologies_guide.md](web3_technologies_guide.md) (the umbrella — the DeFi/NFT economy, wallets, bridges, DAOs; cross-ref for the economy layer) · [defi_guide.md](defi_guide.md) (the DeFi application layer) · [cybersecurity_guide.md](cybersecurity_guide.md) (audit and secure-development context) · [enterprise_middleware_integration_platform_guide.md](enterprise_middleware_integration_platform_guide.md) (integration of chain platforms into bank middleware)
> **Companion guides (banking/, prefix `../banking/`):** [tokenized_assets_guide.md](../banking/tokenized_assets_guide.md) (the institutional tokenization twin — tokenized RWAs, tokenized deposits, custody vendors, Project Guardian detail) · [jpmorgan_chase_software_systems_guide.md](../banking/jpmorgan_chase_software_systems_guide.md) (JPM Coin / Onyx — the longest-running institutional Ethereum-lineage record) · [fireblocks_guide.md](../banking/fireblocks_guide.md) and [bitunix_guide.md](../banking/bitunix_guide.md) (custody infrastructure and the exchange layer) · [mas_regulations_guidelines_guide.md](../banking/mas_regulations_guidelines_guide.md) (the MAS posture, condensed cross-ref in §10; the repo's Cymbal Bank persona conventions live here) · [reap_global_guide.md](../banking/reap_global_guide.md) (stablecoin-collateralised cards on Ethereum-ecosystem stablecoins) · [enterprise_risk_management_guide.md](../banking/enterprise_risk_management_guide.md) (ERM framing for the risk and governance sections)

**How to use this guide:** Section 1 is the overview — the short answer, the key-facts table, and the explicit map of how this guide sits inside the repo's crypto cluster (it is the *platform/protocol* monograph; the code layer, the economy and the Web3 umbrella belong to the siblings). Section 2 is the origin, verified at primary sources: the 2013 whitepaper, the 2014 ether sale, the Yellow Paper, and the 30 July 2015 Frontier launch, plus the full network-upgrade timeline. Section 3 is the architecture — accounts, gas, the EVM, transactions, blocks and state — verified at ethereum.org. Section 4 is the client landscape — execution and consensus clients, and the client-diversity problem (⚠ volatile). Section 5 is the consensus history — the PoW era and the difficulty bomb, the Merge (verified 15 September 2022), and the PoS design in detail (validators, 32 ETH, queues, attestations, finality, slashing, proposer-builder separation). Section 6 is the roadmap eras — the Merge/Surge/Scourge/Verge/Purge/Splurge framing, with shipped upgrades and ⚠ speculative items separated. Section 7 is EIP-4844 proto-danksharding and blobs. Section 8 is the L2 ecosystem — optimistic vs ZK rollups, the canonical projects, and bridging (condensed; the web3 guide carries the full bridge treatment). Section 9 is the enterprise and tokenization context — permissioned Ethereum history (Quorum → Besu), tokenized real-world assets and Project Guardian, all condensed with cross-refs. Section 10 is the regulation — MAS condensed, the SEC spot-ETH-ETF approval (2024), MiCA, and global notes. Section 11 is the Cymbal Bank worked example — an illustrative platform assessment for a Cymbal Bank tokenization pilot, written from a solution architect's seat and clearly marked design fiction. Section 12 is the claims audit (✅/⚠/❌), including the "What Could Not Be Verified" record. Section 13 is the glossary. Section 14 is the cross-reference map and the closing summary. **Reading paths:** *Banking/architecture:* §1 → §9 → §10 → §11. *Protocol engineer:* §3 → §4 → §5 → §7. *Scaling/rollups:* §7 → §8 → §6. *In a hurry:* §1, §5, §6, §11, §14, and the glossary.

**Integrity convention.** Every factual claim in this guide carries one of three marks: **✅** verified this pass against a primary or authoritative source (named in the claims audit, §12); **⚠** flagged — reported, approximate, single-sourced, fast-moving, or contested (client-diversity percentages, TVL figures and validator counts move constantly; treat them as point-in-time snapshots); **❌** refuted or not found. Unmarked statements are domain-stable technical knowledge (what a Merkle Patricia trie is, what an attestation does) rather than research claims. Where a sibling guide carries a verified fact (e.g. the smart-contracts guide's verified Ethereum code-level lineage), this guide marks it ✅ and cites the sibling rather than re-verifying from scratch — the repo convention is that sibling-verified facts are inherited.

## Table of Contents

1. [The Overview](#1-the-overview)
   - 1.1 [The Short Answer](#11-the-short-answer)
   - 1.2 [The Key-Facts Table](#12-the-key-facts-table)
   - 1.3 [This Guide's Place in the Crypto Cluster](#13-this-guides-place-in-the-crypto-cluster)
2. [The Origin, Verified: 2013–2015 and Every Fork Since](#2-the-origin-verified-20132015-and-every-fork-since)
   - 2.1 [The Whitepaper and the Sale](#21-the-whitepaper-and-the-sale)
   - 2.2 [Frontier: 30 July 2015](#22-frontier-30-july-2015)
   - 2.3 [The Network-Upgrade Timeline, Verified at ethereum.org](#23-the-network-upgrade-timeline-verified-at-ethereumorg)
3. [The Architecture: Accounts, Gas, the EVM, Blocks, State](#3-the-architecture-accounts-gas-the-evm-blocks-state)
   - 3.1 [Accounts: EOAs and Contract Accounts](#31-accounts-eoas-and-contract-accounts)
   - 3.2 [Gas: The Meter of Computation](#32-gas-the-meter-of-computation)
   - 3.3 [The EVM](#33-the-evm)
   - 3.4 [Transactions and Blocks](#34-transactions-and-blocks)
   - 3.5 [State and Finality](#35-state-and-finality)
4. [The Client Landscape: Execution and Consensus](#4-the-client-landscape-execution-and-consensus)
   - 4.1 [The Two-Client Architecture](#41-the-two-client-architecture)
   - 4.2 [The Execution-Client Roster](#42-the-execution-client-roster)
   - 4.3 [The Consensus-Client Roster](#43-the-consensus-client-roster)
   - 4.4 [Client Diversity: The Supermajority Problem ⚠](#44-client-diversity-the-supermajority-problem-)
5. [The Consensus History: PoW, the Difficulty Bomb, the Merge, PoS](#5-the-consensus-history-pow-the-difficulty-bomb-the-merge-pos)
   - 5.1 [The PoW Era and the Difficulty Bomb](#51-the-pow-era-and-the-difficulty-bomb)
   - 5.2 [The Beacon Chain and the Merge](#52-the-beacon-chain-and-the-merge)
   - 5.3 [The PoS Design: Validators, Queues, Attestations, Finality](#53-the-pos-design-validators-queues-attestations-finality)
   - 5.4 [Slashing, Exits and Withdrawals](#54-slashing-exits-and-withdrawals)
   - 5.5 [MEV and Proposer-Builder Separation](#55-mev-and-proposer-builder-separation)
6. [The Roadmap Eras: Surge, Scourge, Verge, Purge, Splurge](#6-the-roadmap-eras-surge-scourge-verge-purge-splurge)
   - 6.1 [The Framing, Verified](#61-the-framing-verified)
   - 6.2 [The Shipped Post-Merge Upgrades](#62-the-shipped-post-merge-upgrades)
   - 6.3 [The Speculative Items ⚠](#63-the-speculative-items-)
7. [EIP-4844 and Blobs: Proto-Danksharding](#7-eip-4844-and-blobs-proto-danksharding)
8. [The L2 Ecosystem: Rollups and Bridges](#8-the-l2-ecosystem-rollups-and-bridges)
   - 8.1 [How Rollups Work](#81-how-rollups-work)
   - 8.2 [The Canonical Projects, Verified](#82-the-canonical-projects-verified)
   - 8.3 [Bridging, Condensed](#83-bridging-condensed)
   - 8.4 [The TVL Caveat ⚠](#84-the-tvl-caveat-)
9. [The Enterprise and Tokenization Context](#9-the-enterprise-and-tokenization-context)
   - 9.1 [Permissioned Ethereum: Quorum to Besu](#91-permissioned-ethereum-quorum-to-besu)
   - 9.2 [Tokenized Real-World Assets and Project Guardian, Condensed](#92-tokenized-real-world-assets-and-project-guardian-condensed)
10. [The Regulation, Condensed with Cross-References](#10-the-regulation-condensed-with-cross-references)
    - 10.1 [MAS: The Home-Regulator Posture](#101-mas-the-home-regulator-posture)
    - 10.2 [United States: The Spot ETH ETF](#102-united-states-the-spot-eth-etf)
    - 10.3 [Europe: MiCA](#103-europe-mica)
11. [The Cymbal Bank Worked Example: A Tokenization Pilot Platform Assessment](#11-the-cymbal-bank-worked-example-a-tokenization-pilot-platform-assessment)
    - 11.1 [The Design-Fiction Frame](#111-the-design-fiction-frame)
    - 11.2 [The Decision: Public Ethereum versus Permissioned EVM](#112-the-decision-public-ethereum-versus-permissioned-evm)
    - 11.3 [The Assessment Matrix](#113-the-assessment-matrix)
    - 11.4 [The Recommendation and the Risk Register](#114-the-recommendation-and-the-risk-register)
12. [The Claims Audit](#12-the-claims-audit)
    - 12.1 [The Verified-Facts Table](#121-the-verified-facts-table)
    - 12.2 [Corrections and Discrepancies](#122-corrections-and-discrepancies)
    - 12.3 [What Could Not Be Verified](#123-what-could-not-be-verified)
13. [The Glossary](#13-the-glossary)
14. [Cross-References and the Closing Summary](#14-cross-references-and-the-closing-summary)

---

## 1. The Overview

### 1.1 The Short Answer

**Ethereum** is the first general-purpose blockchain platform: a single, shared, stateful virtual machine — the "world computer" — on which any party can deploy a program (a *smart contract*), pay for its execution in *gas*, and rely on a global validator network, not on any single counterparty, to run it exactly as written. It was conceived by **Vitalik Buterin** in a whitepaper dated **27 November 2013**, funded by a 42-day ether sale in **mid-2014**, specified in Gavin Wood's Yellow Paper (**April 2014**), and launched as the **Frontier** mainnet on **30 July 2015** ✅ (ethereum.org history; EF blog).

Two structural facts define the platform's history. First, Ethereum is *upgradeable by social consensus*: its rules change through coordinated network upgrades (hard forks), and the community has executed over twenty of them — including the 2016 DAO fork that split the chain into Ethereum and Ethereum Classic, and the September 2022 **Merge** that moved the whole network from proof-of-work mining to proof-of-stake. Second, Ethereum chose *rollups* as its scaling strategy: instead of sharding execution, the base layer (Layer 1, "L1") provides settlement and data availability, and Layer-2 networks ("L2s") execute transactions in bulk and post compressed proofs or data back to L1. The March 2024 **Dencun** upgrade (EIP-4844) gave L2s a dedicated cheap data lane — *blobs* — and the roadmap that follows is organised around finishing what Dencun started.

For a bank, the platform's honest summary is: Ethereum is the deepest-liquidity, most-deployed, most-audited public smart-contract platform (the venue where token standards were born), and its permissioned fork-lineage (Quorum → Hyperledger Besu) is the ancestor of most enterprise tokenization pilots — but public Ethereum is *not* a private settlement layer, L2 security models are not all equal, fees are volatile, and its roadmap items are intentions, not commitments. Section 11 turns all of that into a worked platform assessment for Cymbal Bank.

### 1.2 The Key-Facts Table

| Fact | Value | Status |
|---|---|---|
| Whitepaper | Vitalik Buterin, dated 27 November 2013 | ✅ §2.1 (ethereum.org history) |
| Ether sale | Began 22 July 2014, open 42 days, purchasable in BTC | ✅ §2.1 (ethereum.org history) |
| Mainnet launch | *Frontier*, 30 July 2015, 15:26:13 UTC — genesis block mined | ✅ §2.2 (ethereum.org history; EF blog "Ethereum Launches", 30 Jul 2015) |
| Consensus today | Proof-of-stake since The Merge, 15 September 2022 (last PoW block 15,537,393) | ✅ §5.2 (ethereum.org history) |
| Staking minimum | 32 ETH per validator; validator accounts >1.2 million as of April 2026 ⚠ (time-varying) | ✅/⚠ §5.3 (ethereum.org) |
| Time | 12-second slots; 32-slot epochs (~6.4 min); finality normally ~2 epochs (~12.8 min) | ✅ §5.3 (ethereum.org PoS docs) |
| Scaling headline | Dencun, 13 March 2024: EIP-4844 blobs for L2 data; blob target raised 3→6/9 at Pectra (7 May 2025); PeerDAS sampling shipped in Fusaka (3 December 2025) | ✅ §6–§7 (ethereum.org history/roadmap; eips.ethereum.org EIP-7594) |
| Client model | Every node runs two clients: execution (Geth, Nethermind, Besu, Erigon, Reth) + consensus (Lighthouse, Prysm, Teku, Nimbus, Lodestar, Grandine) | ✅ §4 (ethereum.org) |
| Client-diversity snapshot (Oct 2025) | Execution: Geth 41% / Nethermind 38% / Besu 16%; consensus: Lighthouse 42.7% / Prysm 30.9% / Teku 13.9% | ⚠ §4.4 (ethereum.org, volatile) |
| Token standards home | ERC-20 (2015), ERC-721 (2018), ERC-1155 (2018), ERC-4626 (2021) — dates verified in the smart-contracts guide | ✅ inherited (smart_contracts_guide.md §5) |
| Largest-ever protocol transition | PoW → PoS: energy use down ~99.95%; net ETH issuance cut by roughly 88% (reported) | ✅/⚠ §5.2 (ethereum.org; issuance figure reported) |
| Institutional milestone (US) | Spot ETH ETFs approved to list 23 May 2024; trading began 23 July 2024 | ✅ §10.2 (Reuters; SEC 19b-4 orders) |
| Institutional milestone (EU) | MiCA: Reg (EU) 2023/1114; stablecoin rules from 30 June 2024; full application 30 December 2024 | ✅ §10.3 (EUR-Lex; ESMA) |
| Enterprise lineage | JPMorgan Quorum (2016) → ConsenSys (Aug 2020) → Hyperledger Besu the surviving permissioned-EVM mainline | ✅ §9.1 (ConsenSys press release; Reuters) |

### 1.3 This Guide's Place in the Crypto Cluster

The repo's crypto cluster divides the subject by layer, and this guide is deliberately the *protocol/platform* monograph. The sibling boundaries, stated so readers cross-reference instead of re-deriving:

| Layer | Owned by | What this guide does instead |
|---|---|---|
| Ledger basics (hash chains, consensus taxonomy, blockchain types) | [blockchain_technology_guide.md](blockchain_technology_guide.md) | Cross-refs §3–§4 rather than re-explaining what a Merkle tree or a 51% attack is |
| Code layer (Solidity, ERC standards, EIP process, proxy patterns, SWC classes, incident history) | [smart_contracts_guide.md](smart_contracts_guide.md) | Cross-refs; inherits its verified Ethereum code-level lineage (§3.1 of that guide) instead of re-deriving |
| Economy (DeFi/NFT markets, wallets, bridges as user infrastructure) | [web3_technologies_guide.md](web3_technologies_guide.md) and [defi_guide.md](defi_guide.md) | Cross-refs the economy; the bridge *technology* appears here only as the condensed §8.3 |
| Exchanges, custody, payments companies | banking/[bitunix_guide.md](../banking/bitunix_guide.md), banking/[fireblocks_guide.md](../banking/fireblocks_guide.md), banking/[reap_global_guide.md](../banking/reap_global_guide.md) | Cross-refs in §9–§10 |
| Institutional tokenization (products, pilots, custody) | banking/[tokenized_assets_guide.md](../banking/tokenized_assets_guide.md) and [jpmorgan_chase_software_systems_guide.md](../banking/jpmorgan_chase_software_systems_guide.md) | Cross-refs; §9 keeps only the platform-side context |
| MAS/Singapore regulation | banking/[mas_regulations_guidelines_guide.md](../banking/mas_regulations_guidelines_guide.md) | Condensed cross-ref in §10.1 |

What remains uniquely here: the *origin and upgrade timeline* (§2), the *architecture* (§3), the *client landscape* (§4), the *consensus deep history and PoS design* (§5), the *roadmap eras* (§6), *EIP-4844 blobs* (§7), the *L2 rollup ecosystem* (§8), the *permissioned/platform enterprise context* (§9), and the *platform-selection worked example* (§11).

---

## 2. The Origin, Verified: 2013–2015 and Every Fork Since

### 2.1 The Whitepaper and the Sale

The origin record, verified at the primary timeline on ethereum.org and cross-referenced to the smart-contracts guide's verified lineage for the code-level history:

- **27 November 2013 — the whitepaper.** Ethereum.org's own history dates the introductory paper, published by Vitalik Buterin, to 27 November 2013, before the project's 2015 launch ✅. The whitepaper framed Ethereum as "an alternative protocol for building decentralized applications" — a blockchain with a built-in Turing-complete programming language on which anyone can write contracts and decentralized applications, a "single canonical machine" whose state everyone agrees on (the source of the "world computer" description).
- **2014 — specification and sale.** Gavin Wood's Yellow Paper, the formal technical definition of the protocol, was released on **1 April 2014** ✅ (ethereum.org history). The ether sale opened on **22 July 2014** for **42 days**, purchasable in BTC ✅ (ethereum.org history); the commonly reported result is roughly 60 million ETH sold for about US$18 million (31,531 BTC at then-prices) — **⚠ reported figures, not re-verified at a primary ledger this pass** (see §12.3). The founders' roster (Buterin, Wood, Hoskinson, Di Iorio, Lubin) is inherited from the smart-contracts guide's verified lineage ✅; exact "N co-founders" lists vary and are treated as ⚠ there and here.
- **The code-level history (Solidity, the first contracts, the EIP process) is NOT re-derived here** — the smart-contracts guide carries it: Solidity's 2014 proposal and 2015 release, ERC-20's creation in November 2015, and the EIP-1 standards process are all verified in [smart_contracts_guide.md](smart_contracts_guide.md) §3–§5 and inherited by reference ✅.

### 2.2 Frontier: 30 July 2015

The **Frontier** release launched the live network on **30 July 2015 at 15:26:13 UTC** when the genesis block was mined ✅ (ethereum.org history; the Ethereum Foundation blog's "Ethereum Launches" post of 30 July 2015 records the moment the genesis block was loaded). Frontier was deliberately bare-bones — "a live, but barebone implementation... intended for technical users, specifically developers" ✅ (ethereum.org). A **"Frontier thawing"** upgrade followed on 7 September 2015 (block 200,000), lifting a 5,000-gas-per-transaction cap and setting a 51-gwei default gas price so that normal transactions (which require 21,000 gas) could flow ✅ (ethereum.org history). Homestead, the first planned upgrade signalling production-readiness, activated on **14 March 2016** ✅.

The DAO fork of **20 July 2016** (block 1,920,000) is the platform's defining governance moment: after an insecure contract was drained of over 3.6 million ETH (17 June 2016), the community voted by carbonvote (over 85% of votes) to hard-fork and move the funds to a withdrawal contract; miners who refused the fork continued the original chain as **Ethereum Classic** ✅ (ethereum.org history; the hack's reentrancy mechanics and the code-level lesson are the smart-contracts guide's §8.1 territory — inherited, not re-derived).

### 2.3 The Network-Upgrade Timeline, Verified at ethereum.org

The full upgrade record below is verified this pass against ethereum.org's "Timeline of all Ethereum forks" page (execution upgrades are named for Devcon cities; consensus upgrades for stars; the combined short names like "Dencun" come from pairing them after the Merge):

| Upgrade | Date (UTC) | Headline content | Status |
|---|---|---|---|
| Frontier | 30 Jul 2015 | Mainnet genesis; bare-bones release for developers | ✅ |
| Frontier thawing | 7 Sep 2015 | Lifted tx cap; gas price set; 21,000-gas txs enabled | ✅ |
| Homestead | 14 Mar 2016 | First planned upgrade; EIP-2/7/8; production readiness | ✅ |
| DAO fork | 20 Jul 2016 | Funds restored after the DAO hack → Ethereum Classic split | ✅ |
| Tangerine Whistle | 18 Oct 2016 | First response to the Sep/Oct 2016 DoS attacks (EIP-150) | ✅ |
| Spurious Dragon | 22 Nov 2016 | Second DoS response; EIP-155 replay protection; EIP-170 24,576-byte code-size cap | ✅ |
| Byzantium | 16 Oct 2017 | Block reward 5→3 ETH; difficulty-bomb delay (EIP-649); precompiles for ZK-SNARKs (EIP-196/197) | ✅ |
| Constantinople | 28 Feb 2019 | Reward 3→2 ETH; bomb delay (EIP-1234); EIP-1014 CREATE2 | ✅ |
| Istanbul | 8 Dec 2019 | Gas-cost optimisation; SNARK/STARK-friendly ops; CHAINID opcode (EIP-1344) | ✅ |
| Muir Glacier | 2 Jan 2020 | Bomb delayed ~4,000,000 blocks / ~611 days (EIP-2384) | ✅ |
| Beacon Chain genesis | 1 Dec 2020 | PoS chain boots after 16,384 × 32-ETH deposits | ✅ |
| Berlin | 15 Apr 2021 | Gas-cost reform (EIP-2929), access lists (EIP-2930), typed tx envelope (EIP-2718) | ✅ |
| London | 5 Aug 2021 | **EIP-1559 fee market** — base fee burned; bomb delay (EIP-3554) | ✅ |
| Altair | 27 Oct 2021 | Beacon-chain first upgrade: sync committees for light clients; stiffer inactivity/slashing penalties | ✅ |
| Arrow Glacier | 9 Dec 2021 | Bomb delayed to June 2022 (EIP-4345) | ✅ |
| Gray Glacier | 30 Jun 2022 | Bomb delayed to September 2022 (EIP-5133) | ✅ |
| Bellatrix | 6 Sep 2022 | Beacon chain readied for the Merge; full penalties armed | ✅ |
| **Paris (The Merge)** | **15 Sep 2022** | PoW switched off at block 15,537,393; PoS on; difficulty bomb removed | ✅ |
| Shapella (Shanghai-Capella) | 12 Apr 2023 | Staking withdrawals (EIP-4895) — staked ETH redeemable | ✅ |
| Dencun (Cancun-Deneb) | 13 Mar 2024 | **EIP-4844 proto-danksharding blobs**; EIP-1153 transient storage; EIP-4788 beacon-root in EVM | ✅ |
| Pectra (Prague-Electra) | 7 May 2025 | EIP-7702 EOA code; EIP-7251 max effective balance 2,048 ETH; blob target 6/max 9 (EIP-7691) | ✅ |
| Fusaka (Osaka-Fulu) | 3 Dec 2025 | PeerDAS (EIP-7594); blob-parameter-only forks; gas-limit raise to 60M with 16.7M per-tx cap | ✅ |
| Glamsterdam (Amsterdam-Gloas) | Q4 2026 (planned) | Enshrined proposer-builder separation; block-level access lists | ⚠ in development |
| Hegotá (Bogotá-Heze) | 2027 (planned) | FOCIL inclusion lists; frame transactions | ⚠ in development, scope not final |

The 2016–2022 forks appear here as platform history; their *code-level* consequences (the EIP-170 size cap, EIP-1559's economics, EIP-2718's typed transactions) are mechanics this guide uses in §3, and their security lessons are the smart-contracts guide's domain.

---

## 3. The Architecture: Accounts, Gas, the EVM, Blocks, State

This section is verified against ethereum.org's developer documentation (accounts, gas, transactions, blocks, EVM pages). Unmarked detail is domain-stable protocol knowledge.

### 3.1 Accounts: EOAs and Contract Accounts

Ethereum has exactly two account types, both identified by a 20-byte address:

- **Externally Owned Account (EOA)** — controlled by a private key. An EOA can initiate transactions (transfers, contract calls, deployments). It has a balance and a *nonce* (the count of transactions sent, which prevents replay).
- **Contract account** — holds EVM bytecode plus its own storage; it cannot initiate transactions on its own, it acts only when called by a transaction or by another contract. This is the account type that makes Ethereum a *programmable* ledger.

A "wallet" in Ethereum is most often the key-management wrapper around an EOA. Since **Pectra (May 2025)**, EOAs can temporarily adopt code via **EIP-7702** ("set EOA account code") — the first native step toward smart-contract-powered wallets (batching, gas sponsorship, recovery) without forcing a full account-abstraction redesign ✅ (ethereum.org). Full account abstraction (frame transactions, where the *account itself* decides what makes a transaction valid) is slated for Hegotá ⚠ (§6.3).

### 3.2 Gas: The Meter of Computation

Every operation on Ethereum costs **gas** — the unit that prices EVM computation, storage and data. Gas is Ethereum's answer to the halting problem and its spam defence: a transaction declares a *gas limit* (maximum units it will consume) and a fee; if execution exceeds the limit it reverts (with the fee consumed). A plain value transfer costs **21,000 gas** ✅ (ethereum.org history, Frontier-thawing note). Contract creation was capped by **EIP-170** (Spurious Dragon, 2016) at **24,576 bytes** of deployed code ✅.

Since **London (EIP-1559, August 2021)**, the fee market works as ✅ (ethereum.org):

- A protocol-computed **base fee** per gas unit is burned (removed from supply) — it adjusts between blocks toward a target block fullness, making fees predictable.
- Users add an optional **priority fee** (tip) to incentivise inclusion, which goes to the block proposer (previously the miner; post-Merge the validator).
- Blocks are variable-sized around a 15M-gas target (post-Fusaka the default *limit* was raised to 60M with a 16.7M per-transaction cap ✅ — ethereum.org roadmap).
- The `BASEFEE` is readable on-chain (EIP-3198); fee *refunds* were cut in EIP-3529 (London) to close gas-token schemes.

Gas is paid in **ETH** (denominated in *gwei*, 10⁻⁹ ETH). The separate **blob fee market** introduced by EIP-4844 is covered in §7.

### 3.3 The EVM

The **Ethereum Virtual Machine** is the deterministic execution engine every node runs: a stack-based, 256-bit-word machine executing compiled bytecode (from Solidity, Vyper, Huff or any EVM-targeting language). Determinism is the design spine — every node executing the same transaction against the same state must reach the same result, because state is replicated, not merely recorded. Key properties this guide relies on later:

- **Stack machine with memory and storage** — storage is persistent (part of world state), memory is per-execution; the stack is 1,024 items deep.
- **Accounts, calls and opcodes** — `CALL`, `STATICCALL` (Byzantium, EIP-214), `DELEGATECALL` (Homestead, EIP-7), `REVERT` (Byzantium, EIP-140), transient storage `TLOAD/TSTORE` (Dencun, EIP-1153), and the `SELFDESTRUCT` restriction to same-transaction (Dencun, EIP-6780) shape contract behaviour.
- **Precompiles** — native elliptic-curve operations added over time (Byzantium's EIP-196/197 for pairing over BN254, enabling early ZK-SNARK verifiers; Istanbul's EIP-1108 made them cheaper; Pectra added BLS12-381 precompiles EIP-2537) ✅ — this is *why* ZK rollups chose Ethereum: cheap on-chain proof verification.
- **Cryptographic base** — Keccak-256 hashing and secp256k1 signatures; ECDSA for EOA transactions, BLS12-381 for consensus-layer signatures.

The EVM's gas-scheduled, storage-heavy design is why L1 execution is expensive and why the platform's scaling answer is *move execution off-chain* (§8) rather than make L1 cheaper per se. Smart-contract *development* (Solidity, frameworks, testing, the EIP-170 limit, oracle integration) is the smart-contracts guide's §6 — cross-referenced, not re-derived.

### 3.4 Transactions and Blocks

**Transactions** are signed instructions that change state atomically: transfers, contract calls, and contract deployments. Since Berlin (EIP-2718) Ethereum supports **typed transactions**; the three in production use are the legacy type, the **EIP-1559 type-2** (maxFeePerGas/maxPriorityFeePerGas) and the **EIP-4844 type-3 blob transaction** (§7). Every transaction carries a nonce, a signature, gas parameters and data. The user signs with their private key; the network validates signature, balance and nonce before execution.

**Blocks** in PoS Ethereum are produced on a fixed 12-second **slot** rhythm (one proposer per slot, chosen by RANDAO — §5.3). A block header commits to the *state root* (the Merkle-Patricia root of all account balances and storage), the transactions root, the receipts root, the parent hash, the beacon-root and the blob commitments — so every block is a compact commitment to the entire resulting world state. The 2016 DoS attacks and their fixes (Tangerine Whistle/Spurious Dragon repricing of state-access opcodes, EIP-150/160/161/2929) are the reason state-access opcodes are priced aggressively — a robustness lesson that carried straight into PoS.

### 3.5 State and Finality

Ethereum's **state** is the full mapping of accounts (balance, nonce, storage, code) — the "world computer's" RAM — committed by the state root in each block. Nodes store and serve it: **full nodes** validate everything and keep recent state (older data prunable/regenerable); **archive nodes** keep every historical state (terabytes); **light nodes** verify only block headers ✅ (ethereum.org nodes-and-clients). The **Purge** roadmap era (§6.1) is about shrinking state and history costs (history expiry, state expiry research).

**Finality** in PoS Ethereum means economic finality: a block is *finalized* under Casper-FFG when two-thirds of staked ETH has attested to it across checkpoints, after which reverting it would require mass-slashing of stakers (see §5.3). Normal-case finality is ~2 epochs (~12.8 minutes) — the "settlement" a bank can actually rely on, distinct from the ~12-second probabilistic confirmation of block *inclusion*.

---

## 4. The Client Landscape: Execution and Consensus

### 4.1 The Two-Client Architecture

Since the Merge, an Ethereum node runs **two separate client programs** plus an optional validator: the **execution client** (the former "Eth1" client) executes transactions in the EVM and holds the latest state and database; the **consensus client** (the former "Eth2"/Beacon node) implements the PoS algorithm and tracks the chain head; the **validator client** (usually bundled with the consensus client) handles signing duties ✅ (ethereum.org). They communicate over the Engine API ✅ (ethereum.org history, Paris note). All clients implement common specifications — the Yellow Paper lineage, the execution specs, the consensus specs and the EIPs — which is what makes multiple *independent* implementations of the same protocol possible. Ethereum is unusual among blockchains in having production-grade client diversity at all; most competitors run a single reference client ✅ (ethereum.org client-diversity).

### 4.2 The Execution-Client Roster

Verified roster at ethereum.org (names ✅; the language/org column is domain-stable knowledge):

| Client | Language | Maintainer | Notes |
|---|---|---|---|
| **Geth** (Go Ethereum) | Go | Ethereum Foundation-adjacent core team (geth.ethereum.org) | The original and historically dominant client; "the reference" for most tooling |
| **Nethermind** | C#/.NET | Nethermind | Strong execution-research track; second-largest share in the Oct-2025 snapshot |
| **Besu** | Java | Hyperledger (Linux Foundation); originated at ConsenSys/PegaSys | The enterprise-friendly EL client; also the permissioned-EVM mainline (§9.1) |
| **Erigon** | Go | Erigon team (Ledgerwatch lineage) | Focus on efficiency/archive sync |
| **Reth** | Rust | Paradigm | Newest mainstream EL client; modular, performance-oriented |

### 4.3 The Consensus-Client Roster

| Client | Language | Maintainer | Notes |
|---|---|---|---|
| **Lighthouse** | Rust | Sigma Prime | Largest consensus share in the Oct-2025 snapshot |
| **Prysm** | Go | Prysmatic Labs | Long the most-deployed beacon client |
| **Teku** | Java | ConsenSys | Enterprise-friendly (Besu pairing) |
| **Nimbus** | Nim | Status | Lightweight; light-client focus |
| **Lodestar** | TypeScript | ChainSafe | Full TypeScript beacon client; light-client focus |
| **Grandine** | Rust | Grandine team | Newer entrant, listed by ethereum.org |

Any execution client can be paired with any consensus client — a deliberate property that produces the *client matrix* (e.g. Geth+Lighthouse, Besu+Teku, Nethermind+Lighthouse are common institutional pairings). Running a node yourself is the trust-minimised way to read the chain ("don't trust, verify" ✅ — ethereum.org nodes-and-clients); the practical alternative is an RPC/API provider, which re-introduces a trusted third party. For a bank, node operation and provider diversification are operational-risk decisions (the fireblocks guide covers the custody-infrastructure side).

### 4.4 Client Diversity: The Supermajority Problem ⚠

Client diversity is Ethereum's "insurance policy": a bug or attack affecting one client is contained if that client is a minority. The ethereum.org threat model is precise ✅:

- A consensus-client bug with **>33%** share could **prevent finality** (the chain stops finalizing — users cannot rely on settlement).
- A consensus-client bug with **≥66%** (supermajority) share could make the chain **split and finalize an invalid chain**, stranding a large validator set that faces slashing or forced exit to return.
- A dominant *execution* client (like Geth) similarly risks finalizing bugged or censored execution payloads.

**History and current state (⚠ point-in-time):** for years Geth ran ~85% of execution nodes (ethereum.org's client-diversity page states this majority as the problem case), and the danger got public attention in 2024–25 when a Geth supermajority (reported ~84% in early 2024) coincided with disclosed Geth bugs and renewed "run a minority client" campaigns. The push had visible effect: ethereum.org's snapshot dated **October 2025** shows **Geth 41.0% / Nethermind 38.0% / Besu 16.0% / Erigon 3.0% / Reth 2.0%** on execution, and **Lighthouse 42.7% / Prysm 30.9% / Teku 13.9% / Nimbus 8.7% / Lodestar 2.7% / Grandine 1.0%** on consensus ✅ (ethereum.org client-diversity page, snapshot Oct 2025 — *volatile; check supermajority.info, ethernodes.org, clientdiversity.org and rated.network for live figures*). Consensus-layer attribution is harder (blockprint-style fingerprinting) ⚠. Even after improvement, no single client has yet reached the <33% ideal in the consensus layer — Lighthouse sits just under 43% ⚠ — and the *validator-set* distribution (who actually proposes/attests) can differ from the node count. This section is deliberately dated: **the client mix is one of the most volatile facts in this guide.**

---

## 5. The Consensus History: PoW, the Difficulty Bomb, the Merge, PoS

### 5.1 The PoW Era and the Difficulty Bomb

From Frontier (2015) to the Merge (2022), Ethereum secured itself with **proof-of-work** mining (Ethash, memory-hard to resist ASICs) — the general consensus taxonomy is the blockchain guide's §4, cross-referenced rather than re-derived. Two PoW-era mechanisms matter for understanding *why* the Merge happened when it did:

**Block rewards were cut twice as part of bomb-delay forks**: Byzantium (October 2017) cut 5→3 ETH, Constantinople (February 2019) cut 3→2 ETH ✅ (ethereum.org history). Reward cuts were bundled with difficulty-bomb delays precisely because the bomb was designed to force the PoS transition.

**The difficulty bomb** ("Ice Age") was code that exponentially increased PoW difficulty, making blocks slower and slower until the chain seizes up — a self-destruct timer intended to guarantee Ethereum could not stay on PoW forever. It was delayed six times before the Merge finally removed it ✅ (ethereum.org history; OKX's delay chronology as secondary):

| Fork | Date | EIP | Effect |
|---|---|---|---|
| Byzantium | 16 Oct 2017 | EIP-649 | Delayed ~1 year (reward 5→3) |
| Constantinople | 28 Feb 2019 | EIP-1234 | Delayed ~12 months (reward 3→2) |
| Muir Glacier | 2 Jan 2020 | EIP-2384 | Delayed ~611 days / 4,000,000 blocks |
| London | 5 Aug 2021 | EIP-3554 | Delayed to December 2021 |
| Arrow Glacier | 9 Dec 2021 | EIP-4345 | Delayed to June 2022 |
| Gray Glacier | 30 Jun 2022 | EIP-5133 | Delayed to September 2022 |
| **Paris (Merge)** | **15 Sep 2022** | EIP-3675 | **Bomb removed — PoW ended** |

The pattern is the lesson: each delay was a negotiated commitment that "the next fork will be the one", and the community honoured it in 2022.

### 5.2 The Beacon Chain and the Merge

The path to PoS ran through a separate chain built in parallel:

- **14 October 2020** — the staking deposit contract deployed on Ethereum mainnet ✅ (ethereum.org history).
- **1 December 2020** — **Beacon Chain genesis**, once 16,384 deposits of 32 ETH (524,288 ETH) had been made ✅ (ethereum.org history). The Beacon Chain ran PoS consensus in parallel with PoW mainnet for 21 months — a long public test of Casper-FFG and LMD-GHOST before they secured real value.
- **27 October 2021 (Altair)** — sync committees (light-client support) and stiffer penalties ✅.
- **6 September 2022 (Bellatrix)** — the Beacon Chain was Merge-ready: full inactivity and slashing penalties armed, fork choice updated for the transition ✅.
- **15 September 2022, 06:42:42 UTC — The Merge (Paris).** When the PoW chain hit the terminal total difficulty of 58,750,000,000,000,000,000,000 at block **15,537,393**, the next block became the first PoS block ✅ (ethereum.org history). EIP-3675 upgraded consensus to PoS; EIP-4399 replaced the DIFFICULTY opcode with PREVRANDAO ✅. Energy consumption fell ~99.95% ✅ (ethereum.org roadmap); reported net ETH issuance fell ~88% ⚠ (reported figure). The Merge was designed to be invisible to applications — no transaction history was lost and no chain halt occurred.

### 5.3 The PoS Design: Validators, Queues, Attestations, Finality

Verified against ethereum.org's PoS documentation:

- **Becoming a validator**: deposit **32 ETH** to the deposit contract and run three pieces of software (execution + consensus + validator clients). The deposit joins an **activation queue** that rate-limits how fast the validator set can grow ✅ — the *churn limit* scales with the active set, and since Dencun (EIP-7514) the **activation** rate is capped at 8 validators per epoch ✅ (ethereum.org history, Deneb note). This queue is why "staking onboarding time" can stretch to weeks in a bull market ⚠ (queue length is time-varying).
- **Time**: slots of **12 seconds**, epochs of **32 slots (~6.4 minutes)** ✅. One validator is pseudo-randomly selected (RANDAO) to *propose* each slot's block; committees of validators *attest* (vote) each slot ✅.
- **Attestations**: each active validator attests once per epoch — voting for the current head, the *source* checkpoint and the *target* checkpoint. Attestations are the network's heartbeat: they drive fork choice and finality and earn the validator its rewards ✅.
- **Fork choice (LMD-GHOST)**: the chain head is the branch with the greatest accumulated weight of attestations ✅ — proposer boosting (Altair-era) mitigates short-range reorgs.
- **Finality (Casper-FFG)**: the first block of each epoch is a *checkpoint*; if a pair of checkpoints attracts votes from ≥2/3 of staked ETH, the target becomes *justified* and the previous target becomes *finalized* ✅. Finality normally takes two epochs (~12.8 min). Reverting a finalized block is economically impossible without mass slashing — the property banks should map to "settlement finality", distinct from mere inclusion.
- **Crypto-economic security**: rewards accrue for correct participation; penalties hit missed duties; dishonest behaviour (equivocation — proposing two blocks for one slot, or sending contradictory attestations) triggers **slashing** ✅. If the chain fails to finalize for more than four epochs, the **inactivity leak** bleeds stake from validators voting against the majority, letting the honest majority regain 2/3 and finalize ✅ (ethereum.org PoS docs).

**Validator set scale ⚠ (time-varying):** validator accounts passed 1.2 million by April 2026 per ethereum.org's withdrawals page — meaning >38M ETH staked at 32 ETH each, a figure that moves with market conditions and is flagged rather than certified here.

### 5.4 Slashing, Exits and Withdrawals

- **Slashing mechanics** ✅ (ethereum.org PoS docs): an immediate penalty of 1/4096 of effective balance (up to 0.5 ETH) on day 1; a *correlation penalty* on day 18 that scales with how many others are slashed at the same time (minor for a lone slashed validator, up to 100% of stake in a mass-slashing event); ejection on day 36. The three slashing conditions in the consensus spec are double-proposing (proposer slashing), double-voting and surround-voting (attester slashing) — the spec detail is domain knowledge, the penalty schedule is ✅.
- **Exits and withdrawals** ✅ (ethereum.org withdrawals page): with Shapella (12 April 2023, EIP-4895) staking withdrawals went live — rewards above 32 ETH on *legacy* (Type-1, 0x01) validators are auto-swept every few days (gas-free, consensus-initiated), and full exits process on a queue then get swept (up to 16 withdrawals per block, in validator-index order — the "sweep clock"). Pectra (May 2025) added **compounding (Type-2, 0x02) validators**: effective balance can grow to **2,048 ETH** (EIP-7251), rewards compound, multiple validators can be **consolidated**, and exits/partial withdrawals can be triggered from the execution layer (EIP-7002) ✅.

### 5.5 MEV and Proposer-Builder Separation

**MEV (Maximal Extractable Value)** is the value a block proposer can extract by choosing, ordering or inserting transactions (arbitrage, liquidations, sandwiching) — see the smart-contracts guide §8.3 for the code-level treatment. MEV reshaped PoS Ethereum's economics:

- **Out-of-protocol PBS**: since the Merge, the Flashbots **MEV-Boost** middleware lets validators auction their block space to specialised *builders* via *relays*; the validator (proposer) signs only the winning block and cannot see inside it. MEV-Boost became the default for most institutional/industrial validators — reported relay-built-block shares around 90%+ in 2023–24 **⚠ reported, time-varying** — trading MEV risk for *builder/relay centralization* and new censorship vectors (OFAC-compliant relays were a 2022–24 controversy).
- **Enshrined PBS (ePBS)**: the roadmap answer is to put PBS into the protocol itself so validators can outsource block assembly safely *without trusting external software*. ePBS is scheduled for **Glamsterdam (Q4 2026, in development ⚠)** ✅ (ethereum.org roadmap), with **FOCIL** (fork-choice-enforced inclusion lists — a validator committee can force inclusion of valid transactions, so no single builder can censor) planned for **Hegotá (2027, scope not final ⚠)** ✅ (ethereum.org roadmap).

---

## 6. The Roadmap Eras: Surge, Scourge, Verge, Purge, Splurge

### 6.1 The Framing, Verified

The Merge was never meant to end Ethereum's development — it was the prerequisite for everything after it. In the weeks
following 15 September 2022, **Vitalik Buterin published an updated Ethereum roadmap** that re-framed the post-Merge program
as five further rhyming phases — **the Surge, the Scourge, the Verge, the Purge and the Splurge** ✅ (ethereum.org's roadmap
hub uses exactly these names for the forward program; the trade-press record of the late-2022 release dates it to early
November, a Saturday — **⚠ exact dating is secondary-sourced**, and the surrounding framing essays circulated from October
2022 onward). The framing is *thematic, not chronological*: the phases are parallel workstreams that overlap and feed each
other, several are already partly shipped, and ethereum.org is explicit that the roadmap is "a set of intentions" that "will
change as new information and technology become available" ✅ (ethereum.org roadmap). The same reframing retired the pre-2022
sharding plan outright: with rollup-centric scaling (2020) and danksharding maturing, "shard chains are no longer needed and
have been dropped from the roadmap" ✅ (ethereum.org).

| Era | What it targets | Status as of this writing |
|---|---|---|
| The Merge | The PoW → PoS transition itself | ✅ Done — 15 September 2022 (§5.2) |
| The Surge | Scalability: cheap data availability for rollups, blob space, the danksharding endgame, the ~100,000-TPS aspiration ⚠ | Partly shipped: Dencun (2024) and Fusaka PeerDAS (2025) ✅; full danksharding unscheduled (§6.3, §7) |
| The Scourge | MEV and censorship resistance: proposer-builder separation, inclusion lists, credible neutrality | Scheduled: ePBS at Glamsterdam, FOCIL at Hegotá — both ⚠ in development (§5.5, §6.2) |
| The Verge | Statelessness and cheap verification ("verge" as in *verify*): Verkle/STARK-based state proofs, light clients | Research ⚠ (§6.3) |
| The Purge | Protocol simplification: history expiry, state-expiry research, deletion of legacy cruft (precompiles, SELFDESTRUCT remnants) | Research ⚠ (§6.3) |
| The Splurge | Everything else: single-slot finality, account abstraction, post-quantum cryptography, deep cryptography | Research ⚠; fragments shipped (EIP-7702 at Pectra) or scheduled (frame transactions at Hegotá) |

The names map onto the roadmap themes ethereum.org tracks today — scaling, security, user experience, privacy — but the era
names remain the shorthand the community actually uses. Two discipline notes carry through the rest of this section. First,
the eras are not a delivery schedule: several of the "done" cells above are only *partly* done, which is why §6.2 separates
what has actually shipped from §6.3's speculative backlog. Second, only the shipped column is load-bearing for platform
planning; a roadmap item is an intention, not a commitment (§12 keeps the audit).

### 6.2 The Shipped Post-Merge Upgrades

The shipped record below is inherited from the verified tables in §1.2 and §2.3 (do not re-derive it there) and was
**re-confirmed this pass against ethereum.org's roadmap page** ✅:

| Upgrade | Date | Headline content | Status |
|---|---|---|---|
| Shapella (Shanghai–Capella) | 12 Apr 2023 | Staking withdrawals (**EIP-4895**) + EIP-3651 warm COINBASE | ✅ §2.3; ethereum.org this pass |
| Dencun (Cancun–Deneb) | 13 Mar 2024 | **EIP-4844 blob transactions** (proto-danksharding, §7); EIP-1153 transient storage; EIP-4788 beacon-block root in the EVM | ✅ §2.3; ethereum.org this pass |
| Pectra (Prague–Electra) | 7 May 2025 | EIP-7251 compounding validators to 2,048 ETH; EIP-7002 execution-layer exits; EIP-7514 activation cap; EIP-7691 blob target 6/max 9; EIP-7702 EOA code | ✅ §2.3/§5.4; ethereum.org this pass |
| Fusaka (Osaka–Fulu) | 3 Dec 2025 | **PeerDAS (EIP-7594)** data-availability sampling; blob-parameter-only (BPO) forks; gas limit raised to 60M with a 16.7M per-tx cap | ✅ §2.3; ethereum.org this pass |

Reading the shipped arc: **Shapella** unblocked the staking economy — the ETH locked since December 2020 became redeemable,
and the withdrawal machinery (partial sweeps, exit queues) is the PoS plumbing detailed in §5.4. **Dencun** was the scaling
inflection: it gave rollups a dedicated cheap data lane (blobs) and thereby changed the L2 fee structure permanently — §7 is
devoted to it. **Pectra** was the staking-and-accounts upgrade: compounding validators (2,048 ETH ceilings), EL-triggered
exits and the activation cap all reduce validator-set friction (§5.3–§5.4), while EIP-7702 gives EOAs temporary code — the
first native step toward smart-contract wallets (its EIP-7702 mechanics are in §3.1). **Fusaka** opened the data-availability
endgame: PeerDAS lets nodes sample blob availability peer-to-peer instead of downloading every blob, and BPO forks let the
blob target move between major upgrades without a coordinated hard fork ✅ (ethereum.org).

Two further upgrades are **on the board with dates but not shipped — ⚠ in development, scope live, and re-confirmed against
ethereum.org this pass** ✅: **Glamsterdam (Amsterdam–Gloas), Q4 2026** — enshrined proposer-builder separation (§5.5, the
protocol-native answer to MEV-Boost centralization) plus mandatory block-level access lists for parallel execution and
cheaper state-heavy calls ✅ (ethereum.org roadmap); and **Hegotá (Bogotá–Heze), 2027** — FOCIL inclusion lists (a validator
committee that can force inclusion of valid transactions, killing single-builder censorship) plus *frame transactions* (the
account itself defines transaction validity — the full account-abstraction step beyond EIP-7702). ethereum.org's own language
for Hegotá: **"scope is not final"** ✅. Treat both as scheduled intentions (§6.1), not as committed functionality — the FOCIL
content doubles as the censorship-resistance upgrade the Scourge era promises (§5.5).

### 6.3 The Speculative Items ⚠

Everything below uses real roadmap language but carries **no committed date — flagged ⚠ and not load-bearing for any
architecture or procurement decision.** ethereum.org itself estimates some items are five to ten years out and stresses that
urgency can shift overnight (a sudden quantum leap would reprioritise post-quantum work) ✅.

- **Statelessness (Verge)** — stateless clients that verify new blocks without storing large state, using cryptographic
witnesses ✅ (ethereum.org statelessness). The original Verge path ran through Verkle trees; the research direction of
travel in 2025–26 is toward SNARK/STARK-based state proofs that avoid the Verkle transition entirely — **⚠ direction of
travel is reported research news, not a settled protocol decision**, and no fork has been scheduled.
- **History expiry (Purge)** — EIP-4444-style proposals to cap how long nodes must keep historical data (the EIP number is a
proposal, not a shipped change); ethereum.org warns that if history expires, applications may have to source historical
data elsewhere ✅. **State expiry** — making old state actually *disappear* — is further out and research-only ⚠.
- **Single-slot finality (Splurge)** — finality in one 12-second slot instead of ~2 epochs (~12.8 minutes, §5.3). SSF
requires restructuring attestation aggregation (the orbit-SSF line of work) so a slot's votes settle before the next slot —
real research, no schedule ⚠.
- **Full danksharding (Surge)** — PeerDAS (§6.2) is the *intermediate* sampling step; full danksharding pushes sampling down
to light clients and multiplies blob counts beyond today's 6/9-per-block caps (§7). Unscheduled ⚠.
- **zkEVM on L1 (Verge/Splurge)** — letting validators verify blocks via zero-knowledge proofs instead of re-executing them,
buying gas-limit headroom without raising node hardware requirements ✅ (ethereum.org zkEVM roadmap item — the far-term L1
use of the same technology the L2s of §8.2 run today). No fork scheduled ⚠.
- **Post-quantum security (Splurge)** — replacing secp256k1 and BLS12-381 signatures before quantum computers can break them;
ethereum.org calls it lower priority, likely 5–10 years out ✅/⚠, and notes that Hegotá's frame-transaction design
deliberately leaves "a path to post-quantum signature schemes" ✅ (ethereum.org).
- **Scourge extras** — beyond the scheduled FOCIL/ePBS (§6.2): proposer-diversity research, multi-proposer designs, and
MEV-burn proposals are contested research, not plans ⚠; the shipped reality of MEV today remains out-of-protocol MEV-Boost
(§5.5).

---

## 7. EIP-4844 and Blobs: Proto-Danksharding

Dencun's centerpiece (13 March 2024, §6.2) was **EIP-4844, proto-danksharding**: a deliberately incomplete first slice of
*danksharding* — the full design, named for researcher Dankrad Feist, in which rollups post their bulk data to L1 as
**blobs** rather than as permanent calldata. *Proto-* because it ships the blob data lane without the full sampling machinery
of the endgame design; PeerDAS (Fusaka, 3 December 2025, ✅ §6.2) is the first installment of that machinery, and full
danksharding remains the unscheduled finish line (§6.3). The mechanics, domain-stable with the EIP-verified anchors marked:

- **Blobs are ephemeral data with committed hashes.** Blob-carrying transactions are the EIP-2718 type-3 envelope (§3.4);
each blob is ~128 KB of opaque data (4,096 × 32-byte field elements — the EIP-specified shape). The execution layer never
sees blob contents: the EVM sees only a **versioned hash** commitment (a KZG polynomial commitment, `0x01`-prefixed) per
blob, and can check data against its commitment through the point-evaluation precompile (EIP-4844). Blob creation required
a one-time **KZG trusted setup** ceremony (January 2023, ~140,000 contributions — **⚠ reported; ceremony mechanics are
secondary-sourced here**), secure under a 1-of-N honesty assumption.
- **A separate blob fee market.** EIP-4844 gives blobs their own gas accounting, independent of execution gas: a **blob base
fee** per blob gas, adjusting EIP-1559-style toward a per-block target, with the fee **burned** like execution base fees
(§3.2) so blob space cannot be trivially spammed. The per-block envelope was **target 3 / max 6 at Dencun**, raised to
**target 6 / max 9 at Pectra** (EIP-7691 — inherited ✅ §2.3, re-confirmed ✅ ethereum.org), and Fusaka's
**blob-parameter-only (BPO) forks** allow the target to be raised between major upgrades without a coordinated hard fork ✅
(ethereum.org).
- **Why it cut L2 fees.** Pre-Dencun, rollups published batch data as calldata — permanent, EVM-visible, and priced as
execution gas (~16 gas per byte, the most expensive data lane on the chain). Blobs are cheap by design and do not compete
with execution blockspace: they are stored temporarily by consensus nodes (~18 days / 4,096 epochs of guaranteed
availability — a protocol parameter, with extension under discussion ⚠) and pruned afterwards. The reported result was a
**~10–100× reduction in L2 fees** around Dencun and sustained since **⚠ reported figures** — ethereum.org's own L2
marketing numbers (average L1 transaction ~US$0.026 vs L2 ~US$0.0014) tell the same story at the order-of-magnitude level,
and are illustrative, not audited ⚠.
- **The trade-off, and PeerDAS.** Cheap-but-ephemeral DA means an L2 must actually pull its blobs inside the availability
window (or accept archival risk), and the *guarantee* that blobs can be verified as available is exactly what PeerDAS
(EIP-7594) strengthens: peer-to-peer **data-availability sampling** lets nodes verify blob availability without downloading
every blob — "more efficient data availability for rollups... making running a node more accessible while maintaining
decentralization" ✅ (ethereum.org roadmap, Fusaka slide). Full danksharding would push sampling down to light clients and
raise blob counts well beyond today's caps — unscheduled ⚠ (§6.3).
- **Positioning.** Blobs are a *data* lane, not an execution lane: plain L1 transaction fees were untouched by EIP-4844, and
the blob market's demand side is precisely the L2 ecosystem of §8 — blobspace is the commodity the rollups buy. EIP-4844's
sibling Dencun EIPs matter to L2 infrastructure too: EIP-1153 transient storage cheapened complex contract patterns, and
EIP-4788's beacon-block root in the EVM lets contracts (and bridges, §8.3) verify consensus-layer state without trusting an
oracle ✅ (ethereum.org).

---

## 8. The L2 Ecosystem: Rollups and Bridges

Ethereum's scaling answer is *off-chain execution, L1 settlement*: a **Layer-2 (L2)** network executes transactions in bulk
and posts compressed results back to Ethereum, which acts as settlement and data-availability layer. The strict definition
matters for security: a **rollup** is an L2 that posts its data **on Ethereum L1** — "this greatly enhances user security and
Ethereum's permissionless vision"; projects that store data elsewhere are not direct Ethereum extensions ✅ (ethereum.org).
There is no official L2 — "Ethereum is permissionless — technically anyone can create a layer 2" ✅ (ethereum.org) — which is
why the verified roster of §8.2 is only the canonical subset of a long tail. The ledger-basics survey (blockchain guide's
L2/EVM sections) and the code layer (smart-contracts guide) carry the underlying mechanics; this section keeps the platform
view: how rollups work (§8.1), the canonical projects verified (§8.2), bridging condensed (§8.3), and the TVL caveat (§8.4).

### 8.1 How Rollups Work

Both rollup families do the same architectural move — batch many L2 transactions, compute the new state off-chain, and post a
compact commitment plus data to L1 — and differ in *how the L1 is convinced the state transition was correct*:

- **Optimistic rollups** post batches and *assume* they are valid unless challenged. Anyone running a **fraud proof** against
a suspect batch within a challenge window — canonically **~7 days** (per-rollup configuration ⚠) — triggers re-execution on
L1; a successful challenge reverts the bad batch and slashes the sequencer's bond. Withdrawals are therefore delayed by the
window (a user must wait out the challenge period to exit to L1). The security assumption is explicit: **one honest
challenger**. Modern implementations run interactive, bisection-based fault-proof systems (the OP Stack's Cannon,
Arbitrum's boLD); these were progressively opened to *permissionless* challenge in 2024–25 — a moving operational target ⚠
best treated as domain-stable "fraud-proof rollup" mechanics rather than pinned dates. Canonical examples: Arbitrum One, OP
Mainnet, Base (§8.2).
- **ZK (validity) rollups** post, with every batch, a **validity proof** (SNARK or STARK) that the state transition is
correct; the L1 verifies the proof and accepts the batch — no challenge window, no waiting out a fraud-proof period,
withdrawals in hours rather than days (prover- and proof-dependent ⚠). The cost is *proving*: generating proofs is
compute-heavy and latency-sensitive, which is why proving markets and shared provers are an industry of their own.
Compatibility with Ethereum's EVM exists on a spectrum: **EVM-equivalent zkEVMs** (Polygon zkEVM, Scroll, Linea) aim to run
Solidity bytecode near-unmodified, while **native-ZK stacks** (Starknet's Cairo VM — a purpose-built language, *not* EVM by
default) and zkSync Era (EVM with native account abstraction) trade drop-in compatibility for prover efficiency. This
taxonomy is domain-stable; Ethereum's Byzantium-era precompiles (§3.3) are why cheap on-chain proof verification exists at
all.
- **Common anatomy.** Both families route user transactions through a **sequencer** that orders batches — today almost always
a single operator, a real (if widely accepted) trust and liveness assumption ⚠ whose decentralisation is an active
research/roadmap topic (based-rollups, enshrined sequencing — §6.3 territory). Batch data rides the blob lane of §7. The
security-inheritance sentence that matters: **funds on a rollup enjoy L1-grade safety only to the extent the data is on L1
and the withdrawal path is L1-enforced** — which is precisely what separates a rollup from a validium or an independent
chain ✅ (ethereum.org).

### 8.2 The Canonical Projects, Verified

Families below are verified ✅ against ethereum.org's L2 listings (Base: "an Optimistic Rollup built with the OP Stack";
Starknet: "a general purpose ZK Rollup based on STARKs and the Cairo VM") and the March 2023 launch record ✅ (Polygon's own
mainnet-blog; Defiant/Blockworks coverage of zkSync Era). **Mainnet dates not re-fetched from the projects' own docs this
pass are ⚠ project-reported** (the repo convention: reported facts are flagged, not silently asserted):

| Project | Operator / team | Family | Mainnet | Status |
|---|---|---|---|---|
| Arbitrum One | Offchain Labs | Optimistic (fraud proofs) | 2021 ⚠ (project-reported; widely documented as Aug 2021) | Family domain-stable; date ⚠ not re-fetched this pass |
| OP Mainnet (Optimism) | OP Labs | Optimistic (OP Stack) | 2021 ⚠ (project-reported; Dec 2021) | Family domain-stable; date ⚠ |
| Base | Coinbase | Optimistic (OP Stack) | 2023 ⚠ (project-reported; Aug 2023) | Family ✅ ethereum.org; date ⚠ |
| zkSync Era | Matter Labs | ZK (zkEVM, native account abstraction) | Mar 2023 ✅ (trade press: Defiant, Blockworks) | ✅ launch record; family domain-stable |
| Starknet | StarkWare | ZK validity (STARKs, Cairo VM) | 2021 alpha ⚠ (project-reported; mainnet matured in stages) | Family ✅ ethereum.org; date ⚠ |
| Polygon zkEVM | Polygon Labs | ZK (EVM-equivalent zkEVM) | 27 Mar 2023 ✅ (Polygon Labs mainnet-beta blog) | ✅ primary source |
| Scroll | Scroll | ZK (EVM-equivalent zkEVM) | 2023 ⚠ (project-reported; Oct 2023) | Family domain-stable; date ⚠ |

Reading the canonical roster: the **optimistic trio** is Arbitrum One — the longest-lived optimistic rollup (Offchain Labs;
migrated to the faster Nitro engine in 2022 ⚠) — OP Mainnet, whose **OP Stack** became the dominant rollup framework
(Bedrock-era modularity, the "Superchain" thesis of shared, interoperable OP-Stack chains), and **Base**, Coinbase's OP-Stack
deployment — the exchange-operated bellwether whose institutional significance is picked up in §9's enterprise context. The
**ZK quartet** split the zkEVM race of 2023: Matter Labs' zkSync Era and Polygon's zkEVM reached public mainnet within days
of each other in March 2023 (✅ verified above), Starknet brings the longest STARK-proving pedigree with its non-EVM Cairo
stack, and Scroll rounds out the EVM-equivalent school. The roster extends well beyond these seven — ethereum.org's networks
list alone adds Linea (ConsenSys' zkEVM, 2023 ⚠), Ink, Unichain and more, and the "is it a rollup?" test from §8.1 (data on
L1) is the filter to apply to any newcomer ✅ (ethereum.org).

### 8.3 Bridging, Condensed

L2s are separate state machines, so moving assets or messages between them (or back to L1) requires a **bridge** — and the
bridge, not the rollup, is where most L2 risk has historically lived. The condensed taxonomy (the web3 guide carries the full
bridge treatment, including the wallet and UX layer — [web3_technologies_guide.md](web3_technologies_guide.md) — and the
smart-contracts guide's incident history owns the code-level write-ups):

- **Canonical bridges (lock-mint, L1-enforced).** The rollup's own official bridge holds assets in an L1 escrow contract and
mints the L2 representation. Withdrawals go back through L1 and inherit the rollup's security model — including the
optimistic **7-day exit delay** (§8.1) or the ZK rollup's proof-based fast exit. No extra trust layer beyond the rollup's
own contracts.
- **Burn-mint bridges.** For assets native to one chain (a stablecoin issuer bridging its token, say), the source
representation is *burned* and a fresh one minted on the destination — the supply-preserving pattern, but note the minting
authority is the issuer or its agent, not L1 — ⚠ issuer-controlled variants carry counterparty risk by design.
- **Third-party / generalized message bridges.** Independent networks (Wormhole and its peers) relay arbitrary messages with
their own validator/relayer sets — adding a trust assumption *outside* both chains. They are where the ecosystem's worst
losses have occurred.

The risk history is the lesson: the **2022 bridge-hack cluster** — Wormhole (~US$320M, Feb 2022), Ronin (~US$625M, Mar 2022),
Horizon (~US$100M, Jun 2022), Nomad (~US$180–190M, Aug 2022) — **⚠ reported totals; the web3 guide and smart-contracts guide
carry the verified incident records**. The pattern in every case was compromise of the bridge's own signature/validator layer
or a contract flaw *in the bridge*, not a break of Ethereum itself. Canonical rollup bridges have no external validator layer
to compromise, but their escrow contracts remain concentrated-value targets; **EIP-4788's beacon-block-root opcode** (Dencun,
§6.2) was aimed squarely at this problem, letting bridges verify consensus-layer state on-chain and move toward
trust-minimized designs ✅ (ethereum.org). Platform-level guidance for a bank: use the canonical bridge for the chain you are
on, budget the exit delay in operations (days on optimistic rollups, hours on ZK ⚠), and treat third-party bridges as a risk
decision to be justified, not a default.

### 8.4 The TVL Caveat ⚠

Every TVL number attached to this ecosystem — per-project totals, the "which L2 is biggest" rankings, the aggregate across
rollups — is **⚠ third-party, unaudited, methodology-dependent and fast-moving**. L2BEAT and DeFiLlama are the standard
trackers, but their definitions differ (locked-on-L1 vs bridged vs natively-issued), the same assets are double-counted
across chains and layers, and "value secured" is not the same as "value at risk in L2 contracts". Treat any figure —
including the ones implied in §8.2's notes — as a point-in-time snapshot, and for institutional purposes read *quality*
indicators instead of raw size: is the project a true rollup (data on L1, §8.1)? What **stage** has it reached on the
L2BEAT-style maturity ladder (training wheels — upgradeable contracts, operator-controlled sequencing — removed
progressively; Stage 0 → 2 taxonomy ⚠)? How long is the exit, and who controls the sequencer and the upgrade keys? Those
answers, not TVL rankings, are what survive contact with a risk committee; the claims audit in §12 records what this guide
could and could not certify.


---

## 9. The Enterprise and Tokenization Context

This section is the platform-side half of the enterprise story: what happened to Ethereum's *code* when banks adopted it (the permissioned-EVM lineage), and what tokenization means at the protocol layer. The institutional half — products, pilots, custody vendors and company records — is owned by the banking guides; per the sibling map in §1.3 it is cross-referenced here, not re-derived. Dated or single-sourced items carry ⚠.

### 9.1 Permissioned Ethereum: Quorum to Besu

When enterprises looked at Ethereum in 2015–16 they wanted the EVM and not much else: a public ledger, permissionless validators, proof-of-work costs, no access control and no confidentiality looked like liabilities, not features. The answer was to fork the *client software* and change the *network rules* — keep Ethereum's bytecode, account model and tooling; replace consensus, membership and privacy. The verified lineage, inherited from the §1.2 key-facts table: **JPMorgan's Quorum (2016)** was acquired by **ConsenSys (announced August 2020)** ✅ (ConsenSys press release; Reuters — inherited §1.2), after which the Quorum brand was retired and its code consolidated into **Hyperledger Besu**, the surviving permissioned-EVM mainline ✅ (§1.2; execution-client roster §4.2).

- **Quorum's design (2016, JPMorgan; a go-ethereum fork)** added three enterprise pieces: *node and validator permissioning* (only whitelisted parties transact and propose); *private transactions* (payloads held off the shared ledger by a separate transaction manager — Constellation, later Tessera — with only hashes committed on-chain); and *a BFT-family or Raft consensus replacing PoW* for deterministic, near-instant finality. All three remain the vocabulary of enterprise Ethereum.
- **Besu after the consolidation** (domain-stable; roster §4.2) is the Java execution client that runs in two modes: as a public-mainnet execution client (§4.2), and as the base of *permissioned consortium networks*, where the usual consensus is **QBFT** (a Quorum-Istanbul-BFT variant) — immediate finality, no forks, and a validator set defined by the consortium's own rules rather than by 32-ETH deposits (§5.3).
- **Ethereum-compatible, not Ethereum.** A permissioned Besu chain executes identical bytecode and speaks the same JSON-RPC, but its security model is membership plus honest-majority among *known* validators — none of L1's staking economics, finality, censorship-resistance or fork-choice properties transfer. The taxonomy (permissioned vs permissionless, BFT vs Nakamoto consensus) is the blockchain guide's §4, cross-referenced rather than re-derived.

The platform-side rationale for running one, and its mirror-image cost:

- **Regulatory and data posture** — validators are regulated entities; transaction data can be confined to a jurisdiction and to the consortium; private transactions keep client positions and NAV off any public ledger (data-residency arguments hold).
- **Fee and throughput control** — no public gas market: consortium networks run with nominal or administratively set fees, so unit cost is flat and predictable, in contrast to the EIP-1559 market of §3.2.
- **Operational control** — the bank runs or co-runs validators, defines the set, and governs upgrades on its own schedule instead of waiting on L1 social consensus (§2.3).
- **The costs** — no permissionless liquidity (only members can hold or trade the token); a smaller, known validator set is a concentration risk rather than a million-validator security surface; and ecosystem gravity (§8, the web3 guide) points at L1 and the major L2s, not at a private chain. The consolation is portability: because the EVM is shared, contracts written for a permissioned pilot can move to an L2 or L1 later — the exit path that §11.4 exploits.

The two rails side by side, which is the comparison §11 scores in full:

| Dimension | Permissioned EVM (Besu/QBFT) | Public Ethereum L1 |
|---|---|---|
| Consensus and finality | QBFT among known validators; immediate, forkless finality | Casper-FFG plus LMD-GHOST; ~12.8-min normal-case finality (§5.3) |
| Membership | Whitelisted nodes and validators only | Permissionless — anyone can run a node or stake 32 ETH (§5.3) |
| Transaction visibility | Public or private by design (private-transaction manager) | Public by default (§3.1); pseudonymity only |
| Fee market | Nominal or administratively set; flat unit cost | EIP-1559 base fee plus priority tip, market-priced (§3.2) |
| Security model | Honest majority of a known, contractually bound set | Crypto-economic: staked ETH, slashing, inactivity leak (§5.4) |
| Upgrades | Consortium-governed, on the consortium's schedule | Social consensus across clients and stakers (§2.3) |
| Ecosystem access | EVM tooling only; no permissionless liquidity or DeFi | Full L1 plus L2 ecosystem: DeFi, oracles, wallets, bridges (§8, web3 guide) |

The table is why the enterprise story is not a simple "permissioned won": every permissioned advantage (control, privacy, flat fees, fast finality) is the mirror image of a permissionless property surrendered (open membership, public audit, crypto-economic security, ecosystem liquidity). Besu's dual-mode existence — one client, two network postures — is the industry's acknowledgement that both are needed, often by the same institution at different stages.

### 9.2 Tokenized Real-World Assets and Project Guardian, Condensed

**Tokenization at the protocol level** is issuing a claim — to an asset, a fund unit, a debt instrument or a deposit — as a token on a programmable ledger, so that transfer, settlement, corporate-action logic and compliance checks execute as contract code instead of back-office reconciliation. The code layer (which standards: ERC-20 fungibles, ERC-721/1155 for collectibles, ERC-4626 vaults for yield-bearing fund units, allow-listing patterns) is the smart-contracts guide's verified domain — inherited by reference, not re-derived ✅ ([smart_contracts_guide.md](smart_contracts_guide.md) §5). What belongs in a *platform* monograph is the settlement question: a token is only as good as the finality of the rail it lives on (§3.5) and the exit that rail offers back into fiat (§8.3).

**MAS Project Guardian** is the institutional anchor for regulated tokenization pilots: announced by MAS in **November 2022** ✅ (inherited — [tokenized_assets_guide.md](../banking/tokenized_assets_guide.md), the repo's institutional twin, carries the announcement record and pilot inventory), it is an industry-pilot programme testing tokenization in wholesale funding markets, later extended to tracks spanning fixed income, FX and fund/asset management (⚠ track inventory is time-varying and sibling-carried). MAS observes rather than licenses each experiment — the regulatory signal is that pilots should happen (§10.1). Platform-side reading: Guardian pilots are deliberately rail-agnostic, but most have run on permissioned or semi-permissioned EVM rails with public-EVM compatibility kept in view — the same "start controlled, keep the exit path" pattern that §11.4 recommends.

**JPM Coin / Onyx** — JPMorgan's deposit-token and permissioned-network programme on Quorum-lineage code — is the longest-running institutional Ethereum-lineage record and is owned by [jpmorgan_chase_software_systems_guide.md](../banking/jpmorgan_chase_software_systems_guide.md) ✅ (inherited); the full RWA treatment (asset classes, custody vendors, tokenized-deposit designs) belongs to [tokenized_assets_guide.md](../banking/tokenized_assets_guide.md). Neither is re-derived here.

**Platform-side implications for a tokenization programme** (product-level detail stays in the institutional twin):

- **Standards are the portability layer.** Contracts written to ERC-20/ERC-4626-grade interfaces (smart-contracts guide §5) run on Besu, an L2 or L1 alike — which is what makes "start permissioned, migrate later" a credible strategy rather than a hope (§11.4).
- **Finality defines the product promise.** A tokenized deposit or fund unit is "settled" only at the rail's finality event (§3.5): ~12.8 minutes on L1, seconds on QBFT, minutes-plus-challenge-window on optimistic L2s — and the product disclosure must name that event.
- **Composability is the public-rail advantage and the permissioned-rail absence.** On L1/L2 a token can meet liquidity, collateral and payments rails it did not plan for (§8); inside a consortium it meets only what the consortium built.
- **Custody and settlement rails, tokenized-deposit designs and the vendor landscape** are owned by [tokenized_assets_guide.md](../banking/tokenized_assets_guide.md) and [fireblocks_guide.md](../banking/fireblocks_guide.md) — cross-ref, do not duplicate.

---

## 10. The Regulation, Condensed with Cross-References

Regulation is sibling-owned territory: the MAS guide carries the home-regulator depth, and the tokenized-assets guide carries product-level rules. This section condenses only the three headlines a platform architect actually needs — Singapore (the repo's home-regulator setting), the United States (the spot-ETH-ETF milestone) and Europe (MiCA) — with the ✅ marks inherited from the §1.2 key-facts table where the fact was already verified there.

### 10.1 MAS: The Home-Regulator Posture

MAS regulates *activity, not technology*: a ledger or token is judged by the financial activity it implements — under the Securities and Futures Act when the token is a capital-markets product, under the Payment Services Act for digital-payment-token services, and under newer digital-money instruments as they mature. The depth is [mas_regulations_guidelines_guide.md](../banking/mas_regulations_guidelines_guide.md), which also hosts the repo's Cymbal Bank persona conventions (§11.1); this guide keeps only the two positions that shape a platform choice:

- **Stablecoins.** MAS finalised a single-currency stablecoin (SCS) framework through 2023–24 — the PSN02 notice pathway — confining the full regime to SCS pegged to the SGD or G10 currencies, with reserve, capital, redemption and issuance-cap requirements; other stablecoins sit under lighter or indirect treatment. ⚠ Dated and sibling-carried; see the MAS guide for the exact chronology.
- **Tokenization.** Project Guardian (§9.2) is the signal: MAS wants supervised pilots to happen, technology-neutrally, with attention to custody, settlement finality and AML — not to which chain was chosen. For Cymbal's purposes the consequence is that the *bank* is accountable for conduct and disclosure on whatever rail it picks; the rail is an architecture decision, and MAS comfort with it is a gating criterion rather than a given (⚠ posture as of 2025; see §12.3).
- **DPT services.** Providing digital-payment-token services (custody, exchange venues) is licensed activity under the Payment Services Act, and MAS tightened retail-access rules through 2022–23 — no incentives, no credit-card purchase channels, restricted advertising (⚠ sibling-carried chronology). For this guide the point is negative: the Cymbal pilot keeps customers out of DPT custody entirely (§11.1), so the PSA's DPT licensing does not gate the product — the SFA and stablecoin frameworks do.
- **The stablecoin linkage.** The 2023–24 SCS framework matters to Ethereum mainly because the ecosystem's dominant payment stablecoins are not SGD-pegged: a bank issuing its own SGD tokenized deposit sits in a different regulatory box from a G10 stablecoin issuer, and the distinction (deposit liability vs stored-value instrument) is exactly what the MAS guide and the tokenized-assets guide dissect. This guide only flags it (§12.3).

### 10.2 United States: The Spot ETH ETF

After the spot-BTC-ETF approvals of January 2024 opened the US wrapper market, ether followed: the SEC approved **19b-4 exchange-rule filings for spot ETH ETFs on 23 May 2024**, and once S-1 registration statements were effective the funds **began trading on 23 July 2024** ✅ (inherited §1.2 — Reuters; SEC 19b-4 orders). The BTC precedent itself had needed a federal appeals court to reverse the SEC's denial of Grayscale's conversion bid in August 2023 — the litigation context that framed how the ETH filings were read (unmarked domain-stable context). Architect-level reading:

- The approvals were exchange-rule orders — a finding that the proposed rule changes are consistent with the Exchange Act — **not** a Commission determination on ether's legal status. Applicants had removed staking from their structures, so the ETFs hold ETH and earn no staking yield (unmarked structure facts; the security/commodity debate is noted in §12.2).
- The launch vehicles were cash-creation spot products from BlackRock, Fidelity, Grayscale (converting its ETHE trust), Bitwise, VanEck, Franklin and Invesco/Galaxy; ⚠ the issuer roster and AUM move constantly and are point-in-time.
- Platform consequence: a regulated US on-ramp for ETH removes the "can we even touch this asset" objection at the wrapper level — the network underneath is unchanged, but the *asset* now has SEC-approved plumbing, which is what institutional policy teams needed (see §9.2 for the token side of the same story).

### 10.3 Europe: MiCA

**Reg (EU) 2023/1114**, the Markets in Crypto-Assets Regulation, is the EU's horizontal regime ✅ (inherited §1.2 — EUR-Lex; ESMA): the stablecoin titles (asset-referenced tokens and e-money tokens) applied from **30 June 2024**, and the full regime — including CASP authorisation for crypto-asset services and market-abuse rules — applied from **30 December 2024**. Architect-level reading:

- MiCA regulates *services and stablecoins*, not the protocol: running an Ethereum node or deploying a contract is not in itself a regulated activity; issuing a token or providing custody, trading or exchange services is.
- The stablecoin titles reach the L2 fee rails indirectly: the dominant USD stablecoins must qualify as e-money tokens to be offered EU-wide (⚠ compliance status is time-varying; see [reap_global_guide.md](../banking/reap_global_guide.md) for the payments angle).
- Tokens that qualify as MiFID financial instruments fall outside MiCA's core scope and into the traditional securities regime — the classification analysis is a product-team input to any tokenization decision (unmarked domain-stable).

**Rest-of-world notes.** The UK (FCA's phased 2024–26 regime), Hong Kong (VASP licensing) and Japan (PSA/FTEA) run parallel tracks with different boundary lines between "token" and "security"; market-level detail is carried by the exchange and custody guides ([bitunix_guide.md](../banking/bitunix_guide.md), [fireblocks_guide.md](../banking/fireblocks_guide.md)), and is not re-derived here.

---

## 11. The Cymbal Bank Worked Example: A Tokenization Pilot Platform Assessment

### 11.1 The Design-Fiction Frame

**Cymbal Bank** is the fictional Singapore-headquartered bank used for worked examples across this repo; the persona conventions live in [mas_regulations_guidelines_guide.md](../banking/mas_regulations_guidelines_guide.md). No real bank is depicted — every figure below is illustrative design fiction.

**The business ask.** Cymbal's Wealth & Deposits unit wants to pilot *tokenized structured deposits and open-ended fund units* for a closed cohort — 20,000 SGD wealth customers, four products, roughly S$250M pilot NAV — with three headline promises: 24/7 subscription and redemption requests, atomic settlement against the bank's own books, and a regulator-ready audit trail. The pilot runs 12 months, MAS-observed, in the Project-Guardian-style posture of §10.1. Constraints handed to the architect:

- Settlement must map to *finality*, not inclusion — operations books on finality, so the rail's finality time defines the operating model (§3.5, §5.3).
- KYC/AML stays at the bank; wallets are custodial (see [fireblocks_guide.md](../banking/fireblocks_guide.md)); customers never run nodes or hold keys.
- The middleware estate (core banking, payments, integration platform) is not being rewritten — [enterprise_middleware_integration_platform_guide.md](enterprise_middleware_integration_platform_guide.md) owns that layer; the pilot must land via adapters, not a new backbone.
- MAS-observed means the *bank* is accountable for conduct and disclosure on whatever rail it picks (§10.1) — regulatory comfort with the rail is a gating criterion, scored as such below.

**What "done" looks like — the pilot's success criteria** (design fiction, agreed with the business owners before scoring):

- 24/7 subscription and redemption requests, with settlement booked on *finality*, not on batch cycles — the operating-model change the pilot exists to prove.
- On-ledger NAV integrity: the fund-unit and structured-deposit NAV logic is contract-enforced where it can be, and reconciled to the bank's books daily — the audit trail a regulator can read.
- Zero customer key custody: every customer position is held in Cymbal's custodial wallets; customers interact through the existing banking app, not a wallet UI.
- A clean exit run: full cohort redemption and the contractual kill-switch executed in the final month without manual intervention — the property that makes "pilot" acceptable to risk and treasury.
- Cost per redemption event at or below the legacy T+2 unit-trust process — the economic thesis the pilot must not embarrass.

**The pilot timeline (illustrative).** Months 1–2: rail build and consortium permissioning (rail C). Months 3–4: contract development and the two independent audits (§11.4). Month 5: cohort onboarding in waves. Months 6–10: live operation of the four products. Month 11: exit drill and full redemption run. Month 12: the decision gate — continue (scale cohort), migrate (execute the rail-B option), or kill — scored against the §11.3 criteria re-weighted by the business owners.

> **Design fiction.** Cymbal Bank, the cohort, the NAV, the weights and the scores are illustrative. This assessment describes no real bank, product or pilot.

### 11.2 The Decision: Public Ethereum versus Permissioned EVM

The shortlist is three rails, each mapping to a section of this guide:

- **A. Public Ethereum L1** (§3, §5) — the contracts live on mainnet; the bank pays gas in ETH; normal-case finality is ~2 epochs (~12.8 minutes) ✅ (§5.3); all positions are visible on a public ledger.
- **B. An Ethereum L2 app-chain** (§8) — an OP-Stack or Arbitrum Orbit chain on which Cymbal (or a joint venture) operates the sequencer, posting blobs to L1 for data availability (Dencun, §7). Settlement is L1 finality plus the L2's own design: optimistic-rollup rails carry a challenge and exit window, ZK rails carry proof-generation latency — ⚠ per-network configuration, not a single number (§8).
- **C. Permissioned EVM — Besu with QBFT** (§9.1) — Cymbal and two co-validator institutions run a consortium chain with immediate finality, private transactions and administratively set fees.

The nine decision criteria, fixed before scoring and weighted to reflect the business ask — a *pilot*, so time-to-market, cost and reversibility deliberately outweigh raw ecosystem: (1) settlement finality and operating hours; (2) fee volatility; (3) privacy and data residency; (4) validator/operator control; (5) regulatory posture; (6) ecosystem and talent; (7) total cost; (8) time-to-market; (9) middleware interoperability. Scoring is 1–5, where 5 = best for this pilot. The matrix is §11.3; the arithmetic is shown so the weights can be argued with.

Why these three rails and no more: a non-EVM permissioned DLT (the Corda-class alternative) was excluded because Cymbal's middleware, vendor and talent surface is EVM-shaped ([enterprise_middleware_integration_platform_guide.md](enterprise_middleware_integration_platform_guide.md), [smart_contracts_guide.md](smart_contracts_guide.md)) — adopting a second VM family would double every cost row below; a *private* fork of L1 with no L2 exit (the pre-Besu Quorum pattern, §9.1) was excluded because it forfeits the portability argument that makes rail C reversible. The three survivors form a clean ladder — C is B without the public-settlement layer; B is A with an operator in the middle — which is why the recommendation (§11.4) treats them as phases of one strategy rather than rivals.

### 11.3 The Assessment Matrix

| Criterion (weight) | A. Public L1 | B. L2 app-chain | C. Permissioned EVM (Besu/QBFT) |
|---|---|---|---|
| Settlement finality and operating hours (15%) | 2 — ~12.8 min normal-case finality; 24/7, but finality, not inclusion, drives ops | 4 — L1 finality inherited; fast L2 confirmations; 24/7; optimistic rails add a ~7-day challenge window for exits ⚠ | 5 — immediate QBFT finality in seconds; 24/7 by design; no probabilistic settlement |
| Fee volatility (10%) | 1 — public EIP-1559 market (§3.2); spikes are an ops-budgeting problem | 4 — blob data availability is cheap (§7); sequencer fees still variable but low at pilot volume | 5 — nominal, consortium-set fees; flat unit cost |
| Privacy and data residency (15%) | 1 — ledger fully public; positions, NAV and flows visible (pseudonymous only) | 2 — public by default; privacy needs a sub-design (ZK or a separate chain) | 5 — private transactions; data confined to consortium and jurisdiction |
| Validator/operator control (10%) | 1 — permissionless; Cymbal has no operator role (§5.3) | 4 — Cymbal runs or co-runs the sequencer; L1 safety remains external | 5 — Cymbal plus co-validators define the set |
| Regulatory posture (20%) | 2 — MAS comfort with retail-facing deposit/fund tokenization on a public chain is not established (⚠ §10.1) | 3 — sequencer accountability helps; public-ledger questions remain | 4 — a consortium of regulated banks is a familiar posture for MAS-observed pilots |
| Ecosystem and talent (5%) | 5 — maximal tooling, standards, auditors, wallet and oracle support | 4 — fast-growing; L2 tooling is now standard practice | 2 — small talent pool; bespoke operations |
| Total cost, 12-month pilot (10%) | 2 — gas, custody, oracles, monitoring; L1 operations are real money | 3 — infrastructure plus L1 DA fees plus sequencer operations | 4 — infrastructure only; consortium governance overhead |
| Time-to-market (10%) | 3 — contracts deploy fast; compliance on a public rail does not | 3 — chain launch plus bridge plus compliance | 4 — controlled environment; permissioning is the only gate |
| Middleware interoperability (5%) | 2 — public RPC adapters only; no private-transaction fit | 3 — standard RPC; sequencer APIs to build | 5 — permissioned RPC and private-transaction manager match bank-middleware patterns |
| **Weighted total** | **1.90** | **3.25** | **4.45** |

The totals recompute from the table: A = 0.30+0.10+0.15+0.10+0.40+0.25+0.20+0.30+0.10 = 1.90; B = 0.60+0.40+0.30+0.40+0.60+0.20+0.30+0.30+0.15 = 3.25; C = 0.75+0.50+0.75+0.50+0.80+0.10+0.40+0.40+0.25 = 4.45. The scores are design fiction; the *method* — weight the criteria before scoring, keep the arithmetic visible, treat the outcome as a decision input rather than a verdict — is the reusable part.

### 11.4 The Recommendation and the Risk Register

**Recommendation: run Phase 1 on rail C — permissioned EVM (Besu/QBFT) — with rail B (an L2 app-chain) as the stated migration target and rail A (public L1) as the long-horizon option.** Three reasons carry it:

- **The binding constraints are regulatory posture, privacy and middleware interop — and C wins all three.** MAS-observed means the consortium-of-regulated-banks posture is the one the regulator has seen work (§10.1); private transactions keep client positions and NAV off any public ledger; and permissioned RPC plus a private-transaction manager is the integration pattern Cymbal's middleware already speaks ([enterprise_middleware_integration_platform_guide.md](enterprise_middleware_integration_platform_guide.md)).
- **EVM compatibility makes the choice reversible.** The same ERC-20/ERC-4626-style contracts, custody code and NAV logic run unchanged on an L2 or L1; the exit path is code portability, not a bridge (§9.1). Rail C never strands the bank's work.
- **B's advantages are real but unearned at pilot scale.** Liquidity, ecosystem and public settlement pay off only if the product goes open — a Phase-2 decision this cohort (20,000 customers, S$250M) does not force. Choosing C now buys a controlled 12-month experiment at roughly half of B's infrastructure complexity.

What would flip the decision to B or A today: a hard MAS signal that retail-facing tokenized deposits must settle on a public chain (none observed as of writing — ⚠ posture, §10.1, §12.3), or demonstrated secondary-market demand from the cohort itself (§9.2). Phase 2 (out of scope here) should compare B against A directly on the §8 criteria — sequencer centralization, challenge-window liquidity, data-availability cost — using this pilot's live data as input.

**Decision-gate metrics at month 6** (the numbers the business owners should re-score against, not the architect's scores):

- Experienced time-to-finality per transaction class versus the §11.3 assumptions (QBFT seconds versus the L1/L2 reference figures of §5.3 and §8).
- Unit economics: cost per subscription, per redemption, per NAV reconciliation — rail C's flat-fee promise tested against the middleware adapter costs it hides.
- Regulatory temperature: MAS feedback across the observation meetings — whether the §10.1 posture held in practice for a deposit instrument (⚠; see §12.3).
- Customer-side friction: the 24/7 promise versus cut-off complaints; the "no wallet UI" constraint versus feature requests.
- Rail-B readiness: sequencer-operations runbook maturity, data-availability fee forecast at 10× pilot volume (blob pricing per §7), and the bridge/exit design — so that "migrate" at month 12 is an execution, not a project.

Two scoring caveats worth stating aloud. First, "regulatory posture" carries the largest single weight (20%) because MAS comfort gates the pilot itself — a rational bank weights the criterion that can stop the programme above the criteria that merely cost it. Second, "ecosystem and talent" is weighted at 5% precisely because the pilot cohort is captive and custodial; if the product ever goes open, that weight must rise before the Phase-2 comparison is run, or the matrix will mislead.

**Risk register (pilot view).**

| Risk | L / I | Mitigation |
|---|---|---|
| Consortium liveness and concentration: a small validator set is an outage and collusion surface | Med / High | 5–7 validators across at least three independent estates (Cymbal plus co-validators); failover runbooks; quarterly drills; apply §4.4's diversity logic in miniature |
| Private-transaction-manager compromise (Tessera-class key store) | Low / High | HSM custody; separate key hierarchy; rotation policy; cross-ref [cybersecurity_guide.md](cybersecurity_guide.md) |
| Regulatory reclassification mid-pilot (tokenized deposit vs security under the SFA) | Low / High | MAS-observed design from day one; product-level legal sign-off; contractual kill-switch redeeming in fiat |
| Smart-contract defect in fund-unit or NAV logic | Low / High | Two independent audits plus invariant fuzzing; bug-bounty; timelocked upgrade pattern (standards and incident history: [smart_contracts_guide.md](smart_contracts_guide.md)) |
| Middleware integration drift against the core-banking freeze | Med / Med | Adapter layer owned by the integration team; contract-first APIs; patterns per [enterprise_middleware_integration_platform_guide.md](enterprise_middleware_integration_platform_guide.md) |
| Conduct risk from the 24/7 promise meeting ops reality | Med / Med | Finality-based booking SLA; explicit cut-off and recovery communications; cohort capped at 20,000 |
| Phase-2 migration illusion: assuming C-to-B is free | Low / Med | EVM-only contracts from day one (no Besu-specific opcodes); export and replay tested during the pilot; NAV-critical state never written only into a private-transaction manager |

This is a pilot assessment, not a procurement: the month-6 decision gate is "continue, migrate, or kill" against the §11.3 criteria, with the weights re-scored by the business owners rather than the architect.

---

## 12. The Claims Audit

This audit aggregates the headline claims on which this guide stakes a ✅, ⚠ or ❌: the §1.2 key-facts inherited into later sections, plus the platform claims made new in §9–§11. Convention (from the front matter): ✅ verified at a primary source this pass, or inherited from a sibling guide that verified it — marked "inherited"; ⚠ reported, approximate, single-sourced or time-varying; ❌ refuted or not found. Unmarked statements in the body are domain-stable knowledge (roster facts, standards mechanics, protocol design) and are not audit rows.

### 12.1 The Verified-Facts Table

| Claim | Where stated | Status and source |
|---|---|---|
| Quorum lineage: JPMorgan (2016) → ConsenSys acquisition (announced Aug 2020) → Hyperledger Besu the surviving permissioned-EVM mainline | §1.2, §9.1 | ✅ inherited §1.2 (ConsenSys press release; Reuters) |
| MAS Project Guardian announced November 2022 | §9.2 | ✅ inherited — [tokenized_assets_guide.md](../banking/tokenized_assets_guide.md) (MAS media release) |
| US spot-ETH-ETF 19b-4 approvals 23 May 2024 | §1.2, §10.2 | ✅ inherited §1.2 (Reuters; SEC 19b-4 orders) |
| Spot-ETH-ETF trading from 23 July 2024 | §1.2, §10.2 | ✅ inherited §1.2 (Reuters) |
| MiCA = Reg (EU) 2023/1114; stablecoin rules 30 June 2024; full application 30 December 2024 | §1.2, §10.3 | ✅ inherited §1.2 (EUR-Lex; ESMA) |
| PoS time: 12-second slots, ~6.4-min epochs, normal-case finality ~2 epochs (~12.8 min) | §5.3, §11.3 | ✅ (ethereum.org PoS docs) |
| Dencun 13 March 2024 shipped EIP-4844 blobs; blob target raised 3→6/9 at Pectra | §6–§7, §11.2 | ✅ (ethereum.org history/roadmap; eips.ethereum.org EIP-7594) |
| EIP-1559 (London, Aug 2021): base fee burned; plain transfer 21,000 gas | §3.2 | ✅ (ethereum.org) |
| Execution/consensus client roster and the two-client architecture | §4.2–§4.3, §9.1 | ✅ (ethereum.org); roster facts domain-stable |
| MAS single-currency stablecoin framework finalised 2023–24 (PSN02 pathway) | §10.1 | ⚠ sibling-carried ([mas_regulations_guidelines_guide.md](../banking/mas_regulations_guidelines_guide.md)); chronology not re-verified this pass |
| Client-diversity snapshot: Geth 41% / Nethermind 38% / Besu 16%; Lighthouse 42.7% | §4.4 | ⚠ point-in-time (ethereum.org, Oct 2025); volatile by nature |
| Validator accounts >1.2 million, April 2026 | §5.3 | ⚠ time-varying (ethereum.org) |
| L2 exit/challenge windows (optimistic rails ~7 days) | §8, §11.3 | ⚠ per-network configuration, not a protocol constant |
| All §11 figures: cohort, NAV, weights, scores, risk ratings | §11 | Design fiction by construction — carry no verification status; deliberately excluded from audit |

### 12.2 Corrections and Discrepancies

- **Spot-ETF approval is not an SEC finding on ether's legal status.** The 23 May 2024 orders approved exchange rule changes under the Exchange Act; media shorthand reading them as "the SEC called ETH a commodity" is wrong. This guide states the narrower fact (§10.2) and nothing more.
- **"Quorum is dead" is half-right.** The brand retired after the 2020 ConsenSys acquisition, but the code lineage survives as Besu — and JPMorgan's own Onyx network runs on that lineage (the jpmorgan guide). "Abandoned enterprise Ethereum" confuses a brand with a codebase.
- **Project Guardian is not a MAS sandbox licence.** It is an industry pilot programme that MAS observes and facilitates; MAS's FinTech Regulatory Sandbox is a separate, application-based framework. Conflating the two overstates the regulatory blessing a Guardian-style pilot carries (⚠ sibling-carried; see the MAS guide for the sandbox distinction).
- **"Permissioned EVM is Ethereum" is wrong in the direction that matters.** Same bytecode, different consensus and membership — L1 finality and security properties do not transfer (§9.1). The reciprocal error — expecting privacy on L1 — is equally wrong: the public ledger is public; confidentiality requires permissioned rails, L2 designs or cryptographic techniques (the smart-contracts guide's ZK territory).
- **Arithmetic check.** The §11.3 weighted totals (1.90 / 3.25 / 4.45) were recomputed from the per-criterion scores and weights and agree; the numbers are illustrative, but the addition is not an error.
- **Dating discipline.** MiCA's two application dates (30 June 2024 stablecoins; 30 December 2024 full) and the two ETF dates (23 May / 23 July 2024) are stated consistently in §1.2, §10 and here.

### 12.3 What Could Not Be Verified

The honest list of what this pass could not confirm — items readers should treat as ⚠ or re-check at their date of reading:

- **Live network counters**: current validator counts, total ETH staked, client-diversity percentages, L2 TVL and data-availability fees. All are point-in-time; the snapshots in §4.4 and §5.3 are dated and will have moved (live sources are named in §4.4).
- **The spot-ETH-ETF issuer roster and aggregate AUM as of September 2026** — the launch-era roster is stated (§10.2); current composition was not re-verified this pass.
- **MAS's current posture (as of writing) on retail-facing tokenized deposits settling on public chains** — the cited posture is 2023–25-era and sibling-carried; no 2026 primary-source check was performed (§10.1).
- **Project Guardian's live pilot inventory and track list** — sibling-owned ([tokenized_assets_guide.md](../banking/tokenized_assets_guide.md)); only the November 2022 announcement date is inherited and marked ✅.
- **USD-stablecoin MiCA compliance status** (whether USDC/USDT hold EU e-money-token authorisations) — a moving target, ⚠ (§10.3).
- **Roadmap delivery dates**: Glamsterdam (Q4 2026) and Hegotá (2027) are in-development plans, not commitments (§6.3).
- **Market data of any kind**: ETH price, gas prices, blob base fees — time-varying by nature and deliberately not audited.
- **Every figure in §11** — design fiction by construction; unverifiable and intended to be.

---

## 13. The Glossary

Terms are defined as used in this guide; code-level terms carry cross-refs to the smart-contracts guide rather than full definitions here.

| Term | Meaning |
|---|---|
| L1 / L2 | Layer 1 = the Ethereum base chain (settlement, security, data availability); Layer 2 = a network settling to L1 that inherits its security while executing elsewhere (§8) |
| Rollup | An L2 that executes transactions off-chain and posts compressed data or proofs to L1; optimistic rollups assume validity unless challenged, ZK rollups prove it (§8) |
| Blob | EIP-4844's cheap, temporary data lane on L1 that rollups use for data availability (Dencun, §7) |
| Data availability (DA) | The guarantee that an L2's posted data is available for anyone to reconstruct state or challenge fraud; blobs made DA cheap (§7–§8) |
| Challenge window | The period during which an optimistic rollup's posted state can be disputed before it is treated as final (§8) |
| Sequencer | The L2 operator that orders transactions and posts batches to L1; its centralisation is the main L2 trust caveat (§8) |
| Bridge | The mechanism moving assets between L1 and L2 or between two chains; a primary hack surface (§8.3, web3 guide) |
| App-chain | An application-specific L2 (OP-Stack or Arbitrum Orbit pattern) run for one product or institution — rail B of §11 |
| EVM | The Ethereum Virtual Machine — the deterministic, stack-based execution engine every node runs (§3.3) |
| EOA | Externally Owned Account — the private-key-controlled account type (§3.1) |
| Gas / base fee | The unit pricing EVM work; since EIP-1559 a protocol-set base fee is burned and a tip rewards the proposer (§3.2) |
| Validator | A 32-ETH staker proposing and attesting blocks under PoS (§5.3) |
| Slot / epoch | 12-second slot (one block); 32-slot epoch (~6.4 min), the finality checkpoint rhythm (§5.3) |
| Finality | Economic finality under Casper-FFG: a checkpoint backed by 2/3 of staked ETH that reverting would mass-slash (§3.5, §5.3) |
| RANDAO | The beacon-chain randomness beacon that selects slot proposers (§5.3) |
| Fork choice | The rule (LMD-GHOST) by which nodes pick the canonical head from attestation weight (§5.3) |
| MEV | Maximal Extractable Value — value extracted from transaction ordering; the driver of PBS (§5.5) |
| PBS | Proposer-Builder Separation — separating block *proposal* from block *assembly* to contain MEV and censorship; out-of-protocol today (MEV-Boost), enshrined on the roadmap (§5.5) |
| Slashing | The penalty for validator equivocation or other consensus faults, up to full stake in mass events (§5.4) |
| Dencun | The March 2024 upgrade (Cancun-Deneb) that shipped EIP-4844 blobs (§6–§7) |
| EIP / ERC | Ethereum Improvement Proposal (protocol) / Ethereum Request for Comments (application standard) — the process lives in the smart-contracts guide |
| RWA tokenization | Issuing a token representing a claim to an off-chain asset — fund units, bonds, deposits (§9.2) |
| Tokenized deposit | A deposit liability represented on a ledger as a token (distinct from a stablecoin); the Cymbal pilot's core instrument (§11) |
| Permissioned EVM | An Ethereum-compatible network (e.g. Besu/QBFT) with whitelisted validators and, optionally, private transactions (§9.1) |
| ZK-proof | Zero-knowledge proof — a cryptographic proof of a statement's truth revealing nothing else; the validity basis of ZK rollups (§3.3, §8) |

---

## 14. Cross-References and the Closing Summary

**The cross-reference map.** This guide is the *platform/protocol* monograph of the repo's crypto cluster; the sibling guides own the layers above and beside it. When a topic appears in two guides, the sibling's treatment is the deep one.

| Guide | Relationship to this guide |
|---|---|
| [blockchain_technology_guide.md](blockchain_technology_guide.md) | Ledger basics and consensus taxonomy — §9.1's permissioned-vs-permissionless framing cross-refs here |
| [smart_contracts_guide.md](smart_contracts_guide.md) | The code layer: Solidity, ERC standards, the EIP process, incident history — §9.2's standards and §11.4's contract-risk mitigations inherit from it |
| [web3_technologies_guide.md](web3_technologies_guide.md) | The Web3 umbrella — DeFi/NFT economy, wallets, bridges as user infrastructure (§8.3's bridge caveats live here) |
| [defi_guide.md](defi_guide.md) | The DeFi application layer that gives L1/L2 rails their liquidity argument (§11.4) |
| [cybersecurity_guide.md](cybersecurity_guide.md) | Secure development and audit context for §11.4's smart-contract and key-management mitigations |
| [enterprise_middleware_integration_platform_guide.md](enterprise_middleware_integration_platform_guide.md) | The integration layer §11 assumes — bank middleware to chain-platform adapters |
| [tokenized_assets_guide.md](../banking/tokenized_assets_guide.md) | The institutional tokenization twin — RWAs, tokenized deposits, custody vendors, Project Guardian detail (§9.2) |
| [jpmorgan_chase_software_systems_guide.md](../banking/jpmorgan_chase_software_systems_guide.md) | JPM Coin / Onyx — the Quorum-lineage company record (§9.1–§9.2) |
| [mas_regulations_guidelines_guide.md](../banking/mas_regulations_guidelines_guide.md) | The MAS posture (§10.1) and the repo's Cymbal Bank persona conventions (§11.1) |
| [fireblocks_guide.md](../banking/fireblocks_guide.md), [bitunix_guide.md](../banking/bitunix_guide.md) | Custody infrastructure and the exchange layer (§11.1 custodial wallets; §10.3 rest-of-world notes) |
| [reap_global_guide.md](../banking/reap_global_guide.md) | Stablecoin-collateralised payments on Ethereum-ecosystem stablecoins (§10.3) |
| [enterprise_risk_management_guide.md](../banking/enterprise_risk_management_guide.md) | ERM framing for §11.4's risk register and the governance discussion |

**A reading path for a bank technologist.** If the whole guide is too much, the load-bearing sequence is §1 (the short answer and key facts) → §3 (architecture — finality and gas are the two concepts everything else leans on) → §5 (consensus — what staking, finality and slashing actually promise) → §8 (L2s — where the ecosystem is building) → §9–§10 (what banks and regulators have actually done) → §11 (the assessment that combines them). The glossary (§13) is the companion for that path, and §12 is the honesty check to revisit whenever a headline number from this guide is quoted in a board paper.

**The closing summary.** The monograph's spine, in one paragraph: Ethereum began as a whitepaper's bet on a "world computer" (§2), became a replicated state machine whose architecture rewards precision — accounts, gas, the EVM, finality (§3) — secured today by a diverse two-client network (§4) and by proof-of-stake economics whose finality, slashing and MEV mechanics a bank must read before trusting it (§5). The roadmap eras (§6), the Dencun blob lane (§7) and the L2 ecosystem (§8) define where the platform is going; the enterprise lineage (§9) and the regulatory milestones (§10) define where it has been accepted. The Cymbal Bank assessment (§11) is the whole guide in one decision: three rails, nine weighted criteria, one recommendation — start controlled on a permissioned EVM, keep the EVM-compatible exit path to an L2 or L1 open, and let the audit (§12) and glossary (§13) keep every claim honest and every term precise.

For an architect the structural fact is that every rail in this guide is defined by its relationship to the same shared state machine: permissioned EVMs borrow its code, L2s borrow its security, regulators now orbit its asset class, and the roadmap's own warnings — client diversity, PBS centralisation, uncommitted dates — are published in the open rather than hidden. Twelve years after Frontier, through a DAO fork, a difficulty bomb, a merge and a blob fork, Ethereum remains the world computer.