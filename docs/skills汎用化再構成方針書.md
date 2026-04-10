# Skills 汎用化再構成方針書

## 目的

このリポジトリの skill 群を、`tutor-marketplace` 再現用の最小パックから、より広いアプリ開発で再利用できる skill pack へ再構成する。

この再構成では、次の 2 点を同時に満たすことを目標にする。

- 汎用 skill は、特定アプリを前提にせず trigger しやすいこと
- 既存の講師マーケットプレイス再現フローは、example overlay として維持できること

## 背景

現状の pack は、汎用 skill と講師マーケットプレイス固有の実装知識が混在している。

特に次の問題がある。

- `using-marketplace-agent-skills` が pack 全体の入口になっており、利用導線がアプリ固有
- 一部 skill の `description` と本文に `tutor`, `student`, `booking`, `offering` などのドメイン語が残っている
- `references/` と `docs/` に generic なチェックリストと example 固有資料が混在している
- skill の責務と層が混ざっており、core workflow / stack / product pattern / example overlay の境界が曖昧

## 成功条件

- 汎用 skill の `name` と `description` だけを見ても、講師マーケットプレイス専用に見えない
- 別種のアプリでも、適切な skill を trigger しやすい
- example 固有の知識は overlay と reference に局所化されている
- 現行の tutor marketplace は、example skill と example docs を使って引き続き再現できる
- 段階移行中も、既存利用者が完全に壊れない

## 非ゴール

- すべての skill 名を一括でリネームすること
- すべての docs を一度に書き直すこと
- 現時点で actual skill implementation を全面改修すること

今回は、再構成の方針、分類、移行順、検証基準を固めることを目的とする。

## 再構成の原則

### 1. skill 本体は手順と判断基準に寄せる

`SKILL.md` には、再利用されるべき workflow、設計原則、レッドフラグ、検証条件を残す。

アプリ固有の内容は次へ逃がす。

- specific route 名
- specific table 名
- 固定 URL
- 固定 status 名
- 画面一覧
- アプリ固有 QA フロー

### 2. example を skill 本体から分離する

具体例は必要だが、base skill の本文に埋め込まず、reference や example overlay に置く。

### 3. 語彙を中立化する

base skill では、次のような中立語に置き換える。

- `tutor` / `student` → `provider` / `customer` または `role A` / `role B`
- `booking` → `reservation` / `order` / `transaction record`
- `offering` → `listing` / `service option` / `resource`
- `tutor dashboard` → `provider portal`

### 4. 層を明示する

skill を次の 4 層に分けて考える。

- `core`: 実装の進め方、設計、品質、検証
- `stack`: 特定技術スタックの実装パターン
- `pattern`: マーケットプレイス、予約、返金、ロール別 UI などの業務パターン
- `example`: 具体アプリ向け overlay、runbook、入力 spec

### 5. 互換性を段階的に外す

既存 skill 名がすでに利用されている可能性があるため、いきなり削除しない。

移行期は次のどちらかで対応する。

- 既存 skill を薄い wrapper として残し、新 skill へ案内する
- `description` に移行先を明記して段階的に廃止する

## 目標の構成

### Layer 1: Core

用途:

- 要件整理
- task 分解
- 段階実装
- API 設計
- UI 品質
- デバッグ
- セキュリティ
- 出荷判断

対象:

- `spec-driven-development`
- `planning-and-task-breakdown`
- `incremental-implementation`
- `api-and-interface-design`
- `frontend-ui-engineering`
- `debugging-and-error-recovery`
- `security-and-hardening`
- `shipping-and-launch`

### Layer 2: Stack

用途:

- Next.js + Supabase Auth
- Postgres + RLS
- Stripe Connect Express
- Next.js + Supabase + Stripe の deploy

対象:

- `nextjs-supabase-ssr-auth`
- `postgres-rls-and-public-read-models`
- `stripe-connect-express-onboarding`
- `deployment-for-nextjs-supabase-stripe`

### Layer 3: Pattern

用途:

- ロール別 portal / navigation
- 単一 seller 向け Stripe Checkout
- webhook 整合
- refund / cancellation workflow

対象:

- `role-based-marketplace-pages`
- `checkout-booking-and-destination-charges`
- `payment-webhook-reconciliation`
- `refunds-and-booking-cancellation`

### Layer 4: Example

用途:

- 講師マーケットプレイス再現の入口
- 固定された順序、画面、状態遷移、QA

