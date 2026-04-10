# Stripe イベントマトリクス

| イベント | Source of truth | 想定 DB 効果 | 備考 |
|---|---|---|---|
| `checkout.session.completed` | Checkout Session | booking → `paid` | 保存: `stripe_payment_status`, `stripe_payment_intent_id` |
| `checkout.session.async_payment_succeeded` | Checkout Session | booking → `paid` | 非同期決済成功時 |
| `checkout.session.async_payment_failed` | Checkout Session | booking → `payment_failed` | `payment_status` も保存 |
| `checkout.session.expired` | Checkout Session | booking → `expired` | 未完了 Checkout の期限切れ |
| `account.updated` | Connected Account | tutor profile status sync | `details_submitted`, `charges_enabled`, `payouts_enabled` |
| `refund.created` | Refund | refund state sync | optimistic update 後の整合用 |
| `refund.updated` | Refund | refund state sync | `succeeded`, `requires_action` などを反映 |
| `refund.failed` | Refund | booking state の再評価 | refund が失敗したら booking を `paid` 側へ戻す |

## 更新時の lookup 順序

### Checkout session 更新
1. `metadata.bookingId`
2. fallback: `stripe_checkout_session_id = session.id`

### Refund 更新
1. `metadata.bookingId`
2. fallback: `stripe_refund_id = refund.id`

## failure 方針
DB 更新に失敗したら webhook は `500` を返し、Stripe の再送を使って整合性回復する。
