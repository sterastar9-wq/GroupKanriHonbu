# 国保連請求

ラクコからの請求CSV取得、営業所別の重複検査・整理、取込送信V2への取込、電子請求認証、本番送信までのスキル・運用ルール・フローです。

## スキル

- [国保連請求スキル](skills/kokuho-claim-flow/SKILL.md)
- [「スキル化して」の標準処理](skills/rules-skill-flow-capture/SKILL.md)

## ルール・フロー

| 工程 | ルール | フロー |
|---|---|---|
| ラクコからダウンロード | [ルール](skills/kokuho-claim-flow/references/download-rules.md) | [フロー](skills/kokuho-claim-flow/references/download-flow.md) |
| 取込送信V2ログイン・営業所切替 | [ルール](skills/kokuho-claim-flow/references/login-rules.md) | [フロー](skills/kokuho-claim-flow/references/login-flow.md) |
| 取込・電子請求認証・本番送信 | [ルール](skills/kokuho-claim-flow/references/import-send-rules.md) | [フロー](skills/kokuho-claim-flow/references/import-send-flow.md) |

[元マニュアルに基づく手順](skills/kokuho-claim-flow/references/procedure.md)

## 呼び出し例

- 「国保連請求データダウンロード、2026年○月分」
- 「国保連請求、2026年○月分を送信前まで進めて」
- 「確認済みの○年○月分、沖洲の国保連請求を本番送信して」

本番は送信後の認証情報を営業所に対応させて選択し、直後の結果を確認します。エラー・結果不明の場合は報告して停止し、無条件に再送しません。

## 利用環境と検証状況

スキル内のWindowsパスは実証環境の設定です。他の端末では許可済みの実在パスへ対応付けてください。認証情報の値は含みません。必要な外部スキル（computer-use、rakuco-login-session、skill-creator等）は各環境で用意してください。

2026-09-25時点で、両営業所のCSV取得・検証・整理と、沖洲の取込・電子請求認証・送信確認画面まで実証済みです。送信後の認証情報選択はユーザー指定の手順で、実機検証は未実施です。川島のローカルログイン候補欠落は未解決で、実行時に再確認します。

このリポジトリへの保存は請求の送信許可を意味しません。CSV・PDF・実行ログ・.env・認証秘密値は収録しません。
