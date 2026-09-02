# Web3 Technologies: The Owned Web

**The Web3-Ecosystem Umbrella — the concept (Gavin Wood's 2014 coinage, verified at the primary source), the technology layers, the application layers (DeFi, NFTs, DAOs, identity, storage, oracles), the governance and tokenomics, the regulation, and the Cymbal Bank strategy assessment — sitting above the repo's crypto cluster, whose sibling deep-dives carry the mechanics this guide only maps.**

> **Author:** Jack Liu Shurui, Solution Architect
> **Context:** Technology Research — the umbrella/ecosystem view of Web3 for the repo's blockchain-crypto cluster; the sibling guides are the specific deep-dives that this guide cross-references instead of re-deriving
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Last Updated:** September 2026
> **Companion guides (technology/, same folder):** [blockchain_technology_guide.md](blockchain_technology_guide.md) (the ledger/platform mechanics — consensus, blocks, the EVM, L2s, bridges; cross-ref for the chain layer) · [smart_contracts_guide.md](smart_contracts_guide.md) (the EIP/ERC process, ERC-20/721/1155/4626 token standards, proxy patterns, SWC security classes, incident history; cross-ref for contracts, token standards and the NFT context) · [defi_guide.md](defi_guide.md) (the DeFi application-layer deep-dive — AMMs, lending, stablecoins, DeFi risks and regulation; cross-ref for DeFi) · [distributed_auth_guide.md](distributed_auth_guide.md) (identity architecture — authn/authz in distributed systems; cross-ref for the Web3-identity subsection) · [common_crawl_guide.md](common_crawl_guide.md) (the open web's memory — the "other" decentralised-content corpus; lightly relevant to the storage subsection) · [cloud_object_storage_lakehouse_guide.md](cloud_object_storage_lakehouse_guide.md) (centralised object storage as the contrast case for decentralised storage economics)
> **Companion guides (banking/, prefix `../banking/`):** [tokenized_assets_guide.md](../banking/tokenized_assets_guide.md) (the institutional twin — tokenized RWAs, tokenized deposits, custody vendors, Project Guardian; cross-ref for custody and tokenization) · [bitunix_guide.md](../banking/bitunix_guide.md) (the centralised exchange — the on/off-ramp layer) · [reap_global_guide.md](../banking/reap_global_guide.md) (stablecoin-collateralised cards — the stablecoin/payments bridge) · [payment_rails_guide.md](../banking/payment_rails_guide.md) (the global payment-rails map — how crypto rails relate to fiat rails) · [mas_regulations_guidelines_guide.md](../banking/mas_regulations_guidelines_guide.md) (the SG/MAS regulatory regime — the PS Act 2019, DPT licensing, notices; cross-ref for regulation, do not re-derive) · [market_data_consumption_guide.md](../banking/market_data_consumption_guide.md) (the market-data value chain — the institutional analogue of the oracle problem)

**How to use this guide:** Section 1 is the overview — the short answer, the key-facts table, why a bank should care, and how this guide relates to the blockchain and DeFi siblings. Section 2 is the concept, verified at primary sources: Wood's 2014 coinage and his 2018 restatement, the Web 1.0/2.0/3.0 framing, the defining principles, and the semantic-web terminological overlap. Section 3 is the debate — named advocates and named critics, with rebuttals, flagged where claims are unquantified. Section 4 is the technology-layer map — one or two sentences per layer plus the cross-ref to the sibling that carries the mechanics, plus the Web3-specific bits the siblings do not cover (dApp frontends, wallet-connect UX, browser extensions, IPFS-hosted frontends). Sections 5–8 are the application layers: DeFi (a verified-but-condensed layer map pointing at the defi guide), NFTs, DAOs, and identity/storage/oracles. Section 9 is governance and tokenomics — token voting, airdrops, supply schedules. Section 10 is the regulation — the MAS posture condensed (cross-ref'd, not re-derived) and the international regimes. Section 11 is the Cymbal Bank worked example — an illustrative Web3 strategy assessment, clearly marked design fiction built on the verified facts of the guide. Section 12 is the claims audit (✅/⚠/❌), with a "What Could Not Be Verified" subsection. Section 13 is the glossary. Section 14 is the cross-reference map and the closing summary. **Reading paths:** *Banking/architecture audience:* §1 → §10 → §11 → §4. *Engineer/developer:* §4 → §5 → §8 → §9. *Risk/compliance:* §3 → §9 → §10 → §11. *In a hurry:* §1, §2, §11, §14, and the glossary.

**Integrity convention.** Every factual claim in this guide carries one of three marks: **✅** verified this pass against a primary or authoritative source (named in the claims audit, §12); **⚠** flagged — reported, approximate, single-sourced, fast-moving, or contested (TVL figures and hack totals move with the market; treat them as orders of magnitude); **❌** refuted or not found. Unmarked statements are domain-stable technical knowledge (what a Merkle root is, what a liquidity pool does) rather than research claims. Where a sibling guide carries a verified fact, this guide marks it ✅ and cites the sibling rather than re-verifying from scratch — the repo's convention is that sibling-verified facts are inherited.

## Table of Contents

1. [The Overview](#1-the-overview)
   - 1.1 [The Short Answer](#11-the-short-answer)
   - 1.2 [The Key-Facts Table](#12-the-key-facts-table)
   - 1.3 [Why a Bank Should Care: The Cymbal Bank Lens](#13-why-a-bank-should-care-the-cymbal-bank-lens)
   - 1.4 [The Relationship to the Blockchain Guide and the DeFi Guide](#14-the-relationship-to-the-blockchain-guide-and-the-defi-guide)
2. [The Concept: Wood's 2014 Coinage and the "Owned Web"](#2-the-concept-woods-2014-coinage-and-the-owned-web)
   - 2.1 [The Coinage, Verified: "ĐApps: What Web 3.0 Looks Like" (17 April 2014)](#21-the-coinage-verified-đapps-what-web-30-looks-like-17-april-2014)
   - 2.2 [The Restatement: "Why We Need Web 3.0" (2018)](#22-the-restatement-why-we-need-web-30-2018)
   - 2.3 [The Framing: Read-Only → Read-Write → Read-Write-Own](#23-the-framing-read-only--read-write--read-write-own)
   - 2.4 [The Defining Principles](#24-the-defining-principles)
   - 2.5 [The Semantic-Web Lineage: A Terminological Overlap, Flagged](#25-the-semantic-web-lineage-a-terminological-overlap-flagged)
3. [The Debate: Advocates vs Critics](#3-the-debate-advocates-vs-critics)
   - 3.1 [The Critique, Named and Verified](#31-the-critique-named-and-verified)
   - 3.2 [The Rebuttals](#32-the-rebuttals)
   - 3.3 [What the Debate Leaves Unquantified](#33-what-the-debate-leaves-unquantified)
4. [The Technology Layers](#4-the-technology-layers)
   - 4.1 [The Layer Map](#41-the-layer-map)
   - 4.2 [The Web3-Specific Bits the Siblings Do Not Cover](#42-the-web3-specific-bits-the-siblings-do-not-cover)
5. [The Application Layers I: DeFi](#5-the-application-layers-i-defi)
   - 5.1 [The Protocol Map, Verified and Condensed](#51-the-protocol-map-verified-and-condensed)
   - 5.2 [The Scale Question: TVL Is an Order of Magnitude](#52-the-scale-question-tvl-is-an-order-of-magnitude)
6. [The Application Layers II: NFTs](#6-the-application-layers-ii-nfts)
   - 6.1 [The Standard and the Origin Moment](#61-the-standard-and-the-origin-moment)
   - 6.2 [The Critique That Stuck: Moxie's NFT Experiment](#62-the-critique-that-stuck-moxies-nft-experiment)
7. [The Application Layers III: DAOs](#7-the-application-layers-iii-daos)
   - 7.1 [The Concept and the 2016 Origin](#71-the-concept-and-the-2016-origin)
   - 7.2 [The Notable Cases](#72-the-notable-cases)
   - 7.3 [The Legal-Treatment Debate](#73-the-legal-treatment-debate)
8. [The Application Layers IV: Identity, Storage, Oracles](#8-the-application-layers-iv-identity-storage-oracles)
   - 8.1 [Web3 Identity: DIDs and Verifiable Credentials](#81-web3-identity-dids-and-verifiable-credentials)
   - 8.2 [Decentralised Storage: IPFS and Filecoin](#82-decentralised-storage-ipfs-and-filecoin)
   - 8.3 [Oracles: The Bridge to Off-Chain Reality](#83-oracles-the-bridge-to-off-chain-reality)
9. [The Governance and the Tokenomics](#9-the-governance-and-the-tokenomics)
   - 9.1 [Token-Based Governance](#91-token-based-governance)
   - 9.2 [Airdrops: The Distribution Mechanism](#92-airdrops-the-distribution-mechanism)
   - 9.3 [Tokenomics: Supply, Emissions, Vesting](#93-tokenomics-supply-emissions-vesting)
10. [The Regulation](#10-the-regulation)
    - 10.1 [The SG/MAS Posture, Condensed](#101-the-sgmas-posture-condensed)
    - 10.2 [The International Regimes](#102-the-international-regimes)
11. [The Cymbal Bank Worked Example: A Web3 Strategy Assessment](#11-the-cymbal-bank-worked-example-a-web3-strategy-assessment)
    - 11.1 [The Design-Fiction Frame](#111-the-design-fiction-frame)
    - 11.2 [The Opportunities](#112-the-opportunities)
    - 11.3 [The Risks](#113-the-risks)
    - 11.4 [The Regulatory Overlay](#114-the-regulatory-overlay)
    - 11.5 [The Scored Recommendation](#115-the-scored-recommendation)
12. [The Claims Audit](#12-the-claims-audit)
    - 12.1 [The Verified-Facts Table](#121-the-verified-facts-table)
    - 12.2 [What Could Not Be Verified](#122-what-could-not-be-verified)
13. [The Glossary](#13-the-glossary)
14. [Cross-References and the Closing Summary](#14-cross-references-and-the-closing-summary)

---

## 1. The Overview

### 1.1 The Short Answer

**Web3** (also written "Web 3.0") is the umbrella term for a proposed next iteration of the web built on blockchains, cryptographic tokens, and peer-to-peer networks, in which users — not platform companies — own the infrastructure, the data, and the value layer. The term was coined in the blockchain sense in **2014 by Gavin Wood**, co-founder of Ethereum, who defined it in an April 2014 essay as a "post-Snowden" web of zero-trust interaction: public information published, agreed information placed on a consensus-ledger, private information kept secret, and communication over encrypted channels with pseudonymous identities (verified at the primary source, §2.1). The one-line summary that captures the pitch is the progression **read → write → own**: Web 1.0 was read-only static pages, Web 2.0 added user-generated content on centralised platforms, and Web3 promises that users *own* their assets, identity, and data — hence this guide's subtitle theme, **the owned web** (the framing is popular in the a16z school of Web3 advocacy; ⚠ the exact slogan's provenance is marketing rather than a single citable primary source — §2.3, §12.2).

For a bank, the honest summary is more nuanced. Web3 is not one technology but a stack — chains, contracts, frontends, wallets, exchanges, stablecoins — and it is simultaneously (a) a real, verifiable technology movement with a documented history, (b) a speculative asset market whose crashes (LUNA, FTX, 2022) are cautionary events, and (c) a contested narrative that prominent technologists have called a myth. The bank's job is to separate the three: adopt the verifiable technology where it earns its keep (tokenization, programmable money, verifiable identity), price the risk of the market, and ignore the hype either way. The rest of this guide gives the map for doing exactly that.

### 1.2 The Key-Facts Table

| Fact | Value | Status |
|---|---|---|
| "Web3" coined (blockchain sense) | Gavin Wood, essay "ĐApps: What Web 3.0 Looks Like", originally posted 17 April 2014 on gavofyork's blog | ✅ §2.1 |
| Wood's restatement | "Why We Need Web 3.0" (2018): "an executable Magna Carta"; "It was over four years ago that I coined the term" | ✅ §2.2 |
| Web 1.0 / 2.0 eras | Web 1.0 ~1991–2004 (static pages); Web 2.0 "the web as platform", from ~2004 | ✅ §2.3 |
| Semantic-web overlap | Tim Berners-Lee's 1999 Semantic Web was *also* sometimes called "Web 3.0" — terminological collision | ⚠ §2.5 |
| First major DAO | The DAO — launched 30 Apr 2016; hacked 17 Jun 2016 (~3.6M ETH, ≈US$50M at the time); hard fork 20 Jul 2016 → Ethereum Classic | ✅ §7.1 |
| MakerDAO / DAI | DAI launched on Ethereum mainnet 18 Dec 2017; MakerDAO formed 2014 by Rune Christensen (rebranded Sky, Aug 2024) | ✅ §5.1 |
| CryptoKitties | Launched 28 Nov 2017 (Dapper Labs/Axiom Zen); congested Ethereum in Dec 2017; ERC-721 use case | ✅ §6.1 |
| ERC-721 | Created 24 Jan 2018 (Entriken, Shirley, Evans, Sachs); status **Final** | ✅ §6.1 |
| DeFi protocols | Uniswap 2 Nov 2018; Compound Sep 2018; Aave Jan 2020 (rebrand of ETHLend); Curve Jan 2020; Yearn early 2020 (YFI Jul 2020); Lido 2020 | ✅/⚠ §5.1 |
| UNI airdrop | 16 Sep 2020; 15% of 1B supply retroactive to historical users (snapshot 1 Sep 2020) | ✅ §9.2 |
| Nouns DAO | Launched Aug 2021; one Noun NFT auctioned daily; 100% of proceeds to treasury | ✅ §7.2 |
| ENS airdrop / DAO | $ENS claiming opened 8 Nov 2021; ENS DAO launched with it | ✅ §7.2, §9.2 |
| ARB airdrop | Announced 16 Mar 2023; ARB live 23 Mar 2023 (Arbitrum Foundation) | ✅ §9.2 |
| IPFS / Filecoin | IPFS: Juan Benet/Protocol Labs, 2014 paper, alpha Feb 2015; Filecoin: ICO Aug 2017 (>US$200M), mainnet Oct 2020, FIL token | ✅ §8.2 |
| W3C DID | Decentralized Identifiers v1.0 — W3C Recommendation 19 July 2022 | ✅ §8.1 |
| MiCA | Regulation (EU) 2023/1114 — in force Jun 2023; ART/EMT rules from 30 Jun 2024; CASP regime from 30 Dec 2024 | ✅ §10.2 |
| Named critiques | Moxie Marlinspike, "My first impressions of web3" (7 Jan 2022); Stephen Diehl, "Web3 is bullshit" (Nov 2021); David Gerard, *Attack of the 50 Foot Blockchain* (2017); Jack Dorsey's Dec 2021 tweets | ✅/⚠ §3.1 |

### 1.3 Why a Bank Should Care: The Cymbal Bank Lens

Cymbal Bank's interest in Web3 is not the token market — it is the **infrastructure migration** underneath it. Four threads matter to a Singapore-licensed bank:

1. **Tokenization of real-world assets** — the same rails that run DeFi can run tokenized money-market funds, bonds, and deposits; MAS is actively piloting this under Project Guardian (cross-ref [tokenized_assets_guide.md](../banking/tokenized_assets_guide.md), which carries the institutional deep-dive and the Project Guardian record). Web3's wallet-and-contract stack is the *plumbing* for that.
2. **Programmable money** — stablecoins and tokenized deposits are the "programmable money" leg of the stack; the bank's own stablecoin strategy (cross-ref [reap_global_guide.md](../banking/reap_global_guide.md) for the stablecoin-collateralised-card analogue) is a Web3-ecosystem decision.
3. **Identity and verifiable credentials** — W3C-standardised DIDs and verifiable credentials (✅ §8.1) are a candidate primitive for onboarding, KYC reuse, and client self-sovereignty; the bank already has a distributed-auth architecture guide ([distributed_auth_guide.md](distributed_auth_guide.md)) that this plugs into.
4. **Custody revenue and the exchange layer** — institutional custody of digital assets is a fee business the bank can operate under licence, and the exchange layer (cross-ref [bitunix_guide.md](../banking/bitunix_guide.md)) is where the retail flow lives.

The counterweight: every one of these threads runs through a market that lost hundreds of billions in 2022 (FTX, LUNA), a regulatory landscape that is still being written (MAS, MiCA, the US SEC), and a technology whose own critics say it is not decentralised in practice (§3). The Cymbal Bank worked example (§11) scores all of this into a recommendation.

### 1.4 The Relationship to the Blockchain Guide and the DeFi Guide

This guide is the **ecosystem umbrella**; the siblings are the deep-dives. The division of labour:

- [blockchain_technology_guide.md](blockchain_technology_guide.md) is the **ledger layer**: consensus, blocks, the EVM, platforms, L2s, bridges. This guide's §4 maps that layer in one paragraph and points there for mechanics. Its §9 ("Decentralized Finance") is a ~85-line DeFi overview; the defi guide expands it.
- [defi_guide.md](defi_guide.md) is the **DeFi application layer**: AMMs, lending, stablecoins, composability, DeFi-specific risks, and DeFi regulation (§9 there). This guide's §5 is a verified-but-condensed layer map that *points* at the defi guide — it does not re-derive the mechanics.
- [smart_contracts_guide.md](smart_contracts_guide.md) is the **contract and standards layer**: the EIP/ERC process, ERC-20/721/1155/4626, proxies, SWC security classes, and the incident history. This guide's §6 (NFTs) and §9 (tokenomics) cross-ref its §5 for the standards themselves.

Rule of thumb: if a fact is about *how a chain, contract, or AMM works*, it lives in the sibling; if it is about *how the Web3 ecosystem fits together, who is arguing about it, and what a bank should do*, it lives here.

## 2. The Concept: Wood's 2014 Coinage and the "Owned Web"

### 2.1 The Coinage, Verified: "ĐApps: What Web 3.0 Looks Like" (17 April 2014)

The primary source is verified this pass: **gavwood.com/dappsweb3.html** carries Wood's essay **"ĐApps: What Web 3.0 Looks Like"** with the note *"originally posted Wednesday, 17 April 2014 on gavofyork's blog Insights into a Modern World"* ✅. The essay is the canonical statement of the original vision, and its claims are worth quoting precisely:

- Web 3.0 is the **"post-Snowden" web** — a reimagination of the web "with a fundamentally different model for the interactions between parties," because entrusting information to organisations "is a fundamentally broken model."
- The information policy is: **"Information that we assume to be public, we publish. Information that we assume to be agreed, we place on a consensus-ledger. Information that we assume to be private, we keep secret and never reveal."** Communication happens over encrypted channels with pseudonymous identities — "we engineer the system to mathematically enforce our prior assumptions, since no government or organisation can reasonably be trusted."
- The stack has **four components**: (1) a decentralised, encrypted static-content publication system (content-addressed, BitTorrent-style); (2) an identity-based pseudonymous messaging system; (3) **the consensus engine** — "a means of agreeing some rules of interaction, in the knowledge that future interactions will automatically and irrevocably result in the enforcement exactly as specified… the first workable example of which is the Ethereum project"; and (4) the integrated user interface — a browser-like front end for what Wood calls the **ĐApp** ("né webpage/website").
- The close: **"Say 'hello' to Web 3.0, a Secure Social Operating System."**

Note the accent: Wood wrote **ĐApps** (the essay title on gavwood.com), which is the form used in the primary source; later usage normalised to "dApps." The task of verifying this pass confirmed the essay's existence, date, and content at the primary URL ✅. (The commonly-cited fuller title variant "DApps: What Web 3.0 Looks Like and the New Stack" appears in secondary literature; the page itself carries the shorter title — flagged ⚠ in §12.2 as a citation-form discrepancy, not a factual one.)

### 2.2 The Restatement: "Why We Need Web 3.0" (2018)

Four years later Wood restated and sharpened the thesis in **"Why We Need Web 3.0"** (2018). The original gavwood.com URL for the essay no longer resolves (404 this pass ⚠), but the full text was verified this pass via a mirror that preserves the original, with the source attribution *gavofyork.medium.com/why-we-need-web-3-0-5da4f2bf95ab* ✅. Verified content, quoting directly:

- **"It was over four years ago that I coined the term 'Web 3.0.'"** — Wood himself dating the coinage to ~2014, consistent with the April 2014 essay ✅.
- **"The internet today is broken by design."** — centralisation concentrates "wealth, power and influence" in "the greedy, the megalomaniacs, or the plain malicious."
- Web 3.0 is **"an inclusive set of protocols to provide building blocks for application makers"** replacing HTTP/AJAX/MySQL, giving users "strong and verifiable guarantees about the information they are receiving, what information they are giving away, and what they are paying."
- The famous line: consider Web 3.0 **"an executable Magna Carta — 'the foundation of the freedom of the individual against the arbitrary authority of the despot.'"**
- Browsers become "wallets or key stores," representing "a person's assets and identity online, allowing us to pay for something, or prove who we are, without needing to appeal to a bank or identity service."
- Honesty about pace: adoption "will be neither fast nor clean," and "the timeline could be measured in decades rather than months."

These two essays are the concept's canon: 2014 gives the architecture, 2018 gives the rhetoric ("executable Magna Carta") that Web3's advocates still use.

### 2.3 The Framing: Read-Only → Read-Write → Read-Write-Own

The popular framing maps the three eras onto three verbs (⚠ the slogan is advocacy shorthand, not a standardised definition — §12.2):

| Era | User posture | Value capture |
|---|---|---|
| **Web 1.0** (~1991–2004) | "Read" — static pages; most users are consumers, not producers ✅ | Publishers/ISPs |
| **Web 2.0** (~2004–) | "Read-Write" — user-generated content on centralised platforms, "the web as platform" ✅ | Platform companies (the "Big Tech" critique) |
| **Web3** | "Read-Write-Own" — users own assets, identity, and data via keys and tokens; platforms become protocols | Token holders |

The Web 1.0/2.0 dating is verified at the Wikipedia "Web3" article's account ✅ (Web 1.0 ~1991–2004 static; Web 2.0 "the web as platform" from ~2004, O'Reilly-era). The "read-write-own" formulation is the a16z-school pitch — the same school that published Chris Dixon's essay "Why Web3 Matters" (future.a16z.com), which Moxie Marlinspike cites as "the general thesis" of the movement ✅ (the essay URL is linked from Moxie's verified essay; the standalone "Read Write Own" book title is ⚠ not re-verified this pass, §12.2).

### 2.4 The Defining Principles

Across the two Wood essays and the ecosystem's own self-description, three principles recur (unmarked: domain-stable characterisation, with the verified citations where they exist):

1. **Decentralisation** — no single party controls the ledger, the namespace, or the platform. Wood's 2014 essay: the model "draws its strength from the network effect of consensus" rather than from ICANN/Verisign-style central trust ✅.
2. **Ownership** — users hold keys, not accounts; assets and identity are self-custodied. Wood's 2018 essay: wallets-as-browsers represent "a person's assets and identity online" ✅.
3. **Trustlessness and verifiability** — interactions are "mathematically enforced" (2014) and give "strong and verifiable guarantees" (2018) ✅; you verify, rather than trust.

The tension to hold onto for §3: critics agree these are the *stated* principles and dispute that the *practice* achieves them (Moxie: users still trust Infura/Alchemy and MetaMask; Diehl: the economics re-centralise; §3.1).

### 2.5 The Semantic-Web Lineage: A Terminological Overlap, Flagged

"Web 3.0" was **not invented** by the blockchain movement. Tim Berners-Lee's **Semantic Web** concept (dating to his 1999 book *Weaving the Web* and the W3C's 2001 *Scientific American* article) was also frequently labelled "Web 3.0" — a web of machine-readable linked data rather than a web of tokens. The Wikipedia "Web3" article explicitly flags the collision: "The term is sometimes confused with the Semantic Web, as both have sometimes been referred to as 'Web 3.0'", and notes Web3 is "distinct from Tim Berners-Lee's 1999 concept of a Semantic Web" ✅. The two uses are now disambiguated in practice — "Web3" (no space, blockchain sense) vs "Web 3.0"/"Semantic Web" (linked-data sense) — but older literature and many non-technical articles still mix them. Mark this ⚠ in the sense that the terminological overlap is real, documented, and a standing source of confusion rather than a settled matter; any Web3 reading list should carry the caveat.

## 3. The Debate: Advocates vs Critics

Web3 may be the only technology movement whose canonical texts include its own obituaries. This section presents the named critiques (verified where possible), the named rebuttals, and then flags what the debate itself leaves unquantified. The goal is not to settle the argument — it is to give the reader the actual texts, with their actual dates, so the argument can be read first-hand.

### 3.1 The Critique, Named and Verified

**Moxie Marlinspike, "My first impressions of web3" (7 January 2022, moxie.org) ✅.** Verified at the primary URL this pass. Marlinspike (founder of Signal) built two dApps and an NFT to test the thesis that "web1 was decentralized, web2 centralized everything into platforms, and web3 will decentralize everything again" (citing the a16z essay as "the general thesis"). His findings, in his own words:

- **"People don't want to run their own servers, and never will"** — and a protocol "moves much more slowly than a platform"; decentralised systems "often remain stuck in time."
- **The client/server reality:** "Blockchains are designed to be a network of peers, but not designed such that it's really possible for your mobile device or your browser to be one of those peers." Clients therefore call remote nodes: "Almost all dApps use either Infura or Alchemy… even when you connect a wallet like MetaMask to a dApp… MetaMask is just making calls to Infura!" The responses "aren't even signed" — clients trust two companies' JSON blobs, which "doesn't seem like the best privacy situation" ("Imagine if every time you interacted with a website in Chrome, your request first went to Google").
- **The NFT critique:** NFT metadata is usually a URL with "no hash commitment" — "Anyone with access to that machine… can change the image, title, description… regardless of whether or not they 'own' the token." He minted an NFT that displayed a 💩 emoji in wallets while looking normal on marketplaces — "What you bid on isn't what you get." OpenSea then removed it without explanation.
- His conclusion: web3 is at risk of becoming **"web2x2 (web2 but with even less privacy)"**, and "I don't think it's on a trajectory to deliver us from centralized platforms."

**Stephen Diehl, "Web3 is bullshit" (November 2021, stephendiehl.com).** ⚠ The essay's existence and authorship are corroborated by multiple independent references (including the *Tech Won't Save Us* podcast episode "Web3 is a Scam, Not a Revolution w/ Stephen Diehl," which discusses it) ✅, but the primary page could not be fetched this pass — the domain resolved to a blocked address and the Wayback Machine holds no capture ⚠ (§12.2). Diehl's published arguments, as summarised in the corroborating record: Web3 is best understood as a marketing rebrand of crypto-asset speculation; "decentralization" rhetoric functions to avoid securities regulation and consumer protection; and the technical claims (decentralised identity, decentralised storage) are either already-solved problems or worse versions of centralised equivalents. Because the primary text was not re-read this pass, attribute the specifics at one remove: the *existence* of the essay is ✅, its exact claims are ⚠ as summarised.

**David Gerard, *Attack of the 50 Foot Blockchain* (2017, book; blog at davidgerard.co.uk/blockchain).** ✅ The blog is verified live this pass (a long-running "Bitcoin, Blockchain, Ethereum and Smart Contracts" news-and-analysis site by Gerard, a journalist and librarian). The book — *Attack of the 50 Foot Blockchain: Bitcoin, Blockchain, Ethereum and Smart Contracts* (2017) — is the book-length statement of the "blockchain is solutionism" critique: blockchains as a solution looking for a problem, cryptocurrency as speculation plus crime, and the "enterprise blockchain" pitch as vapourware. (Book details ⚠ single-source: the site and the critique are verified; the book's publisher/edition details were not re-verified this pass.)

**Jack Dorsey's December 2021 tweets.** ✅ The event is verified: *Bloomberg*, 21 December 2021 — "Jack Dorsey Stirs Uproar by Dismissing Web3 as a Venture Capitalists' Plaything" — reporting Dorsey's dismissal of web3 as a venture-capital-owned buzzword (also noted in the Wikipedia "Web3" reception account: Musk and Dorsey "have argued that web3 only serves as a buzzword or marketing term"). The famous tweet text — *"You don't own 'web3.' The VCs and their LPs do. It will never escape their incentives"* — is ⚠ quoted from widely-reproduced reporting rather than re-verified at the (now-archived) tweet this pass.

**The "Web3 is a myth" arguments.** The sustained versions argue that: (a) adoption is negligible outside speculation — Bloomberg's Olga Kharif, December 2021: the idea "is a long way from proving its use beyond niche applications, many of them tools aimed at crypto traders" ✅ (as reported in the Wikipedia "Web3" reception section); (b) the economics re-centralise — the Quartz headline "Web3 is not the decentralized utopia you've been promised" (19 January 2022) ✅; (c) the 2022 collapses (LUNA/UST in May, FTX in November) demonstrated that the "trustless" stack still concentrates catastrophic risk in a few opaque intermediaries ⚠ (interpretation; the events themselves are documented in the repo's crypto guides); and (d) environmental and harm concerns — the Wikipedia account lists energy use, pyramid-scheme characterisations, and regulatory-arbitrage readings as standing criticisms ✅.

### 3.2 The Rebuttals

**Vitalik Buterin's response to Moxie (January 2022) ✅.** The exchange is verified: *Fortune*, 10 January 2022 — "Ethereum founder Vitalik Buterin defends web3 from Moxie Marlinspike" — reporting Buterin's counter that the centralisation Moxie observed (Infura/Alchemy dependence, unsigned reads) is an *early-stage* convenience, not a design endpoint: the roadmap (light clients, in-browser verification, statelessness, account abstraction) is precisely about making client-side verification real. The rebuttal's structure — "what you observed is the current stack, and the whole point of the roadmap is to fix it" — is the movement's standard answer to the "it's just web2" critique.

**The "it's early / the stack is being built" argument.** Wood himself pre-empted it in 2018: adoption "will be neither fast nor clean," "the timeline could be measured in decades rather than months" ✅, and Bitcoin/IPFS-style components "will gain traction first, probably in niche areas, much as Linux found traction 'under the radar' in server backrooms." On this view, Moxie's 2022 snapshot is a 1995-era photo of the internet: technically accurate, historically premature.

**The empirical rebuttal.** Independent of rhetoric, a set of verifiable facts have *already* moved: a W3C Recommendation for DIDs (19 July 2022, ✅ §8.1); a functioning multi-billion-dollar DeFi economy whose TVL is measurable at DefiLlama (✅ §5.2); ERC-721/ERC-20 standards finalised and implemented at scale (✅ §6.1); and — most relevant to a bank — central banks and regulators running tokenization pilots (MAS Project Guardian, cross-ref [tokenized_assets_guide.md](../banking/tokenized_assets_guide.md)) rather than dismissing the stack. The "myth" claim has to be squared with the fact that the standards bodies, the banks, and the regulators are all building on the technology.

### 3.3 What the Debate Leaves Unquantified

Almost every quantified claim in the Web3 debate is an estimate, and this guide flags them rather than certifying them:

- **"Web3 is 1% of the web"** (or any variant) — no canonical measurement exists; there is no agreed definition of what counts as a "Web3 site" (a dApp? an IPFS page? a page with a wallet button?) ⚠.
- **Adoption/active-user figures** — wallet counts and DAU estimates come from vendor dashboards with different counting rules ⚠.
- **TVL and market-cap figures** — time-varying by definition; DefiLlama (✅ §5.2) is the least-bad reference but its numbers are still a snapshot, not a truth ⚠.
- **"Decentralisation scores"** — academic attempts exist, but none is canonical ⚠.

The honest summary of the debate: both sides argue from the same verified facts and disagree about *which time horizon counts*. Critics photograph the present stack; advocates film the roadmap. A bank should read both, and then look at what the regulators are actually licensing.

## 4. The Technology Layers

### 4.1 The Layer Map

Web3 is a stack, not a product. From the bottom up, with the sibling that carries the mechanics:

| Layer | What it is | Mechanics live in |
|---|---|---|
| **L1/L2 chains** | The settlement layer — Ethereum and other L1s; L2 rollups (Arbitrum, Optimism, Base) for scale; cross-chain bridges | [blockchain_technology_guide.md](blockchain_technology_guide.md) (consensus, EVM, L2s, bridges, platforms) |
| **Smart contracts** | The program layer — token standards (ERC-20/721/1155/4626), DeFi logic, DAO governance contracts | [smart_contracts_guide.md](smart_contracts_guide.md) (standards, security, incidents) |
| **dApp frontends** | The UI layer — ordinary web apps (React et al.) that talk to a chain via an RPC provider; often served from IPFS | This guide §4.2 (the siblings do not cover it) |
| **Wallets & custody** | The key layer — browser-extension/self-custody wallets (MetaMask et al.); institutional custody (Fireblocks, BitGo, Coinbase Custody as vendors) | [tokenized_assets_guide.md](../banking/tokenized_assets_guide.md) §8 (custody); [bitunix_guide.md](../banking/bitunix_guide.md) (exchange custody) |
| **Exchanges & on/off-ramps** | The fiat-crypto gateways — CEXs (Bitunix et al.) and DEXs (Uniswap et al.) | [bitunix_guide.md](../banking/bitunix_guide.md) (CEX); [defi_guide.md](defi_guide.md) §4 (DEX/AMM) |
| **Stablecoins** | The money layer — the unit of account that makes DeFi usable and payments programmable | [defi_guide.md](defi_guide.md) §6 (design space, risks); [reap_global_guide.md](../banking/reap_global_guide.md) (stablecoin payments); [tokenized_assets_guide.md](../banking/tokenized_assets_guide.md) §2 |
| **Identity / storage / oracles (horizontal)** | The trust plumbing — DIDs & verifiable credentials; IPFS/Filecoin; price/data feeds | This guide §8; [distributed_auth_guide.md](distributed_auth_guide.md); [market_data_consumption_guide.md](../banking/market_data_consumption_guide.md) |

Two sentences per layer, as promised: **chains** provide the shared, append-only settlement ledger (consensus, blocks, finality — see the blockchain guide); **contracts** encode the rules as deterministic programs (see the smart-contracts guide); **frontends** are the web apps users actually see (see §4.2); **wallets** hold the keys that *are* the accounts (self-custody vs institutional custody is the key design fork — see the tokenized-assets guide's custody section for the institutional side); **exchanges** convert fiat↔token and provide the retail rails (CEX for on/off-ramp, DEX for permissionless trading); **stablecoins** provide the price-stable money leg (see the defi guide and the reap guide for the two institutional angles); and **identity, storage, oracles** are the three horizontals that let the stack reach users, hold data, and see the real world (see §8).

### 4.2 The Web3-Specific Bits the Siblings Do Not Cover

The siblings cover chains, contracts, and DeFi mechanics in depth; they do not cover the *experience* layer, which is where Web3's distinctive (and most-criticised) design decisions live:

**dApp frontends are just websites.** Moxie's verified observation: "there is nothing particularly 'distributed' about the apps themselves: they're just normal react websites. The 'distributedness' refers to where the state and the logic/permissions for updating the state lives: on the blockchain" ✅. A dApp is a web app whose state lives on-chain; the frontend itself is hosted like any web app — often, in a Web3-native touch, **on IPFS** (content-addressed, so the UI has a hash and can be served by any gateway; pinning services keep it alive — §8.2).

**The "connect wallet" button is the Web3 login.** Where Web2 apps have username/password, Web3 apps have a wallet-connect flow: the dApp asks the wallet (browser extension or mobile app, via WalletConnect-style protocols) for the user's address and for signed transactions. UX consequences: (a) every action is an explicit signature — which users approve blindly, which is how phishing and drainer scams work ⚠; (b) the wallet vendor becomes a choke point — MetaMask routes through Infura (Moxie ✅); (c) "wallet-as-identity" means the address is a pseudonymous account with no recovery — lose the key, lose the assets (the "not your keys, not your coins" problem, unmarked domain knowledge).

**Browser-extension wallets.** MetaMask (2016) made Ethereum usable in a browser tab and became the default Web3 client; the extension injects a provider into pages, holds keys locally, and signs transactions. The critique follows immediately: the *key custody* is local, but the *chain access* is centralised via RPC providers — the exact asymmetry Moxie documented ✅. Account abstraction (ERC-4337) is the ecosystem's fix-in-progress: wallets as smart contracts, with recovery, sponsored gas, and social login ⚠ (the EIP exists and is widely implemented; details move fast — cross-ref the smart-contracts guide's standards process for how to read an EIP's maturity).

**The RPC-provider bottleneck.** Public chains are decentralised; *accessing* them currently funnels through a small set of API providers (Infura, Alchemy — "basically two companies," Moxie ✅). This is the single most-cited gap between Web3's claims and its practice, and the thing most likely to change as light clients and in-browser verification mature (§3.2).

**On/off-ramps are where the regulation bites.** The decentralised stack stops at the fiat border: converting SGD↔tokens happens at a licensed exchange (cross-ref [bitunix_guide.md](../banking/bitunix_guide.md) for the CEX anatomy, and [mas_regulations_guidelines_guide.md](../banking/mas_regulations_guidelines_guide.md) for the licensing that gates it). Every retail Web3 user crosses this border; every bank's Web3 strategy starts and ends here (§10, §11).

## 5. The Application Layers I: DeFi

DeFi is the largest *functioning* application of the Web3 stack — a parallel financial system running on smart contracts. The full mechanics (AMMs, lending, stablecoins, composability, DeFi-specific risks, DeFi regulation) live in [defi_guide.md](defi_guide.md); this section is the verified layer map that points at it.

### 5.1 The Protocol Map, Verified and Condensed

| Protocol | What it is | Verified origin |
|---|---|---|
| **MakerDAO / DAI** | Overcollateralised stablecoin DAO — mint DAI against collateral; MKR holders govern parameters | DAI on Ethereum mainnet **18 Dec 2017** ✅ (Wikipedia, Dai article); MakerDAO formed 2014 by Rune Christensen ✅; rebranded Sky Aug 2024 ✅ |
| **Uniswap** | The reference AMM/DEX — constant-product pools, x·y=k; v2 May 2020, UNI Sep 2020 | Created **2 Nov 2018** by Hayden Adams ✅ (Wikipedia) |
| **Compound** | First algorithmic money market — cTokens, utilisation-priced rates; COMP liquidity mining (Jun 2020) ignited "DeFi Summer" | **September 2018** ✅ (verified record of [defi_guide.md](defi_guide.md) §3.2: founded 2017 by Robert Leshner and Geoff Hayes) |
| **Aave** | Pooled lending with aTokens, variable/stable rates, flash loans; GHO stablecoin 2023 | **January 2020** relaunch of ETHLend (founded 2017 by Stani Kulechov) ✅ (Wikipedia) |
| **Curve** | StableSwap AMM for pegged assets; veCRV vote-escrow ("Curve Wars") | **January 2020** ✅ (IQ.wiki; Curve docs: StableSwap algorithm by Michael Egorov, Nov 2019) |
| **Yearn** | Yield aggregation/vaults | Early **2020** as Andre Cronje's personal tool; **YFI "fair launch" July 2020** ✅ (Binance Academy; Unchained) |
| **Lido** | Liquid staking — stETH; largest staking derivative | **2020** ⚠ (widely reported late-2020 launch; primary launch post not re-verified this pass) |
| The label "DeFi" | The movement name | Coined in an **August 2018** Telegram chat ✅ (verified record of [defi_guide.md](defi_guide.md): widely reported, anecdotal transcript ⚠) |

The four building blocks, one sentence each (mechanics in the defi guide): **DEX/AMM** — algorithmic price discovery against liquidity pools (Uniswap/Curve); **lending** — overcollateralised, pool-based money markets with algorithmic rates (Aave/Compound); **stablecoins** — the price-stable money leg (DAI's collateralised design vs USDT/USDC's centralised reserves vs the failed LUNA/UST algorithmic design, defi guide §6); **yield/staking** — vaults and liquid staking that turn positions into composable yield (Yearn/Lido). The property that makes it "Web3-native" is **composability** — every protocol's contracts call every other protocol's contracts ("money legos"), which is also the source of systemic risk (a hack in one pool propagates; defi guide §7–§8).

### 5.2 The Scale Question: TVL Is an Order of Magnitude

Total Value Locked (TVL) — the dollar value of assets deposited into DeFi contracts — is the ecosystem's headline metric, and the correct way to use it is as an order of magnitude, not a precise number:

- The reference aggregator is **DefiLlama** (defillama.com): open and transparent DeFi analytics tracking TVL, volume, and yields across thousands of protocols and hundreds of chains, with an open-source methodology ✅ (site verified this pass).
- Historical trajectory ⚠ (press estimates, order of magnitude): roughly US$1B at the start of 2020 → ~US$15B+ by year-end ("DeFi Summer", per the defi guide's verified timeline) → tens of billions through 2021 → a peak widely reported around **US$180B in late 2021** → a brutal drawdown in 2022 (LUNA/FTX) → partial recovery on L2s and new chains through 2023–2026. Exact figures at any date are snapshots; cite DefiLlama, not memory.
- Why TVL overstates health: it counts double-counted/composited positions (a token locked in a vault that is itself collateral elsewhere), and it says nothing about users or revenue ⚠ (methodological caveat, widely acknowledged).

For the bank's purposes: DeFi's scale is real but niche-relative (a fraction of global bank assets), its users are predominantly crypto-native, and its regulatory perimeter is still being drawn (defi guide §9). It matters to Cymbal Bank as a *capability demonstration* (what programmable money can do) and as a *risk laboratory* (what happens when code, leverage, and panic meet — §11.3), not as a near-term revenue market.

## 6. The Application Layers II: NFTs

### 6.1 The Standard and the Origin Moment

The NFT standard and its breakout moment are both verified:

- **ERC-721** — "Non-Fungible Token Standard" — is **Final** on eips.ethereum.org ✅: authors **William Entriken, Dieter Shirley, Jacob Evans, Nastassia Sachs**; created **24 January 2018**; requires EIP-165; frames NFTs as "deeds" for "distinguishable" assets — physical property, virtual collectibles, negative-value assets (loans). Dieter Shirley is the Dapper Labs connection: he was CTO/co-founder at Axiom Zen/Dapper Labs, the CryptoKitties shop ✅ (the EIP's own references list CryptoKitties as the flagship implementation).
- **CryptoKitties** — launched **28 November 2017** by Dapper Labs (spun out of Axiom Zen, Vancouver) ✅ (Wikipedia): breedable, tradeable virtual cats as ERC-721 tokens; by December 2017 the game congested the Ethereum network — the first mainstream demonstration that NFTs were real demand and that the chain could not yet carry it ✅.
- The standards deep-dive (interface functions, safeTransfer, ERC-1155, metadata) lives in [smart_contracts_guide.md](smart_contracts_guide.md) §5.2 — cross-ref, don't re-derive.
- The 2021 mania ⚠: NFT trading volume exploded in 2021 (the Beeple/"Everydays" work sold at Christie's for **US$69.3M in March 2021** — widely reported press fact, not re-verified this pass); collections like CryptoPunks and Bored Ape Yacht Club became cultural artefacts; then the market crashed with the 2022 bear. Treat all NFT price/volume figures as ⚠ press estimates.

### 6.2 The Critique That Stuck: Moxie's NFT Experiment

The most durable NFT critique is Moxie Marlinspike's verified January 2022 experiment (§3.1): most NFT metadata is a **URL with no hash commitment**, so the "asset" a buyer owns is a pointer to a mutable web server — he minted an NFT that rendered as a 💩 emoji in wallets while looking normal on marketplaces ("What you bid on isn't what you get"), and OpenSea removed it without explanation ✅. The ecosystem's partial answers: **on-chain SVG/generative art** (all data in the contract), **IPFS/Arweave metadata** (content-addressed, so at least the bytes are pinned — §8.2), and marketplace-level content policies. The residual truth: an ERC-721 token certifies *ownership of a registry entry*; what the entry *means* depends on where its metadata lives. A bank touching NFTs (collectibles custody, tokenized deeds, fund units) should treat the metadata layer as a first-class risk, exactly as the smart-contracts guide treats proxy upgradeability.

## 7. The Application Layers III: DAOs

### 7.1 The Concept and the 2016 Origin

A **DAO** (Decentralized Autonomous Organization) is an organisation whose rules are encoded in smart contracts and whose decisions are made by token-holder voting — no board, no CEO, treasury and governance on-chain. The concept's origin moment is **The DAO** (2016), verified at Wikipedia's account ✅:

- **Launched 30 April 2016** via a 28-day crowdsale; it became one of the largest crowdfunding campaigns in history — >US$150M of ETH by 21 May 2016, ~14% of all Ether issued at the time; >11,000 investors.
- **Hacked 17 June 2016** via a recursive-call vulnerability: ~3.6M ETH (~one-third of the committed funds) drained, worth ≈US$50M at the time (reported range US$50–60M ⚠ — the smart-contracts guide's incident table carries the same figure with the same caveat).
- **The community hard-forked Ethereum on 20 July 2016** to restore the funds, splitting the chain — the original chain continued as **Ethereum Classic** ✅. The fork remains the canonical example of "code is law" colliding with "people are the law."
- **Regulatory aftermath:** the SEC's July 2017 investigative report (Release 2017-131) concluded DAO tokens were securities and US securities law could apply to their offer and sale ✅ (per Wikipedia's citation of the SEC release).

The lessons that still define DAO design: code review matters more than governance theatre; "autonomous" does not mean "safe"; and when a DAO fails at scale, the resolution is political (a fork), not technical.

### 7.2 The Notable Cases

- **MakerDAO** — the longest-lived meaningful DAO: MKR holders govern the DAI system's parameters (collateral types, stability fees, liquidation ratios) ✅ (Wikipedia, Dai article; DAI live 18 Dec 2017; rebranded Sky Aug 2024).
- **Uniswap governance** — launched with the UNI token on **16 September 2020** ✅ (Uniswap blog): UNI holders govern the treasury, fee switch, and grants; verified initial parameters: 1% of supply to propose, 4% 'yes' quorum, 7-day voting, 2-day timelock (§9.1).
- **Aragon** — DAO-framework platform (create-your-own-DAO), launched 2016–2017 ⚠ (widely reported; not primary-verified this pass).
- **Moloch** — MolochDAO (2019) popularised "minimal viable governance" (ragequit, guilds) for Ethereum public-goods funding ⚠ (widely reported; not primary-verified this pass).
- **Nouns DAO** — launched **August 2021** ✅: one generative Noun NFT auctioned every 24 hours, 100% of proceeds to the treasury, each Noun = one vote — a "perpetual auction treasury" model (nouns.wtf; corroborating analyses; exact launch day 8 vs 17 August varies by source ⚠).
- **ENS DAO** — launched with the $ENS airdrop, claiming opened **8 November 2021** ✅ (ENS blog): ENS token holders govern the Ethereum Name Service (§9.2).

### 7.3 The Legal-Treatment Debate

The open legal question — *who is the DAO, and who is liable?* — is unresolved and jurisdiction-dependent ⚠:

- **Securities law:** the SEC's 2017 DAO report set the precedent that DAO tokens can be securities ✅; US enforcement actions through 2021–2026 against token issuers (the Gensler-era framing, §10.2) extended that logic — whether a *governance* token is a security remains the central uncertainty ⚠.
- **Legal personhood:** some jurisdictions created DAO-specific structures (e.g., Wyoming's DAO LLC, 2021 ⚠ reported); others treat DAOs as general partnerships by default — which would make members jointly liable, a result most DAO participants have not consented to in any meaningful sense ⚠.
- **The "decentralization defense":** if a protocol is genuinely autonomous with no controlling group, securities/AML enforcement has no natural defendant — which is exactly why critics (Diehl, §3.1) argue "decentralization" rhetoric is a regulatory-arbitrage strategy ⚠ (interpretation, both sides).
- **The bank's version of this debate** is narrower and practical: if Cymbal Bank interacts with a DAO (lends to it, custodies its tokens, validates its identity), who signs? The answer today is usually "the foundation or the core team," which is precisely the centralisation the DAO form claims to avoid — an operational, not philosophical, point for §11.

## 8. The Application Layers IV: Identity, Storage, Oracles

### 8.1 Web3 Identity: DIDs and Verifiable Credentials

**Decentralized Identifiers (DIDs) v1.0 became a W3C Recommendation on 19 July 2022** ✅ (w3.org/TR/did-core, verified this pass: status line "W3C Recommendation 19 July 2022"; editors Manu Sporny, Amy Guy, Markus Sabadello, Drummond Reed). The model, in brief: a **DID** is a URI whose controller holds the keys; a **DID document** carries the cryptographic verification methods and service endpoints; the controller can "prove control over it without requiring permission from any other party" — decoupled from central registries, identity providers, and certificate authorities ✅ (the spec's own abstract). The companion piece is the **Verifiable Credentials (VC) data model** — also a W3C standard ✅ (existence verified; exact version/date ⚠ not re-verified this pass): an issuer signs a claim (e.g., "this client passed KYC"), the holder stores it in a wallet, and any verifier checks the signature without calling the issuer — the "trust triangle" that makes credentials portable.

Why this matters to Web3 and to a bank: DID+VC is the standardised answer to "wallet-as-identity" (§4.2) — it separates *identifier* from *platform*, and it makes **KYC reuse** and **self-sovereign identity** architecturally possible. The bank's own identity architecture guide ([distributed_auth_guide.md](distributed_auth_guide.md)) covers authn/authz mechanics; the Web3 addition is the *verifiable-credential issuance* role — a bank is a natural VC issuer (KYC, employment, account-ownership credentials). Regulatory caveat ⚠: self-sovereign identity interacts awkwardly with FATF travel-rule and AML "beneficial ownership" requirements — the identifiers are pseudonymous by design, and regulators require real-world linkage somewhere in the chain.

### 8.2 Decentralised Storage: IPFS and Filecoin

The storage layer's origins are verified ✅ (Wikipedia + primary sources):

- **IPFS (InterPlanetary File System)** — created by **Juan Benet**, who founded **Protocol Labs in May 2014**; the IPFS whitepaper dates to 2014; an alpha launched **February 2015** ✅. The design: content-addressed, peer-to-peer, BitTorrent-like file sharing over a distributed hash table — a file is identified by its hash, any node can serve it, and gateways bridge to HTTP browsers ✅.
- **Filecoin** — the incentivised storage layer on top of IPFS: storage providers earn **FIL** tokens for provable storage (proof-of-replication, proof-of-spacetime) ✅. Verified record: whitepaper July 2014; **ICO August 2017** that raised **>US$200M within 30 minutes** ✅ (Wikipedia); **mainnet went live October 2020** ✅ (Filecoin blog, "Filecoin Mainnet is Live" — Liftoff Week 19–23 Oct 2020); the Filecoin Foundation donated 50,000 FIL (≈US$10M) to the Internet Archive in April 2021 ✅.
- **Pinning** is the practical detail: IPFS guarantees *addressing*, not *presence* — content stays available only while someone pins it (pinning services and Filecoin deals are the paid answers) ✅/unmarked (mechanics; the pinning-services market is ⚠ vendor landscape, fast-moving).
- The Web3 use cases: dApp frontends hosted on IPFS (§4.2), NFT metadata on IPFS/Arweave (§6.2), and archival datasets (the Internet Archive collaboration above ✅).
- The contrast cases, cross-ref'd lightly: [common_crawl_guide.md](common_crawl_guide.md) (the open web's largest *centralised* corpus — the "memory of the web" that decentralised storage wants to replicate) and [cloud_object_storage_lakehouse_guide.md](cloud_object_storage_lakehouse_guide.md) (the economics and durability guarantees of centralised object storage — the performance/durability bar decentralised storage is compared against). A bank's honest read: IPFS/Filecoin are real, verifiable, and useful for *public, tamper-evident, content-addressed* data (regulatory disclosures, audit archives, metadata); they are not a general-purpose replacement for the bank's private, governed, low-latency storage.

### 8.3 Oracles: The Bridge to Off-Chain Reality

The **oracle problem** (unmarked domain knowledge): a blockchain is deterministic and self-contained — it cannot see the price of a stock, the weather, or a fiat FX rate without someone *bringing the data in*. **Oracles** are the services that feed external data on-chain (price feeds for lending protocols, randomness, identity attestations, market data), and they are the classic centralisation point in a "trustless" stack: the chain verifies *everything except the oracle's word*.

- **Chainlink** is the dominant decentralised oracle network — founded 2017, mainnet 2019 ⚠ (widely reported; not primary-verified this pass). Its institutional relevance is verified via the repo's own record: a **Swift + Chainlink CCIP pilot (November 2024)** settled tokenized fund subscriptions across chains and traditional rails — cross-ref [tokenized_assets_guide.md](../banking/tokenized_assets_guide.md) (Project Guardian fund-pilot section) ✅ (inherited sibling verification).
- **Oracle risk** is a standing vulnerability class: price-feed manipulation (via flash loans) has broken lending protocols repeatedly — the mechanics and incident record live in [smart_contracts_guide.md](smart_contracts_guide.md) §8 and [defi_guide.md](defi_guide.md) §8; do not re-derive here.
- **The institutional analogue** is the market-data value chain: [market_data_consumption_guide.md](../banking/market_data_consumption_guide.md) maps how a bank consumes vendor feeds (pricing, entitlements, latency, governance). The Web3 lesson for a bank is that an oracle is just a market-data feed with a signing key — the bank already knows how to govern feeds; the new part is deciding *who is allowed to write them on-chain* and *what happens when the feed is wrong* (the answer today: liquidation cascades).


## 9. The Governance and the Tokenomics

Governance and tokenomics are the two mechanisms that make the "own" in read-write-own concrete: **governance** decides who controls the protocol, and **tokenomics** decides how the token's supply, distribution, and claims work. Both are Web3-native inventions — no Web2 platform has a token vote or an airdrop — and both are where the securities-law question (§10.2) bites hardest, because both turn "users" into "holders."

The two mechanisms interlock: governance determines *who decides* (the token holders), tokenomics determines *who holds* (the distribution and supply design), and the securities question asks whether the token that does both is an investment contract. A bank reading this section should keep the trio in one view — §9.1 (the decision machinery), §9.2 (how tokens get into holders' hands), §9.3 (the supply economics behind both).

### 9.1 Token-Based Governance

Token-based governance (unmarked domain characterisation; verified examples cited): a protocol's parameters — lending rates, collateral types, fee switches, treasury spending — are set not by a board but by token-holder vote, executed on-chain by the protocol's governance contracts. The standard mechanics:

- **Proposal → vote → timelock.** Anyone (or anyone holding a minimum stake) submits a proposal; token holders vote in a fixed window; a passed proposal sits in a **timelock** — typically two days — before execution, giving users time to exit. The timelock is governance's safety valve. Uniswap's verified parameters (§7.2): 1% of supply to propose, 4% 'yes' quorum, 7-day voting, 2-day timelock ✅.
- **Delegation.** Voting power can be delegated to representatives (Compound's model), which makes governance usable by non-technical holders but concentrates decisions in a few delegators ⚠ (reported pattern).
- **The whale problem.** Token-weighted voting is plutocratic by construction: an entity with 10% of supply holds 10% of votes. The community's design answers — conviction voting, quadratic voting, vote-escrow (veCRV, §5.1) — trade expressiveness for capture-resistance ⚠ (design-space characterisation, no canonical winner).
- **Governance attacks are a documented incident class.** Flash-loaned tokens have been borrowed to pass malicious proposals; the canonical case is the April 2022 **Beanstalk** exploit (≈US$182M drained via a governance proposal passed with flash-loaned votes ⚠ press figure; the incident record is sibling-verified in [defi_guide.md](defi_guide.md) §8.4 ✅ existence).
- **The legal hook.** Governance tokens are the instrument the securities question targets: if holders expect profits from the efforts of others (the Howey test), the token can be a security whatever its label — the SEC's DAO report (2017, ✅ §7.1) set the precedent, and the Gensler-era framing (§10.2) extended it. ⚠ contested legal interpretation; the Howey test itself is settled law (unmarked).

The governance-design arms race is worth one concrete example: **vote-escrowed tokens** (veTokens) — the design where holders lock tokens for up to four years to earn voting power and rewards. Curve introduced the model with **veCRV** (January 2020, ✅ §5.1), and the resulting **"Curve Wars"** — protocols competing to accumulate veCRV so they can direct Curve's emissions toward their own pools — became the defining governance battle of DeFi in 2021–2022, complete with a wrapping layer (Convex) that let users earn yield on locked positions ⚠ (widely documented; figures press-reported). The lesson for a bank: governance design is *competitive infrastructure* — the ve model answered the whale problem by making voting power expensive to hold, and created a new market (vote markets, bribery) in the process ⚠. If Cymbal Bank ever interacts with a ve-governed protocol, "who controls the votes" is a market-structure question, not a technicality.

The bank's read: on-chain governance is both a *capability* — transparent, auditable, programmable decision-making — and a *liability pattern*, because the "decentralization defense" (§7.3) cuts both ways. A bank dealing with a DAO should ask who controls the timelock, who holds the multisig, and who the big delegators are — the answer, in practice, is usually a small identifiable group, which is exactly who a regulator will come looking for.

The governance decision types, in summary:

| Decision | Who decides | Mechanism | Example (verified) |
|---|---|---|---|
| Protocol parameters (rates, collateral, fees) | Token holders | On-chain vote → timelock | Uniswap fee switch: 4% quorum, 7-day vote, 2-day timelock ✅ §9.1 |
| Treasury spending | Token holders | Vote → multisig execution | Nouns: 100% of auction proceeds to treasury, spent by vote ✅ §7.2 |
| Token supply (emissions, burns) | Token holders or protocol rule | Embedded in the tokenomics design (§9.3) | EIP-1559 burn is protocol-level, not voted ✅ §9.3 |
| Protocol upgrades / forks | Token holders + node operators | Off-chain signalling + client adoption | The 2016 fork decision was community-wide, not a token vote ✅ §7.1 |

Note the last row: token votes govern *the contracts*, not the *chain* — chain-level change (upgrades, forks) is decided by node operators and the community, which is why the "governance token controls everything" pitch is overstated ⚠ (interpretation; the token/chain distinction is domain-stable).

### 9.2 Airdrops: The Distribution Mechanism

An **airdrop** (unmarked domain characterisation) is the retroactive distribution of a protocol's tokens to historical users — the Web3-native answer to "how does a network with no company and no IPO distribute its value?" It bootstraps a holder base, rewards early users, and — critically for regulation — distributes tokens *without a sale* (⚠ whether "free" distribution still constitutes an offer is precisely the securities question §10.2 leaves open). The verified instances:

- **UNI (Uniswap), 16 September 2020** ✅ — 15% of the 1 billion-token supply airdropped retroactively to historical users (snapshot 1 September 2020); the airdrop that defined the format and lit the fuse under "DeFi Summer" (§5.1).
- **$ENS (Ethereum Name Service), 8 November 2021** ✅ — claiming opened together with the ENS DAO launch (§7.2); widely described as the largest airdrop of 2021 by recipient count ⚠ (press characterisation).
- **ARB (Arbitrum), announced 16 March 2023, live 23 March 2023** ✅ — the L2 era's landmark airdrop, run by the Arbitrum Foundation; the exact supply share to users was press-reported ⚠.
- **The farming economy** ⚠ — airdrops created a professional "airdrop-farming" industry (sybil accounts, wash activity) and a counter-industry of sybil-detection filters; both sides are fast-moving and unquantified (§3.3's honesty rule applies).

| Airdrop | Date | Verified parameters |
|---|---|---|
| **UNI** (Uniswap) | 16 Sep 2020 | 15% of 1B supply, retroactive to historical users; snapshot 1 Sep 2020 ✅ |
| **$ENS** (Ethereum Name Service) | 8 Nov 2021 | Claiming opened with the ENS DAO launch; recipient count widely called the largest of 2021 ⚠ |
| **ARB** (Arbitrum) | Announced 16 Mar 2023; live 23 Mar 2023 | Run by the Arbitrum Foundation; user share of the 10B supply press-reported ⚠ |

The mechanics worth understanding, one paragraph: an airdrop runs **snapshot → claim → (sometimes) vesting**. The snapshot fixes eligible addresses at a block height (UNI's snapshot was 1 September 2020, ✅ above); the claim window opens with a web page and gas costs; and some airdrops vest or gate claims on continued activity (retroactive versus active-user designs ⚠ design space). Two consequences follow: eligibility is *deterministic and auditable* once the snapshot is published — a compliance-friendly property — and the claim event itself is a taxable or reportable event in several jurisdictions ⚠ (§9.2).

The bank's read: airdrops matter to Cymbal Bank in three ways. (a) They are a **distribution-mechanics innovation** worth understanding — they are how a token network launches without a securities sale. (b) They are a **compliance event** when they touch clients: US tax treatment of airdropped tokens as income at receipt ⚠ (reported; not re-verified this pass), and EU marketing rules under MiCA ⚠. (c) They are mostly **irrelevant to the bank's own product line** — a licensed bank's tokenized products distribute by subscription and redemption, not by airdrop.

### 9.3 Tokenomics: Supply, Emissions, Vesting

**Tokenomics** (unmarked domain characterisation) is the token's economic design: the supply schedule, the emissions curve, the vesting locks, and the burn mechanisms that jointly determine who holds the token, how fast new supply arrives, and what claims the token carries. The design space:

- **Supply schedules.** Fixed-cap designs (bitcoin's 21M cap — unmarked domain knowledge) versus inflationary designs (ETH's post-Merge net issuance is low but nonzero ⚠) versus elastic designs (the failed UST model, §5.1). EIP-1559 — the London upgrade of August 2021 — introduced the **fee-burn** mechanism: part of every transaction fee is destroyed, giving ETH deflationary pressure at high usage ✅ (mechanism verified in [blockchain_technology_guide.md](blockchain_technology_guide.md); cross-ref for the mechanics).
- **Emissions.** Staking and yield-farming rewards are *marketing budgets denominated in tokens*: protocols pay new tokens to attract liquidity, and the "emissions treadmill" — each protocol must out-emit rivals to keep TVL — is the standing critique of the model ⚠ (interpretation, widely argued).
- **Vesting and unlocks.** Team and investor tokens vest over years, and scheduled **unlock events** are the market's known sell-pressure calendar ⚠ (market practice; the calendars live on token-unlock trackers, not re-verified here).
- **The token archetypes** (⚠ classification contested — the SEC disputes the boundaries): *utility* tokens (gas, access), *governance* tokens (§9.1), *security* tokens (tokenized securities — the institutional twin, cross-ref [tokenized_assets_guide.md](../banking/tokenized_assets_guide.md)), and *meme / zero-utility* tokens — the last being the cleanest demonstration that token price is not token value.
- **The LUNA/UST case** — the canonical tokenomics failure: an algorithmic stablecoin that paid ever-higher yields to grow, whose May 2022 death spiral destroyed tens of billions of market value ⚠ (press figure; the mechanics are sibling-verified in [defi_guide.md](defi_guide.md) §6.4 ✅).

The supply archetypes in one table:

| Design | Example | Behaviour | Status this pass |
|---|---|---|---|
| Fixed cap | Bitcoin (21M) | Scarcity by construction; issuance halves over time | Unmarked domain knowledge |
| Low/adaptive inflation | Ethereum post-Merge | Net issuance low and burn-linked (EIP-1559); supply can shrink at high usage | ⚠ net-issuance figures move; mechanism ✅ §9.3 |
| Collateral-minted | DAI | Minted against deposited collateral; supply tracks demand, not a schedule | ✅ §5.1 |
| Elastic / algorithmic | UST (Terra) | Supply adjusts to defend the peg — failed in May 2022 | ✅ mechanics; ⚠ magnitudes (§9.3) |

Note what the table does *not* contain: none of these designs gives the token holder a claim on cash flows — the fundamental difference between a token and a share, and the root of the securities question (§9.3, §10.2).

The bank's read: evaluate a token as you would equity with no cash flows — value rests on governance rights, utility claims, and secondary-market demand ⚠; the vesting schedule and emissions curve are the closest thing to a balance sheet; and the LUNA lesson is that yield-driven growth is a liability, not a feature.

A disciplined tokenomic review checklist, derived from this section (unmarked: standard practice for the repo's crypto-cluster guides): (1) **Supply** — fixed or inflationary? What is the emissions rate and who receives it (users, stakers, team, treasury)? (2) **Distribution** — what percentage went to the team and investors, and on what vesting schedule? An unlock calendar is a known sell-pressure event, not a surprise. (3) **Burn mechanics** — is any fee stream destroyed (EIP-1559-style) or redirected to the treasury? (4) **Claims** — what does the token actually entitle its holder to: governance votes, fee shares, utility, or nothing? (5) **The Howey overlay** — if holders' returns depend on the team's continued efforts, the token is a security candidate regardless of its stated purpose (§10.2) ⚠ (legal interpretation). A token that fails (1)–(3) or trips (5) is a market exposure, not an infrastructure investment.

## 10. The Regulation

The regulatory map is fully carried by two siblings — [mas_regulations_guidelines_guide.md](../banking/mas_regulations_guidelines_guide.md) (the Singapore regime) and [defi_guide.md](defi_guide.md) §9 (MiCA and the global patchwork). This section condenses both to the Web3-ecosystem view; it does not re-derive the mechanics.

### 10.1 The SG/MAS Posture, Condensed

MAS's posture in one line: **regulate the activity, not the technology** — the same Acts and notices govern token services wherever they sit, and the regulator has been deliberately selective about who gets licensed. The verified spine:

| Layer | The rule | Status |
|---|---|---|
| **DPT licensing** | Digital Payment Token services regulated under the **Payment Services Act 2019**; full regime applied from **October 2022** (SPI vs MPI tiers); ~30+ MPI licences with DPT scope by 2025 ⚠ (market count) | ✅ PS Act mechanics verified in the MAS guide |
| **Safeguarding** | Customer money and tokens held apart from the firm's assets; **no lending of customer DPTs** — the two hard rules | ✅ |
| **AML/CFT** | **PSN02** (revision effective 30 June 2025): CDD, travel-rule-aligned transfer information, sanctions screening, transaction monitoring | ✅ |
| **Technology risk** | **FSM-N30** (technology risk management) and **FSM-N31** (cyber hygiene), both effective **30 June 2025** — the DPT sector now carries bank-grade reliability/availability/recoverability and cyber obligations | ✅ |
| **Extraterritorial reach** | **FSMA 2022 Part 9** (phase 3, effective 30 June 2025): Singapore-incorporated DPT providers serving customers abroad need an FSMA licence | ✅ |
| **Stablecoins** | **SCS framework, August 2023** — the world's first comprehensive stablecoin regime: 100% reserve backing at all times, par redemption within five business days, monthly attestation and annual audits; an SCS issuer may not offer other services (no lending, staking, or DPT dealing) | ✅ |
| **Consumer protection** | Anti-scam circular (October 2024) for e-wallet MPIs: transaction alerts, kill-switches, mule-account detection | ✅ |
| **Tokenization** | **Project Guardian** — MAS-led pilots with global banks for tokenized funds, deposits, and settlement (cross-ref [tokenized_assets_guide.md](../banking/tokenized_assets_guide.md) for the pilot record) | ✅ sibling-verified |

The MAS Web3 timeline, condensed (all dates verified in the MAS guide's record this pass):

| Date | Event | Significance |
|---|---|---|
| 2019 | **Payment Services Act** passed | Forward-looking framework covering DPT services ✅ |
| 2020 | DPT services brought under PS Act licensing | First licensing wave; SPI/MPI tiers defined ✅ |
| Oct 2022 | **Full DPT regime applied** | Licensing, safeguarding, AML obligations in force ✅ |
| Aug 2023 | **SCS stablecoin framework** | World-first comprehensive stablecoin regime ✅ |
| 30 Jun 2025 | **FSM-N30 / FSM-N31** effective; **FSMA Part 9** (extraterritorial DPT) | DPT firms carry bank-grade tech-risk and cyber duties; MAS reaches Singapore-incorporated providers serving abroad ✅ |
| 2026 | Stablecoin issuance service legislated into the PS Act ⚠ (verify) | The SCS framework's statutory home |

The strategic read for Cymbal Bank: MAS has already drawn the shape of the industry — licensed gateways, full-spectrum supervision, bank-grade tech-risk duties on token firms, and a stablecoin regime that makes compliance a product feature. The regulator's famous selectivity (the Binance application was rejected ⚠ reported) makes the licence itself scarce — and an incumbent bank's licensed status is therefore a moat in this market, not a burden.

### 10.2 The International Regimes

- **EU — MiCA, Regulation (EU) 2023/1114** ✅ — the first comprehensive crypto-asset regime in a major jurisdiction: in force June 2023; the stablecoin rules (ART/EMT) applicable from 30 June 2024; the CASP (crypto-asset service provider) licensing regime from 30 December 2024. It replaces the member-state patchwork with an EU-wide passportable licence and is the benchmark every other regime is compared against (detail in [defi_guide.md](defi_guide.md) §9.1).
- **US — enforcement-led and still contested** ⚠ — the SEC's Gensler-era framing ("most crypto tokens are securities") ran through enforcement rather than legislation: the July 2023 *SEC v. Ripple* ruling (programmatic XRP sales not securities; institutional sales were ⚠ press-reported, not re-verified this pass), the spot BTC ETF approvals of January 2024 and spot ETH ETFs later in 2024 ⚠ (press-verified events), the regulatory aftermath of the FTX collapse, and FIT21 — a market-structure bill that passed the House in May 2024 with an unresolved Senate fate ⚠. The US remains the jurisdiction where the securities classification of governance tokens (§9.1) is the open question.
- **UK — FCA-led** ⚠ — the cryptoasset financial-promotion regime (from October 2023) and a stablecoin legislative agenda; reported, not re-verified this pass.
- **Japan — the first mover** ⚠ — exchange licensing under the Payment Services Act from 2017 made Japan the first major market with a formal licensing regime; widely reported, not re-verified this pass.
- **FATF — the global floor** ✅/unmarked — the FATF travel rule (Recommendation 16 and the VASP guidance) is the international AML standard that the national regimes implement (PSN02 in Singapore, MiCA's transfer-of-funds rule in the EU); domain-stable.
- **Hong Kong, the UAE, and the offshore centres** ⚠ — competing to be the crypto-friendly hub; fast-moving, not verified this pass.

The one-line map: three regimes coexist — the EU's comprehensive legislation, the US's enforcement-led contest, and Asia's licensed-gateway model (Singapore, Japan, Hong Kong) — and a bank's Web3 strategy must be built to the strictest regime it touches. For Cymbal Bank that is the MAS stack above, plus MiCA if it serves EU clients.

| Jurisdiction | Model | Status this pass | Bank consequence |
|---|---|---|---|
| **Singapore (MAS)** | Licensed gateways; activity-based | ✅ verified (§10.1) | The compliance bar; licence scarcity is a moat |
| **EU (MiCA)** | Comprehensive legislation; passportable | ✅ verified (dates) | Second regime to design for if EU clients are served |
| **US (SEC)** | Enforcement-led; classification contested | ⚠ press-reported (§10.2) | Watch item: securities classification of governance tokens |
| **UK (FCA)** | Promotions regime + stablecoin agenda | ⚠ reported | Watch item for the stablecoin product line |
| **Japan** | Early exchange licensing (2017) | ⚠ reported | The historical first-mover; DPT licensing precedent |
| **FATF (global)** | Travel-rule floor | ✅/unmarked | Implemented locally (PSN02); the data layer every product inherits |

**The stablecoin regulatory race** deserves its own note because it is the fastest-moving regulatory front a bank touches directly (cross-ref [reap_global_guide.md](../banking/reap_global_guide.md) and [tokenized_assets_guide.md](../banking/tokenized_assets_guide.md) for the product context): MAS's SCS framework (August 2023, ✅ §10.1) and the EU's MiCA ART/EMT rules (applicable 30 June 2024, ✅) both landed in a narrow window, and the US stablecoin legislation was still in flight this pass ⚠ (press-reported). The pattern is striking: the two regimes that actually regulate stablecoins comprehensively are the ones that make *reserve backing, redemption, and attestation* the product definition — which is a compliance standard incumbent banks already meet, and the reason the bank's stablecoin opportunity (§11.2) is scored "conditional," not "speculative."

The EU–US divergence is the structural feature to watch: MiCA gives the EU a *licence to build* — a compliant issuer knows its obligations in statute — while the US enforcement-led approach keeps the *classification question* open, which is why US-domiciled token projects have increasingly routed through the EU, Singapore, and the UAE ⚠ (market behaviour, press-reported). For a bank the asymmetry is favourable: the jurisdictions with clear rules are the ones where licensed institutions can operate, and the jurisdictions without them are the ones to avoid until they settle.

## 11. The Cymbal Bank Worked Example: A Web3 Strategy Assessment

### 11.1 The Design-Fiction Frame

This worked example follows the repo's convention (see the smart-contracts and bitunix guides' worked examples): **Cymbal Bank is the repository's fictional bank persona** — an illustrative Singapore-licensed bank used to make a guide's findings operational. The assessment below is **design fiction**: a plausible strategy review built on the verified facts of this guide, not a description of any real bank's decision. It is written the way Cymbal Bank's strategy team would write it, using the guide's verified anchors and the MAS overlay of §10.1.

**The mandate:** the board has asked for a Web3 strategy assessment — not "should we buy crypto," but "which parts of the Web3 stack, if any, should a licensed bank build on, and in what order?"

**The starting point** is §1.3's four threads — tokenization of real-world assets, programmable money, identity and verifiable credentials, and custody-plus-exchange revenue — each of which this guide has since verified as a real, buildable layer (Project Guardian pilots ✅ §10.1; stablecoins and the SCS regime ✅ §10.1; W3C-standard DIDs ✅ §8.1; the custody vendor model ✅ cross-ref [fireblocks_guide.md](../banking/fireblocks_guide.md)). The assessment below scores those four threads as product opportunities, overlays the MAS instruments each would trigger, and sequences them into a phased recommendation.

### 11.2 The Opportunities

Six candidate moves, each tied to verified anchors from this guide:

| # | Opportunity | What it is | Verified anchors |
|---|---|---|---|
| 1 | **Tokenized funds & RWAs** | Issue tokenized money-market funds and tokenized deposits on permissioned or partner rails | Project Guardian pilots ✅ ([tokenized_assets_guide.md](../banking/tokenized_assets_guide.md)); token standards ✅ ([smart_contracts_guide.md](smart_contracts_guide.md) §5) |
| 2 | **Institutional custody** | Licensed custody of client digital assets — a fee business run under the bank's licence | [fireblocks_guide.md](../banking/fireblocks_guide.md) (the vendor model); [tokenized_assets_guide.md](../banking/tokenized_assets_guide.md) §8 |
| 3 | **Stablecoin issuance** | Issue an MAS-compliant SCS stablecoin (SGD-pegged) or partner with an existing issuer | SCS framework ✅ §10.1; [reap_global_guide.md](../banking/reap_global_guide.md) (the payments analogue) |
| 4 | **Verifiable credentials** | Issue W3C-standard DID/VC credentials for KYC reuse and client self-sovereignty | DIDs W3C Recommendation 19 Jul 2022 ✅ §8.1; [distributed_auth_guide.md](distributed_auth_guide.md) |
| 5 | **Exchange / on-ramp partnership** | White-label or partner with a licensed exchange for SGD↔token on/off-ramps | [bitunix_guide.md](../banking/bitunix_guide.md) (the CEX anatomy); MAS licensing §10.1 |
| 6 | **Web3 client services** | Custody, VC, and on-ramp wrappers for a Web3-native client segment (funds, VCs, DAO treasuries) | §7.3 (the DAO-signing problem); §8.1 (VC issuance) |

The common denominator across all six: **every move is a licensed activity built on the Web3 stack, and none requires the bank to hold or trade unbacked tokens.** Opportunities 1, 2, and 4 are the "plumbing" plays — they reuse the wallet-and-contract stack as infrastructure, which is exactly how MAS is piloting it (Project Guardian, ✅ §10.1). Opportunities 3, 5, and 6 are the "product" plays — they face the market directly and carry the compliance burden first. The scoring in §11.5 follows that split.

One caveat on the whole table: the near-term revenue pool for each opportunity is small relative to the bank's core business — tokenized funds, custody, and VC issuance are collectively a rounding error on a licensed bank's balance sheet today ⚠ (no canonical market-size figure exists; §3.3's honesty rule applies). The strategy's rationale is *positioning and option value* — building the licensed plumbing while the market is young and the licence is scarce — not near-term revenue.

### 11.3 The Risks

The risk register, each entry tied to verified material in this guide:

- **Market risk** — the asset class lost hundreds of billions in 2022 (LUNA, FTX — §3.1, §5.2); any P&L exposed to unbacked token prices is a tail-risk position, not an investment. Mitigation: no proprietary trading of unbacked tokens; fee businesses only.
- **Technology risk** — smart-contract exploits are a standing incident class ([smart_contracts_guide.md](smart_contracts_guide.md) §8–§9); the oracle layer is a manipulable trust point (§8.3); NFT-style metadata pointers can be swapped underneath a token (§6.2). Mitigation: the bank custodies tokens, not contracts; vendors are audited (Fireblocks-class); no customer funds sit in unlicensed DeFi protocols.
- **Operational and UX risk** — the wallet layer is the attack surface: blind signing, drainer scams, key loss ("not your keys, not your coins" — §4.2). Mitigation: bank-grade recovery and policy controls (the custody vendors' model, cross-ref [fireblocks_guide.md](../banking/fireblocks_guide.md)); no self-custody UX for retail without guardrails.
- **Regulatory risk** — licensing scope (which activity needs which licence — §10.1), the securities classification of any token the bank touches (§9.1, §10.2), travel-rule data obligations (PSN02), and the extraterritorial reach of FSMA Part 9. Mitigation: the overlay in §11.4 built into each product from day one.
- **Reputational risk** — association with scams, rug pulls, and the 2022 collapses; the "web3 is a myth" critique (§3.1) is a standing narrative. Mitigation: the bank's brand touches only the *licensed* layer of the stack; the unregulated layer stays at arm's length.

The 2022 stress test, as this guide's evidence records it: LUNA/UST showed that a tokenomics design can fail catastrophically and take a whole ecosystem down with it (§9.3); FTX showed that a supposedly regulated intermediary can be a fraud at the centre of a trustless stack (§3.1, cross-ref [bitunix_guide.md](../banking/bitunix_guide.md) for the exchange-anatomy record); and the Beanstalk-style attacks showed that governance code can be weaponised (§9.1). None of the three failures invalidates the underlying stack — all three validate the bank's risk register above. The board question this section answers: *what happens to each opportunity in a 2022 repeat?* Phase 1's answer is "nothing" — fees and custody survive a bear market; Phase 2's answer is "moderately" — stablecoin demand rises in flight-to-quality but redemption risk is the SCS-tested control; Phase 3's answer is "watch the partner" — exchange counterparty risk is the FTX lesson.

The risk register in one table:

| Risk | Verified anchor | Bank control |
|---|---|---|
| Smart-contract exploit | Incident history, smart-contracts guide §8–§9 ✅ | Custody tokens, not contracts; vendor audits |
| Oracle manipulation | §8.3; defi guide §8 ✅ | No customer funds in DeFi; governed feeds |
| Metadata swap under a token | §6.2, Moxie's experiment ✅ | Treat the metadata layer as a first-class risk |
| Key loss / phishing / drainers | §4.2 (blind signing) ⚠ | Bank-grade recovery and policy controls |
| Exchange counterparty | FTX 2022 ✅ §3.1 | Partner governance; no balance-sheet exposure |
| Token securities classification | §9.1, §10.2 ⚠ legal | Strictest-regime rule; no unbacked-token books |

### 11.4 The Regulatory Overlay

Each opportunity mapped to its MAS instrument (condensed from §10.1; the MAS guide carries the mechanics):

| Opportunity | Primary instruments | Design consequences |
|---|---|---|
| Tokenized funds / deposits | Existing bank licence + FSMA tech-risk notices (FSM-N05) | A tokenized deposit is a deposit under the Banking Act; technology risk sits under the same TRM obligations as core banking |
| Custody | PS Act DPT licence (or MPI with DPT scope) | Safeguarding: client tokens segregated; **no lending of client DPTs** — the hard rule ✅ |
| Stablecoin issuance | SCS framework (Aug 2023) | 100% reserve backing, 5-business-day par redemption, monthly attestation, no other services for the issuer ✅ |
| Verifiable credentials | No DPT licence needed; PDPA plus MAS outsourcing/TRM rules apply | Credential issuance is a data-governance product, not a payment service |
| Exchange partnership | The partner holds the PS Act licence; the bank governs the partnership | Third-party risk management under the MAS outsourcing guidelines |
| Web3 client services | Composite of the above, per product | Each wrapper inherits the strictest instrument in its stack |

The one regulatory insight: MAS has made the boundaries *knowable* — every Web3 product the bank might build maps onto an existing instrument. The unregulated residue — holding unbacked tokens, interacting with unlicensed DeFi — is precisely what the bank does not do.

The licensing decision tree, in one paragraph: **does the product hold or move client tokens?** If yes, it is a DPT service under the PS Act — custody, exchange, and transfer services each need their licence scope (or MPI with DPT scope), with safeguarding and the no-lending rule as the hard controls ✅. **Does the product issue a stablecoin?** Then the SCS framework applies — 100% reserves, five-business-day par redemption, monthly attestation, no other services for the issuer ✅. **Does the product tokenize something the bank already does?** A tokenized deposit or fund unit is governed by the existing banking/securities licence, with the tokenization itself treated as technology risk under FSM-N05 ✅. **Does the product issue credentials?** DID/VC issuance is data governance, not a payment service — PDPA and the outsourcing/TRM rules apply, no DPT licence needed ✅. Every opportunity in §11.2 resolves through exactly one of these four branches — which is the strongest evidence that the Web3 stack, for a licensed bank, is an infrastructure question rather than a frontier question.

### 11.5 The Scored Recommendation

Scorecard (illustrative, on this guide's evidence — H/M/L ratings, not a numerical model):

| Opportunity | Strategic fit | Risk-adjusted return | Regulatory readiness | Verdict |
|---|---|---|---|---|
| 1. Tokenized funds & RWAs | H | H | H | **Phase 1 — do** |
| 2. Institutional custody | H | H | H | **Phase 1 — do** |
| 4. Verifiable credentials | M | M | H | **Phase 1 — pilot** |
| 3. Stablecoin issuance | M | M | M | **Phase 2 — conditional** |
| 6. Web3 client services | M | M | M | **Phase 2 — wrap Phase 1** |
| 5. Exchange / on-ramp | M | L–M | M | **Phase 3 — partner, don't build** |

**The recommendation — adopt the plumbing, price the market, ignore the hype:**

- **Phase 1 (first 18 months):** join the tokenization lane MAS is actively piloting (Project Guardian-style tokenized funds — cross-ref the tokenized-assets guide's pilot record); stand up licensed custody on a Fireblocks-class vendor; pilot DID/VC credential issuance for KYC reuse. These three share one infrastructure bet — the wallet-and-contract stack as plumbing — and none requires the bank to hold unbacked tokens.
- **Phase 2 (18–36 months):** stablecoin issuance *if* the SCS compliance burden is met and client demand is proven (the reap-guide payments analogue shows the use case); wrap the Phase 1 products into a Web3 client-services bundle for funds and DAO treasuries — remembering the §7.3 question: *who signs for the DAO?* (the answer today is usually the foundation or core team, which the bank should treat as a corporate client, not a protocol).
- **Phase 3 (36 months+):** exchange and on-ramp via partnership rather than build — the licence is scarce, the business is fee-thin, and the FTX lesson (§3.1) is that exchange counterparty risk belongs on the exchange's own balance sheet, not the bank's.
- **Standing guardrails:** no proprietary trading of unbacked tokens; no customer funds in unlicensed DeFi protocols; no retail leverage products; the strictest-regime rule (MAS, plus MiCA if EU clients) applied to every product; and a quarterly re-read of the claims audit (§12), because the fastest-moving items on this list are the ones marked ⚠.

The one-line strategy: **the bank's Web3 strategy is not a crypto strategy** — it is a licensed-infrastructure strategy that uses the Web3 stack where it earns its keep and stays out of the token market where it does not.

**What would change the recommendation** (a design-fiction stress on the rating inputs): a confirmed institutional-demand signal for SGD stablecoins would pull opportunity 3 forward from Phase 2; a second major exchange collapse would push opportunity 5 from "partner" to "don't" and strengthen the case for building the on-ramp inside the bank's own licence; a definitive US Supreme Court ruling on token classification (§10.2) would either de-risk the whole product line (tokens as non-securities) or cap it (tokens as securities → the bank's licensed products gain relative advantage); and a MAS Project Guardian productionisation would accelerate Phase 1 by removing the pilot-status uncertainty. Each trigger maps to a verifiable event on this guide's radar — the honest way to run a strategy review on a fast-moving ecosystem.

## 12. The Claims Audit

This section is the repo's honesty convention applied to this guide: every ✅ claim above is either verified this pass at the named source or inherited from a sibling guide's verified record; every ⚠ is reported, approximate, single-sourced, or fast-moving; ❌ marks refuted or not-found. Primary sources are listed in §14.

### 12.1 The Verified-Facts Table

| Claim | Verification | Status |
|---|---|---|
| Wood coined "Web3" in the April 2014 essay "ĐApps: What Web 3.0 Looks Like" (17 April 2014) | Primary source gavwood.com/dappsweb3.html read this pass (§2.1) | ✅ |
| Wood's 2018 restatement ("Why We Need Web 3.0": "an executable Magna Carta"; "over four years ago that I coined the term") | Mirror of the original essay read this pass; original URL 404s (§2.2) | ✅ |
| Web 1.0 ~1991–2004 static; Web 2.0 "the web as platform" from ~2004 | Wikipedia "Web3" article account (§2.3) | ✅ |
| Semantic-Web / "Web 3.0" terminological collision | Wikipedia "Web3" article explicitly flags it (§2.5) | ✅ |
| Moxie Marlinspike, "My first impressions of web3" (7 Jan 2022) — Infura/Alchemy dependence, NFT-metadata critique, "web2x2" | Primary source moxie.org read this pass (§3.1) | ✅ |
| Stephen Diehl, "Web3 is bullshit" (Nov 2021) exists | Corroborated by multiple independent references; primary page unreachable this pass (§3.1, §12.2) | ✅ existence / ⚠ claims |
| Dorsey's December 2021 dismissal of Web3 | Bloomberg, 21 Dec 2021; tweet text quoted from reporting (§3.1) | ✅ event / ⚠ quote |
| Buterin's response to Moxie (Jan 2022) | Fortune, 10 Jan 2022 (§3.1) | ✅ |
| DAI on Ethereum mainnet 18 Dec 2017; MakerDAO 2014, rebranded Sky Aug 2024 | Wikipedia + sibling record (§5.1) | ✅ |
| Uniswap created 2 Nov 2018; Compound Sep 2018; Aave Jan 2020; Curve Jan 2020; YFI Jul 2020 | Wikipedia / defi-guide verified record (§5.1) | ✅ |
| DefiLlama as the reference TVL aggregator | defillama.com read this pass (§5.2) | ✅ |
| ERC-721 created 24 Jan 2018, status Final; authors Entriken, Shirley, Evans, Sachs | eips.ethereum.org read this pass (§6.1) | ✅ |
| CryptoKitties launched 28 Nov 2017 | Wikipedia (§6.1) | ✅ |
| The DAO: launched 30 Apr 2016; hacked 17 Jun 2016 (~3.6M ETH); fork 20 Jul 2016 → Ethereum Classic | Wikipedia + sibling incident record (§7.1) | ✅ |
| SEC DAO report (July 2017): DAO tokens are securities | SEC Release 2017-131, cited via Wikipedia (§7.1) | ✅ |
| Uniswap governance parameters (1% propose / 4% quorum / 7-day vote / 2-day timelock) | Uniswap blog (Sep 2020) (§7.2, §9.1) | ✅ |
| Nouns DAO launched Aug 2021; one Noun auctioned daily; 100% of proceeds to treasury | nouns.wtf + corroborating analyses; exact day varies by source (§7.2) | ✅/⚠ |
| ENS airdrop and DAO — claiming opened 8 Nov 2021 | ENS blog (§7.2, §9.2) | ✅ |
| UNI airdrop 16 Sep 2020 — 15% of 1B supply retroactive (snapshot 1 Sep 2020) | Uniswap blog (§9.2) | ✅ |
| ARB airdrop announced 16 Mar 2023, live 23 Mar 2023 | Arbitrum Foundation announcements (§9.2) | ✅ |
| W3C DID Recommendation, 19 July 2022 | w3.org/TR/did-core read this pass (§8.1) | ✅ |
| IPFS: Benet/Protocol Labs May 2014, paper 2014, alpha Feb 2015; Filecoin ICO Aug 2017 >US$200M; mainnet Oct 2020 | Wikipedia + Filecoin blog (§8.2) | ✅ |
| Swift + Chainlink CCIP pilot (Nov 2024) settled tokenized funds across rails | Inherited from the tokenized-assets guide's verified record (§8.3) | ✅ |
| MiCA dates (Reg 2023/1114: in force Jun 2023; ART/EMT 30 Jun 2024; CASP 30 Dec 2024) | EUR-Lex, via the key-facts table and defi guide §9.1 | ✅ |
| MAS: full DPT regime from Oct 2022; PSN02/FSM-N30/FSM-N31 30 Jun 2025; FSMA Part 9 30 Jun 2025; SCS framework Aug 2023; anti-scam circular Oct 2024 | MAS guide's verified record this pass (§10.1) | ✅ |
| EIP-1559 fee burn (London upgrade, Aug 2021) | Blockchain-guide mechanism record (§9.3) | ✅ |
| Beanstalk governance attack (Apr 2022), flash-loaned votes | Defi-guide §8.4 incident record — existence ✅, magnitude ⚠ (§9.1) | ✅/⚠ |
| The DAO crowdsale scale: >US$150M of ETH by 21 May 2016 (~14% of issued ETH; >11,000 investors) | Wikipedia "The DAO" account (§7.1) | ✅ |
| The 2016 fork split: original chain continued as Ethereum Classic | Wikipedia + sibling record (§7.1) | ✅ |
| Moxie's NFT was removed by OpenSea without explanation | Reported in his verified essay (§6.2) | ✅ |
| DAI's collateralised design vs USDT/USDC centralised reserves vs failed UST algorithmic design | Defi-guide §6 verified record (§5.1) | ✅ |
| The "DeFi" label coined in an August 2018 Telegram chat | Sibling defi-guide verified record (§5.1) | ✅ |
| The Filecoin Foundation's 50,000 FIL donation to the Internet Archive (April 2021) | Filecoin blog record (§8.2) | ✅ |
| Wood's four-component stack (publication, messaging, consensus engine, UI) | Primary essay read this pass (§2.1) | ✅ |

### 12.2 What Could Not Be Verified

The items this guide flags rather than certifies — reported, single-sourced, fast-moving, or unreachable this pass:

- **Wood's 2018 essay at its original URL** — gavwood.com URL 404s this pass; verified via a mirror instead (§2.2). ⚠
- **The fuller title variant** "DApps: What Web 3.0 Looks Like and the New Stack" — appears in secondary literature; the primary page carries the shorter title (§2.1). ⚠
- **"Read Write Own"** — the standalone book title attributed to the a16z school was not re-verified this pass (§2.3). ⚠
- **The "read → write → own" slogan's exact provenance** — advocacy shorthand; no single citable primary source (§2.3). ⚠
- **Stephen Diehl's essay text** — primary page unreachable; the Wayback Machine holds no capture; claims attributed at one remove (§3.1). ⚠
- **David Gerard's book publisher/edition details** — blog verified; book details single-sourced (§3.1). ⚠
- **Dorsey's tweet text verbatim** — quoted from widely reproduced reporting, not the archived tweet (§3.1). ⚠
- **DeFi TVL peak (~US$180B, late 2021) and the trajectory figures** — press estimates; treat as order of magnitude; cite DefiLlama live (§5.2). ⚠
- **Beeple / Christie's US$69.3M (March 2021) and all NFT volume figures** — widely reported press facts, not re-verified this pass (§6.1). ⚠
- **Lido's exact 2020 launch date** — widely reported; primary launch post not re-read (§5.1). ⚠
- **Aragon and Moloch launch dates; Wyoming DAO LLC details** — reported, not primary-verified (§7.2, §7.3). ⚠
- **Nouns DAO's exact launch day** (8 vs 17 August 2021) — sources vary (§7.2). ⚠
- **Chainlink's founding/mainnet dates** — widely reported, not primary-verified (§8.3). ⚠
- **The ~30+ MPI DPT-licence count and the Binance application rejection** — market counts, reported (§10.1). ⚠
- **US items (Ripple ruling details, spot BTC/ETH ETF approvals, FIT21 status); UK FCA regime; Japan's 2017 licensing** — press-reported, not re-verified this pass (§10.2). ⚠
- **US tax treatment of airdropped tokens as income at receipt** — reported; the guidance was not re-read this pass (§9.2). ⚠
- **Hack magnitudes** — LUNA/UST (tens of billions), Beanstalk (≈US$182M), Euler (≈US$197M): press/post-mortem figures; incident *existence* is sibling-verified, magnitudes are ⚠ (§9.1, §9.3). ⚠
- **"Web3 is 1% of the web" and all adoption/user counts** — no canonical measurement exists (§3.3). ⚠
- **The audit itself** — nothing in this guide was refuted this pass: every claim checked was verified or flagged; ❌ entries are reserved for claims that check out false, and this pass found none. ✅

## 13. The Glossary

| Term | Definition |
|---|---|
| **Airdrop** | Retroactive distribution of a protocol's tokens to historical users — bootstraps holders and rewards early users without a sale (§9.2) |
| **AMM** | Automated Market Maker — a DEX design that prices assets algorithmically against liquidity pools (x·y=k) instead of an order book (§5.1; defi guide §4) |
| **Bridge** | A mechanism moving tokens or messages between chains; a standing hack target (blockchain guide) |
| **CEX / DEX** | Centralised exchange (a licensed company running an order book) vs decentralised exchange (on-chain, permissionless) (§4.1; bitunix guide / defi guide) |
| **Composability** | The property that DeFi contracts call each other ("money legos"), enabling complex positions and systemic risk (§5.1) |
| **DAO** | Decentralized Autonomous Organization — an organisation whose rules are encoded in smart contracts and whose decisions are token-holder votes (§7) |
| **dApp / ĐApp** | Decentralised application — a frontend whose state and rules live on-chain (§2.1, §4.2) |
| **DID** | Decentralized Identifier — a W3C-standard URI whose controller holds the keys, decoupled from central registries (§8.1) |
| **DPT** | Digital Payment Token — MAS's regulatory term for crypto assets under the PS Act (§10.1) |
| **EIP / ERC** | Ethereum Improvement Proposal / Ethereum Request for Comments — the standards process and the token standards it produces (smart contracts guide §4–§5) |
| **Emissions** | New token supply issued over time (staking/yield rewards) — the token market's inflation schedule (§9.3) |
| **EVM** | Ethereum Virtual Machine — the deterministic execution environment for smart contracts (blockchain guide) |
| **Filecoin** | The incentivised storage network on IPFS — providers earn FIL for provable storage (§8.2) |
| **Flash loan** | An uncollateralised loan repaid within one transaction; the amplifier of many exploits (smart contracts guide §8.3) |
| **Gas** | The fee paid to execute a transaction, proportional to computation (blockchain guide) |
| **IPFS** | InterPlanetary File System — content-addressed peer-to-peer storage; addressing by hash, presence requires pinning (§8.2) |
| **L1 / L2** | Layer 1 (a base chain) vs Layer 2 (a rollup or sidechain scaling on top) (blockchain guide) |
| **Liquid staking** | Staking whose receipt token (e.g. stETH) stays usable in DeFi (§5.1) |
| **MPI / SPI** | Major Payment Institution / Standard Payment Institution — the two PS Act licensing tiers (§10.1) |
| **NFT** | Non-Fungible Token (ERC-721) — a token certifying ownership of a registry entry for a distinguishable asset (§6) |
| **Oracle** | A service feeding external data on-chain; the stack's classic centralisation point (§8.3) |
| **Read-Write-Own** | The Web3 slogan mapping Web 1.0 (read) → Web 2.0 (read-write) → Web3 (ownership) (§2.3) |
| **Rollup** | An L2 that batches transactions onto L1, inheriting its security (blockchain guide) |
| **RWA** | Real-World Asset — a physical or traditional financial asset tokenized on-chain (tokenized assets guide) |
| **SCS** | MAS's stablecoin-framework label — the August 2023 regime for SGD- and G10-pegged stablecoins (§10.1) |
| **Smart contract** | A deterministic program executed by the chain, encoding rules no party can change unilaterally (smart contracts guide) |
| **Stablecoin** | A token designed to hold a stable value against a peg — fiat-backed, crypto-backed, or algorithmic (the failed third kind) (§5.1; defi guide §6) |
| **Timelock** | A delay between a passed governance proposal and its execution, letting users exit (§9.1) |
| **Tokenomics** | The token's economic design — supply, emissions, vesting, burns (§9.3) |
| **Travel rule** | The AML obligation to share sender/receiver information across transfers (FATF Rec. 16; PSN02 in Singapore) (§10.1–§10.2) |
| **TVL** | Total Value Locked — the dollar value of assets deposited in DeFi; an order-of-magnitude metric (§5.2) |
| **VC** | Verifiable Credential — a signed claim a holder presents to any verifier without calling the issuer (§8.1) |
| **Vesting** | Scheduled lockup of team/investor tokens; unlock events are the sell-pressure calendar (§9.3) |
| **Wallet** | The key-holding client that *is* the account in Web3 — self-custody vs institutional custody is the design fork (§4.1–§4.2) |
| **Address** | The pseudonymous account identifier derived from a key pair — the "account number" of Web3 (§4.2) |
| **Consensus** | The mechanism by which distributed nodes agree on chain state (PoW, PoS — blockchain guide) |
| **Delegation** | Transferring voting power to a representative — makes governance usable, concentrates decisions (§9.1) |
| **Liquidity pool** | The token pair locked in an AMM contract that provides price and liquidity (defi guide §4) |
| **Liquidation** | The forced sale of collateral when a loan's collateral ratio breaches the threshold (defi guide §5) |
| **StableSwap** | Curve's pegged-asset AMM formula — low-slippage swaps for stable pairs (§5.1) |
| **Sybil attack** | Creating many fake identities to farm airdrops or votes; the target of sybil-detection filters (§9.2) |
| **Treasury** | The DAO's on-chain asset pool, spent by governance vote (§7.2, §9.1) |
| **Finality** | The guarantee that a confirmed transaction will not be reversed (blockchain guide) |
| **KYC / AML** | Know-Your-Customer / Anti-Money-Laundering — the compliance layer the licensed gateways apply at the fiat border (§10) |
| **Merkle tree** | The hash-tree data structure that lets a block commit to many transactions compactly and prove membership (blockchain guide) |
| **Multisig** | A wallet requiring M-of-N signatures to move funds — the standard DAO/treasury control (§7.3, §9.1) |
| **On/off-ramp** | The licensed fiat↔token gateway at the edge of the decentralised stack (§4.2) |
| **PoW / PoS** | Proof-of-Work / Proof-of-Stake — the two consensus families securing chains (blockchain guide) |
| **RPC provider** | The API service (Infura, Alchemy) through which wallets and dApps actually reach the chain — the centralisation point Moxie documented (§3.1, §4.2) |
| **Security token** | A token that is itself a security — tokenized equity, debt, or fund units (tokenized assets guide) |
| **Staking** | Locking tokens as collateral for consensus/validation in exchange for rewards (blockchain guide) |
| **Web3 / Web 3.0** | The blockchain sense (this guide) vs the older Semantic-Web sense — the terminological collision documented in §2.5 |
| **Web2 / Web3** | The platform web (read-write, centralised) vs the ownership web (read-write-own, token-based) (§2) |

## 14. Cross-References and the Closing Summary

**Technology siblings (same folder):** [blockchain_technology_guide.md](blockchain_technology_guide.md) — the ledger layer: consensus, blocks, the EVM, L2s, bridges, EIP-1559; mapped in §4.1, cited in §9.3. · [smart_contracts_guide.md](smart_contracts_guide.md) — the contract/standards layer: ERC-20/721/1155/4626, the EIP lifecycle, proxies, SWC classes, the incident history; cross-ref'd in §6 and §9. · [defi_guide.md](defi_guide.md) — the DeFi application layer: AMMs, lending, stablecoins, composability, risks, MiCA; cross-ref'd in §5, §9.1, §9.3, §10.2. · [distributed_auth_guide.md](distributed_auth_guide.md) — identity architecture; §8.1 plugs DID/VC into it. · [common_crawl_guide.md](common_crawl_guide.md) and [cloud_object_storage_lakehouse_guide.md](cloud_object_storage_lakehouse_guide.md) — the centralised-web contrast cases for §8.2's storage economics.

**Banking siblings (prefix `../banking/`):** [tokenized_assets_guide.md](../banking/tokenized_assets_guide.md) — the institutional twin: tokenized RWAs, tokenized deposits, custody vendors, Project Guardian; cross-ref'd in §1.3, §8.3, §10.1, §11. · [fireblocks_guide.md](../banking/fireblocks_guide.md) — institutional custody mechanics (MPC, policy engines); §4.1 and §11. · [bitunix_guide.md](../banking/bitunix_guide.md) — the exchange layer (CEX anatomy, proof-of-reserves, the licensing patchwork); §4.1 and §11. · [reap_global_guide.md](../banking/reap_global_guide.md) — stablecoin-collateralised payments; §1.3 and §11. · [payment_rails_guide.md](../banking/payment_rails_guide.md) — the fiat rails underneath the crypto rails. · [mas_regulations_guidelines_guide.md](../banking/mas_regulations_guidelines_guide.md) — the SG regime this guide condenses in §10.1. · [market_data_consumption_guide.md](../banking/market_data_consumption_guide.md) — the oracle problem's institutional analogue (§8.3).

**Where this guide sits in the repo's crypto cluster:** this guide is the umbrella — the ecosystem view that ties the cluster together. The ledger mechanics live in [blockchain_technology_guide.md](blockchain_technology_guide.md); the contract and standards layer in [smart_contracts_guide.md](smart_contracts_guide.md); the DeFi application layer in [defi_guide.md](defi_guide.md); and the institutional/banking side — exchanges, custody, stablecoin payments, tokenized assets, and the MAS regime — in the banking-cluster guides named above. The division of labour is the one stated in §1.4: mechanics in the siblings, ecosystem map, debate, regulation overview, and bank strategy here. A reader coming from any sibling should be able to find, in this guide, where their topic sits in the stack and which sibling carries the detail.

**Primary sources used this pass:** gavwood.com/dappsweb3.html (Wood's 2014 essay, read this pass); the 2018 "Why We Need Web 3.0" text via its attributed mirror (original URL 404); moxie.org, "My first impressions of web3" (7 January 2022, read this pass); eips.ethereum.org (ERC-721, read this pass); w3.org/TR/did-core (W3C Recommendation, 19 July 2022, read this pass); defillama.com (read this pass); davidgerard.co.uk/blockchain (read this pass); Wikipedia ("Web3", "Dai (cryptocurrency)", "CryptoKitties", "The DAO", "Filecoin", "IPFS" — cited per claim); the sibling guides' verified records (defi guide, smart-contracts guide, tokenized-assets guide, MAS guide, blockchain guide — inherited facts marked ✅ with the sibling named); EUR-Lex (MiCA, Regulation (EU) 2023/1114, via the key-facts table and defi guide §9.1).

**Closing summary.** Web3 is the proposal that the web's next era is one of ownership: the concept Gavin Wood coined in April 2014 — a post-Snowden web of consensus ledgers, encrypted messaging, and pseudonymous identities — has since grown into a full stack of chains, contracts, wallets, exchanges, and stablecoins, overlaid with application layers — DeFi, NFTs, DAOs, identity, storage, oracles — that are real, verifiable, and unevenly decentralised. The debate between advocates and critics is best read as a disagreement about time horizons: critics photograph the present stack, advocates film the roadmap, and the regulators — led in Singapore by MAS — have settled the practical question by licensing the gateways, supervising the activity, and letting the technology compete on compliance. For Cymbal Bank, the strategy follows from the evidence: adopt the plumbing where it earns its keep — tokenization, custody, verifiable credentials — price the market honestly, and ignore the hype either way. Throughout, the integrity convention has kept every claim auditable — ✅ where verified at the source, ⚠ where reported or fast-moving, ❌ where refuted, and nothing refuted this pass. That is the honest map of the Web3 ecosystem: the umbrella over this repository's crypto cluster, and the reason the whole movement is best understood as a single bet — that the next era of the internet will be the owned web.
