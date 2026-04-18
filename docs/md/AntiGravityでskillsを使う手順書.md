---
marp: true
theme: default
paginate: true
size: 16:9
style: |
  section {
    font-size: 20px;
    line-height: 1.45;
  }
  h1, h2 {
    color: #0f172a;
  }
  strong {
    color: #1d4ed8;
  }
  code {
    font-size: 0.88em;
  }
  section.code-fit {
    font-size: 18px;
    line-height: 1.32;
  }
  section.code-fit code {
    font-size: 0.78em;
  }
---

# AntiGravityでskillsを使う手順書

`minimal-skills-pack` を  
**Antigravity で使える形にするための 手順書**

- 対象: `skills/` と `references/` を Antigravity で利用したい人
- 前提: repository root は `C:\Users\username\work\minimal-skills-pack`
- 確認日: 2026-04-18

---

## 先に結論

Antigravity の一次情報で確認できるカスタマイズ機構は  
**Rules** と **Workflows** です。

- Global rule
  - `~/.gemini/GEMINI.md`
- Global workflow
  - `~/.gemini/antigravity/global_workflows/<YOUR_WORKFLOW_NAME>.md`
- Workspace rules
  - `your-workspace/.agents/rules/`
- Workspace workflows
  - `your-workspace/.agents/workflows/`

**Claude Code / Codex のような `SKILL.md` 専用配置先は、確認した一次情報では見当たりません。**  
そのため、この repository では  
**skill を workflow に変換して使う方法** を採ります。

---

## 前提

- skill 本体
  - `C:\Users\username\work\minimal-skills-pack\skills`
- 参照資料
  - `C:\Users\username\work\minimal-skills-pack\references`
- Antigravity 用の配置先
  - `./.agents/workflows`
  - `./.agents/references`

この repository の `SKILL.md` は  
`../../references/...` を参照する前提です。

そのため Antigravity へ持っていくときは、次の 2 つが必要です。

- `references` を `./.agents/references` へコピーする
- workflow 内の参照を `../references/...` へ読み替える

---

## もっとも確実な方法

Antigravity の workspace customization は  
**workflow を `./.agents/workflows` に置く**方式です。

この repository では、次の変換がもっとも確実です。

- `skills/<skill-name>/SKILL.md`
  - `./.agents/workflows/<skill-name>.md` へ変換
- `references/*`
  - `./.agents/references/` へコピー

利点:

- Antigravity の公式な workspace workflows 配置先に合わせられる
- `/workflow-name` で明示呼び出ししやすい
- `references` も同じ workspace 内に閉じ込められる

---

## コマンドはどこで実行するか

この資料では、**Antigravity の中ではなく、先にターミナルで準備** します。

- Windows の場合
  - `PowerShell` または `Windows Terminal` を開く
- Git Bash を使う場合
  - `Git Bash` を開く

このあと出てくるコマンドの最初の 1 行は、  
**作業場所をこの repository に移動する命令** です。

- PowerShell
  - `Set-Location C:\Users\username\work\minimal-skills-pack`
- bash
  - `cd /c/Users/username/work/minimal-skills-pack`

ここでやっていることは次の 3 つです。

1. `./.agents/workflows` を作る
2. `./.agents/references` を作る
3. `skills` を Antigravity 用 workflow に変換する

---

## 変換手順: PowerShell

**どこでやるか**

- Windows の `PowerShell` または `Windows Terminal`

**このコマンドで何をするか**

- repository root に移動する
- Antigravity 用フォルダを作る
- `references` をコピーする
- 各 `SKILL.md` を workflow 用 Markdown に変換する

次のスライドのコマンドを、そのまま貼り付けて実行します。

---

<!-- _class: code-fit -->

## PowerShell コマンド

```powershell
Set-Location C:\Users\username\work\minimal-skills-pack

New-Item -ItemType Directory -Force -Path .\.agents | Out-Null
New-Item -ItemType Directory -Force -Path .\.agents\workflows | Out-Null
New-Item -ItemType Directory -Force -Path .\.agents\references | Out-Null

Copy-Item -Recurse -Force -Path .\references\* -Destination .\.agents\references\

Get-ChildItem -Directory .\skills | ForEach-Object {
  $skillName = $_.Name
  $skillFile = Join-Path $_.FullName 'SKILL.md'
  $outFile = ".\.agents\workflows\$skillName.md"

  $content = Get-Content -LiteralPath $skillFile -Raw
  $content = [regex]::Replace($content, '(?s)^---\r?\n.*?\r?\n---\r?\n', '')
  $content = $content -replace '\.\./\.\./references/', '../references/'

  Set-Content -LiteralPath $outFile -Value $content -Encoding UTF8
}
```

---

## 変換手順: bash

Git Bash 前提です。  
WSL を使う場合は `/mnt/c/Users/...` に読み替えてください。

