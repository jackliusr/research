# AI Security Architect: A Skill-Gap Analysis - the repository as evidence of knowledge, and the accountability it cannot show

> **For:** Jack Liu Shurui - Solution Architect, Cymbal Bank, Singapore
> **By:** Jack Liu Shurui, Solution Architect
> **Scope:** the seven requirement areas of an AI Security Architect role, tested against this repository as evidence of documented knowledge and against the profile as evidence of accountability
> **Date:** 12 November 2026
> **Series:** fifth role-analysis guide in `personal/`; predecessors `data_architect_skillgaps_guide.md`, `skill_gaps_enterprise_architect_guide.md`, `senior_data_platform_architect_guide.md`, `agent_runtime_engineer_skill_gaps_guide.md`

### Contents

1. How this guide relates to the repository, the target role and the decoder
2. The evidence audit - the repository read as evidence
3. What the role actually owns, and what it does not
4. The requirement-by-requirement map
5. The heatmap
6. The organisational-verb class
7. The real gaps, ranked
8. What genuinely helps
9. The credential question
10. The interview and the assessment
11. A staged plan
12. The anti-patterns of self-assessment in this genre
13. The one-page verdict
14. The claims audit
15. What Could Not Be Verified, the glossary and the cross-references
16. The closing summary

---

## 1. How this guide relates to the repository, the target role and the decoder

### 1.1 The series convention

This guide keeps the convention set by its predecessors. A blockquote header addressed to the subject; a numbered table of contents; an explicit statement of what was measured, when, and with which command; an evidence-graded treatment of every external claim; a glossary; a claims audit with rejected rows; and a closing section that says plainly where the sources are silent. The form exists because a skill-gap guide that cannot be audited is a horoscope with tables in it.

### 1.2 What this guide does that its predecessors did not

The four predecessors compared architecture roles to one another or moved the subject out of the track entirely, and asked how much of the target role the profile already evidenced. They measured **domain knowledge** and produced a percentage and a study plan. That was the right instrument for a lateral move, and the right instrument for a move from design to delivery.

This guide asks a different question, because the target role is a different kind of job. An AI Security Architect is not hired for what they can explain; they are hired for what they are answerable for. The repository is an unusually good instrument for the first question - what does this person know - and structurally incapable of answering the second. So the guide splits into two audits that use the same body of evidence for opposite purposes:

- Section 2 reads 686 markdown files as a corpus and counts it. This is the strongest evidence of AI-security knowledge that exists in the profile, and the counts are real.
- Sections 5 to 7 read the same corpus for the vocabulary of **authority** - strategy owned, board chaired, programme run, certification held - and find almost none of it.

The tension between those two readings is the guide's subject. Where the second audit contradicts the thesis, the guide says so in the same sentence rather than smoothing it.

### 1.3 The decoder: what the job description's verbs mean operationally

Job descriptions in security functions are written in a vocabulary that sounds like knowledge and functions as mandate. The table below is the decoder used throughout. Every row is a claim about the **object** the verb produces, not about the skill it implies.

| JD phrase | What it produces in a real organisation | What proves it exists |
|:--|:--|:--|
| develop and maintain the strategy | a versioned document with a named owner and a review cycle | the document, its owner field, its change log |
| establish minimum security standards | a baseline with a compliance test and an exception route | the standard, the test, the exception register |
| review and approve architectures | a decision right with a veto and a record | a minute, a conditions list, a signature |
| establish and manage a red-team programme | a calendar, participants, a budget, a findings register | the register, the remediation log, the vendor contract |
| act as primary SME | a named designation inside a governance structure | a terms-of-reference document naming the role |
| provide regular reporting to senior leadership | a cadence with a named audience and consequences | the pack, the forum, the minutes |
| collaborate with external partners | a relationship the institution holds | a contract, an NDA, a joint methodology |
| support regional security organisations | a mandate that crosses legal entities | an org chart, a delegation of authority |

Read the right-hand column. Not one of those artefacts is a document a person can write alone in a repository. That is the whole argument of this guide in one table.

### 1.4 Three kinds of gap, defined before they are used

The genre's characteristic failure is to collapse three different things into one word. This guide keeps them separate from the first page, and every verdict in section 4 carries one of these labels.

- **Knowledge gap** - you do not know the thing. Remedy: study, and the repository proves this subject can close these faster than most.
- **Experience gap** - you know the thing but have not done it. Remedy: doing it once, under supervision, in a real environment. Study does not close it; a first repetition does.
- **Accountability gap** - you have done the thing, or could, but have never carried the consequence or held the authority. Remedy: someone with authority delegates it to you. No amount of study closes it, because the missing object is other people's obligation to accept your decision.

Conflating them is not a stylistic slip. It converts an accountability gap into a reading list, and a reading list is the only remedy a document is able to prescribe. A guide that does this is comfortable to read and useless to act on. This one pays the cost of being uncomfortable in sections 6 and 7 instead.

### 1.5 The integrity rule

Nothing below claims security work was performed. Where the profile is silent, the guide writes "the profile is silent". Where the profile is ambiguous, it writes "ambiguous" and names both readings. Where a claim would be useful to the subject but is not supported, it appears in section 14 as a REJECTED row with the reason stated.

---

## 2. The evidence audit - the repository read as evidence

### 2.1 Method

Measured 12 November 2026 from the repository root, 686 markdown files in scope. Every count below is a **file count**, not an occurrence count: the command form is `grep -rlEi "<term>" --include='*.md' . | wc -l`, so a file containing a term forty times contributes one. This matters for interpretation - a count of 227 for `guardrail` means 227 files discuss guardrails somewhere, not that guardrails are treated in depth 227 times - and it is the reason the thin numbers in 2.4 are more informative than the strong ones in 2.3.

The figures below were taken in one pass on 12 November 2026 with the command form above, and are used here exactly as measured - not rounded, not extrapolated, and not refreshed from a later scan.

### 2.2 Directory scale

| Directory | .md files | Note |
|:--|--:|:--|
| repository total | 686 | in scope for every count below |
| `technology/` | 425 | the engineering corpus |
| `banking/` | 165 | the domain corpus |
| `management/` | 57 | the organisational corpus |
| `personal/` | 9 | this series |
| `technology/ai_llm/` | 103 | the AI core |
| `technology/ai_llm/rag/` | 20 | a fifth of the AI core is retrieval |

### 2.3 The strong vocabulary - what the corpus genuinely covers

| Term | Files | Term | Files |
|:--|--:|:--|--:|
| guardrail | 227 | AI agent | 124 |
| human in the loop | 111 | attestation, SOC 2 | 105 |
| architecture review, design review | 103 | model risk | 103 |
| PII | 101 | risk assessment | 95 |
| model risk management, SR 11-7 | 88 | ai governance | 84 |
| incident response | 84 | sensitive data | 69 |
| third-party (ai or risk) | 67 | EU AI Act | 55 |
| prompt injection | 55 | threat model | 54 |
| third-party risk | 54 | security operations, SecOps | 47 |
| steering committee, review board | 47 | three lines | 45 |
| control library, control framework | 40 | responsible AI | 40 |
| vendor risk | 37 | ai risk | 35 |
| red team | 35 | penetration test, pen test | 32 |
| OWASP | 32 | jailbreak | 30 |
| model card | 30 | zero trust | 27 |
| risk taxonomy | 25 | NIST AI RMF | 23 |
| GRC | 17 | ISO/IEC 42001 | 16 |
| MITRE ATLAS, ATT&CK | 11 | | |

### 2.4 The thin vocabulary - the numbers that decide this analysis

These are the counts that matter, and they are the ones a reader should sit with. Each is a term the job description uses as a **noun the role owns**.

| Term | Files | Reading |
|:--|--:|:--|
| ai security (as a phrase) | 16 | the discipline is rarely named as such |
| data poisoning | 7 | treated inside red-team guides, not as a control family |
| model extraction | 6 | same |
| model manipulation | 1 | the compound pattern returns 30 only via bare "poisoning" |
| model hosting | 8 | no architectural treatment of the hosting environment |
| inference security | 0 | absent as a phrase |
| agent abuse, agent hijack | 0 | absent as phrases |
| security awareness, security training | 4 | none of them describes a programme |
| secure SDLC | 7 | present as method, not as an operated lifecycle |
| security lifecycle | 2 | effectively absent |
| AI or model supply chain | 7 | against 138 for bare "supply chain" |
| security strategy | 1 | one incidental line, not an enterprise strategy |
| design authority | 7 | the body is named; the seat is not evidenced |
| architecture governance | 13 | described, not operated |
| control testing, control assurance | 8 | the testing half of control ownership is thin |
| AI assurance | 4 | thin |
| AI inventory, AI register, AI use case inventory | 11 | the artefact a strategy lands in is thin |
| tabletop, table-top | 12 | exercise design is thin |
| bug bounty | 9 | adjacent to red teaming, not the same thing |
| capability maturity, SAMM | 10 | maturity measurement is thin |

### 2.5 Reproducibility - dispatch figures that did not survive re-measurement

This guide was commissioned with a set of counts supplied in advance. Seven of them did not reproduce when measured on 12 November 2026. Printing them is not housekeeping; it is the difference between a measurement and a number, and the failure modes are instructive.

| Dispatched pattern | Dispatched | Re-measured | Why it did not reproduce |
|:--|--:|--:|:--|
| secure SDLC, security lifecycle | 53 | 9 | components measured separately: 7 + 2 |
| AI or model supply chain | 97 | 7 | ungrouped pattern; bare "supply chain" is 138 across all domains |
| agent abuse, agent hijack | 5 | 0 | neither phrase appears in any file |
| model hosting, inference security | 23 | 8 | all eight are "model hosting"; "inference security" is zero |
| RAG.*security | 147 | 113 | a loose cross-paragraph regex; matches proximity, not coverage |
| AI.*SecOps | 23 | 19 | much of it incidental co-occurrence in unrelated paragraphs |
| model manipulation, poisoning | 6 | 30 | bare "poisoning" carries the count; the first term alone is 1 |

The pattern in the failures is consistent: ungrouped alternations, unbounded `.*` bridges, and terms whose common substring belongs to an unrelated word. That last failure has a security-specific form worth naming, because a similar exercise was bitten by it. A case-insensitive search for the certification acronym **CISM** returns 113 files - only because it matches "criticism". With word boundaries applied, the security-certification acronyms (CISSP, CISM, CRISC, CISA, CCSP, GIAC) appear in **20 files**, and every one of them discusses certification as a topic or recommends obtaining one. None claims the author holds one. A count that matches a substring inside an unrelated word is not evidence of anything except the regex.

### 2.6 The eight dedicated AI-security guides

These are the corpus's centre of gravity. They are cited by section throughout this guide, because a filename proves a document exists and a section proves a claim was made.

