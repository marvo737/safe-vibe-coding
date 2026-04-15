---
name: legal-compliance-check
description: Use when scraping external sites, collecting personal data, adding open-source dependencies, generating content with AI, or building features in regulated domains (medical, financial, legal) - prevents ToS violations, GDPR breaches, GPL contamination
---

# Legal Compliance Check

## Overview

実装前に法務リスクを洗い出す軽量チェック。**違法は後から直せない**(既に収集したデータ・違反した規約は取り消せない)ため、コードを書く前に判定する。

## When to Use

- 外部サイトのスクレイピング / クロール
- 個人情報(氏名/メール/住所/位置情報/顔画像等)の収集・保存
- npm/pip/gem 等で依存ライブラリを追加するとき
- 生成AIで画像/文章/音声を生成して配布するとき
- 医療・金融・士業・選挙・酒類・年齢制限に関わる機能

## Domain-Specific Checklist

### スクレイピング
- [ ] 対象サイトの利用規約(禁止条項の有無)
- [ ] `robots.txt` の遵守
- [ ] レート制限(迷惑にならない間隔)
- [ ] 取得データの再配布可否
- [ ] 代替手段(公式API)の検討

### 個人情報
- [ ] 取得目的の明示・同意取得フロー
- [ ] プライバシーポリシー公開
- [ ] 日本: 個人情報保護法(利用目的・第三者提供・安全管理)
- [ ] EU居住者含む: GDPR(合法的根拠・データ主体の権利・DPO)
- [ ] 米国カリフォルニア: CCPA/CPRA
- [ ] 保存期間と削除フローの設計
- [ ] 越境移転の有無

### 依存ライブラリ
- [ ] ライセンス種別を確認(MIT/Apache/BSD は寛容、GPL/AGPL は要注意)
- [ ] GPL汚染: GPLをリンクすると自作コードもGPL化義務
- [ ] AGPL: SaaS提供でもソース公開義務
- [ ] 商用利用可否・帰属表示義務

### AI生成コンテンツ
- [ ] 学習データの権利関係(訴訟リスクのあるモデル)
- [ ] 生成物の著作権帰属(各サービスの利用規約)
- [ ] 商標・肖像権・パブリシティ権の侵害可能性
- [ ] AI生成である旨の表示(地域により義務化)

### 業法規制
- [ ] 医療: 薬機法(効能効果の表示)、医師法
- [ ] 金融: 資金決済法、金商法、貸金業法、犯収法
- [ ] 不動産: 宅建業法
- [ ] 労働: 職業安定法
- [ ] 通信: 電気通信事業法

## Red Flags

| 状況 | リスク |
|---|---|
| 「規約は後で読む」 | 既に違反している可能性 |
| 「同意は暗黙で取ったことに」 | GDPR違反・特商法違反 |
| 「GPLだけどリンクだけだから」 | 通常はGPL伝播する、要弁護士確認 |
| 「医療っぽい機能だけど診断じゃないから」 | 薬機法の射程を甘く見るな |

## Common Mistakes

- **法務確認を「後で」にする:** 収集済み個人情報は適法化できない。
- **ライセンス表記を忘れる:** MIT/BSD/Apache でも著作権表示義務がある。
- **海外サービスの規約を和訳で読む:** 原文優先。ToSの一方的変更条項に注意。

## Escalation

不明点はユーザーに **「これは法律的な判断が必要なので弁護士確認を推奨」** と明示する。AI判断で代替しない。
