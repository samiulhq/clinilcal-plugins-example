---
name: ta-oncology
description: Oncology therapeutic-area conventions for clinical programming. Use with clinical-programming-sop whenever the study or request is oncology (RECIST 1.1 tumour response, PFS, OS, BOR, ORR, DCR, TTR, duration of response, ADRS/ADTTE). Supplies the ta='oncology' search parameter and oncology vocabulary; contains no procedure.
---

# Oncology TA conventions (parameters only)

- Search parameter: `ta="oncology"` for every `search_assets` call.
- The full TA convention is the validated spec asset `spec-recist11-response-v1.0`. Fetch it with `get_asset` when you need the details.

## RECIST 1.1 vocabulary (ADRS `PARAMCD = "OVR"`, `AVALC`)

CR (complete response), PR (partial response), SD (stable disease), PD (progressive disease), and NE (not evaluable). NON-CR/NON-PD counts as SD for best response. TA numeric coding: CR=1, PR=2, SD=3, PD=4, NE=5.

## Oncology concepts in the knowledge base

| Endpoint | concept_id | Dataset |
|---|---|---|
| PFS | `derive_pfs` | ADTTE |
| OS | `derive_os` | ADTTE |
| BOR | `derive_bor` | ADRS |
| ORR | `derive_orr` | ADRS |
| DCR | `derive_dcr` | ADRS |
| TTR | `derive_ttr` | ADTTE |
| Study censoring rules | `censor_rules_onc101` (study-specific) | subject-level |

Concepts not listed here may have no validated asset. Duration of response (DoR) is one example. Search anyway; if nothing reaches the threshold, the SOP gap procedure applies.

## Philosophy

- **Events:** the first qualifying occurrence. PFS counts PD or death; OS counts death from any cause.
- **Censoring:** at the last date the subject was known to be event-free with an **adequate** assessment (CR/PR/SD; NE is not adequate).
- **New anti-cancer therapy:** tumour assessments after it are not used for tumour-based endpoints. OS is not censored by it.
- **BOR requires confirmation per the SAP:** CR/PR need a repeat assessment at least 28 days later, and SD needs a minimum duration from the time origin.
- **Precedence:** the study SAP overrides TA defaults. Study censoring rules live in one validated study asset that every derivation reuses.
