# Reconciliation contract

## Canonicalization

Preserve raw values and normalize only for comparisons:

- certificate number: text, preserving leading zeroes
- dates: `YYYY-MM-DD`
- times: `HH:MM`
- blank and `-`: null for event generation
- names: Unicode NFKC plus trimmed/collapsed whitespace for warnings only

Do not normalize character variants such as `崎` and `﨑` unless an approved alias record explicitly documents the conversion.

## Required validation and comparison

### 1. Requested month and XLSX validity

Treat the downloaded XLSX as the source of record for row-level processing. Confirm that every included raw date falls in the month requested by the user. Validate its 15-column schema and reconcile raw row count, distinct people, service-status count, work/break durations, meal count, and transport count. A wrong-month date or invalid source schema is an error. Rakuco UI conditions are not retained or used for reconciliation.

### 2. XLSX ↔ CSV

Derive an expected event multiset from the XLSX and master snapshot. Keep the certificate number and raw row reference in review-only data. Compare it against the seven CSV fields as a multiset, not a set. Require counts for 出勤 and 退勤, per person/day, and the whole run to agree.

### 3. Optional approved reference aggregate ↔ CSV

When the user provides a historical or approved MF aggregation sheet/CSV for the same account and month, compare its seven-column event multiset with the candidate. Report both directional deltas. A mismatch blocks release until it is explained; this comparison is supplementary and cannot substitute for XLSX ↔ CSV.

## Exception policy

| Condition | Outcome |
|---|---|
| Missing or ambiguous certificate match | ERROR, block release |
| Missing MF employee number | ERROR and block release by default; write `excluded_unassigned.csv` and `master_preflight.csv`. Release an excluded subset only with explicit user approval. |
| Invalid/out-of-scope date | ERROR, block release |
| Blank or `-` start/end time | No event; count in report |
| Name difference with same certificate | WARN; retain raw and master values |
| Any comparison mismatch | ERROR, block release |

`PASS` requires an XLSX whose dates all match the requested month, a valid XLSX, and zero errors in the XLSX ↔ CSV comparison. A warning may remain only when it has no event-level or identity mismatch.

## Run artifacts

```text
runs/<run-id>/
  raw_export.xlsx
  master_snapshot.csv
  master_preflight.csv
  candidate.csv
  final.csv                 # only after R1
  excluded_unassigned.csv
  expected_events.csv
  reconciliation.json
  manifest.json
```

The manifest records selected account/month/options, timestamps, file hashes, reviewer results, counts, and release decision.
