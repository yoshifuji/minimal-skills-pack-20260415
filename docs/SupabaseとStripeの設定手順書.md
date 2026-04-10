# Supabase / Stripe 設定手順マニュアル

このドキュメントは、このリポジトリの `README.md` とは別に、Supabase と Stripe の設定を「どの画面で」「どの項目を」操作するかまで分かるように整理した運用向け手順書です。

対象リポジトリ:

- `supabase/schema.sql`
- `lib/supabase/database.types.ts`（スキーマの TypeScript 型。`schema.sql` と同期）
- `lib/supabase/client.ts`
- `lib/supabase/server.ts`
- `lib/supabase/admin.ts`
- `lib/checkout-cancel.ts`（Checkout キャンセル時の `pending` → `cancelled`）
- `app/checkout/cancel/page.tsx`
- `app/api/connect/account-link/route.ts`
- `app/api/stripe/webhook/route.ts`

最終確認日: 2026-04-05

## 1. このアプリで必要な設定の全体像

このアプリは次の前提で動きます。

- Supabase Auth のメール/パスワード認証を使う
- Supabase の `profiles`, `tutor_profiles`, `lesson_offerings`, `bookings` などを `supabase/schema.sql` で作る
- Stripe Connect の connected account 種別は Express
- Stripe Checkout で決済し、Webhook で booking を更新する（DB 更新に失敗した場合はエンドポイントが 500 を返し、Stripe がイベントを再送する）
- Checkout の `cancel_url` から戻ったとき、条件を満たせば `pending` の予約を `cancelled` に更新する
- Stripe の `account.updated` を受けると講師の Stripe 状態を同期する

このため、環境ごとに最低限必要なのは次です。

| 項目 | 開発環境 | 本番環境 |
| --- | --- | --- |
| Supabase プロジェクト | 開発用プロジェクトを1つ作成 | 本番用プロジェクトを別で作成 |
| Stripe 環境 | Sandbox / Test mode | Live mode |
| Webhook | `stripe listen` でローカル転送 | Dashboard の Webhooks で公開 URL を登録 |
| Auth メール | 開発では確認メール OFF 推奨、またはチームメンバー宛のみ | 確認メール ON + Custom SMTP 推奨 |
| アプリ URL | `http://localhost:3000` | `https://<本番ドメイン>` |

## 2. 環境変数と管理先

このアプリで使う主要な環境変数は次です。

| 環境変数 | 取得元 | 用途 |
| --- | --- | --- |
| `NEXT_PUBLIC_APP_URL` | 自分で決める | アプリのベース URL |
| `NEXT_PUBLIC_SUPABASE_URL` | Supabase Project URL | クライアント/サーバー両方の Supabase 接続先 |
| `NEXT_PUBLIC_SUPABASE_PUBLISHABLE_KEY` | Supabase の `publishable` キー | 公開側の Supabase 接続 |
| `SUPABASE_SECRET_KEY` | Supabase の `secret` キー | サーバー側の管理操作 |
| `STRIPE_SECRET_KEY` | Stripe API key | Stripe サーバー API 呼び出し |
| `STRIPE_WEBHOOK_SECRET` | Stripe Webhook Signing secret | Webhook 署名検証 |
| `STRIPE_CONNECTED_ACCOUNT_COUNTRY` | 自分で決める | Connect account 作成時の国 |
| `NEXT_PUBLIC_PLATFORM_CURRENCY` | 自分で決める | 通貨コード |
| `PLATFORM_FEE_BPS` | 自分で決める | プラットフォーム手数料率（basis points）。**0〜10000 の整数**を推奨。解釈できない値・範囲外はアプリ側で **1500** にフォールバックする |

開発環境では `.env.local` に保存します。本番環境ではホスティング先の環境変数管理画面に保存してください。`SUPABASE_SECRET_KEY` と `sk_live_...` はソースコードに置かないでください。

### 2-1. TypeScript 型と Supabase スキーマ

