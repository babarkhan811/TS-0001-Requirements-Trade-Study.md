\# ADR-0001: Requirements Trade Study Baseline Selection



\## Status

Proposed



\## Context

During Sprint 1, multiple requirement conflicts were identified regarding the AI-Powered Digital Forensics Assistant for Medical Devices. Specifically, balancing unverified probabilistic AI models against strict forensic tracing rules, handling extreme data loads (5,000 events/s/site), and ensuring strict offline device boundaries (EN-01) while needing access to live manufacturer cloud data.



\## Decision

We have decided to baseline an architecture that prioritizes safety-critical parameters while deferring unverified selections until physical validation. 

1\. \*\*Detection:\*\* Hold the hybrid deterministic/probabilistic requirement tier but defer final model selection until benchmarked against the `VV-02` corpus.

2\. \*\*Ingestion:\*\* Deploy hybrid source adapters with a durable on-premises buffer to handle heavy load variations.

3\. \*\*Connectivity:\*\* Hold fully on-premises core tracking (EN-01) and relegate external cloud polling to a completely optional, segregated egress relay.



\## Consequences

\* \*\*Positive:\*\* Ensures strict compliance with safety-critical boundaries and protects forensic integrity from prompt injection or data drops.

\* \*\*Negative:\*\* System components cannot be conclusively selected or declared compliant until physical verification reports are signed off in Sprint 2.



\## References

Linked Trade Study: \[TS-0001-Requirements-Trade-Study.md](TS-0001-Requirements-Trade-Study.md)



