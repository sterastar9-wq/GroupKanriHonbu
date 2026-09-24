# Rakuco export source schema

## Expected workbook section

The Rakuco work-time export is expected to contain the section `<利用者毎の詳細情報>`. The row immediately below may contain a note, followed by a header row and detailed records. The parser locates the section title rather than assuming a fixed row number.

The current expected detail schema is columns A:O:

| Column | Header | Use |
|---|---|---|
| A | 利用者名 | Display evidence only |
| B | 受給者証番号 | Primary join key to the master |
| C | 日付 | Source for both CSV date columns |
| D | サービス提供状況 | Live/source reconciliation |
| E/F | 通所開始時間 / 通所終了時間 | Live/source reconciliation |
| G/H | 食事提供 / 送迎 | Live/source reconciliation |
| I/J | 作業開始時間 / 作業終了時間 | CSV 出勤 / 退勤 events |
| K/L | 作業合計時間 / 休憩合計時間 | Aggregate reconciliation |
| M/N/O | 在宅ワーク / メモ / 作業内容 | Live/source reconciliation |

If the header order, the section title, or an essential header is absent, stop the run as `SOURCE_SCHEMA_MISMATCH`. Do not guess a replacement mapping.

## Master snapshot schema

Create a read-only CSV snapshot from the user-approved master containing exactly:

```text
受給者証番号,MF従業員番号,姓,名
```

The standard current mapping is `01_利用者マスタ`: J → 受給者証番号, L → MF従業員番号, E → 姓, F → 名. Verify the master tab and headers during each run; never reuse an old snapshot as current master evidence.
