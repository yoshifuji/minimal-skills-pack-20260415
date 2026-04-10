---
name: incremental-implementation
description: 変更を段階的に届けるためのスキルです。複数ファイルにまたがる機能や変更を実装するときに使ってください。一度に大量のコードを書こうとしているときや、task が大きすぎて 1 手で着地しづらいときにも使います。
---

# 段階的実装

## 概要

薄い垂直スライスで作ります。1 つ実装し、テストし、検証し、それから広げます。機能全体を 1 回で実装しないでください。各 increment はシステムを動作し、テスト可能な状態で残すべきです。これが大きな機能を扱えるようにする実行規律です。

## 使う場面

- 複数ファイルにまたがる変更を実装するとき
- task breakdown から新機能を作るとき
- 既存コードをリファクタリングするとき
- テスト前に 100 行超を書きたくなっているとき

**使わない場面:** すでに最小 scope である単一ファイル・単一関数の変更。

## increment サイクル

```
┌──────────────────────────────────────┐
│                                      │
│   Implement ──→ Test ──→ Verify ──┐  │
│       ▲                           │  │
│       └───── Commit ◄─────────────┘  │
│              │                       │
│              ▼                       │
│          Next slice                  │
│                                      │
└──────────────────────────────────────┘
```

各 slice でやること:

1. **Implement** 最小の完全機能を作る
2. **Test** テストスイートを回す（なければ書く）
3. **Verify** slice が期待通り動くことを確認する（テスト成功、build 成功、手動確認）
4. **Commit** 説明的なメッセージで進捗を保存する（atomic commit の指針は `git-workflow-and-versioning` を参照）
5. **Move to the next slice** 前に進み、やり直さない

## スライス戦略

### 垂直スライス（推奨）

stack を 1 本通る完全な経路を作ります。

```
Slice 1: Create a task (DB + API + basic UI)
    → Tests pass, user can create a task via the UI

Slice 2: List tasks (query + API + UI)
    → Tests pass, user can see their tasks

Slice 3: Edit a task (update + API + UI)
    → Tests pass, user can modify tasks

Slice 4: Delete a task (delete + API + UI + confirmation)
    → Tests pass, full CRUD complete
```

各 slice は、動く end-to-end 機能を届けます。

### 契約先行スライス

backend と frontend を並列に進める必要があるとき:

```
Slice 0: Define the API contract (types, interfaces, OpenAPI spec)
Slice 1a: Implement backend against the contract + API tests
Slice 1b: Implement frontend against mock data matching the contract
Slice 2: Integrate and test end-to-end
```

### リスク先行スライス

最もリスクが高い、または不確実な部分を最初に片付けます。

```
Slice 1: Prove the WebSocket connection works (highest risk)
Slice 2: Build real-time task updates on the proven connection
Slice 3: Add offline support and reconnection
```

Slice 1 が失敗すれば、Slice 2 と 3 に投資する前に分かります。

## 実装ルール

### Rule 0: まず単純さ

コードを書く前に、「動く最も単純な方法は何か」を自問します。

書いた後は、次を確認します。
- もっと少ない行数でできないか
- この抽象化は複雑さに見合っているか
- staff engineer が見て「なぜ素直にやらなかったのか」と言わないか
- 仮想の将来要件ではなく、今の task に対して作っているか

```
SIMPLICITY CHECK:
✗ Generic EventBus with middleware pipeline for one notification
✓ Simple function call

✗ Abstract factory pattern for two similar components
✓ Two straightforward components with shared utilities

✗ Config-driven form builder for three forms
✓ Three form components
```

似た 3 行のコードは、早すぎる抽象化より良いです。まず素直で明らかに正しい版を作ってください。最適化はテストで正しさが証明された後です。

### Rule 0.5: scope 規律

task に必要なものだけを触ります。

Do NOT:
- "Clean up" code adjacent to your change
- Refactor imports in files you're not modifying
- Remove comments you don't fully understand
- Add features not in the spec because they "seem useful"
- Modernize syntax in files you're only reading

task 範囲外で改善点を見つけても、記録するだけで直さないでください。

