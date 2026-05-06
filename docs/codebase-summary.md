# VietPay V1 — コードベースサマリー

**生成日:** 2026-05-05 · **ソース:** Repomix Agent + 依存関係グラフアナライザー
**スコープ:** V2前のリバースエンジニアリング・ギャップ分析および移行計画の前提条件

---

## 1. 概要

| メトリクス | 値 |
|--------|-------|
| 総LOC | 124,000 |
| 言語 | TypeScript (82%)・SQL (11%)・YAML (7%) |
| サービス | NestJSマイクロサービス6つ |
| APIエンドポイント | 89 |
| DBテーブル | 47（共有スキーマ1つ — アンチパターン） |
| テストカバレッジ（行） | 71.2%（目標 ≥ 80%） |
| コードベース年数 | 4年（2022年Q3初コミット） |
| アクティブコントリビューター（90日） | 12名 |
| 直近本番デプロイ | 6日前 (v1.18.4) |

---

## 2. サービスインベントリ

| サービス | LOC | 最終更新 | テストカバレッジ | ステータス |
|---------|-----|--------------|----------|--------|
| `auth-service` | 12,400 | 3日前 | 88% | ✅ 安定 |
| `kyc-service` | 8,100 | 21日前 | 79% | ✅ 安定 |
| `wallet-service` | 18,200 | 昨日 | 64% | ⚠ ゴッドモジュール |
| `transfer-service` | 26,300 | 2日前 | 81% | ⚠ ホットパス・複雑 |
| `payment-service` | 22,800 | 5日前 | 73% | ✅ Sepay統合 |
| `notification-service` | 9,600 | 14日前 | 86% | ✅ 安定 |

**共有インフラ**:
- `shared/lib/`（8.4k LOC）— ドメイン型・エラークラス・バリデーター
- `infra-cdk/`（12k LOC TypeScript）— AWS CDKスタック
- `tests/`（約24k LOC、テストファイル・フィクスチャ・ファクトリー）

---

## 3. 依存関係グラフのハイライト

### サービス間依存関係

```
auth → kyc                       (kyc.completedイベントをコンシューム)
wallet ↔ transfer  (循環)        (アンチパターン・DB共有)
transfer → payment               (Sepayリレー)
notification ← 全サービス         (Kafkaコンシューマー)
```

**検出された循環**: 2件
- `wallet ↔ transfer`（共有DBスキーマ `vietpay`、両サービスが同一テーブルを読み書き）
- `wallet → ledger-helper → wallet`（内部ヘルパーがwalletに逆インポート）

**境界違反**: DDDパターンに反するクロスサービスの直接importが18件存在いたします（例：`transfer-service/src/...`が`wallet-service/src/...`をAPIではなく直接import）。

---

## 4. 技術スタック

### ランタイム
- Node.js 20 LTS
- NestJS 10
- TypeScript 5.3
- React Native 0.74 + Expo SDK 51（モバイル）

### データ
- PostgreSQL 15（RDS Multi-AZ、ap-southeast-1）
- Redis 7（ElastiCache、クラスターモード）
- Kafka 3.7（MSK、ブローカー3台）
- S3（KYCドキュメント・領収書）+ KMS

### 統合
- Sepay（VietQR + 銀行送金アグリゲーター）
- VNPT eKYC（CMND/CCCDスキャン + ライブネス）
- FCM + APNs（プッシュ通知）
- Stringee（SMS OTP）

### インフラ
- AWS CDK（TypeScript）IaC
- ECS Fargate（k8s不使用）
- CloudFront CDN
- DataDog（メトリクス + ログ + APM）
- Sentry（モバイル + バックエンドエラー）

---

## 5. ホットスポット（リファクタリング時に特に注意すべき箇所）

| ファイル | LOC | 循環的複雑度 | 変更頻度 | リスク |
|------|-----|----------------------|-----------------|------|
| `wallet-service/wallet.controller.ts` | 814 | 47 | 高 | 🔴 ゴッドクラス |
| `transfer-service/transfer.service.ts` | 612 | 38 | 非常に高 | 🔴 クリティカルパス |
| `transfer-service/legacy-mapper.ts` | 478 | 29 | 中 | 🔴 テストゼロ |
| `payment-service/sepay-adapter.ts` | 392 | 22 | 中 | 🟡 リトライロジック複雑 |
| `wallet-service/ledger.service.ts` | 348 | 18 | 高 | 🟡 不変条件未ドキュメント |

---

## 6. DBスキーマスナップショット

47テーブルが単一の共有スキーマ `vietpay` 内に存在いたします（アンチパターン — V2で分割予定）：

```
authテーブル       (8): users, sessions, device_keys, otp_log, ...
kycテーブル        (5): kyc_records, kyc_documents, manual_review, ...
walletテーブル     (7): wallets, wallet_balances, ledger_entries, ...
transferテーブル   (9): transfers, idempotency_keys, daily_limits, ...
paymentテーブル    (8): payments, qr_codes, webhooks, linked_banks, ...
notification (4): notifications, prefs, device_tokens, ...
audit / shared (6): audit_log, feature_flags, system_events, ...
```