| Guide | Lines | Sections that carry security weight |
|:--|--:|:--|
| `banking/ai_risk_register_guide.md` | 816 | s3 failure-mode inventory; s4 controls that do the work; s5 evidence and the audit question; s7 three lines; s8 third-party entries; s9 agentic extension |
| `technology/ai_verify_guide.md` | 706 | s4 the eleven governance principles mapping; s5 technical tests |
| `technology/ai_llm/prompt_injection_guide.md` | 679 | s3 types; s4 vectors; s6 detection and prevention; s7 defence architectures; s8 in RAG; s9 in agentic systems |
| `technology/ai_llm/ai_governance_bias_redteaming_guide.md` | 763 | s4 bias measurement and algorithmic audits; s6 LLM bias in practice |
| `technology/ai_llm/ai_red_teaming_guide.md` | 709 | s2 attack taxonomy family map; s5 poisoning, extraction, inversion; s6 agentic and RAG surfaces; s8 the process; s9 tooling |
| `technology/ai_llm/ai_governance_framework_guide.md` | 707 | s2 global instruments; s3 Singapore and MAS FEAT; s5 operating model and role positioning; s6 model-risk integration; s7 lifecycle gates; s9 regulatory mapping |
| `technology/ai_llm/ai_verify_toolkit_guide.md` | 900 | s2 architecture; s3 test engine; s4 algorithm library and what it omits |
| `technology/ai_llm/frontier_ai_risk_guide.md` | 865 | s2 the contested taxonomy; s3 the frontier safety frameworks; s4 dangerous-capability evaluations; s5 the governance layer |

### 2.7 The adjacent guides the measurement found

Five further files carry material the eight do not, and they strengthen the audit rather than diluting it - they show the same knowledge applied in a different frame.

| Guide | Lines | Sections worth citing |
|:--|--:|:--|
| `technology/adversarial_ml_attacks_guide.md` | 1058 | s3 threat taxonomy and frameworks; s6 attack types; s7 risk assessment for ML systems; s9 defences; s11 operational ML security |
| `technology/llm_development_risks_security_guide.md` | 974 | one section per OWASP LLM01-LLM10; s14 defence-in-depth; s15 secure development lifecycle; s16 testing and red teaming; s19 governance |
| `technology/security_by_design_guide.md` | 720 | s2 threat modelling; s5 security testing; s6 DevSecOps; s9 the security-by-design review worked example |
| `banking/ai_genai_banking_compliance_guide.md` | 717 | s3 governance requirements; s4 the GenAI risk taxonomy; s5 privacy; s6 security requirements; s8 the gap assessment worked example |
| `technology/cybersecurity_guide.md` | 670 | the security organisation; s5.5 awareness as a programme; s6 the frameworks map; s9 a security-programme worked example |

One honest note on the last row. `technology/cybersecurity_guide.md` s5.5 does treat security awareness as a programme, and it does so with the right vocabulary - cadence, audience, measurement. It is a section in a guide about security organisations, written by someone describing how such programmes are built. It is not a programme the author ran, and the difference between those two things is the difference this entire guide is about.

### 2.8 The shape the numbers show

Two vocabularies, one corpus, and they do not have the same density.

The vocabulary of AI-security **knowledge** is dense and, in places, unusual for a solution architect: 227 files touch guardrails, 124 touch AI agents, 111 touch human-in-the-loop, 103 touch model risk, 95 touch risk assessment, 88 touch SR 11-7 or model risk management, 84 touch AI governance, 55 touch the EU AI Act, 55 touch prompt injection, 54 touch threat modelling. Those are not the counts of someone who read a summary. They are the counts of someone who wrote the corpus.

The vocabulary of AI-security **authority** is close to absent. `security strategy` appears in 1 file and it is an incidental line in `technology/secure_red_hat_openshift_guide.md`, not an enterprise security strategy. `agent abuse` and `agent hijack` appear in 0. `security awareness` and `security training` appear in 4, and none describes a programme with a cadence and an audience. `design authority` appears in 7, always as a body being described rather than a seat being held. `control testing` and `control assurance` appear in 8 - the half of control ownership that requires someone to be answerable for whether the control worked.

That asymmetry is the finding. It is not manufactured by the choice of terms: the terms on the thin list are the ones the job description uses as nouns the role owns, and they are thin precisely because owning them produces documents this repository does not contain - minutes, registers, exception logs, delegation letters.

## 3. What the role actually owns, and what it does not

Before testing requirements one by one, it is worth being precise about the object of the role, because the job description is longer than its scope. Read as a whole it describes a **second-line security function** with a design remit: the role owns the security **position** on AI, not the AI systems themselves.

### 3.1 What is owned

| Owned object | What ownership means here |
|:--|:--|
| the enterprise AI security strategy | it is that person's document; it lapses if they leave |
| the AI cybersecurity risk taxonomy and control library | jointly with Information Security GRC, but the AI content is theirs |
| AI security standards, guidelines, procedures | the baseline every project is measured against |
| the AI red-teaming programme | the calendar, the vendors, the findings register |
| the pre-production assurance gate | a recommendation to deploy, or a condition, or a refusal |
| the AI security awareness programme | the modules, the audience, the measurement |
| the SME designation | named in governance terms of reference, not self-appointed |

### 3.2 What is shared

| Shared object | With whom | What sharing costs |
|:--|:--|:--|
| risk taxonomy and control library | Information Security GRC | the taxonomy must fit the enterprise risk language, not the AI one |
| AI architecture principles | Enterprise Architecture | principles that conflict with the EA reference architecture are dead on arrival |
| threat intelligence to control translation | Cyber Defense, threat intel | a control proposed without detection coverage is a wish |
| legal and regulatory interpretation | Legal, Compliance | the role cites the obligation; it does not interpret it |
| third-party AI assessment | Procurement, vendor risk | the security questions are the role's; the contract lever is not |
| defensive AI deployment | SecOps | the role assesses; SecOps operates |

### 3.3 What a reader might assume is in scope, and is not

- **Not the AI product owner.** The role does not decide what the AI system does, which use case is funded, or what the model's accuracy target is. It decides whether the system may be deployed given its security properties. Those are different decision rights and conflating them is how security architects become blockers rather than advisers.
- **Not the model-risk function.** Model risk management (SR 11-7 lineage, 88 files in this corpus) owns model validation. The AI security role consumes validation output and adds the adversarial and supply-chain view. Where the two overlap - model manipulation, extraction - the boundary is negotiated, not given.
- **Not the SOC.** `security operations` appears in 47 files here, and none of them describes the author running a detection engineering queue.
- **Not the data protection officer.** PII appears in 101 files and sensitive data in 69, but the accountability for lawful processing sits elsewhere. The role contributes the technical control, not the legal basis.
- **Not the platform engineering team.** Model hosting and inference infrastructure belong to whoever runs the platform. The role sets the security requirements for that environment. Section 4.3 shows this is one of the thinnest parts of the corpus, and it is thin precisely because it is the part that belongs to someone else's job.

### 3.4 Why the boundary matters more than the org chart

Two failure modes follow from getting this boundary wrong, and both are common in security architecture hires.

The first is **scope inflation by the candidate**. A person who has spent years as a solution architect is used to owning the design, and the habit of ownership does not switch off when the title changes. In a security role that habit reads as overreach: the security architect who starts specifying which vector database the team should use has stopped being the person who can credibly say the design is unsafe, because they are now a co-author of it. Independence is the asset, and it is spent by participation.

The second is **scope collapse**, which is the more likely failure for this subject. Someone who has never held a security mandate tends to over-correct: to treat the role as advisory, to produce guidance instead of decisions, to describe risk rather than to refuse it. Section 11's kill criterion exists because scope collapse is hard to see from inside - it looks like being helpful, and it produces a role that everyone likes and no one is bound by.

The distinction that matters for everything that follows: **it does not own the AI systems, it owns the security position on them.** A position is only real when someone with a veto has to consider it. That is why section 6 spends its length on verbs.

---

## 4. The requirement-by-requirement map

Seven areas, thirty-six requirement rows, one verdict each. The verdict vocabulary is fixed and each value means one thing:

- **MET** - met as documented knowledge. Never read this as "met". The corpus demonstrates the knowledge; nothing demonstrates the mandate.
- **PARTIAL** - the knowledge exists for part of the requirement, or exists at method level while the operated artefact is absent.
- **NOT EVIDENCED** - neither the profile nor the corpus supports it.
- **UNKNOWN** - the sources are silent, and silence is not absence. The current role's internal content is not described in the profile beyond "GenAI solutions architecture" and architecture governance.

The gap type is knowledge, experience, accountability, or none. Where a row carries two types the first is the binding one.

### 4.1 AI Security Strategy and Architecture

| Requirement | Evidence for / against | Verdict | Gap type |
|:--|:--|:--|:--|
| Develop and maintain the enterprise AI security strategy | `ai_governance_framework_guide.md` s5 builds an operating model and s7 a gate structure; `security strategy` appears in 1 file, incidentally, in `technology/secure_red_hat_openshift_guide.md` | NOT EVIDENCED | accountability |
| Define security requirements and architectural principles for AI systems, applications, agents, ML platforms | `llm_development_risks_security_guide.md` s14 defence-in-depth architecture; `ai_risk_register_guide.md` s4 control-to-mode map; `ai_red_teaming_guide.md` s6 agentic surfaces | PARTIAL | experience |
| Establish minimum security standards for internally developed and third-party AI | `control library, control framework` in 40 files; `ai_risk_register_guide.md` s4 and s8; `ai_genai_banking_compliance_guide.md` s6 | PARTIAL | experience |
| Review and approve AI solution architectures from a security perspective | `architecture review, design review` in 103 files, all method; `design authority` in 7, all describing a body; no minute, conditions list or signature anywhere in the corpus | NOT EVIDENCED | accountability |
| Provide security guidance across the AI development lifecycle | `llm_development_risks_security_guide.md` s15 secure development lifecycle; `security_by_design_guide.md` s6 DevSecOps; `secure SDLC` in 7 files, `security lifecycle` in 2 | PARTIAL | experience |
| Collaborate with Enterprise Architecture | profile records architecture governance in the current role; `skill_gaps_enterprise_architect_guide.md` documents EA framework fluency (TOGAF, BIAN); no record of a joint security-EA artefact | PARTIAL | experience |

The knowledge half of this area is strong and the authority half is empty, and the split runs exactly along the line between writing and deciding. A person who can write s5 and s7 of the governance framework guide has the design vocabulary for an enterprise AI security strategy; the corpus contains the components of one. What it does not contain, and what no document could contain, is a strategy with this subject's name in the owner field. The row that matters most is the fourth: reviewing architecture is the role's daily decision, and 103 files about review methodology are evidence about method, not about ever having held the pen.

