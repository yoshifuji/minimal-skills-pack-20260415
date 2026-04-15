# Codexでskillsを使う手順書

この手順書は、`C:\Users\yoshi\work\minimal-skills-pack-20260415\skills` 配下の skill を、Codex で使えるようにするためのものです。

確認日: 2026-04-15

## 前提

- repository root: `C:\Users\yoshi\work\minimal-skills-pack-20260415`
- skill 本体: `C:\Users\yoshi\work\minimal-skills-pack-20260415\skills`
- 参照資料: `C:\Users\yoshi\work\minimal-skills-pack-20260415\references`

`bash` の例は Git Bash を想定して `/c/Users/...` 形式で書いています。  
WSL を使う場合は `/mnt/c/Users/...` に読み替えてください。

この repository の各 `SKILL.md` は、`../../references/...` を参照する前提で書かれています。  
そのため、Codex 用の `.agents/skills` だけでなく、`.agents/references` もあわせて用意してください。

## もっとも確実な方法

Codex の公式な repository-local skill 配置は `.agents/skills/<skill-name>/SKILL.md` です。  
この repository では、次のように project 内へコピーして使うのが確実です。  
`./.agents` がまだ存在しない場合は、先に `./.agents` を作成してください。下記のコマンドにはその作成処理も含まれています。

```powershell
Set-Location C:\Users\yoshi\work\minimal-skills-pack-20260415

New-Item -ItemType Directory -Force -Path .\.agents | Out-Null
New-Item -ItemType Directory -Force -Path .\.agents\skills | Out-Null
New-Item -ItemType Directory -Force -Path .\.agents\references | Out-Null

Copy-Item -Recurse -Force -Path .\skills\* -Destination .\.agents\skills\
Copy-Item -Recurse -Force -Path .\references\* -Destination .\.agents\references\
```

```bash
cd /c/Users/yoshi/work/minimal-skills-pack-20260415

mkdir -p ./.agents/skills ./.agents/references

cp -R ./skills/. ./.agents/skills/
cp -R ./references/. ./.agents/references/
```

## 反映確認

1. repository root で Codex を起動します。

```powershell
Set-Location C:\Users\yoshi\work\minimal-skills-pack-20260415
codex
```

```bash
cd /c/Users/yoshi/work/minimal-skills-pack-20260415
codex
```

2. Codex セッション内で skill 一覧を確認します。

```text
/skills
```

3. 明示的に skill を呼ぶときは、`$` で skill 名を指定します。

```text
$auth-and-session
$schema-and-migrations
```

4. 暗黙利用も可能です。  
たとえば「ログイン、ログアウト、保護ページを実装したい」と書くと、`description` に一致した skill が読み込まれます。

## skill を更新したとき

root の `skills/` や `references/` を編集した場合は、もう一度同じコピーを実行してください。

```powershell
Set-Location C:\Users\yoshi\work\minimal-skills-pack-20260415

Copy-Item -Recurse -Force -Path .\skills\* -Destination .\.agents\skills\
Copy-Item -Recurse -Force -Path .\references\* -Destination .\.agents\references\
```

```bash
cd /c/Users/yoshi/work/minimal-skills-pack-20260415

cp -R ./skills/. ./.agents/skills/
cp -R ./references/. ./.agents/references/
```

Codex は skill 変更を自動検出しますが、反映されない場合は Codex を再起動してください。

## 全プロジェクトで使いたい場合

個人環境の global skill として使う場合は、`$HOME\.agents\skills` と `$HOME\.agents\references` に配置します。

```powershell
Set-Location C:\Users\yoshi\work\minimal-skills-pack-20260415

New-Item -ItemType Directory -Force -Path $HOME\.agents | Out-Null
New-Item -ItemType Directory -Force -Path $HOME\.agents\skills | Out-Null
New-Item -ItemType Directory -Force -Path $HOME\.agents\references | Out-Null

Copy-Item -Recurse -Force -Path .\skills\* -Destination $HOME\.agents\skills\
Copy-Item -Recurse -Force -Path .\references\* -Destination $HOME\.agents\references\
```

```bash
cd /c/Users/yoshi/work/minimal-skills-pack-20260415

mkdir -p "$HOME/.agents/skills" "$HOME/.agents/references"

cp -R ./skills/. "$HOME/.agents/skills/"
cp -R ./references/. "$HOME/.agents/references/"
```

この場合、どの repository で Codex を起動しても同じ skill を使えます。

## 1つの元ディレクトリで管理したい場合

Codex の公式ドキュメントでは、skill folder の symlink をサポートしています。  
この repository を単一の正本として維持したい場合は、Windows の junction を使う方法もあります。

```powershell
Set-Location C:\Users\yoshi\work\minimal-skills-pack-20260415

New-Item -ItemType Directory -Force -Path .\.agents | Out-Null
New-Item -ItemType Junction -Path .\.agents\skills -Target (Resolve-Path .\skills) | Out-Null
New-Item -ItemType Junction -Path .\.agents\references -Target (Resolve-Path .\references) | Out-Null
```

```bash
cd /c/Users/yoshi/work/minimal-skills-pack-20260415

mkdir -p ./.agents
ln -s "$(pwd)/skills" ./.agents/skills
ln -s "$(pwd)/references" ./.agents/references
```

既に `.agents\skills` や `.agents\references` がある場合は、先に削除してから作成してください。  
運用上の安全性を優先するなら、まずはコピー方式で始めるほうが無難です。  
Windows の bash で `ln -s` を使う場合は、環境によって管理者権限または Developer Mode が必要です。

## 一次情報

- OpenAI Developers, `Agent Skills – Codex`
  - https://developers.openai.com/codex/skills
  - 確認した要点: `SKILL.md` 必須、明示呼び出しは `/skills` または `$`、repository-local の配置先は `.agents/skills`、`$HOME/.agents/skills` も使用可、symlinked skill folders をサポート
