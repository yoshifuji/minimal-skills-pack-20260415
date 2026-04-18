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

# Codexでskillsを使う手順書

`minimal-skills-pack-20260415` を  
**Codex で使える形にするための Marp 版手順書**

- 対象: `skills/` と `references/` を Codex で利用したい人
- 前提: repository root は `C:\Users\username\work\minimal-skills-pack-20260415`
- 確認日: 2026-04-15

---

## 前提

- skill 本体
  - `C:\Users\username\work\minimal-skills-pack-20260415\skills`
- 参照資料
  - `C:\Users\username\work\minimal-skills-pack-20260415\references`
- Codex 用の配置先
  - `./.agents/skills`
  - `./.agents/references`

この repository の `SKILL.md` は  
`../../references/...` を参照する前提です。  
そのため **`skills` と `references` をセットで配置** します。

---

## もっとも確実な方法

Codex の repository-local skill 配置は次です。

- `.agents/skills/<skill-name>/SKILL.md`

この repository では、**project 内へコピーして使う方式** がもっとも確実です。

- 利点
  - Codex の標準構成にそのまま合わせられる
  - skill 側の相対参照を保ちやすい
  - project ごとに完結する

---

## コピー手順: PowerShell

```powershell
Set-Location C:\Users\username\work\minimal-skills-pack-20260415

New-Item -ItemType Directory -Force -Path .\.agents | Out-Null
New-Item -ItemType Directory -Force -Path .\.agents\skills | Out-Null
New-Item -ItemType Directory -Force -Path .\.agents\references | Out-Null

Copy-Item -Recurse -Force -Path .\skills\* -Destination .\.agents\skills\
Copy-Item -Recurse -Force -Path .\references\* -Destination .\.agents\references\
```

`./.agents` がまだ無い場合も、このコマンドでまとめて作成できます。

---

## コピー手順: bash

Git Bash 前提です。  
WSL を使う場合は `/mnt/c/Users/...` に読み替えてください。

```bash
cd /c/Users/username/work/minimal-skills-pack-20260415

mkdir -p ./.agents/skills ./.agents/references

cp -R ./skills/. ./.agents/skills/
cp -R ./references/. ./.agents/references/
```

---

## 反映確認

1. repository root で Codex を起動する
2. `/skills` で一覧を見る
3. `$skill-name` で明示呼び出しする
4. description ベースの暗黙利用も試す

```powershell
Set-Location C:\Users\username\work\minimal-skills-pack-20260415
codex
```

```text
/skills
$auth-and-session
$schema-and-migrations
```

---

## 呼び出し方のポイント

- 明示呼び出し
  - `/skills` で一覧確認
  - `$skill-name` で指定
- 暗黙利用
  - 依頼文の内容が `description` に合うと自動選択される

例:

- 「ログイン、ログアウト、保護ページを実装したい」
  - `auth-and-session` 系の skill が読み込まれやすい

---

## skill を更新したとき

`skills/` や `references/` を編集したら、同じコピーを再実行します。

```powershell
Set-Location C:\Users\username\work\minimal-skills-pack-20260415

Copy-Item -Recurse -Force -Path .\skills\* -Destination .\.agents\skills\
Copy-Item -Recurse -Force -Path .\references\* -Destination .\.agents\references\
```

```bash
cd /c/Users/username/work/minimal-skills-pack-20260415

cp -R ./skills/. ./.agents/skills/
cp -R ./references/. ./.agents/references/
```

Codex は自動検出しますが、反映されない場合は再起動してください。

---

## 全プロジェクトで使いたい場合

global skill として使うなら、配置先は次です。

- `$HOME\.agents\skills`
- `$HOME\.agents\references`

```powershell
Set-Location C:\Users\username\work\minimal-skills-pack-20260415

New-Item -ItemType Directory -Force -Path $HOME\.agents | Out-Null
New-Item -ItemType Directory -Force -Path $HOME\.agents\skills | Out-Null
New-Item -ItemType Directory -Force -Path $HOME\.agents\references | Out-Null

Copy-Item -Recurse -Force -Path .\skills\* -Destination $HOME\.agents\skills\
Copy-Item -Recurse -Force -Path .\references\* -Destination $HOME\.agents\references\
```

どの repository で Codex を起動しても同じ skill を使えます。

---

## 1つの元ディレクトリで管理したい場合

Codex の公式ドキュメントでは  
**symlinked skill folders をサポート** しています。

Windows なら junction、bash なら symlink を使えます。

```powershell
Set-Location C:\Users\username\work\minimal-skills-pack-20260415

New-Item -ItemType Directory -Force -Path .\.agents | Out-Null
New-Item -ItemType Junction -Path .\.agents\skills -Target (Resolve-Path .\skills) | Out-Null
New-Item -ItemType Junction -Path .\.agents\references -Target (Resolve-Path .\references) | Out-Null
```

- 既存の `.agents\skills` や `.agents\references` がある場合は先に整理が必要です
- 安全性を優先するなら、まずはコピー方式で始めるのが無難です

---

## 補足と一次情報

- `SKILL.md` が必須です
- repository-local は `.agents/skills`
- global は `$HOME/.agents/skills`
- 明示呼び出しは `/skills` と `$skill-name`
- symlinked skill folders をサポートしています

一次情報:

- OpenAI Developers
  - `Agent Skills – Codex`
  - `https://developers.openai.com/codex/skills`
