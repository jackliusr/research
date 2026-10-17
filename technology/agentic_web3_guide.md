# Agentic Web3: The Counterparty With No Legal Person Behind It

*Two disciplines have grown up in this repository without ever meeting: the blockchain and web3 cluster, which owns keys, custody, settlement and irreversibility, and the AI-agent cluster, which owns reasoning, tools, autonomy and control. This repository contains zero mentions of the word "agent" across its entire web3 guide, and zero mentions of chain, blockchain, smart contract, web3 or crypto across its two agent-engineering guides — a measured seam, not a rhetorical one. This guide is the first thing here to stand in the middle of it. It does not re-derive the chain layers, the tokenomics, the regulation, DeFi, stablecoins, the autonomous-agent taxonomy or the agentic-engineering lifecycle; every one of those is owned by a named sibling it cross-references by section. What it does own is the **intersection**: what changes when an autonomous, non-deterministic actor is given a wallet on a deterministic, irreversible settlement layer. Its thesis is one sentence: **an agent with a wallet is not a user with an account; it is a counterparty with no legal person behind it.** Everything distinctive follows from that — custody without a customer, authorisation without a human signature, action that settles irreversibly, and a counterparty that cannot be sued, onboarded, or asked to explain itself.*

**Jack Liu Shurui, Solution Architect**

> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)  
> **Last Updated:** October 2026  
> **Companion guides (technology/, same folder):** [web3_technologies_guide.md](web3_technologies_guide.md) — the web3 umbrella: Gavin Wood's 2014 coinage (§2), the advocates-vs-critics debate (§3), the technology layers (§4), DeFi (§5), NFTs (§6), DAOs (§7), identity/storage/oracles (§8), governance and tokenomics (§9), the regulation (§10), a Cymbal web3-strategy assessment (§11) and the claims audit (§12). This guide **does not restate the layers, tokenomics or regulation**; it cross-references them and owns only what Wood's essay never contemplated — an autonomous actor as the party to a transaction. [blockchain_technology_guide.md](blockchain_technology_guide.md) — the chain itself: what a blockchain is (§1), architecture (§2), consensus (§4), smart contracts (§5), L2 scaling (§7), DeFi (§9), enterprise/banking (§10), trends/challenges/regulation (§11); cross-referenced, never re-derived. [defi_guide.md](defi_guide.md) — DeFi itself: the stack (§2), the AMM (§4), lending (§5), stablecoins (§6), risks (§8), regulation (§9), the institutional angle (§10). §8 of *this* guide asks the agent question *against* those mechanics; it does not re-explain them. [smart_contracts_guide.md](smart_contracts_guide.md) — the ERC process and token standards in depth, cross-referenced where this guide names a standard. [technology/w3c_dids_guide.md](w3c_dids_guide.md) — off-chain decentralised identity: DID Core (§2), the DID document (§3), methods (§6), resolution (§8). [verifiable_credentials_guide.md](verifiable_credentials_guide.md) — the credential half: the data model (§2), formats (§4), revocation (§6), the KYC use case (§8). [non_human_identity_guide.md](non_human_identity_guide.md) — the authority/register discipline for non-human identities: the ownership model (§7), privilege (§7.3), the agent as NHI (§8). §4 of this guide cross-references it for *who owns the key* rather than restating it.  
> **Companion guides (technology/ai_llm/, prefix `ai_llm/`):** [autonomous_agents_guide.md](ai_llm/autonomous_agents_guide.md) — the autonomous-agent taxonomy, the autonomy spectrum, architectures, control and safety, evaluation and enterprise/banking adoption; its **§10.3 "The Agent Economy"** already frames agent-to-agent commerce and names x402, A2A and MCP as emerging rails, and carries the repository's own line — *"The 'agent economy' as a term is analyst/VC-driven; treat size estimates with skepticism"* — which §6 and §12 of this guide **inherit rather than repeat**. [agentic_engineering_guide.md](ai_llm/agentic_engineering_guide.md) — the engineering discipline: the lifecycle phases (§4–§11), the open problems (§15), the Cymbal worked example (§16) and the claims audit (§17); cross-referenced, not restated.  
> **Companion guides (banking/, prefix `../banking/`):** [tokenized_assets_guide.md](../banking/tokenized_assets_guide.md) — tokenisation in banking: stablecoins (§2), tokenized deposits (§3), the regulatory landscape (§9), banking and treasury (§10). §6 of this guide uses it **for the stablecoin-rail question only** and does not re-derive stablecoins. [enterprise_risk_management_guide.md](../banking/enterprise_risk_management_guide.md) and [ai_risk_register_guide.md](../banking/ai_risk_register_guide.md) — the risk and register frames §9 applies to an agent counterparty.  
> **Method note:** this pass had live web access in **October 2026** and retrieved the following **at source by direct page extraction**: the ethereum.org EIP repository pages for **EIP-712, ERC-1271, ERC-4337, EIP-7702, ERC-7579, ERC-20, ERC-721 and ERC-8004** (each page's own header row gave the status and the creation date quoted in §15); **x402.org** and **docs.x402.org** (the *batch-settlement* scheme page and the *facilitator* core-concept page); the **x402.org Linux Foundation launch announcement dated 14 July 2026**; **a2a-protocol.org/latest/specification/** (latest released version 1.0.0); and **modelcontextprotocol.io/specification/** (authoritative schema dated 2026-07-28). `web_search` was **intermittently empty on this host**; where a search returned nothing usable that is recorded as a **tool limitation, not as evidence of absence** — §16.1. The x402 project site displays live counters (transactions, volume, buyers, sellers); **this guide deliberately prints none of them**, and §12 and §15 record that omission. **Every illustrative figure is labelled illustrative and attached to the fictional Cymbal Bank scenario only; Cymbal Bank is the sole bank persona and the sole institution in any worked example.** No token, protocol or chain is ranked, valued or recommended. **No market size, growth rate, adoption figure, volume, transaction count, user count or forecast is printed anywhere in this guide**; §15 records that omission explicitly.

---

**Purpose.** The enterprise has spent thirty years learning to onboard a *person* as a counterparty: collect an identity, run KYC, sign a contract, open an account, monitor the relationship, and — when something goes wrong — know whom to call, whom to sue, whom to report to and whom to close. In parallel, two technical communities have each built half of a new settlement primitive without noticing it. The blockchain and web3 community built a settlement layer with no intermediary, where a key is the whole of the authorisation and a transaction is final the moment it is included: it has custody, irreversibility, a public ledger and a bearer asset, but — until very recently — no autonomous actor to use them. The AI-agent community built a non-deterministic actor that reasons, plans, calls tools and acts without a human in the loop: it has autonomy, tool use and a loop, but no native money, no bearer asset and no irreversible settlement of its own. This guide is the seam between them. It treats **Agentic Web3** as the discipline of putting a wallet in the hands of an agent and asking what that does to custody, authorisation, liability, identity and control. It is a *governance and architecture* guide, not a builder's manual: it explains what changes at the boundary of the two halves, what the retrieved specifications actually say, and — repeatedly, because it matters more here than anywhere else in this repository — what they do **not** say.

**Verification-markers convention.** ✅ = verified this pass against a named primary source, with the source's own status and date given. ⚠ = vendor claim, project announcement, approximate, or a single secondary source. ⚠-knowledge = well-established industry knowledge not re-verified this pass. ❌ = could not be verified at source; every ❌ is recorded honestly in §16.1. The consolidated ledger is the Claims Audit (§15). A standard, ERC number, protocol or version is cited **only where the specification, spec repository or primary project documentation was retrieved this pass, with its date and its status (draft/review/final)**; no ERC is ever described from its number alone (HAZARD 1, §15).

**How this guide is organised.** §1 is the overview: the decoder, the thesis stated plainly, and the boundary declared by name against all five owner guides plus the DID/VC and non-human-identity siblings. §2 is the seam itself — the measured evidence that the two halves were never connected here, and what each side was missing. §3 is what makes an agent *on-chain* different: a non-deterministic actor against a deterministic, irreversible environment. §4 is the agent wallet and the custody question. §5 is identity, on-chain against off-chain. §6 is the payment rails. §7 is the smart-contract-triggering agent and liability. §8 is the DeFi-agent question. §9 is the banking and custody angle. §10 is the structural regulated view. §11 is the failure modes. §12 is the state of practice. §13 is the Cymbal Bank worked example. §14 is the anti-patterns. §15 is the claims audit. §16 records what could not be verified, holds the glossary and cross-references, and closes. Cross-references follow repo convention: same-directory guides by plain filename, `ai_llm/...` for the AI sub-folder, `../banking/...` otherwise.

---

**Table of Contents**

1. [The Overview: the Decoder, the Thesis and the Boundary](#1-the-overview-the-decoder-the-thesis-and-the-boundary)
   - 1.1 [The Decoder](#11-the-decoder)
   - 1.2 [The Thesis, Stated Plainly](#12-the-thesis-stated-plainly)
   - 1.3 [The Boundary — What This Guide Does Not Own](#13-the-boundary--what-this-guide-does-not-own)
2. [The Two Halves, and Why They Were Separate](#2-the-two-halves-and-why-they-were-separate)
   - 2.1 [The Measured Seam](#21-the-measured-seam)
   - 2.2 [What the Web3 Half Had, and What It Lacked](#22-what-the-web3-half-had-and-what-it-lacked)
   - 2.3 [What the Agent Half Had, and What It Lacked](#23-what-the-agent-half-had-and-what-it-lacked)
   - 2.4 [Why the Seam Is Now the Interesting Part](#24-why-the-seam-is-now-the-interesting-part)
   - 2.5 [The Two Sentences That Explain the Seam](#25-the-two-sentences-that-explain-the-seam)
3. [What Makes an Agent On-Chain Different](#3-what-makes-an-agent-on-chain-different)
   - 3.1 [The Collision](#31-the-collision)
   - 3.2 [The Deterministic Environment Against the Non-Deterministic Actor](#32-the-deterministic-environment-against-the-non-deterministic-actor)
   - 3.3 [The Five Properties That Change](#33-the-five-properties-that-change)
   - 3.4 [The Three Misreadings](#34-the-three-misreadings)
4. [The Agent Wallet](#4-the-agent-wallet)
   - 4.1 [The Custody Question](#41-the-custody-question)
   - 4.2 [The Standing Authority to Spend](#42-the-standing-authority-to-spend)
   - 4.3 [The Mechanisms Named, Not Restated](#43-the-mechanisms-named-not-restated)
   - 4.4 [The Control Before the Technology](#44-the-control-before-the-technology)
   - 4.5 [The Custody Matrix](#45-the-custody-matrix)
5. [The Identity Question, On-Chain Against Off-Chain](#5-the-identity-question-on-chain-against-off-chain)
   - 5.1 [What an On-Chain Agent Identity Is](#51-what-an-on-chain-agent-identity-is)
   - 5.2 [The Three Registries of ERC-8004](#52-the-three-registries-of-erc-8004)
   - 5.3 [The Off-Chain Contrast, Cross-Referenced](#53-the-off-chain-contrast-cross-referenced)
   - 5.4 [The Honest Limit](#54-the-honest-limit)
   - 5.5 [What the Registries Do Not Replace](#55-what-the-registries-do-not-replace)
6. [The Payment Rails](#6-the-payment-rails)
   - 6.1 [The x402 Rail at the Specification Level](#61-the-x402-rail-at-the-specification-level)
   - 6.2 [The Schemes and the Facilitator](#62-the-schemes-and-the-facilitator)
   - 6.3 [The Stablecoin-Rail Question](#63-the-stablecoin-rail-question)
   - 6.4 [The Communication Rails, Cross-Referenced](#64-the-communication-rails-cross-referenced)
   - 6.5 [The Rails Side by Side](#65-the-rails-side-by-side)
7. [The Smart-Contract-Triggering Agent](#7-the-smart-contract-triggering-agent)
   - 7.1 [What the Chain Records, and What It Does Not](#71-what-the-chain-records-and-what-it-does-not)
   - 7.2 [The Revert and the Mistake](#72-the-revert-and-the-mistake)
   - 7.3 [Where Accountability Sits](#73-where-accountability-sits)
   - 7.4 [The Design Rule](#74-the-design-rule)
8. [The DeFi-Agent Question](#8-the-defi-agent-question)
   - 8.1 [What Is Claimed](#81-what-is-claimed)
   - 8.2 [What Is Demonstrated](#82-what-is-demonstrated)
   - 8.3 [What the Sources Do Not Establish](#83-what-the-sources-do-not-establish)
   - 8.4 [A Note on Yield, Returns and Advice](#84-a-note-on-yield-returns-and-advice)
9. [The Banking and Custody Angle](#9-the-banking-and-custody-angle)
   - 9.1 [The Agent as a Counterparty](#91-the-agent-as-a-counterparty)
   - 9.2 [Onboarding Something That Cannot Be Onboarded](#92-onboarding-something-that-cannot-be-onboarded)
   - 9.3 [Custody of Assets an Agent Controls](#93-custody-of-assets-an-agent-controls)
   - 9.4 [Why the Existing Frameworks Do Not Map](#94-why-the-existing-frameworks-do-not-map)
   - 9.5 [The Question Set a Bank Should Ask](#95-the-question-set-a-bank-should-ask)
10. [The Regulated View, Structural Only](#10-the-regulated-view-structural-only)
    - 10.1 [What a Supervisor Would Ask](#101-what-a-supervisor-would-ask)
    - 10.2 [What This Pass Established About the Rules](#102-what-this-pass-established-about-the-rules)
    - 10.3 [The Control Posture That Survives Either Way](#103-the-control-posture-that-survives-either-way)
11. [The Failure Modes](#11-the-failure-modes)
    - 11.1 [The Table](#111-the-table)
    - 11.2 [The Modes, in Prose](#112-the-modes-in-prose)
    - 11.3 [The Common Thread](#113-the-common-thread)
12. [The State of Practice](#12-the-state-of-practice)
    - 12.1 [Demonstrably Real](#121-demonstrably-real)
    - 12.2 [Pilot, and Promise](#122-pilot-and-promise)
    - 12.3 [The Inherited Scepticism](#123-the-inherited-scepticism)
    - 12.4 [What "Pilot" Means Here](#124-what-pilot-means-here)
13. [The Cymbal Bank Worked Example](#13-the-cymbal-bank-worked-example)
    - 13.1 [The Scenario](#131-the-scenario)
    - 13.2 [The One Decision the Intersection Changed](#132-the-one-decision-the-intersection-changed)
    - 13.3 [The Thesis, at the End](#133-the-thesis-at-the-end)
14. [The Anti-Patterns](#14-the-anti-patterns)
15. [The Claims Audit](#15-the-claims-audit)
16. [What Could Not Be Verified, the Glossary, the Cross-References and the Closing Summary](#16-what-could-not-be-verified-the-glossary-the-cross-references-and-the-closing-summary)
    - 16.1 [What Could Not Be Verified](#161-what-could-not-be-verified)
    - 16.2 [The Glossary](#162-the-glossary)
    - 16.3 [The Cross-References](#163-the-cross-references)
    - 16.4 [The Closing Summary](#164-the-closing-summary)

---

## 1. The Overview: the Decoder, the Thesis and the Boundary

### 1.1 The Decoder

The vocabulary of this subject is unstable and, worse, contested: two adjacent words — "wallet" and "agent" — each already mean something specific in a discipline that is not this one. This table fixes the definitions this guide uses. Where a definition is a specification's or a project's, it is attributed and dated; where it is this guide's own framing, it is written as a definition and not dressed up as a quotation.

| Term | What it means in this guide |
|---|---|
| **Agentic Web3** | The intersection discipline: the study of an autonomous AI agent that holds a blockchain wallet, triggers on-chain state changes, or pays and is paid over an internet-native payment rail. It is the *seam*, not a new technology stack — the stack belongs to the [web3_technologies_guide.md](web3_technologies_guide.md) and the [blockchain_technology_guide.md](blockchain_technology_guide.md), and the agent belongs to the [autonomous_agents_guide.md](ai_llm/autonomous_agents_guide.md). |
| **The on-chain agent** | An AI agent whose relevant actions are transactions: it calls a smart contract, moves a token, or engages a payment protocol whose settlement is on a chain. Distinguished from the general agent of [autonomous_agents_guide.md](ai_llm/autonomous_agents_guide.md) by exactly one property — its output settles irreversibly. |
| **The agent wallet** | The key material and the on-chain account an agent acts through (or is authorised to act through). "Wallet" here is the cryptographic account, never the cash-handling human agent network (see the false friend, §1.1.1). |
| **The smart contract** | Deterministic on-chain code that executes the same way every time given the same state. Mechanics, the ERC process, proxy patterns and security classes are owned by [smart_contracts_guide.md](smart_contracts_guide.md); this guide treats the contract only as the *thing an agent calls*. |
| **The ERC standard** | An Ethereum Request for Comments standard with a status (Draft, Review, Final) and a creation date. This guide cites one **only where the specification page was retrieved this pass**, with its status and date, and never infers what an ERC does from its number (HAZARD 1). |
| **The agent identity** | On-chain: the identifier an agent is registered under on a registry (e.g. an ERC-8004 Identity Registry entry, §5). Off-chain: a DID and a verifiable credential (§5.3, cross-referenced). This guide keeps the two strictly apart, because they prove different things. |
| **The allowance / delegation** | A standing, pre-authorised permission for one address to move another's assets or to act on its behalf — the `approve`/`allowance` pattern of ERC-20 ✅ (ERC-20, Final, created 2015-11-19), or a module/session-key in a modular smart account. **The agent-era analogue of a mandate** (§4.2). |
| **The settlement** | The irreversible on-chain change of state that completes an action. The whole of the difference between an agent that *proposes* and an agent that has *acted* is whether settlement has occurred. |
| **The x402-style payment rail** | An internet-native payment protocol in which a server answers `402 Payment Required`, the client pays, and retries — x402 is the worked example (§6), governed by the Linux Foundation's x402 Foundation under vendor-neutral governance (announced 14 July 2026). |
| **The A2A and MCP rails** | The agent *communication* rails: A2A (agent-to-agent interoperability, latest released version 1.0.0) and MCP (model-to-tool integration, authoritative schema dated 2026-07-28). Owned by [autonomous_agents_guide.md](ai_llm/autonomous_agents_guide.md) §10.3; this guide cross-references them and adds only what ERC-8004 says about the gap they leave (§5.2). |

#### 1.1.1 The False Friend, Named

Two phrases in this repository look like this subject and are not. First, **"agent wallet"** in [payments_hub_guide.md](../banking/payments_hub_guide.md) §683 means the **cash-in/cash-out agent network** — a *human* agent who holds physical cash on behalf of a bank, with a wallet of float. That is a human correspondent, a different sense of the word, and **not this subject**. Second, **Adyen Agentic** and its joining of the x402 Foundation appear in [adyen_guide.md](../banking/adyen_guide.md) §362 and §716 as a **company claim** ⚠, cited there and not re-derived here. Neither is an agent with a wallet on a chain; both are named so the reader does not mistake them for one.

### 1.2 The Thesis, Stated Plainly

**An agent with a wallet is not a user with an account; it is a counterparty with no legal person behind it.**

Every distinctive property of this subject is a consequence. A **user with an account** is, in the full sense, *onboarded*: a legal person is identified, an agreement is signed, a custodian holds segregated assets, a dispute has somewhere to go, a supervisor has someone to report on, and the account can be frozen, closed and traced back to a human. A **counterparty with no legal person behind it** has none of that. It has a key or an authorisation; it acts; the action settles; and when the action is wrong there is no natural person *in the transaction* to be the defendant. That is not a security caveat bolted on at the end — it is the defining structural fact of the intersection, and §4, §7, §9 and §10 are three different views of it. Where the evidence complicates the thesis, this guide says so: §5 finds that on-chain agent identity is genuinely being standardised, but proves control of a key and not the identity of a person; §8 finds the DeFi-agent claim larger than its demonstrations; §12 finds the "agent economy" framing analyst-driven and marks it as such.

### 1.3 The Boundary — What This Guide Does Not Own

This guide is a *seam* guide and earns its place only by **not** restating its neighbours. The boundary is declared by name.

- **[web3_technologies_guide.md](web3_technologies_guide.md) (705 lines)** owns: the concept and Wood's 2014 coinage (§2), the advocates-vs-critics debate (§3), the technology layers (§4), DeFi as an application layer (§5), NFTs (§6), DAOs (§7), identity/storage/oracles (§8), governance and tokenomics (§9), the regulation (§10), a Cymbal web3-strategy assessment (§11) and the claims audit (§12). **This guide does not restate the layers, the tokenomics or the regulation.** It adds exactly one subject the umbrella cannot: the autonomous actor as a party to a transaction.
- **[blockchain_technology_guide.md](blockchain_technology_guide.md) (1,030 lines)** owns the chain: what a blockchain is (§1), architecture (§2), consensus (§4), smart contracts (§5), L2 scaling (§7), DeFi (§9), enterprise/banking (§10) and trends/challenges/regulation (§11). **[defi_guide.md](defi_guide.md) (715 lines)** owns DeFi itself: the stack (§2), the AMM (§4), lending (§5), stablecoins (§6), risks (§8), regulation (§9) and the institutional angle (§10). **This guide does not restate any of them**; §8 asks the agent question *against* the DeFi mechanics and sends the reader to §4 and §5 of the DeFi guide for the mechanics themselves.
- **[tokenized_assets_guide.md](../banking/tokenized_assets_guide.md) (799 lines)** owns tokenisation in banking: stablecoins (§2), tokenized deposits (§3), the regulatory landscape (§9) and banking/treasury (§10). **This guide cross-references it for the stablecoin-rail question in §6 only, and does not re-derive stablecoins.**
- **[autonomous_agents_guide.md](ai_llm/autonomous_agents_guide.md) (924 lines)** owns the autonomous-agent taxonomy, the autonomy spectrum, architectures, design patterns, control and safety, evaluation, frameworks and enterprise/banking adoption — and its §10.3 already frames agent-to-agent commerce and names x402, A2A and MCP. This guide **inherits §10.3's framing and its scepticism in §6 and §12 and does not repeat it**.
- **[agentic_engineering_guide.md](ai_llm/agentic_engineering_guide.md) (898 lines)** owns the engineering discipline: the lifecycle phases (§4–§11), the open problems (§15), the Cymbal worked example (§16) and the claims audit (§17). Cross-referenced, not restated.
- **Off-chain identity** is owned by [w3c_dids_guide.md](w3c_dids_guide.md) (DID Core §2, the DID document §3, methods §6, resolution §8) and [verifiable_credentials_guide.md](verifiable_credentials_guide.md) (the data model §2, formats §4, revocation §6, the KYC use case §8). §5.3 of this guide uses them as the **contrast case** and re-derives nothing.
- **The authority/register discipline for non-human identities** is owned by [non_human_identity_guide.md](non_human_identity_guide.md) — the ownership model (§7), privilege (§7.3) and the agent as NHI (§8). §4 of this guide cross-references it for *who owns the key and who registers the principal* rather than restating it.

---

## 2. The Two Halves, and Why They Were Separate

### 2.1 The Measured Seam

The claim that this repository's web3 discipline and its AI-agent discipline have never been connected is not an impression. It was measured this pass, across **668 `.md` files**, with:

```
cd /home/ubuntu/research && for t in \
  "agentic web3|web3.*agent|agent.*web3" "\bx402\b" "ERC.8004" \
  "agent wallet" "on.chain.*agent" "AI agent.*(blockchain|crypto|chain)"; do
    echo "$t: $(grep -rliE "$t" --include='*.md' . | wc -l)"
done
```

The results were:

| Pattern | Files matched across 668 `.md` files |
|---|---|
| `agentic web3\|web3.*agent\|agent.*web3` | **0** |
| `\bx402\b` | **2** — [adyen_guide.md](../banking/adyen_guide.md) and [autonomous_agents_guide.md](ai_llm/autonomous_agents_guide.md) |
| `ERC.8004` | **0** |
| `agent wallet` | **1** — [payments_hub_guide.md](../banking/payments_hub_guide.md), and it is the false friend of §1.1.1 |
| `on.chain.*agent` | **3** |
| `AI agent.*(blockchain\|crypto\|chain)` | **7** |

The decisive tests are sharper still, because they isolate the *guides that own the two disciplines*:

```
grep -ciE "agent" technology/web3_technologies_guide.md                                   # 0
grep -ciE "blockchain|on.chain|web3|smart contract|crypto" \
     technology/ai_llm/agentic_engineering_guide.md                                       # 0
grep -ciE "blockchain|on.chain|web3|smart contract|crypto" \
     technology/ai_llm/autonomous_agents_guide.md                                         # 0
```

A note on method, because the count is the evidence: in extended-regex `grep` the alternation operator is `|`, and an **escaped** `\|` returns a false zero, so the numbers above were obtained with unescaped `|`. The result is unambiguous. The guide that owns the web3 discipline mentions the word **"agent" zero times**; the two guides that own the agent-engineering discipline mention **chain, blockchain, web3, smart contract or crypto zero times each**. Whatever the two communities are doing elsewhere, in this repository they have never been connected. That is the measured observation that motivates this guide — not a thesis about the world, but a fact about the library this guide is joining.

### 2.2 What the Web3 Half Had, and What It Lacked

The [web3_technologies_guide.md](web3_technologies_guide.md) and [blockchain_technology_guide.md](blockchain_technology_guide.md) own a settlement environment with four properties no prior financial rail had assembled together:

- **Custody without an institution.** A key is the whole of the authority over an asset; there is no custodian standing between the holder and the value. Cross-referenced, not re-derived: the layer and custody maps are §4 and §8 of the web3 guide.
- **A bearer asset.** Possession of the key *is* the ability to spend — the asset does not "belong to an account at an institution"; it belongs to whoever can sign. The token standards are owned by [smart_contracts_guide.md](smart_contracts_guide.md).
- **Irreversible settlement.** A confirmed transaction is final; there is no chargeback rail and no reversal desk. The consensus and finality mechanics are §4 of the blockchain guide.
- **A public ledger.** The state everyone can see, and — as ERC-8004's own security section puts it ✅ — on-chain pointers and hashes *"cannot be deleted, ensuring audit trail integrity"*.

What that half lacked, until the agent arrived, was an **autonomous actor**. Everything it had was built for a human who clicks *confirm*: a person holds the key, a person decides, a person is the counterparty. The web3 discipline was, in the strict sense, a *human*-custody discipline that happened to be automated at the level of the transaction.

It is worth noting how *little* was technically missing, and how *late* anyone noticed. Nothing in the web3 stack had to change for an agent to use it — at the settlement layer, an agent is an address with a key, and the stack already knew how to serve an address with a key. The absence of autonomous actors from the web3 writing in this repository is therefore not a technical incompatibility; it is a fact about *who the community was writing for*, which is why the seam closed quietly the moment the agent discipline began looking for a rail rather than announcing itself as an event.

### 2.3 What the Agent Half Had, and What It Lacked

The [autonomous_agents_guide.md](ai_llm/autonomous_agents_guide.md) and [agentic_engineering_guide.md](ai_llm/agentic_engineering_guide.md) own a non-deterministic actor with four properties the web3 half never contemplated:

- **Reasoning over a goal.** The agent plans, reflects and acts in a loop (architectures owned by autonomous_agents_guide §3).
- **Tool use.** It calls functions, APIs and other agents (design patterns §4; the A2A and MCP rails §10.3).
- **Autonomy as a spectrum.** It can act without a human in the loop for some class of actions (§2 the autonomy spectrum).
- **Control, safety and evaluation.** Guardrails, human-in-the-loop and sandboxes (§5) exist precisely because the actor is not trusted to be deterministic.

What that half lacked was **money**: no native means of settlement, no bearer asset, no finality. An agent could *decide* to buy something and could not *pay* for it except through an account held by the human principal — a card on file, an API key with a prepaid balance, a bank account behind an authorisation the agent did not itself hold.

### 2.4 Why the Seam Is Now the Interesting Part

Each half has exactly the property the other was missing. Web3 has final settlement and no autonomous actor; agents have an autonomous actor and no final settlement. Bringing them together is not additive — it produces a party that can *decide and settle in the same motion*, which is the precise thing the human-custody assumption of the first half and the "human pays" assumption of the second half both depended on not existing. The rest of this guide is the working-out of that.

### 2.5 The Two Sentences That Explain the Seam

A useful test for whether a claim belongs to this guide or to a sibling is to ask which of three sentences it completes.

- *"…and therefore the settlement is final, the state is public, and the key is the authority."* This is a **web3** sentence; it is owned by [web3_technologies_guide.md](web3_technologies_guide.md), [blockchain_technology_guide.md](blockchain_technology_guide.md) and [defi_guide.md](defi_guide.md).
- *"…and therefore the actor reasons, plans, calls tools and can act without a human."* This is an **agent** sentence; it is owned by [autonomous_agents_guide.md](ai_llm/autonomous_agents_guide.md) and [agentic_engineering_guide.md](ai_llm/agentic_engineering_guide.md).
- *"…and therefore an actor that reasons has an authority that settles irreversibly, and no person is accountable for what it did."* This is an **agentic-web3** sentence, and it is the only kind this guide writes.

The seam is precisely the set of sentences of the third kind, and the reason this guide does not need to own the first two kinds is that the repository already does. The corollary is a reading rule: if a section of this guide ever starts explaining *what an AMM is* or *what a ReAct loop is*, it has crossed its own boundary and is duplicating a sibling — a failure mode this guide is built to avoid.

---

## 3. What Makes an Agent On-Chain Different

### 3.1 The Collision

Every other guide in this repository can assume a boundary between a *decision* and a *settlement*, with a human or a contract in the gap. A card authorisation is a decision, and settlement happens later, and either can be stopped. A wire is a decision, and it can be recalled within a window. Even a smart contract, which settles instantly, was *written* by a person and *called* by a person who chose the parameters. The agentic-web3 collision removes the gap: a **non-deterministic actor** issues a call into a **deterministic environment** that will execute exactly as written and settle exactly once, with no recall.

It is worth being precise about which part is non-deterministic. The **environment** is not: given the same prior state and the same call, the contract behaves identically, and this is what makes a chain auditable at all. The **actor** is: an LLM-based agent given the same goal twice may plan differently, choose a different tool, or pass different parameters. So the collision is not two random systems meeting; it is a *deterministic, unforgiving executor* being driven by a *stochastic planner*. The executor will faithfully and irreversibly carry out whatever the planner happened to decide.

### 3.2 The Deterministic Environment Against the Non-Deterministic Actor

Three consequences follow, and each is a theme later in this guide.

1. **Settlement cannot be recalled.** The moment a transaction is included, the agent's decision has become a fact about the world's shared state. There is no "undo" primitive. This is the entire difference between an agent that *recommends a payment* and an agent that has *made* one.
2. **The state is public.** Everyone can see the action, the counterparty and the amount. That cuts two ways: a virtue for audit (§7), and a leak the agent's principal may not have intended — a public, irreversible, and by-design undeletable record of an agent's activities.
3. **The contract behaves identically every time while the agent does not.** A contract cannot be "asked what it meant"; it can only be read, and it does exactly what it says. Any mismatch between what the agent *intended* and what the contract *does* is resolved silently in favour of the contract.

### 3.3 The Five Properties That Change

Stated compactly, these are the five properties that invert when the actor becomes an agent:

| Property | With a human user | With an agent counterparty |
|---|---|---|
| **Custody** | A person holds the key; a custodian may hold the asset | The key or the standing authority may sit with a program (§4) |
| **Authorisation** | A human signs each material action | A standing allowance or session key acts in the human's absence (§4.2) |
| **Settlement** | Recallable within a window; chargeback rails exist | Final on inclusion; no reversal (§3.2) |
| **Identity** | A legal person is known to a counterparty | A key is known; a legal person is an open question (§5) |
| **Accountability** | A named counterparty can be sued, onboarded, reported on | There is no natural person *in the transaction* (§7, §9, §10) |

### 3.4 The Three Misreadings

Three readings of the collision are common and all three are wrong in a way worth naming, because each collapses the agent back into a category that already has an owner, a control set and a guide.

1. **"It is just a robot trading."** A trading robot is a deterministic program with a fixed strategy; its behaviour is specifiable, testable and reproducible. The agent's behaviour is not specifiable in advance — it is a stochastic planner over a goal — and that difference is the whole reason the control problem is different.
2. **"It is just a smart contract."** A smart contract is code a human deploys and calls, and it does what the deployer wrote. The agent is the *caller*, not the contract; its decision-making is off-chain and only the *consequence* is on-chain. The contract's determinism is not the agent's.
3. **"It is just a bank API client."** A bank-API client acts inside a system where an institution holds the assets, can refuse the request, and can reverse a mistake — recall is a property of the system, not the client. The agent's on-chain action, if it settles, is neither refusable after the fact nor reversible; the property the bank API quietly provided is the property the chain does not.

The seam is the point at which each collapse fails — where the actor remains non-deterministic, off-chain in its reasoning, and acting on an environment that will not take the action back.

---

## 4. The Agent Wallet

### 4.1 The Custody Question

The first question an architect asks about an agent with a wallet is not *which chain* but *who holds the key, and under what authority*. "The agent's wallet" is shorthand for a stack of decisions that are frequently left unmade:

- **Whose key is it?** If the agent holds the private key, the agent has the whole of the authority — there is no further check, because on a chain the key *is* the authorisation. If the key sits with a custodian and the agent holds only a scoped authority, then the agent's power is whatever that scope permits and no more.
- **Who can change the target?** ERC-8004's own mechanism makes this concrete and is the sharpest illustration in this guide. Its reserved metadata key **`agentWallet`** ✅ "represents the address where the agent receives payments and is initially set to the owner's address" (ERC-8004, **Draft**, Standards Track: ERC, created 2025-08-13). Crucially, the ERC-8004 specification states that this key **cannot be set via `setMetadata()`** — and that "to change it, the agent owner must prove control of the new wallet by providing a valid EIP-712 signature for EOAs or ERC-1271 for smart contract wallets". In other words: even the standard that gives agents a portable identity keeps the *payment address* tied to a proof of human (or contract) control, precisely because a wallet address that anything could silently repoint is a custody hazard. The agent receives payments at a wallet the *owner* controls and can only move that anchor by signing.
- **What happens when the principal and the agent disagree?** With a human-held key, the human wins by definition. With a standing allowance, the principal's only defence is the scope of the allowance — which is why §4.2 treats the allowance as the load-bearing object.

This is not first a cryptographic problem; it is a **control** problem that happens to be lived out in cryptography. The [non_human_identity_guide.md](non_human_identity_guide.md) owns the general discipline — the ownership model (§7), privilege (§7.3) and the agent as a non-human identity (§8) — and it makes the point this guide will not restate: an identity without a named owner becomes an orphan, and an orphan with a key is an unmanaged authority. Read §4 of this guide as the *chain-specific* instance of that guide's §7.

### 4.2 The Standing Authority to Spend

The single most consequential object in an agent's custody stack is not the key — it is the **standing authority**: the permission granted in advance that lets the agent act while the human is not watching. On-chain, that authority has a precise shape, and the shape is old. ERC-20 ✅ (Final, Standards Track: ERC, created 2015-11-19) provides `approve`/`allowance` so that tokens "can be spent by another on-chain third party". A standing allowance granted to an agent's address is exactly that: a **delegation with no natural person behind it** on the receiving side.

The agent-era framing that makes this legible to a banker is: **the allowance is the new mandate.** In discretionary portfolio management, a mandate is a written scope — these assets, this strategy, these limits — under which a manager acts without asking each time. On-chain, an allowance or a session key is an *executable* mandate: it is the same idea with the humans removed from the middle and the scope expressed as contract logic rather than prose. And it inherits the mandate's oldest hazard: a scope that outlives its purpose remains a live authority. A token approval granted for one integration and never revoked is a mandate that no one can find the paperwork for (§11, mode 5).

### 4.3 The Mechanisms Named, Not Restated

Four retrieved specifications define the mechanisms by which an agent's spending authority can be constrained and structured. Each is cited here with its status and date; the ones owned by a sibling guide are cross-referenced rather than re-examined.

- **ERC-4337: Account Abstraction Using Alt Mempool** ✅ (**Final**, Standards Track: ERC, created 2021-09-29, authors including Vitalik Buterin, requires EIP-712 and EIP-7702). It introduces the `UserOperation` structure, an `EntryPoint` contract, bundlers, and — the extension that matters here — **paymasters**, "extensions that can sponsor gas". The agent-relevant consequence: an agent that does not itself hold native gas can still act, because a *paymaster* pays the fee on its behalf. That is a second party's authority inserted into the loop, and it must be governed as such — the paymaster has a policy about which operations it will sponsor, and that policy is a control the agent's principal may not have written.
- **EIP-7702: Set Code for EOAs** ✅ (**Final**, Standards Track: Core, created 2024-05-07). It adds a transaction type that lets an EOA — "the 'ordinary' account a person holds a key for" — *permanently* set code for itself via authorization tuples, writing a delegation indicator `(0xef0100 || address)`. For this guide the significance is boundary-marking: the line between "a person's account" and "a contract account" is exactly where an agent wallet sits, and EIP-7702 lets a personal account *become* contract-like on demand. The security section of the specification itself carries front-running-initialization and storage-management caveats, which is the point — the mechanism widens what an EOA can do and widens the attack surface with it.
- **ERC-7579: Minimal Modular Smart Accounts** ✅ (**Draft**, Standards Track: ERC, created 2023-12-14, requires EIP-165, EIP-1271, EIP-2771, EIP-4337). It defines interfaces for modular smart accounts and modules. This is where the agent-era mandate lives in the modern account model: a **spend-limit or session-key module** is a piece of code that a smart account installs to grant exactly the narrow, time-bounded, limit-bounded authority an agent needs — and to withdraw it later. "Allowance as mandate" is, in the modular-account pattern, literally a module you install and uninstall.
- **ERC-1271: Standard Signature Validation Method for Contracts** ✅ (**Final**, Standards Track: ERC, created 2018-07-25). Because contracts have no private key, a contract must be able to say whether a signature is valid *on its behalf*: `isValidSignature(hash, signature)` returning the magic value `0x1626ba7e`. This is what makes agent accounts that are *contracts* rather than EOAs legible to the signature machinery — and it is the mechanism ERC-8004's wallet-change proof falls back on for smart-contract wallets.
- **EIP-712: Typed structured data hashing and signing** ✅ (**Final**, Standards Track: Interface, created 2017-09-12). This is the mechanism by which an off-chain "intent" becomes an on-chain authorisation — the signer expresses a structured intent in a form a contract can verify. Its own security section names **replay attacks** and **frontrunning attacks**, and states plainly that the standard "does not include replay protection". Both caveats are directly agent-relevant: an agent's authorisation, once signed and broadcast, can be replayed or front-run by a third party, and the specification itself says the *application* must be the one to behave safely.

### 4.4 The Control Before the Technology

The mechanism list is long precisely because the control question is hard, and no specification in the list removes it. Three reframings that a designer should carry from this section:

1. **The scope is the mandate, and the mandate must expire.** A limit without a deadline is an authority with no reviewer. Session keys and modular limits (ERC-7579) exist so that "for this task, for this long, up to this much" is expressible — use them.
2. **Whoever pays the gas is in the loop.** A paymaster (ERC-4337) is a second authority. If the agent's principal did not choose the paymaster's policy, someone else's policy governs when the agent can act.
3. **The wallet address is a control anchor, not a detail.** ERC-8004's insistence that only a proof of control may change `agentWallet` exists because a silently repointable payment address is a silent redirection of value. Treat any wallet that *can* be repointed without a proof as a finding.

The discipline of *registering the principal and owning the identity* lives in [non_human_identity_guide.md](non_human_identity_guide.md) §7; this section is the on-chain mechanics of putting that discipline into effect.

### 4.5 The Custody Matrix

The custody question has several independent axes, and a design is only as strong as its weakest one. The matrix is a checklist, not a prescription:

| Axis | Question | Failure if unaddressed |
|---|---|---|
| **Key possession** | Who holds the key that can move value — the agent's program, the principal, or a custodian? | A program holding an unrestricted key is an unbounded authority (§11, mode 1) |
| **Authority scope** | What standing authority exists, over which assets, up to what limit, until when? | An unexpiring, uncapped mandate outlives its purpose (§4.2, §11, mode 5) |
| **Gas / sponsorship** | Who pays for the transaction, and whose policy governs whether it may proceed? | A paymaster's policy silently governs the agent's ability to act (§4.3) |
| **Revocation** | Through what mechanism, exercised by whom, can the authority be withdrawn — and has it been tested? | An authority that cannot be revoked in practice is not a control |
| **Principal binding** | Which named responsible person or entity is bound to the agent's registry identity? | The accountability question of §7.3 has no answer |

The last row is the one a banking architect should read first, because it is the precondition for the other four being meaningful: scoping and revoking an authority only matters if there is someone to hold responsible for having scoped it. A perfectly limited allowance granted by nobody accountable is a control with no owner.

---

## 5. The Identity Question, On-Chain Against Off-Chain

### 5.1 What an On-Chain Agent Identity Is

An on-chain agent identity is a registry entry: a portable identifier, held on a contract, that a counterparty can look up before deciding to interact. It answers "is this agent the agent it claims to be, and what do others say about it?" It does **not**, on its own, answer "which legal person stands behind it?" — and keeping those two questions separate is the whole of §5. This is the identity half of the thesis: the on-chain registry makes an agent *identifiable* while leaving it, in the legal sense, still a counterparty with no natural person.

### 5.2 The Three Registries of ERC-8004

The one retrieved standards specification for machine-readable agent identity is **ERC-8004: Trustless Agents** ✅ (**Draft**, Standards Track: ERC, created **2025-08-13**; authors Marco De Rossi (@MarcoMetaMask), Davide Crapis, Jordan Ellis (Google), Erik Reppel (Coinbase); requires EIP-155, EIP-712, EIP-721, EIP-1271; licensed CC0). Its own abstract states that it "proposes to use blockchains to **discover, choose, and interact with agents across organizational boundaries** without pre-existing trust, thus **enabling open-ended agent economies**". It defines **three registries**:

- **Identity Registry** — "a minimal on-chain handle based on ERC-721 with URIStorage extension that resolves to an agent's registration file, providing every agent with a portable, censorship-resistant identifier". Because it is an ERC-721, **every agent is an NFT** and is therefore *transferable* — an ownership-and-transfer point that is load-bearing for custody (§4) and for liability (§7).
- **Reputation Registry** — "a standard interface for posting and fetching feedback signals", with scoring and aggregation "both on-chain (for composability) and off-chain (for sophisticated algorithms)".
- **Validation Registry** — "generic hooks for requesting and recording independent validators checks (e.g. stakers re-running the job, zkML verifiers, TEE oracles, trusted judges)".

Its mechanism details, cited exactly: an agent's global identifier is `{namespace}:{chainId}:{identityRegistry}` (e.g. `eip155:1:0x742...`); the **`agentId` is the ERC-721 `tokenId`**; and the **`agentURI`** must resolve to an agent registration file, which may use `ipfs://`, `https://`, or a base64 `data:` URI. The registration file advertises services — web, A2A (pointing at an agent card), MCP, ENS, DID, email — and its `supportedTrust` field names "reputation", "crypto-economic" and "tee-attestation". There is an **optional endpoint-domain verification**: an agent may prove control of an HTTPS endpoint-domain by publishing `https://{endpoint-domain}/.well-known/agent-registration.json` containing a `registrations` list whose `agentRegistry` and `agentId` match the on-chain agent.

And its own honesty, which this guide quotes because it is the thesis in the standard's own words ✅: *"While this ERC cryptographically ensures the registration file corresponds to the on-chain agent, it cannot cryptographically guarantee that advertised capabilities are functional and non-malicious."* The ERC is explicit that its trust models (reputation, validation, TEE attestation) exist to *support* that verification need — i.e. to be one answer to it, not a guarantee against it. It also states, for the discovery gap, that "MCP allows servers to list and offer their capabilities... while A2A handles agent authentication, skills advertisement via AgentCards, direct messaging, and complete task-lifecycle orchestration" — but "these agent communication protocols don't inherently cover agent discovery and trust", which is the gap ERC-8004 claims to fill. Payments, it says, are "orthogonal to this protocol and not covered here", though its examples show **x402 payments** enriching feedback signals — the one place the payment rail and the identity registry touch.

### 5.3 The Off-Chain Contrast, Cross-Referenced

The off-chain approach to the same problem is owned elsewhere in this repository and is not re-derived here. [w3c_dids_guide.md](w3c_dids_guide.md) owns the decentralised-identifier standard (DID Core §2), the DID document (§3), the methods (§6) and resolution (§8). [verifiable_credentials_guide.md](verifiable_credentials_guide.md) owns the credential data model (§2), the formats (§4), revocation (§6) and the KYC use case (§8). The contrast that matters for this guide is a single sentence: the **DID + VC** approach is a way for an *issuer* — a bank, a government, a registrar — to make a signed claim about a *subject* that a verifier can check; the **on-chain registry** approach is a way for a *registry* to record that an *address* controls an *agent*, with reputation and validation attached. They are complementary, not competing: ERC-8004's own registration file carries a `DID` endpoint, and a credential could in principle be referenced by an agent's registration. But the two prove different things, and §5.4 is about exactly what the on-chain one does not prove.

### 5.4 The Honest Limit

Stated plainly, because it is the thesis and it must not be softened: **an on-chain agent identity proves control of a key, not the identity of a legal person.** ERC-8004's Identity Registry tells a counterparty that *this agent is the agent registered under this tokenId*, that its registration file is cryptographically bound to the on-chain entry, and — with the optional domain check — that the operator controls a domain. Reputation tells the counterparty what *other counterparties* have said. Validation tells it that *some validator* did a check. None of it names a company, establishes incorporation, establishes that a human is accountable, or creates anything a court can enforce against. It is the *reputation and validation* that substitute for onboarding — and that substitution is a mechanism, not a legal person. §15 records, as verified-by-omission, that this pass found **no retrieved standards specification for a cross-chain agent identity that links an on-chain agent to an accountable natural person**; §16.1 carries the honest entry.

### 5.5 What the Registries Do Not Replace

A discovery registry, a reputation score and a validation hook are genuinely useful, and it matters not to mistake what they substitute for. They substitute for a **directory** — an agent can be found, looked up and checked. They do **not** substitute for a **register** in the regulated sense: a list of accountable principals with named owners, the discipline owned by [non_human_identity_guide.md](non_human_identity_guide.md) §7. A reputation score attached to a tokenId says what counterparties have said about an address; a register says who is answerable for it.

The two questions are not interchangeable: **the on-chain registries answer "which agent is this?"; only an off-chain register answers "whose is this?"** — and this guide's thesis is that the second question does not go away because the first has been answered. ERC-8004 is admirably honest about the limit, stating that it "cannot cryptographically guarantee that advertised capabilities are functional and non-malicious" ✅; the further limit this section adds is that it does not try to say who the owner is either, and does not claim to.

---

## 6. The Payment Rails

### 6.1 The x402 Rail at the Specification Level

The one payment standard this pass retrieved at source is **x402**. It is described by its own project site as *"an open, neutral standard for internet-native payments"* ✅ (x402.org, retrieved October 2026). Mechanically it is small and old-fashioned: a server that requires payment answers a request with the HTTP status code `402 Payment Required`; the client pays and retries. The x402 documentation states the flow in five steps ✅: (1) the buyer requests a resource; (2) the server responds `402 Payment Required` including payment instructions; (3) the buyer prepares and submits a payment payload; (4) the server verifies and settles the payment itself or via an **x402 facilitator**'s `/verify` and `/settle` endpoints; (5) if payment is valid, the server provides the resource. The docs describe it as enabling "AI agents that autonomously pay for API access" and describe the standard as having "0 fees built in". x402.org states the standard is **blockchain-agnostic** — it "supports all EVM-compatible chains, Solana, and more" — with **stablecoin payments the primary use case**, and says it is "also extensible to traditional payment methods".

**Governance, cited as a governance fact.** The **Linux Foundation announced the operational launch of the x402 Foundation on 14 July 2026** and "the completed contribution of the x402 protocol by Coinbase" ✅ (x402.org announcement page, dated 14 July 2026, retrieved October 2026). The Foundation stewards x402 "under formal, open governance", and the announcement says the protocol supports "payment types ranging from traditional cards to stablecoins". Membership is cited here **only as a governance fact, and as a project announcement, not as evidence of adoption** ⚠: founding/premier members reported at the launch include Adyen, AWS, American Express, Circle, Cloudflare, Coinbase, Fiserv, Google, Mastercard, Monad Foundation, MoonPay, Ripple, Shopify, Solana Foundation, Stellar Development Foundation, Stripe and Visa. The repository's own prior record of Adyen's x402 Foundation membership is the company claim in [adyen_guide.md](../banking/adyen_guide.md) §362 ⚠.

### 6.2 The Schemes and the Facilitator

The x402 docs name payment **schemes** and a **facilitator** role; both are cited from the retrieved documentation ✅. The scheme retrieved in full this pass is **`batch-settlement`**: "for high-throughput payments where many requests are authorized individually and redeemed later in batches. The EVM implementation uses stateless unidirectional payment channels: a buyer deposits funds into onchain escrow once, then signs off-chain cumulative vouchers for each request." Its six steps are Deposit (ERC-20 funds into escrow, using **EIP-3009 or Permit2**, submitted by the facilitator), Voucher (a signed cumulative voucher per request), Verify (the server serves without waiting for on-chain transfer), Claim (the channel manager claims many channels in one transaction), Settle, and Refund. Its docs cross-reference sibling schemes it calls **`exact`** and **`upto`** ✅ (named as "Exact" and "Upto" on the batch-settlement page's "See Also"; the *full* specification of those two was not retrieved this pass and is not described here beyond their names). Networks are identified with **CAIP-2** identifiers — the docs' code uses a network string of the form `eip155:84532`.

The **facilitator** is "an optional but recommended service that simplifies the process of verifying and settling payments between clients (buyers) and servers (sellers)". The docs are explicit that it "verifies payment payloads submitted by clients" and "settles payments on the blockchain on behalf of servers" ✅, and — a sentence this guide underlines — that it "**does not hold funds or act as a custodian** — it performs verification and execution of onchain transactions based on signed payloads provided by clients". The public `x402.org` facilitator is "intended for development and testnet workflows"; production deployments are told to use a production facilitator, run their own, or self-facilitate. The specification repository is **github.com/x402-foundation/x402** (with history at coinbase/x402), and the docs describe themselves as "the credibly neutral source of truth for x402", under the Apache-2.0 license.

**A deliberate omission.** The x402 project site displays live counters — transactions, volume, buyers and sellers over a trailing window — on its landing page ✅. **This guide prints none of those figures.** They are exactly the market/adoption/volume numbers HAZARD 2 forbids, they are project-reported and unaudited, and the repository's inherited scepticism about agent-economy sizing (§12.3) applies. §15 records that omission and why. The *existence* of the counters is the reportable fact; the numbers are not.

For an agent, the structural significance of the x402 shape is that **no account is created**. The rails-design point in [autonomous_agents_guide.md](ai_llm/autonomous_agents_guide.md) §10.3 — prepare the rails, escrow or approval for agent-initiated payments — is where this guide stops: the archetype is a payment that a server grants without onboarding, which is the payment-rail half of "a counterparty with no legal person behind it".

### 6.3 The Stablecoin-Rail Question

The most common settlement asset on such a rail is a stablecoin, and the reason is structural rather than promotional: a bearer asset that a program can hold and a program can spend is what makes machine-to-machine settlement possible at all. **The stablecoin question itself — what a stablecoin is, how the reserves work, how the rail is regulated — is owned by [tokenized_assets_guide.md](../banking/tokenized_assets_guide.md) §2 and is not re-derived here.** This guide cross-references it for exactly one point: the agent-payment rail assumes a bearer settlement asset exists and works, and that assumption is an *inherited* dependency on the tokenisation and stablecoin questions that the tokenized-assets guide already owns. If stablecoin settlement is contested — and it is — then the agent-payment rail inherits the contest.

### 6.4 The Communication Rails, Cross-Referenced

The *non-payment* rails are owned by [autonomous_agents_guide.md](ai_llm/autonomous_agents_guide.md) §10.3, which already names **x402** ("HTTP-based agent payments, 2025"), **A2A** and **MCP** and frames the agent economy — and this guide **does not repeat that framing**. Two retrieved primary sources give the rails their current shape and can be cross-referenced without re-derivation:

- **A2A (Agent2Agent)** ✅ — "an open standard designed to facilitate communication and interoperability between independent, potentially opaque AI agent systems" (a2a-protocol.org/latest/specification/, retrieved October 2026). **Latest released version 1.0.0** (previous 0.3.0, 0.2.6, 0.1.0). Its goals are interoperability, collaboration, discovery, flexibility, security and asynchronicity; its guiding principles include reusing HTTP, JSON-RPC 2.0 and Server-Sent Events, and **"opaque execution"** — agents collaborate on declared capabilities "without needing to access each other's internal state, memory, or tools".
- **MCP (Model Context Protocol)** ✅ — "an open protocol that enables seamless integration between LLM applications and external data sources and tools" (modelcontextprotocol.io/specification/, retrieved October 2026), using JSON-RPC 2.0 between **Hosts**, **Clients** and **Servers**, with server features Resources, Prompts and Tools and client features including **Elicitation**. The authoritative schema retrieved is dated **2026-07-28**, and an older specification version **2025-06-18** is the version ERC-8004's own registration-file example names. The spec itself states ✅: *"While MCP itself cannot enforce these security principles at the protocol level, implementors **SHOULD**"* build consent and authorization flows, access controls and data protections.

The connection between the identity and payment halves is stated once and then left to the siblings: **ERC-8004 does not cover payments** ("orthogonal... not covered here") and **MCP does not enforce its own security principles at the protocol level**, so the identity registry, the payment rail and the communication rail are three separate systems whose *combination* — not any one of them — is what an agent wallet actually is.

### 6.5 The Rails Side by Side

The retrieved rails do four different jobs, and conflating them is a common error:

| Rail | Job | Retrieved status | What it does **not** do |
|---|---|---|---|
| **x402** | Payment over HTTP (`402`), program-to-program | Governed by the Linux Foundation x402 Foundation since 14 July 2026 ✅ | Does not create an account or an identity; does not onboard anyone |
| **A2A** | Agent-to-agent communication and task collaboration | Latest released version **1.0.0** ✅ | Does not cover payment or settlement |
| **MCP** | LLM-app-to-tool/data integration | Schema dated **2026-07-28** ✅; itself says it "cannot enforce these security principles at the protocol level" | Does not cover discovery, trust or payment |
| **ERC-8004** | On-chain agent discovery and trust (Identity / Reputation / Validation) | **Draft**, created 2025-08-13 ✅ | Does not cover payments ("orthogonal… not covered here"); proves key control, not legal-person identity |

The honest summary is that the four together — identity, communication, tool access and payment — remain four separate systems, each with its own status, and **none of them onboard a person**. An agent with all four is still a counterparty with no legal person behind it; the rails make it *findable*, *reachable* and *payable*, not *accountable*.

---

## 7. The Smart-Contract-Triggering Agent

### 7.1 What the Chain Records, and What It Does Not

When an agent calls a smart contract, the chain records a *factually complete but intention-poor* record. It records that address A called function F with parameters P on contract C at block N, that the call succeeded or reverted, what value moved, and every event the contract emitted. It does not record who the agent was acting for, what the agent believed it was doing, what its instructions said, why it chose parameter P over P′, or whether a human approved. The chain is a perfect ledger of what happened and a total silence about why.

For audit this is genuinely valuable — ERC-8004's point that on-chain pointers and hashes "cannot be deleted, ensuring audit trail integrity" ✅ applies equally to an agent's transaction trail. But the audit trail is of *actions*, not of *intentions*, and the gap between the two is where this section's problem lives.

### 7.2 The Revert and the Mistake

The distinction a chain can and cannot make:

- **A reverting transaction is legible.** If the agent calls a contract and the call fails — wrong state, failed a require, insufficient allowance — the transaction reverts, the state change does not happen, and the ledger says so cleanly. A revert is the environment *refusing* an action.
- **An agent's mistake is not.** If the agent calls a contract *successfully* with the wrong parameters — right function, wrong recipient, wrong amount, wrong timing — the chain records a perfectly successful transaction. There is no "this was a mistake" flag. The ledger cannot distinguish a correct action from a well-formed erroneous one, because from the contract's deterministic point of view there is no difference at all: it did exactly what it was told, which is exactly what it is for.

This is the definitive breakdown of the collision in §3.1: the environment's rigour applies to *form*, and the agent's fallibility lives in *intent*, and the chain has no register for intent. The security caveat in ERC-8004 is the same observation from the identity side ✅ — the standard "cannot cryptographically guarantee that advertised capabilities are functional and non-malicious" — but the transaction-level version is sharper, because unlike a bad capability, a wrong-but-valid transaction has already settled.

### 7.3 Where Accountability Sits

With no natural person in the transaction, the question "who is accountable?" has four candidate answers and no clean one:

1. **The principal who deployed the agent** — the entity that gave it a wallet and standing authority. In law this is the strongest candidate; in the transaction it is invisible.
2. **The agent's owner-of-record on the registry** — but ERC-8004's Identity Registry is an ERC-721, so the agent is **transferable**: ownership can move after the agent was deployed and before it acted, which is precisely the fact a claimant would need to be certain of.
3. **The developer of the agent or of its instructions** — often contractually distant, and the point at which liability dissolves into a supply chain.
4. **The protocol or contract the agent called** — which did exactly what it said it would, and is therefore the least culpable party in the chain.

The honest statement, and this guide will not improve on it: the accountability question has to be answered *by design, in advance* — by binding the agent's identity to a named, responsible principal and by keeping the authority scoped — because the settlement layer will not answer it after the fact. That is a control architecture, and it is the subject of §9 and §10. Where liability law itself is concerned, this guide takes no position and asserts no rule (§10).

### 7.4 The Design Rule

The section reduces to one design rule, and it is the rule a reviewer should hold an agentic-web3 design to: **make the transaction legible before it settles, because it will not be legible afterwards.** Concretely:

- **Rehearse where recall still exists** — a testnet or a forked state — so that a wrong-but-valid call is discovered before it is final (§3.2).
- **Insert a human threshold** for any action above a size the principal will regret, so that the irreversibility is chosen, not accidental.
- **Cap and expire the authority** so that no single mistaken decision can exceed the scope the principal granted (§4.2, §4.5).
- **Record off-chain what the chain cannot** — the intent, the instruction and the approval — because the chain will faithfully record *that* the agent acted and be silent about *why*, *under whose instruction*, and *within what limit* (§7.1).

The chain is a perfect record of actions and a total silence about intentions. A design that supplies the intentions is auditable; one that relies on the chain for them is not.

---

## 8. The DeFi-Agent Question

### 8.1 What Is Claimed

The loudest claims in this subject are about *DeFi agents*: an autonomous agent that reads markets, forms a strategy, and executes on-chain without a human — trading, providing liquidity, managing collateral, arbitraging across venues — and the associated claim that DeFi is the natural habitat of agentic finance because both are programmatic. This guide records the claim and resists it, for the same reason the repository resists the agent-economy sizing claims (§12.3): a category claim is not evidence.

What must be kept separate, and is kept separate here: an agent *using* DeFi primitives is entirely demonstrable and unremarkable — it is one more program calling the same contracts every other program calls. What is *claimed and larger* is the idea of the autonomous, self-directing DeFi actor as a mature phenomenon. The gap between "an agent can call a swap" and "an agent is a competitive financial actor" is the gap this section is about.

### 8.2 What Is Demonstrated

What the retrieved sources establish is a set of *mechanics* that an agent can in principle drive, and an identity/validation machinery that could in principle be pointed at such an agent:

- The **DeFi mechanics** — the AMM (§4), lending (§5), stablecoins (§6) — are owned by [defi_guide.md](defi_guide.md) and are cross-referenced, not re-derived. The point this guide adds is only that these are *callable primitives*: an agent's on-chain action in DeFi is a contract call like any other, and therefore inherits §7's liabilities (a wrong-but-valid transaction settles) in full.
- The **identity and validation machinery** — ERC-8004's Validation Registry, with "stakers re-running the job, zkML verifiers, TEE oracles, trusted judges" ✅ — is *designed* for exactly the case where a counterparty wants independent assurance that an agent actually did what it claims, or is not malicious. That is the standard's contribution to the DeFi-agent question: not that agents are safe, but that a mechanism for *checking* them was proposed, in Draft, in August 2025.

### 8.3 What the Sources Do Not Establish

Nothing retrieved this pass establishes that any particular agentic-DeFi strategy is profitable, competitive, safe or adopted at scale. Nor does anything retrieved establish that an autonomous agent performs better or worse than a human at any DeFi activity — no source in this pass makes that comparison with evidence. In the interest of HAZARD 2, this guide prints **no yield, return, TVL, volume or performance figure**, and **ranks no protocol**. The DeFi-agent question is, as of this pass, a question with a mechanism and an aspiration, and the aspiration is not the mechanism.

### 8.4 A Note on Yield, Returns and Advice

Stated plainly, per HAZARD 2: this guide prints **no yield, return, TVL, volume or performance figure** for any DeFi protocol, and **ranks none**. It takes **no position** on the value, merits or risks of any token, protocol or chain, and it is **not investment or token advice**. The DeFi-agent question is treated here strictly as a question about what the sources *establish* (§8.2) versus what is *claimed* (§8.1) versus what remains *unestablished* (§8.3) — not as a proposition to be evaluated on performance grounds. Where [defi_guide.md](defi_guide.md) §4 and §5 own the AMM and lending mechanics, the reader goes there for the mechanics; this guide only asks what changes when the caller is an agent. The reason to be strict about this is §12.3's inherited scepticism: the same framing that produces unverified size estimates produces unverified performance claims, and this guide will not print either.

---

## 9. The Banking and Custody Angle

### 9.1 The Agent as a Counterparty

A bank's entire counterparty apparatus assumes a **legal person**. Corporate onboarding, credit assessment, transaction monitoring, sanctions screening, relationship management and dispute resolution all resolve, at the end, to an entity that can be identified, contracted with, and if necessary sued. An agent with a wallet is a counterparty in the *technical* sense — it can send and receive value and the bank's systems will see an address — and not in the *legal* sense: there is no entity to onboard, no signatory, no registered office, no jurisdiction of incorporation. The thesis is at its sharpest here. Everything a bank does to a counterparty on the way in, and everything it does to one on the way out, presumes a person; the agent has neither.

### 9.2 Onboarding Something That Cannot Be Onboarded

The KYC/AML question follows directly and is not speculative. If an agent can transact, then a bank facing that agent has to decide *whose* KYC it is running. The candidates mirror §7.3 — the deploying principal, the registry owner-of-record, the developer — and none is discoverable from the transaction itself. Two structural observations, both of which the [autonomous_agents_guide.md](ai_llm/autonomous_agents_guide.md) §10.3 already made in the framing this guide inherits: the architect's stance is to treat agent counterparties "as a new channel with the same KYC/AML obligations", and to prepare "escrow/approval for agent-initiated payments". The obligations do not disappear because the counterparty is a program; they *relocate* to whatever legal entity can be identified as the principal — which is why binding an agent's registry identity to a responsible principal (§7.3) is not a nicety but the precondition for any compliance treatment at all. The trust-fabric and KYC-instrument details (verifiable-credential KYC, §8 of [verifiable_credentials_guide.md](verifiable_credentials_guide.md)) are owned there and cross-referenced, not re-derived.

### 9.3 Custody of Assets an Agent Controls

Custody is where the intersection most directly touches a bank's existing business. A bank that custodies assets for clients already has a custody discipline: segregation, reconciliation, authority-to-instruct, and a named client behind every holding. An agent that controls on-chain assets through a standing authority (a key, an allowance, a session key) inserts a party into the custody chain that is neither the client nor the bank. The [tokenized_assets_guide.md](../banking/tokenized_assets_guide.md) owns the banking-and-treasury custody question (§10) and this guide cross-references it rather than re-deriving it; the point this guide adds is that agent-control changes the *authority-to-instruct* leg specifically. "The client instructed us" becomes "the client's agent instructed us through a standing authority the client set up", and the bank must be able to demonstrate, months later, that the authority existed, that it covered the action, and that it had not been revoked (§4.2, and §11 mode 5). The register and ownership discipline for that demonstration is [non_human_identity_guide.md](non_human_identity_guide.md) §7.

The subtlety a custodian will notice first is that an agent's authority can be *invisible to the bank's own records*. An on-chain allowance or a session key does not necessarily appear anywhere in the bank's systems — it lives on a chain, granted by a key the bank may not hold, and it may have been set up by the client months earlier for an unrelated purpose. A custody relationship in which the client's instruction can arrive through an authority the custodian cannot see at the time is a custody relationship that needs a *discovery* step, not just an authorisation step: before accepting agent-mediated instructions, the custodian has to be able to enumerate the authorities that exist. That is, once again, the inventory problem of [non_human_identity_guide.md](non_human_identity_guide.md) §3 and the orphan problem of its §4, seen from the custody side.

### 9.4 Why the Existing Frameworks Do Not Map

The cleanest way to state the banking problem is that every framework a bank owns was built around a **subject with a legal personality**. Three concrete failures of mapping:

- **Counterparty risk** presupposes a counterparty whose obligations can be enforced; an agent has none. The risk, if any, sits with the principal — who may be a vendor, a customer, or the bank's own client, and is not the address on the ledger.
- **Operational risk** treats "the system did X" as an internal control question; here the actor is *external*, non-deterministic, and able to settle irreversibly, which puts it somewhere operational risk has no box for.
- **The risk register** ([ai_risk_register_guide.md](../banking/ai_risk_register_guide.md)) and the enterprise risk frame ([enterprise_risk_management_guide.md](../banking/enterprise_risk_management_guide.md)) can *hold* the agent as a risk item, but only once the bank has named the principal behind it — which returns to §9.2. This guide cross-references those frames rather than re-deriving their mechanics.

None of this is a prediction that banks *will* face agents tomorrow. It is the structural statement of why facing one is not a variant of facing a customer: the customer apparatus has a person in it, and the agent does not.

### 9.5 The Question Set a Bank Should Ask

Before treating any agent as a transacting party, the questions a bank's risk and compliance functions need answered — structural, and drawn from this guide's own reasoning rather than from any retrieved rule:

1. **Who is the legal person?** Named, onboarded, and contractually bound to the agent's actions (§7.3, §9.1).
2. **What is the agent's authority, exactly?** Scope, limit, expiry and a revocation path that has been exercised (§4.2, §4.5).
3. **Who pays the gas, and under whose policy?** The paymaster question — a second party's policy in the loop (§4.3).
4. **What can be recalled, and what cannot?** If the answer is "nothing", is a human-approval threshold in place (§3.2, §7.4)?
5. **What does the audit record contain?** On-chain actions *plus* the off-chain intent and authority the chain cannot hold (§7.1, §10.1).
6. **What happens on a mistake?** Not "the counterparty will make us whole" — there is no counterparty with enforceable obligations — but a named principal who is contractually answerable (§7.3, §9.4).
7. **Whose KYC was run, and against which entity?** The obligation relocates to the principal, not to the address (§9.2).

A design that cannot answer all seven has a control gap somewhere in §4 or §7, and reading the agent's registration file will not close it. The registration file is an input to the answer; the answer is an entity with a name and a signature.

---

## 10. The Regulated View, Structural Only

### 10.1 What a Supervisor Would Ask

This pass retrieved **no regulator rule on agent-initiated on-chain payments**. That is not a gap in effort; it is the finding, recorded in §15 and §16.1. Accordingly this section asserts **no** MAS, DORA, PSD2, SWIFT, Basel or FATF obligation on agent-initiated on-chain payments, and names **no jurisdiction's rule** without a source. What it can do — and what is genuinely useful to a banker — is state the *structural* questions a supervisor would have to answer before any such rule could exist. They are questions, not obligations:

- **Who is the accountable legal person for an action taken by an agent?** No retrieved source answers this. It is the gating question: without an answer, "who do we supervise?" has no answer either.
- **What is the licensing perimeter of an agent that takes and settles payments?** If a machine holds a bearer asset and pays for services autonomously, whether any regulated activity is being performed — and by whom — is undetermined by the sources retrieved.
- **How does an AML/CFT regime treat a counterparty that cannot supply a beneficial owner?** The same question as §9.2, seen from the supervisor's side.
- **What does an auditor attest to, and against what record?** The ledger gives an immutable action trail (§7.1); the *intent* and the *authority* trail live off-chain, and no retrieved source standardises how those combine into an auditable record.
- **What is the consumer- and market-protection story for an irreversible settlement that no one recalled?** The lack of a recall primitive (§3.2) is a fact; whether it needs a rule is not something this pass established.

The disciplined move for a licensed institution is to hold these as *open questions* and to design controls (principal binding, scoped authority, revocation, monitoring) that would satisfy a supervisor whichever way the rules land — not to assert rules that no retrieved source supports.

There is a structural reason the questions are hard to answer, and it is worth stating rather than leaving implicit. A rule is written *against a subject* — a person, a firm, an activity. Every one of the questions above asks a supervisor to name the subject of the rule, and every one of them runs into the same fact: the agent is not a subject. It is an instrument. Any eventual rule will therefore have to attach the obligation to *some* legal person — the deployer, the operator, the developer, or the institution that granted the authority — and this pass found no source that says which. That is not a gap in this guide's research; it is the current state of the subject, and §16.1 records it as such.

### 10.2 What This Pass Established About the Rules

Exactly two governance-level facts about the *standard* layer were established ✅, and neither is a regulatory rule:

1. The **x402 Foundation was operationally launched on 14 July 2026** under the Linux Foundation, "to standardize internet-native payments for AI agents and applications", stewarding the protocol contributed by Coinbase (x402.org announcement, dated 14 July 2026). A vendor-neutral governance body now exists for the payment rail.
2. **ERC-8004** is a **Draft** standards-track proposal (created 2025-08-13) that explicitly says payments are "orthogonal to this protocol and not covered here" and that it cannot guarantee advertised capabilities are "functional and non-malicious" ✅. A Draft identity proposal is the *opposite* of a rule.

Registration in a standards track, or membership of a foundation, is not supervision. This guide will not use either as a proxy for regulatory treatment.

### 10.3 The Control Posture That Survives Either Way

Because no rule on agent-initiated on-chain payments was retrieved (§10.2), the only defensible regulatory posture is one that is correct whichever way the rules land. Three elements:

- **Bind before you deploy.** A named, onboarded principal behind every agent authority is the precondition for any regime's treatment of the agent, because every regime asks "who?" first (§7.3, §9.1).
- **Scope and expire by default.** An authority that is small, time-bounded and revocable is defensible under almost any rule; an unbounded, permanent one is not — and it is the wrong design even absent any rule (§4.2, §11).
- **Keep the off-chain record.** The intent, the authority and the approval trail are what an auditor will need; the chain supplies the action trail but not these (§7.1, §7.4).

This is not a reading of any retrieved source; it is the design that does not depend on which source eventually matters. Stated as such, per §10.2, and offered as a control posture rather than a compliance claim.

---

## 11. The Failure Modes

### 11.1 The Table

| Symptom | Cause | Guardrail |
|---|---|---|
| **The agent drains its own wallet** | The agent holds the key or a broad allowance with no cap, and a single bad decision (or a loop, or an error) empties the balance | Never give an agent unbounded authority: cap the allowance, use a session key with a limit and an expiry (ERC-7579 module), keep the settlement balance minimal, and monitor spend per window (§4.2–4.3) |
| **An irreversible mistake** | A well-formed transaction with wrong parameters settles successfully and cannot be recalled (§3.2, §7.2) | Pre-settlement simulation/rehearsal on a testnet or fork, a human approval step for actions above a threshold, and a design that assumes no recall exists |
| **A prompt-injected on-chain action** | The agent treats untrusted input (a web page, a tool result, a counterparty's message) as instruction and calls a contract or pays an address it should not | Treat the wallet as a privileged tool, gate it behind the same controls the agent-engineering guide's containment patterns describe (cross-referenced in [agentic_engineering_guide.md](ai_llm/agentic_engineering_guide.md) and [autonomous_agents_guide.md](ai_llm/autonomous_agents_guide.md) §5), and scope the on-chain authority so injection cannot reach value |
| **"The agent did it" as blame-avoidance** | The principal, the operator or the developer points at the agent as if it were an accountable party, when the agent has no legal personality (§7.3) | Bind the agent's registry identity to a named responsible principal *before* deployment, and document the authority chain so accountability cannot dissolve |
| **A delegation that outlives its purpose** | An allowance, session key or module was granted for one task and never revoked; it stays a live authority (§4.2) | Expiry by default, an inventory of standing authorities with an owner, a review cadence, and a revocation path that in practice works |

### 11.2 The Modes, in Prose

**The drain** is the purest illustration of the thesis: there is no customer whose balance is protected, no chargeback desk, and no bank whose fraud rules catch the pattern — only a key with a balance and an agent deciding what to do with it. The guardrail is not better agent reasoning; it is bounding the authority so that even a wrong decision is bounded.

**The irreversible mistake** is the failure the chain's own rigour makes worse. Because the environment cannot tell a correct action from a well-formed erroneous one (§7.2), the only defences are *before* settlement: rehearsal, thresholds, and a design that does not pretend a recall exists.

**The prompt-injected on-chain action** is the failure that fuses the two disciplines' worst properties — the agent's susceptibility to injected instruction, and the chain's irreversibility. A prompt injection that can reach a wallet is a prompt injection that can settle. The containment machinery is owned by the agent guides (cross-referenced above); this guide's contribution is that the *on-chain* authority must be scoped so that containment failures do not reach value.

**"The agent did it"** is the failure mode unique to this seam. It is not a technical fault but a governance one, and it is the reason §7.3 and §9.4 keep returning to the same precondition: name the principal in advance, because after the fact there is no one in the transaction to name.

**The delegation that outlives its purpose** is the quiet one. Nothing goes wrong on the day; the authority simply remains, unowned and unreviewed, until something uses it. It is the on-chain instance of the orphan in [non_human_identity_guide.md](non_human_identity_guide.md) §4.2, and it is defeated by the same discipline: expiry, inventory, ownership, review.

### 11.3 The Common Thread

The five modes differ in mechanism and agree in structure. Each one is a version of the same gap: a decision and its settlement with no accountable person in between. The drain is an authority with no owner; the mistake is a settlement with no recall; the injection is an instruction with no human filter; the blame-avoidance is an accountability with no principal; the stale delegation is an authority with no expiry. The guardrails differ too — scope, rehearsal, containment, named principal, expiry — but every guardrail is a way of putting *something accountable* back into the gap where a human used to stand. That is why the failure-mode list and the custody matrix (§4.5) and the bank question set (§9.5) keep arriving at the same first item: name the principal, then bound the authority, then keep the record. Everything else is detail.

---

## 12. The State of Practice

### 12.1 Demonstrably Real

Applying the repository's own honesty convention, what is *demonstrably real* as of this pass is a short list, and it is short on purpose:

- **Published specifications with a status.** ERC-8004 Draft (2025-08-13); ERC-7579 Draft (2023-12-14); and the Final standards this guide relies on — EIP-712, ERC-1271, ERC-4337, EIP-7702, ERC-20, ERC-721 — each with its status and date recorded in §15. A published specification, even a Draft, is a real artefact that can be read, implemented and critiqued.
- **A Linux-Foundation-governed payment standard.** The x402 Foundation's operational launch on 14 July 2026, with the protocol contributed by Coinbase and stewarded under vendor-neutral governance ✅. A neutral governance body for a payment rail is a concrete, checkable fact.
- **Communication rails with released versions.** A2A at released version 1.0.0 and MCP with a spec schema dated 2026-07-28 ✅ — both real, both readable, both with the security caveats their own specs state.
- **A stated gap.** ERC-8004's own claim that MCP and A2A "don't inherently cover agent discovery and trust" ✅ is evidence that the identity gap is acknowledged *by the standards community itself*, not invented by sceptics.

The pattern in this list is deliberate: everything demonstrably real is a *document* — a specification with a status, a governance announcement with a date, a released version. Nothing demonstrably real is a *number about a market*. That asymmetry is the single most useful thing a reader can take from this section, because it is a filter that works even as the subject moves: ask whether a claim about agentic web3 points to a retrievable artefact, or to a figure. If it points to a figure, treat it as a project or vendor claim until proven otherwise, and check whether it is drawn from a dashboard (§6.2).

### 12.2 Pilot, and Promise

What is *pilot* — demonstrated in controlled settings but not established as scale — is agent-initiated payment over HTTP payment rails; the x402 documentation's described use cases ("AI agents that autonomously pay for API access") are the shape of a pilot, and this guide treats them as such. What is *promise* — asserted in framing language and not established by anything retrieved this pass — is the "open-ended agent economies" of ERC-8004's abstract ✅ and the "agent economy" framing generally. The promise is on the record; the establishment is not.

### 12.3 The Inherited Scepticism

This guide **inherits, and does not restate**, the repository's position from [autonomous_agents_guide.md](ai_llm/autonomous_agents_guide.md) §10.3 ✅: *"The 'agent economy' as a term is analyst/VC-driven; treat size estimates with skepticism."* Applied here: what any agent-economy framing *sizes* — transaction counts, volumes, buyer and seller numbers, adoption estimates — is exactly what HAZARD 2 forbids this guide to print, and exactly what is least trustworthy in the subject. The x402 project site's own displayed counters §6.2 are the archetype: real as a surface, unverified as evidence, and deliberately not reproduced. **This guide prints no forecast.** It takes no view on which of the standards will win, which rails will carry traffic, or whether the agent economy will arrive.

**This guide is not investment advice and not token advice.**

### 12.4 What "Pilot" Means Here

"Pilot" in this guide means a described, code-level capability demonstrated in a controlled setting — an agent paying for API access over HTTP `402`, or a smart account granting a bounded session authority — and **not** a measured outcome at scale. Anything that reports an outcome at scale (volume, count, adoption, growth) is, by this guide's discipline, either unprinted (§6.2, §12.3) or recorded as a project or vendor claim ⚠. The distinction matters because it is the same one the repository draws about the agent economy in [autonomous_agents_guide.md](ai_llm/autonomous_agents_guide.md) §10.3: a capability can be real while a market around it is asserted. A reader should be able to tell, from any sentence in this guide, whether it reports a retrieved artefact (a specification with a status) or a claim about a market — and the answer is always the former.

---

## 13. The Cymbal Bank Worked Example

### 13.1 The Scenario

*Illustrative design fiction. Cymbal Bank is the only institution in this example; the scenario is built on the verified facts above and no other named bank appears. The figures are illustrative.*

Cymbal Bank runs a treasury operation. A vendor offers Cymbal a "treasury agent": an AI agent that monitors a stated set of on-chain conditions and, when they are met, calls a smart contract to rebalance a small stablecoin position — no human in the loop, per the vendor's pitch. The vendor's agent comes with an ERC-8004-style registration: an identity-registry entry, a reputation score built from other counterparties' feedback, and a validation hook. It is, in the vendor's framing, *discoverable, reputable and validated*. Cymbal's treasury team likes it. Cymbal's risk function asks the question this guide exists to force.

### 13.2 The One Decision the Intersection Changed

Cymbal's question was **not** "is the agent's strategy good?" — that is a strategy question, and Cymbal's risk function could assess it the way it assesses any vendor strategy. The intersection changed the question to: **"whose money is at risk, and who is the counterparty we are relying on?"**

Working the difference through:

- The agent, on-chain, is a **registered identity** ✅ (ERC-8004): an `agentId` that is an ERC-721 tokenId, a registration file bound to a chain entry, a reputation score, a validation hook. To the *protocol*, the agent is a first-class, discoverable party.
- The agent is also **a counterparty with no legal person behind it**. Its registration proves control of a key ✅ ("cannot cryptographically guarantee that advertised capabilities are functional and non-malicious"), and its ERC-721 form means its **ownership is transferable** — the entity that owns the agent can change, and Cymbal would have to be certain, at the moment of any action, *which* principal owned and operated it.
- Its authority to act is a **standing authority** (§4.2). Whatever Cymbal grants — a wallet, an allowance, a session key — is a mandate that acts while Cymbal is not watching and that, if not scoped and expiring, is a live authority someone else's policy may also govern (a paymaster, §4.3).
- Its actions **settle irreversibly** ✅. There is no recall primitive (§3.2). A wrong-but-valid rebalance is a completed transaction, not a rejected proposal (§7.2).

The one decision this changed: **Cymbal did not grant the agent a position of its own to manage.** It could not, because there was no accountable entity to grant it to and no framework to onboard one. Instead Cymbal reframed the engagement so that the *counterparty* was the vendor — a legal person, onboarded the ordinary way — and the agent was treated as **an instrument the vendor operates under a scoped, expiring, capped authority on Cymbal's own terms**. The mandate was made executable (a capped allowance with an expiry, §4.2) and owned (a named Cymbal employee registered as the responsible principal, per [non_human_identity_guide.md](non_human_identity_guide.md) §7, and the vendor contractually accountable for the agent's actions). The reputation score and validation hook were treated as *inputs* Cymbal read, not as substitutes for the onboarding it did on the vendor.

### 13.3 The Thesis, at the End

Cymbal did not make the agent safe; it made the *principal* legible and the *authority* bounded, because those are the only two moves available when the counterparty cannot be onboarded. Everything that was hard about the decision — the transferable ownership, the standing authority, the irreversibility, the absent defendant — came from the same structural fact that this whole guide has been circling: an agent with a wallet is not a user with an account; it is a counterparty with no legal person behind it. The bank's answer was not to solve the agent. It was to find, name and hold the person behind it — and to make the agent's authority small enough that the person behind it could still be responsible for what it did.

---

## 14. The Anti-Patterns

Eleven patterns that this guide's reasoning, applied, says to avoid. They are the negative image of §4, §7 and §9.

1. **"The agent is the counterparty."** No. The agent is an instrument; the counterparty is a legal person who must be named. Asserting otherwise is the thesis misread (§7.3, §9.1).
2. **Onboarding the agent instead of the principal.** A reputation score and a validation hook are not KYC. The onboarding happens on the *principal*; the agent gets an authority, not a customer record (§5.4, §9.2).
3. **Granting an unbounded authority because the agent "only" does one thing.** Scope is cheap and the blast radius is not; an unsatisfiable-but-still-authorized capability is one prompt injection away from settling (§11, mode 1 and 3).
4. **Treating allowance as a forever permission.** A standing allowance or session key granted for one task and never expired is an orphaned mandate (§4.2, §11 mode 5; [non_human_identity_guide.md](non_human_identity_guide.md) §4.2).
5. **Assuming settlement can be reversed.** There is no recall primitive. Any design that depends on "we'll just undo it" is built on a mechanism that does not exist (§3.2, §7.2).
6. **Reading an ERC by its number.** A standard's behaviour is never inferred from its number; it is read from its specification, with its status and date. Anything else is the fabrication magnet HAZARD 1 names (§15).
7. **Trusting a Draft as if it were Final — or dismissing it because it is Draft.** ERC-8004 and ERC-7579 are Draft; that is a status to *state*, not a status to conceal or to over-read (§15).
8. **Treating the x402 member list as adoption evidence.** Membership is a governance fact, recorded as a project announcement ⚠. It is not a volume, a count, or a forecast, and it is not printed as one (§6.1, §12.3).
9. **Printing the socket's own dashboard as evidence of scale.** The x402 site displays live counters; a figure on a project's landing page is project-reported and unaudited and is not printed here (§6.2).
10. **Letting "the agent did it" end the analysis.** The sentence is the beginning of a governance problem, not a resolution of one (§7.3, §11 mode 4).
11. **Reading the two disciplines as one discipline.** They remain two disciplines joined at a seam. The web3 machinery and the agent machinery are each owned by their own guides; this guide owns the seam and nothing else (§1.3).

**A plain statement, made once and carried into the claims audit.** **This guide gives no investment advice and no token advice.** It takes **no position** on the value, merits or risks of any token, protocol or chain, ranks none, and — per HAZARD 2 — prints no market size, growth rate, adoption figure, volume, transaction count, user count, forecast, yield or return anywhere. Where such a figure would normally appear, this guide names its absence instead.

---

## 15. The Claims Audit

This section is the repository's honesty convention applied to this guide. Every ✅ claim above was verified this pass at the named source (with the source's own status and date); every ⚠ is a project announcement, a company claim, approximate or single-sourced; ⚠-knowledge is established practice not re-verified; ❌ marks something that could not be verified at source. Two rules governed the table and are themselves recorded as rows: **no standard, ERC number, protocol or version is cited without the specification page having been retrieved this pass, with its status and date** (HAZARD 1), and **no market, adoption, volume or return figure is printed anywhere** (HAZARD 2). Nothing was refuted this pass; ❌ is reserved for what could not be established.

| # | Claim | Status | Source, with date and kind |
|---|---|---|---|
| 1 | **ERC-8004: Trustless Agents** exists; status **Draft** (Standards Track: ERC); created **2025-08-13**; authors Marco De Rossi, Davide Crapis, Jordan Ellis (Google), Erik Reppel (Coinbase); requires EIP-155, EIP-712, EIP-721, EIP-1271; CC0 | ✅ | eips.ethereum.org/EIPS/eip-8004, retrieved October 2026 — **standards specification** |
| 2 | ERC-8004 abstract: use blockchains to "discover, choose, and interact with agents across organizational boundaries" without pre-existing trust, "enabling open-ended agent economies" | ✅ | eips.ethereum.org/EIPS/eip-8004 — **standards specification** |
| 3 | ERC-8004 defines three registries — **Identity** (ERC-721 + URIStorage, portable/censorship-resistant id), **Reputation** (feedback interface, on- and off-chain aggregation), **Validation** (hooks for stakers re-running the job, zkML verifiers, TEE oracles, trusted judges) | ✅ | eips.ethereum.org/EIPS/eip-8004 — **standards specification** |
| 4 | ERC-8004: agent id = `{namespace}:{chainId}:{identityRegistry}` (e.g. `eip155:1:0x742...`); `agentId` = ERC-721 tokenId; `agentURI` resolves to a registration file via `ipfs://`, `https://` or base64 `data:` | ✅ | eips.ethereum.org/EIPS/eip-8004 — **standards specification** |
| 5 | ERC-8004 reserved key **`agentWallet`** cannot be set via `setMetadata()`; initially the owner's address; changing it requires EIP-712 (EOA) or ERC-1271 (contract) proof of control | ✅ | eips.ethereum.org/EIPS/eip-8004 — **standards specification** |
| 6 | ERC-8004 optional endpoint-domain verification via `/.well-known/agent-registration.json` | ✅ | eips.ethereum.org/EIPS/eip-8004 — **standards specification** |
| 7 | ERC-8004 security caveat, quoted: "it cannot cryptographically guarantee that advertised capabilities are functional and non-malicious"; and "on-chain pointers and hashes cannot be deleted, ensuring audit trail integrity" | ✅ | eips.ethereum.org/EIPS/eip-8004 — **standards specification** |
| 8 | ERC-8004: MCP lists capabilities and A2A handles auth/skills/task lifecycle, but "don't inherently cover agent discovery and trust"; payments are "orthogonal to this protocol and not covered here", with x402 shown enriching feedback | ✅ | eips.ethereum.org/EIPS/eip-8004 — **standards specification** |
| 9 | **x402** described as "an open, neutral standard for internet-native payments"; HTTP `402 Payment Required` flow in five steps; blockchain-agnostic (EVM, Solana, more); stablecoins the primary use case | ✅ | x402.org and docs.x402.org, retrieved October 2026 — **project documentation** |
| 10 | **Linux Foundation announced the operational launch of the x402 Foundation on 14 July 2026**, and "the completed contribution of the x402 protocol by Coinbase"; stewarded under vendor-neutral governance | ✅ | x402.org announcement page, dated 14 July 2026, retrieved October 2026 — **dated announcement** |
| 11 | Founding/premier members include Adyen, AWS, American Express, Circle, Cloudflare, Coinbase, Fiserv, Google, Mastercard, Monad Foundation, MoonPay, Ripple, Shopify, Solana Foundation, Stellar Development Foundation, Stripe, Visa | ⚠ | x402.org announcement, 14 July 2026 — **project announcement**; membership is a governance fact, not adoption evidence, and no count or figure is drawn from it |
| 12 | x402 schemes include `batch-settlement` (EVM stateless unidirectional payment channels; deposit via EIP-3009 or Permit2; signed cumulative vouchers; batched redemption); `exact` and `upto` are named as sibling schemes | ✅ (scheme details) / ⚠ (`exact`, `upto` named only) | docs.x402.org/schemes/batch-settlement, retrieved October 2026 — **project documentation**; the full `exact`/`upto` specs were **not** retrieved and are not described beyond their names |
| 13 | x402 **facilitator** verifies and settles, "does not hold funds or act as a custodian"; networks identified with CAIP-2 (e.g. `eip155:84532`) | ✅ | docs.x402.org/core-concepts/facilitator, retrieved October 2026 — **project documentation** |
| 14 | The x402 project site **displays live counters** (transactions, volume, buyers, sellers); this guide prints none of them | ✅ (observation) / deliberate omission | x402.org landing page, retrieved October 2026 — **project documentation**; the figures are unprinted under HAZARD 2 (row 26) |
| 15 | **A2A** is an open standard for interoperability between independent, potentially opaque agents; **latest released version 1.0.0** (prior 0.3.0, 0.2.6, 0.1.0); built on HTTP, JSON-RPC 2.0, SSE; principle of "opaque execution" | ✅ | a2a-protocol.org/latest/specification/, retrieved October 2026 — **standards specification** |
| 16 | **MCP** integrates LLM apps with external data/tools; JSON-RPC 2.0; Hosts/Clients/Servers; server features Resources/Prompts/Tools; client feature Elicitation; spec itself says "MCP itself cannot enforce these security principles at the protocol level" | ✅ | modelcontextprotocol.io/specification/, retrieved October 2026 — **standards specification**; authoritative schema dated **2026-07-28**; older version **2025-06-18** named |
| 17 | **EIP-712** — status **Final** (Standards Track: Interface), created **2017-09-12**; typed structured-data signing; security section names replay and frontrunning; "does not include replay protection" | ✅ | eips.ethereum.org/EIPS/eip-712, retrieved October 2026 — **standards specification** |
| 18 | **ERC-1271** — status **Final** (Standards Track: ERC), created **2018-07-25**; `isValidSignature(hash, signature)` returning magic value `0x1626ba7e`; exists because contracts have no private key | ✅ | eips.ethereum.org/EIPS/eip-1271, retrieved October 2026 — **standards specification** |
| 19 | **ERC-4337** — status **Final** (Standards Track: ERC), created **2021-09-29**; authors incl. Vitalik Buterin; requires EIP-712, EIP-7702; UserOperation structure, EntryPoint, bundlers, paymasters that can sponsor gas | ✅ | eips.ethereum.org/EIPS/eip-4337, retrieved October 2026 — **standards specification** |
| 20 | **EIP-7702** — status **Final** (Standards Track: Core), created **2024-05-07**; lets an EOA set code via authorization tuples, writing delegation indicator `(0xef0100 \|\| address)`; carries front-running-init and storage caveats | ✅ | eips.ethereum.org/EIPS/eip-7702, retrieved October 2026 — **standards specification** |
| 21 | **ERC-7579** — status **Draft** (Standards Track: ERC), created **2023-12-14**; interfaces for modular smart accounts and modules; requires EIP-165/1271/2771/4337 | ✅ | eips.ethereum.org/EIPS/eip-7579, retrieved October 2026 — **standards specification** |
| 22 | **ERC-20** — status **Final** (Standards Track: ERC), created **2015-11-19**; `approve`/`allowance` so tokens "can be spent by another on-chain third party" | ✅ | eips.ethereum.org/EIPS/eip-20, retrieved October 2026 — **standards specification** |
| 23 | **ERC-721** — status **Final** (Standards Track: ERC), created **2018-01-24**; the standard ERC-8004's Identity Registry extends (hence agents are transferable) | ✅ | eips.ethereum.org/EIPS/eip-721, retrieved October 2026 — **standards specification** |
| 24 | **The seam, measured:** across 668 `.md` files, `agentic web3\|web3.*agent\|agent.*web3` = 0 files; `ERC.8004` = 0; `\bx402\b` = 2; `agent wallet` = 1 (the false friend); `on.chain.*agent` = 3; `AI agent.*(blockchain\|crypto\|chain)` = 7; **`agent` count in web3_technologies_guide.md = 0**; chain vocabulary count in agentic_engineering_guide.md = 0 and in autonomous_agents_guide.md = 0 | ✅ | `grep` run in this repository this pass — **repository measurement** (method noted in §2.1: unescaped `\|` alternation) |
| 25 | The false friend: `agent wallet` in [payments_hub_guide.md](../banking/payments_hub_guide.md) §683 means the **human** cash-in/cash-out agent network; Adyen Agentic + x402 Foundation membership is a **company claim** in [adyen_guide.md](../banking/adyen_guide.md) §362/§716 | ✅ (false friend) / ⚠ (Adyen claim) | payments_hub_guide.md §683 and adyen_guide.md §362/§716, read this pass — **repository** / **company claim** |
| 26 | **No market, adoption, volume, transaction-count, user-count, yield, return or forecast figure is printed anywhere in this guide**, including none from the x402 site counters (row 14) | ✅ (deliberate omission) | HAZARD 2 applied throughout; §6.2 and §12.3 record the omission and reason — **editorial constraint** |
| 27 | **No standard, ERC number, protocol or version is cited anywhere in this guide whose specification page was not retrieved this pass with its status and date** | ✅ (method) | Every citation in this table traces to a retrieved page named in its Source column — HAZARD 1 applied |
| 28 | The repository's inherited position that "the 'agent economy' as a term is analyst/VC-driven; treat size estimates with skepticism" | ✅ (inherited) | [autonomous_agents_guide.md](ai_llm/autonomous_agents_guide.md) §10.3 — **repository, inherited** |
| 29 | **No regulator or jurisdiction rule on agent-initiated on-chain payments was retrieved** this pass | ❌ | Not found at source this pass; §10 asserts no MAS/DORA/PSD2/SWIFT/Basel/FATF obligation; see §16.1 |
| 30 | **No retrieved standards specification links a cross-chain on-chain agent to an accountable natural person** | ❌ | Not found at source this pass; ERC-8004's identity proves key control, not legal-person identity; see §16.1 |

---

## 16. What Could Not Be Verified, the Glossary, the Cross-References and the Closing Summary

### 16.1 What Could Not Be Verified

Recorded honestly, per the repository convention, rather than filled in from memory:

- **A standards specification for a cross-chain agent identity tying an on-chain agent to an accountable natural person.** This pass found **none**. ERC-8004 is Draft, is per-chain-itinerant (`{namespace}:{chainId}:{identityRegistry}`), and proves control of a key rather than the identity of a legal person (§5.4). If a widely discussed "agent identity standard" is alluded to in a conversation, the honest position is: it is *discussed*; what would establish it is a **retrieved specification with a version and a status**; until then it is unverified and this guide treats it as such. ❌
- **Any regulator or jurisdiction rule on agent-initiated on-chain payments.** Nothing was retrieved. §10 states the structural questions and asserts **no** obligation under any named regime. ❌
- **`web_search` behaviour on this host.** `web_search` was intermittently empty this pass. That is recorded as a **tool limitation, not as evidence of absence** — the primary sources were retrieved by **direct page extraction** instead, and where a search was empty it means the search failed, not that the material does not exist. ⚠ (tool)
- **The full specifications of the x402 `exact` and `upto` schemes.** Only `batch-settlement` was retrieved in full; `exact` and `upto` are recorded as **named** only (row 12), not described. ⚠
- **Any measure of agent-economy scale.** Not merely unverified but deliberately unprinted (HAZARD 2, §12.3). The x402 site's counters exist and are not reproduced. ⚠/omission.

The five entries above are the whole of the unverified list, and their common shape is worth noting: none of them is a fact this guide needed and could not find; each is a fact that either does not yet exist as a retrievable artefact, or that a responsible guide should not print (the scale figures). That distinction — *absent from the record* versus *withheld by policy* — is the reason §16.1 exists as its own subsection rather than as a footnote.

### 16.2 The Glossary

- **Account abstraction** — making an on-chain account behave like a program rather than a bare keypair; ERC-4337 (Final) and ERC-7579 (Draft) are the retrieved specifications here. §4.3.
- **Agent2Agent (A2A)** — the agent-to-agent communication standard; latest released version 1.0.0. §6.4.
- **`agentWallet`** — ERC-8004's reserved, non-`setMetadata`-able payment-address key, initially the owner's address; change requires a proof of control. §4.1.
- **Allowance / delegation** — a standing permission for one address to move another's assets or act on its behalf; the agent-era analogue of a mandate. §4.2.
- **Bundler** — the ERC-4337 actor that packages UserOperations for inclusion. §4.3.
- **CAIP-2** — the chain-agnostic identifier format used to name networks; `eip155:84532` in the x402 docs. §6.2.
- **Draft / Review / Final** — the status of a standards-track proposal; a status to *state*, never to infer behaviour from. §15.
- **EntryPoint** — the ERC-4337 contract through which UserOperations are validated and executed. §4.3.
- **Facilitator** — the x402 service that verifies and settles payments and "does not hold funds or act as a custodian". §6.2.
- **Agent registry (ERC-8004)** — the Identity, Reputation and Validation registries that make an agent discoverable, scored and checkable. §5.2.
- **Paymaster** — an ERC-4337 extension that can sponsor gas; a second authority in the loop. §4.3.
- **Replay attack** — reusing a valid signed authorisation; named in EIP-712's security considerations, which state the standard "does not include replay protection". §4.3.
- **Revert** — the environment refusing an action; legible on-chain, unlike an agent's well-formed mistake. §7.2.
- **Session key** — a narrow, time- and/or limit-bounded authority an agent acts through; expressed on-chain as a module (ERC-7579). §4.2, §4.3.
- **Settlement** — the irreversible on-chain change of state that completes an action. §1.1, §3.2.
- **Standing authority** — any pre-granted permission that lets an agent act while the principal is not watching; the load-bearing object of §4. §4.2.
- **UserOperation** — the ERC-4337 pseudo-transaction object. §4.3.
- **x402** — the HTTP-native payment standard (`402 Payment Required`), stewarded by the Linux Foundation's x402 Foundation. §6.1.
- **Accountable principal** — the named person or entity that bears responsibility for an agent's actions; the thing this guide's thesis says is missing inside the transaction. §7.3, §9.5.
- **Agentic Web3** — the intersection discipline this guide owns; a non-deterministic actor acting on a deterministic, irreversible settlement layer. §1.1, §2.5.
- **Attestation** — a signed claim by a validator (TEE oracle, zkML verifier, re-execution) that a check was performed; the Validation Registry's currency. §5.2.
- **Delegation indicator** — the `(0xef0100 || address)` marker EIP-7702 writes to an EOA's code. §4.3.
- **ERC / EIP** — Ethereum Request for Comments / Ethereum Improvement Proposal; a standard with a status (Draft, Review, Final) and a creation date, read from its document, never inferred from its number. §1.1, §15.
- **Mandate** — in this guide, the analogue of a discretionary mandate: a scoped, executable standing authority. §4.2.
- **On-chain / off-chain** — whether the effect settles to shared chain state (on-chain) or exists only in a server's memory (off-chain); the difference between an agent that has acted and one that has proposed. §7.1.
- **Proof of control** — cryptographic evidence that a party holds the key to an address; what ERC-8004 requires to change `agentWallet`, and what an on-chain agent identity proves (and no more). §4.1, §5.4.
- **Session key / module** — a narrow, bounded authority installed into a smart account; the on-chain form of a limited mandate. §4.2, §4.3.
- **Smart account** — a contract account that can be programmed with modules, as opposed to a bare EOA; the ERC-7579 object. §4.3.
- **Transferable NFT** — the ERC-721 property inherited by ERC-8004's Identity Registry: an agent's ownership can move, which complicates the accountability question. §5.2, §7.3.
- **x402 Foundation** — the Linux Foundation body, operationally launched 14 July 2026, that stewards x402 under vendor-neutral governance. §6.1.

### 16.3 The Cross-References

- **The web3 umbrella and the chain** — [web3_technologies_guide.md](web3_technologies_guide.md) (§2 coinage, §3 debate, §4 layers, §5 DeFi, §6 NFTs, §7 DAOs, §8 identity/storage/oracles, §9 tokenomics, §10 regulation, §11 Cymbal, §12 audit); [blockchain_technology_guide.md](blockchain_technology_guide.md) (§1–§11); [smart_contracts_guide.md](smart_contracts_guide.md) (ERC process, token standards, security classes).
- **DeFi** — [defi_guide.md](defi_guide.md) §2 (stack), §4 (AMM), §5 (lending), §6 (stablecoins), §8 (risks), §9 (regulation), §10 (institutional); used in §8 and not re-derived.
- **Tokenisation and stablecoins** — [tokenized_assets_guide.md](../banking/tokenized_assets_guide.md) §2 (stablecoins), §3 (tokenized deposits), §9 (regulatory landscape), §10 (banking/treasury); used in §6.3 and §9.3 and not re-derived.
- **The agent discipline** — [autonomous_agents_guide.md](ai_llm/autonomous_agents_guide.md) (taxonomy, autonomy spectrum, architectures, control/safety §5, evaluation, adoption §8, and **§10.3 The Agent Economy**, inherited in §6 and §12); [agentic_engineering_guide.md](ai_llm/agentic_engineering_guide.md) (lifecycle §4–§11, open problems §15, Cymbal §16, audit §17).
- **Off-chain identity** — [w3c_dids_guide.md](w3c_dids_guide.md) (§2 DID Core, §3 DID document, §6 methods, §8 resolution); [verifiable_credentials_guide.md](verifiable_credentials_guide.md) (§2 data model, §4 formats, §6 revocation, §8 KYC); used in §5.3 and not re-derived.
- **Non-human identity** — [non_human_identity_guide.md](non_human_identity_guide.md) (§7 ownership model, §7.3 privilege, §8 the agent as NHI, §4.2 the orphan); used in §4 and §11 and not restated.
- **Banking risk and compliance frames** — [enterprise_risk_management_guide.md](../banking/enterprise_risk_management_guide.md), [ai_risk_register_guide.md](../banking/ai_risk_register_guide.md), [financial_risk_compliance_systems_guide.md](../banking/financial_risk_compliance_systems_guide.md); used in §9.4 and not re-derived.
- **The false friend** — [payments_hub_guide.md](../banking/payments_hub_guide.md) §683 (the *human* agent network); [adyen_guide.md](../banking/adyen_guide.md) §362/§716 (the Adyen Agentic company claim).

### 16.4 The Closing Summary

Two disciplines lived in this repository without meeting: one with custody, keys, finality and irreversibility, and one with reasoning, tools, autonomy and control. This guide stood in the seam and asked what happens when the second gets a wallet on the first. The answer was not additive. The environment stays deterministic and unforgiving; the actor stays stochastic and fallible; and the gap that all other guides can assume — a human, or a contract, in between a decision and its settlement — is gone. What remains is a subject that can identify itself on a chain (ERC-8004, Draft, 2025-08-13) but proves only control of a key; that can pay over an HTTP-native rail (x402, governed since 14 July 2026 by a Linux Foundation body) without creating an account; that acts through a standing authority which is an executable mandate; that settles irreversibly; and that no bank, supervisor or court can onboard, because there is no legal person inside the transaction to onboard. The evidence complicates the thesis in places — a genuine identity standard is being written, a genuine governance body exists, real mechanisms (ERC-4337, EIP-7702, ERC-7579, EIP-712, ERC-1271) exist to scope authority — and none of it supplies the missing thing. The standards can make an agent *recognisable* and *bounded*; only a named principal makes it *accountable*.

Which is why the guide ends where the banking architect has to begin, with the two questions that survive every specification, every rail and every governance body — the two this guide has returned to from §1 to §13: **whose is this?**, and **who is answerable for what it did?** Neither is a cryptographic question. The chain will never answer either one, because the chain has no register for the thing that makes them answerable — a person. What the industry can do, and what this repository's conventions require a guide to say plainly, is to stop presenting the technical standards as though they were the answer. They are the tools. The answer is a named counterparty, a bounded authority and a kept record — the unglamorous three that every section of this guide kept arriving at.

**an agent with a wallet is not a user with an account; it is a counterparty with no legal person behind it.**