It is worth being concrete about what an AI security strategy actually contains, because the phrase sounds grander than the object. In practice it is five decisions and one page of prose: what the organisation will not do with AI (prohibited use classes); what must be true before any AI system touches production (the gates); who may approve an exception and on what evidence; how AI systems are inventoried and classified; and which external obligations the programme answers to. The corpus has material for all five - prohibited-use logic in the governance guide's risk appetite discussion, gates in s7, exception handling in the risk register's rating section, inventory discussion scattered across 11 files, obligations in s9. What it does not have is the sentence that turns five components into a ratified position: *this is what we will accept*.

That sentence is the whole distance. It cannot be written by the person who will be governed by it; it has to be proposed by the security function and accepted by an executive who now owns the consequences. Which is why the honest assessment of area 1 is not that the subject cannot produce the strategy - the material is there, and better organised than most - but that producing it would be a proposal rather than an act.

### 4.2 AI Risk Management

| Requirement | Evidence for / against | Verdict | Gap type |
|:--|:--|:--|:--|
| Maintain and evolve the AI cybersecurity risk taxonomy and control library with GRC | `ai_risk_register_guide.md` s3 failure-mode inventory and s4 controls; `risk taxonomy` in 25 files; `GRC` in 17; `frontier_ai_risk_guide.md` s2 on why taxonomies are contested | MET | knowledge |
| Identify, assess and prioritise AI-specific cyber risk | `ai_risk_register_guide.md` s6 rating and appetite, s11 the register in operation; `risk assessment` in 95 files | MET | none |
| Develop risk assessment methodologies for AI use cases and AI-enabled processes | `ai_risk_register_guide.md` s6; `ai_governance_framework_guide.md` s7 gates; `adversarial_ml_attacks_guide.md` s7 risk assessment for ML systems | MET | none |
| Monitor the threat landscape and translate emerging threats into controls | `adversarial_ml_attacks_guide.md` s3 taxonomy; `ai_red_teaming_guide.md` s7 standards; but `MITRE ATLAS, ATT&CK` in only 11 files | PARTIAL | knowledge |
| Support enterprise risk management | `three lines` in 45 files; `model risk` in 103 and `SR 11-7` in 88; `ai_governance_framework_guide.md` s5 role positioning and s6 committee lattice | PARTIAL | experience |

This is the area where the thesis is most clearly contradicted, and the guide should say so without hedging. Four of the five requirements are **knowledge requirements** in substance: a taxonomy, a rating method, an assessment methodology, and a translation discipline. The corpus meets three of them fully and one partially, and it meets them at a level of specificity - control-to-failure-mode mapping, an evidence test for audit ("a claim of control without an artifact is an assertion", s5), a three-lines adaptation with named role positioning (s5.5) - that is unusual outside a risk function. If this were the whole job, the verdict would be favourable.

It is not the whole job, and the fifth row is where the area turns. "Support enterprise risk management" is not a method; it is a seat at a table where the enterprise's risk appetite is argued. The corpus describes that table accurately and has never sat at it.

### 4.3 AI Security Standards and Controls

| Requirement | Evidence for / against | Verdict | Gap type |
|:--|:--|:--|:--|
| Develop AI security standards, guidelines and procedures aligned to recognised frameworks | `NIST AI RMF` in 23 files, `ISO/IEC 42001` in 16, `EU AI Act` in 55; `ai_governance_framework_guide.md` s2, s4, s9; `ai_verify_guide.md` s4 maps eleven principles | MET | knowledge |
| Requirements covering generative AI and LLMs | `prompt injection` in 55, `jailbreak` in 30, `guardrail` in 227; `llm_development_risks_security_guide.md` one section per OWASP LLM01-LLM10 | MET | knowledge |
| Requirements covering AI agents | `AI agent` in 124 files; `ai_red_teaming_guide.md` s6; `ai_risk_register_guide.md` s9 agentic extension; but `agent abuse` and `agent hijack` in 0 | PARTIAL | knowledge |
| Requirements covering RAG | `technology/ai_llm/rag/` holds 20 files; `prompt_injection_guide.md` s8; the `RAG.*security` figure of 113 is a proximity match, not coverage | PARTIAL | knowledge |
| Requirements covering model hosting environments | `model hosting` in 8 files; `inference security` in 0 | NOT EVIDENCED | knowledge |
| AI supply chain security | `AI or model supply chain` in 7 files against 138 for bare "supply chain"; `attestation, SOC 2` in 105; `ai_risk_register_guide.md` s8 | PARTIAL | knowledge |
| Sensitive data handling | `PII` in 101, `sensitive data` in 69; `ai_genai_banking_compliance_guide.md` s5 privacy and s6 security requirements | MET | knowledge |
| Third-party AI services | `third-party (ai or risk)` in 67, `third-party risk` in 54, `vendor risk` in 37; `ai_risk_register_guide.md` s8 | MET | knowledge |

Six of the eight rows are met or partly met on knowledge, which makes this the strongest area in the guide and the one place where "the gap is not what you know" is simply true rather than rhetorical. The framework alignment is not name-dropping: the corpus uses NIST AI RMF as a structure, ISO/IEC 42001 as a management-system requirement set, and the EU AI Act as an obligations map, and it puts them in a bank-specific stack in s4 of the governance guide.

The two rows that fail are informative. **Model hosting** is the gap that most clearly belongs to someone else's job - the platform team owns the hosting environment, and the corpus reflects that division of labour by never describing it. **Agent abuse** at zero, in a corpus with 124 files about agents, is the sharpest single number in this audit: the subject writes about agents extensively as an architecture pattern and almost not at all as an attack surface in its own right. `ai_red_teaming_guide.md` s6 is the exception that proves the rule - one strong section, no dedicated treatment.

### 4.4 AI Red Teaming and Security Assurance

| Requirement | Evidence for / against | Verdict | Gap type |
|:--|:--|:--|:--|
| Establish and manage the enterprise AI red teaming programme | `ai_red_teaming_guide.md` s8 the process and s9 automated vs human tooling; `red team` in 35 files; no calendar, budget, participant list or findings register exists | PARTIAL | accountability |
| Coordinate technical assessments prior to production deployment | `ai_risk_register_guide.md` s4 treats evaluation as a gate; `AI assurance` in 4 files; `control testing` in 8 | PARTIAL | experience |
| Track remediation to closure | `incident response` in 84 files; `control testing, control assurance` in 8; no remediation log, no ageing metric, no closure record | PARTIAL | experience |
| Work with external partners on methodology for prompt injection, jailbreak, data poisoning, model manipulation, model extraction, agent abuse, unauthorised data disclosure | `prompt injection` 55 and `jailbreak` 30 are well covered; `data poisoning` 7, `model extraction` 6, `model manipulation` 1, `agent abuse` 0; no named partner relationship anywhere | PARTIAL | knowledge |

Read the fourth row's seven items against the counts and the shape is a slope: the two attack classes that are easy to write about from public material (prompt injection, jailbreak) are dense; the four that require hands on a system or a vendor relationship (poisoning, extraction, manipulation, agent abuse) are thin to absent. That slope is the signature of a reading-and-writing pass rather than an engagement, and it is the most useful single diagnostic in this guide.

The first row is where the accountability gap is cleanest. `ai_red_teaming_guide.md` s8 describes a red-team process competently - scoping, rules of engagement, reporting, retest. A programme is that process plus a standing commitment of people, money and calendar inside an organisation. The corpus has the first and cannot have the second. Section 7 ranks this at five, and section 11 says what one real exercise would do to the gap.

The fourth row deserves one more observation, because it is the row where a candidate is most likely to be caught out in interview. The clause lists seven attack classes and asks for collaboration with external partners *on methodologies* for them. A vendor methodology is a product: a test plan, a coverage matrix, a severity scale, and a report template that the vendor's clients have agreed to accept. Knowing how prompt injection works, even in detail, is not the same object. The tell in interview is whether the candidate describes the attack or describes the engagement - scoping, access, safe-word, evidence handling, retest window. The corpus is strong on the first and has almost nothing on the second, and that asymmetry is more diagnostic than any count.

### 4.5 AI in Cyber Defense

| Requirement | Evidence for / against | Verdict | Gap type |
|:--|:--|:--|:--|
| Identify opportunities to use AI within security operations | `security operations, SecOps` in 47 files; `cybersecurity_guide.md` describes the security organisation and its tooling; no survey of AI-assisted SOC tooling | PARTIAL | knowledge |
| Partner with Cyber Defense and SecOps to evaluate and deploy AI-driven security capability | no evidence of a joint deployment; the profile describes GenAI solution architecture, not security tooling | NOT EVIDENCED | experience |
| Assess risk and benefit of AI-assisted security tools | adjacent material exists - `llm_development_risks_security_guide.md` s19 governance applies by analogy; nothing assesses a security vendor's model | PARTIAL | knowledge |
| Develop governance and measurement for defensive AI use cases | `AI assurance` in 4 files; `capability maturity, SAMM` in 10; no measurement framework for a defensive deployment | NOT EVIDENCED | knowledge |

This is the thinnest of the seven areas on both axes, and it is the one a reader is most likely to skim past because it sounds like an opportunity rather than a requirement. It is a requirement: the role is expected to sit between the AI programme and the SOC and to be the person who can say whether a model-driven detection tool is worth its risk. The corpus knows how to assess an AI system that faces customers; it has almost nothing on assessing an AI system that faces attackers, and nothing at all on measuring whether one worked.

The gap here is genuinely a knowledge gap in part, and that is the honest reading. Unlike areas 1 and 7, this one could be substantially closed by study plus one deployment, because the object of assessment is a tool rather than an institution.

There is also a second-order point worth stating, because it is the direction the industry is moving. Defensive AI is not only "use a model to triage alerts"; it is also "the SOC now depends on a model, and that model is an attack surface". A detection tool that ingests untrusted content, or an agent that can query logs and take containment actions, imports every attack class from area 4 into the SOC. The role that owns AI in cyber defense therefore has to hold both sides at once: the benefit case and the new exposure. The corpus holds neither side for the SOC specifically, which is why this row is THIN rather than PARTIAL.

### 4.6 Awareness and Capability Building

| Requirement | Evidence for / against | Verdict | Gap type |
|:--|:--|:--|:--|
| Develop and maintain enterprise AI security awareness and training programmes | `security awareness, security training` in 4 files; `cybersecurity_guide.md` s5.5 gives one section to awareness as a programme; no module, no curriculum, no cadence, no measurement | NOT EVIDENCED | experience |
| Educate technical teams | the corpus itself is the evidence of teaching material - 686 files, several explicit worked examples; no evidence of delivery to a cohort | PARTIAL | experience |
| Provide SME support to security teams, architects, developers and business stakeholders | the writing supports the claim of expertise; nothing evidences a support obligation or a queue | UNKNOWN | accountability |
| Support capability across regional security organisations | no regional mandate, no entity structure, nothing in the footprint | NOT EVIDENCED | accountability |