**外部キー**: 64件。クロスドメインFK（アンチパターン）：12件（wallet → users、transfer → wallets等）— V2ではイベント経由の結果整合性に置き換えます。

---

## 7. APIサーフェス

**89のRESTエンドポイント**を6サービスに展開しております：

```
auth-service:           14エンドポイント（signup, otp, login, biometric, ...）
kyc-service:            8エンドポイント（submit, status, documents, review, ...）
wallet-service:         11エンドポイント（balance, history, ledger, ...）
transfer-service:       18エンドポイント（transfer, cancel, history, limits, ...）
payment-service:        24エンドポイント（banks, topup, qr, webhooks, ...）
notification-service:   14エンドポイント（inbox, prefs, devices, ...）
```

**公開向け**（モバイルクライアント）：67エンドポイント
**内部管理**：22エンドポイント

OpenAPI仕様は `docs/openapi-v1.18.yaml`（自動生成）にございます。

---

## 8. ビルド & デプロイ

| | |
|---|---|
| モノレポ | Turborepo |
| パッケージマネージャー | pnpm 9 |
| CI | GitHub Actions（サービスごとにlint + test + コンテナビルド） |
| CD | GitHub Actions → CodeDeploy → ECS Blue/Green |
| モバイルビルド | Expo EAS |
| ビルド時間 | 約14分（フルパイプライン） |
| デプロイ時間 | ステージング約8分、本番約12分 |

---

## 9. 本番フットプリント

| | |
|---|---|
| MAU | 312,000（2026年5月） |
| デイリーアクティブ | 41k |
| 日次トランザクション | 78k（平均）・ピーク145k |
| 連携銀行口座 | 287k |
| ストレージ（KYCドキュメントS3） | 3.4 TB |
| 日次Sepayコール | 約210k |
| コスト（月額AWS） | 約$8,400 |

---

## 10. 既知の問題 / 技術的負債

（全インベントリは `docs/tech-debt-inventory.md` にございます）

- ゴッドクラス14件（>500 LOC、>25循環的複雑度）— 上位3件は§5に記載
- wallet + transfer間の共有DBスキーマ（V2で分割必須）
- 古い依存関係23件（うち4件はCVE Highあり）
- クリティカルモジュール4件のテストカバレッジ0%：`legacy-mapper`・`audit-shipper`・`slo-emitter`・`compliance-classifier`
- コード内に「Refund」機能に言及する未対応TODO 6件（アーキテクチャ未整備）
- eKYC手動レビューキューにSLAトラッキング未実装

---

## 11. リバースエンジニアリングノート（AI抽出）

Reverse Engineer Agentがコードを解析した結果でございます：

### アーキテクチャ上の意思決定（コードから推定）

- **冪等性**: Redis SETNXとTTL 24時間、フォールバックでDB参照。同パターンが6箇所で繰り返されているため、ヘルパーへ抽出すべきです。
- **監査ログ**: 全write操作で`audit-log` Kafkaトピックへイベント送信。コンシューマーがS3 Object Lockへ7年保管（SBV（ベトナム国家銀行）コンプライアンス対応）。
- **フィーチャーフラグ**: LaunchDarkly使用。本番で23フラグがアクティブです。
- **マルチリージョン**: `ap-southeast-1`のみアクティブ。`eu-central-1`はプロビジョニング済みですが非アクティブ（休眠DR）です。

### 想定外 / 未ドキュメントの挙動（characterizationテストでピン留めが必要）

- `transfer-service/legacy-mapper.ts:142` — V1ではDBトランザクションがロールバックしてもHTTP 200を返す挙動（落とし穴）。モバイルアプリがこれに依存しているため、characterizationテストで必ずピン留めいたします。
- `wallet-service/balance.service.ts:88` — キャッシュ30秒、ただしtransferイベントによる無効化が非同期のみ（レースウィンドウ約50ms）。モバイルUIは古いデータをグレースフルに処理いたします。
- `payment-service/sepay-adapter.ts:201` — 指数バックオフで3回リトライ、ただし3回目は別エンドポイント（レガシーv0 API）を使用。Sepayベンダーがサポートスレッド#4421にて確認済みです。

---

## 12. V2向けレコメンデーション

**アーキテクチャ**:
- 共有DBスキーマを分割：`wallet_db` と `transfer_db` を分離（高リスク移行 → ストラングラーフィグ + デュアルライト方式）
- walletゴッドクラスを4モジュールへ分解：`wallet-balance`・`wallet-ledger`・`wallet-policy`・`wallet-history`
- 冪等性パターンを共有ライブラリへ抽出

**品質**:
- カバレッジゼロのモジュール4件はリファクタ前にcharacterizationテストを追加
- 古い依存関係23件をアップグレード（CVE Critical 4件含む）
- 想定外挙動（上記3件の未ドキュメント挙動）をcharacterizationテストでピン留め

**プロセス**:
- ビッグバンではなくストラングラーフィグで移行（312k MAUに対するリスク低減）
- フルロールアウト後72時間のハイパーケアオンコール体制
- カットオーバー前にお客様SME（プロダクトオーナー）の承認を必須といたします

---

**次のステップ**: Step 02ギャップ分析（お客様V2要件ブリーフと比較）へ投入し、gap-analysis.mdを生成いたします。
