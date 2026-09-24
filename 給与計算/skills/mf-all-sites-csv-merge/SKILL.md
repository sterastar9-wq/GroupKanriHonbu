---
name: mf-all-sites-csv-merge
description: "Merge a specified month’s validated, site-by-site Rakuco MF attendance CSVs into one MF import CSV. Use when the user says 「全拠点まとめて」 or asks to consolidate all site CSVs; do not use to create individual site exports."
---

この配置版は[共通運用ルール](../../AGENTS.md)を優先する。ブロック・接続障害では自動再試行せず停止報告する。


# 全拠点MF取込CSVの統合

指定月の拠点別MF取込CSVを、内容を落とさずに一つのMF取込CSVへ統合する。

## 入力の確定

- ユーザーが対象月と拠点を指定していればそれを採用する。「全拠点」はその依頼で作成済み・明示された拠点別CSVを意味し、現在の利用者マスターの所属で再判定しない。
- 同月・同拠点に複数版のCSVがある場合は、件数、作成時刻、照合記録を一覧にして、どの版を統合するかユーザーに確認する。旧版・再出力版を推測で混在させない。
- 個人除外は、拠点別CSVを作る段階で明示承認されたものだけを引き継ぐ。統合時に現在の所属、在籍状態、氏名表記などを理由に行を除外・追加しない。

## 統合と検証

1. 各入力CSVが同じ7列ヘッダー、UTF-8 BOM、対象月の行だけで構成されることを確認する。詳しい形式は [references/validation.md](references/validation.md) を読む。
2. 可能なら各拠点のラクコ原本との照合記録を確認する。原本件数とCSV件数が一致しない拠点は停止して差分を提示する。
3. 入力CSVの行をそのまま多重集合として連結し、日付・従業員番号・時刻・種別で並べ替える。重複らしく見える行も、入力に存在する限り勝手に削除しない。
4. 出力の行多重集合が入力群の合計と完全一致すること、拠点別件数・全体件数・出勤/退勤件数が一致することを確認する。
5. `MF取込_YYYY年M月_全拠点.csv` をUTF-8 BOMで出力し、対象拠点、各件数、合計、除外済みの人、照合結果を報告する。

MFへのアップロード・インポートは、このスキルの範囲外。ユーザーが明示的に依頼したときだけ実行する。
