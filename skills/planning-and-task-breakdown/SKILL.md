---
name: planning-and-task-breakdown
description: 作業を順序立った task に分解するスキルです。spec や明確な要件があり、実装可能な task に落としたいときに使ってください。task が大きすぎて着手しづらいとき、scope を見積もりたいとき、並列作業が可能なときにも使います。
---

# 計画とタスク分解

## 概要

作業を、明示的な acceptance criteria を持つ小さく検証可能な task に分解します。良い task 分解は、確実に仕事を終える agent と、絡まった混乱を生む agent の差になります。すべての task は、1 回の集中した session で実装、テスト、検証できる小ささであるべきです。

## 使う場面

- spec があり、実装可能な単位に分けたいとき
- task が大きすぎる、または曖昧で着手しづらいとき
- 複数 agent や複数 session に並列化したいとき
- human に scope を伝える必要があるとき
- 実装順が自明でないとき

**使わない場面:** scope が明白な単一ファイル変更、または spec 自体に十分定義された task がすでに入っている場合。

## 計画プロセス

### Step 1: Plan Mode に入る

コードを書く前に、read-only mode で動きます。

- spec と関連コードベースを読む
- 既存の pattern と convention を特定する
- component 間の依存関係を整理する
- risk と unknown を記録する

**計画中にコードを書いてはいけません。** 出力は implementation ではなく plan document です。

### Step 2: 依存グラフを特定する

何が何に依存しているかを整理します。

```
Database schema
    │
    ├── API models/types
    │       │
    │       ├── API endpoints
    │       │       │
    │       │       └── Frontend API client
    │       │               │
    │       │               └── UI components
    │       │
    │       └── Validation logic
    │
    └── Seed data / migrations
```

実装順は依存グラフを下から積み上げます。基盤を先に作ります。

### Step 3: 垂直スライスに分ける

database を全部、API を全部、UI を全部という順ではなく、1 本の完全な機能経路を順に作ります。

**悪い例（水平分割）:**
```
Task 1: Build entire database schema
Task 2: Build all API endpoints
Task 3: Build all UI components
Task 4: Connect everything
```

**良い例（垂直分割）:**
```
Task 1: User can create an account (schema + API + UI for registration)
Task 2: User can log in (auth schema + API + UI for login)
Task 3: User can create a task (task schema + API + UI for creation)
Task 4: User can view task list (query + API + UI for list view)
```

各垂直スライスは、動作し、テスト可能な機能を届けます。

### Step 4: task を書く

各 task は次の構造に従います。

```markdown
## Task [N]: [短い説明的タイトル]

**Description:** この task が何を達成するかを説明する 1 段落。

**Acceptance criteria:**
- [ ] [具体的でテスト可能な条件]
- [ ] [具体的でテスト可能な条件]

**Verification:**
- [ ] Tests pass: `npm test -- --grep "feature-name"`
- [ ] Build succeeds: `npm run build`
- [ ] Manual check: [何を確認するかの説明]

**Dependencies:** [依存する Task 番号、または "None"]

**Files likely touched:**
- `src/path/to/file.ts`
- `tests/path/to/test.ts`

**Estimated scope:** [Small: 1-2 files | Medium: 3-5 files | Large: 5+ files]
```

### Step 5: 順番を付け、checkpoint を置く

task は次のように並べます。

1. Dependencies are satisfied (build foundation first)
2. Each task leaves the system in a working state
3. Verification checkpoints occur after every 2-3 tasks
4. High-risk tasks are early (fail fast)

明示的な checkpoint を入れます。

```markdown
## Checkpoint: After Tasks 1-3
- [ ] All tests pass
- [ ] Application builds without errors
- [ ] Core user flow works end-to-end
- [ ] Review with human before proceeding
```

## task サイズの指針

| Size | Files | Scope | Example |
|------|-------|-------|---------|
| **XS** | 1 | Single function or config change | Add a validation rule |
| **S** | 1-2 | One component or endpoint | Add a new API endpoint |
| **M** | 3-5 | One feature slice | User registration flow |
| **L** | 5-8 | Multi-component feature | Search with filtering and pagination |
| **XL** | 8+ | **Too large — break it down further** | — |

task が L 以上なら、さらに小さく分解すべきです。agent は S と M の task で最も安定して動きます。

**さらに分解すべきサイン:**
- 1 回の集中 session を超える時間がかかる（目安: agent 作業で 2 時間超）
- acceptance criteria を 3 個以下の bullet で説明できない
- 2 つ以上の独立 subsystem にまたがる（例: auth と billing）
- task title に "and" が入り始める（2 つの task である兆候）

## plan document テンプレート

```markdown
# 実装計画: [機能名 / プロジェクト名]

## 概要
[何を作るかの 1 段落要約]

## アーキテクチャ判断
- [重要な判断 1 と理由]
- [重要な判断 2 と理由]

## Task List

### Phase 1: Foundation
- [ ] Task 1: ...
- [ ] Task 2: ...

### Checkpoint: Foundation
- [ ] Tests pass, builds clean

### Phase 2: Core Features
- [ ] Task 3: ...
- [ ] Task 4: ...

### Checkpoint: Core Features
- [ ] End-to-end flow works

### Phase 3: Polish
- [ ] Task 5: ...
- [ ] Task 6: ...

### Checkpoint: Complete
- [ ] All acceptance criteria met
- [ ] Ready for review

## リスクと対策
| Risk | Impact | Mitigation |
|------|--------|------------|
| [Risk] | [High/Med/Low] | [Strategy] |

## 未解決事項
- [human の入力が必要な質問]
```

## 並列化の余地

複数 agent や複数 session を使えるときは次を目安にします。

- **安全に並列化できる:** 独立した feature slice、実装済み機能の test、documentation
- **順番にやるべき:** database migration、shared state 変更、依存チェーン
- **調整が必要:** API contract を共有する feature（先に contract を定義してから並列化）

## よくある正当化

| Rationalization | Reality |
|---|---|
| 「進めながら考えればいい」 | そのやり方は絡まった実装と手戻りを生みます。10 分の計画が数時間を救います。 |
| 「task は明白」 | それでも書き出してください。明示化すると隠れた依存や抜けた edge case が見えます。 |
| 「計画はオーバーヘッド」 | 計画こそ task です。計画なしの implementation はただのタイピングです。 |
| 「全部頭に入っている」 | context window は有限です。書かれた計画は session 境界や compaction を越えて残ります。 |

## レッドフラグ

- 書かれた task list なしで implementation を始める
- acceptance criteria のない「機能を実装する」だけの task
- plan に verification step がない
- すべての task が XL サイズ
- task 間に checkpoint がない
- dependency order を考慮していない

## 検証

implementation を始める前に次を確認します。

- [ ] すべての task に acceptance criteria がある
- [ ] すべての task に verification step がある
- [ ] task dependency が特定され、正しく並べられている
- [ ] どの task も触るファイルが概ね 5 個以下である
- [ ] major phase 間に checkpoint がある
- [ ] human が plan を確認し、承認している
