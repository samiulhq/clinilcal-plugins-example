---
name: study-onc101
description: Study ONC-101 parameters for clinical programming. Use with clinical-programming-sop and ta-oncology for any ONC-101 request (deriving or QC-ing ADTTE/ADRS endpoints such as PFS, OS, BOR, ORR, DCR, TTR, DoR). Supplies study_id, languages, SAP version, data cutoff, naming convention, mandatory censoring asset and the draft reviewer; contains no procedure.
---

# Study ONC-101 (parameters only)

| Parameter | Value |
|---|---|
| `study_id` | `ONC-101` |
| TA | oncology (see `ta-oncology`) |
| Production language | SAS (SAS Viya compute) |
| QC language | R |
| SAP | SAP v2.0 2026-03-01 (`spec-sap-onc101-tte-v2.0`) |
| ADaM | ADaM IG 1.3 |
| Data cutoff | SAS: `%let dcut = '31MAR2026'd;` and use `&dcut`. R: `dcut <- as.Date("2026-03-31")` |
| Input data | `/home/samiul/Governed-Agentic-Programming/data/adsl.xpt`, `adrs.xpt`. SAS: `libname ... xport`. R: `haven::read_xpt()`. No `.rds`. |
| Program root | `program_root` from `get_study_context` (never hard-code it) |
| Naming | `adam/<dataset>.sas` writes `output/<dataset>.sas7bdat`; `qc/qc_<dataset>.R` writes `output/qc_<dataset>.xpt`; `qc/qc_report_<dataset>.qmd` |
| Draft reviewer | M. Lindqvist (Lead QC Programmer). Name her in every `propose_asset` rationale. |

## Study rules

- **`censor_rules_onc101` must be applied for PFS and OS** (and TTR). In SAS, that means `%include` `sas-censor-onc101-v1.0` **before** `%derive_pfs`, `%derive_os`, or `%derive_ttr`. In R, `source()` `r-censor-rules-onc101-v1.0` and pass `censor_fun = censor_rules_onc101`.
- The ADTTE keys for QC comparison are `USUBJID, PARAMCD`, and the compared variables are `AVAL, CNSR, ADT`.
- Output dataset names: production ADTTE (PFS) is `out.adtte`, saved as `output/adtte.sas7bdat`. The QC output is `output/qc_adtte.xpt`.
