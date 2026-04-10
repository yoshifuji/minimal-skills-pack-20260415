---
name: nextjs-supabase-ssr-auth
description: Next.js App Router で、SSR cookie と role-aware routing を使った Supabase Auth を実装するスキルです。signup / login / logout、保護ページ、現在ユーザーが必要な route handler、または権限付き server-only admin path を構築するときに使ってください。
---

# Next.js + Supabase SSR Auth

## 概要

browser、server component、route handler、権限付き admin path をまたいで、一貫した auth system を構築します。狙いは予測可能な role-aware behavior です。signup で耐久的な profile row を作り、login は role に応じて redirect し、保護ページはサーバー側で access を強制し、webhook path は cookie middleware から分離します。

## 使う場面

- App Router で Supabase の email / password auth を使うとき
- 現在 session が必要な SSR ページ
- role-based redirect と dashboard access control
- secret-key client が必要な server-only flow

次の用途には使わないでください。
- SSR のない純粋な SPA auth
- OAuth 専用フロー
- raw request body が必要な webhook handler

## コアプロセス

1. **client の種類を意図的に分ける**
   - browser client は client component と form submission 用
   - server client は SSR session 読み取り用
   - admin client は権限付き join、webhook、state sync 用
   - secret key は厳密に server-side に閉じ込める

2. **middleware を通して cookie を維持する**
   - `@supabase/ssr` middleware を使って session state を更新する
   - `/api/stripe/webhook` は除外する
   - static asset と image も除外する
   - 無関係な route を変化させない

3. **database で role を bootstrap する**
   - signup 時に `full_name` と `role` を user metadata として送る
   - `auth.users` に DB trigger を追加する
   - すべての user に `profiles` row を作る
   - role が `tutor` のときは `tutor_profiles` row も自動作成する

4. **role lookup を集約する**
   - `getCurrentUserAndProfile()` を追加する
   - dashboard page 用に `requireRole()` を追加する
   - 未認証ユーザーは `/login` に redirect する
   - 認証済みユーザーが `/login` や `/signup` に来たら正しい dashboard へ redirect する

5. **form を薄く保つ**
   - client signup form は Supabase `signUp()` を呼ぶ
   - client login form は `signInWithPassword()` を呼ぶ
   - logout は browser client の `signOut()` を使う
   - auth 変更後は `router.refresh()` で新しい session で server component を再描画する

6. **admin path を標準ルートにしない**
   - SSR + RLS で読めるものは server client を使う
   - admin client は権限付き write、集約 join、webhook、または意図的に RLS を bypass する role check にだけ使う

## 実装パターン

```text
Client form
  -> browser Supabase auth call
  -> session cookie set
  -> middleware keeps session fresh
  -> server component / route handler reads user
  -> admin client used only where privilege is intentional
```

推奨ファイル分割は `flow.md` を参照してください。

## よくある正当化

| Rationalization | Reality |
|---|---|
| 「admin client を全部で使えばいい」 | それでは auth 境界が消え、意図しない権限漏れが起きやすくなります。 |
| 「RLS だけで十分で、role helper は不要」 | page と API には明示的な redirect / deny の振る舞いが必要です。 |
| 「webhook も同じ middleware path でいい」 | 署名検証は raw body に依存するため、middleware が干渉してはいけません。 |
| 「client-side の redirect check だけで十分」 | 保護された dashboard は server-side でも role を強制しなければなりません。 |

## レッドフラグ

- client code に `SUPABASE_SECRET_KEY` が出てくる
- login / signup page が認証済みユーザーを追い出さない
- dashboard access が client-side guard にだけ依存している
- route handler が browser client から current user を取得している
- middleware が Stripe webhook route で動いている

## 検証

- [ ] signup で `profiles` row が作られ、tutor signup では `tutor_profiles` row も作られる
- [ ] `/login` と `/signup` が認証済みユーザーを role ごとに redirect する
- [ ] `/dashboard/tutor` が server-side で tutor role を必須にしている
- [ ] `/dashboard/student` が server-side で student role を必須にしている
- [ ] browser logout で session が消え、navigation が更新される
- [ ] `/api/stripe/webhook` が auth middleware から除外されている
