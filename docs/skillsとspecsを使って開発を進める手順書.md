# skillsとspecsを使って開発を進める手順書

この手順書は、次の 2 つを AI コーディングエージェントに読ませながら、開発を進めるためのものです。

- この skills 集
- `C:\Users\username\work\minimal-skills-pack-20260415\specs`

対象読者は、Codex または Claude Code を使い始めたばかりの人です。  
「何をどの順番でやればよいか」が分かるように、かなり細かく書いています。

## この手順書でやること

この手順書では、次の流れを扱います。

1. 新しい開発用フォルダを作る
2. `specs` の中から使う仕様書をそのフォルダにコピーする
3. `skills` をそのフォルダで使えるようにする
4. Codex または Claude Code を起動する
5. 最初の入力で、いきなり実装させずに `SPEC.md` とタスク分解を作らせる
6. 1 タスクずつ実装させる
7. 途中で詰まったときに立て直す

## 先に知っておくこと

- `skills` は「作業のやり方の指示書」です。
- `specs` は「今回作るアプリの要求仕様」です。
- 先に仕様とタスク分解を作らせてから実装に入ると、失敗しにくくなります。
- 初学者ほど、最初から「全部作って」と頼まないほうが安全です。

## おすすめの進め方

おすすめは、次の 2 つを分けて考えるやり方です。

- skills 集の repository
  - `C:\Users\username\work\minimal-skills-pack-20260415`
- 実際にアプリを作る作業用 project
  - 例: `C:\Users\username\work\my-first-app`

こうして分けると、skills 集そのものを壊さずに開発できます。

## 事前準備

開発を始める前に、次のどちらを使うか決めてください。

- Codex
- Claude Code

skills の有効化手順は、先に次の手順書を見てください。

- Codex を使う場合
  - [Codexでskillsを使う手順書.md](<./Codexでskillsを使う手順書.md>)
- Claude Code を使う場合
  - [ClaudeCodeでskillsを使う手順書.md](<./ClaudeCodeでskillsを使う手順書.md>)

## Step 1. 作業用 project を作る

まず、実際にアプリを作るための空フォルダを作ります。  
ここでは例として `my-first-app` という名前にします。

### PowerShell

```powershell
New-Item -ItemType Directory -Force -Path C:\Users\username\work\my-first-app | Out-Null
Set-Location C:\Users\username\work\my-first-app
```

### bash

`bash` の例は Git Bash を想定して `/c/Users/...` 形式で書いています。  
WSL を使う場合は `/mnt/c/Users/...` に読み替えてください。

```bash
mkdir -p /c/Users/username/work/my-first-app
cd /c/Users/username/work/my-first-app
```

## Step 2. 使う spec を project にコピーする

`specs` フォルダの中身を直接参照させることもできます。  
ただし、初学者は **使う spec ファイルを project 内へコピーしてから進める** ほうが安全です。

理由は次のとおりです。

- AI に「どの spec を読めばよいか」を明確に伝えやすい
- project の中だけ見れば必要な情報がそろう
- 将来、spec を少し書き換えたくなったときに project 側で管理しやすい

今回は `specs` にある次のファイルを使う例で説明します。

- `C:\Users\username\work\minimal-skills-pack-20260415\specs\汎用サービス(予約マーケットプレイス)仕様プロンプト.md`

### PowerShell

```powershell
Set-Location C:\Users\username\work\my-first-app

New-Item -ItemType Directory -Force -Path .\docs\input-specs | Out-Null

Copy-Item `
  -Path "C:\Users\username\work\minimal-skills-pack-20260415\specs\汎用サービス(予約マーケットプレイス)仕様プロンプト.md" `
  -Destination .\docs\input-specs\
```

### bash

```bash
cd /c/Users/username/work/my-first-app

mkdir -p ./docs/input-specs

cp "/c/Users/username/work/minimal-skills-pack-20260415/specs/汎用サービス(予約マーケットプレイス)仕様プロンプト.md" ./docs/input-specs/
```