対象:

- `using-marketplace-agent-skills`
- `specs/講師マーケットプレイス再現仕様プロンプト.md`
- `docs/講師マーケットプレイス実行手順書.md`
- example 向け references 一式

## 命名方針

### 維持するもの

すでに generic で trigger 品質の高い skill は、無理に rename しない。

例:

- `spec-driven-development`
- `planning-and-task-breakdown`
- `api-and-interface-design`
- `security-and-hardening`

### リネームするもの

現在の名前にアプリ固有の文脈が強く残る skill は rename する。

候補:

- `role-based-marketplace-pages` → `role-based-portals-and-navigation`
- `checkout-booking-and-destination-charges` → `stripe-single-seller-checkout`
- `refunds-and-booking-cancellation` → `refunds-and-cancellation-workflows`

### 分割するもの

meta skill と example overlay は分離する。

候補:

- `using-marketplace-agent-skills`
  - 汎用側: `marketplace-foundation-workflow`
  - example 側: `example-tutor-marketplace`

### prefix の扱い

全skillへ機械的に prefix を付ける必要はない。

ただし、新規作成または大幅 rename を行う skill では、scope を明確にするため prefix を許可する。

例:

- `pattern-role-based-portals-and-navigation`
- `example-tutor-marketplace`

## ディレクトリ方針

skill registry の単純さを保つため、`skills/` 直下に 1 skill 1 directory を維持する。

```text
skills/
  spec-driven-development/
  planning-and-task-breakdown/
  nextjs-supabase-ssr-auth/
  pattern-role-based-portals-and-navigation/
  example-tutor-marketplace/

references/
  generic/
  examples/
    tutor-marketplace/

docs/
  generic/
  examples/
    tutor-marketplace/
```

## 実行計画

## Phase 1: 棚卸しと target taxonomy 固定

### Task 1: 現行 skill / references / docs の分類を確定する

**Description:**  
各 skill と補助資料を、`core / stack / pattern / example / archive` に分類し、どこまで generic 化できるかを判断する。

**Acceptance criteria:**

- [ ] 全 skill が target layer を持っている
- [ ] 全 skill に対して `keep / generalize / rename / split / archive` の action が定義されている
- [ ] references と docs も generic / example に分類されている

**Verification:**

- [ ] `docs/skills分類表.md` に一覧化されている
- [ ] high coupling 項目が明示されている

**Dependencies:** None

**Estimated scope:** Small

### Task 2: 命名ルールと migration 方針を固定する

**Description:**  
rename が必要な skill のみ対象にし、既存名を残すものと wrapper を置くものを分ける。

**Acceptance criteria:**

- [ ] rename 対象が明示されている
- [ ] 既存名を残す理由が説明されている
- [ ] compatibility 期間の扱いが定義されている

**Verification:**

- [ ] 方針書に命名セクションがある
- [ ] 互換性ポリシーが書かれている

**Dependencies:** Task 1

**Estimated scope:** Small

## Checkpoint: Taxonomy 固定

- [ ] 分類表と方針書が揃っている
- [ ] rename 対象と keep 対象の境界が説明できる
- [ ] example と generic の分離ルールが合意できる

## Phase 2: 汎用 skill 本体の中立化

### Task 3: stack / pattern skill からドメイン語を抜く

**Description:**  
`tutor`, `student`, `booking`, `offering` などの語を中立化し、本文を「再利用できる判断基準」中心へ編集する。

**Acceptance criteria:**

- [ ] stack / pattern skill の `description` が generic になっている
- [ ] 本文の主要セクションから app 固有 nouns が除去されている
- [ ] 具体例は reference または example overlay 側へ移されている

**Verification:**

- [ ] `SKILL.md` の冒頭だけ読んでも講師マーケットプレイス専用に見えない
- [ ] 既存のレッドフラグと検証項目が generic に言い換えられている

**Dependencies:** Task 2

**Estimated scope:** Medium

### Task 4: pattern skill を必要に応じて rename / split する

**Description:**  
現名称が強く app を想起させるものは rename し、必要なら wrapper skill を残す。

**Acceptance criteria:**

- [ ] rename 対象 skill の target name が決まっている
- [ ] split する skill は責務境界が明記されている
- [ ] wrapper を残す場合は移行先が明記されている

**Verification:**

