# Service Marketplace Skills Pack

provider/customer 型の **サービス予約マーケットプレイス** を実装するための agent-skills パックです。  
core workflow、stack skill、generic marketplace pattern、legacy tutor example overlay をまとめています。

主な前提スタック:

- Next.js App Router
- TypeScript
- Supabase Auth + Postgres + RLS
- Stripe Checkout
- Stripe Connect Express
- destination charges

## 構成

### Core workflow

- `spec-driven-development`
- `planning-and-task-breakdown`
- `incremental-implementation`
- `api-and-interface-design`
- `frontend-ui-engineering`
- `debugging-and-error-recovery`
- `security-and-hardening`
- `shipping-and-launch`

### Stack

- `nextjs-supabase-ssr-auth`
- `postgres-rls-and-public-read-models`
- `stripe-connect-express-onboarding`
- `deployment-for-nextjs-supabase-stripe`

### Generic marketplace pattern

- `marketplace-foundation-workflow`
- `role-based-portals-and-navigation`
- `stripe-single-seller-checkout`
- `payment-webhook-reconciliation`
- `refunds-and-cancellation-workflows`

### Legacy compatibility wrappers

- `using-marketplace-agent-skills`
- `role-based-marketplace-pages`
- `checkout-booking-and-destination-charges`
- `refunds-and-booking-cancellation`

### Example overlay

- `example-tutor-marketplace`
- `はじめに.md`
- `docs/講師マーケットプレイス実行手順書.md`
- `references/examples/tutor-marketplace/`

## 入口

### 新しい service marketplace を作る

1. `specs/講師マーケットプレイス再現仕様プロンプト.md` を読む  
   注: ファイル名は互換性のため維持していますが、中身は generic なサービス予約 marketplace の example overlay です。
2. `skills/marketplace-foundation-workflow/SKILL.md` を読む
3. `spec-driven-development` で `SPEC.md` を作る
4. `planning-and-task-breakdown` で `tasks/plan.md` と `tasks/todo.md` を作る
5. generic skill の順で schema → auth → portals → connect → checkout → webhook → refund → deploy と進める

### 旧 tutor marketplace 例をたどる

1. `はじめに.md` を読む
2. `skills/example-tutor-marketplace/SKILL.md` を読む
3. `docs/講師マーケットプレイス実行手順書.md` を読む
4. `references/examples/tutor-marketplace/` を参照しながら進める

## References

### 共通 reference

- `references/Stripeイベントマトリクス.md`
- `references/環境変数とシークレット一覧.md`
- `references/セキュリティチェックリスト.md`
- `references/アクセシビリティチェックリスト.md`
- `references/パフォーマンスチェックリスト.md`

### Tutor example reference

- `references/examples/tutor-marketplace/データモデル概要.md`
- `references/examples/tutor-marketplace/デザインブリーフ.md`
- `references/examples/tutor-marketplace/手動QAフロー.md`
- `references/examples/tutor-marketplace/実行成果物チェックリスト.md`

旧 path の tutor reference は compatibility redirect として残しています。

## 期待する成果物

- `SPEC.md`
- `tasks/plan.md`
- `tasks/todo.md`
- `app/` 配下の Next.js 実装
- `components/` 配下の UI 実装
- `lib/` 配下の auth / Stripe / data helpers
- `supabase/schema.sql`
- `.env.example`
- deploy / setup docs