The distinction this area turns on is between **producing teaching material** and **running an awareness function**. A guide is a thing you write once; an awareness programme is a thing that runs four times a year against a population you can name, with a completion rate and a pre/post delta. `cybersecurity_guide.md` s5.5 understands that distinction and describes the second thing well. The corpus has never done it, and the four-file count is the thinnest coverage in the audit of any requirement the job description treats as a standing deliverable.

The last row deserves a plain statement, because it is easy to misread as a formality. "Support capability across regional security organisations" means the role's advice is expected to be authoritative in more than one legal entity, which requires a mandate that crosses them. Nothing in the profile suggests the subject's current remit extends beyond one institution, and there is no evidence of any multi-entity mandate in any prior role.

One further honest note on the "educate technical teams" row, which is graded PARTIAL rather than NOT EVIDENCED. The corpus is, functionally, a teaching corpus: worked examples, decoded jargon, explicit section maps, anti-pattern lists. That is what good training material looks like. What is missing is the delivery loop - a cohort, a schedule, an assessment, a follow-up - and the delivery loop is what turns material into capability. The distinction matters for how this gap is closed: it is not closed by writing another guide.

### 4.7 Governance and Stakeholder Engagement

| Requirement | Evidence for / against | Verdict | Gap type |
|:--|:--|:--|:--|
| Act as the primary Information Security SME for AI | no security title in the profile at any point in the career; no security function named | NOT EVIDENCED | accountability |
| Partner with Data and AI, Legal, Compliance, Procurement and Enterprise Architecture | `ai_governance_framework_guide.md` s5.5 role positioning maps which existing bank role takes which AI accountability; `ai_genai_banking_compliance_guide.md` s3 and s8 | PARTIAL | experience |
| Support governance forums, steering committees and security review boards | `steering committee, review board` in 47 files; `three lines` in 45; design of such forums is documented, attendance is not | PARTIAL | accountability |
| Provide regular status and risk reporting to senior leadership | `model risk` reporting material in 103 files, `SR 11-7` in 88; no pack, no cadence, no named audience | NOT EVIDENCED | accountability |
| Represent Information Security in strategic AI initiatives | nothing in the profile or the footprint | NOT EVIDENCED | accountability |

Four of five rows are accountability gaps, which makes this the area where the thesis holds most completely and where the least can be done about it by preparation. The one partial row is genuinely partial in the subject's favour: s5.5 of the governance guide does the hard analytical work of mapping AI accountabilities onto roles a bank already has - which function owns the model, which owns the data, which owns the third party, which owns the security position - and doing that mapping well is what a governance role actually spends its time on. It is the closest thing in the corpus to the job.

What is missing is the design's other half: the politics. A governance forum is not a diagram; it is a group of people with competing priorities and a chair who decides what reaches the committee. Nothing in 686 files describes having navigated one.

The reporting row is worth one sentence more, because it is the row most often misjudged by technical candidates. "Provide regular status and risk reporting to senior leadership" is graded NOT EVIDENCED and a reader may object that the corpus contains plenty of analysis suitable for executives. It does. But a status report to senior leadership has three properties analysis does not: it is short, it carries a recommendation with a named owner, and it is delivered on a date whether or not the news is good. The discipline is not the writing; it is the cadence and the accountability for what the number says when it moves the wrong way.

### 4.8 Reading the map against the thesis

Thirty-six rows: **7 MET, 18 PARTIAL, 10 NOT EVIDENCED, 1 UNKNOWN**, and the gap-type column tells a cleaner story than the verdict column does. Every MET row is a knowledge row. No row typed accountability is graded MET, without exception - accountability rows are PARTIAL, NOT EVIDENCED or UNKNOWN, never met. The thesis survives the test with one important qualification, and the qualification is not decorative:

**The thesis is too strong where the requirement is a method and too weak where the requirement is a mandate.** Areas 2 and 3 are largely method, and there the corpus meets the requirement outright - a reader who took the thesis literally would underrate how much of this job is analytical work the subject can already do. Areas 1, 4 and 7 are largely mandate, and there the corpus meets almost nothing. A more accurate restatement, which section 16 uses, is: the gap is not what you know about AI security, it is what you have been accountable for - and roughly half this job description is written as accountability.

Two further readings of the row counts are worth recording, because both cut against a simple story.

First, the PARTIAL band of 18 is the largest, and it is not a hedge. PARTIAL means the knowledge exists for part of the requirement, or exists at method level while the operated artefact is absent - which is a precise description of a person who has written a control library and never had one tested. Collapsing those 18 rows into "met" would flatter; collapsing them into "not met" would be false. The band is where the whole argument lives.

Second, only one row is graded UNKNOWN, and it is a row that would matter if it turned out positive: 4.6, the SME support obligation to security teams, architects, developers and business stakeholders. That is a property of the sources rather than of the subject: the profile describes roles and not remits, so the internal content of a two-decade career is largely invisible. Every UNKNOWN in this guide is an invitation to supply a fact, and the fact is usually held by the subject rather than by the repository.

---

## 5. The heatmap

Knowledge scored on the corpus counts and section citations; authority scored on whether any artefact, title, mandate or decision right is evidenced anywhere in the profile or the footprint.

| Area | Demonstrated knowledge | Demonstrated authority | Binding gap type |
|:--|:--|:--|:--|
| 1. Strategy and architecture | SOLID | NOT EVIDENCED | accountability |
| 2. Risk management | DENSE | UNKNOWN | experience |
| 3. Standards and controls | DENSE | NOT EVIDENCED | experience |
| 4. Red teaming and assurance | SOLID | NOT EVIDENCED | accountability |
| 5. AI in cyber defense | THIN | NOT EVIDENCED | knowledge |
| 6. Awareness and capability | THIN | NOT EVIDENCED | experience |
| 7. Governance and stakeholders | SOLID | NOT EVIDENCED | accountability |
| Organisational-verb class | SOLID | NOT EVIDENCED | accountability |

Scoring key: DENSE means the area is covered by a dedicated guide of 700 lines or more plus adjacent material; SOLID means covered across several guides without one dedicated treatment; THIN means single-digit to low-double-digit file counts with no dedicated treatment; ABSENT means the phrase does not appear.

### 5.1 The diagonal, and the places it bends

Six of the eight rows sit on a diagonal: knowledge SOLID or DENSE, authority flat. That is the shape the thesis predicts, and it is worth noting that the two darkest knowledge cells (areas 2 and 3) are also the two areas where the job description reads most like a specification rather than a mandate. The corpus is best exactly where the role is most documentable and worst where the role is most institutional.

Two rows sit below the diagonal rather than on it, and the reason is structural rather than personal.

- **Area 5 is THIN on knowledge, not only on authority.** It is the one requirement area where the corpus itself fails, and it fails for a reason worth understanding: the AI-facing-attackers problem is not a solution-architecture problem, so the subject has never had a professional reason to write about it. A gap produced by the shape of a career is easier to close than one produced by a limitation, and this one would close in weeks of focused work.
- **Area 6 is THIN on knowledge and absent on authority.** Awareness work is the least written-about part of a security function in any corpus, because the practitioners who do it write slides rather than essays. The subject is not unusual here; the area is simply unglamorous, and the four-file count reflects that as much as it reflects anything about the subject.

One further bend deserves its own note, because it is the only place where the authority column is not flat.

- **Area 2 carries UNKNOWN rather than NOT EVIDENCED for authority.** A solution architect inside a bank's change process is dragged into risk conversations whether or not the job title says so, and the profile records architecture governance as part of the current role. The area's two experience rows in 4.2 are graded PARTIAL for exactly this reason. The profile does not describe what that participation amounts to, so UNKNOWN is the honest grade - and it is the single most valuable thing to clarify, because if the subject already sits in a risk forum with a vote, one of the ten gaps in section 7 shrinks materially.

The organisational-verb row is included because it cuts across all seven areas. It is scored SOLID on knowledge because the corpus describes what these verbs produce, and NOT EVIDENCED on authority because none of them has been exercised. Section 6 takes it apart.

### 5.2 How to read this table without flattering it

A heatmap is the most abusable artefact in this genre, because colour is an argument and a reader remembers the colours rather than the evidence. Three cautions apply to the table above.

**The knowledge column measures volume with structure, not quality.** A DENSE score means there is a dedicated guide of 700 lines or more plus adjacent material. It does not mean the guide is correct, current, or better than what a practitioner would write. Where this guide makes a quality judgement it does so in section 8, with a citation, and the judgement is contestable.

**The authority column is a statement about the sources, not about the person.** NOT EVIDENCED means no artefact in the profile or the footprint evidences the authority. It does not mean the authority is absent; it means the evidence is absent. The distinction is the whole reason section 15 exists.

**No cell is a hiring prediction.** A row reading THIN on knowledge and NOT EVIDENCED on authority describes the state of the evidence for the requirement. Whether the panel forgives it depends on the requisition, the competition and how the candidate handles the interview - which is section 10's subject and not this table's.

## 6. The organisational-verb class

Section 4 tested areas. This section tests **verbs**, because the job description's difficulty is not distributed evenly across its nouns - it is concentrated in seven verbs that all mean "someone else must accept this from you".

| Verb in the JD | What authority looks like | What the repository can show | What it cannot show |
|:--|:--|:--|:--|
| develop and maintain | an artefact that persists, is versioned, and lapses if the owner leaves | the artefact's design, in detail | that it exists, or that it is anyone's |
| establish and manage | a programme with participants, a calendar, a backlog, a budget | the programme's process design | the people, the money, the calendar |
| review and approve | a decision right with a veto and a record | review methodology | a single minute or condition |
| act as the primary SME | a named designation inside a governance structure | expertise, abundantly | the designation |
| provide regular reporting to senior leadership | a cadence, a named audience, consequences | what good reporting contains | that anyone received it |
| collaborate with external partners | a relationship the institution holds | methodology for the collaboration | the contract, the NDA, the partner |
| support regional security organisations | a mandate that crosses legal entities | nothing | everything |

### 6.1 Develop and maintain

"Maintain" is the load-bearing word, not "develop". A maintained artefact has a version number, a change log, a review date and an owner who is asked why it has not been reviewed. The corpus can show what such a document should contain - `ai_governance_framework_guide.md` s7 lays out the gates and their artifacts with a completeness that suggests the author has read real ones. What it cannot show is a document whose next review date is someone's problem. A guide in a public repository has no review date, because nothing breaks when it is stale.

### 6.2 Establish and manage

This is the verb that most clearly separates writing from running. Establishing a programme means getting a line in a budget, a name on a calendar, and a queue of work that other people are expected to clear. Managing it means the queue is measured. `ai_red_teaming_guide.md` s8 describes scoping, rules of engagement, reporting and retest, which is the design of such a programme and a genuinely good one. The distance between that design and a programme is not intellectual - it is seven signatures and a purchase order.

### 6.3 Review and approve

