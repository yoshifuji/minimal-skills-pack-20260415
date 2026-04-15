# Claude Codeでskillsを使う手順書

この手順書は、`C:\Users\yoshi\work\minimal-skills-pack-20260415\skills` 配下の skill を、Claude Code で使えるようにするためのものです。

確認日: 2026-04-15

## 前提

- repository root: `C:\Users\yoshi\work\minimal-skills-pack-20260415`
- skill 本体: `C:\Users\yoshi\work\minimal-skills-pack-20260415\skills`
- 参照資料: `C:\Users\yoshi\work\minimal-skills-pack-20260415\references`

`bash` の例は Git Bash を想定して `/c/Users/...` 形式で書いています。  
WSL を使う場合は `/mnt/c/Users/...` に読み替えてください。

この repository の各 `SKILL.md` は、`../../references/...` を参照する前提で書かれています。  
そのため、Claude Code 用の `.claude/skills` だけでなく、`.claude/references` もあわせて用意してください。

## もっとも確実な方法

Claude Code の公式な project-local skill 配置は `.claude/skills/<skill-name>/SKILL.md` です。  
この repository では、次のように project 内へコピーして使うのが確実です。

```powershell
Set-Location C:\Users\yoshi\work\minimal-skills-pack-20260415

New-Item -ItemType Directory -Force -Path .\.claude | Out-Null
New-Item -ItemType Directory -Force -Path .\.claude\skills | Out-Null
New-Item -ItemType Directory -Force -Path .\.claude\references | Out-Null

Copy-Item -Recurse -Force -Path .\skills\* -Destination .\.claude\skills\
Copy-Item -Recurse -Force -Path .\references\* -Destination .\.claude\references\
```

```bash
cd /c/Users/yoshi/work/minimal-skills-pack-20260415

mkdir -p ./.claude/skills ./.claude/references

cp -R ./skills/. ./.claude/skills/
cp -R ./references/. ./.claude/references/
```

## 反映確認

1. repository root で Claude Code を起動します。

```powershell
Set-Location C:\Users\yoshi\work\minimal-skills-pack-20260415
claude
```

```bash
cd /c/Users/yoshi/work/minimal-skills-pack-20260415
claude
```

2. 明示的に呼ぶときは slash command で skill 名を指定します。

```text
/auth-and-session
/schema-and-migrations
```

3. 暗黙利用も可能です。  
現在の `SKILL.md` には `name` と `description` が入っているため、Claude Code は `description` を見て自動で skill を選べます。

4. `.claude/skills` を Claude Code 起動後に新規作成した場合は、1 回再起動してください。  
既に `.claude/skills` が存在している状態での追加・編集・削除は、現在のセッションに自動反映されます。

## skill を更新したとき

root の `skills/` や `references/` を編集した場合は、もう一度同じコピーを実行してください。

```powershell
Set-Location C:\Users\yoshi\work\minimal-skills-pack-20260415

Copy-Item -Recurse -Force -Path .\skills\* -Destination .\.claude\skills\
Copy-Item -Recurse -Force -Path .\references\* -Destination .\.claude\references\
```

```bash
cd /c/Users/yoshi/work/minimal-skills-pack-20260415

cp -R ./skills/. ./.claude/skills/
cp -R ./references/. ./.claude/references/
```

## 全プロジェクトで使いたい場合

個人環境の global skill として使う場合は、`$HOME\.claude\skills` と `$HOME\.claude\references` に配置します。

```powershell
Set-Location C:\Users\yoshi\work\minimal-skills-pack-20260415

New-Item -ItemType Directory -Force -Path $HOME\.claude | Out-Null
New-Item -ItemType Directory -Force -Path $HOME\.claude\skills | Out-Null
New-Item -ItemType Directory -Force -Path $HOME\.claude\references | Out-Null

Copy-Item -Recurse -Force -Path .\skills\* -Destination $HOME\.claude\skills\
Copy-Item -Recurse -Force -Path .\references\* -Destination $HOME\.claude\references\
```

```bash
cd /c/Users/yoshi/work/minimal-skills-pack-20260415

mkdir -p "$HOME/.claude/skills" "$HOME/.claude/references"

cp -R ./skills/. "$HOME/.claude/skills/"
cp -R ./references/. "$HOME/.claude/references/"
```

この場合、どの project で Claude Code を起動しても同じ skill を使えます。

## 補足

- Claude Code の `name` は `/skill-name` になります。
- `description` は自動呼び出しの判定に使われます。
- この repository の skill 構成は Claude Code の `SKILL.md` 形式と整合しています。
- ただし、Claude Code の公式ドキュメントでは skill directory の symlink / junction 利用までは明示されていません。確実性を優先するなら、この手順書どおりコピー方式で運用してください。

## 一次情報

- Anthropic Claude Code Docs, `Extend Claude with skills`
  - https://code.claude.com/docs/en/slash-commands
  - 確認した要点: `SKILL.md` 必須、`name` は `/skill-name` になる、project-local の配置先は `.claude/skills`、global は `~/.claude/skills`、起動後の変更は live detection されるが top-level directory を新規作成した場合は再起動が必要