- [ ] 新旧 skill の対応表がある
- [ ] 旧 skill 単体で読んでも移行先が分かる

**Dependencies:** Task 3

**Estimated scope:** Medium

## Checkpoint: Base skill 中立化

- [ ] generic skill 群が講師マーケットプレイス前提でなくなっている
- [ ] pattern skill が別アプリでも流用しやすい名前と本文になっている

## Phase 3: references / docs の分離

### Task 5: references を `generic` と `examples` に分ける

**Description:**  
generic checklist と example 固有 matrix / QA / data model を別ディレクトリへ整理する。

**Acceptance criteria:**

- [ ] 汎用チェックリストが `references/generic/` 側へまとまっている
- [ ] example 固有資料が `references/examples/tutor-marketplace/` 側へまとまっている
- [ ] 各 skill から参照先が追える

**Verification:**

- [ ] generic skill が example reference を直接必須にしていない
- [ ] example skill から必要な reference に辿れる

**Dependencies:** Task 1

**Estimated scope:** Medium

### Task 6: docs を generic runbook と example runbook に分ける

**Description:**  
現在の実行手順書を example 側へ寄せ、汎用 pack の導入と使い方は別文書に切り出す。

**Acceptance criteria:**

- [ ] generic の導入文書が存在する
- [ ] 講師マーケットプレイス実行手順書は example docs として位置づけ直されている
- [ ] README からの導線が整理されている

**Verification:**

- [ ] 初見の利用者が generic pack と example pack の違いを把握できる
- [ ] generic docs が特定アプリの画面一覧に依存しない

**Dependencies:** Task 5

**Estimated scope:** Medium

## Checkpoint: 情報配置の分離

- [ ] skill 本体、generic references、example references の役割が分かれている
- [ ] docs の入口が generic と example で分離されている

## Phase 4: 互換性と移行導線

### Task 7: deprecated skill の wrapper / 移行案内を作る

**Description:**  
rename または split の対象になった旧 skill に、移行先の明示を追加する。

**Acceptance criteria:**

- [ ] 旧 skill から移行先が分かる
- [ ] 廃止予定の説明がある
- [ ] 参照先 skill が 1 ステップで開ける

**Verification:**

- [ ] 旧 skill 名で task を始めても、新構成へ誘導できる

**Dependencies:** Task 4

**Estimated scope:** Small

## Phase 5: 検証

### Task 8: 3 つの想定シナリオで trigger と導線を検証する

**Description:**  
次の 3 シナリオで、skill trigger と doc 導線が破綻しないか確認する。

- 非マーケットプレイス SaaS
- 決済あり二面マーケットプレイス
- 既存の tutor marketplace

**Acceptance criteria:**

- [ ] generic app で example skill を読まなくても進められる
- [ ] marketplace app で pattern skill を選びやすい
- [ ] tutor marketplace では既存フローを再現できる

**Verification:**

- [ ] 代表 prompt を 3 本作り、適切な skill が選ばれるか確認する
- [ ] skill trigger に必要な語彙が `description` に残っている

**Dependencies:** Task 7

**Estimated scope:** Medium

## 優先順位

最初の 1 スプリントでは、次を優先する。

1. 分類表を確定する
2. `using-marketplace-agent-skills` の分割方針を固める
3. `role-based-marketplace-pages` / `checkout-booking-and-destination-charges` / `refunds-and-booking-cancellation` の rename 方針を固める
4. references を generic / example に切り分ける

## リスクと対策

| Risk | Impact | Mitigation |
|------|--------|------------|
| rename しすぎて既存利用者が迷う | High | generic な skill は維持し、rename は必要箇所に限定する |
| generic 化の結果、trigger が弱くなる | High | `description` に具体的な利用場面を残しつつ語彙だけ中立化する |
| example を抜きすぎて skill が抽象化しすぎる | Medium | 具体例は削除せず reference に移す |
| references の移動で導線が壊れる | Medium | SKILL.md から参照先を明示し、README も最後に整理する |
| stack skill と pattern skill の責務がまた混ざる | Medium | `技術依存の実装` と `業務フローの設計` を分けてレビューする |

## 次の実装対象

この方針書の次に着手すべき変更は次の通り。

1. `docs/skills分類表.md` を基準に keep / rename / split を確定する
2. `using-marketplace-agent-skills` の後継構成を作る
3. pattern skill を 3 本から順に generic 化する
4. references と docs を generic / example に再配置する
