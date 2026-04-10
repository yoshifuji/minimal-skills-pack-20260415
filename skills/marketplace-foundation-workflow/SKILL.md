---
name: marketplace-foundation-workflow
description: 二面サービス marketplace の基盤 workflow を整理する meta skill です。provider/customer role、公開 listing、Stripe Connect、Stripe Checkout、webhook、refund を持つ marketplace を実装するとき、次にどの skill を使うか決めたいとき、または実装順を固定したいときに使ってください。
---

# Marketplace Foundation Workflow

## 概要

この skill は、service marketplace を **再利用可能な順序** で組み立てるための入口です。対象は、provider が listing を公開し、customer が日時を選んで予約や注文を行い、Stripe Connect と Checkout で money movement を処理する構成です。

新しい project では、まずこの skill を基準にし、必要なら stack skill や pattern skill を追加してください。

## 対象の前提

- Next.js App Router
- TypeScript
- Supabase Auth + Postgres + RLS
- Stripe Checkout
- Stripe Connect Express
- 単一 provider 単位で決済する flow

この前提から大きく外れる場合は、汎用 core skill を優先し、この skill の順序をそのまま当てはめないでください。

## Skill マップ

```text
Task arrives
   │
   ├─ Need product / route / API spec first?
   │     → spec-driven-development
   │
   ├─ Need ordered implementation tasks?
   │     → planning-and-task-breakdown
   │
   ├─ Need auth, SSR session, role redirects?
   │     → nextjs-supabase-ssr-auth
   │
   ├─ Need schema, triggers, indexes, RLS?
   │     → postgres-rls-and-public-read-models
   │
   ├─ Need public discovery or role-based portals?
   │     → role-based-portals-and-navigation
   │       + frontend-ui-engineering
   │
   ├─ Need provider onboarding / payout readiness?
   │     → stripe-connect-express-onboarding
   │
   ├─ Need reservation creation + Checkout + destination charges?
   │     → stripe-single-seller-checkout
   │
   ├─ Need webhook durability / retries / redirect recovery?
   │     → payment-webhook-reconciliation
   │       + debugging-and-error-recovery
   │
   ├─ Need cancellation / refund handling?
   │     → refunds-and-cancellation-workflows
   │       + security-and-hardening
   │
   └─ Need deploy / env / launch coordination?
         → deployment-for-nextjs-supabase-stripe
           + shipping-and-launch
```

## 推奨実装順

1. spec を固定し、`SPEC.md` を作る
2. `tasks/plan.md` と `tasks/todo.md` を作る
3. schema / trigger / RLS を固める
4. auth / signup / login / middleware / role redirect を作る
5. public discovery と role-based portal を作る
6. Connect onboarding と account state sync を作る
7. reservation 作成と Checkout を作る
8. success / cancel / webhook の三系統を整合させる
9. cancellation / refund workflow を作る
10. security hardening を確認する
11. deploy / launch の整合を取る

## 絶対に外せないルール

1. **schema と auth を payment より先に作る**
2. **pending reservation を先に保存してから Checkout Session を作る**
3. **webhook を durable source of truth として扱う**
4. **refund は UI state ではなく money movement として扱う**
5. **secret key / service role を client component へ漏らさない**
6. **`/api/stripe/webhook` は raw body 署名検証のため middleware 対象外にする**

## 検証

- [ ] `SPEC.md` がある
- [ ] `tasks/plan.md` と `tasks/todo.md` がある
- [ ] 実装順が `schema → auth → pages → connect → checkout → webhook → refund → deploy` になっている
- [ ] `npm run typecheck` と `npm run build` が通る
- [ ] project 固有の manual QA と deliverable checklist がある
