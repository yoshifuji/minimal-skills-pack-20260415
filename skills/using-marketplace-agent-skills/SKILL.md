name: using-marketplace-agent-skills
description: 旧 tutor marketplace 入口 skill の compatibility wrapper です。既存の docs や prompt が `using-marketplace-agent-skills` を参照しているときに使い、generic な service marketplace なら `marketplace-foundation-workflow`、legacy tutor example なら `example-tutor-marketplace` へ誘導してください。
---

# Legacy Wrapper: using-marketplace-agent-skills

この skill 名は互換性のために残しています。新規の入口としては使わず、次のどちらかへ移ってください。

- generic な service marketplace を始める:
  - `skills/marketplace-foundation-workflow/SKILL.md`
- 既存の tutor/student 例をたどる:
  - `skills/example-tutor-marketplace/SKILL.md`

## 読み替え

- `role-based-marketplace-pages` → `role-based-portals-and-navigation`
- `checkout-booking-and-destination-charges` → `stripe-single-seller-checkout`
- `refunds-and-booking-cancellation` → `refunds-and-cancellation-workflows`

## 注意

- 新しい project では、この wrapper に新ルールを足さない
- tutor 固有語彙が必要なら `example-tutor-marketplace` を使う
- 決済と refund の実装は新 skill 名へ読み替える
