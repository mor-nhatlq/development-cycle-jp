# VietPay プロジェクトチェンジログ

## [v1.0.0] — 2026-06-05（MVPローンチ）

### 追加
- Idempotency Key付きP2P送金（1取引1,000万VND · 1日1億VND）
- Sepay VietQR決済連携
- VNPT eKYCアダプタ（ライブネス検出付き）
- ウォレット台帳サービス（PostgreSQL、アトミックなdebit/credit）
- カーソルページネーション付きトランザクション履歴
- FCM + APNsプッシュ通知
- 銀行口座連携（Vietcombank、Techcombank、MB）
- iOS + Androidアプリ（TestFlight #142 · Play Store内部トラック）

### セキュリティ
- OWASP Top-10監査クリア
- SBV（ベトナム国家銀行）規制準拠のAML/KYCコンプライアンス
- PCI-DSS Level 1準備
- サービス間mTLS強制適用

### パフォーマンス
- Transfer p95 < 3秒（エンドツーエンド）
- 99.9%稼働率SLA目標
- マルチリージョンデプロイ（ap-southeast-1、eu-central-1）

## [v0.9.0-rc] — 2026-05-28

- ハードニングスプリント完了
- 218/220テスト通過（カバレッジ96%）
- スモークテスト 24/24 ✓

## [v0.8.0-beta] — 2026-05-15

- MVPの8フェーズ全てコード完成
- コードレビュー + セキュリティ監査クローズ（HIGH指摘3件修正）

## [v0.1.0] — 2026-04-01

- プロジェクトキックオフ · Planner Agentが12週間計画を生成
- 技術スタック承認（NestJS · PostgreSQL · React Native · Kafka）
