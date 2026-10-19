# CyberArk and Privileged Access: A Vault Nobody Rotates

*Privileged access management is the oldest control discipline in enterprise security and the one most often bought rather than practised. This guide owns the discipline: what a privileged account is, why it is different from an ordinary one, and what each of the four controls - vaulting, session recording, just-in-time elevation and rotation - actually prevents and does not prevent. It owns one vendor as an instance of that discipline: CyberArk Software Ltd., acquired by Palo Alto Networks on 11 February 2026, rebranded as Idira on 12 May 2026, and still called CyberArk in most contracts, URLs and community pages. It does not own non-human identity as an asset class, the secrets-management vendor comparison, mainframe privileged access, key management, or the zero-trust parent discipline; each is declared out of scope by name in section 1.4 and cross-referenced rather than restated. Its thesis is one sentence: privileged access is the one control where the tool and the process are the same thing, so a vault that nobody rotates is a database of passwords.*

**Jack Liu Shurui, Solution Architect**

> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)  
> **Last Updated:** October 2026  
> **Companion guides (technology/, same folder):** [non_human_identity_guide.md](non_human_identity_guide.md) - the sibling that owns non-human identity as a managed asset class; its section 5 owns the identity-versus-secret distinction and its section 8 owns the agent-as-non-human-identity framing, and this guide restates neither. [zero_trust_mainframe_guide.md](zero_trust_mainframe_guide.md) - section 6, "Privileged Access, Vaulting and the Named Identity", owns mainframe privileged access including 6.1 vaulting and brokering, 6.2 the firecall identity, 6.3 session brokering and 6.4 the named identity. [openbao_vs_vault_guide.md](openbao_vs_vault_guide.md) - the secrets-management vendor comparison and the vault-versus-secrets-manager boundary. [kmip_key_management_guide.md](kmip_key_management_guide.md) - the key-management discipline and the HSM/KMIP layer beneath any vault. [zero_trust_network_architecture_guide.md](zero_trust_network_architecture_guide.md) - the parent discipline: the CISA five-pillar model and NIST SP 800-207 doctrine, of which privileged access is one control family. [audit_as_code_guide.md](audit_as_code_guide.md) - the evidence-as-code frame this guide's audit section leans on.  
> **Companion guides (technology/ai_llm/, prefix `ai_llm/`):** [agent_decision_policy_guide.md](ai_llm/agent_decision_policy_guide.md) - the authority half of the agent problem: the four postures and who may decide. [agent_sandboxing_strategies_guide.md](ai_llm/agent_sandboxing_strategies_guide.md) - the containment half: the isolation spectrum and tool-execution patterns. [enterprise_agentic_platform_architecture_guide.md](ai_llm/enterprise_agentic_platform_architecture_guide.md) - section 6.2 identity integration, the platform context an agent registry has to fit into.  
> **Companion guides (banking/, prefix `../banking/`):** [enterprise_risk_management_guide.md](../banking/enterprise_risk_management_guide.md) - the risk-and-control frame that a PAM programme is an instance of. [ai_risk_register_guide.md](../banking/ai_risk_register_guide.md) - the register an agentic identity eventually has to appear in. [regtech_guide.md](../banking/regtech_guide.md) - the obligation-to-evidence mapping this guide's section 8 mirrors.  
> **Method note:** this pass had live web access in **October 2026**. Verification used **direct page extraction of primary URLs**: the Palo Alto Networks press release announcing the agreement to acquire CyberArk (30 July 2025), the release announcing completion (11 February 2026), the PR Newswire republication of that release carrying a correction notice, the PR Newswire release introducing Idira (12 May 2026), the cyberark.com/company landing page now serving Idira content, the Idira privileged-access-management product page, the Idira agentic-identity product page, and the NIST CSRC publication page for SP 800-53 Rev. 5. `web_search` was **intermittently unreliable on this host**; an empty search result is recorded as a **TOOL LIMITATION, not as absence of material**, and where a search was empty a known primary URL was extracted directly instead. Three primary sources would not extract at all this pass - EUR-Lex (for DORA), the PCI Security Standards Council's PCI DSS PDF, and a Businesswire release - and each is recorded as blocked in section 15.1 rather than filled in from memory. **Every vendor statistic in this guide is attributed as a vendor claim, with its date and its document**, and no vendor statistic is asserted as fact anywhere. All illustrative figures in section 11 are labelled illustrative and belong to the fictional Cymbal Bank, the sole institution in the sole worked example. No vendor is ranked, sized or compared on capability; no market-share, quadrant, customer-count or competitor-ranking figure appears anywhere.

---

**Purpose.** An enterprise that has governed people for a decade still cannot answer three questions about its privileged access: which accounts exist, who may use them, and what they did. The machinery of workforce identity - the directory, the joiner-mover-leaver process, the access review, the offboarding checklist - was built for subjects that have a manager, a start date and an end date. A privileged account frequently has none of the three: the service account has no manager, the local administrator on an appliance has no start date, the shared root password has no end date, and the vendor's remote-access account has an owner who works for another company. This guide writes the discipline that governs that population. It is deliberately written **vendor-neutral for its first eight sections** and only becomes a guide about one product when the discipline is already on the table, because the failure this guide is trying to prevent is the substitution of a purchase for a practice. It explains what makes an account privileged, what the four canonical controls each do and do not do, what a real deployment costs in people rather than licences, what a regulator and an auditor actually ask for, how the discipline fails in practice, and what the arrival of the autonomous agent does to a control model built for humans and scripts. It does **not** re-explain non-human identity as an asset class, the secrets-management vendor landscape, mainframe privileged access, or key management: those are owned by named siblings, and section 1.4 declares the boundary rather than blurring it.

**Verification-markers convention.** ✅ = verified this pass against a named primary source, with the source dated and the retrieval date given as October 2026. ⚠ = vendor claim, approximate, or a single secondary source. ⚠-knowledge = well-established industry knowledge not re-verified this pass. ❌ = could not be verified at source; every ❌ is recorded in section 15.1. The consolidated ledger is the Claims Audit in section 14.

**How this guide is organised.** Section 1 is the overview: the three-name decoder, the definition of the discipline, the thesis and the boundary. Section 2 is what a privileged account is and why the ordinary access model fails on it. Section 3 is the core: the four controls, each with its mechanism, what it prevents, what it does not prevent and what it costs. Section 4 identifies the vendor with dates and sources. Section 5 decodes the product surface by capability category. Section 6 is the agentic identity problem, the newest and least settled part. Section 7 is the deployment and operating reality. Section 8 is the banking and regulated angle, including the auditor's question list and the evidence table. Section 9 is the honest assessment, including where the vendor's own material complicates the thesis. Section 10 is the failure modes. Section 11 is the Cymbal Bank worked example. Section 12 is the anti-patterns. Section 13 separates what is demonstrably true from what is vendor promise. Section 14 is the claims audit and the three-ratio conflict. Section 15 records what could not be verified, holds the glossary and the cross-references. Section 16 closes. Cross-references follow repository convention: same-folder guides by plain filename, `ai_llm/...` for the AI sub-folder, `../banking/...` otherwise.

---

**Table of Contents**

