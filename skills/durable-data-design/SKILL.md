---
name: durable-data-design
description: Use when designing DB schemas, entity relationships, deletion policies, money/time fields, or idempotency for external API calls - prevents migration hell and data corruption that cannot be undone
---

# Durable Data Design

## Overview

> コードは変えられても、データ構造はそう簡単には変えられない。

実装後の破壊的スキーマ変更は本番データと共に地獄を生む。**最初に正しく設計する** ためのチェックリスト。

## When to Use

- 新規エンティティ/テーブルの追加
- 削除機能の実装
- 金額・日時・通貨を扱うフィールド
- 外部APIを呼び出す処理(決済・通知・データ連携)
- トランザクション境界が複数リソースにまたがる処理

## Pre-Schema Checklist

### 1. エンティティと関係の明示
- [ ] 主要エンティティを列挙(User, Order, Product…)
- [ ] 関係性を図示(1:N, N:N, 所有/参照)
- [ ] 集約境界(どこまでを1トランザクションで守るか)

### 2. 削除方針を最初に決める

| 方式 | 使用場面 | 実装 |
|---|---|---|
| 物理削除 | 本当に不要、監査義務なし | `DELETE FROM` |
| 論理削除 | 復旧可能性、監査証跡必要 | `deleted_at TIMESTAMP NULL` |
| 別テーブル退避 | 大量データで性能劣化を避けたい | `*_archive` テーブルに移動 |

**関連データのカスケード動作(CASCADE / SET NULL / RESTRICT)を全FKで明示。**

### 3. 金額・日時

| 項目 | 推奨 |
|---|---|
| 金額 | 整数型(円なら円、USDならminor units = cents)。`FLOAT`禁止 |
| 通貨 | `amount` と `currency` を必ずペアで持つ |
| 日時 | 保存は **UTC**、表示のみタイムゾーン変換 |
| 型 | `TIMESTAMP WITH TIME ZONE` / ISO8601文字列 |

### 4. 冪等性(外部API呼び出し)

- [ ] リトライされても二重課金・二重送信しない設計
- [ ] 冪等性キー(UUID) をリクエストに含める
- [ ] サーバー側で `(idempotency_key, result)` を記録
- [ ] Stripe等の外部サービスは native idempotency key を使う

### 5. トランザクション境界

- [ ] 何と何を同時にコミット/ロールバックすべきか明示
- [ ] 外部API呼び出しはトランザクション外に出す(長時間ロック回避)
- [ ] 分散処理は Outbox Pattern / Saga を検討

## Red Flags

| 状況 | 将来の痛み |
|---|---|
| 「とりあえず string で」 | 列分割・型変更で本番停止マイグレ |
| 「円は FLOAT でいいや」 | 0.1+0.2 = 0.30000000000000004 問題 |
| 「タイムゾーンはJSTで保存」 | 海外展開・夏時間で破綻 |
| 「削除は DELETE 文で」 | 監査対応不能、誤削除復旧不能 |
| 「冪等性は後で」 | 決済二重引き落としで訴訟 |

## Common Mistakes

- **MySQL enum をDBに直書き:** 値追加/削除が `ALTER TABLE` 必須で本番停止リスク。文字列 + アプリ側バリデーションが無難。PostgreSQL の enum 型は `ALTER TYPE ... ADD VALUE` で比較的安全に追加可能(削除は不可)なので一概に禁止ではない。
- **created_at/updated_at を忘れる:** 全テーブルに入れる。トラブル調査の生命線。
- **論理削除のユニーク制約:**
  - PostgreSQL / SQLite: 部分ユニークインデックス `CREATE UNIQUE INDEX ... ON users(email) WHERE deleted_at IS NULL`
  - MySQL / MariaDB: 部分インデックス非対応。`deleted_at` を `NOT NULL DEFAULT '1970-01-01'` で扱い `(email, deleted_at)` でユニーク、または生成列 `deleted_flag` を用意する回避策が必要
