---
name: schema-and-migrations
description: Webアプリの DB 定義、migration、index、制約、外部キー、変更順序を設計するときに使ってください。新しい table を作るとき、既存 schema を安全に変更するとき、migration と型生成をそろえたいときに使います。
---

# Schema And Migrations

## 概要

DB 定義は、後から直すコストが高い部分です。table、column、constraint、index をその場の都合で足すのではなく、**データの寿命と変更順序**を意識して設計してください。migration は「動けばよい」ではなく、「本番でも安全に流せるか」で判断します。

## コアプロセス

1. **まず entity と寿命を整理する**
   - 何を保存するのか
   - 誰が所有するのか
   - 作成後に何が更新されるのか
   - 削除は論理削除か物理削除か

2. **table と制約を決める**
   - primary key
   - foreign key
   - nullable / not null
   - unique
   - status の allowed values
   - timestamps

3. **index を意図的に入れる**
   - 一覧取得の条件
   - 所有者 lookup
   - 外部サービス id lookup
   - 並び替えで使う列

4. **migration を小さく切る**
   - table 作成
   - backfill
   - not null 制約追加
   - index 追加
   - app code 切り替え
   - 古い column の削除

5. **変更順序を守る**
   - 先に DB を足す
   - その後 app code を切り替える
   - 最後に古い schema を外す

6. **型と schema を同期する**
   - 生成済み DB types があるなら更新する
   - migration だけ変えて型を古いままにしない

詳細な確認項目は `../../references/DB変更チェックリスト.md`、インデックス設計の考え方は `../../references/インデックス設計ガイド.md`、見直しの観点は `../../references/DB設計レビューチェックリスト.md` を参照してください。

## よくある正当化

| Rationalization | Reality |
|---|---|
| 「とりあえず text で保存して後で考える」 | status や ownership の曖昧さは後で大きな不具合になります。 |
| 「index は遅くなってから足せばよい」 | lookup key が明白なら最初から入れるべきです。 |
| 「1 本の migration で全部済ませたい」 | 変更が大きいほど rollback も review も難しくなります。 |
| 「型生成は後回しでよい」 | schema と型の不一致は app 側のバグを増やします。 |

## レッドフラグ

- foreign key がなく、所有関係が code 側だけにある
- lookup で使う列に index がない
- breaking change を 1 回の migration に詰め込んでいる
- nullable にした理由を説明できない
- schema は変えたのに生成済み型が古い

## 検証

- [ ] table、constraint、index の意図を説明できる
- [ ] migration が安全な順序で分かれている
- [ ] ownership や status の制約が schema に表現されている
- [ ] 主要 lookup 列に index がある
- [ ] schema 変更後に型や生成物を同期している
