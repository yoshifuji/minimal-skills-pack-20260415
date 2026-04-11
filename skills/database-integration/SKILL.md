---
name: database-integration
description: Webアプリから DB を安全に読み書きする data access 層を実装するときに使ってください。一覧取得、詳細取得、作成、更新、削除、query helper、repository、server-only な DB access を整理したいときに使います。
---

# Database Integration

## 概要

DB 呼び出しを route handler や component に散らすと、仕様変更に弱くなります。DB とアプリの間には、**読み書きの責務をまとめた層**を置いてください。query を使い回せること、型が安定していること、server-only 境界が守られていることが重要です。

## コアプロセス

1. **data access の置き場所を決める**
   - `lib/data.ts`
   - `lib/repositories/`
   - `lib/services/`
   - project で一貫した置き場を選ぶ

2. **読み取りと書き込みを分ける**
   - 一覧取得
   - 詳細取得
   - 作成
   - 更新
   - 削除
   - 役割ごとに helper を整理する

3. **戻り値を安定させる**
   - nullable を明示する
   - not found と forbidden を混ぜない
   - UI がそのまま扱える shape に整える

4. **route handler から query 詳細を追い出す**
   - route は input を受ける
   - data access 層は DB を読む
   - business rule は service helper に寄せる

5. **server-only 境界を守る**
   - browser から直接使わない client は分離する
   - secret が必要な access は server 側に閉じる
   - 管理権限 access は専用 helper にする

6. **失敗を明示的に扱う**
   - not found
   - duplicate
   - validation 済みなのに保存できないケース
   - 外部キー制約違反

設計例は `../../references/DBアクセスパターン.md`、失敗時の扱いは `../../references/問い合わせエラー対応.md`、サーバーと画面の境界は `../../references/サーバーと画面の境界.md` を参照してください。

## よくある正当化

| Rationalization | Reality |
|---|---|
| 「この query は 1 回しか使わない」 | 1 回しか使わなくても、route に埋め込むと境界が崩れます。 |
| 「UI で DB shape をそのまま使えば早い」 | DB shape は UI 都合で変わります。直接依存させないでください。 |
| 「null か error かは呼び出し側で考えればよい」 | 呼び出し側が増えるほど、扱いが不統一になります。 |
| 「admin access も同じ helper でよい」 | 権限境界がぼやけます。分けてください。 |

## レッドフラグ

- component から直接 DB client を呼んでいる
- route handler ごとに同じ query がコピーされている
- not found と forbidden が同じ null で返ってくる
- browser で使う module に server-only access が混ざっている
- 更新処理が transaction なしで分散している

## 検証

- [ ] DB 呼び出しの主な置き場が統一されている
- [ ] route handler が query 詳細を持ちすぎていない
- [ ] 戻り値の shape と null 条件が明示されている
- [ ] server-only access が browser へ漏れていない
- [ ] 同じ lookup や update が helper にまとめられている
