---
name: performance-sizing
description: Use when writing loops containing DB queries, search features over large datasets (100K+ rows), long-running synchronous handlers, or features whose cost scales with user count - prevents N+1, timeouts, and production slowdowns
---

# Performance Sizing

## Overview

「ローカルで速い = 本番で速い」は嘘。**データ量とアクセス頻度に対するスケール試算** を実装前に行い、N+1問題と長時間ハンドラを機械的に排除する。

## When to Use

- ループ/配列map内でDBまたは外部APIを呼ぶコード
- 検索/一覧表示機能(特に100K件以上想定)
- ファイル処理・バッチ・レポート生成(数秒以上かかる処理)
- Webhook/決済/メール送信のような外部I/O
- ページング未実装の一覧API

## Scale Sanity Checklist

### 1. N+1問題の検出
```
# Bad: ユーザー1000人それぞれに追加クエリ
users = User.all
users.each { |u| puts u.orders.count }

# Good: JOIN / プリロード / カウンタキャッシュ
users = User.includes(:orders)
```
**コードレビューで `for`/`forEach`/`map` 内の `await db.*` / `SELECT` / `find_by` を機械的に警告する。**

### 2. インデックス設計
- [ ] WHERE句・JOIN条件・ORDER BYのカラムにインデックス
- [ ] 複合インデックスの列順は **選択性の高い順**
- [ ] 論理削除がある場合は `WHERE deleted_at IS NULL` を含む部分インデックス
- [ ] `EXPLAIN` で Seq Scan が無いか確認(大テーブルのみ)

### 3. 大量データのレイテンシ見積

| 件数 | インデックスあり | インデックスなし |
|---|---|---|
| 1K | 即時 | 即時 |
| 100K | 数ms | 数秒 |
| 10M | 数十ms | 数十秒〜タイムアウト |

**100万件想定で許容レイテンシを満たすか試算。**

### 4. 長時間処理は非同期化
- HTTPハンドラは原則3秒以内に返す
- メール送信・PDF生成・外部API連携は **ジョブキュー**(Sidekiq / SQS / Cloud Tasks)
- LLM呼び出しはストリーミング or 非同期 + ポーリング/WebSocket

### 5. ページング
- 一覧APIは常に `LIMIT` を強制
- オフセットページングは深いページで遅い → カーソルページング(id/created_at基準)

## Red Flags

| コード/状況 | 問題 |
|---|---|
| `for item in list: db.query(...)` | N+1 |
| `SELECT * FROM ... ORDER BY created_at` (インデックスなし) | フルスキャン |
| HTTPハンドラ内で外部APIを複数回 | タイムアウトリスク |
| 「件数少ないから」 | 本番で伸びる前提がない |
| ページング未実装の一覧エンドポイント | メモリ枯渇 |

## Common Mistakes

- **ローカルの少データでベンチ:** 本番スキーマに近いダミーデータで検証する。
- **キャッシュで隠蔽:** 根本のN+1を残してキャッシュで誤魔化すと障害時に顕在化。
- **インデックス貼りすぎ:** 書き込み性能が落ちる。必要なものだけ。
