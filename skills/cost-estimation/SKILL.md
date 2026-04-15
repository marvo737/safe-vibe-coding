---
name: cost-estimation
description: Use when integrating pay-per-use services (LLM APIs, cloud functions, managed DBs, egress-heavy features) or before deploying features that scale with user count - prevents runaway bills and bankruptcy-by-bug
---

# Cost Estimation

## Overview

「バズった瞬間に破産」を防ぐ。実装着手前の **手計算試算** と、デプロイ時の **予算アラート先行設定** で、青天井請求を機械的に止める。

## When to Use

- LLM API (OpenAI / Anthropic / Gemini) を呼ぶ機能
- AWS Lambda / Cloud Functions / Cloud Run などの従量課金
- マネージドDB(読み取り/書き込み単位課金: Firestore, DynamoDB)
- 画像/動画配信(egress課金)
- 外部有料API(SendGrid, Twilio, Stripe手数料 等)

## Pre-Implementation Checklist

**1. 単価を確認**
- 課金単位: request / token (input/output別) / read / write / GB-second / egress GB
- 無料枠の範囲と上限超過後の単価

**2. 負荷試算(最低3シナリオ)**

| シナリオ | DAU | 1人あたり呼び出し | 1回あたり単位消費 | 月額想定 |
|---|---|---|---|---|
| 現状 | 10 | 5 | - | $ |
| 想定 | 100 | 10 | - | $ |
| バズ | 10,000 | 20 | - | $ |

バズ試算で3桁以上の金額が出たら **設計を見直す**(キャッシュ / レート制限 / 非リアルタイム化)。

**3. ハードリミット機能の確認**
- AWS Budgets Actions: IAMポリシー適用(権限制限) / SCP適用 / EC2・RDS停止などをトリガ可能。ただし Lambda / Bedrock / API Gateway など **サーバーレス系の課金主因は直接停止できない** ものが多く、Lambda経由での停止実装やサービス削除スクリプトを別途用意する必要がある
- GCP Budget alerts はアラートのみ、自動停止は Pub/Sub + Cloud Functions で自実装が必要
- OpenAI / Anthropic の usage limit / spend limit 設定
- 各LLMクライアントで `max_tokens` / timeout を必ず指定

## Deployment Checklist

デプロイと **同時に** 実施(後回し禁止):

- [ ] 予算アラート設定(月額想定の50%/80%/100%)
- [ ] サービス単位の上限(可能なら自動停止)
- [ ] レート制限(per-user / per-IP)
- [ ] 認証必須化(公開エンドポイントは攻撃で即枯渇)

## Post-Deployment

- リリース直後1週間は **毎日** ダッシュボード確認
- 週次トレンドを記録
- 急増時の原因特定手順を事前に用意(どのエンドポイント/誰)

## Red Flags

| 状況 | リスク |
|---|---|
| 「無料枠に収まるはず」 | 試算なしの楽観、設定ミスで即超過 |
| 「認証なしで公開」 | ボットに叩かれ数時間で数百万円の事例多数 |
| 「ループ内でLLM呼ぶ」 | 1リクエストで数百回呼ばれる罠 |
| 予算アラート未設定のままデプロイ | 検知が請求書到着時になる |

## Common Mistakes

- **output tokenの見落とし:** LLMはinputよりoutputが高額。max_tokensで上限を切る。
- **再帰的呼び出し:** Lambda → SQS → Lambda の循環で無限課金。トリガー設計を図示して確認。
- **CloudFront/egressの想定漏れ:** 動画/画像は配信コストが本体より大きくなる。
