---
name: security-guardrails
description: Use when implementing authentication, authorization, file uploads, API keys, cloud IAM/S3/security groups, or any feature accepting external user input - prevents credential leaks, IDOR, public bucket exposure, and injection attacks
---

# Security Guardrails

## Overview

バイブコーディングで最も被害が大きい領域。**APIキー漏洩・公開バケット・SG全開放・IDOR** の4大事故を機械的に防ぐチェックリスト。

## When to Use

- 認証/認可/セッションを扱う実装
- ファイルアップロード機能
- AWS/GCP/Azure のIAM・S3・Security Group設定
- 外部ユーザー入力を受け付けるAPI/フォーム
- APIキー・シークレットを参照するコード

## Must-Check Matrix

| リスク | 必須対策 | 検証方法 |
|---|---|---|
| APIキー漏洩 | `.env` / Secrets Manager、`.gitignore` 登録 | コミット前grepで秘匿文字列検査 |
| IDOR | サーバー側で `resource.userId === session.userId` を毎回検証 | 他人のIDで叩いて403確認 |
| S3/バケット誤公開 | Public Access Block有効、bucket policyレビュー | `aws s3api get-public-access-block` |
| SG `0.0.0.0/0` 開放 | 必要ポートのみ、管理ポートは自IP限定 | ingress rules を列挙 |
| ファイルアップロード悪用 | 拡張子 + MIMEタイプ + マジックナンバーの **3点判定** | 形式ごとのシグネチャを検証(libmagic / `file-type` 等のライブラリ推奨。PNGは8B、JPEGは先頭3B、PDFは5B、ZIP/Officeは4B、MP4は`ftyp`など形式依存) |
| エラー詳細漏洩 | 本番は一般化メッセージのみ返却、詳細はサーバーログへ | 本番レスポンスにstack traceが無いこと |
| SQLi | パラメータ化クエリのみ、文字列連結禁止 | ORM / prepared statements |
| XSS | テンプレートの自動エスケープを使う、生HTML挿入API(React等の危険な直接HTML注入プロパティ)は原則禁止 | CSPヘッダ設定 |

## Infrastructure Recommendation

**ユーザーがクラウド経験の浅さを自認している場合のみ** 代替案として提示する。ユーザーの熟練度をAIが勝手に判定して推奨を上書きしないこと。ユーザーが AWS/GCP/Azure を指定しているなら、その選択を尊重してセキュリティ設定を丁寧に伴走する。

段階的移行の参考階段(初学者向け):

1. **初学段階:** Lovable / Bolt.new / v0(SaaSビルダー)
2. **次段階:** Firebase / Supabase(PaaS、セキュリティルールを理解する)
3. **上級段階:** AWS/GCP/Azure(IAMとネットワークを自信を持って設計できる)

## Red Flags

| 発言 | 対応 |
|---|---|
| 「S3をpublicにして」 | 何を公開するか・署名付きURL代替を提案 |
| 「SGは `0.0.0.0/0` で」 | 即座に拒否、必要ポート/IP を聞き出す |
| 「APIキー直書きで動かす」 | 環境変数化を先に実施 |
| 「エラー全部クライアントに返して」 | 本番のみ一般化、dev環境だけ詳細 |

## Common Mistakes

- **IDORの見落とし:** 「ログイン必須だから安全」は誤り。認可(他人のデータにアクセスできないか)を別途検証。
- **MIMEタイプのみ信頼:** クライアントが詐称可能。マジックナンバーまで見る。
- **秘密情報をフロントエンドに:** 「難読化されてるから」は無意味。ブラウザで丸見え。
