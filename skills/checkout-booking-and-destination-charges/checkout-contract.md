# チェックアウト契約

## リクエストボディ
- `offeringId: uuid`
- `scheduledFor: ISO datetime`
- `notes?: string`

## 必須のセッションメタデータ
- `bookingId`
- `offeringId`
- `tutorId`
- `studentId`

## insert 時に必須の予約フィールド
- `lesson_offering_id`
- `tutor_id`
- `student_id`
- `scheduled_for`
- `notes`
- `status = pending`
- `price_amount`
- `currency`
- `application_fee_amount`

## 必須のリダイレクト URL
- success → `/checkout/success?session_id={CHECKOUT_SESSION_ID}`
- cancel → `/checkout/cancel?booking_id={booking.id}`
