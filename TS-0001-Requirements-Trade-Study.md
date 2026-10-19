\# TS-0001: Requirements Trade Study (Sprint 1 Baseline)



\## 1. Document Control \& Metadata

\* \*\*Document ID:\*\* NTX-QUEST26-DFTA-TS-002

\* \*\*Version / Date:\*\* 2.1 (RFP-based revision) / 17 October 2026

\* \*\*Project Name:\*\* AI-Powered Digital Forensics Assistant for Medical Devices

\* \*\*Controlling Source:\*\* NovaTechX QUEST 2026 RFP — AI-Powered Digital Forensics Assistant for Medical Devices

\* \*\*Authors:\*\* Babar Yousaf, Rooshan Salim Butt, Atif Javed

\* \*\*Affiliation:\*\* National University of Technology, Nutech Research Center, Islamabad, Pakistan

\* \*\*Decision Status:\*\* Proposed architecture baseline; team approval and prototype verification remain pending

\* \*\*Target Audience:\*\* Gate 1 Review Panel \& Claude V\&V Gate Reviewer



\---



\## 2. Executive Summary

This trade study establishes the comprehensive system-level architectural baseline for the \*\*AI-Powered Digital Forensics Assistant for Medical Devices\*\* ahead of the Gate 1 review. The system must ingest highly heterogeneous evidence, process data under extreme load constraints, and deliver traceable forensic results while maintaining a strict, non-negotiable air-gapped medical boundary. 



All mandatory constraints, requirement references, and numerical acceptance targets in this study come from the supplied NovaTechX QUEST 2026 RFP. Because the RFP states targets but does not provide prototype test results, unmeasured performance is explicitly left unverified at this stage. This document outlines the open choices, trade-offs, and verification gates required to close each engineering decision.



\---



\## 3. Scope \& Non-Negotiable RFP Constraints

This study treats the assistant as one integrated system, evaluating five open implementation decisions. The following parameters represent pass/fail gates and are treated as non-negotiable criteria:



\* \*\*Medical-Device Boundary:\*\* Passive tap/mirror monitoring only; no device write, command, configuration path, or outbound traffic to monitored devices (Trace: FR-09, SF-01, EN-04).

\* \*\*Evidence Coverage:\*\* Ingest all six core sources; document each connector with an Interface Control Document (ICD); disclose unavailable sources and partial coverage (Trace: FR-01, FR-29/30, IF-05, SF-05).

\* \*\*Evidence Format \& Integrity:\*\* Normalize to version-controlled MDFE (FHIR R4 primary); use cryptographic hashes, WORM preservation, trusted time, chain of custody, and verifiable export (Trace: FR-04, FR-10, IF-01/02, DP-07/08).

\* \*\*Detection \& Safe Degradation:\*\* Deterministic rule/signature tier plus purpose-built probabilistic anomaly tier; expose degraded state and preserve tier-1 operation when the AI budget is exceeded (Trace: FR-06, PR-07, SF-04, RL-04).

\* \*\*Human Oversight:\*\* Trace every AI conclusion to evidence; visibly separate observed facts from inferences; require review for specified high-criticality / low-confidence findings (Trace: FR-11/14/22, US-02, SF-07).



\---



\## 4. Numeric Acceptance Targets

The proposed architecture must be engineered to hit and verify the following performance metrics:

\* \*\*Detection Performance:\*\* Sensitivity $\\geq$ 0.90; $\\leq$ 1 false positive per 1,000 device-days for high-criticality devices (Must be demonstrated on versioned VV-02 corpus, including infusion-pump drug-library tampering).

\* \*\*Finding Latency:\*\* $\\leq$ 60 seconds at the 95th percentile ($p\_{95}$) under nominal load for hospital-sourced events.

\* \*\*Load Constraints:\*\* 5,000 events/second/site; 20,000 concurrent device identities sustained without evidence loss.

\* \*\*Outage / Recovery:\*\* No loss across a 24-hour upstream-source outage; Recovery Point Objective (RPO) 0; Recovery Time Objective (RTO) $\\leq$ 4 hours.

\* \*\*Investigation Queries / Reports:\*\* 30-day report $\\leq$ 5 minutes; 90-day timeline query $\\leq$ 3 seconds at $p\_{95}$ full query window.