1. [The Overview: Three Names, One Thesis, and the Boundary](#1-the-overview-three-names-one-thesis-and-the-boundary)
   - 1.1 [The Three Names, Decoded](#11-the-three-names-decoded)
   - 1.2 [What PAM Is, and Why This Guide Leads with the Discipline](#12-what-pam-is-and-why-this-guide-leads-with-the-discipline)
   - 1.3 [The Thesis](#13-the-thesis)
   - 1.4 [The Boundary: What This Guide Does Not Own](#14-the-boundary-what-this-guide-does-not-own)
2. [What a Privileged Account Is, and Why It Is Different](#2-what-a-privileged-account-is-and-why-it-is-different)
   - 2.1 [Reach, Not Seniority](#21-reach-not-seniority)
   - 2.2 [The Three Populations](#22-the-three-populations)
   - 2.3 [Why the Ordinary Access Model Fails](#23-why-the-ordinary-access-model-fails)
   - 2.4 [Standing Privilege: The Actual Object of the Discipline](#24-standing-privilege-the-actual-object-of-the-discipline)
   - 2.5 [The Asymmetry That Explains the Product](#25-the-asymmetry-that-explains-the-product)
3. [The Four Controls and What Each Actually Prevents](#3-the-four-controls-and-what-each-actually-prevents)
   - 3.1 [Vaulting](#31-vaulting)
   - 3.2 [Session Recording and Monitoring](#32-session-recording-and-monitoring)
   - 3.3 [Just-in-Time Elevation with Approval](#33-just-in-time-elevation-with-approval)
   - 3.4 [Automated Credential Rotation](#34-automated-credential-rotation)
   - 3.5 [The Consolidated Table](#35-the-consolidated-table)
4. [The Vendor, Precisely Identified](#4-the-vendor-precisely-identified)
   - 4.1 [The Corporate Facts, Dated](#41-the-corporate-facts-dated)
   - 4.2 [What the Acquisition Changed](#42-what-the-acquisition-changed)
   - 4.3 [What It Did Not Change](#43-what-it-did-not-change)
   - 4.4 [The Leadership and the Naming](#44-the-leadership-and-the-naming)
5. [The Product Surface, Decoded](#5-the-product-surface-decoded)
   - 5.1 [The Capability Table](#51-the-capability-table)
   - 5.2 [The Human Pillar](#52-the-human-pillar)
   - 5.3 [The Machine and Agentic Pillars](#53-the-machine-and-agentic-pillars)
   - 5.4 [The Packaging Honesty](#54-the-packaging-honesty)
6. [The Agentic Identity Problem](#6-the-agentic-identity-problem)
   - 6.1 [What Changes When the Identity Is an Agent](#61-what-changes-when-the-identity-is-an-agent)
   - 6.2 [The Vendor's Own Problem Statement](#62-the-vendors-own-problem-statement)
   - 6.3 [What the Four Controls Mean for a Non-Human Actor](#63-what-the-four-controls-mean-for-a-non-human-actor)
   - 6.4 [What Is Genuinely Unsettled](#64-what-is-genuinely-unsettled)
   - 6.5 [The Standards Gap](#65-the-standards-gap)
7. [The Deployment and Operating Reality](#7-the-deployment-and-operating-reality)
   - 7.1 [Discovery and the Credential Inventory](#71-discovery-and-the-credential-inventory)
   - 7.2 [Onboarding in Waves](#72-onboarding-in-waves)
   - 7.3 [The Integration Surface](#73-the-integration-surface)
   - 7.4 [The Organisational Work](#74-the-organisational-work)
   - 7.5 [Before and After: The Estate](#75-before-and-after-the-estate)
   - 7.6 [The Two Timelines](#76-the-two-timelines)
8. [The Banking and Regulated Angle](#8-the-banking-and-regulated-angle)
   - 8.1 [The Obligations That Push Toward PAM](#81-the-obligations-that-push-toward-pam)
   - 8.2 [What an Auditor Actually Asks](#82-what-an-auditor-actually-asks)
   - 8.3 [The Evidence Table](#83-the-evidence-table)
   - 8.4 [What This Guide Does Not Assert](#84-what-this-guide-does-not-assert)
9. [The Honest Assessment](#9-the-honest-assessment)
   - 9.1 [What This Class of Product Solves](#91-what-this-class-of-product-solves)
   - 9.2 [What It Displaces Rather Than Solves](#92-what-it-displaces-rather-than-solves)
   - 9.3 [The Lock-In Question](#93-the-lock-in-question)
   - 9.4 [A Product Is Not a Control](#94-a-product-is-not-a-control)
   - 9.5 [Where the Vendor's Own Material Complicates the Thesis](#95-where-the-vendors-own-material-complicates-the-thesis)
10. [The Failure Modes](#10-the-failure-modes)
    - 10.1 [The Table](#101-the-table)
    - 10.2 [The Modes in Prose](#102-the-modes-in-prose)
11. [The Cymbal Bank Worked Example](#11-the-cymbal-bank-worked-example)
    - 11.1 [The Estate](#111-the-estate)
    - 11.2 [The Discovery](#112-the-discovery)
    - 11.3 [The Decision the Discipline Changed](#113-the-decision-the-discipline-changed)
    - 11.4 [The Thesis, Restated](#114-the-thesis-restated)
12. [The Anti-Patterns in Adopting This Class of Product](#12-the-anti-patterns-in-adopting-this-class-of-product)
13. [The State of Practice](#13-the-state-of-practice)
    - 13.1 [True, and Promised](#131-true-and-promised)
    - 13.2 [What to Watch Over the Next Year](#132-what-to-watch-over-the-next-year)
14. [The Claims Audit](#14-the-claims-audit)
    - 14.1 [The Table](#141-the-table)
    - 14.2 [The Ratio Conflict, Printed Side by Side](#142-the-ratio-conflict-printed-side-by-side)
15. [What Could Not Be Verified, the Glossary and the Cross-References](#15-what-could-not-be-verified-the-glossary-and-the-cross-references)
    - 15.1 [What Could Not Be Verified](#151-what-could-not-be-verified)
    - 15.2 [The Glossary](#152-the-glossary)
    - 15.3 [The Cross-References](#153-the-cross-references)
16. [The Closing Summary](#16-the-closing-summary)

---

## 1. The Overview: Three Names, One Thesis, and the Boundary

### 1.1 The Three Names, Decoded

Three names are in circulation for one thing, and a reader who does not separate them will misread every document they touch in the next two years. The company that owns the product is Palo Alto Networks. The product brand is Idira, unveiled 12 May 2026. The name CyberArk survives in three specific places: the legal entity acquired, the ticker Palo Alto Networks intends to adopt on the Tel Aviv Stock Exchange, and the name in almost every existing contract, purchase order, URL, runbook and community forum post. The table below is the decoder this guide uses throughout.

| Name | What it is | Status | Date | What it is called in the field |
|---|---|---|---|---|
| **Palo Alto Networks** | The listed company (NASDAQ: PANW) that now owns the product | Owner; listed and trading | Acquisition completed 11 Feb 2026 | "PANW", "the platform company", "the parent" |
| **CyberArk Software Ltd.** | The Israeli-domiciled legal entity acquired; formerly NASDAQ: CYBR | Wholly owned subsidiary | Agreement 30 Jul 2025, completed 11 Feb 2026 | "CyberArk", the name on the contract, the invoice and the support entitlement |
| **Idira** | The product and platform brand replacing "CyberArk Identity Security Platform" as the name of the offering | Generally available from launch | Unveiled 12 May 2026 | "Idira", "the Idira platform", and - for some years yet - "CyberArk" by everyone who has used it since before May 2026 |
| **CYBR** | The ticker CyberArk traded under on Nasdaq; the ticker PANW intends to adopt for a secondary listing on the Tel Aviv Stock Exchange | Nasdaq listing retired on acquisition; TASE use is a stated intent, not a completed listing | Intent announced 11 Feb 2026 | "CYBR", and in the Israeli market, a brand tribute rather than a company |
| **cyberark.com** | The web domain | Still serving Idira pages, including the company page titled "Idira - The Identity Security Platform" | As at October 2026 | "the CyberArk site", even though the page says Idira throughout |

Two consequences follow, and both are practical rather than pedantic. First, **a rebrand is not a migration**: no customer's contract, connector, agent or runbook changed on 12 May 2026, and the domain did not move, so "CyberArk" remains the correct search term for anyone trying to find documentation. Second, **the three names date differently**. A document written in March 2026 that says "CyberArk Identity Security Platform" is not wrong; a document written in October 2026 that calls the *product* CyberArk is using the old brand, and a document that calls the *company* Idira is using the wrong name entirely.

The arithmetic of the rebrand is worth stating because it is unusual. The acquisition completed on 11 February 2026 and the new brand was unveiled on 12 May 2026: three months, not a gradual fade across several release cycles. Palo Alto Networks announced the intent to acquire on 30 July 2025, closed on 11 February 2026, and renamed the platform on 12 May 2026. Whatever else is true, the branding decision was made fast.

### 1.2 What PAM Is, and Why This Guide Leads with the Discipline

Privileged access management is the set of controls that govern accounts able to change the configuration, data or availability of a system: the administrator, the root account, the database owner, the service account a batch job runs as, the domain controller's replication credential, the console account on a network appliance, the emergency account kept for the day the identity provider is down. PAM as a discipline is older than any product sold under the label. Its four canonical controls - **vaulting**, **session recording**, **just-in-time elevation** and **credential rotation** - are control patterns that appear, in vendor-neutral language, in published control frameworks: least privilege and account management in NIST SP 800-53 (AC-6, AC-2), authenticator management (IA-5), and audit record generation and review (AU-12, AU-2); and "privileged access rights" as an explicitly named control in ISO/IEC 27001:2022 Annex A 8.2. The frameworks name the controls; they do not name a product.

This guide leads with the discipline for a reason that is itself the thesis. A PAM product is not a scanner or a firewall: it does not detect something and report it. It **interposes itself between a human or a machine and a target system**, and it holds the credential, brokers the session and records the action. That means the product's behaviour *is* the control's behaviour. If the vault holds credentials nobody rotates, the control that the vault was bought to satisfy does not exist - the inventory exists, the encryption exists, the compliance dashboard exists, and the control does not. There is no other category in security where the gap between "installed" and "operating" is so wide, and none where the gap is so easy to hide, because a vault at 100 per cent onboarding looks identical to a vault at 100 per cent onboarding with 12 per cent rotation coverage.

### 1.3 The Thesis

**Privileged access is the one control where the tool and the process are the same thing: a vault that nobody rotates is a database of passwords.**

Unpacked in four lines:

- **The tool is not a sensor.** A SIEM that is poorly tuned still ingests logs; a vulnerability scanner with a stale feed still scans. A vault with no rotation schedule, no owner per account and no review of its recordings is not a degraded control - it is a **credential store with a compliance logo**, and it concentrates every password it holds into one target.
- **The process is not paperwork.** Deciding which account may reach which system, for how long, approved by whom, is the control. The product records and enforces that decision; it cannot make it, because the decision requires knowing what the system does and who is accountable for it.
- **Therefore the failure mode is procedural and looks technical.** Estates do not fail because the vault was misconfigured. They fail because rotation was scoped as a project rather than a schedule, because session review was assigned to nobody, because the emergency path became the everyday path.
- **And therefore the vendor's roadmap cannot close the gap on its own.** Every increment of automation - AI-summarised sessions, automatic termination of anomalous sessions, agent kill switches - moves judgement into the tool. That may be the right bet. It is a bet, and section 9.5 states the disagreement plainly rather than resolving it in the vendor's favour.

### 1.4 The Boundary: What This Guide Does Not Own

This guide owns **PAM as a discipline** and **one vendor's product as an instance of it**. Five adjacent subjects are owned elsewhere in this repository, and the boundary is declared by name so that neither guide repeats the other.

| Owner guide | What it owns | What this guide does not restate |
|---|---|---|
| [non_human_identity_guide.md](non_human_identity_guide.md) | Non-human identity as a managed **asset class**: the inventory problem, the orphan, ownership, lifecycle, and the agent-as-NHI framing (its section 5 is the identity-versus-secret distinction; its section 8 is the agent as a non-human identity) | The taxonomy of non-human identities, the orphan as the central failure case, and the argument that an identity is not a secret. This guide uses those conclusions and cites them; it does not re-derive them. |
| [openbao_vs_vault_guide.md](openbao_vs_vault_guide.md) | The **secrets-management vendor comparison** - the open-source and commercial secrets engines, their architectures and their trade-offs | Any comparison of secrets managers, and any claim about which secrets engine is appropriate. This guide treats "secrets management" as a product category adjacent to PAM and says where the two overlap (section 5.1). |
| [zero_trust_mainframe_guide.md](zero_trust_mainframe_guide.md) | **Mainframe privileged access**: section 6, "Privileged Access, Vaulting and the Named Identity", with 6.1 vaulting and brokering, 6.2 the firecall (emergency) identity, 6.3 session brokering and 6.4 the named identity | The mainframe-specific mechanics: the z/OS security server, the RACF-style profile model, the console and TSO paths, the batch-submission identity. This guide refers to mainframe accounts as one account class in an estate and stops there. |
| [kmip_key_management_guide.md](kmip_key_management_guide.md) | **Key management** and the HSM/KMIP layer: key lifecycle, key custody, the protocol between a key manager and an HSM | How a vault protects its own master key, how keys are wrapped, what KMIP does. A PAM vault is a consumer of that layer, not an implementation of it. |
| [zero_trust_network_architecture_guide.md](zero_trust_network_architecture_guide.md) | The **zero-trust parent discipline**: the CISA five-pillar model, NIST SP 800-207 doctrine, the estate migration plan | The zero-trust argument itself, and the identity-pillar architecture. Privileged access is one control family inside that architecture; this guide does not re-argue zero trust. |

The line is easy to state: **if the question is "what is this class of identity and how should it be governed", the answer is in [non_human_identity_guide.md](non_human_identity_guide.md); if the question is "how does an administrator reach a system, and what stops them", the answer is here.**

---
## 2. What a Privileged Account Is, and Why It Is Different

### 2.1 Reach, Not Seniority

The common definition is wrong in a way that matters operationally. "A privileged account is an administrator's account" is how vendors open a pitch and how organisations scope a project, and it produces an inventory that omits most of the risk. The defining property is **reach**: an account is privileged because of what it can reach and change, not because of whose name is on it or what its job title is.

The working test this guide uses has three questions. Can the account **change the configuration** of a system that other systems depend on? Can it **read or modify data** beyond what its business function requires? Can it **alter the controls that would record it** - stop a service, change a log destination, modify a permission, disable an agent? An account that answers yes to the third question is privileged regardless of what it is called, because an actor holding it can erase the evidence of everything else it does.

That test pulls in populations the org chart does not label "admin": the DBA who can read every table in the core banking schema, the helpdesk operator who can reset any password including an administrator's, the ERP business user with a role that can post journals without approval, the CI/CD service account that can deploy to production, the backup service account that can read every file on every server, the network engineer's login to the firewall management plane, the monitoring account with a read-only credential that happens to be able to run commands on every host. Each is privileged. None is called an administrator.

### 2.2 The Three Populations

The estate contains three populations, and they are not variants of each other. They differ in how they authenticate, what a lifecycle event even means for them, and therefore which control fits.

| Population | Examples | How it authenticates | What "leaver" means | Control that actually fits |
|---|---|---|---|---|
| **Human administrators and privileged business users** | Domain admins, Unix root via sudo, database administrators, network engineers, helpdesk with password-reset rights, finance users with posting authority | Interactive login, often with MFA, often to a jump host; sessions that a person is sitting in front of | An HR event, a ticket, a manager, a date | Session brokering and recording, just-in-time elevation, approval workflows |
| **Service, machine and application accounts** | Database logins used by an application, scheduled-job accounts, integration accounts between packages, local administrator on servers and appliances, CI/CD deploy accounts, backup agents, monitoring agents | Non-interactive: a connection string, a key file, a service principal, a mounted secret, a hard-coded password in a config | Nothing - there is no HR event, and the account outlives the project that created it | Vaulting with application delivery, automated rotation, ownership registration |
| **Agents and autonomous workloads** | An LLM-driven operations agent, a ticket-triage agent with tool access, an agent that can call an API to change a record, an MCP-connected assistant | Delegated or brokered credentials, workload identity, on-behalf-of tokens; sometimes a human's stolen session | Nothing, and often no record that it exists at all | An open problem; section 6 states what is settled and what is not |

The third population is new and is the reason the vendor reorganised its product around human, machine and agentic in 2026. The first two populations are old, and the discipline's four controls were designed for the first and retrofitted to the second. Almost every failed PAM programme in a large estate failed because it applied the first population's controls to the second.

### 2.3 Why the Ordinary Access Model Fails

Ordinary access control - the model behind the identity provider, the access review, the joiner-mover-leaver process and the role catalogue - rests on five assumptions. A privileged account breaks all five.

**(a) It assumes a subject who can be trained, disciplined and fired.** The model works because a person has an interest in keeping their job, can be instructed, and can be removed. A service account has none of these properties, and an agent has no employment relationship at all. There is no sanction available against a credential.

**(b) It assumes access is granted to do a job.** Access requests in the ordinary model describe a role or a function: "finance analyst", "HR business partner". Privileged access is frequently granted to **fix a system at 03:00**, which is not a role. It is an event. Modelling an event as a role produces a permanent entitlement for an occasional need - which is precisely the standing privilege problem of section 2.4.

**(c) It assumes a human timescale for joiner-mover-leaver.** A person joins, changes team, and leaves on dates that HR records. A service account's leaver event never fires, because no system emits it: the project ends, the contractor's engagement closes, the application is decommissioned, and the account remains. The repository's [non_human_identity_guide.md](non_human_identity_guide.md) treats this as the orphan case and section 4.2 there is the fuller treatment; the point here is narrower - **the ordinary model has no event to hook**.

**(d) It assumes access reviews catch drift.** A review works by asking an owner whether an entitlement is still needed. A privileged account that no one will admit to owning never enters the review, and an account whose "owner" is a team distribution list produces an approval from someone who cannot know. The review is not failing; it is being asked a question about a subject that has no answer.

**(e) It assumes accountability is one-to-one.** The model's integrity rests on the proposition that an action in a log maps to one person. A shared root password, a shared local administrator, a shared database owner account or a vendor's remote-access account makes every action attributable to a group, which in practice means attributable to nobody. This is not a logging defect; it is a property of the account design.

### 2.4 Standing Privilege: The Actual Object of the Discipline

**Standing privilege is the privilege that exists when nobody is using it.** It is the entitlement that persists at 04:00 on a Sunday, on a system nobody is working on, held by an account nobody is watching. It is the object of the discipline, and naming it precisely changes what gets measured.

The reason is arithmetic. An administrator uses privileged access for a small number of minutes or hours in a working month. The entitlement exists for all of it, plus every night, plus every holiday, plus every month the administrator is on leave, plus the period after they leave the company and before someone notices. The exposure is not proportional to the use; it is proportional to the **existence**. Every control in section 3 is, in one way or another, an attempt to make existence track use: just-in-time elevation shortens existence to the task, rotation shortens the life of the credential that grants it, vaulting removes the credential from the place where it can be used ambiently, and session recording makes the use visible.

A practical measurement follows, and it is the one this guide recommends over an onboarding percentage: for each account class, **the ratio of standing privileged entitlement to observed privileged use**. An account with permanent domain admin that is used twice a quarter is a 40:1 or worse exposure ratio. An estate that can state that ratio per class knows more about its privileged risk than an estate that can state its vault onboarding percentage, because the first measures the exposure and the second measures the deployment.

### 2.5 The Asymmetry That Explains the Product

The three populations have different economics, and the controls do not transfer between them. This is the single most useful thing to understand about why PAM products are sold as modules rather than as one thing.

Session recording is close to useless for a service account: there is no keystroke stream to watch, no window to capture, and no human behaviour to deter, and the "session" may be a single API call. What a service account needs is a credential it never holds and a rotation it never notices. Conversely, automated rotation is a poor fit for a human's interactive account at the moment of use: rotating a password mid-session, or forcing a re-authentication on a schedule that has nothing to do with the task, converts a security control into a workflow interruption, and the organisation's response to that interruption is the shared account it was trying to eliminate.

Vaulting cuts across both, which is why it is the oldest and most universal component - but even vaulting behaves differently: a human checks a credential out and uses it interactively, while an application needs the credential **delivered** at connection time, injected into a session, or brokered entirely so that the application never sees it. Those are different engineering problems with different failure modes, and the industry solved them at different times.

The asymmetry is not a marketing artefact. It is why a single "PAM programme" with a single success metric will misreport: the human half can be measured in sessions and approvals, the machine half can only be measured in coverage, rotation success and orphan counts, and the two measures move independently. An estate can be at 95 per cent on one and 20 per cent on the other, and the dashboard will show one number.

---

## 3. The Four Controls and What Each Actually Prevents

This is the section that makes the rest of the guide usable by someone who never buys this or any other product. Each control is described in the same four parts: the mechanism, what it prevents, what it does **not** prevent, and what it costs in people and breakage. The four-part structure is deliberate, because vendor material reliably supplies the first two and omits the last two.

### 3.1 Vaulting

**The mechanism.** The credential is removed from the place it currently lives - a spreadsheet, a wiki page, a script, an administrator's memory, a configuration file - and stored encrypted in a central repository. Access to it becomes a transaction: an identity authenticates to the vault, is authorised for that credential, and either receives it for a bounded period (checkout) or, better, never receives it at all, because the vault injects it into the connection or brokers the session on the user's behalf. The vault records who took which credential, when, and for what stated reason.

**What it prevents.** Credential sprawl, which is the state in which the same password exists in eleven places, three of which nobody can name. Casual credential access: the person who could read the wiki can no longer read the password, and the person who could shoulder-surf the terminal sees a brokered session instead of a password prompt. It produces the **inventory** - the list of credentials the organisation actually holds - which is the artefact every other control depends on. And it is the precondition for rotation: you cannot rotate a credential you do not control, in a place you cannot reach, on a schedule you cannot enforce.

**What it does not prevent.** It does not prevent an attacker who is already inside an authorised session. Once a credential is checked out or a session is brokered, the vault has done its job; what happens next is section 3.2's problem. It does not prevent the vault itself being compromised, and this is the uncomfortable one: **vaulting makes the vault the single highest-value target in the estate**, because it is the one system whose compromise yields every other system's credentials. The vendor's own product page names the attack pattern - "vault sweeping" - in its description of identity threat detection, which is an implicit admission that the concentration is real. It does not prevent a credential being pasted into a script by someone entitled to check it out, or embedded in a container image, or committed to a repository; the vault governs the checkout, not the afterlife of the string. And it does not prevent the account being used directly on the target by anyone who still knows the password because rotation has not yet happened.

**The operational cost.** Connectors: each target platform needs a way for the vault to change and verify its credential, and the platforms that most need it - appliances, older operating systems, vendor-supplied accounts, packaged applications with their own user stores - are the ones with the weakest interfaces. Application onboarding: a hard-coded connection string has to be changed to a vault call or a broker, which is a code change in an application nobody wants to touch. Tier-0 obligations: the vault must be highly available, backed up, patched, disaster-recovery tested, monitored, and itself protected by privileged access control, which means the programme has to solve its own problem for its own infrastructure. Network placement: the vault and its session proxies sit on a path between every administrator and every target, so they become a latency and availability dependency for operations. And people: someone has to own the vault, approve the checkouts, and answer the 03:00 call when the vault is down and nobody can log in to fix it.

### 3.2 Session Recording and Monitoring

**The mechanism.** The privileged session is routed through a proxy rather than made directly from the administrator's workstation to the target. The proxy authenticates the user, injects or brokers the credential, and records what happens: keystrokes, commands, window titles, sometimes full-motion video of the desktop, file transfers, and the text of the session. The recording is stored, indexed against the user, the target, the time and (where integrated) the change record, and made searchable. Newer implementations add automated analysis - the vendor's page describes "AI-generated summaries" that "surface anomalous commands" - which is an attempt to solve the review problem described below.

**What it prevents.** The deniable action. This is the control's real product: not the prevention of the act, but the elimination of the claim that it did not happen. It gives forensic evidence after an incident (what did the intruder run, in what order, with what output), it supports audit sampling (show me a session for this change window), and it exerts a deterrent effect that is real but unmeasurable. It also catches the insider who is authorised but acting improperly, which no amount of entitlement management will catch, because the entitlement was legitimately granted.

**What it does not prevent.** It does not prevent the action. A recording is a camera, not a lock; the destructive command executes and is then available for replay. It does not prevent abuse that looks like legitimate work - the administrator who reads a customer table they had no business reading is doing something the session recording faithfully captures and no rule will flag unless someone looks. It does not cover sessions that bypass the proxy: direct-to-target paths that remain open, console access, out-of-band management interfaces, a local console at a branch, an application that connects directly. And a recording nobody reviews prevents nothing at all; it only documents.

**The operational cost.** Storage, and it is not trivial: full-motion video of interactive sessions across an administrator population accumulates at a rate that surprises finance, and retention policies are usually written by legal rather than by security. Privacy and works-council obligations: in several jurisdictions, recording an employee's full desktop session requires consultation, a lawful basis, and a notice; an estate that installs recording first and asks later creates an industrial-relations problem that delays the programme by a year. Personal data inside recordings: a session on a customer system may display customer records, so the recording inherits the data-protection obligations of the data it captured. Review labour: someone must sample, with a defined cadence, and have the authority to raise a finding. Latency and compatibility: the proxy sits in the path, and protocols that do not proxy cleanly (some graphical clients, some file-transfer tools, some vendor appliances) will either break or need an exception.

### 3.3 Just-in-Time Elevation with Approval

**The mechanism.** The entitlement does not exist by default. When the administrator needs privilege, they request it - through a portal, a chat command, a ticket - stating the target, the task and the duration. An approver (a human, a policy, or a change record) grants it, and the grant is time-bounded and auto-revoked. On the target side the grant may be implemented as an ephemeral account, a temporary group membership, a sudo rule with an expiry, or a brokered session that carries the privilege without the user holding it. Zero standing privilege (ZSP) is the name for the end state in which no account holds a permanent privileged entitlement.

**What it prevents.** Standing privilege, directly and measurably: the entitlement's existence is reduced to the task's duration, and the ratio in section 2.4 improves by construction. It removes the dormant entitlement that an attacker who steals a token, a session cookie or a password can use at leisure, because at 04:00 the entitlement is not there. It produces a request record with a stated business reason, which is the artefact an auditor actually wants and which no other control produces.

**What it does not prevent.** A rubber stamp. Approval is a human decision and behaves like every human decision made at volume: within weeks the approver is clicking approve on reflex, and the request record becomes a record of a click rather than of a judgement. It does not prevent the approved privilege being used for something other than the approved task, because authorisation happens at grant time, not at action time; an hour of domain admin for "restart a service" is an hour of domain admin. It does not survive a slow queue: if the approval takes forty minutes and the incident needs four, the organisation will build a path around the control, and that path is usually the emergency account (section 10, mode 4).

**The operational cost.** Latency, paid by the operations team at the worst moment - the 03:00 page that waits for an approver in another time zone is the canonical cost, and it is not a hypothetical. Approval staffing: someone must be available, briefed, and authorised, around the clock, for every account class. Integration with change management: an approval that is not tied to a change record is a click, and tying it to one requires the ticketing system and the PAM tool to agree on identifiers. Design work: mapping tasks to entitlement bundles so that "restart the payment service" is a requestable thing rather than "domain admin for an hour" is a modelling exercise in the business's own vocabulary, and it is the part of the programme most often skipped.

### 3.4 Automated Credential Rotation

**The mechanism.** The vault generates or sets a new credential on the target through a connector, stores the new value, and - this is the part that separates rotation from password changing - **verifies** that the new credential works, then reconciles the systems that depend on it. Rotation can be scheduled (every N days), event-driven (on checkout, on suspicion, on staff departure) or both. For machine identities the modern form is not rotation at all but short-lived credentials issued on demand, which is a different mechanism with a different failure profile; [non_human_identity_guide.md](non_human_identity_guide.md) section 5 owns that distinction and this guide does not re-derive it.

**What it prevents.** The long-lived credential, and with it the value of an old one. A credential stolen from a backup tape, a log, a memory dump or a previous employer's notes is worthless after the rotation that follows. It bounds the blast radius of a suspected exposure: the answer to "the password leaked, what now" becomes "it was rotated at the last cycle and we can rotate now", rather than an incident. It also removes the stale credential that outlives its purpose - the account for the contractor who left, the application that was decommissioned, the appliance nobody has logged into since the project closed.

**What it does not prevent.** It does not prevent the credential being used **today**. Rotation is not revocation: between the compromise and the next cycle, a stolen credential works exactly as well as a legitimate one, and if the attacker is already inside, rotation may only inconvenience them once. It does not prevent breakage, and this is the control's dominant operational risk: applications that hard-code the credential in a configuration file, a scheduled task, a service definition, a connection pool, a JAR, a container image or a firmware default will fail at the moment the password changes, and they will fail silently and at night. And rotation without verification is **worse than no rotation**, because it produces a report that says the estate is covered while the credential on the target and the credential in the vault have diverged and nobody knows which one the application is using.

**The operational cost.** Dependency mapping: before you rotate an account you must know every consumer of it, and that knowledge usually does not exist in any single place. Connector coverage per platform, with the same long tail as vaulting. Maintenance windows and change records for the rotations that cannot be done live. An exception register for the accounts that genuinely cannot rotate, with an owner and a compensating control, because otherwise the exceptions are invisible and the percentage is a lie. And a second privileged credential - the account the vault itself uses to change passwords - which is the recursion at the heart of the discipline: the rotator must be protected, and it usually ends up as the most privileged account in the estate.

### 3.5 The Consolidated Table

| Control | Prevents | Does not prevent | Operational cost | The failure that makes it theatre |
|---|---|---|---|---|
| **Vaulting** | Credential sprawl; casual credential access; the absence of an inventory; the inability to rotate at all | An attacker inside an authorised session; compromise of the vault itself; a checked-out credential pasted into a script; direct use of a password that has not yet been rotated | Connectors per platform; application code changes; tier-0 availability, backup, DR and patching; network placement; named ownership and on-call | 100 per cent onboarding with an inventory that has no owner column, and a vault that is now the estate's richest target |
| **Session recording** | The deniable action; the absence of forensic evidence; the untraceable insider | The action itself; abuse that looks legitimate; sessions that bypass the proxy; anything at all if nobody reviews the recordings | Storage and retention; privacy consultation; personal data inside recordings; review labour with authority to act; proxy latency and protocol compatibility | Petabytes of recordings, zero findings ever raised, and a review process that consists of a retention policy |
| **Just-in-time elevation** | Standing privilege; the dormant entitlement; the missing request record with a business reason | A rubber-stamp approval; use of the granted privilege for a different task; the queue that pushes people to the emergency path | Human latency at 03:00; around-the-clock approver staffing; change-record integration; task-to-entitlement modelling | Median approval time of four seconds and a 99.8 per cent approval rate, reported as a success metric |
| **Rotation** | The long-lived credential; the value of a stolen or leaked one; the stale credential that outlives its purpose | Use of the credential today; breakage in anything that hard-codes it; divergence between vault and target when verification is skipped | Dependency mapping; connector coverage; maintenance windows; an exception register with owners; a protected rotation account | A rotation percentage that counts rotations with no verification, so the report is green and the application is running on the old password |

Read the "does not prevent" column as a set and the shape of the product category appears: no single control covers a population, no two controls cover the same failure, and each one's weakness is another one's job. That is the honest argument for a suite - not that one vendor does everything well, but that the four controls are complementary and the operational cost of integrating four separate tools is real. It is also the argument against buying the suite before the process, because four controls operated by nobody fail in four different ways.

---
## 4. The Vendor, Precisely Identified

### 4.1 The Corporate Facts, Dated

Corporate facts in this subject are unusually load-bearing, because the product's name changed twice in nine months and a reader who confuses the events will draw the wrong conclusions about support, contracts and roadmap. The table below is the sequence, with the status of each row.

| Date | Event | Status and source |
|---|---|---|
| 1999 | CyberArk founded in Israel | ⚠-knowledge: not verified at a primary source this pass. A dated secondary source (Nasdaq, 10 September 2014) describes the company as "founded in 1999" in its IPO coverage; no company filing was reached (section 15.1) |
| September 2014 | Initial public offering on Nasdaq; ordinary shares trade as **CYBR** | ⚠-knowledge: consistent with the September 2014 IPO coverage and the ticker in every subsequent release, but not re-verified against an SEC filing this pass |
| 30 July 2025 | Definitive agreement: Palo Alto Networks to acquire CyberArk | ✅ Palo Alto Networks press release, 30 July 2025, over PR Newswire |
| 11 February 2026 | Acquisition completed | ✅ Palo Alto Networks press release, 11 February 2026, over PR Newswire |
| 11 February 2026 | Intent announced to pursue a secondary listing on the Tel Aviv Stock Exchange and to adopt the **CYBR** ticker there; PANW remains the Nasdaq ticker | ✅ same release, stated as an intent, **not** as a completed listing |
| 12 May 2026 | Platform rebranded **Idira** and launched, generally available the same day | ✅ PR Newswire release, 12 May 2026 |

The transaction terms are stated identically in both releases: **"$45.00 in cash and 2.2005 shares of Palo Alto Networks common stock for each CyberArk ordinary share"** ✅. The 30 July 2025 release describes this as representing "an equity value of approximately $25 billion" and "a 26% premium to the unaffected 10-day average of the daily VWAPs of CyberArk as of Friday, July 25, 2025" ✅ - the footnote to that release gives the reference prices as $196.66 per Palo Alto Networks share and $380.17 per CyberArk share. The $25 billion is an **announced equity value at the date of the agreement**, not a price paid at completion and not a valuation of anything today; this guide prints it only in that form.

Two commitments in the completion release matter more to a customer than the price. The first: **"CyberArk's Identity Security solutions will continue to be available as a standalone platform"** ✅. The second: "Existing customers will experience no disruption and will benefit from an accelerated roadmap" ✅ - which is a forward-looking statement in a document that carries a forward-looking-statements disclaimer, and should be read as one.

One small honesty note about the record itself. The 11 February 2026 release was republished by PR Newswire with a correction notice reading that "changes have been made ... the complete, corrected release follows, with additional details at the end", and the correction recorded at the foot of the republished text is that "the link behind 'secure every identity' was incorrect" ✅. The correction was to a hyperlink, not to a fact, and the copy this guide quotes is identical in both versions. It is recorded here because a reader checking the primary source will encounter the notice and should know what it says.

### 4.2 What the Acquisition Changed

**Ownership.** A Nasdaq-listed Israeli-founded company became a wholly owned subsidiary of a larger, Nasdaq-listed platform company. The practical consequence is not the logo; it is that the product's roadmap is now set inside a portfolio, and the commercial relationship is with a vendor that also sells network security, cloud security and security operations. A buyer negotiating a PAM contract in 2027 is negotiating with the portfolio owner.

**Brand and product naming.** "CyberArk Identity Security Platform" became "Idira" on 12 May 2026. The release describes it as "the next-generation identity security platform to discover, control and govern all identities" and frames the launch as "a significant upgrade with expanded capabilities for existing CyberArk customers" ✅. The cyberark.com domain continued to serve the pages, with the company page titled "Idira - The Identity Security Platform - Palo Alto Networks" ✅ - so the brand changed and the URL did not.

**Packaging.** The 12 May 2026 release describes four migration paths for existing customers and states explicitly that several capabilities arrive "with the purchase of additional licenses" ✅. That is a commercial change with an operational consequence: "one platform" is a packaging and integration claim, and the modules are separately licensed (section 5.4).

**An explicitly agentic roadmap.** Both the July 2025 agreement release and the May 2026 launch put autonomous agents at the centre. The July 2025 release's news summary states that the combination "Will deliver Identity Security for agentic AI ... by providing foundational controls for this emerging class of privileged identities" ✅ - the phrase "this emerging class of privileged identities" is the vendor's own description of agents, and it is the most useful sentence in either document.

### 4.3 What It Did Not Change

An acquisition changes who owns the product. It does not change the problem the product addresses, and the discipline's economics survive the transaction intact.

- **The control problem is unchanged.** Credentials still sprawl, service accounts still have no owner, standing privilege still exists at 04:00, and a human still has to decide what "rotated" means for each account class. None of that is affected by which company signs the invoice.
- **The customer's implementation burden is unchanged.** Discovery, onboarding, connector coverage, dependency mapping, approver staffing, recording review and the exception register are the customer's work, and no acquisition reduces them. The vendor's platform consolidation may reduce the number of consoles; it does not reduce the number of decisions.
- **The rotation question is unchanged.** A vault still needs someone to decide, per account class, what rotation means: which accounts can rotate automatically, which need a maintenance window, which cannot rotate at all and what compensates, and how verification is proved. This is the single most consequential decision in a PAM programme and it is not a product feature.
- **Competition is unchanged.** The category contains other vendors, including BeyondTrust, Delinea, Wallix and Broadcom's PAM portfolio. They are named here only to record that this is not a single-vendor category; this guide does not compare them, rank them or assess their capabilities, and a reader should treat any ranking - including one implied by a vendor's own material - as a claim to be tested.
- **The standards position is unchanged.** No standards body blessed the acquisition, the rebrand or the product's architecture. The control frameworks named in section 1.2 predate all of it and will outlive it.

### 4.4 The Leadership and the Naming

Two names appear in the primary record and both are worth noting precisely, because each marks a different moment.

**Matt Cohen, CEO of CyberArk**, is quoted in the 11 February 2026 completion release ✅. That is the last primary document in this pass in which a CyberArk CEO speaks as the head of the acquired company.

**Peretz Regev, Chief Product and Technology Officer, Idira, Palo Alto Networks**, is quoted in the 12 May 2026 launch release ✅. The title is a portfolio title inside the parent company, not a CEO title, and it is the first primary document in this pass in which the product's leadership is named under the new brand.

The observation this guide will make is narrow and factual: **the person who spoke for the product changed between the completion and the rebrand, and the title changed with the brand.** What that implies about the organisation is not something this guide can establish from public material, and it does not speculate about departures, retirements or reorganisations. A reader who needs that history should read the filings, not a research guide.

On the naming itself, one detail is worth preserving because it explains why the old name will not go away. The completion release says Palo Alto Networks "plans to adopt the 'CYBR' ticker on the TASE, providing a world-class tribute to the brand CyberArk built while ensuring its identity remains a cornerstone of the global strategy" ✅. A company that keeps the old ticker as a tribute on a second exchange has decided that the old name has equity. That decision is consistent with what every practitioner sees in October 2026: the product is Idira, the contract says CyberArk, the URL says cyberark.com, and the runbook says "log into CyberArk".

---

## 5. The Product Surface, Decoded

### 5.1 The Capability Table

The product is easiest to understand as a set of capability categories, each of which maps onto a discipline control from section 3 or onto an adjacent discipline. The vendor's names are given because in several cases the name is the thing a buyer will be quoted, but the mapping column is what matters: a capability that does not map to a control is a feature, and a control with no capability mapped to it is a gap in the product.

| Capability category | What it does | The vendor's name for it | Discipline control it maps to |
|---|---|---|---|
| **Vaulting and rotation** | Stores system-level credentials encrypted; injects or brokers them; rotates them on a schedule with verification | Vaulting & Rotation | Vaulting (3.1) and rotation (3.4) |
| **Zero standing privilege and JIT** | Removes permanent entitlements; creates ephemeral privilege for the duration of a task and destroys it at the end | Zero Standing Privileges | JIT elevation (3.3) |
| **Infrastructure access** | Agentless access to AWS, Azure, Google Cloud and Kubernetes through native CLIs and consoles; session brokering and JIT entitlements without a VPN or bastion | Infrastructure Access | Vaulting and brokering (3.1), JIT (3.3) |
| **Endpoint privilege management** | Removes local administrator rights on Windows, macOS and Linux; replaces them with policy-based elevation | Endpoint Security | Least privilege at the endpoint; a precondition for vaulting coverage |
| **Session intelligence** | Brokers and isolates privileged sessions; produces AI-generated summaries that surface anomalous commands | Session Intelligence | Session recording and monitoring (3.2) |
| **Identity threat detection and response** | Analyses signals across the identity estate; terminates risky sessions or raises authentication requirements on suspicious lateral movement or "vault sweeping" | Identity Threat Detection & Response | Detection and response layered on 3.1-3.3; not a preventive control |
| **Secrets management for machine identities** | Stores and delivers application secrets; manages the lifecycle of credentials applications use | Secrets Management; Application Credentials Delivery; Unified Secrets Governance | Vaulting for the machine population (3.1); adjacent to the secrets-management discipline owned by [openbao_vs_vault_guide.md](openbao_vs_vault_guide.md) |
| **Agentic identity** | Registers agents, enriches them with context, brokers their credentials, kills their access, manages their lifecycle and audits their actions | Agent Registry, Agent Context, Agent Identity Broker, Agent Kill Switch, Agent Lifecycle Management, Agent Audit & Governance | An unsettled extension of all four controls; section 6 |
| **Identity governance** | Access certifications, lifecycle automation and policy for the workforce identity estate | Identity Governance | Adjacent: the review process that catches drift (2.3, d) |
| **Workforce password management** | Password vaulting, SSO and self-service reset for the general workforce | Workforce Password Management | Adjacent: credential hygiene outside the privileged population |
| **Vendor privileged access** | Browser-based, agentless, isolated access for third parties with JIT scoping and session recording | Vendor Privileged Access | Vaulting, JIT and recording applied to a population with no employment relationship |

### 5.2 The Human Pillar

The human half of the product is the half with a twenty-year history, and it is the most complete. The vendor's own PAM page describes it as unifying "vaulting, Zero Standing Privileges, and session isolation into a single enforcement experience" ✅, and the six capabilities it lists - zero standing privileges, infrastructure access, endpoint security, session intelligence, vaulting and rotation, and identity threat detection and response ✅ - are the four controls of section 3 plus endpoint control and detection.

Two details in that material are worth extracting because they are engineering, not marketing. The infrastructure-access capability is described as **"Native, agentless access to cloud and Kubernetes"** using "native CLI and consoles", with "agentless session brokering and JIT entitlements to eliminate VPN and bastion dependencies" ✅. That is a real architectural claim with a real consequence: agentless means no software on the target, which widens the range of targets you can cover and narrows the depth of control you have on each. The endpoint capability is described as removing local administrator rights "across Windows, macOS and Linux" and replacing them with "policy-based elevation" ✅ - which is the "first mile" of privilege, the local admin account on a laptop or server that is the usual foothold for lateral movement.

The session-intelligence capability is the one to read carefully, because it contains both the honest problem statement and the automation bet in the same sentence. The vendor says it will "broker and isolate privileged sessions across infrastructure and SaaS", that "AI-generated summaries surface anomalous commands in real time", and that this closes "the audit gap between authentication and action" ✅. Brokering and isolation are mature mechanisms. The AI summary is the interesting part: it is an attempt to solve section 3.2's review problem - the recording nobody watches - by automating the watching. Whether it works is not something this guide can establish, and section 13.1 files it under vendor promise rather than demonstrated capability.

### 5.3 The Machine and Agentic Pillars

The machine pillar is where the discipline's older vocabulary and the vendor's newer packaging diverge most. "Secrets management" in this portfolio covers the storage and delivery of application secrets, and the vendor's product navigation separately names "Unified Secrets Governance" and "Application Credentials Delivery" ✅. A reader should understand that this overlaps the secrets-management discipline owned by [openbao_vs_vault_guide.md](openbao_vs_vault_guide.md) without being identical to it: a secrets manager is built for applications fetching secrets at high volume with low ceremony, while a PAM vault is built for brokered human access with approval and recording. The convergence - one product doing both - is commercially attractive and architecturally awkward, because the two have different availability requirements, different throughput profiles and different blast radii if compromised.

The agentic pillar is the newest and is described in section 6. Structurally it is six capabilities: a registry that discovers agents across SaaS, cloud and developer environments and onboards them into a central repository; context that enriches each agent with ownership, purpose, status and permissions; an identity broker that enforces dynamic access policy and JIT access for agents; a kill switch that denies access when an agent "goes rogue"; lifecycle management from creation to decommission; and audit and governance that shows "what actions agents perform and on behalf of what user" ✅. One additional fact belongs here rather than in section 6: the vendor states that agent identity security is "now natively available with Prisma AIRS" for customers of that product ✅, which means the agentic capability is also sold through a second portfolio product - a packaging detail that matters when a buyer is trying to work out what they already own.

### 5.4 The Packaging Honesty

The 12 May 2026 release is unusually explicit about packaging, and a buyer should read it as a pricing document rather than a product document. It describes four paths for existing CyberArk customers ✅:

| Path | What the customer receives | What costs extra |
|---|---|---|
| **Traditional PAM (IT Standard)** | Discovery and user-experience improvements automatically; ability to add ZSP | ZSP, agentic and machine identity protections require "the purchase of additional licenses" |
| **Modern PAM (IT Enterprise and Dev)** | Discovery, ZSP and user-experience enhancements "at no additional cost" | Agentic and machine identity security are optional purchases |
| **Workforce Access** | User-experience improvements | Full ZSP, traditional PAM, and agentic and machine protections are upgrades |
| **Machine and AI Identity Security** | Customers with Secrets or Workload licenses can add traditional PAM and ZSP | Added "via new licenses" |

Three conclusions follow, and none of them is a criticism of the vendor - they are simply what the document says. First, **"one platform" is a packaging and integration claim, not a single product**: the platform is a set of separately licensed modules with a shared console, a shared data model and a shared policy engine to a greater or lesser degree. Second, **the migration path a customer is on determines what they get for free**, which means two customers with the same product name can have materially different capability. Third, **the release says the platform is generally available "today, with additional capabilities coming later this year"** ✅ - so a capability described in the launch material may not be available at the moment of purchase, and a buyer should require the specific capability, not the platform, to be demonstrated in their own environment before signing.

This guide does not assert version numbers, release numbers, feature-level availability dates or module SKUs beyond what those documents state. Where a capability is described as coming later, this guide says so; where the source is silent, this guide is silent.

---
## 6. The Agentic Identity Problem

### 6.1 What Changes When the Identity Is an Agent

The discipline in section 3 was built for two kinds of actor: a person who logs in, and a process that connects with a credential baked into its configuration. An autonomous agent is neither, and the difference is not a matter of degree.

**Intent is generated at runtime.** A person's privileged action is preceded by a decision the person can explain, and a script's action is preceded by code someone reviewed. An agent decides what to do while it is running, from a goal, a context and a model. The action may be correct, and it may not be predictable from the request. This is the property that makes "approve this access request" conceptually difficult: the approver is being asked to authorise an action that has not been determined yet.

**Scope is discovered at runtime.** A person knows which system they are logging into. An agent with tool access may determine mid-task that it needs a different system, a different API or a different credential, and may obtain it if the environment allows. Scope that is granted up front is therefore either too wide (to cover the unknown) or too narrow (to complete the task), and the agent has an incentive to widen it.

**Tools chain.** A single agent task can become a sequence of calls across systems, each individually authorised, whose combination accomplishes something none of the individual calls was scoped for. The composed action is the unit of risk, and it is not visible in any single authorisation decision.

**The session may be a plan rather than a login.** There may be no interactive session to record: no keystrokes, no terminal, no window. There is a plan, a series of tool invocations, and a result. The audit artefact the discipline relies on - the recorded session - may not exist in any recognisable form, and the boundary of "the session" (one task? one conversation? one agent lifetime?) is undefined.

### 6.2 The Vendor's Own Problem Statement

It is worth saying plainly that the vendor describes this problem better than most of its competitors and better than most analysts: the six challenge cards on its agentic-identity product page are a genuinely good problem statement, and they are worth reproducing as the agenda even for a reader who never buys the product ✅.

| Challenge | The vendor's formulation | Why it is the right question |
|---|---|---|
| **New hybrid identity class** | "AI agents blend human reasoning with machine scale and speed. This hybrid identity class without guard rails amplifies organizational risk." | It names the category correctly: an agent is not a service account with a better prompt, and it is not a person. |
| **Exploding attack surface** | "Agents require broad permissions across your sensitive resources, turning a single compromised agent into a master key for your entire environment." | It states the concentration risk that broad tool access creates. |
| **Privilege drift** | "Agents operate outside traditional human controls and are known to hallucinate and escalate privileges to achieve goals." | This is the most candid sentence in the vendor's agentic material. An agent that escalates to complete its task is doing exactly what it was optimised to do. |
| **Invisible workforce** | "The ease of creating AI agents leads to shadow AI. Without secure onboarding to a controlled registry, you're blind to risks of your agentic workforce." | It is the orphan problem of [non_human_identity_guide.md](non_human_identity_guide.md) section 4.2, arriving in a new population. |
| **Scale without scrutiny** | "As AI agents and MCP servers proliferate across your environment, lack of controls and governance creates blind spots and burdens your security team." | It names the model-context-protocol server layer as part of the estate, which most inventories still omit. |
| **Accountability gap** | "Without a clear view of agent activities, you can't prove who or what authorized an action, putting your organization at compliance risk." | It states the audit problem as an evidentiary one, which is what it is. |

What the vendor then offers is a registry, context enrichment, a broker, a kill switch, lifecycle management and audit ✅ - and section 6.3 asks the question the marketing does not: what do the four controls of section 3 actually mean when the actor is an agent?

### 6.3 What the Four Controls Mean for a Non-Human Actor

| Control | As applied to a person or a script | As it would have to apply to an agent | What breaks |
|---|---|---|---|
| **Vaulting** | Store the credential; check it out or inject it at connection time | The agent may need to obtain credentials **mid-plan**, for a system it did not know it needed when the task began. Vaulting becomes a brokered credential-issuance service with a policy engine that can answer a request the task did not declare in advance | A checkout model assumes the requester knows what it wants before it starts. An agent often does not, and a vault that refuses mid-plan requests will be worked around with a static credential |
| **Session recording** | Record keystrokes, commands and screen; review a sample | Record the **action trace**: which tool ran, with which arguments, against which resource, with what result, under which delegated authority - and preserve the reasoning context that produced the action if attribution is to mean anything | The "session" boundary is undefined, the volume is machine-scale, and a recording that captures a plan captures the model's reasoning, which raises data-protection and privilege questions nobody has resolved |
| **JIT elevation** | Request, approve, time-bound grant, auto-revoke | Per-task entitlement with a **delegation chain**: the human who initiated the task, the agent acting, the tools invoked, the resources touched. The approver may also be a machine, and a machine approving a machine is a policy question, not a workflow | The approval latency model collapses when the task takes seconds; the "who approved" question has no human answer; and the delegation chain is the artefact that must exist and currently has no standard form |
| **Rotation** | Replace the password on a schedule; verify | Short-lived tokens and workload identity: the credential is issued for the task and expires with it, which is a different discipline with different plumbing - attestation, token exchange, workload identity federation | Rotation as the discipline defines it (change a password on a target) is the wrong mechanism. The right mechanism is owned by [non_human_identity_guide.md](non_human_identity_guide.md) section 5 and section 8, and this guide does not re-derive it |

The table's fourth row is the important one, because it shows that the agentic problem is not "apply the four controls to agents". For one of the four - rotation - the answer is that the control is superseded by a different mechanism. For another - recording - the artefact has to be redefined. For the other two the mechanism survives but the assumptions behind it do not.

### 6.4 What Is Genuinely Unsettled

This guide's assessment is that the following are open questions, not implementation details, and that anyone claiming otherwise is selling something:

- **There is no settled definition of an agent's identity boundary.** Is the identity the agent software, the deployed instance, the conversation, the task, or the human who invoked it? Each answer implies a different registry, a different credential lifecycle and a different audit record. The vendor's answer - a registry of agents with an owner and a purpose ✅ - is one reasonable answer, and it is not yet an industry answer.
- **There is no agreed way to attribute an action to a principal when the agent chose the action.** The vendor's audit description is the most concrete public attempt: audits show "which human user initiated the agent, which AI agent identity acted, which tools ran and which resource was touched" ✅. That is a four-part chain, and it is a good start. It still does not answer the harder question: if the agent chose an action the human did not intend and could not have predicted, is the action the human's?
- **There is no standard for what an agent's least privilege means when its task is open-ended.** Least privilege is defined against a role or a function. An open-ended task has no enumerable function set, so "least privilege" degenerates either into a static allow-list that constrains the agent into uselessness or into a broad grant that violates the principle.
- **There is no accepted evidence format for an agent's audit trail.** A session recording is a de facto standard for a human session because the protocols are standard. An action trace across tools, models and resources has no schema, and without one, an auditor cannot sample it and a court cannot rely on it.
- **There is no consensus on whether an agent's credential should be issued to the agent or to the human who deployed it.** The second option is the failure mode of section 10, mode 6: the agent authenticates as a named engineer, and every action it takes is attributed to a person who did not take it. The first option requires the registry, the ownership model and the lifecycle that do not yet exist in most estates. Both options are in production right now, which is the definition of an unsettled practice.

### 6.5 The Standards Gap

The standards do not cover this, and saying so precisely is more useful than gesturing at "emerging regulation".

NIST SP 800-53 Rev. 5, the control catalogue most enterprise control frameworks are mapped to, was published in September 2020 with updates as of 10 December 2020 ✅ (verified at the NIST CSRC publication page this pass). Its account-management and least-privilege controls are written around **accounts and users**, and its vocabulary has no category for an autonomous agent that chooses its own actions - ⚠-knowledge at the level of control text, since the catalogue's control text was not read line by line this pass. NIST SP 800-207, the zero-trust architecture doctrine, frames its subject as users, assets and resources, and its authentication model assumes a subject that can be identified before a session begins - ⚠-knowledge, and the fuller treatment of what 800-207 does and does not say lives in [zero_trust_network_architecture_guide.md](zero_trust_network_architecture_guide.md).

The consequence is that **the most detailed public description of agent identity governance in October 2026 is vendor material**, and vendor material is written by the party that intends to sell the answer. That is not a reason to ignore it - the six challenge cards in section 6.2 are a better problem statement than the standards currently offer - but it is a reason to treat the capability names as a description of an intent rather than as a definition of a control. The containment half of the problem is owned by [agent_sandboxing_strategies_guide.md](ai_llm/agent_sandboxing_strategies_guide.md), the authority half by [agent_decision_policy_guide.md](ai_llm/agent_decision_policy_guide.md), and the platform context by [enterprise_agentic_platform_architecture_guide.md](ai_llm/enterprise_agentic_platform_architecture_guide.md); this guide's contribution is the narrow one - **an agent is a privileged identity, and the PAM controls do not yet fit it.**

---

## 7. The Deployment and Operating Reality

### 7.1 Discovery and the Credential Inventory

The first artefact of a PAM programme is not a vault. It is a list, and producing it is the hardest technical work in the programme.

You cannot vault what you cannot enumerate. Enumeration in a real estate is not one scan: it is a set of probes across directories, local accounts on every server and workstation, databases, network devices, hypervisors, cloud accounts, CI/CD systems, application configuration files, repository secrets, container images and the accounts embedded in vendor packages. It finds, with reliable regularity:

- accounts nobody knew existed, often local accounts created by an installer or a previous administrator;
- accounts embedded in scripts, cron jobs, scheduled tasks, service definitions, connection strings and application configuration files;
- accounts in CI/CD pipelines and deployment tooling, which are frequently the most privileged accounts in the estate and the least governed;
- accounts on appliances with no programmable interface, which cannot be rotated and must be handled as exceptions;
- accounts whose owner has left the organisation, or whose owner is a team that no longer exists, or whose owner is a vendor;
- and duplicates: the same credential used by several systems, which is the case that makes rotation dangerous because rotating it for one consumer breaks the others.

The inventory is the deliverable, and it needs three columns that a scan will not populate by itself: **owner**, **account class**, and **rotation method**. An inventory without an owner column is a list of strings (section 10, mode 5).

### 7.2 Onboarding in Waves

The order of onboarding is a risk decision, not a scheduling convenience. The defensible order is by account class, with the classes whose failure is loud and immediate first, and the classes whose failure is silent and nocturnal last.

1. **Human administrative accounts with interactive access** - the population the four controls were designed for, where failures are visible because a person is present and complains.
2. **Privileged business users and vendor/third-party accounts** - smaller, high-value, and the population where JIT and recording produce the clearest audit evidence.
3. **Application and service accounts for systems with an owner who understands the dependency** - the middle ground, where breakage is discoverable in a test cycle.
4. **Service accounts for scheduled and batch workloads** - last, and deliberately so: they break at night, they break silently, and the person who understands the dependency may have left.

The reason to put batch last is not that it is less important. It is that a failure there is discovered by a business user at 08:00 the next morning, which means every rotation error becomes an incident, and an early incident in a PAM programme is how programmes get descoped. Onboarding the noisy classes first builds the operational muscle - the connector testing, the verification step, the exception process - on failures that are cheap.

### 7.3 The Integration Surface

A PAM deployment is not a product installation; it is a set of integrations, and each one is a project dependency with an owner in another team.

| Integration | What it must do | What goes wrong |
|---|---|---|
| **Directory** | Authenticate administrators; resolve groups to entitlement policy; carry joiner-mover-leaver events | Group structure that does not match the actual administrative model, so policy is written against a fiction |
| **SIEM / detection** | Receive vault, session and entitlement events; correlate with other identity telemetry | Events arrive without the target or change-record context needed to be actionable, so nobody builds a rule |
| **Ticketing and change management** | Tie a JIT request and a session to a change record | The two systems identify the same change differently, so the link is manual or absent |
| **PAM-to-target connectors** | Change and verify credentials; broker sessions; enumerate accounts | Long-tail platforms with no API, which become permanent exceptions |
| **Network placement** | Put the vault and session proxies on the path between administrators and targets without becoming a single point of failure | The proxy becomes a latency dependency for every operational task, and its outage stops work |
| **Backup, DR and patching** | Treat the vault as the tier-0 system it is | The vault is backed up but never restore-tested, so a disaster becomes a credential-loss event |
| **Monitoring** | Watch the watcher: vault availability, failed rotations, orphaned accounts, approval anomalies | The vault has no owner with an on-call rota, so its failures are discovered by users |

The network and tier-0 points deserve emphasis because they are the ones buyers underweight. **The vault becomes a tier-0 system** the moment it holds the credentials to tier-0 systems: it must be protected at least as well as what it protects, which means its own administrative access is privileged access, its own backups contain the keys to the estate, and its availability is on the critical path of every operational task including the recovery from an incident. A PAM programme that does not plan for the vault's own disaster recovery has moved the single point of failure rather than removed it.

### 7.4 The Organisational Work

Four roles have to be filled by named people, and in most programmes none of them is:

- **The vault owner** - accountable for the platform, its availability, its patching, its DR and its roadmap. This is a platform-engineering role, not a security-governance role, and it needs an on-call rota.
- **The approver, per account class** - authorised to say yes and to say no, briefed on what the access actually permits, and available when the work happens. An approver who is measured on response time will approve everything (section 10, mode 3).
- **The reviewer** - samples session recordings on a defined cadence, with the authority to raise a finding and a workflow that closes it. A reviewer without authority produces observations; a reviewer with authority produces change.
- **The exception owner, per exception** - the person accountable for an account that cannot be vaulted, rotated or brokered, and for the compensating control. Unowned exceptions are how a 98 per cent coverage figure hides a 20 per cent risk.

These are day jobs held by people who already have one. The programmes that succeed write the roles into position descriptions and allocate time; the programmes that fail add them to an existing security engineer's list and call the result an operating model.

### 7.5 Before and After: The Estate

The honest way to describe maturity in this discipline is not a five-level model with invented levels. It is a before-and-after of observable states.

| Before | After |
|---|---|
| Standing administrative rights held permanently by a large population | Named accounts with JIT elevation, time-bounded and approved against a change record |
| Shared accounts: root, local administrator, database owner, vendor remote access | Brokered sessions where the credential is never disclosed, and shared accounts retired or explicitly excepted with a compensating control |
| Credentials in spreadsheets, wikis, scripts, configuration files and repository history | Credentials in a vault, delivered at connection time, with checkout records |
| Service accounts of unknown number and unknown ownership | An inventory with an owner, an account class and a rotation method per account |
| No session evidence; an incident review reconstructs intent from application logs | Session and action records attributable to a principal, samplable by an auditor |
| A rotation schedule that exists in a document | A rotation schedule with verification, an exception register and a reported coverage figure that excludes unverified rotations |
| Access reviews that approve what is already there | Reviews that remove entitlements, with the removals counted as the output |

### 7.6 The Two Timelines

**The product installs in weeks. The operating model takes a year, and the second is the deliverable.**

The installation timeline is real and short: a vault cluster, a connector or two, a directory integration, an initial set of accounts, a pilot population. A competent team can have sessions brokered and recordings landing within a quarter. What takes the year is everything in sections 7.1 to 7.4: the enumeration that finds the accounts nobody knew about, the dependency mapping that makes rotation safe, the wave plan that sequences by account class, the integration work in five other teams' systems, and the four roles that have to be staffed. An organisation that measures the programme by the installation will declare success in month three and discover in month twelve that it owns an inventory and not a habit.

---
## 8. The Banking and Regulated Angle

### 8.1 The Obligations That Push Toward PAM

No regulator mandates a product. Regulators mandate outcomes - restricted access, authenticated users, logged activity, protected availability - and an institution operating a few thousand servers, a mainframe, a core banking package and a Kubernetes platform will find that the only practical way to evidence those outcomes at scale is a control set resembling PAM. The obligations below are the ones that do the pushing. Each is marked with what this pass could and could not verify.

| Regime | The obligation, as this guide states it | Verification status |
|---|---|---|
| **PCI DSS v4.0.1** | Requirement 7: restrict access to system components and cardholder data by business need to know. Requirement 8: identify users and authenticate access to system components. Requirement 10: log and monitor all access to system components and cardholder data | ⚠-knowledge on the requirement numbering and titles. The Council's document library was extracted this pass and confirms that **PCI DSS v4.0.1** is the current standard ✅, but the standard's PDF would not extract (section 15.1), so the requirement text was not read |
| **DORA, Regulation (EU) 2022/2554** | ICT security and access-control obligations for financial entities, understood to be anchored in Article 9 ("Protection and prevention") | ⚠-knowledge. EUR-Lex would not extract this pass in either URL form tried (section 15.1), so the article number and title are **not verified** and no clause text is quoted. The application date is therefore omitted rather than asserted |
| **NIST SP 800-53 Rev. 5** | AC-6 least privilege; AC-2 account management; IA-5 authenticator management; AU-2 event logging and AU-12 audit record generation | ⚠-knowledge on the control titles. The publication itself was verified: **SP 800-53 Rev. 5, published September 2020, with updates as of 10 December 2020** ✅ (NIST CSRC) |
| **SOX section 404, through the ITGC frame** | General controls over program change, access to programs and data, and computer operations, tested by the external auditor | ⚠-knowledge. The ITGC framing is established practice; no clause text is quoted and no auditor guidance is cited |
| **ISO/IEC 27001:2022 Annex A 8.2** | "Privileged access rights" as a named control in the annex | ⚠-knowledge. The standard is paywalled and was not read this pass; the control's existence and number are asserted as established knowledge, and its text is not quoted |

Two things follow from that column, and both are methodological rather than legal. First, **the mapping from an obligation to a PAM control is an interpretation**, and an institution should have its own compliance function make it rather than accept a vendor's mapping table. Second, **the obligation is always about evidence**, not about capability: PCI DSS does not ask whether you have a vault, it asks you to show that access is restricted, that users are identified and that access is logged. That is why section 8.3 is a table of artefacts.

### 8.2 What an Auditor Actually Asks

The following are in the auditor's voice, because the fastest way to design a PAM programme is to answer them in advance. They are the questions this guide has seen asked, phrased as a request for an artefact rather than an assertion of a requirement.

1. **Show me the inventory.** Every privileged account, with an owner.
2. **Show me who approved this access, and when it expires.** For a sample of accounts, the approval and the expiry date.
3. **Show me the last access review, and what was removed.** Not who attended - what was taken away.
4. **Show me a session recording for this change window.** A specific change, a specific target, a specific time.
5. **Show me that this password was rotated, and that the application still works.** The rotation log plus evidence of a successful authentication after it.
6. **Show me the break-glass account's usage for the last twelve months.** Every use, with the incident or change it relates to.
7. **Show me the service accounts and their owners.** Including the ones with no interactive login.
8. **Show me that the person who approved is not the person who used it.** Segregation of duties, demonstrated on a sample.
9. **Show me the administrative accounts that were removed in the last quarter, and who had them.** The leaver path, evidenced.
10. **Show me the exceptions.** Accounts that are not vaulted, not rotated or not brokered, with the reason and the compensating control.
11. **Show me how you know the recording covers the session.** The evidence that the path is brokered, not that a recording exists.
12. **Show me the same evidence for the systems you outsourced.** The vendor's access, under your control, with the same artefacts.

The pattern in the list is worth naming: **every question is a sample request, and every sample request assumes the artefact exists at account level**, not at programme level. An institution that can produce a dashboard saying "98 per cent onboarded" and cannot produce the owner of account 4,117 has answered a different question from the one asked.

### 8.3 The Evidence Table

| Obligation | The artefact | Who produces it | How it is sampled |
|---|---|---|---|
| Access restricted by business need to know | Account inventory with owner and account class; entitlement-to-role mapping | PAM platform owner, with the business owner per account | Random sample across account classes, weighted to the accounts with the widest reach |
| Users identified and authenticated | Checkout and session records linking a named principal to an action; MFA enrolment evidence for the privileged population | PAM platform owner plus the identity provider team | Sample of sessions in the audit period, traced from principal to target |
| All access logged and monitored | Session recordings or action traces, retained per policy; alerting rules and their tuning record | Security operations, with the PAM owner | One session per sampled change, replayed against the change record |
| Segregation of duties | Approval records showing approver and requester as different principals; role design documentation | The approving function, not the platform team | Sample of approvals, checked against the requester identity |
| Timely removal of access | Joiner-mover-leaver evidence: terminations reconciled against privileged entitlements within a defined window | Identity governance, reconciled with HR | The full population of leavers in the period, not a sample - this one is cheap to test exhaustively |
| Credential hygiene | Rotation log with verification results; exception register with owners and compensating controls | PAM platform owner | Every account class represented; all exceptions reviewed |
| Third-party access | Vendor accounts, their JIT grants, their session records and their sponsorship | The business sponsor of each vendor relationship | Every active vendor account, since the population is small |
| Resilience of the control itself | Vault DR test results; vault availability reporting; the vault's own privileged access review | Platform engineering | Annual DR test evidence plus the last quarter's availability |

### 8.4 What This Guide Does Not Assert

This section names regimes and control objectives. It does not state a penalty, a fine amount, a deadline, an enforcement action or an application date, because none of those was verified at a primary source this pass and several would be stale or jurisdiction-specific by the time a reader acts on them. It does not assert that any institution is compliant or non-compliant, does not describe a regulator's finding against any institution, and does not present the auditor's questions as a legal requirement - they are the questions a control framework implies, in the voice of someone testing it. An institution should take its obligations from its own regulator, its own legal counsel and the published text of the standard, and use this section only to check that the artefacts it can produce match the questions it will be asked.

---

## 9. The Honest Assessment

### 9.1 What This Class of Product Solves

Stated without decoration, five things are genuinely solved by a competently operated PAM deployment, and each is a real problem that existed before the product did:

- **Credential sprawl**, by giving the estate one place where a credential lives and one record of who took it.
- **Unmanaged standing privilege**, by making privilege a time-bounded grant rather than a permanent property of an account.
- **The deniable action**, by producing a record that an actor was present and what they did.
- **The stale credential**, by rotating on a schedule and - where verification is real - proving the rotation took effect.
- **The inventory problem, partially.** A vault discovers and enumerates the accounts it is asked to hold, which is more than most estates had before. It is partial because the accounts nobody onboards are exactly the accounts nobody finds, and because discovery within a PAM tool is scoped to the platforms it can reach.

### 9.2 What It Displaces Rather Than Solves

The discipline's honest accounting is that most of what a PAM programme does is **move risk and work**, not eliminate them.

**It moves risk into the vault.** Concentration is the mechanism: one system now holds the credentials to everything, which is why the vendor's own detection material names "vault sweeping" as a behaviour to detect. The vault is a better-defended target than a spreadsheet, and it is a far more valuable one.

**It moves risk into the approval queue.** A rubber-stamped approval is a weaker control than a well-designed standing entitlement with monitoring, because it produces a record that says a human decided, when no human decided. The queue also relocates the failure to a place where it is invisible: nobody audits the approver's judgement until an incident.

**It converts a technical problem into a process problem that needs staffing.** Vault owner, approvers, reviewers, exception owners. This is the real cost, and it recurs annually, unlike the licence.

**It makes the vault a tier-0 dependency whose outage stops operations.** An estate that cannot log in to its servers because the vault is down has traded an availability risk for a security control, and the trade is usually correct and never free.

**And it can create a false sense of control.** The dashboard is green, the onboarding figure is 98 per cent, and the break-glass account is in daily use because the JIT path is slower than the incident. The evidence of the control's failure is inside the system that reports the control's success.

### 9.3 The Lock-In Question

The lock-in here is not the licence fee. It is the **process encoded in the product**: the account classes, the approval workflows, the entitlement bundles, the connector configurations, the session policies, the exception register and the reports an auditor has learned to accept. That is the asset that is expensive to move, because it is not data that can be exported - it is a set of decisions expressed as configuration, and re-expressing them in another product is a programme, not a migration.

Three specifics are worth stating plainly and without prediction. The vault is proprietary, the session-brokering protocol is proprietary, and the connectors are proprietary, so an exit is a re-implementation rather than a lift-and-shift. The cost of migration is **proportional to how much process you encoded**: an estate that used the product as a password store can leave in a quarter, and an estate that built its joiner-mover-leaver, its change approval and its audit evidence on top of it cannot. And the acquisition adds a **portfolio-level commercial relationship**: the buyer is now negotiating with a company that supplies several other parts of the estate, which changes leverage in both directions and is neither good nor bad in itself. A buyer should price the exit before signing - not because an exit is likely, but because the exercise reveals how much process is about to be encoded, and that is the number that determines the real commitment.

### 9.4 A Product Is Not a Control

**A PAM product is a control-enforcement mechanism, not a control.**

The control is the decision about who may reach what, made by a human with the authority to make it and the knowledge to make it well. The product enforces that decision, records that it was enforced, and makes the enforcement consistent across thousands of accounts. It cannot make the decision, because the decision requires knowing what the system does, what the business consequence of misuse is, and who is accountable - and none of those three facts is in the product.

This distinction is not semantic. It determines what a programme must produce to be real: a **policy** (who may reach what, under which conditions), an **owner** (who is accountable for each account class), a **schedule** (when rotation happens and how it is verified), a **review** (who looks at the evidence and what they do about it), and an **exception process** (what happens when the policy cannot be applied). A product can be procured in a quarter. Those five artefacts take a year, and they are the control. An organisation that buys the product and not the five artefacts has bought an enforcement mechanism with nothing to enforce, which is the most expensive way to store a password.

### 9.5 Where the Vendor's Own Material Complicates the Thesis

The thesis in section 1.3 says the tool and the process are the same thing, and that a vault nobody rotates is a database of passwords. The vendor's own material both supports and complicates it, and the honest reading is to say so.

**The support is direct.** The privileged-access product page states: **"Manual rotations and disconnected vaults for on-prem and cloud create security blind spots and operational friction that slow down digital transformation"** ✅. That is the vendor conceding the thesis in its own words: a vault that is not rotated and not connected is a blind spot, and the problem is operational rather than technical. The same page's framing - "Traditional PAM protects the privileged few yet, attackers go after everyone else" ✅ - concedes the second half: coverage of a narrow administrator population was never the same as control.

**The complication is the roadmap.** The vendor's answer to the review problem is more automation: AI-generated summaries that "surface anomalous commands" ✅, native ITDR that will "automatically terminate risky sessions or elevate authentication requirements" ✅, and an agent kill switch that denies access when an agent "goes rogue" ✅. Each of these is a bet that **the tool can supply the judgement the thesis says must come from outside it**. If an AI summary reliably identifies the anomalous command, then the reviewer the thesis demands is partly replaceable. If automatic termination is accurate, then the approval decision moves into the product. If a kill switch works, then the accountability gap narrows.

This guide does not resolve that disagreement, and says so: **it is unresolved, and it is the most important open question in the category.** The thesis holds with certainty for the estate as it exists in October 2026, where the judgement is human and frequently absent. Whether it holds in five years depends on whether the automation is accurate enough to be trusted with decisions that currently require an accountable person - and that is an empirical question that no vendor release answers, because a vendor release is not a measurement.

---
## 10. The Failure Modes

### 10.1 The Table

The seven failures below are the ones that recur, and each is recognisable from a metric. They are presented as symptom, cause and guardrail because the symptom is what a programme reports, the cause is what a programme did, and the guardrail is what a programme should have built instead.

| # | Failure | Symptom | Cause | Guardrail |
|---|---|---|---|---|
| 1 | **A vault nobody rotates** | 100 per cent onboarding, 0 per cent rotated in the last quarter | Rotation was scoped as a project - "we will rotate everything by Q3" - instead of a schedule with an owner per account class | Rotation with verification, a named owner per account class, and a reported coverage figure that excludes rotations that were not verified |
| 2 | **A session recording nobody reviews** | Petabytes of recordings, zero findings ever raised | Review was assigned to nobody, or to someone with no authority to act, or defined as "available on request" | Sampling on a defined cadence, by a named reviewer, with a finding workflow that closes and a count of findings per period |
| 3 | **Just-in-time that everyone approves** | Median approval time four seconds, 99.8 per cent approval rate, reported as a success | The approver is measured on response time rather than on risk, and the request form asks for a reason nobody reads | Approval tied to a change record, approval quality sampled, and the approval rate itself treated as an anomaly indicator rather than a KPI |
| 4 | **A break-glass account that is the normal path** | The emergency account is used weekly and never in an incident | The JIT path is slower, narrower or less capable than the emergency path, so the rational operator uses the emergency path | Break-glass use triggers a review within a defined window, and the everyday path is made faster than the emergency one |
| 5 | **An inventory that is a spreadsheet** | The "credential inventory" is a file whose owner column is empty | Discovery was a one-off scan, delivered as a report rather than as a feed | Discovery on a schedule, ownership as a required field, reconciliation against the directory, and an orphan count reported alongside coverage |
| 6 | **The agent whose credential was issued to a person** | An agent authenticates as a named engineer or a shared service account, so its actions are attributed to a human who did not take them | The agent was deployed before the identity question was asked, and the fastest way to make it work was to give it a credential that already existed | An agent registry with an accountable owner and its own credential, and a standing rule that no agent may inherit a human's identity |
| 7 | **The account with two owners is the same as no owner** | Every privileged account has an owner recorded, and half the owners are team distribution lists or "the platform team" | Ownership was treated as a field to populate rather than an accountability to assign, and a shared mailbox satisfies the validation rule | Ownership must resolve to a named individual accountable for the account's purpose; a distribution list is a finding, not an owner |

### 10.2 The Modes in Prose

The thread through the seven is that each one is a **metric measuring the wrong thing**. Onboarding measures the deployment. Approval rate measures the queue. Coverage measures the scan. None of them measures the control, and all of them are what a programme reports upward, because they are the numbers that go up.

Modes 1 and 5 are the same failure at different stages: the estate bought an inventory and called it a habit. Mode 1 is the credential half - the vault holds the credential and nothing changes it - and mode 5 is the account half - the list exists and nothing owns it. Both produce a green report and both leave the exposure exactly where it was.

Modes 3 and 4 are a pair, and they are causally linked. A rubber-stamped approval (mode 3) and a break-glass account in daily use (mode 4) are the same organisation's two responses to the same design fault: the everyday path was designed for the auditor rather than for the operator. When the JIT path is slower than the incident, the operator routes around it, and the routing is not misconduct - it is the only way to do the job. The guardrail for mode 4 is therefore not more control on the emergency account; it is a faster everyday path.

Mode 6 is the newest and the least understood, and it is the one this guide expects to see most often in 2027. The failure is invisible in every report, because the agent's actions appear as a legitimate engineer's actions in every log the estate has. The only detector is an inventory that says which principals are agents - which is why the registry is the load-bearing capability in the vendor's agentic material, and why buying the agentic module before the registry exists produces a dashboard with nothing true in it.

---

## 11. The Cymbal Bank Worked Example

**This section is fictional and illustrative.** Cymbal Bank is a hypothetical institution, it is the only institution in this guide, and every number below is invented to show a shape rather than to describe a real estate. No real regulator, incident, fine or enforcement action is referenced or implied.

### 11.1 The Estate

Cymbal Bank is a mid-sized retail and commercial bank. It runs a mainframe estate for deposits and cards, a core banking package on mid-range systems, a few thousand Windows and Linux servers, a Kubernetes platform for the digital channels, several hundred database instances across two vendors, a nightly batch estate that closes the ledger, and an outsourcing relationship under which a service provider operates part of the payments chain.

| Account class | Rough count | Who owns it | Current control |
|---|---|---|---|
| Human domain and server administrators | 420 | Infrastructure operations | Shared admin accounts; a password spreadsheet for the legacy half |
| Database administrators and application DB logins | 640 | Data platform, nominally; application teams in practice | Mixed: some vaulted, most in connection strings |
| Mainframe privileged identities | 90 | Mainframe operations | Vaulted, brokered, recorded - the best-controlled class in the bank |
| Service and batch accounts | 1,900 | Unclear for roughly two thirds | Partly vaulted, rarely rotated, exceptions undocumented |
| Kubernetes and CI/CD deploy identities | 260 | Digital engineering | Cluster-admin in the pipeline; long-lived tokens |
| Network, appliance and hypervisor accounts | 310 | Network and platform teams | Local accounts, manual rotation on a spreadsheet calendar |
| Third-party and vendor access | 180 | Business sponsors, nominally | Vendor remote-access tool with shared credentials |
| Break-glass and emergency accounts | 40 | Security operations | Vaulted, unmonitored in practice |
| **Total** | **3,840 known to the estate** | | |

### 11.2 The Discovery

The programme's first inventory returned **4,100 privileged accounts - against 3,840 the estate had catalogued and the 380 the original business case had assumed**, because the business case had counted administrators and the discovery counted administrators plus everything that could change something. Of the 4,100, **1,900 were service accounts and 1,180 of those had no identifiable owner**. **61 per cent of interactive administrative access was standing** - a permanent entitlement used, on the sample the team examined, for a median of under two hours a month. The vault was at **98 per cent onboarding** of the accounts it had been told about, and the rotation schedule covered **12 per cent** of them, of which roughly half had verification enabled.

That is the thesis in one estate: the bank had bought the inventory and not the habit. Every number in the previous paragraph is illustrative, and the shape is the point - onboarding is a deployment measure, rotation is an operating measure, and the two diverge by a factor that no dashboard shows because they are reported on different slides.

The discovery also produced two findings the business case had not anticipated. The CI/CD deploy identity could create a cluster-admin binding in the production Kubernetes platform, which made it the most privileged identity in the digital channel and the least governed. And the vendor remote-access account was shared between two outsourcing providers, so a session by either was attributable to neither.

### 11.3 The Decision the Discipline Changed

The audit committee's finding was predictable and the bank's first response was the predictable one: buy more modules. The proposal was an endpoint privilege module, an identity governance module and the agentic module, on the argument that coverage was the gap.

The discipline's response was different, and it changed two things. First, the bank **defined what "rotated" means per account class** - automated with verification for interactive accounts; automated with verification and a dependency register for application accounts; maintenance-window rotation for batch; documented exception with a compensating control for the appliance accounts that cannot rotate programmatically. Second, it **refused to report a rotation percentage that counted an unverified rotation**, which meant the first honest number the committee saw was 6 per cent rather than the 12 per cent in the earlier pack, and the second honest number, six months later, was 41 per cent.

The module purchase was deferred by two quarters and the money went into three things instead: an ownership campaign that resolved 1,180 unowned service accounts to named individuals (and decommissioned 240 that nobody could justify), a break-glass review that found the emergency accounts in weekly use because the JIT path required a ticket that took a day, and a rotation engineering effort on the batch estate that ran to a schedule rather than a project. None of those three was a product.

### 11.4 The Thesis, Restated

Cymbal Bank ended the year with a smaller inventory than it started with, a lower onboarding percentage than the slide deck had promised, and a control that an auditor could sample. What changed was not the tool. What changed was that the bank decided what the words in its own reports meant, and then made the estate match the words - which is the only sense in which a PAM programme is ever real. A vault that nobody rotates is a database of passwords, and the difference between that and a control is a decision about what "rotated" means, made by someone with the authority to enforce it.

---

## 12. The Anti-Patterns in Adopting This Class of Product

The twelve below are ordered roughly by how early they occur, and each is written as the decision that produces it rather than as a moral failing.

1. **Buying the vault before defining the account classes.** The account class determines the control, the rotation method and the owner. Buying first means the product's default taxonomy becomes the bank's taxonomy, and the defaults are organised around a human administrator.
2. **Treating onboarding percentage as the success metric.** Onboarding measures how much of the inventory the vault was told about is in the vault. It says nothing about rotation, ownership, review or exception coverage, and it is the number most likely to be green while the control is absent.
3. **Scoping rotation to "all accounts" in one wave.** A single wave guarantees a mass breakage event, which guarantees an exception list that is longer than the original scope, which guarantees a percentage that is meaningless.
4. **Letting the PAM programme be owned by the security team while the operations teams carry the breakage.** The team that experiences the 03:00 failure must be in the design, or the design will be routed around.
5. **Measuring JIT by tickets closed rather than by standing privilege removed.** A ticket count measures the queue's throughput; the control's output is the reduction in permanent entitlement, and only one of those two numbers can go down while the other goes up.
6. **Using session recording as a substitute for review.** Recording is evidence production. Review is the control. An estate that has installed the first and not the second has built a very expensive archive.
7. **Buying the agentic module before the agent inventory exists.** A registry capability with nothing registered produces a dashboard that is empty in a way that reads as safe. The inventory comes first, and it is produced by asking the platform and engineering teams, not by the product.
8. **Assuming platform consolidation removes the integration work.** One console is not one integration. The directory, the SIEM, the ticketing system, the change record and the connectors all still have to be wired, and a consolidated vendor means those wires terminate at one supplier rather than that they disappear.
9. **Designing the break-glass path last.** The emergency path is the one the organisation will use when everything else fails, which is the moment the design is least able to tolerate an afterthought.
10. **Letting the vendor's roadmap set the control model.** A roadmap is a statement about what will be sold next. A control model is a statement about what the institution will accept as evidence. The second must be written by the institution, or the first will quietly become it.
11. **Treating the acquisition as a reason to pause.** A rebrand shipped; no customer's deployment changed on the day. Pausing a programme for an ownership change that did not alter the product's function costs a year and buys nothing.
12. **Reporting a control as "implemented" when it is "licensed".** The most common and least forgivable, because it is the one that survives into an audit committee pack and is discovered, later, by someone sampling a session.

---
## 13. The State of Practice

### 13.1 True, and Promised

The distinction below is the one a buyer needs and the one vendor material never draws. The left column contains propositions that are true by construction or by mechanism - they would remain true if every vendor in the category disappeared tomorrow. The right column contains propositions that are true only if a product performs as described, and none of them has been independently measured in public material this pass.

| Demonstrably true | Vendor promise |
|---|---|
| Vaulting works as a credential-inventory and access-broker mechanism: a credential stored in one place, issued to one principal, with a checkout record, is a materially different exposure from the same credential in a spreadsheet | That privilege can be extended to every identity in the estate without a proportional increase in operational cost - the proposition that underlies "democratising" privileged access, which is a volume claim with no volume economics attached |
| Session brokering produces attributable records: if the path is brokered, the session is captured, and the record can be tied to a principal and a target | That AI summarisation closes the review gap - that a generated summary reliably surfaces the anomalous command in a population of sessions too large for humans to sample, at an accuracy that a reviewer can act on |
| Short-lived credentials reduce the value of a stolen one: a credential that expires in minutes is worth less to an attacker than one that expires in ninety days, regardless of how it was stolen | That agent identity can be governed at machine speed by the same platform, when the identity boundary, the attribution model and the least-privilege definition for an agent are all unresolved (section 6.4) |
| The four controls are named in published control frameworks - least privilege and account management in NIST SP 800-53, privileged access rights in ISO/IEC 27001:2022 Annex A 8.2 - so the discipline does not depend on a vendor's taxonomy for its legitimacy | That consolidation reduces risk rather than relocating it: one platform covering human, machine and agentic identities reduces the number of consoles and increases the concentration, and which effect dominates is an estate-specific question that no release note answers |
| Rotation, where it is verified, detects the divergence between the credential on the target and the credential in the vault, which is a real and common failure that no other control catches | That a single data model and policy engine spanning IAM, PAM and endpoint removes the seams attackers use - a claim about integration depth that a buyer can only test by asking which decisions are actually made in one place |

The asymmetry between the columns is the guide's practical advice in one line: **buy the left column, pilot the right one.** A contract that pays for demonstrated behaviour in the buyer's own estate, on the buyer's own account classes, is worth more than a capability list, and it is available from any vendor willing to be measured.

### 13.2 What to Watch Over the Next Year

These are things to watch, not predictions, and each has a specific observable:

- **Whether the rebrand changes contract terms or support paths.** The observable is the next renewal: whether the counterparty, the entitlement name or the support escalation path changes, and whether the standalone-platform commitment in the 11 February 2026 release survives contact with a portfolio sales motion.
- **Whether the agentic capabilities become generally available, and how they are licensed.** The 12 May 2026 release states that the platform is generally available "today, with additional capabilities coming later this year" ✅. The observable is which of the six agentic capabilities ship as included, which require a new licence, and whether the kill switch and the registry are separable - a registry without a broker is an inventory, and a broker without a registry is a policy with no subjects.
- **Whether the TASE dual-listing happens.** The 11 February 2026 release states an **intent** to pursue a secondary listing and to adopt the CYBR ticker there ✅. The observable is a listing, not an announcement; until then the ticker reference is a statement of intent.
- **Whether independent guidance on agent identity appears.** The observable is a standards body or a regulator publishing a definition of an agent's identity boundary, an attribution model, or an evidence format - anything that lets an auditor sample an agent's actions without relying on a vendor's schema. Until that exists, section 6.5's finding stands.
- **Whether the review problem is solved by automation or by staffing.** The observable is whether estates report findings from automated session analysis that they act on, or whether they report the same zero findings they reported when the review was manual.

---

## 14. The Claims Audit

### 14.1 The Table

Verified this pass = checked against the source named, extracted in **October 2026**, unless a source date is given. The convention follows the repository: ✅ verified this pass against a named primary source with its date; ⚠ vendor claim, approximate, or single secondary source; ⚠-knowledge well-established knowledge not re-verified this pass; ❌ could not be verified at source, recorded in section 15.1. The **Kind** column names what sort of source it is.

| # | Claim | Status | Source, with date and kind |
|---|---|---|---|
| 1 | Palo Alto Networks **completed** its acquisition of CyberArk on **11 February 2026**, on terms of $45.00 in cash and 2.2005 PANW shares per CyberArk ordinary share | ✅ | Palo Alto Networks press release, 11 February 2026, over PR Newswire - issuer press release (primary corporate disclosure) |
| 2 | "CyberArk's Identity Security solutions will continue to be available as a standalone platform" | ✅ | Same release, quoted verbatim - issuer press release |
| 3 | PANW **intends** to pursue a secondary listing on the TASE and to adopt the "CYBR" ticker there, remaining "PANW" on Nasdaq. Stated as an intent, **not** as a completed listing | ✅ | Same release, quoted - issuer press release. The listing itself is unverified (section 15.1) |
| 4 | The platform was rebranded **Idira** and unveiled on **12 May 2026**, generally available the same day | ✅ | "Palo Alto Networks Introduces Idira...", PR Newswire, 12 May 2026 - issuer press release |
| 5 | The acquisition **agreement** was announced **30 July 2025**, at an equity value of "approximately $25 billion" and a "26% premium" to the unaffected 10-day VWAP as of 25 July 2025, for $45.00 cash plus 2.2005 shares per share | ✅ | Palo Alto Networks press release, 30 July 2025 - issuer press release |
| 6 | "Machine identities now outnumber human identities by more than **80 to 1**" | ⚠ | Completion release, 11 February 2026 - vendor claim, no methodology published |
| 7 | "**109:1** machines outnumber human identities", cited to the "2026 Identity Security Landscape" report | ⚠ | Idira launch release, 12 May 2026, and cyberark.com/company stat block citing that report, May 2026 - vendor claim and vendor-commissioned survey |
| 8 | Non-human identities outnumber human financial-services employees "**96-to-1**" | ⚠ | [a16z_big_ideas_2026_guide.md](a16z_big_ideas_2026_guide.md), line 266 (Sean Neville's entry), claims-audit row at line 928 dated **11 December 2025**, and the audit row at line 1007 recording it as "unverified - no source given in the entry" - venture-firm essay, uncited |
| 9 | "**96%** of human users have access beyond what is required" | ⚠ | cyberark.com/company stat block, sourced to the "2026 Identity Security Landscape", Idira by Palo Alto Networks, May 2026, described there as the views of **2,930** global IT and security leaders - vendor-commissioned survey |
| 10 | "**99 out of 100** organizations have adopted AI agents" | ⚠ | Same stat block, same survey, May 2026 - vendor-commissioned survey |
| 11 | "Nearly **90%** of organizations have already suffered an identity-centric breach" | ⚠ | Completion release, 11 February 2026 - vendor claim |
| 12 | "**9 out of 10** organizations experiencing an identity-related breach in the past year" | ⚠ | Idira launch release, 12 May 2026 - vendor claim. Printed adjacent to row 11 deliberately: two breach figures from the same vendor three months apart, with different framings |
| 13 | "**61%** of privileged access requests are fulfilled with standing privilege rather than on-demand" | ⚠ | Idira launch release, 12 May 2026 - vendor claim, no methodology given |
| 14 | "**70%** of security breaches run through identity", and third parties account for "nearly **29%** of all identity breaches" | ⚠ | Idira privileged-access-management product page, undated, no source given - vendor product page |
| 15 | CyberArk was **founded in Israel in 1999** and listed on Nasdaq as **CYBR** in **September 2014** | ⚠-knowledge | Not verified at a primary source this pass. Consistent with the 30 July 2025 release describing CyberArk as "(NASDAQ: CYBR)" ✅ and with dated secondary coverage of the September 2014 IPO; the company's own filings were not reached (section 15.1) |
| 16 | **No vendor survey percentage in this guide is asserted as fact.** Every one appears with its date, its document and its status, and the guide draws no conclusion from any of them | ⚠ (by design) | This guide's own method; see rows 6-14 and section 14.2 |
| 17 | **No market-share, quadrant placement, customer count, competitor ranking, market-size or growth-rate figure is printed anywhere in this guide** | ⚠ (by design) | This guide's own method. Competitors are named in section 4.3 only to record that the category is not single-vendor, and are not compared on capability |
| 18 | The 11 February 2026 release was republished with a correction notice, and the correction recorded was to the hyperlink behind "secure every identity" | ✅ | PR Newswire republication of the release, carrying the notice "changes have been made ... the complete, corrected release follows" - wire-service republication |
| 19 | The July 2025 release described the combination as delivering "Identity Security for agentic AI ... by providing foundational controls for this emerging class of privileged identities" | ✅ | Agreement release, 30 July 2025, news summary - issuer press release, quoted |

**The honesty note.** Rows 1-5, 18 and 19 are the primary record: five press releases and a wire republication, each dated, each quoted rather than paraphrased, and each saying something narrower than the marketing built on it - an agreement is not a completion, an intent to list is not a listing, and a rebrand is not a migration. Rows 6-14 are every vendor statistic the guide touches, and **not one of them is used as evidence for an argument**; they appear because a reader will encounter them and needs to see the date, the document and the status in one place. Rows 16-17 record the two omissions that keep the guide honest: no survey number asserted, and no market position printed.

### 14.2 The Ratio Conflict, Printed Side by Side

| Ratio | Who published it | Date | What it measures | Verifiable? |
|---|---|---|---|---|
| **more than 80 to 1** | Palo Alto Networks, completion release | 11 February 2026 | "Machine identities" versus "human identities" - no definition of either term, no method | No. Vendor claim; no methodology published |
| **109:1** | Palo Alto Networks, Idira launch release, and cyberark.com/company citing the "2026 Identity Security Landscape" | 12 May 2026 (survey dated May 2026) | "Machine and AI identities" versus humans; the site version reads "machines outnumber human identities" | No. Vendor claim resting on a vendor-commissioned survey of 2,930 respondents, with no sample design or question wording published |
| **96-to-1** | Sean Neville, in a16z "Big Ideas 2026"; recorded in this repository's a16z guide | 11 December 2025 | "Non-human identities" versus "human financial-services employees" - a different denominator from either of the above | No. Uncited in the entry, and flagged as such in [a16z_big_ideas_2026_guide.md](a16z_big_ideas_2026_guide.md) line 1007 |

Three sentences, and then the guide stops drawing conclusions from ratios.

The same vendor published **80 to 1 in February 2026 and 109 to 1 in May 2026**, three months apart, in documents that describe the same market; a third party published **96 to 1** in December 2025 with no source at all. None of the three is independently verifiable, the denominators are probably different - machines, identities, non-human credentials, employees in one sector - and **not one of the sources states its method**. No conclusion in this guide is drawn from any ratio, and the three appear here only so that a reader who has seen one of them can see the other two beside it.

The discrepancy is itself the finding. An organisation that cannot state one number about the size of its own core market across three months is telling you that the measurement is not mature - and a number that moves 36 per cent in a quarter without a methodology change is not a measurement, it is a rhetorical device. The correct response for an institution sizing its own problem is to count its own accounts, which is what sections 7.1 and 11.2 are about.

---
## 15. What Could Not Be Verified, the Glossary and the Cross-References

### 15.1 What Could Not Be Verified

Each entry says what was attempted, not merely what is missing. Every one of these is a limit on this guide's evidence, and none has been filled in from memory.

- **The methodology behind every vendor statistic quoted in section 14.** Attempted: extraction of the vendor's company page and its product pages, plus the launch and completion releases. What is published is the percentage and, on the company page, the report title, month and respondent count (2,930). What is not published anywhere reachable is the sample design, the margin of error, the question wording, or the definitions behind "identity-centric breach", "identity-related breach" and "access beyond what is required". The three breach framings are not comparable and the sources do not claim they are.
- **The "2026 Identity Security Landscape" report itself.** Attempted: the citation on cyberark.com/company resolves to a report landing page whose download is form-gated ("Get the Report"). Only the summary statistics reproduced on the landing page were read. The report may contain the methodology the summary omits; this guide could not check.
- **CyberArk's founding year (1999) and IPO date (September 2014).** Attempted: a Businesswire anniversary release dated 24 September 2024 would not extract (all engines failed), and the Nasdaq IPO coverage available through search is dated secondary material rather than a filing. SEC EDGAR full-text search was not queried, and the Form 8-K, Form 6-K and Form S-4 referenced in the 30 July 2025 release were not retrieved. Both facts are therefore marked ⚠-knowledge, and no founder name or founding story is asserted anywhere in this guide.
- **Whether the TASE dual-listing has completed.** The 11 February 2026 release states an **intent** to pursue a secondary listing and to adopt the CYBR ticker there. No Tel Aviv Stock Exchange source was consulted, and the release gives no date. Section 4.1 and row 3 of section 14.1 record it as an intent, and section 13.2 lists the listing as an observable rather than as an event.
- **Whether any independent, non-vendor technical assessment of Idira's agentic capabilities exists.** Attempted: searches for third-party analysis returned nothing usable, and `web_search` was intermittently empty on this host during the pass - recorded as a **tool limitation, not as evidence of absence**. The only analyst voice in the material read is Will Townsend of LoneStar Advisory & Research, quoted inside the vendor's own 12 May 2026 release; this guide does not use that quote as verification, because an analyst quote reproduced in a vendor release is vendor material.
- **The ISO/IEC 27001:2022 Annex A 8.2 text.** The standard is paywalled. The existence and number of the "privileged access rights" control are asserted as ⚠-knowledge, and no clause text is quoted anywhere in this guide.
- **DORA, Regulation (EU) 2022/2554.** Attempted: EUR-Lex in two URL forms (`CELEX:32022R2554` and the ELI form). Both failed with scraper errors. The Article 9 reference in section 8.1 is therefore ⚠-knowledge, no clause text is quoted, and the application date is omitted rather than asserted.
- **PCI DSS v4.0.1 requirement text.** Attempted: the Council's document library (extracted successfully, confirming v4.0.1 as the current standard) and the direct PDF URL, which failed. The requirement numbers and titles in section 8.1 are ⚠-knowledge, not verified quotations.
- **NIST SP 800-53 Rev. 5 control text.** The publication page was verified (September 2020, updates as of 10 December 2020), but the catalogue's control text was not read line by line. The AC-2, AC-6, IA-5, AU-2 and AU-12 titles are ⚠-knowledge.
- **Any SEC filing detail on the transaction.** None was read. Every transaction fact in this guide comes from the two issuer press releases, which is sufficient for the terms and dates and insufficient for anything about consideration mechanics, regulatory conditions or integration accounting.
- **Market position of any kind.** No market-share, quadrant, customer-count, competitor-ranking or growth figure was verified, and none is printed. This is a deliberate omission as well as an evidence gap: the repository's convention is that a number without a dated primary source does not appear.

### 15.2 The Glossary

| Term | Definition |
|---|---|
| **Privileged account** | An account whose holder can change the configuration, data or availability of a system, or can alter the controls that would record them. Defined by reach, not by job title (section 2.1) |
| **Standing privilege** | Privilege that exists when nobody is using it: the permanent entitlement behind an occasional need. The object of the discipline (section 2.4) |
| **Vault** | An encrypted, access-controlled store for credentials, with a record of who retrieved which credential and when. Its value is the inventory and the broker, not the encryption |
| **Credential rotation** | Replacing a credential on a schedule or event, storing the new value centrally, and verifying that the new value works on the target. Rotation without verification is a report, not a control (section 3.4) |
| **Session recording** | Capturing a privileged session - keystrokes, commands, screen - through a proxy, for evidence and review. It documents action; it does not prevent it (section 3.2) |
| **Session brokering / PSM** | Interposing a proxy between the user and the target so the credential is never disclosed to the user and the session can be recorded, isolated and policy-checked. PSM is the common product abbreviation for privileged session management |
| **Just-in-time (JIT) elevation** | Granting privilege at the moment of need, for a bounded period, with an approval and an automatic expiry (section 3.3) |
| **Zero standing privilege (ZSP)** | The end state in which no account holds a permanent privileged entitlement; every privilege is created for a task and destroyed with it |
| **Break-glass / firecall account** | An emergency account kept for the case where the normal access path is unavailable. Its design intent is rare use; its failure mode is daily use (section 10, mode 4). The mainframe-specific treatment is in [zero_trust_mainframe_guide.md](zero_trust_mainframe_guide.md) section 6.2 |
| **Endpoint privilege management (EPM)** | Removing local administrator rights from workstations and servers and replacing them with policy-based elevation, so that the "first mile" of privilege is controlled |
| **Secrets management** | The storage and delivery of application secrets at machine volume, with low ceremony and high availability. Adjacent to PAM, not identical to it (section 5.3); the vendor comparison is [openbao_vs_vault_guide.md](openbao_vs_vault_guide.md) |
| **Identity threat detection and response (ITDR)** | Detection and response focused on identity signals - anomalous authentication, lateral movement, credential-store access - including terminating sessions. A detective control layered on the preventive ones |
| **Agentic identity** | The identity of an autonomous software agent that pursues a goal, chooses actions and invokes tools at runtime. Its governance is unsettled (section 6.4) |
| **Tier-0** | The set of systems whose compromise yields control of everything else - directories, the hypervisor layer, the backup infrastructure, and now the vault. Tier-0 systems must be protected at least as strongly as what they control |

### 15.3 The Cross-References

The five boundary owners, with the section that owns the adjacent subject:

- **[non_human_identity_guide.md](non_human_identity_guide.md)** - non-human identity as an asset class. Section 5 owns the identity-versus-secret distinction; section 8 owns the agent-as-non-human-identity framing; section 4.2 owns the orphan. This guide cites those conclusions and re-derives none of them.
- **[zero_trust_mainframe_guide.md](zero_trust_mainframe_guide.md)** - section 6, "Privileged Access, Vaulting and the Named Identity": 6.1 vaulting and brokering, 6.2 the firecall (emergency) identity, 6.3 session brokering to the platform, 6.4 the named identity. The mainframe half of section 7's wave plan lives there.
- **[openbao_vs_vault_guide.md](openbao_vs_vault_guide.md)** - the secrets-management vendor comparison, and the vault-versus-secrets-manager boundary this guide's section 5.3 refers to without settling.
- **[kmip_key_management_guide.md](kmip_key_management_guide.md)** - key management and the HSM/KMIP layer: how a vault's own keys are custodied, which this guide treats as out of scope.
- **[zero_trust_network_architecture_guide.md](zero_trust_network_architecture_guide.md)** - the zero-trust parent discipline, the CISA five-pillar model and NIST SP 800-207 doctrine; section 4.2 there is the identity-pillar row this guide's subject sits inside.

Adjacent guides cited in the text:

- **[audit_as_code_guide.md](audit_as_code_guide.md)** - evidence produced as code rather than assembled for an audit; the natural implementation of section 8.3's evidence table.
- **[a16z_big_ideas_2026_guide.md](a16z_big_ideas_2026_guide.md)** - the repository's record of the "96-to-1" ratio at line 266, with its claims-audit rows at lines 928 and 1007; cited in section 14.2 and nowhere relied upon.
- **[ai_llm/agent_decision_policy_guide.md](ai_llm/agent_decision_policy_guide.md)** - the authority half of the agent problem: who may decide, and under which posture.
- **[ai_llm/agent_sandboxing_strategies_guide.md](ai_llm/agent_sandboxing_strategies_guide.md)** - the containment half: the isolation spectrum and tool-execution patterns, which is where an agent's blast radius is actually bounded.
- **[ai_llm/enterprise_agentic_platform_architecture_guide.md](ai_llm/enterprise_agentic_platform_architecture_guide.md)** - section 6.2 identity integration: the platform context an agent registry has to fit into.
- **[../banking/enterprise_risk_management_guide.md](../banking/enterprise_risk_management_guide.md)** - the risk-and-control frame a PAM programme is an instance of, and where the accountability question in section 9.4 is treated at portfolio level.
- **[../banking/ai_risk_register_guide.md](../banking/ai_risk_register_guide.md)** - the register an agentic identity eventually has to appear in, with an owner and a rating.
- **[../banking/regtech_guide.md](../banking/regtech_guide.md)** - the obligation-to-evidence mapping that section 8.3 mirrors at the control level.

---

## 16. The Closing Summary

The argument of this guide runs in five moves.

**First, the discipline before the vendor.** Privileged access is defined by reach, not by seniority, and it covers three populations - human administrators, service accounts, and now agents - whose economics differ so much that the controls do not transfer between them. The four canonical controls are named in published frameworks that predate every product in the category, which is the strongest evidence available that the discipline is not a vendor invention.

**Second, the four controls and what each does not do.** Vaulting prevents sprawl and produces the inventory, and it makes the vault the richest target in the estate. Session recording prevents the deniable action, and it prevents nothing at all if nobody reviews it. Just-in-time elevation prevents standing privilege, and it converts a technical control into a queue with a human latency cost. Rotation prevents the long-lived credential, and unverified rotation is worse than none because it produces false assurance. Read the "does not prevent" column as a set and the product category explains itself: four complementary controls, four different failure modes, and no single one of them sufficient.

**Third, the three names and what the acquisition changed.** CyberArk Software Ltd. agreed to be acquired on 30 July 2025 at an announced equity value of approximately $25 billion, was acquired on 11 February 2026, and saw its platform renamed Idira on 12 May 2026 - three months from completion to rebrand. The company, the brand and the contract now carry three different names, and the domain still says cyberark.com. What changed was ownership, naming, packaging and an explicitly agentic roadmap. What did not change was the control problem, the customer's implementation burden, the question of what rotation means per account class, or the existence of competitors.

**Fourth, the agent as the population the discipline was not built for.** An agent generates intent at runtime, discovers scope at runtime, chains tools, and may have no session to record. Of the four controls, one - rotation - is superseded by a different mechanism, one - recording - has to be redefined as an action trace, and two survive with their assumptions broken. The vendor describes this problem better than the standards do, and the standards do not describe it at all: the most detailed public account of agent identity governance in October 2026 is vendor material. The unresolved questions - the identity boundary, the attribution model, least privilege for an open-ended task, the evidence format, and whether the credential belongs to the agent or to its deployer - are not implementation details, and the failure mode that follows from getting the last one wrong is an agent authenticated as a named engineer.

**Fifth, the thesis tested.** The vendor's own product page concedes that "manual rotations and disconnected vaults" create blind spots, which is the thesis in the vendor's words. The vendor's roadmap then answers with more automation - AI-summarised sessions, automatic termination, agent kill switches - which is a bet that the tool can supply the judgement the thesis says must come from outside it. That disagreement is unresolved, and this guide leaves it unresolved on purpose, because the evidence that would settle it does not exist in public material.

The boundary has been held throughout. This guide has not re-derived non-human identity as an asset class, the secrets-management comparison, mainframe privileged access or key management; it has cited those siblings and stopped. It has not printed a market position, a customer count, a ranking or a survey percentage dressed as a fact. And it has not claimed that any of the vendor's statistics measure what they appear to measure, because none of them publishes a method.

What would change the conclusion is observable rather than rhetorical: a standards body defining an agent's identity boundary, an independent assessment of the agentic capabilities, or an estate publishing rotation coverage that survives an auditor's sample. Until one of those exists, the discipline's oldest rule holds - the control is the decision, and the product only enforces it.

What a practitioner should take away is narrower than a product decision and more useful than one:

- Decide what "rotated" means for each account class, and refuse to report a number that counts a rotation nobody verified.
- Name an owner for every privileged account, and treat a distribution list as a finding rather than an owner.
- Make the everyday path faster than the emergency path, or the emergency path becomes the everyday one.
- Review the recordings, with a named reviewer who has the authority to raise a finding and a workflow that closes it.
- Report standing privilege removed rather than tickets closed, and ownership resolved rather than accounts onboarded.
- When an agent arrives, register it, give it its own credential, and never let it inherit a person's.

None of that is in the licence. All of it is the control - and an institution that does those six things with a modest product will outperform one that does none of them with the platform.

**Privileged access is the one control where the tool and the process are the same thing: a vault that nobody rotates is a database of passwords.**