コピーできたら、project 側に次のファイルがある状態にしてください。

- `docs/input-specs/汎用サービス(予約マーケットプレイス)仕様プロンプト.md`

## Step 3. skills を project で使えるようにする

ここでやることは、使う AI によって違います。

- Codex の場合
  - project の中に `./.agents/skills` と `./.agents/references` を用意する
- Claude Code の場合
  - project の中に `./.claude/skills` と `./.claude/references` を用意する

詳しいコマンドは、先ほどの手順書をそのまま使ってください。  
この Step が終わった時点で、開発用 project の中で skills が使える状態になっていれば大丈夫です。

## Step 4. AI を起動する

skills を有効にしたあと、作業用 project で AI を起動します。

### Codex の場合

#### PowerShell

```powershell
Set-Location C:\Users\username\work\my-first-app
codex
```

#### bash

```bash
cd /c/Users/username/work/my-first-app
codex
```

### Claude Code の場合

#### PowerShell

```powershell
Set-Location C:\Users\username\work\my-first-app
claude
```

#### bash

```bash
cd /c/Users/username/work/my-first-app
claude
```

## Step 5. 最初の入力は「計画だけ」にする

ここが一番大事です。  
最初の入力では、**まだ実装させません**。

最初に AI にやらせるのは次の 3 つです。

1. repository と spec を読む
2. 必要な skill を選ぶ
3. `SPEC.md` と `tasks/plan.md` と `tasks/todo.md` を作る

次の入力文をそのまま使って大丈夫です。

```text
この project では、利用可能な skills を使いながら開発を進めてください。

まず次のファイルを読んでください。
- docs/input-specs/汎用サービス(予約マーケットプレイス)仕様プロンプト.md

作業の進め方は次の条件に従ってください。
- 最初はいきなり実装しない
- まず仕様を整理して SPEC.md を作る
- 次に tasks/plan.md と tasks/todo.md を作る
- タスクは大きすぎる単位にせず、縦切りで小さく分ける
- 最初の 3 タスクは特に小さく分ける
- どの skill を使うかも明示する

出力してほしいもの:
- SPEC.md
- tasks/plan.md
- tasks/todo.md

この入力では、まだコード実装は始めないでください。
```

## Step 6. AI の返答を確認する

最初の入力のあと、次の 3 つが作られているか確認してください。

- `SPEC.md`
- `tasks/plan.md`
- `tasks/todo.md`

中身を見るときは、次の点を確認します。

- `SPEC.md`
  - 何を作るかが 1 つの文章で説明されているか
  - スコープ内とスコープ外が分かれているか
  - 技術スタックが spec とずれていないか
- `tasks/plan.md`
  - 実装順が無理のない順番になっているか
  - いきなり大機能から始めていないか
- `tasks/todo.md`
  - 1 タスクが大きすぎないか
  - 1 タスク終わるごとに確認できる単位になっているか

## Step 7. タスクが大きすぎるときの修正依頼

もし `tasks/todo.md` の 1 個 1 個が大きすぎると感じたら、そのまま実装に進まないでください。  
次の入力で、先にタスクを細かくさせます。

```text
tasks/todo.md のタスクがまだ大きいです。

次の条件で分割し直してください。
- 1 タスクごとに完了確認しやすい大きさにする
- できるだけ 1 タスクで 1 つの画面、1 つの API、1 つの責務に寄せる
- 各タスクの完了条件を書く
- 依存関係が分かるようにする

まだ実装は始めないでください。
```

## Step 8. 1 タスク目だけ実装させる

計画に納得できたら、**最初の 1 タスクだけ** 実装させます。  
複数タスクをまとめて頼まないことが重要です。

次の入力文を使ってください。

```text
では tasks/todo.md の 1 タスク目だけ実装してください。

条件:
- 1 タスク目以外は進めない
- 変更したファイルを最後にまとめる
- 必要なら関連する小さな補助修正までは行ってよい
- 実装後に typecheck と build を実行する
- 失敗した場合は、何が原因かを整理する

最後に次を報告してください。
- 何を変更したか
- どこまで終わったか
- 実行した確認項目
- 次に進む前に人間が見るべき点
```