\* \*\*Reliability / Source Health:\*\* 99.5% monthly availability (excluding scheduled maintenance); alert when source silence exceeds 15 minutes.

\* \*\*Observed-vs-Inferred Labels:\*\* $\\geq$ 85% agreement provisional target using an independent reviewer protocol.



\---



\## 5. Detailed Trade Studies



\### Trade Study 1 — Probabilistic Detection Model

\* \*\*Decision Question:\*\* Which model should implement the probabilistic tier while the RFP-required deterministic tier remains in place?

\* \*\*Evaluation Matrix:\*\*



| Named Alternative | RFP-Relevant Fit | Main Limitation / Risk | Trade-Study Disposition |

| :--- | :--- | :--- | :--- |

| \*\*Robust statistical anomaly baseline\*\* | Simple reference approach for testing more complex models. | May not capture all context-dependent or multivariate patterns; performance unknown. | Retain as a benchmark, not final choice unless it passes targets. |

| \*\*Unsupervised anomaly detector\*\* | Candidate approach for the required probabilistic anomaly-detection tier. | No project results currently demonstrate sensitivity $\\geq$ 0.90 or false positives $\\leq$ 1. | Prototype candidate; do not select until VV-02 benchmark passes. |

| \*\*Supervised classifier\*\* | Evaluated if representative labelled events/scenarios are available. | Requires representative, correctly labelled data; performance may not generalize. | Benchmark only if data quality/coverage are sufficient; no winner claimed in advance. |

| \*\*Ensemble of probabilistic models\*\* | Could combine complementary signals if testing shows benefit. | Adds compute, tuning, drift-monitoring, and change-control burden without guarantee. | Defer unless comparative tests show material improvement. |



\* \*\*Recommended Decision:\*\* Keep the RFP-mandated hybrid detector. Build a reproducible benchmark. Select the probabilistic model only after the same versioned, held-out VV-02 corpus demonstrates sensitivity $\\geq$ 0.90 and $\\leq$ 1 false positive per 1,000 device-days. Do not report a winner before measurements exist.



\---



\### Trade Study 2 — Evidence Ingestion

\* \*\*Decision Question:\*\* How should six heterogeneous evidence sources be collected, buffered, tracked, and normalized without silent gaps?

\* \*\*Evaluation Matrix:\*\*



| Named Alternative | Fit to Actual Source Mix | Risk / Gap | Disposition |

| :--- | :--- | :--- | :--- |

| \*\*Scheduled polling only\*\* | Retrieves documents and API data for non-publishing sources. | Polling delays conflict with 60s $p\_{95}$ target; needs additional handling for high rates. | Use for document/API sources where appropriate, not system-wide. |

| \*\*Event-streaming-only pipeline\*\* | Suits sources that already publish event streams. | Does not alone cover MDS2 documents, SBOM files, and manufacturer documents. | Not sufficient as the sole ingestion method. |

| \*\*Hybrid source adapters + durable buffer + MDFE normalizer\*\* | Supports event feeds, log/API polling, and document imports while normalizing to MDFE. | Requires per-source ICDs, health monitoring, durable queues, and replay verification. | Recommended pattern; close after all verification targets pass testing. |



\* \*\*Decision:\*\* Use source-specific adapters with an on-premises durable buffer and a version-controlled MDFE normalization service. Support NDR/IoMT telemetry, SIEM logs, firewall logs, MDS2, CycloneDX/SPDX SBOMs, and manufacturer cybersecurity documentation.



\---



\### Trade Study 3 — Forensic Evidence Preservation

\* \*\*Decision Question:\*\* Which storage pattern can satisfy WORM preservation, chain of custody, evidence integrity, and recoverability?

\* \*\*Evaluation Matrix:\*\*



| Named Alternative | RFP Fit | Limitation | Disposition |

| :--- | :--- | :--- | :--- |

| \*\*Relational database only\*\* | Useful for searchable case metadata and queries. | By itself does not establish required WORM preservation and chain of custody. | Not eligible as the only evidence-preservation layer. |

