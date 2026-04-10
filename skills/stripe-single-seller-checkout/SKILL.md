---
name: stripe-single-seller-checkout
description: destination charges と application fee を使って、単一 provider 向けの reservation/order と Stripe Checkout Session を作成するスキルです。customer が provider 所有の listing を予約または注文し、プラットフォームと connected account に支払いを分配する必要があるときに使ってください。
---

# Stripe Single Seller Checkout

## 概要

reservation lifecycle は、支払い完了前から始まります。まず耐久性のある `pending` reservation を作成し、その後で後続の照合に十分な metadata を持つ Checkout Session を作り、最後に Stripe の session id を保存します。これにより、redirect、webhook、refund がすべて同じ行を指せるようになります。

## コアプロセス

1. **customer を認証 / 認可する**
   - サインイン済み customer を必須にする
   - provider と匿名 user は拒否する

2. **listing と provider の readiness を検証する**
   - listing が存在し、active であること
   - provider が connected account id を持っていること
   - checkout 作成前に provider account を同期すること
   - `charges_enabled` または `payouts_enabled` が false なら reservation を止めること

3. **先に reservation 行を作る**
   - `status = pending`
   - 価格と通貨を listing からコピーする
   - `scheduled_for`, `notes`, provider id, customer id, fee amount を保存する

4. **プラットフォーム手数料を計算する**
   - basis points を env から読む
   - 不正値は clamp / fallback する
   - `application_fee_amount` を reservation 行に保存する

5. **Checkout Session を作る**
   - `mode: "payment"`
   - 選択した listing の line item を 1 つ
   - 取得できるなら `customer_email`
   - metadata には `reservationId`, `listingId`, `providerId`, `customerId` を必ず入れる
   - `payment_intent_data.application_fee_amount`
   - `payment_intent_data.transfer_data.destination`
   - `payment_intent_data.on_behalf_of`

6. **session id を保存する**
   - `stripe_checkout_session_id` を reservation に保存する
   - hosted Checkout URL を client に返す

7. **内部失敗をきれいに処理する**
   - reservation insert 後に checkout 作成が失敗した場合、永久に `pending` の行を残さない
   - catch path で reservation を `cancelled` に移す（または同等に無効化する）

8. **redirect 復旧を支える**
   - success page は `session_id` から best-effort で `paid` 同期できる
   - cancel page は customer 所有の `pending` reservation を best-effort で `cancelled` に変換できる

契約チェックリストは `checkout-contract.md` を参照してください。

## よくある正当化

| Rationalization | Reality |
|---|---|
| 「先に Checkout Session を作って、reservation は後で作ればいい」 | それでは success / cancel / webhook から照合する耐久的なローカル行がありません。 |
| 「session id だけで十分、metadata は任意でいい」 | metadata があると、より強く、デバッグしやすい主検索経路になります。 |
| 「checkout 前に provider account を再同期する必要はない」 | onboarding 後に readiness は変化します。入金前に同期してください。 |
| 「失敗した pending 行を残しても問題ない」 | それらは customer の台帳を汚し、復旧ロジックを混乱させます。 |

## レッドフラグ

- `pending` reservation なしで checkout を作っている
- session metadata に `reservationId` がない
- platform fee を計算しているのに reservation に保存していない
- route handler が provider ユーザーに reservation 作成を許している
- cancel URL に reservation id が入っていない

## 検証

- [ ] サインイン済み customer だけが checkout session を作成できる
- [ ] active listing と connected provider readiness を先に検証している
- [ ] Stripe Checkout Session 作成前に `pending` reservation を作っている
- [ ] session metadata に reservation / listing / provider / customer id が入っている
- [ ] session id が reservation に保存されている
- [ ] 内部失敗で孤立した `pending` 行が残らない
- [ ] success page と cancel page に best-effort 復旧に必要な識別子が十分ある
