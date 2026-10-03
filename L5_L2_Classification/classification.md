# L5 Narrow / L2 General Classification — KANTOR_K5
**Platform:** Anticloud | **PAX:** 27B | **IP:** USPTO pending 2026, Anticloud FZ LLE

## L5 Narrow
KANTOR_K5 specializes in structured fact storage and retrieval for the Anticloud domain specifically:
project metadata, TRL levels, benchmark results, regulatory status, API signatures. No general
encyclopedic knowledge. The authoritative Anticloud-specific fact store.

## L2 General
Every tier consumes KANTOR_K5. PAX 27B queries it for grounded factual responses. ANTICODE_AGENT
queries it for API signatures. INTE11ECT_APP queries it for domain definitions.

## PAX Integration
PAX 27B queries KANTOR_K5 via JSON-LD structured lookups before generating factual responses,
grounding outputs in verified Anticloud facts and reducing hallucination for deployment-specific questions.

## AIOSS Audit Relevance
Every fact assertion is versioned and hash-chained. Auditors can verify which version of a fact
was active at the time of any PAX inference by cross-referencing the AIOSS chain timestamps.

## Regulatory
ISO 27001 A.8.3, NIST SP 800-188 (de-identification for any PII in knowledge records)
