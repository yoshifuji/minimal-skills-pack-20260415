# スキーマ・チェックリスト

## テーブル
- `profiles`
- `tutor_profiles`
- `lesson_offerings`
- `bookings`
- `public_tutor_directory`

## トリガー関数
- `set_updated_at()`
- `handle_new_user()`
- `public_tutor_directory` 用の refresh function(s)

## 予約ライフサイクルに必須のカラム
- `status`
- `stripe_checkout_session_id`
- `stripe_payment_intent_id`
- `stripe_payment_status`
- `stripe_refund_id`
- `refund_status`
- `refund_amount`
- `refunded_at`
- `refund_failure_reason`

## 必須ステータス値
- bookings: `pending`, `paid`, `expired`, `cancelled`, `payment_failed`
- refunds: 関連する箇所では Stripe のステータス値をそのまま保存する