**どこでやるか**

- `Git Bash`

**このコマンドで何をするか**

- repository root に移動する
- Antigravity 用フォルダを作る
- `references` をコピーする
- 各 `SKILL.md` を workflow 用 Markdown に変換する

次のスライドのコマンドを、そのまま貼り付けて実行します。

---

<!-- _class: code-fit -->

## bash コマンド

```bash
cd /c/Users/username/work/minimal-skills-pack

mkdir -p ./.agents/workflows ./.agents/references
cp -R ./references/. ./.agents/references/

for skill_dir in ./skills/*; do
  skill_name="$(basename "$skill_dir")"
  out="./.agents/workflows/${skill_name}.md"

  awk '
    NR == 1 && $0 == "---" { in_frontmatter = 1; next }
    in_frontmatter && $0 == "---" { in_frontmatter = 0; next }
    !in_frontmatter { print }
  ' "$skill_dir/SKILL.md" \
    | sed 's#\.\./\.\./references/#../references/#g' \
    > "$out"
done
```

---

## 実行後に見る場所

コマンドを打ち終わったら、次の場所を見ます。

- `C:\Users\username\work\minimal-skills-pack\.agents\workflows`
  - workflow に変換された Markdown が入る
- `C:\Users\username\work\minimal-skills-pack\.agents\references`
  - 参照用 Markdown が入る

たとえば次のファイルがあれば、準備は概ね成功です。

- `.agents\workflows\auth-and-session.md`
- `.agents\workflows\forms-and-validation.md`
- `.agents\references\認証フロー.md`

---

## 反映確認

1. Antigravity で repository root を workspace として開く
2. Editor 右上の `...` から `Customizations` を開く
3. `Workflows` を開く
4. `auth-and-session` や `forms-and-validation` などが見えることを確認する
5. Agent chat で `/workflow-name` を入力して使う

例:

```text
/auth-and-session
Supabase を使って signup / login / logout と保護ページを実装してください。
```

公式の一次情報では、Antigravity の workflow は  
**`/` で呼び出す saved prompt** と説明されています。

---

## 呼び出し方のポイント

- Claude Code / Codex の skill と違って
  - Antigravity では **workflow として明示呼び出し** する前提で考える
- 常に効かせたい指示は
  - `./.agents/rules/` 側へ分ける
- task ごとに使い分けたい指示は
  - `./.agents/workflows/` 側へ置く

たとえば次の切り分けが実用的です。

- `forms-and-validation`
  - workflow として都度呼ぶ
- 命名規約やテスト方針
  - rule として常時適用する

---

## skill を更新したとき

`skills/` や `references/` を編集したら、同じ変換を再実行します。

**どこでやるか**

- 前と同じ `PowerShell` または `Git Bash`

**このコマンドで何をするか**

- 変更後の `references` をコピーし直す
- 変更後の `SKILL.md` から workflow を作り直す

次のスライドのコマンドを、そのまま貼り付けて実行します。

---

<!-- _class: code-fit -->

## 再変換コマンド

```powershell
Set-Location C:\Users\username\work\minimal-skills-pack

Copy-Item -Recurse -Force -Path .\references\* -Destination .\.agents\references\

Get-ChildItem -Directory .\skills | ForEach-Object {
  $skillName = $_.Name
  $skillFile = Join-Path $_.FullName 'SKILL.md'
  $outFile = ".\.agents\workflows\$skillName.md"

  $content = Get-Content -LiteralPath $skillFile -Raw
  $content = [regex]::Replace($content, '(?s)^---\r?\n.*?\r?\n---\r?\n', '')
  $content = $content -replace '\.\./\.\./references/', '../references/'

  Set-Content -LiteralPath $outFile -Value $content -Encoding UTF8
}
```

手で編集した workflow は、この再生成で上書きされます。  
workflow 側で独自調整したいなら、元の `SKILL.md` 側も合わせて直してください。

---

## なぜ workspace-local を勧めるか

Antigravity では global workflow の保存先自体は一次情報で確認できます。  
ただし、この repository のように **workflow から別 Markdown を相対参照する構成** を  
global 運用する方法までは、確認した一次情報では明示されていません。

そのため、この資料では
**workspace-local の `./.agents/workflows` と `./.agents/references` を使う運用** を
推奨します。

---

## 補足と一次情報

- 確認した一次情報では、Antigravity の customization は `Rules` と `Workflows` です
- workspace 配置先は `.agents/rules/` と `.agents/workflows/` です
- workflow は `/` で呼び出す saved prompt です
- **Antigravity の `SKILL.md` 専用ディレクトリは、確認した一次情報では見つかっていません**

一次情報:

- Google Codelabs
  - `Getting Started with Google Antigravity`
  - `https://codelabs.developers.google.com/getting-started-google-antigravity`