- PostgREST / テーブル操作の型安全のため、`lib/supabase/database.types.ts` に `Database` 型を定義しています（`supabase/schema.sql` と内容を揃えてください）。
- ローカルで [Supabase CLI](https://supabase.com/docs/guides/cli) により DB が起動している場合は、次で型を再生成できます（上書きされるので差分は Git で確認してください）。

```bash
npm run supabase:types
```

- CI やローカル検証では `npm run typecheck`（および `npm run build` の型チェック）が通る状態を維持してください。
- `@supabase/ssr` と `@supabase/supabase-js` は **peer 互換のバージョン**が必要です。バージョンだけ片方を古いままにすると、ブラウザ側の `.from()` 推論が壊れることがあります（`package.json` の範囲に合わせて `npm install`）。

## 3. 開発環境の設定手順

### 3-1. Supabase 開発環境

#### Step 1. 開発用 Supabase プロジェクトを作成する

画面:

- `https://supabase.com/dashboard`
- または `https://database.new`

操作:

1. `New project` を押す
2. `Organization` を選ぶ
3. `Project name` に開発用と分かる名前を入れる
4. `Database Password` を設定する
5. `Region` を選ぶ
6. `Create new project` を押す

推奨:

- 本番とは別の開発用プロジェクト名にする
- 例: `tutor-marketplace-dev`

#### Step 2. API URL と API Keys を取得する

画面:

- Project Dashboard
- `Settings` -> `API`
- または `Settings` -> `API Keys`
- または Project の `Connect` ダイアログ

操作:

1. `Project URL` をコピーして `NEXT_PUBLIC_SUPABASE_URL` に入れる
2. `API Keys` で `publishable` をコピーして `NEXT_PUBLIC_SUPABASE_PUBLISHABLE_KEY` に入れる
3. 同じく `secret` をコピーして `SUPABASE_SECRET_KEY` に入れる

補足:

- このリポジトリは `publishable/secret key` の変数名を前提にしています。`Legacy API Keys` の `anon` / `service_role` は使いません。

#### Step 3. スキーマを投入する

画面:

- Project Dashboard
- `SQL Editor`

操作:

1. `SQL Editor` を開く
2. `New query` を押す
3. `supabase/schema.sql` の内容を貼り付ける
4. `Run` を押す

確認ポイント:

- `profiles`
- `tutor_profiles`
- `lesson_offerings`
- `bookings`
- `public_tutor_directory`

が作成されていること

#### Step 4. メール/パスワード認証を確認する

画面:

- Project Dashboard
- `Authentication` -> `Providers`

操作:

1. `Email` provider が有効なことを確認する
2. 開発環境では、ローカル確認を早くしたいならメール確認必須設定を OFF にする

このアプリとの関係:

- `components/SignupForm.tsx` の `signUp()` は `emailRedirectTo` を渡していません
- そのため、確認メールの戻り先は Supabase の `Site URL` 設定に依存します
- 確認メールを ON にした場合、登録直後は session が返らず、ユーザーはメール確認後にログインする流れです

開発でのおすすめ:

- 手早く動作確認するだけなら確認メールを OFF
- メール確認込みで試すなら ON のままにし、後述の `Site URL` を `http://localhost:3000` にする

#### Step 5. URL Configuration を設定する

画面:

- Project Dashboard
- `Authentication` -> `URL Configuration`

操作:

1. `Site URL` に `http://localhost:3000` を入れる
2. `Redirect URLs` に `http://localhost:3000/**` を追加する
3. `Save` を押す

理由:

- Supabase 公式ドキュメントでは、`redirectTo` を指定しない場合のデフォルト遷移先は `Site URL` です
- このアプリは signup 時に `emailRedirectTo` を指定していないため、開発中でも `Site URL` が重要です

#### Step 6. 開発用メール送信の制約を理解する

画面:

- 組織設定の `Team` タブ

重要:

- Supabase のデフォルト SMTP は本番用ではありません
- 開発段階ではチームメンバーのメールアドレスにしか送れません
- レート制限もあります

運用判断:

- 開発だけなら確認メール OFF で進める
- 確認メール ON で試すなら、送信先を Team メンバーのメールにする

#### Step 7. `.env.local` を設定する

ローカルファイル:

```bash
cp .env.example .env.local
```

入力例:

```env
NEXT_PUBLIC_APP_URL=http://localhost:3000

NEXT_PUBLIC_SUPABASE_URL=https://<project-ref>.supabase.co
NEXT_PUBLIC_SUPABASE_PUBLISHABLE_KEY=<supabase publishable key>
SUPABASE_SECRET_KEY=<supabase secret key>
```

#### Step 8. 動作確認

1. `npm install`
2. `npm run dev`
3. `http://localhost:3000/signup` を開く
4. 生徒または講師で登録する
5. メール確認 OFF の場合はそのままログインできることを確認する
6. メール確認 ON の場合は確認メール後にログインできることを確認する

### 3-2. Stripe 開発環境

#### Step 1. Sandbox / Test mode を使う

画面:

- Stripe Dashboard
- `API keys`

操作:

1. Stripe Dashboard にログインする
2. `API keys` を開く
3. Test mode / Sandbox のキーを使う
4. `sk_test_...` を `STRIPE_SECRET_KEY` に入れる

補足:

- 開発では `sk_test_...` を使う
- 本番課金は発生しない

#### Step 2. Connect のブランド設定を入れる

画面:

- Stripe Dashboard
- `Connect settings`

操作:

1. `Connect settings` を開く
2. Onboarding で表示される `brand name`, `color`, `icon` を設定する
3. 必要に応じて `Onboarding options` を開く
4. `STRIPE_CONNECTED_ACCOUNT_COUNTRY=JP` を使うなら、日本向け onboarding を許可する設定を確認する

このアプリとの関係:

- `lib/connect.ts` では `type: "express"` で Express connected account を作成します
- `STRIPE_CONNECTED_ACCOUNT_COUNTRY` の既定値は `JP` です

#### Step 3. Sandbox 用の API Key を `.env.local` に設定する

入力例:

```env
STRIPE_SECRET_KEY=sk_test_xxx
STRIPE_CONNECTED_ACCOUNT_COUNTRY=JP
NEXT_PUBLIC_PLATFORM_CURRENCY=jpy
# 0〜10000 の整数推奨（15% = 1500）。無効な値はアプリ側で 1500 にフォールバック
PLATFORM_FEE_BPS=1500
```

#### Step 4. ローカル Webhook を接続する

開発では Dashboard に公開 URL を作るより、Stripe CLI の転送が最短です。

CLI:

```bash
stripe login
stripe listen --forward-to localhost:3000/api/stripe/webhook
```

操作:

1. `stripe login` で CLI を認証する
2. `stripe listen --forward-to localhost:3000/api/stripe/webhook` を実行する
3. 出力された `whsec_...` を `STRIPE_WEBHOOK_SECRET` に入れる

補足:

- このアプリの Webhook 受信先は `app/api/stripe/webhook/route.ts`
- Checkout 完了イベントで `bookings.status` を更新します
- Supabase への `update` がエラーになった場合は **HTTP 500** を返します。意図的な挙動で、Stripe が配信を再試行します（ログで `bookings update` を確認してください）

#### Step 4-2. Checkout をキャンセルしたときの予約状態

- `app/api/checkout/session/route.ts` の `cancel_url` には `booking_id` がクエリとして付きます。
- ユーザーが `/checkout/cancel?booking_id=<uuid>` に遷移すると、サーバー側で `lib/checkout-cancel.ts` が次を確認します。
  - `booking_id` が有効な UUID か
  - ログイン済みで、その予約の `student_id` と一致するか
  - 予約が `pending` か（それ以外は更新せずメッセージのみ）
- **未ログイン**のままキャンセル画面に来た場合は DB を更新できません（画面に案内が出ます）。確実に `pending` を片付けたい場合は生徒でログインした状態で戻るか、`checkout.session.expired` などの Webhook に任せます。

#### Step 5. Connect webhook も開発で受けたい場合

`account.updated` は connected account 側イベントです。Connect event の動作まで開発機で確認するなら、Stripe CLI の Connect 向け転送を使います。

CLI:

```bash
stripe listen --forward-to localhost:3000/api/stripe/webhook --forward-connect-to localhost:3000/api/stripe/webhook
```

必要に応じてトリガー:

```bash
stripe trigger account.updated --stripe-account acct_xxx
```

#### Step 6. 開発時の推奨イベント

このアプリで最低限確認したいのは次です。

- `checkout.session.completed`
- `checkout.session.async_payment_succeeded`
- `checkout.session.async_payment_failed`
- `checkout.session.expired`
- `refund.created`
- `refund.updated`
- `refund.failed`
- `account.updated`

#### Step 7. 開発の確認フロー

1. 講師で登録する
2. `/dashboard/tutor` で `Connect登録を開始する` を押す
3. Stripe hosted onboarding を完了する
4. `Stripe状態を同期` を押す
5. レッスンを作る
6. 別ユーザーで予約して Checkout へ進む
7. テストカード `4242 4242 4242 4242` で決済する
8. Webhook が到達し `bookings.status = paid` になることを確認する
9. （任意）Checkout で支払いを中断し、`/checkout/cancel?booking_id=...` に戻ったとき、生徒でログインしていれば該当の `pending` が `cancelled` になることを確認する

## 4. 本番環境の設定手順

### 4-1. Supabase 本番環境

#### Step 1. 本番用 Supabase プロジェクトを別で作成する

画面:

- `https://supabase.com/dashboard`

操作:

1. 本番用に新しいプロジェクトを作る
2. 開発と同じプロジェクトを使い回さない

推奨:

- `tutor-marketplace-prod`
- 開発と本番で project ref を明確に分ける

理由:

- Supabase 公式は staging と production を分ける構成を案内しています

#### Step 2. スキーマを本番へ投入する

最小手順の画面操作:

- `SQL Editor` -> `New query` -> `supabase/schema.sql` を実行

推奨運用:

- 本番では手元から都度 SQL を貼るより、migration + CI/CD で反映する

#### Step 3. 本番用 API URL / Keys を取得する

画面:

- `Settings` -> `API`
- `Settings` -> `API Keys`

操作:

1. 本番プロジェクトの `Project URL` を `NEXT_PUBLIC_SUPABASE_URL` に設定
2. 本番プロジェクトの `publishable` を `NEXT_PUBLIC_SUPABASE_PUBLISHABLE_KEY` に設定
3. 本番プロジェクトの `secret` を `SUPABASE_SECRET_KEY` に設定

注意:

- 本番の `SUPABASE_SECRET_KEY` はローカル `.env` ではなく、デプロイ先の Secret 管理に保存する

#### Step 4. URL Configuration を本番ドメインに切り替える

画面:

- `Authentication` -> `URL Configuration`

操作:

1. `Site URL` を `https://<本番ドメイン>` に設定
2. `Redirect URLs` に本番 URL を追加する
3. Preview 環境がある場合は、その URL も追加する

推奨設定例:

- `https://example.com/**`
- `https://www.example.com/**`
- Preview を使うなら `https://*-<team>.vercel.app/**` など

注意:

- 本番では wildcard を広げすぎず、可能なら正確な URL を入れる

#### Step 5. 認証設定を本番向けに調整する

画面:

- `Authentication` -> `Providers`

操作:

1. `Email` provider が有効なことを確認する
2. 本番では確認メール必須設定を ON にする

理由:

- 登録メールの所有確認を省かないため

#### Step 6. Custom SMTP を設定する

画面:

- `Authentication` の設定画面
- `Notifications` または SMTP 設定セクション

操作:

1. SMTP を提供するメールサービスを選定する
2. `Sender email`
3. `Sender name`
4. `Host`
5. `Port`
6. `Username`
7. `Password`

を入力して保存する

推奨:

- 認証メール専用の送信ドメインを用意する
- 送信元は `no-reply@auth.<your-domain>` などに分ける

理由:

- Supabase のデフォルト SMTP は本番向けではありません
- Team メンバー以外に送れず、レート制限もあります

#### Step 7. Email Templates を必要に応じて編集する

画面:

- `Authentication` -> `Email Templates`

必要なケース:

- ブランド名を出したい
- 文面を日本語化したい
- `{{ .SiteURL }}` 前提のリンクを本番ドメインに合わせたい

このアプリとの関係:

- signup 時に `emailRedirectTo` を渡していないため、テンプレートを過度にカスタムしない限り `Site URL` が正しく設定されていれば動作します

### 4-2. Stripe 本番環境

#### Step 1. Live mode の準備をする

画面:

- Stripe Dashboard
- `API keys`

操作:

1. Live mode に切り替える
2. `pk_live_...` と `sk_live_...` を確認する
3. このアプリでは `sk_live_...` を `STRIPE_SECRET_KEY` に設定する

注意:

- Live mode secret key は保存場所を決めてから取り扱う
- 本番鍵は表示回数に制約があるため、Secret manager に即保存する

#### Step 2. Connect settings を本番向けに仕上げる

画面:

- Stripe Dashboard
- `Connect settings`
- `Onboarding options`

操作:

1. Onboarding 画面の `brand name`, `color`, `icon` を本番ブランドにする
2. `Onboarding options` で利用国を確認する
3. `JP` で運用するなら日本向け設定を確認する
4. 銀行口座の収集を onboarding 中に行う方針か確認する

このアプリとの関係:

- `/api/connect/account-link` で Stripe hosted onboarding に遷移します
- `refresh_url` は `/api/connect/account-link/refresh`
- `return_url` は `/dashboard/tutor`

#### Step 3. 本番 Webhook endpoint を作成する

画面:

- Stripe Dashboard
- `Workbench` -> `Webhooks`

このアプリでは webhook が2系統あると理解すると管理しやすいです。

1. Platform account 側イベント
2. Connected accounts 側イベント

##### A. Platform account 側 webhook

用途:

- `checkout.session.completed`
- `checkout.session.async_payment_succeeded`
- `checkout.session.async_payment_failed`
- `checkout.session.expired`

操作:

1. `Workbench` -> `Webhooks`
2. `Create new destination`
3. `Listen to` で `Events on your account` を選ぶ
4. イベントを上記4つ選ぶ
5. `Webhook` を選ぶ
6. `Endpoint URL` に `https://<本番ドメイン>/api/stripe/webhook` を入れる
7. 作成後に endpoint を開く
8. `Signing secret` を開く
9. `STRIPE_WEBHOOK_SECRET` に設定する

##### B. Connected accounts 側 webhook

用途:

- `account.updated`

操作:

1. `Workbench` -> `Webhooks`
2. `Create new destination`
3. `Listen to` で `Events on connected accounts` を選ぶ
4. イベントに `account.updated` を含める
5. `Webhook` を選ぶ
6. `Endpoint URL` に `https://<本番ドメイン>/api/stripe/webhook` を入れる
7. 作成後に endpoint を開く
8. `Signing secret` を開く

重要な判断:

- 同じ URL に 2 つの event destination を向ける運用は可能です
- ただし signing secret は endpoint ごとに別です
- 1本の環境変数 `STRIPE_WEBHOOK_SECRET` しか使っていない現状コードでは、公開 URL を本番で 1 endpoint に集約するか、コードを複数 secret 対応に拡張するかを決めてください

実務上のおすすめ:

- まずは本番では `Events on your account` を必須で設定する
- `account.updated` を自動同期したい場合は、Stripe 側を 1 endpoint 設計に揃えるか、アプリ側を複数 secret 対応へ変更する

#### Step 4. 本番環境変数をデプロイ先へ設定する

最低限:

```env
NEXT_PUBLIC_APP_URL=https://<本番ドメイン>

NEXT_PUBLIC_SUPABASE_URL=https://<prod-project-ref>.supabase.co
NEXT_PUBLIC_SUPABASE_PUBLISHABLE_KEY=<prod publishable key>
SUPABASE_SECRET_KEY=<prod secret key>

STRIPE_SECRET_KEY=sk_live_xxx
STRIPE_WEBHOOK_SECRET=whsec_xxx
STRIPE_CONNECTED_ACCOUNT_COUNTRY=JP
NEXT_PUBLIC_PLATFORM_CURRENCY=jpy
# 0〜10000 の整数推奨。無効な値は 1500 にフォールバック
PLATFORM_FEE_BPS=1500
```

#### Step 5. 本番のリリース前確認

Supabase:

- 本番 Project URL / API keys を入れたか
- `Site URL` が本番ドメインか
- Email provider が有効か
- 確認メール設定が本番方針に合っているか
- Custom SMTP が実送信できるか

Stripe:

- `sk_live_...` を設定したか
- Connect settings のブランド表示が整っているか
- Webhook endpoint が `Delivered` になるか
- Connect onboarding が完了した講師で `charges_enabled` と `payouts_enabled` が true になるか

アプリ:

- 生徒登録
- 講師登録
- 講師 onboarding
- レッスン作成
- Checkout 決済
- booking が `paid` に更新
- `npm run typecheck` / `npm run build` が通ること
- （任意）Checkout キャンセル戻りで `pending` → `cancelled` が期待どおりか

## 5. このアプリ固有の注意点

### Supabase 側

- このリポジトリは `SUPABASE_SECRET_KEY` をサーバー側で使います
- `SUPABASE_SECRET_KEY` は RLS をバイパスできるため、絶対にクライアントへ露出させないでください
- signup 時に `emailRedirectTo` を渡していないので、メール確認の遷移先は `Site URL` 依存です
- テーブル定義を変えたら `supabase/schema.sql` と `lib/supabase/database.types.ts` の両方を更新するか、`npm run supabase:types` で型を再生成してください

### Stripe 側

- このリポジトリは Express connected account を作成します
- `STRIPE_CONNECTED_ACCOUNT_COUNTRY` の設定と Stripe Dashboard の Onboarding options が食い違うと onboarding 方針が混乱します
- `account.updated` は Connect event です。通常の account event webhook だけでは拾えません
- Webhook signing secret は API key とは別物です
- Checkout 関連イベントで DB 更新に失敗した場合、Webhook ハンドラは **500** を返します（再送を利用して整合性を取る想定です）

## 6. 公式ドキュメント

Supabase:

- API keys: https://supabase.com/docs/guides/api/api-keys
- Password-based Auth: https://supabase.com/docs/guides/auth/passwords
- Redirect URLs: https://supabase.com/docs/guides/auth/redirect-urls
- Custom SMTP: https://supabase.com/docs/guides/auth/auth-smtp
- Managing environments: https://supabase.com/docs/guides/deployment/managing-environments

Stripe:

- API keys: https://docs.stripe.com/keys
- Event destinations / Webhooks: https://docs.stripe.com/workbench/event-destinations
- Connect webhooks: https://docs.stripe.com/connect/webhooks
- Stripe-hosted onboarding: https://docs.stripe.com/connect/hosted-onboarding
- Express connected accounts: https://docs.stripe.com/connect/express-accounts

## 7. 参考メモ

このドキュメントで画面名を特定する際に確認した公式情報の要点:

- Supabase は `Settings -> API Keys` または Project の `Connect` ダイアログから鍵を取得できる
- Supabase の `Site URL` は、`redirectTo` 未指定時の既定遷移先になる
- Supabase のデフォルト SMTP は本番向けではなく、チームメンバー宛に限定される
- Supabase は staging と production を別プロジェクトに分ける構成を推奨している
- Stripe は `API keys` 画面で test/live key を切り替える
- Stripe Webhook signing secret は API key とは別
- Stripe Connect event を受けるには `Listen to -> Events on connected accounts` を選ぶ必要がある
- Stripe hosted onboarding の見た目は `Connect settings` で調整する
