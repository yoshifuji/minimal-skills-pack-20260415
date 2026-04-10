# ローンチ・チェックリスト

## デプロイ前
- [ ] schema を適用済み
- [ ] DB types が最新
- [ ] `npm run typecheck`
- [ ] `npm run build`

## 環境変数
- [ ] `NEXT_PUBLIC_APP_URL`
- [ ] `NEXT_PUBLIC_SUPABASE_URL`
- [ ] `NEXT_PUBLIC_SUPABASE_PUBLISHABLE_KEY`
- [ ] `SUPABASE_SECRET_KEY`
- [ ] `STRIPE_SECRET_KEY`
- [ ] `STRIPE_CONNECTED_ACCOUNT_COUNTRY`
- [ ] `NEXT_PUBLIC_PLATFORM_CURRENCY`
- [ ] `PLATFORM_FEE_BPS`

## 初回デプロイ後
- [ ] Supabase Site URL を更新済み
- [ ] Supabase Redirect URLs を更新済み
- [ ] Stripe webhook endpoint を作成済み
- [ ] `STRIPE_WEBHOOK_SECRET` を追加済み
- [ ] アプリを再デプロイ済み

## スモークテスト
- [ ] tutor signup
- [ ] student signup
- [ ] offering creation
- [ ] connect onboarding
- [ ] checkout
- [ ] webhook paid update
- [ ] cancel URL recovery
- [ ] refund initiation
- [ ] refund webhook update
