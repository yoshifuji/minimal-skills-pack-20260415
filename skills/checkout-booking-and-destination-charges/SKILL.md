name: checkout-booking-and-destination-charges
description: 旧 booking/Checkout skill の compatibility wrapper です。既存の docs や prompt が `checkout-booking-and-destination-charges` を参照しているときに使い、generic な単一 provider 決済は `stripe-single-seller-checkout` へ読み替えてください。
---

# Legacy Wrapper: checkout-booking-and-destination-charges

この skill 名は互換性のために残しています。

- 新しい実装:
  - `skills/stripe-single-seller-checkout/SKILL.md`
- tutor/student の historical wording が必要:
  - `skills/example-tutor-marketplace/SKILL.md`

## 読み替え

- `booking` → `reservation`
- `seller` / `tutor` → `provider`
- `buyer` / `student` → `customer`
- `offering` → `listing`