| \*\*On-premises WORM evidence store + cryptographic hashes + custody ledger\*\* | Directly addresses integrity, WORM, trusted time, custody, and verifiable export. | Implementation must prove retention controls, access controls, backup, and recovery work. | Recommended architecture pattern; select specific product only after verification. |

| \*\*Distributed ledger plus evidence storage\*\* | Could distribute audit assertions across participants. | Adds governance and operations complexity without removing need to verify objects. | Not selected for the baseline (not required by RFP). |



\* \*\*Decision:\*\* Preserve original evidence objects in a WORM-capable on-premises store; calculate cryptographic hashes at intake and verify them during analysis/export; record metadata in an append-only custody ledger. Maintain redundant copies and prove RPO 0 / RTO 4 hours in recovery exercises.



\---



\### Trade Study 4 — Deployment and External Connectivity

\* \*\*Decision Question:\*\* Where should the core analysis and evidence live, and how should approved external reference data and manufacturer APIs be accessed?

\* \*\*Evaluation Matrix:\*\*



| Named Alternative | Fit to RFP | Risk / Gap | Disposition |

| :--- | :--- | :--- | :--- |

| \*\*Cloud-first core\*\* | Simplifies centralized hosting and some external API connections. | Outbound dependency for tier-1 analysis, preservation, or reporting conflicts with EN-01. | Not eligible as the baseline due to offline requirement EN-01. |

| \*\*Fully isolated on-premises with signed manual imports\*\* | Meets local/offline operating posture and limits outbound connectivity. | Live manufacturer-cloud API monitoring is unavailable without a separate controlled exchange. | Eligible where live API access is not required; document coverage limitations. |

| On-premises core + segregated controlled relay | Keeps ingestion, preservation, detection, queries, and reports local. Optional relay fetches data. | Relay outage or compromise must not stop tier-1 processing; requires tight separation. | Recommended: core remains on-premises; optional relay handles external updates. |

• Decision: Deploy the core on-premises. Keep any egress relay segregated and allowlisted. Disabling outbound access must not interrupt tier-1 analysis, preservation, or reporting. The relay must never connect to or send traffic toward monitored medical devices.



Trade Study 5 — Reports and Human Oversight



• Decision Question: How should investigations be reported so findings remain traceable, reviewable, and safe?

• Evaluation Matrix:



Named Alternative	Evidence / Safety Fit	Limitation	Disposition

Free-form AI narrative as report authority	Can produce readable narrative text.	Cannot guarantee every conclusion is evidence-supported; vulnerable to adversarial text.	Not eligible as the authoritative report-generation method.

Structured evidence-linked report compiler	Generates reports from typed findings, evidence references, and technique mappings.	Needs a defined finding schema, templates, validation rules, and user testing.	Recommended baseline because it aligns directly with oversight rules.

Structured report + optional constrained narrative summary	May improve readability after findings have been validated and evidence-linked.	Adds prompt-injection, repeatability, and model-change risks; must not alter findings.	Defer until separate adversarial and usability tests pass.



• Decision: Use structured findings and a deterministic report compiler. Each conclusion must have an evidence reference and a type distinguishing observed fact from inference. Never generate clinical diagnosis, treatment, or patient-management recommendations.



6\. Verification Plan \& Decision Log



Decisions are design candidates, not verified compliance claims. Keep each decision open until the listed evidence is produced and reviewed.

• V-01 Model Performance: Pass when sensitivity $\\geq$ 0.90 and $\\leq$ 1 false positive per 1,000 device-days are achieved on declared test data.

• V-02 Latency and Load: Pass when candidate findings $\\leq$ 60s $p\_{95}$, 5,000 events/s/site, 20,000 identities, and 30-day report $\\leq$ 5 min are verified.

• V-03 Outage and Recovery: Pass when fault-injection demonstrates zero accepted-evidence loss during 24-hour source outage, RPO 0, and RTO $\\leq$ 4 hours.

• V-04 Evidence Integrity: Pass when independent verification proves tapering is prevented/detected, and no single point of evidence loss exists.

• V-05 Safety / Privacy: Pass when boundary checks verify zero device write paths, read-only API credentials, and offline continuous execution.

• V-06 Report Integrity / Usability: Pass when adversarial prompt-injection text is resolved and user validation confirms the $\\geq$ 85% observed-vs-inferred distinction agreement.



