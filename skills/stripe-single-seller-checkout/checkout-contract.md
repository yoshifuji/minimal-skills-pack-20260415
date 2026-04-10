# Checkout 契約

## リクエストボディ

- `listingId: uuid`
- `scheduledFor: ISO datetime`
- `notes?: string`

## 必須のセッションメタデータ

- `reservationId`
- `listingId`
- `providerId`
- `customerId`

## insert 時に必須の予約フィールド

- `service_listing_id`
- `provider_id`
- `customer_id`
- `scheduled_for`
- `notes`
- `status = pending`
- `price_amount`
- `currency`
- `application_fee_amount`

## 必須のリダイレクト URL

- success → `/checkout/success?session_id={CHECKOUT_SESSION_ID}`
- cancel → `/checkout/cancel?reservation_id={reservation.id}`
