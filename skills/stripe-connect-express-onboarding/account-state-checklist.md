# アカウント状態チェックリスト

## 保存するフィールド
- `stripe_account_id`
- `details_submitted`
- `charges_enabled`
- `payouts_enabled`
- `last_stripe_sync_at`

## UI に出すべきバッジ
- アカウント作成済み / 未作成
- カード決済 ready / 未 ready
- payout ready / 未 ready
- 詳細提出済み / 要対応

## 最低限必要なルート
- `POST /api/connect/account`
- `POST /api/connect/account-link`
- `GET /api/connect/account-link/refresh`
- `POST /api/connect/sync`
- `POST /api/connect/dashboard-link`
