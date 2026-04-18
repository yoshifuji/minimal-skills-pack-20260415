---
marp: true
theme: default
paginate: true
size: 16:9
style: |
  section {
    font-size: 21px;
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
---

# Claude Codeでskillsを使う手順書

`minimal-skills-pack-20260415` を  
**Claude Code で使える形にするための Marp 版手順書**

- 対象: `skills/` と `references/` を Claude Code で利用したい人
- 前提: repository root は `C:\Users\username\work\minimal-skills-pack-20260415`
- 確認日: 2026-04-15

---

## 前提

- skill 本体
  - `C:\Users\username\work\minimal-skills-pack-20260415\skills`
- 参照資料
  - `C:\Users\username\work\minimal-skills-pack-20260415\references`
- Claude Code 用の配置先
  - `./.claude/skills`
  - `./.claude/references`

この repository の `SKILL.md` は  
`../../references/...` を参照する前提です。  
そのため **`skills` だけでなく `references` も一緒に置く** 必要があります。

---

## もっとも確実な方法

Claude Code の project-local skill 配置は次です。

- `.claude/skills/<skill-name>/SKILL.md`

この repository では、**project 内へコピーして使う方式** がもっとも確実です。

- 利点
  - Claude Code の標準構成にそのまま合わせられる
  - 相対参照を崩しにくい
  - project 単位で完結する

---

## コピー手順: PowerShell

```powershell
Set-Location C:\Users\username\work\minimal-skills-pack-20260415

New-Item -ItemType Directory -Force -Path .\.claude | Out-Null
New-Item -ItemType Directory -Force -Path .\.claude\skills | Out-Null
New-Item -ItemType Directory -Force -Path .\.claude\references | Out-Null

Copy-Item -Recurse -Force -Path .\skills\* -Destination .\.claude\skills\
Copy-Item -Recurse -Force -Path .\references\* -Destination .\.claude\references\
```

---

## コピー手順: bash

Git Bash 前提です。  
WSL を使う場合は `/mnt/c/Users/...` に読み替えてください。

```bash
cd /c/Users/username/work/minimal-skills-pack-20260415

mkdir -p ./.claude/skills ./.claude/references

cp -R ./skills/. ./.claude/skills/
cp -R ./references/. ./.claude/references/
```

---

## 反映確認

1. repository root で Claude Code を起動する
2. slash command で skill を明示呼び出しする
3. 必要なら description ベースの暗黙利用も試す

```powershell
Set-Location C:\Users\username\work\minimal-skills-pack-20260415
claude
```

```text
/auth-and-session
/schema-and-migrations
```

- `name` は `/skill-name` として使われます
- `description` は自動選択の判定に使われます

---

## 起動後の挙動

- `.claude/skills` が **起動前から存在する状態** なら
  - 追加、編集、削除は現在のセッションに反映されます
- `.claude/skills` を **起動後に新規作成** した場合は
  - 1 回 Claude Code を再起動してください

まずは起動前に `.claude/skills` と `.claude/references` を
作っておく運用が安全です。

---

## skill を更新したとき

`skills/` や `references/` を編集したら、同じコピーを再実行します。

```powershell
Set-Location C:\Users\username\work\minimal-skills-pack-20260415

Copy-Item -Recurse -Force -Path .\skills\* -Destination .\.claude\skills\
Copy-Item -Recurse -Force -Path .\references\* -Destination .\.claude\references\
```

```bash
cd /c/Users/username/work/minimal-skills-pack-20260415

cp -R ./skills/. ./.claude/skills/
cp -R ./references/. ./.claude/references/
```

---

## 全プロジェクトで使いたい場合

global skill として使うなら、配置先は次です。

- `$HOME\.claude\skills`
- `$HOME\.claude\references`

```powershell
Set-Location C:\Users\username\work\minimal-skills-pack-20260415

New-Item -ItemType Directory -Force -Path $HOME\.claude | Out-Null
New-Item -ItemType Directory -Force -Path $HOME\.claude\skills | Out-Null
New-Item -ItemType Directory -Force -Path $HOME\.claude\references | Out-Null

Copy-Item -Recurse -Force -Path .\skills\* -Destination $HOME\.claude\skills\
Copy-Item -Recurse -Force -Path .\references\* -Destination $HOME\.claude\references\
```

どの project で Claude Code を起動しても同じ skill を使えます。

---

## 補足と一次情報

- この repository の skill 構成は Claude Code の `SKILL.md` 形式と整合しています
- symlink / junction 利用は公式ドキュメントで明示されていません
- **確実性を優先するならコピー方式** を使ってください

一次情報:

- Anthropic Claude Code Docs
  - `Extend Claude with skills`
  - `https://code.claude.com/docs/en/slash-commands`

確認した要点:

- `SKILL.md` が必要
- project-local は `.claude/skills`
- global は `~/.claude/skills`
- top-level directory を新規作成した場合は再起動が必要
