name: refunds-and-booking-cancellation
description: 旧 refund/cancellation skill の compatibility wrapper です。既存の docs や prompt が `refunds-and-booking-cancellation` を参照しているときに使い、generic な reservation refund workflow は `refunds-and-cancellation-workflows` へ読み替えてください。
---

# Legacy Wrapper: refunds-and-booking-cancellation

この skill 名は互換性のために残しています。

- 新しい実装:
  - `skills/refunds-and-cancellation-workflows/SKILL.md`
- tutor/student の historical wording が必要:
  - `skills/example-tutor-marketplace/SKILL.md`

## 読み替え

- `booking` → `reservation`
- `buyer` / `student` → `customer`
- `seller` / `tutor` → `provider`
