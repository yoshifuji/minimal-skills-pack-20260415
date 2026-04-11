---
name: crud-and-route-handlers
description: Webアプリの一覧、詳細、作成、更新、削除を支える API、route handler、server action を実装するときに使ってください。入力受付、レスポンス形式、エラー処理、一覧取得、詳細取得、保存、削除の流れを安定させたいときに使います。
---

# Crud And Route Handlers

## 概要

一覧、詳細、作成、更新、削除は、どの Web アプリでも繰り返し出てきます。route handler や server action は、**薄く、分かりやすく、失敗時の形がそろっていること**が重要です。input を受ける場所、認証を確認する場所、data access を呼ぶ場所を混ぜないでください。

## コアプロセス

1. **先に契約を決める**
   - request body
   - query params
   - path params
   - success response
   - error response

2. **入口で input を確定する**
   - params を読む
   - body を parse する
   - validation する
   - 認証が必要なら先に確認する

3. **処理の責務を分ける**
   - route handler は request / response を扱う
   - service helper は業務ルールを扱う
   - data access 層は DB 読み書きを扱う

4. **一覧と詳細の返し方をそろえる**
   - filter
   - sort
   - pagination
   - not found
   - empty state

5. **作成 / 更新 / 削除の失敗を揃える**
   - validation error
   - duplicate
   - forbidden
   - not found
   - unexpected error

6. **レスポンス形式を project で統一する**
   - 成功時の payload
   - error code
   - user 向け message
   - debug detail を分離する

ひな形は `../../references/API受け口ひな形.md`、エラー応答のそろえ方は `../../references/エラー応答ガイド.md`、基本の確認項目は `../../references/基本操作チェックリスト.md` を参照してください。

## よくある正当化

| Rationalization | Reality |
|---|---|
| 「小さい endpoint だから全部 route に書いてよい」 | 小さく見えても、責務を混ぜると増えた瞬間に崩れます。 |
| 「一覧 API と詳細 API で error 形式が違っても困らない」 | client 側の分岐が増え、変更が面倒になります。 |
| 「delete は 204 で返せば十分」 | UI が必要とする情報があるなら、結果を明示したほうが扱いやすいです。 |
| 「server action と route handler は流儀が違う」 | 表面は違っても、責務分離の原則は同じです。 |

## レッドフラグ

- route handler が DB query と業務ルールを直に持っている
- success と error の shape が endpoint ごとに違う
- not found と forbidden の区別がない
- 一覧の filter / sort / page の扱いが page ごとに違う
- unexpected error をそのまま user へ返している

## 検証

- [ ] request と response の形が先に定義されている
- [ ] validation、認証、保存処理の責務が分かれている
- [ ] CRUD ごとの error が project 内でそろっている
- [ ] 一覧、詳細、保存、削除の処理が読みやすく分割されている
- [ ] route handler が thin である
