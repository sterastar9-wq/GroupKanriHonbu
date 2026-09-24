---
name: rakuco-mf-punch-export
description: "Download Rakuco work-time exports, validate an MF attendance CSV, and import the validated CSV into Money Forward Attendance. Use for Rakuco 書き出し, 作業時間 exports, MF打刻CSV creation, or its MF勤怠 CSV import."
---

この配置版は[共通運用ルール](../../AGENTS.md)を優先する。ブロック・接続障害では自動再試行せず停止報告する。


# Rakuco Mf Punch Export

Create a release-ready MF attendance CSV only when the downloaded XLSX and CSV agree at row level. Treat the XLSX as the source of record. Verify the user-requested target month against the dates in that XLSX; do not use Rakuco's UI as a second source or retain UI-condition evidence. After a CSV passes all release gates, import that exact final CSV into Money Forward Attendance when the user has requested the import.

## Scope and source of truth

- Resolve the requested facility/account and target month before downloading. For a relative request such as `前月`, state the concrete Japan-time month.
- The raw downloaded XLSX is immutable evidence. Do not overwrite it. Record its filename, timestamp, SHA-256, selected account, selected month, and export options in the run manifest.
- Join people by `受給者証番号`, never by name: Rakuco raw column B → master column J. Return master L `MF従業員番号`, E `姓`, and F `名`.
- Treat names only as display evidence. Trimmed/Unicode-normalized name differences are warnings, not join keys. Never silently replace a character variant.
- Snapshot the current master immediately before every transformation; record its spreadsheet revision or export timestamp. Do not reuse an earlier run's master snapshot after the user says IDs or roster data changed.
- Before generating events, create a certificate-level `master_preflight.csv` for every raw person with at least one work-time event. It must show the certificate, raw name, master name, MF number, event count, and join status. A missing, duplicated, or unassigned MF employee number is an ERROR that blocks release by default. Keep the source rows and reason in `excluded_unassigned.csv`; an exclusion may be published only with the user's explicit approval.
- When the user supplies a known-good MF aggregation sheet or CSV, use it as an additional reference comparator. Compare the full seven-column event multiset before release and report reference-only/candidate-only rows. It never replaces the required XLSX↔CSV comparison.

Read [source schema](references/source-schema.md) before parsing a new Rakuco export. Read [reconciliation contract](references/reconciliation-contract.md) before creating a candidate CSV or declaring a run successful.

## Required multi-agent flow

Delegate these independent roles whenever subagents are available. Give each reviewer only the run artifacts it needs; no reviewer may simply accept another reviewer's conclusion.

1. **Exporter**: in the logged-in Rakuco UI, open `実績` → `書き出しをする`; select the requested scope and download once.
2. **Source reviewer**: independently inspect the XLSX. Validate the 15-column table, target-month dates, raw row set, and aggregates. It owns `xlsx_source_validity`.
3. **Transformer**: snapshot the approved master fields, create events from the raw XLSX, and write a UTF-8 BOM *candidate* CSV. It owns the source-to-event lineage.
4. **CSV reviewer**: independently reconstruct events from the raw XLSX and master snapshot, then multiset-compare all seven output fields against the candidate. It owns `xlsx_vs_csv`.
5. **Release gatekeeper**: verify hashes, reviewer artifacts, exception policy, and all zero-difference conditions. Only this role may rename/copy `candidate.csv` to `final.csv`.
6. **MF importer**: only after R1 PASS and user authorization, open `https://attendance.moneyforward.com/admin/settings/importers/attendances_csv_importers/new`, choose the exact `final.csv`, verify the selected filename, then press `インポート`. Record the import result or error; never select a candidate, excluded, or rejected CSV.

Keep the user informed when a skill-driven action starts or pauses. Ask for a human decision only to resolve an exception, select a target month/account not implied by the request, or authorize a downstream import.

## Hard gates

Before selecting or importing an MF file, read [MF import rules and dialog recovery flow](references/mf-import-dialog-flow.md). A click timeout or `getJsDialog()` returning none does not prove that no confirmation dialog exists. Use only currently permitted UI methods; verify import success and actual punches before reporting completion. Do not replay an import just to test this procedure.

- **E1 — target month**: every included XLSX date matches the user-requested month.
- **S1 — downloaded source validity**: the XLSX has the expected schema and its raw row-set and aggregate checks pass.
- **T1 — transform**: each candidate event has exactly one source-row reference; all joins are unique; dates/times are valid; output duplicates are explained or rejected.
- **C1 — CSV fidelity**: exact seven headers and order; bidirectional event multiset comparisons show `expected-only=0`, `csv-only=0`, and `field-mismatches=0`.
- **B1 — master preflight**: all raw people with work-time events have exactly one current master match and a nonblank MF employee number, unless the user has explicitly authorized the named exclusions.
- **Q1 — reference comparator**: when a known-good aggregate is supplied, its seven-column multiset equals the candidate (`reference-only=0`, `candidate-only=0`).
- **R1 — release**: E1, S1, T1, and C1 are PASS. Any ERROR produces a rejected report and no release CSV.
- **I1 — MF import**: only the R1-PASS `final.csv` is selected; record the displayed import result. A failed import is reported and must not be described as imported.

Use `scripts/build_candidate.py` for deterministic XLSX parsing and event construction. Store every run under a new, date-stamped directory and retain at minimum the raw XLSX, current master snapshot, master preflight, candidate CSV, exception CSV, reconciliation JSON, and manifest.

## Proven output contract

The output uses this exact header order:

```text
従業員番号,苗字,名前,打刻所属日,打刻日,打刻時間,打刻種別
```

For every valid raw `作業開始時間` (I), create one `出勤` event. For every valid raw `作業終了時間` (J), create one `退勤` event. Raw date C populates both date columns. Blank and `-` time values create no event. Use UTF-8 BOM encoding for the candidate and final CSVs.

## Completion report

Report the concrete scope, source and event counts, exclusion count and reason, each gate's PASS/FAIL status, final artifact paths, and—when requested—the MF import result. Do not claim success from matching totals alone; the bidirectional XLSX↔CSV row-level comparison must pass.