```
NOTICED BUT NOT TOUCHING:
- src/utils/format.ts has an unused import (unrelated to this task)
- The auth middleware could use better error messages (separate task)
→ Want me to create tasks for these?
```

### Rule 1: 一度に 1 つ

各 increment では論理的に 1 つだけ変えます。関心事を混ぜないでください。

**Bad:** One commit that adds a new component, refactors an existing one, and updates the build config.

**Good:** Three separate commits — one for each change.

### Rule 2: 常にコンパイル可能に保つ

各 increment の後、project は build でき、既存テストが通る状態でなければなりません。slice の間に codebase を壊れた状態で放置しないでください。

### Rule 3: 未完成機能には feature flag を使う

ユーザーにはまだ出せないが increment を merge したいとき:

```typescript
// 作業中機能の feature flag
const ENABLE_TASK_SHARING = process.env.FEATURE_TASK_SHARING === 'true';

if (ENABLE_TASK_SHARING) {
  // 新しい共有 UI
}
```

これにより、未完成の作業を表に出さずに小さな increment を main branch へ merge できます。

### Rule 4: 安全なデフォルト

新しいコードは安全で保守的な挙動をデフォルトにします。

```typescript
// 安全: デフォルトでは無効、opt-in
export function createTask(data: TaskInput, options?: { notify?: boolean }) {
  const shouldNotify = options?.notify ?? false;
  // ...
}
```

### Rule 5: rollback しやすくする

各 increment は独立に revert できる状態にします。

- Additive changes (new files, new functions) are easy to revert
- Modifications to existing code should be minimal and focused
- Database migrations should have corresponding rollback migrations
- Avoid deleting something in one commit and replacing it in the same commit — separate them

## agent と進めるとき

agent に段階実装を指示するときは次のように書きます。

```
"Let's implement Task 3 from the plan.

Start with just the database schema change and the API endpoint.
Don't touch the UI yet — we'll do that in the next increment.

After implementing, run `npm test` and `npm run build` to verify
nothing is broken."
```

各 increment で何が scope に入り、何が入らないかを明示してください。

## increment チェックリスト

各 increment 後に次を確認します。

- [ ] The change does one thing and does it completely
- [ ] All existing tests still pass (`npm test`)
- [ ] The build succeeds (`npm run build`)
- [ ] Type checking passes (`npx tsc --noEmit`)
- [ ] Linting passes (`npm run lint`)
- [ ] The new functionality works as expected
- [ ] The change is committed with a descriptive message

## よくある正当化

| Rationalization | Reality |
|---|---|
| 「最後にまとめてテストする」 | bug は累積します。Slice 1 の bug が Slice 2-5 をすべて誤らせます。各 slice でテストしてください。 |
| 「一気にやるほうが速い」 | 500 行変えてから壊れるまでは速く感じますが、どこが原因か分からなくなります。 |
| 「この変更は小さすぎて別 commit にするほどではない」 | 小さい commit は無料です。大きい commit は bug を隠し、rollback を痛くします。 |
| 「feature flag は後で足せばいい」 | 機能が未完成なら user-visible であってはいけません。今入れてください。 |
| 「この refactor はついでに含めても小さい」 | feature と refactor を混ぜると、どちらも review と debug が難しくなります。分けてください。 |

## レッドフラグ

- テストを回さずに 100 行超のコードを書いている
- 1 つの increment に無関係な変更が複数入っている
- 「これもついでに」で scope が膨らんでいる
- 速さのために test / verify step を飛ばしている
- increment の途中で build や test が壊れている
- 大きな未コミット変更が積み上がっている
- 3 つ目の use case を待たずに抽象化を作っている
- 「ついでに」で task 外のファイルを触っている
- 1 回限りの操作のために新しい utility file を作っている

## 検証

task の全 increment を終えたら、次を確認します。

- [ ] 各 increment が個別にテストされ、commit されている
- [ ] フル test suite が通っている
- [ ] build が clean である
- [ ] 機能が仕様通りに end-to-end で動く
- [ ] 未コミット変更が残っていない
