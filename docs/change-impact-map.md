# 変更影響マップ ・ V1 → V2

V1モジュール → V2変更のマッピングとなります。

## モジュール影響マトリクス

| モジュール | V1ステータス | V2変更 | リスク |
|---|---|---|---|
| auth-service | 安定 | — | low |
| kyc-service | 安定 | + Vietin eKYCフォールバック | low |
| wallet-service | ⚠ Godモジュール | **分割** ledger-service ＋ balance-cache へ | **HIGH** |
| transfer-service | 安定 | + マルチ通貨フック | medium |
| payment-service | 安定 | + bill-paymentアダプター | medium |
| **NEW** bill-service | — | EVN/インターネット/水道/TVアダプター | medium |
| **NEW** topup-service | — | Viettel/Vinaphone/Mobifone | medium |
| notification-service | 安定 | + bill-paidテンプレート | low |

## データベース影響

```
V1 (shared `vietpay` schema):
  ├─ wallets        → SPLIT to wallet_db.wallets
  ├─ transfers      → SPLIT to transfer_db.transfers
  └─ transactions   → SPLIT to ledger_db.entries

V2 (per-service):
  ├─ wallet_db      (wallet-service owns)
  ├─ ledger_db      (ledger-service owns — NEW)
  ├─ transfer_db    (transfer-service owns)
  ├─ bill_db        (bill-service owns — NEW)
  └─ topup_db       (topup-service owns — NEW)
```

## APIコントラクト影響

- ✅ V1クライアントAPIは全て後方互換性を維持（破壊的変更なし）
- 🔄 内部サービスAPI：12エンドポイントをリファクタリング
- ➕ 8新規エンドポイント追加（請求書＋チャージ）

## モバイルアプリ影響

- 新ボトムタブ「Bills」追加（プレースホルダーを置換）
- ホームダッシュボードにチャージエントリポイント
- 設定：新規請求書プロバイダーをリンク
- アプリバージョンを v1.0 → v2.0 に更新

## 影響範囲の見積り

- **変更コード:** 修正 約32k LOC、新規 約18k LOC
- **再起動サービス（カットオーバー）:** 8件中6件
- **想定ダウンタイム:** 0（デュアルライト戦略）
- **ロールバックウィンドウ:** 7日間（旧DBは読み取り専用）
