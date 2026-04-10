---
name: postgres-rls-and-public-read-models
description: RLS、trigger、明示的な public read model を備えた Supabase / Postgres schema を設計するスキルです。marketplace table、ownership policy、auth bootstrap trigger を作るときや、public page で private data の一部だけを公開すべきときに使ってください。
---

# Postgres RLS と Public Read Model

## 概要

private な運用データは private のまま保ちつつ、public page はシンプルに query できるよう marketplace をモデリングします。1 つの table に役割を詰め込んだり、匿名ユーザーに権限付き view を公開したりせず、trigger と明示的な read model を使ってください。

## 使う場面

- `profiles`, `tutor_profiles`, `lesson_offerings`, `bookings` を定義するとき
- Supabase RLS で ownership / read policy を足すとき
- tutor data の一部を匿名ユーザーへ公開するとき
- TypeScript の DB types を SQL と同期させるとき

次の用途には使わないでください。
- 一時的な mock schema
- ownership 境界のない backend
- base table 全体を安全に公開できる public view

## コアプロセス

1. **write model を定義する**
   - `profiles` はすべての auth user 用
   - `tutor_profiles` は tutor 専用の payment / public profile state 用
   - `lesson_offerings` は tutor の inventory 用
   - `bookings` は予約 + payment lifecycle 用
   - status field には明示的な check 制約を付ける

2. **lifecycle の配管を足す**
   - `set_updated_at()` trigger function
   - `auth.users` に対する `handle_new_user()` trigger
   - table ごとの updated-at trigger
   - `on delete cascade` または `restrict` は意図して使い、偶然に任せない

3. **public read の責務を分離する**
   - public field だけを持つ table として `public_tutor_directory` を作る
   - `profiles` と `tutor_profiles` から trigger で同期する
   - 匿名読み取りを権限付き view に依存させない

4. **ownership ルールから外側へ RLS を書く**
   - Profiles: 自分の row の read / update
   - Tutor profiles: 自分の row の read / insert / update
   - Offerings: public は active なものを read、tutor は自分の row を管理
   - Bookings: student は自分の row に write、tutor は seller である row を read
   - すべての table で明示的に RLS を有効化する

5. **lifecycle 操作用の lookup index を追加する**
   - seller / student の foreign key
   - `stripe_checkout_session_id`
   - `stripe_refund_id`
   - `stripe_account_id`

6. **DB types を再生成する**
   - 生成済み TypeScript DB types を schema と揃える
   - SQL と app types の drift は release blocker として扱う

最低限のチェック項目は `schema-checklist.md` を参照してください。

## よくある正当化

| Rationalization | Reality |
|---|---|
| 「匿名ユーザーは `tutor_profiles` を直接 query すればいい」 | それでは public page が private な payment state に結び付き、将来の漏えいリスクが上がります。 |
| 「security definer view のほうが簡単」 | 同期された public table のほうが public contract を明示でき、推論しやすくなります。 |
| 「RLS は後から足せばいい」 | schema が本番で動き始めると、あらゆる API 判断が巻き戻しにくくなります。 |
| 「DB trigger は不要で、signup 後に app が row を作ればいい」 | auth bootstrap は auth data の近くに置くべきです。app だけの bootstrap は壊れやすいです。 |

## レッドフラグ

- public page が private な tutor table を直接 query している
- table はあるのに RLS が有効化されていない
- booking status が制約のない自由入力テキストになっている
- SQL schema を変えたのに生成済み DB types を再生成していない
- webhook lookup column に index がない

## 検証

- [ ] `profiles`, `tutor_profiles`, `lesson_offerings`, `bookings`, `public_tutor_directory` が存在する
- [ ] role bootstrap trigger が `profiles` と tutor row を正しく作成する
- [ ] public tutor directory が public field だけを含んでいる
- [ ] RLS が有効で、policy が ownership rule と一致している
- [ ] Stripe lookup column に index がある
- [ ] schema 変更後に DB types を再生成している
