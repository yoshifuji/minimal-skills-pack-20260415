---
name: spec-driven-development
description: コーディング前に spec を作るスキルです。新しい project、feature、大きな変更を始めるが、まだ specification がないときに使ってください。要件が不明確、曖昧、または漠然としたアイデアしかないときにも使います。
---

# Spec-Driven Development

## 概要

コードを書く前に構造化された specification を書きます。spec はあなたと human engineer の共有された source of truth であり、何を作るのか、なぜ作るのか、完了をどう判断するのかを定義します。spec のないコードは推測です。

## 使う場面

- 新しい project や feature を始めるとき
- 要件が曖昧または不足しているとき
- 変更が複数ファイルや複数 module にまたがるとき
- アーキテクチャ判断をしようとしているとき
- 実装に 30 分以上かかる task のとき

**使わない場面:** 1 行修正、typo 修正、または要件が明確で自己完結している変更。

## ゲート付き workflow

spec-driven development は 4 つの phase で進みます。現在の phase が検証されるまで、次へ進んではいけません。

```
SPECIFY ──→ PLAN ──→ TASKS ──→ IMPLEMENT
   │          │        │          │
   ▼          ▼        ▼          ▼
 Human      Human    Human      Human
 reviews    reviews  reviews    reviews
```

### Phase 1: Specify

高レベルの vision から始めます。要件が具体化するまで、human に確認質問を投げます。

**前提をすぐ表に出します。** spec 本文を書く前に、何を前提としているかを列挙してください。

```
ASSUMPTIONS I'M MAKING:
1. This is a web application (not native mobile)
2. Authentication uses session-based cookies (not JWT)
3. The database is PostgreSQL (based on existing Prisma schema)
4. We're targeting modern browsers only (no IE11)
→ 違っていたら今修正してください。これで先に進みます。
```

曖昧な要件を黙って補完してはいけません。spec の目的は、コードを書く *前に* 誤解を表面化することです。前提は最も危険な誤解の形です。

**次の 6 つの中核領域を含む spec document を書きます。**

1. **Objective** 何を、なぜ作るのか。ユーザーは誰か。成功とは何か。

2. **Commands** ツール名だけでなく、flag 付きの実行可能コマンド全文。
   ```
   Build: npm run build
   Test: npm test -- --coverage
   Lint: npm run lint --fix
   Dev: npm run dev
   ```

3. **Project Structure** source code、test、docs をどこに置くか。
   ```
   src/           → Application source code
   src/components → React components
   src/lib        → Shared utilities
   tests/         → Unit and integration tests
   e2e/           → End-to-end tests
   docs/          → Documentation
   ```

4. **Code Style** 3 段落の説明より、実際の code snippet 1 つのほうが強いです。命名規約、整形ルール、良い出力例を含めてください。

5. **Testing Strategy** どの framework を使うか、test はどこに置くか、coverage 期待値、何をどの test level で担保するか。

6. **Boundaries** 3 階層で定義します。
   - **Always do:** commit 前に test を回す、命名規約に従う、入力を検証する
   - **Ask first:** database schema 変更、依存追加、CI config 変更
   - **Never do:** secret を commit する、vendor directory を触る、承認なしに failing test を消す

**spec テンプレート:**

```markdown
# Spec: [Project/Feature Name]

## Objective
[何を、なぜ作るか。user story または acceptance criteria。]

## Tech Stack
[framework、language、主要依存と version]

## Commands
[build、test、lint、dev のフルコマンド]

## Project Structure
[directory layout と説明]

## Code Style
[サンプル snippet + 重要な規約]

## Testing Strategy
[framework、test 配置、coverage 要件、test level]

## Boundaries
- Always: [...]
- Ask first: [...]
- Never: [...]

## Success Criteria
[完了をどう判断するか。具体的でテスト可能な条件]

## Open Questions
[human の入力が必要な未解決事項]
```

**指示を success criteria に言い換える。** 曖昧な要件を受け取ったら、具体条件へ変換します。

```
REQUIREMENT: "ダッシュボードを速くして"

REFRAMED SUCCESS CRITERIA:
- Dashboard LCP < 2.5s on 4G connection
- Initial data load completes in < 500ms
- No layout shift during load (CLS < 0.1)
→ この目標設定で合っていますか？
```

これにより、「速く」の意味を推測するのではなく、明確な目標へ向けて反復、retry、問題解決できます。

### Phase 2: Plan

検証済み spec を基に、技術的な implementation plan を作ります。

1. Identify the major components and their dependencies
2. Determine the implementation order (what must be built first)
3. Note risks and mitigation strategies
4. Identify what can be built in parallel vs. what must be sequential
5. Define verification checkpoints between phases

plan は review 可能でなければなりません。human が読んで「その方針でよい」または「X を変えて」と言える状態です。

### Phase 3: Tasks

plan を、離散的で実装可能な task に分けます。

- 各 task は 1 回の集中 session で完了できること
- 各 task に明示的な acceptance criteria があること
- 各 task に verification step（test、build、manual check）があること
- task は重要度ではなく依存順に並ぶこと
- どの task も変更ファイル数は概ね 5 個以下であること

**task テンプレート:**
```markdown
- [ ] Task: [説明]
  - Acceptance: [完了時に真であるべきこと]
  - Verify: [確認方法。test command、build、manual check]
  - Files: [触るファイル]
```

### Phase 4: Implement

`incremental-implementation` と `test-driven-development` に従って、task を 1 つずつ実行します。agent に spec 全体を流し込むのではなく、各 step で必要な spec section と source file だけを `context-engineering` で読み込んでください。

## spec を生かし続ける

spec は一度きりの成果物ではなく、生きた document です。

- **判断が変わったら更新する** data model 変更が必要と分かったら、先に spec を更新し、その後で実装する。
- **scope が変わったら更新する** 追加や削除された feature は spec に反映する。
- **spec も commit する** spec は code と並んで version control に置く。
- **PR で spec を参照する** 各 PR が実装する spec section へリンクする。

## よくある正当化

| Rationalization | Reality |
|---|---|
| 「これは簡単だから spec は要らない」 | 簡単な task に長い spec は不要でも、acceptance criteria は必要です。2 行の spec でも構いません。 |
| 「コードを書いた後で spec を書く」 | それは documentation であって specification ではありません。spec の価値はコード *前* に明確化を強制することです。 |
| 「spec は遅くするだけ」 | 15 分の spec が数時間の手戻りを防ぎます。15 分の waterfall のほうが 15 時間の debugging より安いです。 |
| 「どうせ要件は変わる」 | だからこそ spec は生きた document です。古い spec でも、spec がないよりましです。 |
| 「ユーザーは自分の欲しいものを分かっている」 | 明確な依頼にも暗黙の前提があります。spec はその前提を表面化します。 |

## レッドフラグ

- 書かれた要件なしでコードを書き始める
- 「何が done か」を明確にせずに「そのまま作り始めていい？」と聞く
- spec や task list にない feature を実装している
- 文書化せずにアーキテクチャ判断をしている
- 「何を作るかは obvious」と言って spec を飛ばす

## 検証

implementation に進む前に次を確認します。

- [ ] spec が 6 つの中核領域をすべてカバーしている
- [ ] human が spec を確認し、承認している
- [ ] success criteria が具体的かつテスト可能である
- [ ] Boundaries（Always / Ask First / Never）が定義されている
- [ ] spec が repository 内の file に保存されている
