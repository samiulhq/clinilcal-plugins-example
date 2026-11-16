---
name: clinical-qc
description: Independent QC / double programming of a clinical derivation. Use when the user asks to QC, double program, independently reproduce, or verify a production ADaM derivation (e.g. "QC the PFS derivation independently in R"). Fetches the validated counterpart in the QC language from clinical-kb (never translates), runs it, compares with diffdf, and renders a Quarto QC report. Builds on clinical-programming-sop.
---

# Independent QC (double programming)

Follow `clinical-programming-sop` for headers, execution, verification, and reporting. This skill changes only the retrieval and adds the comparison step.

## Procedure

1. **Production asset.** Use the asset the user names. Otherwise, run `search_assets` in the study's `production_language` (for example, SAS for ONC-101) and take the validated top hit, such as `sas-derive-pfs-v1.2`.
2. **Counterpart.** Call `get_counterpart(asset_id, language=<qc_language>, study_id)`. Fetch the counterpart of every companion the production program included, too (for example, `sas-censor-onc101-v1.0` maps to `r-censor-rules-onc101-v1.0`).
   - If there is no counterpart: **stop.** Report the `no_counterpart` error and offer to `propose_asset` a counterpart draft. **Never translate the production code.**
3. **QC program.** Generate it in the QC language with the same header convention, at `<program_root>/qc/qc_<dataset>.R`. Write every include exactly as given in `include_stmt`. Read the inputs with `haven::read_xpt()` and save the output with `haven::write_xpt()` to `<program_root>/output/qc_<dataset>.xpt`. Run it and verify (no errors or warnings, report `nrow()`), then run the counterpart's `test_path` with `testthat::test_file()`.
4. **Compare.** Read the production output with `haven::read_sas()` (`<program_root>/output/<dataset>.sas7bdat`). Strip labels and formats (`haven::zap_label`, `haven::zap_formats`). Run `diffdf::diffdf(base = production, compare = qc, keys = c("USUBJID", "PARAMCD"))` on the SAP key variables `USUBJID, PARAMCD, AVAL, CNSR, ADT`. Zero issues is PASS. If there are issues, show them; do not "fix" either side.
5. **QC report.** Render `<program_root>/qc/qc_report_<dataset>.qmd` with Quarto. Include the assets used on both sides (id, version, status), the row counts, the diffdf summary, and PASS or FAIL.

## Real counterpart payloads (ONC-101)

`get_counterpart("sas-derive-pfs-v1.2", "R", "ONC-101")`:

```json
{"asset_id": "r-derive-pfs-v1.0", "language": "R", "validation_status": "validated",
 "version": "1.0", "include_kind": "r_library", "include_stmt": "library(onc101derive)",
 "entrypoint": "derive_pfs",
 "call_template": "adtte_pfs <- derive_pfs(adsl, adrs, cutoff = dcut, censor_fun = censor_rules_onc101)",
 "test_path": "/home/samiul/Governed-Agentic-Programming/assets/r/onc101derive/tests/testthat/test-derive_pfs.R"}
```

`get_counterpart("sas-censor-onc101-v1.0", "R", "ONC-101")`:

```json
{"asset_id": "r-censor-rules-onc101-v1.0", "include_kind": "r_source",
 "include_stmt": "source(\"/home/samiul/Governed-Agentic-Programming/assets/r/helpers/censor_rules_onc101.R\")",
 "entrypoint": "censor_rules_onc101",
 "call_template": "cens <- censor_rules_onc101(adsl, adrs, cutoff = dcut)"}
```

No counterpart (real): `{"error": "no_counterpart", "concept_id": "recist11_response", "available_languages": [], "hint": "No validated sibling in that language. Do not translate; offer propose_asset for a counterpart draft."}`

## Talking point

One knowledge base, two languages, true double programming, and no translation. Both sides are independently validated assets linked by `concept_id`.