## Step 9. 2 タスク目以降も同じ形で進める

2 タスク目以降も、基本は同じです。  
毎回「次の 1 タスクだけ」と伝えて進めます。

```text
では tasks/todo.md の 2 タスク目だけ実装してください。
1 タスクずつ進めたいので、他のタスクには手を出さないでください。
実装後は typecheck と build も実行してください。
```

同じ形で 3 タスク目、4 タスク目と進めます。

## Step 10. 途中で会話が長くなったら再開用の入力を使う

セッションが長くなると、AI が前提を取り違えることがあります。  
そんなときは、次のように「今どこまで終わっているか」を整理させてから再開します。

```text
ここまでの状況を整理してください。

確認したいこと:
- 完了済みタスク
- 未着手タスク
- 今の実装が spec から外れていないか
- 次にやるべき 1 タスク

そのうえで、次は tasks/todo.md の次の 1 タスクだけ進めてください。
```

## Step 11. spec と実装がずれてきたときの戻し方

途中で「仕様と違う方向に進んでいる」と感じたら、すぐに軌道修正してください。  
そのまま進めると、後で大きく戻すことになります。

次の入力が使えます。

```text
いまの実装が spec からずれていないか確認してください。

比較対象:
- docs/input-specs/汎用サービス(予約マーケットプレイス)仕様プロンプト.md
- SPEC.md

やってほしいこと:
- ずれている点を列挙する
- 重大なものから順に出す
- まだ修正は始めず、修正方針だけ出す
```

## Step 12. 最後の仕上げ

主要タスクが終わったら、最後に次をまとめて確認させます。

- `tasks/todo.md` に未完了が残っていないか
- `SPEC.md` の要件を満たしているか
- `npm run typecheck` が通るか
- `npm run build` が通るか
- README や操作手順などの文書が足りているか

最後の確認入力は次のようにします。

```text
最終確認をしてください。

確認対象:
- SPEC.md
- tasks/todo.md
- 現在の実装

やってほしいこと:
- 未完了項目を出す
- 受け入れ条件に未達があれば出す
- typecheck と build を実行する
- ドキュメント不足があれば出す

まだ不足があれば、重大なものから順に並べてください。
```

## 初学者がよく迷うポイント

### 1. 最初から全部作らせる

これは失敗しやすいです。  
必ず「計画だけ」から始めてください。

### 2. 1 回で複数タスクを進めさせる

一気に進めると、どこで壊れたか分かりにくくなります。  
「次の 1 タスクだけ」と毎回伝えてください。

### 3. spec を project 外のまま使う

慣れないうちは、project の `docs/input-specs/` にコピーしてから使うほうが安全です。

### 4. skills を有効化しないまま始める

skills が読み込まれていないと、この repository の狙いが活きません。  
先に Codex または Claude Code 側の skill 設定を済ませてください。

### 5. 実装後の確認を省く

`typecheck` や `build` を飛ばすと、あとで原因不明の不具合が増えます。  
各タスクの最後に毎回確認してください。

## 迷ったときの最小セット

時間がないときは、最低でも次の順で進めてください。

1. 作業用 project を作る
2. spec を `docs/input-specs/` にコピーする
3. skills を有効にする
4. AI に `SPEC.md` と `tasks/plan.md` と `tasks/todo.md` を作らせる
5. 1 タスクずつ実装させる
6. 毎回 `typecheck` と `build` を通す

## 最初の入力文だけ見たい人向け

最後に、最初の 1 回目だけに使う短い版も載せておきます。

```text
この project では、利用可能な skills を使いながら開発を進めてください。
まず docs/input-specs/汎用サービス仕様プロンプト.md を読んでください。

最初にやること:
- SPEC.md を作る
- tasks/plan.md を作る
- tasks/todo.md を作る
- タスクは小さく縦切りで分ける

この入力では、まだ実装しないでください。
```
