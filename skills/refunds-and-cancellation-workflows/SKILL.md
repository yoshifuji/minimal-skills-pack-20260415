---
name: refunds-and-cancellation-workflows
description: future の paid reservation/order をキャンセルし、transfer reversal と application fee reversal 付きの Stripe refund を作成するスキルです。customer が自分の今後の予約や注文をキャンセルでき、marketplace が reservation state、payout state、refund state を揃え続ける必要があるときに使ってください。
---

# Refunds And Cancellation Workflows

## 概要

cancel button は money movement です。明示的な認可、時間ルール、冪等性、optimistic な state update、webhook による整合処理を持つ独立 workflow として扱ってください。

## コアプロセス

1. **厳密に認可する**
   - current user が存在すること
   - current role が customer であること
   - reservation が current user の所有物であること

2. **キャンセル可能性を確認する**
   - reservation status が `paid` であること
   - `scheduled_for` がまだ未来であること
   - payment intent / latest charge が存在すること

3. **意図的に refund を作成する**
   - PaymentIntent を読み、必要なら latest charge を expand する
   - refund は次の指定で作る:
     - `charge`
     - `reverse_transfer: true`
     - `refund_application_fee: true`
     - `reason: "requested_by_customer"`
     - reservation id と actor id を持つ metadata
   - reservation id 由来の idempotency key を付ける

4. **即時のローカルパッチを書き込む**
   - Stripe refund payload を reservation patch に変換する
   - refund id、refund status、refund amount、failure reason、timestamp を保存する
   - refund が active なら reservation を `cancelled` に移す
   - refund が即失敗 / 即キャンセルなら、キャンセル成功を装わない

5. **最終確定は webhook に任せる**
   - `refund.created`, `refund.updated`, `refund.failed` が row を同期し続けること
   - customer portal は reservation status と別に refund status を表示すること

6. **キャンセル可能判定を component の外に置く**
   - UI は API を呼ぶ
   - eligibility logic は shared helper に置く
   - ボタン表示と server-side enforcement の両方で再利用する

状態ルールは `refund-decision-table.md` を参照してください。

## よくある正当化

| Rationalization | Reality |
|---|---|
| 「reservation が paid なのは分かっているから charge lookup は不要」 | refund 作成には依然として charge 参照、または同等の payment object 経路が必要です。 |
| 「retry はもう一度ボタンを押せばいい」 | 金融操作に必要なのはユーザー運ではなく idempotency key です。 |
| 「refund 作成が throw しても reservation を cancelled にすればいい」 | それではお金が動いていない偽のキャンセルになります。 |
| 「refund state は後で webhook が処理するから API の即時パッチは不要」 | 即時フィードバックは重要であり、webhook は後で来るか retry が必要になることがあります。 |

## レッドフラグ

- cancel path が所有者以外や provider に refund 発火を許している
- 予定開始後でも reservation をキャンセルできる
- refund 作成に idempotency key がない
- refund path で transfer と application fee を戻していない
- UI がキャンセル後に refund state を隠している

## 検証

- [ ] customer role を持つ reservation owner だけがキャンセルできる
- [ ] future の `paid` reservation だけがキャンセル可能である
- [ ] refund 作成で `reverse_transfer` と `refund_application_fee` を使っている
- [ ] refund 作成に reservation id 由来の idempotency key を使っている
- [ ] reservation row に refund id と refund status が保存される
- [ ] refund webhook update が後続変更を整合できる
