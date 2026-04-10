---
name: deployment-for-nextjs-supabase-stripe
description: Next.js + Supabase + Stripe の marketplace を dev / production 間で安全にデプロイするスキルです。tutor marketplace をローカル開発から先へ進めるとき、環境変数を接続するとき、auth URL を設定するとき、本番 webhook endpoint を作るときに使ってください。
---

# Next.js Supabase Stripe のデプロイ

## 概要

このスタックは、URL と secret が相互に結び付いた 3 つのシステムにまたがります。app host、Supabase Auth、Stripe Webhooks です。デプロイの本質は調整作業です。redirect、メールリンク、onboarding の戻り先、webhook 検証がすべて同じ環境を指すよう、正しい順番で進めてください。

## 使う場面

- 最初のホスト環境デプロイ
- preview から production への昇格
- test 用 secret から live 用 secret への切り替え
- webhook や secret 値のローテーション

次の用途には使わないでください。
- ローカルだけのデモ
- 外部認証や決済連携を持たないデプロイ

## コアプロセス

1. **まずローカルでアプリを安定させる**
   - `npm install`
   - `npm run typecheck`
   - `npm run build`
   - 最初の build failure を本番で調査しない

2. **環境変数を意図的に設定する**
   - App URL
   - Supabase URL + publishable key + secret key
   - Stripe secret key
   - Platform currency、account country、fee bps
   - Webhook secret は endpoint ができる最初の deploy 後でもよい

3. **webhook secret 展開前に一度デプロイする**
   - アプリを push する
   - `/`, `/login`, `/signup` が描画されることを確認する
   - その後で本番 Stripe webhook endpoint を作る

4. **Supabase Auth URL を揃える**
   - `Site URL` を設定する
   - `Redirect URLs` を設定する
   - production に localhost 値を残さない

5. **Stripe webhook endpoint を作る**
   - 宛先を `/api/stripe/webhook` にする
   - checkout、refund、`account.updated` イベントを購読する
   - signing secret を app env に入れる
   - 再デプロイする

6. **本番スモークテストを回す**
   - Tutor signup
   - Student signup
   - Connect onboarding
   - Offering creation
   - Checkout launch
   - Payment completion → booking `paid`
   - Cancel flow
   - Refund flow

`launch-checklist.md` とルートの環境変数リファレンスも参照してください。

## よくある正当化

| Rationalization | Reality |
|---|---|
| 「webhook secret は後で入れればいいし、最初の deploy 確認は省ける」 | 本番で最終的な署名 secret を検証するには、先に endpoint URL が存在している必要があります。 |
| 「Supabase URL 設定は deploy と無関係」 | Site URL が localhost のままだと、メールリンクと auth redirect が壊れます。 |
| 「preview 環境は 1 つの production webhook 設定を共有し続ければいい」 | 単一 secret の webhook 運用は通常 preview 向きではありません。制約を文書化してください。 |
| 「ローカルで build が通るならローンチ時のスモークテストは不要」 | hosted 環境では secret や callback URL の違いで外部連携が壊れ得ます。 |

## レッドフラグ

- production app URL が success / cancel / onboarding の return URL と一致しない
- Supabase `Site URL` がまだ localhost を指している
- Stripe webhook endpoint はあるが、app env に signing secret がない
- deploy checklist から tutor / student の end-to-end flow が抜けている
- 明示的な戦略なしに、preview 環境で完全な payment verification ができる前提になっている

## 検証

- [ ] deploy 前に `typecheck` と `build` が通っている
- [ ] production 用 env vars が正しく設定され、適切にスコープされている
- [ ] Supabase `Site URL` と `Redirect URLs` がデプロイ先ドメインと一致している
- [ ] Stripe webhook endpoint が production URL に向いている
- [ ] endpoint 作成と再 deploy の後に `STRIPE_WEBHOOK_SECRET` が導入されている
- [ ] tutor onboarding、checkout、paid sync、cancel、refund の flow をスモークテストしている
