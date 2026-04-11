---
name: auth-and-session
description: Webアプリにサインアップ、ログイン、ログアウト、セッション維持、保護ページ、現在ユーザー取得を実装するときに使ってください。認証画面、保護ルート、ログイン状態に応じた表示切り替え、パスワード再設定、メール確認の導線が必要なときにも使います。
---

# Auth And Session

## 概要

認証は画面の一部ではなく、アプリ全体の境界です。user が誰なのか、どこまで見てよいのか、どの request が本人のものかを安定して判断できるようにしてください。ログインが通るだけでは不十分です。current user の取得、保護ページ、未認証時の遷移、ログアウト後の整合まで含めて実装してください。

## コアプロセス

1. **認証の入口と出口を固定する**
   - `/signup`, `/login`, `/logout` の導線を決める
   - 必要なら password reset、email verify の導線も決める
   - 認証済み user を `/login` や `/signup` に居残らせない

2. **client の責務を分ける**
   - browser client、server client、admin client を混ぜない
   - server-only な secret や管理権限を browser へ漏らさない
   - current user を取る helper を共通化する

3. **session を継続させる**
   - framework が必要とする middleware や cookie 更新処理を入れる
   - token refresh が必要な実装では、通常 request で自然に更新できるようにする
   - webhook や raw body route では不要な middleware を避ける

4. **保護ページを server 側で守る**
   - page 表示前に current user を確認する
   - 未認証ならログインへ飛ばす
   - 認証済みなら user 用画面へ進める

5. **認証まわりの helper を `lib/` へ寄せる**
   - `getCurrentUser()`
   - `requireUser()`
   - `redirectIfAuthenticated()`
   - provider SDK の初期化

6. **認証文言を user language にする**
   - technical な error をそのまま出さない
   - password reset や verify 待ちの状態を user が理解できる文章にする

詳細な流れは `../../references/認証フロー.md`、パスワード再設定の確認項目は `../../references/パスワード再設定チェックリスト.md`、セッションの境界は `../../references/セッション境界チェックリスト.md` を参照してください。

## よくある正当化

| Rationalization | Reality |
|---|---|
| 「login が通るから完成でいい」 | 認証は導線、保護ページ、ログアウト後の挙動まで含めて成立です。 |
| 「current user は component ごとに取ればよい」 | 取得方法が散ると、挙動も散ります。helper を共有してください。 |
| 「client 側で隠しているから保護されている」 | server 側で確認しない限り、保護とは言えません。 |
| 「admin client をそのまま使ったほうが楽」 | 楽なのは最初だけで、漏えいしたら終わりです。 |

## レッドフラグ

- client code に secret key や service role が見える
- `/login` と `/signup` が認証済み user を追い出さない
- current user の取得方法が page ごとに違う
- 保護ページが client-side guard だけに依存している
- logout 後に header や navigation が更新されない

## 検証

- [ ] signup、login、logout の基本導線が通る
- [ ] 未認証 user は保護ページへ入れない
- [ ] 認証済み user は login / signup 画面に居続けない
- [ ] current user を共通 helper から取得している
- [ ] secret を使う client が browser bundle に入っていない