Approval is the only verb in the list that produces a **veto**, and a veto is the thing a security architect is actually for. Everything else in the role can be done by an adviser; this cannot. The mechanics are unglamorous: a paper goes to a forum, the forum has terms of reference that say the security function must concur before go-live, and the security architect writes a decision - approved, approved with conditions, or not approved - which is recorded and can be audited two years later.

Nothing in the profile supports having written such a decision. `architecture review, design review` appears in 103 files and every one of them is about how to conduct a review; `design authority` appears in 7 and every one describes a body rather than a seat on one. This is gap 4 in section 7, and section 7 says plainly that it is largely not closable by study.

### 6.4 Act as the primary SME

"Primary" is doing the work here. Many people are a subject matter expert; the phrase in a job description means the organisation has designated one person as the answer of record, and that other functions route their questions to that person rather than to whoever is available. Designation is a documentary act - a terms-of-reference clause, an org chart, a distribution list - and it is invisible from outside the institution. The subject's expertise is not in doubt; the designation is absent, and no artefact in a public repository can create it.

### 6.5 Provide regular reporting to senior leadership

"Regular" implies a cadence and "senior leadership" implies an audience with the power to reallocate resources. A report to that audience is different in kind from a written analysis: it is short, it carries a recommendation, and it creates an obligation to act or to explain not acting. The corpus contains plenty of analysis. It contains no pack, no cadence, and no evidence that any senior leader has ever been given a document with this subject's name on the front and had to respond to it. `model risk` material in 103 files shows fluency with what such reporting contains; fluency with contents is not the same as having produced one under a deadline for an audience that could say no.

### 6.6 Collaborate with external partners

