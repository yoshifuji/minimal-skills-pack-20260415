---
name: role-based-portals-and-navigation
description: role-aware な公開 discovery、provider portal、customer portal を構築するスキルです。公開一覧、role-aware navigation、provider 向け運用画面、customer の reservation/order 履歴、status 駆動の user message を実装するときに使ってください。
---

# Role-Based Portals And Navigation

## 概要

この kind の marketplace には 3 つの UX 面があります。public discovery、provider operations、customer operations です。各画面は、内部実装ではなく、user の次の行動を中心に組み立ててください。

## コアプロセス

1. **まず public discovery を作る**
   - home page は public read model から provider を一覧表示する
   - detail page は public profile row と active listing を読む
   - readiness や受付状態は、生の内部 flag ではなく badge や短い文言で見せる

2. **navigation に role state を見せる**
   - Anonymous: browse、login、sign up
   - Customer: customer portal
   - Provider: provider portal
   - current user 名と logout 導線を出す

3. **provider portal を operations funnel として作る**
   - Step 1: payment onboarding status
   - Step 2: public profile
   - Step 3: listings
   - Step 4: incoming reservations or orders
   - provider に、なぜ受け付け可能なのか / でないのかを常に分からせる

4. **customer portal を台帳として作る**
   - summary count
   - 次の upcoming reservation
   - 完全な reservation table
   - refund status column
   - 実際にキャンセル可能な row にだけ cancellation を出す

5. **status 文言は product language にする**
   - `paid`、`pending`、`payment_failed`、`expired`、`cancelled` をそのまま露出しない
   - refund status にも人が読める label を付ける
   - role 名や listing 名は project 用語へ合わせる

6. **form を薄く保つ**
   - client form → API route → `router.refresh()`
   - component に reservation や payment の business logic を重複させない

## よくある正当化

| Rationalization | Reality |
|---|---|
| 「とりあえず生の status code を出せばいい」 | payment state は user の不安を増やします。人が読める status は product の一部です。 |
| 「provider portal は後で複数ページに分ければいい」 | starter としては、明確な次の行動を持つ 1 つの運用 funnel のほうが機能します。 |
| 「匿名 user に action form を見せて、後で失敗させればいい」 | payment flow に入る前に role 境界を明確にすべきです。 |
| 「customer portal は table だけで十分」 | summary count と次の予定があると、走査コストが下がり、意図ある app に見えます。 |

## レッドフラグ

- public page が private な運用文言を使っている
- role-based navigation が page ごとに不一致
- checkout 作成失敗まで provider readiness が隠れている
- customer portal がキャンセル不能 row に cancel button を出している
- `lib/` に置くべき business rule を component が実装している

## 検証

- [ ] home page に provider と public listing が表示される
- [ ] detail page が適切な role にだけ reservation / order action を許可する
- [ ] provider portal が readiness、profile 編集、listing、incoming reservations を見せる
- [ ] customer portal が reservation status、refund status、cancellation 導線を正しく出す
- [ ] anonymous / customer / provider で navigation が正しく切り替わる
