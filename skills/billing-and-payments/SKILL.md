---
name: billing-and-payments
description: Webアプリで決済、請求状態、Webhook、返金を実装するときに使ってください。Stripe Checkout、支払い完了後の状態更新、Webhook 署名検証、返金、失敗時の再試行や整合性維持が必要なときに使います。
---

# Billing And Payments

## 概要

支払い処理は「画面遷移が通る」だけでは完成ではありません。開始、成功、失敗、期限切れ、返金、Webhook 再送まで含めて整合して初めて成立します。お金まわりは曖昧さを残さず、**状態遷移と source of truth** を先に決めてください。

## コアプロセス

1. **支払い開始前の保存方針を決める**
   - 先に注文や予約行を作るか
   - payment intent や checkout session とどう対応づけるか
   - lookup に使う id を何にするか

2. **支払い状態を定義する**
   - pending
   - paid
   - failed
   - expired
   - refunded
   - project で必要な状態だけを持つ

3. **支払い開始の route を thin に保つ**
   - current user を確認する
   - 対象 resource の妥当性を確認する
   - 金額を server 側で決める
   - session / intent を作る
   - 対応する local row に id を保存する

4. **Webhook を本命にする**
   - signature verify
   - event type ごとの分岐
   - local row の lookup
   - DB 更新失敗時の retry
   - best effort の redirect recovery と区別する

5. **返金を独立した flow として扱う**
   - 誰が返金を起こせるか
   - どの状態で返金可能か
   - idempotency key
   - local state と webhook の整合

6. **金額と通貨の計算を server 側で固定する**
   - client から受け取った金額を信じない
   - fee や税の扱いを helper に寄せる
   - rounding を project で統一する

状態遷移は `../../references/支払い状態遷移.md`、支払い通知の確認項目は `../../references/支払い通知チェックリスト.md`、返金の基本は `../../references/返金ガイド.md` を参照してください。

## よくある正当化

| Rationalization | Reality |
|---|---|
| 「success_url で paid にすれば十分」 | redirect だけでは抜けや race が起きます。Webhook を本命にしてください。 |
| 「client から送られた金額をそのまま使えばよい」 | 金額は server 側で確定すべきです。 |
| 「返金は button を押したら終わり」 | 返金は money movement であり、状態同期まで必要です。 |
| 「失敗時はもう一度押せばよい」 | 金融操作は再試行ではなく冪等性で守ります。 |

## レッドフラグ

- payment の local row がなく外部 id と対応づかない
- Webhook の署名検証がない
- DB 更新失敗でも 2xx を返している
- 金額を client 任せにしている
- 返金に idempotency key がない

## 検証

- [ ] 支払い開始前の local row 保存方針が決まっている
- [ ] state machine を説明できる
- [ ] Webhook を本命、redirect を補助として分けている
- [ ] signature verify がある
- [ ] 返金 flow に認可と冪等性がある
