---
name: example-tutor-marketplace
description: legacy tutor marketplace overlay です。既存の docs や prompt が tutor/student、lesson offering、booking を前提にしているとき、または旧 tutor marketplace pack の資料を generic marketplace skills に対応付けたいときに使ってください。
---

# Example Tutor Marketplace

## 概要

この skill は、historical な tutor marketplace 例を generic な marketplace skill 群に対応付けるための overlay です。新しい project を始める入口としては使わず、**既存の tutor/student 用語を generic skill に橋渡しする用途** で使ってください。

新規の service marketplace では、まず `marketplace-foundation-workflow` を使ってください。

## 語彙対応

- `tutor` → `provider`
- `student` → `customer`
- `lesson offering` → `service listing`
- `booking` → `reservation`
- `tutor dashboard` → `provider portal`
- `student dashboard` → `customer portal`

## 先に読むもの

1. `docs/講師マーケットプレイス実行手順書.md`
2. `references/examples/tutor-marketplace/データモデル概要.md`
3. `references/examples/tutor-marketplace/デザインブリーフ.md`
4. `references/examples/tutor-marketplace/手動QAフロー.md`
5. `references/examples/tutor-marketplace/実行成果物チェックリスト.md`

Stripe event と env は共通資料を使ってください。

- `references/Stripeイベントマトリクス.md`
- `references/環境変数とシークレット一覧.md`

## Skill 対応

- pack 全体の順序整理:
  - `marketplace-foundation-workflow`
- tutor/student の公開ページと dashboard:
  - `role-based-portals-and-navigation`
- booking + Checkout:
  - `stripe-single-seller-checkout`
- refund / cancellation:
  - `refunds-and-cancellation-workflows`

既存資料で旧 skill 名が出てきた場合は、互換 wrapper として残っている旧 skill を開くか、上の新 skill 名へ読み替えてください。

## 追加ルール

1. UI 上の文言は tutor / student を維持する
2. 内部の責務分離は generic skill に寄せる
3. tutor 固有の状態表や QA は example reference を優先する
4. 新しい app 固有の判断は、この overlay ではなく project 側の spec に書く

## 検証

- [ ] tutor / student / booking / offering の語彙が UI に保たれている
- [ ] skill の選択は generic 名へ読み替えられている
- [ ] example reference が `references/examples/tutor-marketplace/` に集約されている
