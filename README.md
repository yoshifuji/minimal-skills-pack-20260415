# Webアプリ開発用 Skills 集

この一式は、Web アプリでよく使う機能を **最小 8 本の skill** に絞ってまとめたものです。

## 中心となる 8 本

1. `auth-and-session`
   会員登録、ログイン、ログアウト、ログイン状態の維持
2. `schema-and-migrations`
   DB の設計、変更手順、制約、検索を速くする設定
3. `database-integration`
   アプリから DB を安全に読み書きするための整理
4. `crud-and-route-handlers`
   一覧、詳細、作成、更新、削除と API の受け口
5. `forms-and-validation`
   入力フォーム、入力チェック、エラー表示
6. `role-based-access-control`
   役割ごとの権限、所有者チェック、閲覧制限
7. `file-upload-and-storage`
   画像アップロード、添付ファイル、保存先管理
8. `billing-and-payments`
   決済、支払い状態の更新、返金

## まず読む順番

1. `はじめに.md`
2. `docs/機能別Webアプリskills設計書.md`
3. 必要な skill の `SKILL.md`
4. 対応する `references/`

## まず使いやすい組み合わせ

### 会員登録ありの基本アプリ

- `auth-and-session`
- `schema-and-migrations`
- `database-integration`
- `crud-and-route-handlers`
- `forms-and-validation`

### 管理画面つきアプリ

- 基本の 5 本
- `role-based-access-control`

### 画像アップロードがあるアプリ

- 基本の 5 本
- `file-upload-and-storage`

### 決済があるアプリ

- 基本の 5 本
- `billing-and-payments`

## よく参照する資料

### 認証

- `references/認証フロー.md`
- `references/パスワード再設定チェックリスト.md`
- `references/セッション境界チェックリスト.md`

### DB 設計と DB 接続

- `references/DB変更チェックリスト.md`
- `references/インデックス設計ガイド.md`
- `references/DB設計レビューチェックリスト.md`
- `references/DBアクセスパターン.md`
- `references/問い合わせエラー対応.md`
- `references/サーバーと画面の境界.md`

### 基本操作とフォーム

- `references/API受け口ひな形.md`
- `references/エラー応答ガイド.md`
- `references/基本操作チェックリスト.md`
- `references/フォームパターン集.md`
- `references/入力エラーメッセージガイド.md`
- `references/入力の抜け漏れ集.md`

### 権限、ファイル、決済

- `references/権限表ひな形.md`
- `references/所有者確認チェックリスト.md`
- `references/閲覧制限ページガイド.md`
- `references/アップロードチェックリスト.md`
- `references/保存先セキュリティガイド.md`
- `references/画像取り扱いガイド.md`
- `references/支払い状態遷移.md`
- `references/支払い通知チェックリスト.md`
- `references/返金ガイド.md`

## この一式に残すもの

- `skills/` 配下の最小 8 本
- `references/` 配下の関連資料
- `docs/機能別Webアプリskills設計書.md`
- `はじめに.md`
- `スキルズプロジェクト概要資料.md`

最初は「基本の 5 本」で小さく作り、必要になったときだけ 6〜8 本目を足す進め方を想定しています。