The job description's phrasing is precise and easy to skim: *collaborate with external partners on methodologies*. The subject of that sentence is the institution, not the individual. An external partner - a red-team vendor, an assessment firm, a peer bank in an information-sharing group - contracts with an entity, and the individual who "collaborates" is the entity's representative. Methodological knowledge (which the corpus has, in the guides' treatment of attack classes) does not create the relationship and cannot substitute for it. This is the row where an otherwise strong candidate is most likely to overclaim in interview, and section 10 gives the honest answer shape.

### 6.7 Support regional security organisations

The hardest verb and the shortest treatment, because there is nothing to analyse. A regional mandate means the role's security position is authoritative in more than one jurisdiction or legal entity, which requires a delegation of authority and a reporting line that crosses them. The profile describes one institution in one city. Nothing in any role suggests a multi-entity remit, and no amount of AI-security knowledge produces one.

### 6.8 The point of the section

**A written standard and an owned standard are different objects even when the text is identical.** If the subject's `ai_risk_register_guide.md` s4 were lifted verbatim into a bank's control library, the words would not change and the object would. The owned version has a control owner, a test procedure, a failure escalation path, an exception route, and a date on which someone has to attest that it works. The written version has a heading and a table.

This is why the guide refuses to convert the seven verbs into study tasks. Six of the seven produce an artefact that requires an institution to exist around it. The one that is partly learnable - review methodology, the craft of reading an architecture for security defects - is already the subject's strongest adjacent skill, which is exactly why gap 4 is painful: the capability is present and the seat is not.

### 6.9 Which of these verbs can be borrowed, and which cannot

The verbs are not equally inaccessible, and the difference is worth stating because it is the only actionable content in this section.

**Borrowable, in the current role, without a security title.** Review methodology: a solution architect already reviews designs, and adding a structured threat-model pass to those reviews produces a review artefact with a security dimension. Reporting: any architect who presents to a design forum or a programme board is producing something adjacent to a risk report, and can deliberately shape it as one. Educate: a brown-bag session on prompt injection risk for a delivery team is a real delivery, small but real.

**Borrowable only with sponsorship.** "Develop and maintain" can be practised on an internal standard if someone with authority agrees it will be used - which is a conversation, not a document. "Establish and manage" can be approached by running a single exercise under an existing programme's banner rather than creating a new one.

**Not borrowable at all.** Approve, act as primary SME, support regional organisations. Each of these is a designation, and a designation is granted rather than earned. A candidate who tries to borrow one by behaving as though they hold it will be read as overreaching, which is worse than the gap.

The practical implication for section 11 is that the first horizon should spend its effort on the borrowable column and stop pretending the third column is addressable.

---

## 7. The real gaps, ranked

Ten gaps, ranked by what would disqualify or delay an offer rather than by what is interesting to fix. Each carries its gap type, what would close it, and an honest answer to whether it is closable outside a security job. The ranking rule is simple: a gap that a hiring committee can see in ninety seconds and cannot test around ranks above a gap that can be probed in interview and argued.

### Gap 1: No security function title or security accountability, at any point in the career

**Type: accountability.** Two decades of engineering, team-lead and architecture roles across several employers, none of them in a security function. The profile lists no security role, no security team, no security reporting line. This ranks first because it is the fact from which every other accountability gap follows, and because it is the first thing a hiring manager checks: not "has this person studied security" but "has a security function ever been willing to put this person in front of a regulator".

Closing it: a role, internal or external, where the title or the terms of reference carry security accountability. Nothing smaller works. A certificate does not close it; a project does not close it.

Closable outside a security job: **no.** This is the guide's least comfortable sentence and it should not be softened. It is closable by a lateral move inside the current institution - a security architecture or technology-risk remit attached to the existing role - which is a political act, not a study plan.

### Gap 2: No security certification at the screening stage

**Type: knowledge, in form; accountability, in effect.** The recorded certifications are Claude Certified Architect - Foundations (Anthropic; the profile says 2026, the footprint says approximately 2025), GitOps Fundamentals, Fundamentals for Istio (Solo.io), Log Management Fundamentals (Datadog), Kong Mesh Foundations (Kong) and Dagster Essentials. There is no CISSP, CISM, CRISC, CISA, CCSP or GIAC - verified by reading the education and certification sections of both source files, not inferred.

This ranks second because it is a filter rather than a judgement: many security architect requisitions screen on one of those acronyms before a human reads the CV. It is also the cheapest gap on the list to close, which is why it ranks above gaps that are harder to live with.

Closing it: one certification, taken and passed. See section 9 for which and why. It closes a screening gap and, as section 9's last line says, nothing else.

Closable outside a security job: **yes** - this is the one gap on the list that study alone closes.

### Gap 3: No owned enterprise strategy or standard

**Type: accountability.** The corpus can produce the components of an AI security strategy - principles, gates, taxonomy, control library, framework stack, role positioning - and does so across four guides. What it cannot produce is a strategy with an owner field, an approval record and a review cycle. `security strategy` appears in 1 file, incidentally. This ranks third because the role's first deliverable is exactly this artefact, and a candidate who has never owned one will be asked how they would get one signed, which is a political question disguised as a document question.

Closing it: publish an internal standard with a named approver and a review date. Even in the current role, a GenAI architecture standard with a formal approval trail would materially change this row.

Closable outside a security job: **partially** - the artefact can be produced anywhere an approval process exists, but a security standard needs a security approver to be credible.

### Gap 4: Never chaired or sat on a security review board or design authority with a veto

**Type: accountability.** `design authority` in 7 files, all descriptive. `architecture review, design review` in 103, all method. No minute, no conditions list, no refusal, no signature. This ranks fourth, immediately behind the title gap, because it is the mechanism through which a security architect has any effect at all: without a veto the role is advisory, and an advisory security architect is a consultancy with a badge.

Closing it: membership of a review forum where the security opinion is recorded and can stop a release. In the current institution this may be reachable as an invited participant before it is reachable as a chair.

Closable outside a security job: **no, not really.** Architecture review boards exist outside security functions, and sitting on one is useful, but the specific object - a security veto - requires the security mandate. This is the second of the two gaps the guide says study cannot touch.

### Gap 5: No red-team programme operated

**Type: accountability, with an experience layer.** The methods are written and they are good: `ai_red_teaming_guide.md` s2 taxonomy, s5 poisoning/extraction/inversion, s6 agentic and RAG surfaces, s8 process, s9 tooling, plus a worked example in s10. A programme is that plus people, calendar, budget and a findings register. `red team` appears in 35 files and not one of them is a register. There is also no external partner relationship anywhere in the profile, which the job description names explicitly.

Closing it: run one exercise against one real system, with a scoping document, a findings register, a remediation log and a retest. One is enough to change the conversation from "I have written about this" to "I have done this once".

Closable outside a security job: **yes, once** - a red-team exercise can be commissioned or run by an architecture team against its own system, and doing so is the highest-value single action in section 11.

### Gap 6: No evidence of pre-production assurance gate operation or remediation tracking

**Type: experience.** Evaluation-as-gate is described well (`ai_risk_register_guide.md` s4), and `AI assurance` appears in only 4 files. Remediation tracking is thinner still: `control testing, control assurance` in 8 files, no ageing metric, no closure record, no exception register. A gate that is described but never operated is a diagram; the operational half is the part where a finding is logged, chased, escalated and closed, and where somebody is accountable for the residual risk when it is not closed.

Closing it: own a gate for one release cycle, with a decision record per system and a remediation log with dates.

Closable outside a security job: **partially** - a GenAI solution architect can insist on and operate a pre-production security review for their own platforms, which is the most natural bridge available from the current role.

### Gap 7: Awareness programme design absent

**Type: experience.** `security awareness, security training` in 4 files, none describing a programme with a cadence, an audience and a measurement. `cybersecurity_guide.md` s5.5 treats awareness as a programme in one section and does it competently - which is section-level treatment, not a programme the author ran. This ranks seventh rather than higher because it is a standard deliverable that most candidates have also not run, and because it is the most learnable of the experience gaps.

Closing it: build and deliver one module to a real cohort with a pre/post measurement. Small, checkable, and it converts a NOT EVIDENCED into an evidenced experience.

Closable outside a security job: **yes** - awareness material can be delivered inside any engineering organisation.

### Gap 8: AI in cyber defense is thin

**Type: knowledge.** `security operations, SecOps` in 47 files, but nothing that surveys AI-assisted SOC tooling, nothing that assesses a security vendor's model, nothing on measuring whether a defensive deployment worked. This ranks eighth because it is one of seven areas rather than the core of the role, and because it is the most cleanly closable knowledge gap on the list.

Closing it: a structured study of the defensive-AI tool categories plus one assessment of one tool. Weeks, not years.

Closable outside a security job: **yes.**

### Gap 9: Model hosting and inference security thin

**Type: knowledge.** `model hosting` in 8 files, `inference security` in 0. The role is expected to set requirements for model hosting environments, and the corpus has no architectural treatment of the hosting layer - no inference endpoint hardening, no model-serving isolation, no key and weight protection, no tenancy model. This is thin partly because the platform team owns that layer, which is a good reason and not a defence: the job description asks the security architect to set requirements for it.

Closing it: a focused piece of work on inference infrastructure security, ideally against a real deployment.

Closable outside a security job: **yes**, and it is unusually well suited to the subject, who has 425 files of technology corpus and a platform-engineering background to attach it to.

### Gap 10: Agent abuse and unauthorised data disclosure at the agent boundary

**Type: knowledge.** `agent abuse, agent hijack` in 0 files, in a corpus with 124 files about AI agents. `ai_red_teaming_guide.md` s6 covers agentic surfaces in one section; `ai_risk_register_guide.md` s9 extends the register to agents. Given that the job description names agent abuse explicitly and that agents are where the next two years of AI security incidents will come from, this is the most surprising absence in the corpus. It ranks tenth only because it is a knowledge gap and the most tractable one on the list - which is a comment on the nine above it, not on its importance.

Closing it: a dedicated treatment of agent attack surfaces - tool-scope abuse, confused deputy, memory poisoning, credential reach, multi-agent trust - plus a test matrix.

Closable outside a security job: **yes**, and quickly.

### Reading the ranking

Two gaps - 1 and 4 - are largely not closable by study, and they are the two that matter most. Four - 2, 8, 9, 10 - are cleanly closable and worth doing anyway because they remove screening filters and interview objections. The remainder - 3, 5, 6, 7 - are closable by doing the thing once, which is cheaper than it looks and is the entire content of section 11's first horizon.

The ranking has a shape worth naming: the closable gaps are all about **what the subject would know**, and the unclosable ones are all about **who has accepted the subject's judgement**. That is the thesis again, arrived at from the other direction.

### A note on what is not on this list

Two candidates for the list were considered and excluded, and the exclusions are as informative as the inclusions.

**Depth in any single AI-security subdomain.** The corpus is broad and, in places, deep, but it is not the work of a specialist in any one of them - there is no multi-year research programme behind prompt injection, no dataset of jailbreaks, no published exploit work. That is a real limitation for a role that will be asked "what is your area", and it is not on the list because it is not a gap the role requires: the job description asks for breadth across seven areas and a strategy, not a research specialism.

**Formal AI governance or privacy credentials.** The corpus has 84 files on AI governance and 40 on responsible AI, and no IAPP-style certification. This was excluded because it is subsumed by gap 2 - the certification gap - and because the target role's screening is more likely to name CISSP than an AI governance certificate.

Both exclusions share a reason: a ranked list of ten is a claim about what matters, and a list that includes everything that could be improved is a list that has declined to rank.

## 8. What genuinely helps

The gap analysis is only useful if it is paired with an account of what transfers. Nine things do, and each is cited to a section rather than to a filename, because a section is a claim and a filename is a shelf.

| Transferable strength | Where it is evidenced | Why it transfers to this role |
|:--|:--|:--|
| the risk-register method | `ai_risk_register_guide.md` s3 failure-mode inventory, s4 control-to-mode map, s5 the evidence test, s7 three lines, s8 third-party entries | the role maintains a taxonomy and control library; this is that artefact, built |
| control library and control-to-mode mapping | same, s4 | the hardest part of a control library is mapping controls to failure modes without gaps or duplicates |
| three-lines operating model and role positioning | `ai_governance_framework_guide.md` s5, especially s5.5 | the role is a second-line function and must know where its line sits |
| lifecycle gates and artifacts | `ai_governance_framework_guide.md` s7 | the assurance gate the JD owns is a lifecycle gate with security conditions attached |
| red-team process and attack taxonomy | `ai_red_teaming_guide.md` s2 family map, s8 process, s9 tooling | the programme design exists; only the programme does not |
| the OWASP-mapped risk landscape | `llm_development_risks_security_guide.md` s2-s12, one section per LLM01-LLM10 | this is the vocabulary a standards author needs and most candidates lack |
| threat modelling and secure-by-design review | `security_by_design_guide.md` s2, s9 | architecture review under a security lens is a threat-model discipline before it is a governance one |
| architecture governance in a regulated bank | profile: current role includes architecture governance | knowing how a bank's design authority actually processes a decision is the political prerequisite |
| SR 11-7 and model-risk fluency | `model risk` 103 files, `SR 11-7` 88, `ai_governance_framework_guide.md` s6 | AI security in a bank is negotiated with the model-risk function, not around it |

### 8.1 Three strengths worth stating plainly

**The taxonomy work is genuinely good.** `ai_risk_register_guide.md` s3 does not list AI risks; it inventories failure modes and then maps controls to them in s4, which is the difference between a risk register and a risk poster. Any competent security architect would recognise the method.

**The framework stack is operationalised, not recited.** s4 of the governance guide puts NIST AI RMF, EU AI Act, ISO/IEC 42001 and the Singapore instruments into a stack a bank could actually run, with the conflicts named. Reciting frameworks is common; stacking them without contradiction is not.

**The evidence test is the right instinct.** "A claim of control without an artifact is an assertion" (s5) is the sentence a control-testing function lives by, and it appears unprompted in a guide about registers. That is the mark of someone who thinks like an assurer.

### 8.2 The calibration paragraph

Three claims are commonly made together in this genre and they are not equally true. Separating them is the point of this section.

- **Writing a control library is genuinely close to building one.** The artefact is the deliverable. A control library is a document with an owner and a test procedure; if the written version has the controls, the failure modes, the test descriptions and the exception route, then the marginal work of making it real is approval and maintenance, not authorship. The repository is close to this one, and s4 of the risk-register guide plus s7 of the governance guide are most of the way there.
- **Writing a strategy is not close to owning one.** A strategy is a set of choices about what the organisation will refuse to do, ratified by people who can be held to it. The written version can be excellent and still be a proposal. Nothing about authorship transfers the ratification.
- **Writing about red teaming is not close to running a programme.** A programme is a standing commitment of other people's time. One exercise closes most of the distance; none closes none of it. This is why section 11 puts an exercise ahead of any further writing.

The pattern: **distance to the artefact is short where the artefact is a document, and long where the artefact is an obligation.** Everything in this guide follows from that sentence.

### 8.3 One strength that is easy to miss

The corpus contains something a security function usually has to buy: **worked examples on fictional institutions**. `ai_risk_register_guide.md` s12 runs a register build end to end on a fictional bank; `ai_governance_framework_guide.md` s10 does the same for a governance rollout; `ai_red_teaming_guide.md` s10 walks a GenAI assistant through an assessment; `ai_genai_banking_compliance_guide.md` s8 runs a gap assessment; `cybersecurity_guide.md` s9 runs a security programme.

Worked examples are what make a standard teachable. A control that cannot be illustrated on a concrete system is a control nobody will apply consistently, and the ability to produce the illustration on demand is the difference between a standard that changes behaviour and one that gets filed. This is a genuine, undersold strength, and it is directly usable in the role's awareness and standards work.

The caveat is the same one that runs through the guide: an example is not an engagement. A worked example on a fictional bank proves the author can construct the case; it does not prove the author has run one where the numbers were real and someone disagreed.

---

## 9. The credential question

The two profile files record six certifications in total and no security certification of any kind. Not CISSP, CISM, CRISC, CISA, CCSP or GIAC. This was checked directly in both source files rather than inferred from absence in a summary, and the corpus confirms it indirectly: those acronyms appear in 20 files, every one of them discussing certification as a topic or recommending it, none claiming the author holds one.

That matters here because this is a security architect requisition, and a large share of such requisitions screen on one of those acronyms. What that means and what it does not:

- **It means** a filter may reject the application before a human reads it. That is a real cost measured in opportunities, not in competence.
- **It does not mean** the knowledge is absent. The repository is the counter-evidence, and a 686-file corpus with 84 files on AI governance and 55 on prompt injection is not what an unqualified candidate produces.

### 9.1 What is worth obtaining, and why

| Credential | What it buys | When it is the right call |
|:--|:--|:--|
| CISSP | breadth, the screening filter, and the vocabulary of security management | now - it is the single highest-yield item for this target |
| CCSP | cloud security depth, and a second filter where the JD names cloud | only if the target organisation's JD names cloud security explicitly |
| an AI-security-specific credential | signal of specialisation; little screening value yet | only if the JD names AI security as the discipline, which this one does - but after CISSP, not instead of it |
| CRISC | risk and control fluency, GRC credibility | only if the path turns toward GRC rather than architecture |

The honest ordering is CISSP first, because it is the credential that opens the most doors for a role whose title contains "Security Architect" and whose screening is done by people who do not read repositories. The AI-specific credential is attractive and premature: the market for it as a filter is thin, and it would signal specialisation to an audience that has already assumed it.

### 9.2 This is a standing gap, not a new discovery

`skill_gaps_enterprise_architect_guide.md` line 126 already lists "CISSP or CISM" as a medium-term item, with the note that security knowledge is demonstrated and the certificate adds governance credibility. That was written months ago and the position has not changed. The difference now is priority: for an enterprise architect role a security certificate is a nice-to-have, and for this role it is a screening gate.

**A certification closes a screening gap, not an accountability gap.** It will get the CV read. It will not get the candidate a seat on a review board, and no certificate has ever conferred a veto.

---

## 10. The interview and the assessment

### 10.1 What this loop will test

| Stage | What it probes | What it is really checking |
|:--|:--|:--|
| architecture scenario with a security veto | a design is presented; the candidate must approve, condition or refuse | whether the candidate can hold a position under pressure |
| risk-taxonomy conversation | how the candidate would classify a novel AI use case | whether the taxonomy is internalised or recited |
| red-team programme design under budget | design a programme with a fixed spend and a fixed headcount | whether the candidate has ever had to trade scope against money |
| incident or near-miss narrative | tell us about a time something went wrong | whether the candidate has been close enough to failure to have one |
| stakeholder conflict with Legal or Data and AI | a disagreement about who decides | whether the candidate can lose a room and keep the relationship |
| a standard the candidate wrote | walk us through the document | whether it was written to be used or written to be shown |

### 10.2 Five questions that would expose a writing-only background

| Question | The trap | The honest answer shape |
|:--|:--|:--|
| "Which AI systems have you refused?" | there is no refusal to name | "None. I have not held a security veto. What I have done is write the review criteria I would apply, and here is the one I would have used on a system I know." |
| "Who signed your last standard?" | no signature exists | "Nobody. It is a public guide, not an approved standard. Here is what an approval trail would need to look like." |
| "Tell me about a remediation you chased to closure." | nothing to tell | do not invent one. Offer the closest true thing: a technical issue you owned to closure, described as that rather than as a security remediation. |
| "What is your budget experience for a red-team programme?" | none | "None for red teaming. I can show you the scoping and reporting design and tell you what I would expect to cost, flagged as an estimate." |
| "Why should we put you in front of the regulator?" | you have not been | "You should not yet. Here is the gap and here is how I would close it in the first year." |

### 10.3 "I wrote the standard" versus "I would write the standard"

These two sentences differ by one word and by the entire accountability gap. The first claims an institutional act that did not happen, and an interviewer with a security background will test it in three follow-up questions - who approved it, who objected, what changed between draft three and the signed version - and the claim will collapse, taking the rest of the interview with it.

The second is acceptable, and it is acceptable precisely because it is said plainly. A candidate who says "I have not owned a standard; I have written one in public and I know what the approval path requires" has told the interviewer something true and unusual: that they know the difference. In a role whose whole content is the difference, that is worth more than a confident fabrication, and it is the only version of the claim that survives diligence.

### 10.4 What to bring to the room

The instinct is to bring the corpus, and the instinct is wrong at the wrong scale. A 686-file repository presented as evidence reads as a hobby, and a panel that asks "which one should I read" and receives a directory listing concludes that the candidate cannot prioritise. Three artefacts, chosen for what they prove, are worth more than the whole library.

- **One control library extract**, printed, with the failure mode, the control, the test and the exception route on a single page. It demonstrates the standards work directly and it invites a technical challenge the candidate can win.
- **One threat model of a system the panel knows something about** - a RAG assistant is the safest choice, because every bank now has one. It demonstrates review capability and gives the panel something to push against.
- **One red-team scoping document** for an exercise that was not run, clearly labelled as a design rather than a result. It is honest, it is specific, and it does the work that gap 5 needs doing.

What not to bring: the framework comparison tables. Every candidate has read the frameworks. The differentiator is not knowing NIST AI RMF; it is being able to say what a bank should do differently from the framework's default, which is what s4 of the governance guide argues and what a panel will test.

---

## 11. A staged plan

Two horizons. The first is what can be done before the role, where the constraint is that none of it confers authority. The second is what to do if the role is offered, where the constraint is that credibility in the first quarter is spent, not earned.

### 11.1 Horizon one: the 6-12 months before

Five actions, each tied to a numbered gap in section 7, each with an output someone else could inspect.

| Action | Gap | Checkable output |
|:--|:--|:--|
| publish an AI security standard internally with a named approver and a review date | 3 | the document, its approval record, its review date |
| run one red-team exercise against a real system | 5, 6 | scoping document, findings register, remediation log with dates, retest result |
| operate one pre-production security review, end to end, for a real release | 4, 6 | a decision record with conditions and a closure note |
| build and deliver one AI security awareness module to a real cohort | 7 | the module, the attendance list, a pre/post delta |
| book and sit CISSP (or the chosen equivalent) with a date in the diary | 2 | a booked exam, then a pass |

Two notes on sequencing. The red-team exercise and the pre-production review should be the same system if possible, because the exercise produces findings and the review produces the remediation path, and doing both on one system produces the only artefact on this list that resembles a programme. The awareness module is the cheapest and should not be deferred behind the others.

What none of these five does: give the subject a security title, a veto, or a mandate. They convert four NOT EVIDENCED rows into evidenced experience and one screening gap into a pass. That is the maximum available from outside a security function, and it is worth having.

### 11.2 Horizon two: the first 90 days inside the role

If offered, the first quarter has one job: find out what already exists before adding anything.

**Do first.**

1. **Inventory.** What AI systems are in production, in pilot, and in procurement; what standards exist and who owns them; what reviews are already happening under another name. `AI inventory, AI register, AI use case inventory` appears in 11 files here, which is a warning: the artefact is usually thinner in reality than in the plan.
2. **Gap analysis of the taxonomy and control library against what exists.** Not a rewrite - a diff. Present the diff, not a new library.
3. **Observe the review board for a full cycle before speaking in it.** The purpose is to learn the decision culture: who defers to whom, what a real objection sounds like, which conditions get honoured.
4. **Baseline the red-team position.** What has been tested, by whom, with what findings still open. The baseline is usually worse than assumed and the number is the argument for the programme.
5. **Baseline awareness.** Who has had any AI security training, in what form, measured how. Expect the answer to be "none, informally".

**Do not.**

- **Rewrite the strategy in week two.** A new joiner's strategy is a resignation letter with a delay. The first version should be a diff of the existing one.
- **Run a red team before a remediation path exists.** Findings without a route to closure convert a technical problem into a reputational one, and the security function gets the blame for both.
- **Publish a standard nobody signed.** An unsigned standard is worse than no standard, because it lets people claim compliance with something no one owns.

**Kill criterion.** If by day 90 there is no route to a decision right - no forum where the role's security position is recorded and can stop a release - then the role is advisory despite its title, and the accountability gap this guide describes will not close in it. That is worth knowing at day 90 rather than at month 18, and it is the one thing in this plan that should not be negotiated.

### 11.3 What the plan does not do

Two limits, stated so the plan is not mistaken for a route to the role.

**It does not make the candidate competitive for this role as written.** It makes them competitive for an AI security role with a development remit, or for this role if the panel is willing to trade accountability for demonstrated knowledge and a credible first-year plan. Whether that trade is available is a property of the requisition, and the guide does not claim to know it - section 14 rejects the hiring question for exactly this reason.

**It does not compress.** The five actions in 11.1 are sequenced because each depends on the previous one being real: the exercise produces findings, the review produces the remediation path, the remediation path is what makes the awareness module concrete rather than generic. Running them in parallel produces four documents instead of one artefact chain.

What the plan does do is change the sentence a candidate has to say. Today it is "I have written about this extensively". After twelve months of the plan it is "I ran one exercise, operated one gate, delivered one module, and here is what I got wrong". The second sentence is shorter and worth considerably more.

## 12. The anti-patterns of self-assessment in this genre

Eight failure modes, each of which this guide has had to avoid in the writing and each of which a reader should check it against.

**1. Claiming writing as operation.** The characteristic failure. A 709-line guide about red teaming is evidence of understanding red teaming, and the sentence "I have written extensively about AI red teaming" is true and misleading in the same breath if the next sentence does not say no programme was run. The repair is a habit: every claim of knowledge gets a companion clause naming what it is not.

**2. Confusing familiarity with a framework for having applied it.** Knowing the structure of NIST AI RMF is not the same as having run a NIST-based assessment inside an organisation with a deadline and an auditor. The corpus is strong on framework structure and silent on framework application, and the two are separated by everything in section 6.

**3. Treating a topic list as a capability.** A table of contents that names prompt injection, jailbreaks, poisoning, extraction and agent abuse looks like coverage and is a list of headings until each heading has a control, a test and an owner. `ai_red_teaming_guide.md` mostly escapes this because its sections have substance; a thinner document would not.

**4. Counting files as coverage.** The 686-file corpus invites exactly this error, which is why section 2 pairs every strong count with a thin one and why section 2.5 prints the seven dispatch figures that failed to reproduce. A count is a measurement of how often a word appears, and the frequency of a word is not the quality of a treatment.

**5. Mistaking a research pass for a programme.** A research pass has a beginning and an end and produces documents. A programme has neither and produces decisions. The clearest test: does anything break if it stops? Nothing breaks when a research pass stops.

**6. The flattery reflex.** Every gap reframed as a strength - "no security title, but that means fresh perspective"; "no certification, but certifications do not measure capability". Both statements can be true and both are used to avoid saying the gap is a gap. This guide tries to hold the line: the certification gap is a real screening cost, and it is also genuinely not a competence verdict. Both, in that order.

**7. The sandbag reflex.** The twin failure, and the one this guide is more at risk of. Discounting genuine strengths to seem rigorous - treating 84 files on AI governance as "just writing", treating a control library method as boilerplate - is as inaccurate as inflating them, and it produces advice that is useless in the opposite direction. Section 8 exists to hold that line.

**8. The one specific to this guide: reading "55 files on prompt injection" as "has defended a system against it".** The number is real, the knowledge is real, and the defence is absent. Prompt injection is the canonical example because it is the attack class most written about and least reliably defended in practice: a person can describe the attack taxonomy in detail and still have never had to decide whether a production system's retrieval pipeline was safe to ship. Section 4.3 and 4.4 both make this point; it is repeated here because it is the error a reader is most likely to make on the author's behalf.

### 12.9 Why the genre is structurally prone to all eight

The failures are not accidents of this particular document. They follow from the form. A skill-gap guide is commissioned by the person being assessed, written about that person's evidence, and read by that person first - which means the incentives run toward encouragement. Encouragement is not dishonest, but it selects for the flattering reading of every ambiguous fact, and ambiguity is everywhere in a corpus of 686 files.

The one structural defence available to a document is to make its claims checkable by someone other than its subject. That is what the measurement date, the command form, the reproducibility table and the rejected rows are for: they let a reader who was not in the room re-run the argument and disagree with it on evidence rather than on taste.

---

## 13. The one-page verdict

| Area | Verdict | Gap type |
|:--|:--|:--|
| 1. Strategy and architecture | PARTIAL | accountability |
| 2. Risk management | MET as knowledge, not as mandate | experience |
| 3. Standards and controls | MET as knowledge, thin on hosting and agents | knowledge |
| 4. Red teaming and assurance | PARTIAL | accountability |
| 5. AI in cyber defense | NOT EVIDENCED | knowledge |
| 6. Awareness and capability | NOT EVIDENCED | experience |
| 7. Governance and stakeholders | PARTIAL | accountability |

### 13.1 Four things that help

- The risk-register and control-library method is a deliverable-grade artefact, not a summary (`ai_risk_register_guide.md` s3-s5).
- Framework fluency is operationalised into a bank-runnable stack, not recited (`ai_governance_framework_guide.md` s2, s4, s9).
- The red-team process, attack taxonomy and OWASP-mapped risk landscape are the vocabulary the standards work needs (`ai_red_teaming_guide.md` s2, s8; `llm_development_risks_security_guide.md` s2-s12).
- Model-risk and three-lines fluency means the role's negotiation with the second line starts from shared language (88 files on SR 11-7 and model risk management; `ai_governance_framework_guide.md` s5.5, s6).

### 13.2 Four things that do not

- No security title, no security function, no security accountability at any point in the career.
- No security certification, in a market that screens on one.
- No owned strategy, standard, or review decision - the artefacts that make the role a role.
- No programme run: no red-team calendar, no findings register, no awareness cohort, no remediation log.

### 13.3 Verdict

This is a strong candidate for the **knowledge** half of an AI Security Architect role and an unevidenced candidate for the **authority** half, and the job description is roughly half of each. The corpus would let this person write the standards on day one and defend them in a design review by day thirty; it would not let them stop a release on day one, because stopping a release is a right granted by an institution and not a skill demonstrated in a repository.

The fastest real improvement is not more writing. It is one red-team exercise, one operated review gate, and one certification with a date - after which the conversation changes from what the candidate knows to what the candidate has done once.

The thesis, tested and qualified: the gap is not what you know about AI security, it is what you have been accountable for - with the qualification that areas 2 and 3 are genuinely knowledge requirements and are genuinely met, which is why the guide does not claim the whole job is a mandate.

---

## 14. The claims audit

Every material claim in this guide, with its status and the method used to check it. The counts are the measurement pass of 12 November 2026 taken for this guide; the certification and career facts are from the two source files in `personal/`.

| Claim | Status | Source | Date | Methodology | Quality note |
|:--|:--|:--|:--|:--|:--|
| Repository scale: 686 .md files | ✓ VERIFIED | repo root | 2026-11-12 | find and count by extension | deterministic |
| Directory distribution: technology 425, banking 165, management 57, personal 9, ai_llm 103, rag 20 | ✓ VERIFIED | repo root | 2026-11-12 | per-directory count | sums consistently below the total |
| Strong vocabulary band (27 to 227 files) | ✓ VERIFIED | repo | 2026-11-12 | `grep -rlEi "<term>" --include='*.md' . \| wc -l` | file counts, not occurrences |
| Thin vocabulary band: ai security 16, data poisoning 7, model extraction 6, model manipulation 1, model hosting 8, inference security 0, agent abuse 0, security awareness 4, secure SDLC 7, security strategy 1 | ✓ VERIFIED | repo | 2026-11-12 | same command form | the load-bearing numbers of the guide |
| Seven dispatched figures did not reproduce | ✓ VERIFIED as a negative | repo | 2026-11-12 | re-measurement shown in 2.5 | printed rather than silently corrected |
| Case-insensitive CISM returns 113 files only via "criticism" | ✓ VERIFIED | repo | 2026-11-12 | substring match inspection | 20 files with word boundaries |
| Eight dedicated guides, 679 to 900 lines each | ✓ VERIFIED | repo | 2026-11-12 | `wc -l` | section maps read, not inferred |
| Five adjacent guides cited, 670 to 1058 lines | ✓ VERIFIED | repo | 2026-11-12 | `wc -l` | `cybersecurity_guide.md` s5.5 noted as section-level only |
| No security certification of any kind | ✓ VERIFIED | `jack_liu_profile.md` lines 71-90; `jackliusr_digital_footprint.md` lines 310-321 | 2026-11-12 | direct read of both lists | six certs recorded, none security |
| Claude Certified Architect year: 2026 in one source, approximately 2025 in the other | ⚠ FLAGGED | profile line 81; footprint line 315 | 2026-11-12 | cross-read | unresolved discrepancy, printed as one |
| ST Electronics appears twice: Software Engineer approximately 2003-2008, and Team Leader April 2010 to May 2012 | ⚠ FLAGGED | `jack_liu_profile.md`; `jackliusr_digital_footprint.md` | 2026-11-12 | cross-read | internal contradiction in the sources; printed, not resolved |
| Current employer recorded by name in the sources | ⚠ FLAGGED | footprint 7.1 | 2026-11-12 | read and withheld | withheld by editorial rule; not repeated here |
| The repository demonstrates AI-security knowledge | ✓ VERIFIED as documented knowledge | whole corpus | 2026-11-12 | counts plus section reading | ⚠ depth in any single area varies widely |
| The subject has operated an AI red-team programme | ✗ REJECTED | profile; footprint | 2026-11-12 | searched for calendar, findings, vendor, register | declined: methods written, no programme artefact exists |
| The subject has approved AI architectures from a security perspective | ✗ REJECTED | profile | 2026-11-12 | searched for decision records and security remit | declined: approval is an institutional right never evidenced |
| The subject has held a security title | ✗ REJECTED | profile; footprint | 2026-11-12 | full role list read | declined: no security function at any point |
| The subject would or would not be hired | ✗ REJECTED | n/a | 2026-11-12 | not assessable | declined: depends on the requisition, the panel and the competition |
| No Bugcrowd or HackerOne account, Keybase only, no Kaggle, no patents, no publications, no talks | ✓ VERIFIED | footprint, sections 6-7 | 2026-07-08 | platform searches reported in source | negative findings at the footprint's scope |
| GitHub: 166-173 public repos, 12 stars, 19 forks | ✓ VERIFIED | footprint | 2026-07-08 | API and page count | range given as recorded |

### 14.1 What the audit does not license

A verified count is not a verified capability, and the table above is deliberately split so that the two cannot be confused. Every ✓ row about the corpus establishes that a term appears in N files. None of them establishes that the treatment is good, current, or the author's own work rather than a summary of someone else's. Section 2.8's asymmetry claim is the interpretive step, and it is labelled as an interpretation rather than a measurement.

### 14.2 What to do with a flagged row

Two rows above are flagged rather than verified, and neither should be resolved by this document. The Claude certification year and the ST Electronics period are discrepancies inside the sources, and only the subject can settle them. Until they are settled, any sentence this guide writes about them carries both readings, which is what the flag means.

One flagged row is different in kind: the current employer. That is not a discrepancy but a deliberate withholding under this repository's editorial rule, and it is recorded here so that a reader comparing this guide against its sources knows the omission is intentional rather than an oversight.

---

## 15. What Could Not Be Verified, the glossary and the cross-references

### 15.1 What could not be verified

1. **Whether the current role carries any security accountability at all.** The profile says GenAI solutions and architecture governance. It does not say security, and it does not say the absence of security. This is the single most consequential unknown in the guide.
2. **Whether any architecture review the subject performed carried a security veto, or any veto at all.** A solution architect's review is often a design review with no refusal right. The distinction changes gap 4 from "not closable" to "partly closable".
3. **The size and structure of any team the subject has led or worked in.** The profile records team-lead titles at two earlier employers and gives no headcount anywhere. Every statement about scale in this guide is about documents, not people.
4. **Whether the current institution has an AI security function the subject worked alongside.** If it does, the subject has observed the function at close range, which is a different evidential position from having read about one. The sources are silent.
5. **The real employer's technology stack, security organisation and practices.** Deliberately out of scope, and withheld by editorial rule. Nothing in this guide characterises that institution.
6. **The correct ST Electronics period.** The sources give two overlapping and mutually inconsistent accounts. One of them is wrong and it cannot be determined which from the material available.
7. **Whether the security-certification absence is a fact about the world or a fact about the sources.** The guides record what the sources record. An unlisted certification is not a non-existent one, and this is a negative finding at the sources' scope, not proof.
8. **Whether any red-team exercise was ever run informally** - a workshop, an internal game day, a capture-the-flag. Informal exercises leave no public trace and would not appear in either source.
9. **The true circulation of the written guides.** Stars and forks are recorded (12 and 19 across 166-173 repositories). Whether any of the AI-security material was read by a practitioner, adopted, or cited is unknown, and adoption would be the only evidence in the whole audit that the writing had institutional effect.

### 15.2 Glossary

| Term | Meaning as used here |
|:--|:--|
| AI security architecture | the set of design decisions that determine an AI system's exposure: trust boundaries, data flows, tool scope, model and weight protection, logging |
| AI risk taxonomy | a structured classification of AI-specific failure and attack modes, used to make risk comparable across use cases |
| control library | the catalogue of controls with owners, test procedures and failure mappings that a taxonomy points into |
| design authority | the forum holding the right to approve, condition or refuse a design before build or release |
| red team | an adversarial assessment performed by people tasked to break the system rather than to confirm it works |
| assurance gate | a pre-production checkpoint where evidence is reviewed and a deployment decision is recorded |
| model hosting | the infrastructure and configuration that serves a model: endpoints, isolation, tenancy, keys, weights |
| inference security | the security properties of the serving path specifically - the phrase appears in 0 files in this corpus |
| agent abuse | adversarial manipulation of an AI agent's tools, memory or permissions to achieve an unintended action; 0 files as a phrase |
| model extraction | recovering a model's behaviour or parameters through repeated querying; 6 files |
| data poisoning | corrupting training or retrieval data to influence model behaviour; 7 files |
| prompt injection | instructions smuggled into model input, directly or via retrieved content, that override intended behaviour; 55 files |
| jailbreak | eliciting prohibited output from a model through prompt craft or optimisation; 30 files |
| secure SDLC | a development lifecycle with security activities embedded at each stage rather than appended; 7 files |
| three lines of defence | the operating model separating delivery, risk oversight and independent audit; 45 files |
| AI RMF | the NIST AI Risk Management Framework, used here as a structuring device for AI risk work; 23 files |

### 15.3 Cross-references

Precedent files in this directory, as siblings:

- `data_architect_skillgaps_guide.md`
- `skill_gaps_enterprise_architect_guide.md`
- `senior_data_platform_architect_guide.md`
- `agent_runtime_engineer_skill_gaps_guide.md`
- `ai_products_experience_summary.md`

The eleven AI-security guides this analysis rests on, relative from `personal/`:

1. `../banking/ai_risk_register_guide.md` - 816 lines
2. `../technology/ai_verify_guide.md` - 706 lines
3. `../technology/ai_llm/prompt_injection_guide.md` - 679 lines
4. `../technology/ai_llm/ai_governance_bias_redteaming_guide.md` - 763 lines
5. `../technology/ai_llm/ai_red_teaming_guide.md` - 709 lines
6. `../technology/ai_llm/ai_governance_framework_guide.md` - 707 lines
7. `../technology/ai_llm/ai_verify_toolkit_guide.md` - 900 lines
8. `../technology/ai_llm/frontier_ai_risk_guide.md` - 865 lines
9. `../technology/adversarial_ml_attacks_guide.md` - 1058 lines
10. `../technology/llm_development_risks_security_guide.md` - 974 lines
11. `../technology/security_by_design_guide.md` - 720 lines

Two further adjacent files cited in section 2.7: `../technology/cybersecurity_guide.md` (670 lines) and `../banking/ai_genai_banking_compliance_guide.md` (717 lines).

---

## 16. The closing summary

Thirty-six requirements across seven areas, and the verdicts split cleanly along a line the job description does not draw but the organisation does. Where the requirement is a method - a taxonomy, a control library, a framework stack, a rating scale, a risk-register design - the corpus meets it, in several places at a standard that would survive a security function's review. Where the requirement is a mandate - a strategy owned, a review board sat on, a programme run, a report delivered to people who can act on it, an SME designation, a regional remit - nothing in the profile or the corpus reaches it, and the reason is structural rather than personal: those objects are made of other people's obligations.

The qualification this guide insists on, against its own thesis: roughly half of this job description is written as analytical work, and the subject can do that half now. The thesis is true about the other half, and the other half is what the title is for.

What closes the distance is not more writing. One red-team exercise against a real system, one operated pre-production gate with a decision record, one delivered awareness module with a measurement, one certification with a date. Four actions, all checkable, none of which confers authority and all of which convert silence into evidence.

The order matters, and it is the reverse of the comfortable order. The certification is the easiest and the least valuable; the exercise is the hardest and the most valuable, because it is the only one of the four that produces a finding someone had to fix. A candidate who does them in order of ease will have a certificate and three documents. A candidate who does them in order of value will have a story about a system that failed a test and was fixed before release.

What cannot be closed from outside a security function is the part that matters most: a title that puts the subject's judgement between a system and its users, and an institution that has agreed to be bound by it.

**The gap is not what you know about AI security; it is what you have been accountable for - and this job description is written as an accountability, not a knowledge requirement.**
