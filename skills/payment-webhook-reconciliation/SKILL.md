---
name: payment-webhook-reconciliation
description: Stripe webhook event と redirect recovery path から marketplace の booking state を整合させるスキルです。Stripe を payment / refund state の durable source of truth とし、app database を retry 下でも安全に更新する必要があるときに使ってください。
---

# 支払い webhook の整合処理

## 概要

webhook 処理は脇役ではありません。booking と refund state を整合させる durable な層です。retry、metadata 欠落、一時的な database failure、順不同の user navigation に耐えられるように作ってください。

## 使う場面

- Stripe Checkout の支払い完了または失敗
- refund lifecycle の更新
- connected account status の同期
- webhook を待つ間に best-effort の状態復旧を行う redirect page

次の用途には使わないでください。
- client-only の payment status check
- Stripe 以外の payment provider
- state を失っても許容される stateless demo

## コアプロセス

1. **署名検証を守る**
   - Node.js runtime を使う
   - raw request body を `request.text()` で読む
   - `stripe-signature` を必須にする
   - `STRIPE_WEBHOOK_SECRET` で検証する

2. **対応するイベントだけを dispatch する**
   - `checkout.session.completed`
   - `checkout.session.async_payment_succeeded`
   - `checkout.session.async_payment_failed`
   - `checkout.session.expired`
   - `account.updated`
   - `refund.created`
   - `refund.updated`
   - `refund.failed`

3. **fallback lookup 付きで booking 行を更新する**
   - まず `metadata.bookingId` を試す
   - それで更新できなければ `stripe_checkout_session_id = session.id` に fall back する
   - refund は `stripe_refund_id = refund.id` に fall back する

4. **最低限有用なフィールドを保存する**
   - payment path: booking `status`, `stripe_payment_status`, `stripe_payment_intent_id`
   - refund path: booking `status`, `stripe_refund_id`, `refund_status`, `refund_amount`, `refunded_at`, `refund_failure_reason`
   - account path: tutor readiness sync

5. **失敗意味論を意図的に使う**
   - DB update が失敗したら `500` を返す
   - イベント成功を装わず、Stripe の retry に任せる
   - 失敗した event type と identifier を記録する

6. **webhook を置き換えずに redirect ベースの UX 復旧を足す**
   - success page は `session_id` で Checkout Session を引き、`paid` に同期してよい
   - cancel page は buyer 所有の `pending` booking を `cancelled` に変換してよい
   - どちらも authoritative ではなく best-effort path である

イベント方針は `event-matrix.md` を参照してください。

## よくある正当化

| Rationalization | Reality |
|---|---|
| 「success page が booking を更新しているから webhook は緩くていい」 | redirect は欠落し得ます。retry 可能な経路は webhook です。 |
| 「DB update に失敗しても 200 を返して後で直せばいい」 | 200 を返すと、Stripe の組み込み retry 機構を捨てることになります。 |
| 「metadata は常にあるから fallback lookup は不要」 | fallback lookup は drift や古い row に対する安価な保険です。 |
| 「webhook route でも middleware は有効のままでいい」 | request body への干渉はすべて署名検証失敗のリスクになります。 |

## レッドフラグ

- webhook route が raw body 検証を使っていない
- DB failure でも 2xx を返している
- 更新ロジックが session id だけ、または metadata だけで lookup している
- success page が paid-state 更新の唯一の手段として扱われている
- seller readiness が app DB にあるのに `account.updated` を無視している

## 検証

- [ ] webhook 署名検証に raw body と `STRIPE_WEBHOOK_SECRET` を使っている
- [ ] 対応イベントが明示的な booking / account 更新 path に対応付いている
- [ ] checkout 更新が metadata を先に、session id を次に試している
- [ ] refund 更新が metadata を先に、refund id を次に試している
- [ ] DB failure で `500` を返している
- [ ] success page と cancel page は authoritative ではなく best-effort recovery のみ行っている
