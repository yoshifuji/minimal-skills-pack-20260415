# 機能別 Web アプリ skills 設計書

## 目的

この文書は、多くの Web アプリで繰り返し必要になる機能を、**最小 8 本の skill** として整理するための設計書です。

対象にしているのは、たとえば次のような開発です。

- 会員登録とログインがある
- DB とつながる
- 一覧、詳細、作成、更新、削除がある
- 入力フォームと入力チェックがある
- 権限制御がある
- 画像やファイルを扱う
- 必要に応じて決済がある

## 基本方針

- 1 skill は 1 機能のまとまりにする
- 業務内容ではなく、実装機能で分ける
- 多くの Web アプリで繰り返し使える単位にする
- 最初は 8 本に絞る
- 足りないものを増やす前に、まず 8 本でどこまで作れるかを見る

## 最小 8 本

1. `auth-and-session`
2. `schema-and-migrations`
3. `database-integration`
4. `crud-and-route-handlers`
5. `forms-and-validation`
6. `role-based-access-control`
7. `file-upload-and-storage`
8. `billing-and-payments`

## 一覧表

| skill 名 | 主な役割 | よくある利用場面 |
|---|---|---|
| `auth-and-session` | 認証とログイン状態の維持 | 会員登録、ログイン、閲覧制限 |
| `schema-and-migrations` | DB 設計と DB 変更手順 | テーブル設計、制約、検索を速くする設定 |
| `database-integration` | DB 読み書きの整理 | 共通処理、サーバー側の DB 呼び出し |
| `crud-and-route-handlers` | 基本操作と API | 一覧、詳細、作成、更新、削除 |
| `forms-and-validation` | 入力フォームと入力チェック | 登録フォーム、設定変更フォーム |
| `role-based-access-control` | 権限制御 | 管理者、一般利用者、所有者チェック |
| `file-upload-and-storage` | ファイル管理 | 画像、添付ファイル、保存先管理 |
| `billing-and-payments` | 決済管理 | 支払い開始、状態更新、返金 |

## 1. `auth-and-session`

### 役割

会員登録、ログイン、ログアウト、ログイン状態の維持、閲覧制限ページ、現在の利用者情報の取得をまとめて扱います。

### 使う場面

- 会員登録が必要
- ログイン後だけ見せる画面がある
- 画面上で現在の利用者を表示したい
- サーバー側で本人確認をしたい

### 参照資料

- `references/認証フロー.md`
- `references/パスワード再設定チェックリスト.md`
- `references/セッション境界チェックリスト.md`

## 2. `schema-and-migrations`

### 役割

DB の構造設計、DB 変更手順、制約、検索を速くする設定を扱います。

### 使う場面

- 新しいテーブルを作る
- 項目を増やす、直す
- 制約を追加する
- 後から安全に変更できる形を保ちたい

### 参照資料

- `references/DB変更チェックリスト.md`
- `references/インデックス設計ガイド.md`
- `references/DB設計レビューチェックリスト.md`

## 3. `database-integration`

### 役割

アプリから DB を安全に読み書きするために、DB 操作の置き場所と共通処理を整理します。

### 使う場面

- 一覧や詳細を DB から取る
- 作成、更新、削除をまとめたい
- 画面や API ごとに DB 呼び出しを書き散らしたくない

### 参照資料

- `references/DBアクセスパターン.md`
- `references/問い合わせエラー対応.md`
- `references/サーバーと画面の境界.md`

## 4. `crud-and-route-handlers`

### 役割

一覧、詳細、作成、更新、削除の基本操作と、API の受け口の形をそろえます。

### 使う場面

- 一覧画面や詳細画面を作る
- 登録や編集の API を作る
- 成功時と失敗時の返し方をそろえたい

### 参照資料

- `references/API受け口ひな形.md`
- `references/エラー応答ガイド.md`
- `references/基本操作チェックリスト.md`

## 5. `forms-and-validation`

### 役割

入力フォーム、入力値の検証、エラーメッセージ、保存後の状態更新を扱います。

### 使う場面

- 新規登録フォーム
- 編集フォーム
- 入力エラーをわかりやすくしたい

### 参照資料

- `references/フォームパターン集.md`
- `references/入力エラーメッセージガイド.md`
- `references/入力の抜け漏れ集.md`

## 6. `role-based-access-control`

### 役割

役割ごとの権限、所有者チェック、閲覧制限ページ、管理者権限を扱います。

### 使う場面

- 管理者だけが見られる画面がある
- 自分のデータだけ編集できるようにしたい
- API 側でも権限を確認したい

### 参照資料

- `references/権限表ひな形.md`
- `references/所有者確認チェックリスト.md`
- `references/閲覧制限ページガイド.md`

## 7. `file-upload-and-storage`

### 役割

画像アップロード、添付ファイル、保存先の管理、ファイル情報の管理を扱います。

### 使う場面

- プロフィール画像を扱う
- 資料や添付ファイルを保存する
- 差し替えや削除を管理したい

### 参照資料

- `references/アップロードチェックリスト.md`
- `references/保存先セキュリティガイド.md`
- `references/画像取り扱いガイド.md`

## 8. `billing-and-payments`

### 役割

決済、支払い状態の更新、支払い結果の受け取り、返金を扱います。

### 使う場面

- 支払いを受け付ける
- 支払い完了後に状態を更新する
- 返金処理を扱う

### 参照資料

- `references/支払い状態遷移.md`
- `references/支払い通知チェックリスト.md`
- `references/返金ガイド.md`

## 組み合わせの考え方

### まず基本の 5 本

- `auth-and-session`
- `schema-and-migrations`
- `database-integration`
- `crud-and-route-handlers`
- `forms-and-validation`

### 必要に応じて足す 3 本

- 権限が必要なら `role-based-access-control`
- ファイルが必要なら `file-upload-and-storage`
- 決済が必要なら `billing-and-payments`

## この設計書の使い方

1. 作りたい Web アプリに必要な機能を洗い出す
2. 8 本のうち必要な skill を選ぶ
3. 選んだ skill の `SKILL.md` を読む
4. 対応する `references/` を読む
5. 基本の 5 本から順番に実装する
