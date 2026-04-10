---
name: stripe-connect-express-onboarding
description: Stripe Connect Express を使った seller onboarding を実装し、account readiness を application database に同期するスキルです。各 tutor / seller が connected account を作成し、onboarding を完了し、bookable / not-bookable state を app 上に出す必要があるときに使ってください。
---

# Stripe Connect Express Onboarding

## 概要

connected account onboarding は単発の API call ではなく、長く続く state machine として扱います。app は account を作成または再利用し、seller を Stripe onboarding に送り、途中離脱から復旧し、readiness を marketplace database へ同期し直さなければなりません。

## 使う場面

- 複数 seller の marketplace payout
- Stripe Connect Express を使った tutor / seller onboarding
- connected account 作成、onboarding link 生成、status sync
- Express Dashboard の login link

次の用途には使わないでください。
- 単一 merchant の platform
- seller account を持たない direct charge
- 運用モデルが大きく異なる Standard / Custom Connect flow

## コアプロセス

1. **seller profile row の存在を保証する**
   - Stripe call の前に tutor / seller row が存在することを確認する
   - 既存の `stripe_account_id` があれば再利用する

2. **connected account は一度だけ作る**
   - `type: "express"`
   - `country` は env から取る
   - product が明示的に company account を要求しない限り `business_type: "individual"`
   - `card_payments` と `transfers` を要求する
   - account id を seller row に保存する

3. **onboarding link を発行する**
   - `accountLinks.create()`
   - `type: "account_onboarding"`
   - `refresh_url` は link を再生成できる app route に戻す
   - `return_url` は tutor を dashboard に戻す

4. **readiness を同期する**
   - Stripe account を取得する
   - `details_submitted`, `charges_enabled`, `payouts_enabled`, `last_stripe_sync_at` を保存する
   - manual sync と webhook ベースの `account.updated` sync の両方を支える

5. **seller control plane を表に出す**
   - onboarding を開始 / 再開する button
   - status を同期する button
   - account 作成後に Express Dashboard login link を開く button

6. **readiness を marketplace gate として使う**
   - `charges_enabled` と `payouts_enabled` が両方 true になるまで booking を止める
   - seller dashboard は何が不足しているかを説明しなければならない

状態チェックリストは `account-state-checklist.md` を参照してください。

## よくある正当化

| Rationalization | Reality |
|---|---|
| 「ユーザーが onboarding を押すたび新しい account を作ればいい」 | それでは孤立 account と不整合な payout state が生まれます。保存済み account を再利用してください。 |
| 「`details_submitted` だけ見れば支払い開始していい」 | bookable gate に使うべきなのは `charges_enabled` と `payouts_enabled` です。 |
| 「どうせ webhook が後で同期するから manual button は不要」 | manual sync はローカル開発でも、webhook 設定が遅れているときでも有用です。 |
| 「dashboard login link は任意」 | seller には初回 onboarding 後に Stripe へ戻る導線が必要です。 |

## レッドフラグ

- 同じ tutor に対して複数の connected account が作られている
- booking 可否を誤った Stripe flag から導いている
- 期限切れ onboarding link 用の refresh route がない
- webhook 設定がないと seller dashboard が復旧できない
- account id は作っているのに DB に保存していない

## 検証

- [ ] tutor は最大 1 つの connected account しか作らない
- [ ] onboarding link が app domain 内の refresh / return URL を使っている
- [ ] manual sync で readiness field が DB に更新される
- [ ] `account.updated` webhook でも readiness を同期できる
- [ ] 既存 account に対して Express Dashboard login link が動作する
- [ ] booking gate が `charges_enabled && payouts_enabled` に依存している
