# safe-vibe-coding

AI支援による高速開発(バイブコーディング)で **致命的な失敗だけは避ける** ためのガードレールスキル集。

元記事: [エンジニア歴20年の視点から見たAIコーディングの現実 (Akira-Isegawa, Qiita)](https://qiita.com/Akira-Isegawa/items/00f23d206c504db2ac3b)

## 収録スキル

| スキル | 発動場面 |
|---|---|
| `vibe-coding-overview` | 新規機能実装の着手前に6原則を横断チェックしたいとき |
| `security-guardrails` | 外部公開・認証・ファイルアップロード・クラウドIAMを扱うとき |
| `cost-estimation` | 従量課金のクラウド/LLM/外部APIを組み込むとき |
| `legal-compliance-check` | スクレイピング・個人情報・依存ライブラリライセンスを扱うとき |
| `durable-data-design` | DBスキーマ・エンティティ・削除方針を決めるとき |
| `performance-sizing` | ループ内クエリ・大量データ・長時間処理を実装するとき |
| `incident-response` | 本番障害・データ破損・漏洩が発生したとき |

## 設計方針

- 各スキルの `description` は **発動条件のみ** を記述(プロセスを要約しない)
- `vibe-coding-overview` は複数領域にまたがる横断判定用の索引。単一領域の作業では個別スキルが直接発動する
- 致命傷の回避に集中し、ベストプラクティスの網羅は目指さない

## License

MIT

## 免責

本スキル集は AI コーディング時の一般的なガードレールであり、個別ケースの法務・セキュリティ判断を代替するものではありません。法的リスク・セキュリティインシデント時は専門家に相談してください。
