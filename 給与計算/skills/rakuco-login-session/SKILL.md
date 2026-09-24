---
name: rakuco-login-session
description: "通常Chromeで承認済み.envを使いラクコ・MF勤怠・MF給与へログインする。"
---

# ログイン

共通条件は[運用ルール](../../AGENTS.md)に従う。既存認証済み画面があれば再利用し、認証時だけ実行環境に設定した.envを読む。秘密値を表示せず、正規HTTPSホストを確認してフォームへ入力する。

- ラクコ：coala-prod-member.web.app、RAKUCO_ID / RAKUCO_PASS。
- MF：id.moneyforward.comからattendance.moneyforward.comまたはpayroll.moneyforward.comへ、MF_ID / MF_PASS。
- 認証後の会社名と目的画面を確認して成功とする。OTP・CAPTCHA等は依頼者へ引き継ぐ。
- ブロック・タイムアウトは停止報告し、別経路でのログイン連打をしない。
