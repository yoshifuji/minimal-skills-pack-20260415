---
name: forms-and-validation
description: Webアプリの入力フォーム、入力値検証、エラーメッセージ、送信後の状態更新を実装するときに使ってください。新規作成フォーム、編集フォーム、検索フォーム、サーバー側検証、わかりやすい入力エラー表示が必要なときに使います。
---

# Forms And Validation

## 概要

フォームは user がもっとも直接触る部分です。入力欄を並べるだけではなく、**何を入れればよいか、何が間違っているか、送信後に何が起きたか**が伝わる必要があります。検証は server 側を最終基準にし、client 側では入力しやすさを補助してください。

## コアプロセス

1. **先に入力契約を決める**
   - 必須項目
   - 任意項目
   - 型
   - 長さ
   - allowed values
   - 初期値

2. **検証を 1 か所にまとめる**
   - schema を定義する
   - client 側で補助的に使う
   - server 側で最終判定する

3. **フォームの状態を明示する**
   - 初期状態
   - 入力中
   - 送信中
   - 成功
   - 失敗

4. **error を field と form 全体に分ける**
   - field 単位の入力エラー
   - 権限不足
   - 保存失敗
   - 予期しない障害

5. **保存後の挙動を決める**
   - 成功 message
   - redirect
   - `router.refresh()`
   - form reset
   - 再送信防止

6. **操作しやすさを守る**
   - label を付ける
   - submit button の状態を分かりやすくする
   - keyboard 操作を壊さない
   - error message を user language で出す

パターン集は `../../references/フォームパターン集.md`、文言の考え方は `../../references/入力エラーメッセージガイド.md`、入力の抜け漏れは `../../references/入力の抜け漏れ集.md` を参照してください。

## よくある正当化

| Rationalization | Reality |
|---|---|
| 「client 側でチェックしているから十分」 | client は補助です。最終判定は server です。 |
| 「error は toast 1 つで出せばよい」 | field ごとの間違いは field の近くで示すほうが分かりやすいです。 |
| 「保存後に画面を全部 reload すればよい」 | reload で済む場合もありますが、成功後の user 体験を考えてください。 |
| 「同じ schema を各所にコピペしても問題ない」 | 条件が変わったときに破綻します。1 か所に寄せてください。 |

## レッドフラグ

- 必須項目が UI で分からない
- server 側 validation がない
- field error と form error が混ざっている
- submit 中に多重送信できる
- 保存失敗時に user が何を直せばよいか分からない

## 検証

- [ ] 入力契約が schema として定義されている
- [ ] server 側が最終判定になっている
- [ ] field error と form error が分かれている
- [ ] 送信中、成功、失敗の状態が分かる
- [ ] error message が user language になっている
