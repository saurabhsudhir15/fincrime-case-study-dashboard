# FinCrime Case Study: Synthetic Identity Fraud

**Context:** Anonymized case study for an Indian SME neobank, 2026.

**[View interactive dashboard](https://saurabhsudhir15.github.io/fincrime-case-study-dashboard/)**

**Problem Statement:** Over the past 3 months, the fincrime team observed a 25% increase in suspicious activity alerts on new accounts. The review team is managing a growing backlog, false positives are rising, and legitimate SME members are experiencing unnecessary friction at onboarding. The increase is real, the operational pressure is compounding, and the gap being exploited has not yet been closed. This report diagnoses the root cause and proposes four targeted interventions.

---

## 1. Problem Scoping: Sizing the Spike and Identifying the Right Signals

**Validating the signal:** The first step is confirming the alert increase reflects real fraud growth and not an artifact of a system change.

- A new detection rule, an updated alert definition, or a logging modification would produce the same signal without any real fraud growth.
- Cross-reference alert volume against any rule or model changes in the prior 90 days before proceeding.

### What to measure: scope and fraud signature metrics

| Key Category | Metrics | Why it matters |
| :---- | :---- | :---- |
| **Scope metrics** | New accounts opened per week (13-week trend) | Establish baseline and identify the exact inflection point |
| | Alert rate on new accounts | Rising rate with no rule change confirms real fraud escalation, not a detection sensitivity shift |
| | False positive rate among flagged accounts | High FPR means analysts are spending capacity on legitimate accounts |
| | Review backlog: total cases and median age | Backlog age reveals team capacity pressure. Will not clear until the upstream verification gap is closed |
| | True positive rate: confirmed synthetic of investigated accounts | Stable TPR with rising alert volume confirms more real fraud is entering the system, not just more alerts |
| **Fraud signature metrics** | Time from account creation to first transaction | Tight early distribution in synthetic cohort confirms accounts are activated for immediate use |
| | Time from account creation to credit or loan application | Synthetic accounts optimised to reach credit fast; distribution shape directly informs the velocity gate threshold |
| | Business registration age at account creation (MCA, GSTIN, Udyam) | New or absent business registration is the strongest discriminating signal between synthetic and legitimate accounts |
| | Device fingerprint sharing: Device ID captured at SMS verification step | Enables cross-account clustering — a reliable ring detection anchor |
| | IP clustering: shared registration IPs across accounts | Shared infrastructure across accounts confirms coordinated rather than individual fraud |

### Where the data lives

- Internal systems: account creation and KYC records, alert management system, case management system, device and session logs, transaction history
- Credit bureau data (bureau profile at PAN level: thin file or no-hit signal)
- Business registries: GST Portal (PAN > GSTIN), MCA Portal (PAN > DIN > company details), Udyam
- Historical confirmed synthetic fraud cases (behavioural baseline for cohort comparison)

### Analysis to run

| Analysis | Methodology and objective |
| :---- | :---- |
| **Trend analysis** | Plot alert volume over 13 weeks. Identify the exact inflection point and what changed at that moment. |
| **Cohort comparison** | Confirmed synthetic accounts vs a matched legitimate cohort. Identify which signals most strongly discriminate between the two groups. |
| **Network analysis** | Cluster flagged accounts by shared device ID and IP — reliable anchors for ring detection. |
| **Velocity analysis** | Distribution of time to credit application. Synthetic accounts are optimized to reach credit fast. A tight, fast distribution in the synthetic cohort confirms the credit incentive pattern. |

---

## 2. Hypothesis and Root Cause Analysis

### Map the full space of explanations first

| Internal (Platform side) | External (Fraud environment) |
| :---- | :---- |
| A new detection rule or threshold change was deployed recently, generating more alerts without actual fraud growth | A large data breach made real PAN and Aadhaar combinations available to fraudsters at scale |
| Onboarding friction was reduced to improve legitimate conversion, inadvertently lowering the bar for synthetic accounts | An organized ring is specifically targeting this platform after competitors tightened controls |
| A new credit or lending product was launched, making accounts more valuable to fraudsters | Industry-wide increase in synthetic identity fraud across Indian fintechs following UPI scale |
| A specific verification gap in onboarding was discovered and is now being exploited systematically | |

### Primary hypothesis

The 25% alert increase is driven by an organized network exploiting a gap in business account verification. Synthetic accounts pass individual KYC because the identity documents are real. The business details are fabricated. The gap is at the business registry layer, not the identity layer.

This explains three signals:
- The sudden jump is consistent with organized exploitation of a known gap
- The fraud is designed to pass individual KYC, so the vulnerability must be downstream
- The FPR increase follows from the same gap: real individual identity makes automated rejection difficult, pushing borderline cases to human review

**Three tests to confirm or reject:**

- Test 1: Combined registry absence rate (no GSTIN, DIN/CIN, or Udyam match) in synthetic vs legitimate cohort. Higher absence in synthetic cohort confirms the business registry gap.
- Test 2: Device and IP clustering across flagged accounts. Presence confirms organized ring hypothesis.
- Test 3: Inflection point correlation with any platform product or onboarding change. If correlated, the registry gap pre-existed but became more exploitable after friction was reduced.

**What would invalidate it?** If flagged accounts show diverse, non-clustered profiles, the organized ring hypothesis falls and the problem shifts to individual actors. If combined registry absence is equally common across both cohorts — plausible for solo traders operating informally — the analysis shifts to behavioural signals: velocity, transaction patterns, bureau profile.

---

## 3. Recommendations

The review team is spending capacity on accounts that are individually ambiguous but collectively show clear network patterns. The FPR burden is not a calibration problem — it is a structural consequence of verifying identity without verifying business substance.

### Recommendation 1: Two-Layer Business Registry Cross-Reference (Highest Priority)

Business verification is absent at account creation. A two-layer async approach fixes this without adding onboarding friction.

**Layer 1** — silent PAN-based registry lookup runs at account creation (PAN > GSTIN, PAN > DIN > MCA company details). Zero new fields. Zero impact on the 3-minute onboarding flow. Credit features restricted for accounts with no registry anchor.

**Layer 2** — registry confirmation (GSTIN, CIN, or Udyam number) collected at credit application where the user is motivated to provide it. A legitimate business will have at least one registry anchor. A synthetic account with fabricated details will almost certainly have none.

### Recommendation 2: Network Graph Scoring

Build an identity graph linking accounts by shared device ID and IP. Flag accounts within N degrees of a confirmed synthetic account with a risk multiplier so they surface first in the review queue. A single account with no registry match is ambiguous. That same account sharing a device with confirmed synthetic accounts is not.

### Recommendation 3: Velocity-Based Credit Access Control

Require a minimum account age and transaction history before credit or loan applications are processed. The exact threshold is determined from velocity analysis — specifically the P90 of time-to-credit in the confirmed synthetic cohort. Synthetic accounts are optimized to reach credit fast. Setting the gate above their typical velocity removes the primary incentive without gating legitimate users.

### Recommendation 4: Eval Framework for Detection Health

Three concrete actions:
1. Build a golden set — pull closed cases from the case management system (confirmed fraud and confirmed clean), label them, and refresh as new cases are resolved.
2. Run the detection system against this set weekly and after every rule or model change, tracking precision and recall separately.
3. Set a recall floor — if recall drops below a defined threshold, trigger a model review.

Most fincrime dashboards show alert volume and FPR, which reflect precision. Recall is invisible unless measured deliberately. A system with high precision and low recall looks healthy while real fraud slips through silently. The golden set makes that gap visible before it compounds.

---

## Appendix

### Appendix A: Business Registration Age — Data Sources

- MCA Portal: PAN > DIN > CIN > company details (date of incorporation, company name, address, status — covers Pvt Ltd, LLP, OPC where applicant is a registered director)
- GSTIN via PAN lookup: GST registration date as proxy for business activity
- Udyam number at credit application: date of incorporation and commencement of business

### Appendix B: Cohort Comparison Queries

| Query | What to measure |
| :---- | :---- |
| Time to first transaction | Compare median and distribution (P25/P75). Does synthetic cohort cluster tighter and earlier? |
| Time to credit application | Same distribution comparison. A tight, fast distribution in the synthetic cohort confirms the credit incentive pattern. |
| Bureau profile | Percentage of no-hit or thin-file accounts in each cohort. |
| Device sharing rate | Percentage of accounts sharing a device ID with at least one other flagged account. |
| IP clustering | Percentage of accounts with a shared registration IP across the cohort. |
| Combined registry absence | Percentage with no anchor across all three registries (GSTIN, MCA, Udyam) in each cohort. Expected to be the strongest discriminating signal. |
| Business registration age | Distribution of business age at account creation across both cohorts. |

### Appendix C: Clarifying Questions and Working Assumptions

**Clarifying questions:**
1. Did the alert increase happen uniformly across all acquisition channels or account types?
2. Were there any changes to the onboarding flow, detection rules, or KYC steps in the past 3-4 months?
3. Are flagged accounts passing all individual KYC checks?
4. Is device or IP clustering already visible in the flagged cohort?
5. What is the false positive rate on these alerts vs the historical baseline?
6. What credit or lending products are most represented among flagged accounts?
7. Has FIU-IND issued any recent advisories on synthetic identity fraud?

**Working assumptions:**
1. The 25% increase reflects real fraud growth, not a detection sensitivity change
2. Flagged accounts are largely passing individual KYC — the gap is at the business verification layer
3. The inflection point does not correlate with a new product launch — this is external escalation
4. Clustering is likely present — a 25% jump over 3 months is consistent with organized exploitation
5. This is platform-specific, not industry-wide
6. The business registry verification workflow (MCA, GSTIN, Udyam) is designed for the Indian regulatory context. The framework is replicable in other markets with equivalent local business registries.

### Appendix D: Recommendation 1 — Implementation Detail

**Layer 1 (async, at account creation):**
- PAN > GSTIN via GST portal
- PAN > DIN > CIN > MCA Portal company details
- No match on either: credit restricted, behavioural signals weighted more heavily

**Layer 2 (at credit application):**
- Extend existing GSTIN collection to include CIN or Udyam number
- Udyam additionally returns date of incorporation

**Risk tiering:**
- Match and consistent: standard flow
- Match and inconsistent: flag for review
- No match: enhanced due diligence, credit restricted

### Appendix E: Success Metrics and Risk Register

**Success metrics:**

| Recommendation | Metric | Target |
| :---- | :---- | :---- |
| Rec 1: Registry Cross-Reference | FPR | Declining toward pre-spike baseline within 90 days |
| | Registry absence rate in synthetic cohort | Increasing — more ring accounts surfaced post integration |
| | Registry match rate in legitimate cohort | Stable or improving — no unintended friction on real businesses |
| Rec 2: Network Graph Scoring | First-degree network precision | Above 70% of network-flagged accounts confirmed synthetic after review |
| | Analyst time-to-decision per case | Declining — ring accounts surfacing higher in the review queue |
| Rec 3: Velocity Gate | Legitimate user deferral rate | Below 5% of all credit applications |
| | Time-to-credit distribution | Shifts right within 90 days |
| Rec 4: Eval Framework | Detection recall | Measurable and above 75% within 90 days of golden set reaching 200-300 confirmed cases |
| | Precision and recall tracking | Separated for rules engine and ML model, tracked independently |
| | Out-of-cycle retrain trigger | Activated when recall drops more than 5pp below defined floor |

**Risk register:**

| Recommendation | Risk | Likelihood | Impact | Mitigation |
| :---- | :---- | :---- | :---- | :---- |
| Rec 1 | MCA and GST Portal API downtime disrupts credit access flow | Medium | High | Fail open on account creation, fail conservative on credit access; circuit breaker with retry queue; cache confirmed results for 90 days |
| | Legitimate sole traders with no GSTIN, DIN, or Udyam blocked from credit | Medium | Medium | Credit restricted not blocked; Udyam number at Layer 2 covers micro enterprises below GST threshold |
| | DPDPA compliance risk on PAN data processing | Medium | High | Data minimisation at collection; consent documented at account creation; audit trail for all registry lookup events; legal review before go-live |
| Rec 2 | Legitimate users on shared devices or IPs caught in network flag | Medium | Medium | Risk multiplier raises queue priority — does not auto-reject; N-degree expansion to second degree only after first-degree precision validated above 70% |
| Rec 3 | Legitimate fast movers needing immediate credit gated by velocity threshold | Low | Medium | Soft deferral not hard block; UX message sets expectation with unlock date; P90 of synthetic cohort used as threshold |
| | Sophisticated rings adapt by waiting past the velocity gate | Medium | Medium | Rec 3 operates in combination with Rec 1 and Rec 2 — rings that wait are still caught by registry absence and network scoring |
| Rec 4 | Survivorship bias in golden set | High | High | Include confirmed clean cases and fraud cases missed initially but discovered later; quarterly refresh of 30-50 new cases |
| | Insufficient confirmed fraud cases to reach minimum golden set threshold | Medium | Medium | Start with available cases and quantify confidence interval on recall estimates; flag uncertainty to stakeholders until 200-300 confirmed cases are reached |
