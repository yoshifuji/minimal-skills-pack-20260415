# イベント処理マトリクス

## Checkout 成功
- イベント: `checkout.session.completed`, `checkout.session.async_payment_succeeded`
- パッチ:
  - `status = paid`
  - `stripe_payment_status = session.payment_status ?? "paid"`
  - `stripe_payment_intent_id`

## Checkout 失敗 / 期限切れ
- `checkout.session.expired` → `status = expired`
- `checkout.session.async_payment_failed` → `status = payment_failed`

## アカウント同期
- `account.updated` → 講師アカウントの readiness フィールドを更新する

## 返金同期
Stripe の返金状態を予約パッチへ変換するヘルパーを使います。
- アクティブな返金（`pending`, `requires_action`, `succeeded`）→ 予約 `status = cancelled`
- 失敗 / キャンセルされた返金 → 予約 `status = paid`
