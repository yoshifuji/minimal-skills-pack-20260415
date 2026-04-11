---
name: file-upload-and-storage
description: Webアプリで画像やファイルのアップロード、保存、表示、削除を実装するときに使ってください。プロフィール画像、添付ファイル、公開 URL、署名 URL、アップロード後のプレビューやメタデータ保存が必要なときに使います。
---

# File Upload And Storage

## 概要

ファイル upload は、見た目よりずっと失敗しやすい機能です。サイズ制限、形式制限、保存先、権限、古いファイルの cleanup まで決めておかないと、あとで壊れます。ファイル本体と DB のメタデータをどう対応させるかを先に決めてください。

## コアプロセス

1. **何を保存するか決める**
   - 画像
   - 添付ファイル
   - 公開ファイル
   - user ごとの private file

2. **保存先と公開方法を決める**
   - public URL
   - signed URL
   - 一時 URL
   - browser から直接 upload するか、server 経由にするか

3. **メタデータを持つ**
   - file path
   - file name
   - mime type
   - size
   - owner id
   - uploaded at

4. **upload 前に検証する**
   - 最大サイズ
   - 許可する形式
   - 必要なら画像の縦横
   - owner や対象 resource の権限

5. **置き換えと削除の方針を決める**
   - 新しい画像に差し替えるときの古いファイル削除
   - DB 更新失敗時の orphan file
   - file だけ消えて metadata が残る事故

6. **表示まで含めて設計する**
   - preview の出し方
   - fallback 画像
   - 画像読み込み失敗時の扱い
   - ダウンロード用ファイル名

全体の確認は `../../references/アップロードチェックリスト.md`、保存先の安全性は `../../references/保存先セキュリティガイド.md`、画像の扱いは `../../references/画像取り扱いガイド.md` を参照してください。

## よくある正当化

| Rationalization | Reality |
|---|---|
| 「とりあえず upload できれば完成」 | 表示、削除、差し替えまでないと運用で詰まります。 |
| 「拡張子だけ見れば十分」 | mime type や必要なら中身確認まで考えるべきです。 |
| 「public で置けば楽」 | user ごとの private file は公開 URL に向きません。 |
| 「古いファイルの cleanup は後で考える」 | 後で必ずゴミが残ります。先に方針を決めてください。 |

## レッドフラグ

- サイズ制限がない
- owner チェックなしで upload / delete できる
- file path と DB metadata の対応がない
- 差し替え時の古いファイル削除方針がない
- signed URL が必要なのに public URL を使っている

## 検証

- [ ] 保存先と公開方法を説明できる
- [ ] size / type / owner の確認がある
- [ ] file metadata を DB に保存している
- [ ] 差し替えと削除の flow が決まっている
- [ ] private file を public URL にしていない
