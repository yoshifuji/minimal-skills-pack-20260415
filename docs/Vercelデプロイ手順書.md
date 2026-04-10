# Vercel デプロイ手順書

このプロジェクトは Next.js App Router なので、Vercel へそのままデプロイできます。  
ただし、**Supabase の URL 設定**と**Stripe Webhook の signing secret**は、本番 URL が確定してから入れる手順が必要です。

## 事前確認

- リポジトリが GitHub / GitLab / Bitbucket のいずれかに push 済みである
- `npm install`
- `npm run typecheck`
- `npm run build`

ローカルでここまで通ってから Vercel に上げてください。

## Vercel にアップロードする前に決めること

- 本番 URL を `https://<your-domain>` にするか、Vercel の `https://<project>.vercel.app` を使うか
- Supabase を本番用プロジェクトに切り替えるか
- Stripe を test mode のまま使うか、本番 live mode に切り替えるか

本番運用するなら、Supabase と Stripe は開発用と分けてください。

## 1. Vercel にプロジェクトを作成する

1. Vercel にログインする
2. `Add New` → `Project`
3. 対象リポジトリを選ぶ
4. Framework Preset が `Next.js` になっていることを確認する
5. Root Directory はこのリポジトリのルートのままにする
6. Build Command は既定値の `next build` のままでよい
7. Output Directory は空欄のままでよい

このプロジェクトに `vercel.json` は必須ではありません。

## 2. Vercel の環境変数を登録する

Vercel Project Settings → `Environment Variables` に次を登録します。

### 必須

```env
NEXT_PUBLIC_APP_URL=https://<本番ドメイン>

NEXT_PUBLIC_SUPABASE_URL=https://<your-project-ref>.supabase.co
NEXT_PUBLIC_SUPABASE_PUBLISHABLE_KEY=...
SUPABASE_SECRET_KEY=...

STRIPE_SECRET_KEY=sk_test_... または sk_live_...

STRIPE_CONNECTED_ACCOUNT_COUNTRY=JP
NEXT_PUBLIC_PLATFORM_CURRENCY=jpy
PLATFORM_FEE_BPS=1500
```

### 初回デプロイ時の注意

`STRIPE_WEBHOOK_SECRET` は、**本番の webhook endpoint を Stripe 側で作成したあと**に値が分かります。  
そのため、初回デプロイ時は `STRIPE_WEBHOOK_SECRET` をまだ入れなくて構いません。

補足:

- `NEXT_PUBLIC_APP_URL` は Checkout の success / cancel URL、Connect onboarding の return URL に使われます
- `SUPABASE_SECRET_KEY` と `STRIPE_SECRET_KEY` は server-only の値です
- Preview 環境でも本番 URL に戻したいなら、`NEXT_PUBLIC_APP_URL` は production と同じ値にします

## 3. 初回デプロイを実行する

1. `Deploy` を押す
2. デプロイ完了後、発行された URL を開く
3. `/login` や `/signup` が表示できることを確認する

この時点では、Stripe Webhook 未設定でもアプリ表示は問題ありません。

## 4. Supabase の認証 URL を本番ドメインに合わせる

Supabase Dashboard で次を更新します。

1. `Authentication` → `URL Configuration`
2. `Site URL` に `https://<本番ドメイン>` を設定
3. `Redirect URLs` に次を追加

```txt
https://<本番ドメイン>/**
```

Preview も使う場合は、必要な範囲だけ追加してください。

```txt
https://*-<team-or-scope>.vercel.app/**
```

補足:

- wildcard を広げすぎると不要な URL も通るので、可能なら production は固定 URL のみにしてください
- 本番カスタムドメインを使うなら、Vercel 側でドメイン追加後に Supabase 側も更新してください

## 5. Stripe の本番 Webhook endpoint を作成する

Stripe Dashboard で endpoint を作成します。

1. `Developers` → `Webhooks` または `Event destinations`
2. `Add endpoint`
3. Endpoint URL に次を設定

```txt
https://<本番ドメイン>/api/stripe/webhook
```

4. 少なくとも次のイベントを購読する

```txt
checkout.session.completed
checkout.session.async_payment_succeeded
checkout.session.async_payment_failed
checkout.session.expired
refund.created
refund.updated
refund.failed
```

5. 必要なら `Connected accounts` 側でも `account.updated` を購読する
6. 表示された signing secret `whsec_...` を控える

## 6. `STRIPE_WEBHOOK_SECRET` を Vercel に追加する

1. Vercel Project Settings → `Environment Variables`
2. `STRIPE_WEBHOOK_SECRET=whsec_...` を追加
3. 再デプロイする

これで `/api/stripe/webhook` の署名検証が本番で通るようになります。

## 7. カスタムドメインを使う場合

Vercel Project Settings → `Domains` で本番ドメインを追加します。

ドメイン追加後は次も合わせて更新してください。

- Vercel の `NEXT_PUBLIC_APP_URL`
- Supabase の `Site URL`
- Supabase の `Redirect URLs`
- Stripe Webhook endpoint URL

URL を変更したら、最後に再デプロイします。

## 8. デプロイ後の動作確認

最低限、次を確認してください。

1. 講師登録ができる
2. 生徒登録ができる
3. 講師が Connect onboarding に進める
4. レッスン作成ができる
5. Checkout に遷移できる
6. 決済完了後に webhook が成功し、booking が `paid` になる
7. キャンセル時に `/checkout/cancel` が動く

## Preview 環境の扱い

このコードは `STRIPE_WEBHOOK_SECRET` を 1 つだけ使っています。  
そのため、**Preview ごとに別 webhook endpoint を量産する運用には向いていません**。

安全に運用するなら次のどちらかに寄せてください。

- Preview は UI / 認証確認までに留めて、決済系は production で確認する
- 将来的にコードを拡張して、環境ごとに複数 webhook secret を扱えるようにする

## よくある詰まりどころ

### Connect から戻っても講師が有効化されない

- `NEXT_PUBLIC_APP_URL` が本番 URL とズレていないか確認する
- Stripe onboarding 完了後に講師ダッシュボードで状態同期を実行する
- `account.updated` を webhook で受けているか確認する

### 決済後に booking が `paid` にならない

- Vercel の `STRIPE_WEBHOOK_SECRET` が正しいか確認する
- Stripe Dashboard の webhook 配信履歴で `2xx` になっているか確認する
- Endpoint URL が `https://<本番ドメイン>/api/stripe/webhook` になっているか確認する

### ログイン後の遷移やメールリンクがローカル URL のまま

- Supabase の `Site URL` が `http://localhost:3000` のまま残っていないか確認する
- Vercel の `NEXT_PUBLIC_APP_URL` が本番 URL になっているか確認する

## このリポジトリで Vercel 向けに調整した点

- `npm run typecheck` は `tsconfig.typecheck.json` を使うように変更済み
- `npm run build` は Vercel 標準の Next.js ビルドでそのまま通る構成
- Stripe webhook route は `nodejs` runtime 指定済み
