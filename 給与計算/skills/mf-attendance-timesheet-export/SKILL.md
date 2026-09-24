---
name: mf-attendance-timesheet-export
description: Export Money Forward Cloud Attendance ledgers and audit a completed day for missing clock records or unrequested overtime. Use for 出勤簿データ downloads and daily attendance-notice lists; do not modify attendance records.
---

この配置版は[共通運用ルール](../../AGENTS.md)を優先する。ブロック・接続障害では自動再試行せず停止報告する。


# Money Forward Cloud Attendance: 出勤簿データエクスポート

Use the logged-in normal **マネーフォワード クラウド勤怠** administration page. This is not the separate 勤怠Plus product and does not use a public API.

## Scope and authorization

- This skill creates an export job and downloads its output. Do not edit attendance records, alerts, employee settings, or approval status.
- Follow the user’s requested export conditions exactly. If the month is stated relatively, resolve it using the current date in Japan Standard Time.
- Do not use undocumented HTTP endpoints or internal requests. Operate the visible browser form and verify its current DOM state before each state-changing action.

## Page and form

Open or use the logged-in page:

`https://attendance.moneyforward.com/admin/settings/exporters/daily_attendance_item_csv_exporters/new`

The export submission is asynchronous. After starting it, a new row appears at the top of **エクスポート履歴** with a processing status. Wait until that exact row reads `ダウンロード可能` before downloading it.

Use durable labels and DOM selectors rather than export-history record IDs, which vary each time.

| Setting | Form control |
| --- | --- |
| Single/multiple month | `input[name="admin_settings_exporters_daily_attendance_item_csv_exporter_form[period_type]"]` with `single_month` or `multiple_month` |
| Month picker | `input[placeholder="YYYY/MM"]`; the visible picker synchronizes hidden year/month inputs |
| Output unit | `input[name="admin_settings_exporters_daily_attendance_item_csv_exporter_form[filter_by]"]` with `employee_ids` or `cutoff_day_value` |
| Employee selection | `input.multiselect__input[placeholder="従業員を選択してください"]`; the `全て` tag means all employees |
| Sort | `input[name="admin_settings_exporters_daily_attendance_item_csv_exporter_form[sort_order]"]` with `employee_code` or `month` |
| Time format | `#admin_settings_exporters_daily_attendance_item_csv_exporter_form_attendance_time_format` |
| Include actual stamps | `input[name="admin_settings_exporters_daily_attendance_item_csv_exporter_form[with_original_record_model_times]"]` with `true` or `false` |
| Include actual working time | `input[name="admin_settings_exporters_daily_attendance_item_csv_exporter_form[with_actual_working_time]"]` with `true` or `false` |
| Start export | `input[name="commit"][value="エクスポート"]` |

For the normal daily-check report, use the user’s specified settings. A common requested configuration is: one month, all employees, employee-code ordering, `時刻（1時間30分を01:30と表示）`, and both inclusion flags set to `false`.

## Download workflow

1. Confirm every requested setting from the rendered DOM immediately before submission.
2. Click `エクスポート` once. Capture the new top history row’s generated filename and timestamp so retries cannot download an older file.
3. Poll that exact row every 30 seconds for up to 10 minutes. A `処理中` status is not an error and must not create another export for the same conditions.
4. If the row explicitly reports failure or the browser is blocked, stop and report; do not automatically retry.
5. Once the row is downloadable, select `xlsx形式(Excelブックファイル)` in that row’s format dropdown. The value is `excel`.
6. Click the `ダウンロード` button within the same history row once, then verify the browser download event or other available download completion signal.
7. Report the generated filename and the requested period. If browser tooling cannot expose the download location, say so plainly rather than inventing a path.

The generated name normally resembles `daily_attendance_report_YYYY_MM_YYYYMMDD_HHMMSS`; the browser typically adds `.xlsx` for the Excel selection.

## Failure handling

- If authentication is required, follow the sibling rakuco-login-session skill with the approved .env credentials. Hand off OTP when necessary.
- If generation remains `処理中`, keep polling the same row for the full 10 minutes. Do not start another export for the same conditions.
- If the same row has not become `ダウンロード可能` after 10 minutes, report it as timed out and stop. Do not treat a shorter wait as failure.
- If a download event is unavailable after one confirmed click, inspect the on-page row and report that the download request was issued but that its local save could not be verified. Do not repeatedly click without evidence.
