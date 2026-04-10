# Skills 分類表

## 判定ラベル

- `Keep`: 名称と責務を維持する
- `Generalize`: 名称は維持しつつ、本文や description を中立化する
- `Rename`: 名称と本文を見直す
- `Split`: 汎用 skill と example overlay に分割する
- `Archive`: 履歴として残し、主導線から外す

## Skill 一覧

| 現行 skill | 現在の層 | 結合度 | 推奨 action | target layer | target name / 方針 | 補足 |
|---|---|---:|---|---|---|---|
| `api-and-interface-design` | core | 低 | Keep | core | 維持 | すでに汎用。description も安定している |
| `checkout-booking-and-destination-charges` | pattern | 高 | Rename | pattern | `stripe-single-seller-checkout` | `booking`, `tutor`, `student` を中立化する |
| `debugging-and-error-recovery` | core | 低 | Keep | core | 維持 | アプリ固有要素は薄い |
| `deployment-for-nextjs-supabase-stripe` | stack | 中 | Generalize | stack | 維持または軽微改名 | app URL や flow 例を generic にする |
| `frontend-ui-engineering` | core | 低 | Keep | core | 維持 | そのまま再利用可能 |
| `incremental-implementation` | core | 低 | Keep | core | 維持 | そのまま再利用可能 |
| `nextjs-supabase-ssr-auth` | stack | 中 | Generalize | stack | 維持 | role redirect 例を generic 化する |
| `payment-webhook-reconciliation` | pattern | 中 | Generalize | pattern | 維持 | booking 前提を `order/reservation` へ中立化する |
| `planning-and-task-breakdown` | core | 低 | Keep | core | 維持 | そのまま再利用可能 |
| `postgres-rls-and-public-read-models` | stack | 中 | Generalize | stack | 維持 | `profiles` 等の固有 table を例へ移す |
| `refunds-and-booking-cancellation` | pattern | 高 | Rename | pattern | `refunds-and-cancellation-workflows` | 予約系以外にも流用できる表現にする |
| `role-based-marketplace-pages` | pattern | 高 | Rename | pattern | `role-based-portals-and-navigation` | marketplace 専用語を外す |
| `security-and-hardening` | core | 低 | Keep | core | 維持 | そのまま再利用可能 |
| `shipping-and-launch` | core | 低 | Keep | core | 維持 | そのまま再利用可能 |
| `spec-driven-development` | core | 低 | Keep | core | 維持 | そのまま再利用可能 |
| `stripe-connect-express-onboarding` | stack | 中 | Generalize | stack | 維持 | tutor/seller の例を provider 側へ中立化する |
| `using-marketplace-agent-skills` | example meta | 高 | Split | example + pattern meta | `example-tutor-marketplace` と汎用 meta へ分割 | pack 全体の入口が app 固有になっている |

## 優先度つき skill 対応順

### 最優先

1. `using-marketplace-agent-skills`
2. `role-based-marketplace-pages`
3. `checkout-booking-and-destination-charges`
4. `refunds-and-booking-cancellation`

理由:

- pack 全体のアプリ依存を最も強く作っている
- trigger しやすさと再利用性に直結する

### 次点

1. `payment-webhook-reconciliation`
2. `nextjs-supabase-ssr-auth`
3. `postgres-rls-and-public-read-models`
4. `stripe-connect-express-onboarding`
5. `deployment-for-nextjs-supabase-stripe`

理由:

- 技術依存はあるが、語彙と例を整理すれば汎用 pack に残せる

### 後回しでよい

- `api-and-interface-design`
- `frontend-ui-engineering`
- `incremental-implementation`
- `debugging-and-error-recovery`
- `security-and-hardening`
- `shipping-and-launch`
- `planning-and-task-breakdown`
- `spec-driven-development`

理由:

- すでに generic として機能している

## References 分類

| 現行ファイル | 現在の性質 | 推奨 action | target 配置 | 補足 |
|---|---|---|---|---|
| `references/アクセシビリティチェックリスト.md` | generic | Keep | `references/generic/` | そのまま汎用資料にできる |
| `references/セキュリティチェックリスト.md` | generic | Keep | `references/generic/` | そのまま汎用資料にできる |
| `references/パフォーマンスチェックリスト.md` | generic | Keep | `references/generic/` | 参照している skill 名は後で整える |
| `references/Stripeイベントマトリクス.md` | pattern + example | Generalize | `references/generic/stripe/` または pattern skill 配下 | `booking` 前提を弱めるなら流用可能 |
| `references/環境変数とシークレット一覧.md` | stack + example | Generalize | `references/generic/nextjs-supabase-stripe/` | 現行変数名は stack 固有だが再利用価値が高い |
| `references/データモデル概要.md` | example | Move | `references/examples/tutor-marketplace/` | 講師マーケットプレイス専用 |
| `references/デザインブリーフ.md` | example | Move | `references/examples/tutor-marketplace/` | example 資料として保持 |
| `references/実行成果物チェックリスト.md` | example | Move | `references/examples/tutor-marketplace/` | 汎用成果物基準とは分ける |
| `references/手動QAフロー.md` | example | Move | `references/examples/tutor-marketplace/` | 固定フローなので example 側へ移す |

## Docs / Specs 分類

| 現行ファイル | 現在の性質 | 推奨 action | target 配置 | 補足 |
|---|---|---|---|---|
| `README.md` | example 寄り入口 | Rewrite | repo 入口 | 汎用 pack と example pack の両方を案内する構成へ |
| `はじめに.md` | example | Move | `docs/examples/tutor-marketplace/` | generic の入口とは分ける |
| `docs/講師マーケットプレイス実行手順書.md` | example | Move | `docs/examples/tutor-marketplace/` | example runbook として保持 |
| `docs/SupabaseとStripeの設定手順書.md` | stack + example | Split | generic docs + example docs | 手順と app 固有注意点を分離する |
| `docs/Vercelデプロイ手順書.md` | stack | Generalize | `docs/generic/` | stack docs として汎用化できる |
| `docs/最小パック選定メモ.md` | 履歴メモ | Archive | `docs/archive/` | 将来の入口にはしない |
| `スキルズプロジェクト概要資料.md` | 説明資料 | Rewrite or Archive | `docs/archive/` または再生成 | 現状は tutor marketplace pack 前提 |
| `specs/講師マーケットプレイス再現仕様プロンプト.md` | example spec | Move | `specs/examples/tutor-marketplace/` | example 入力として維持 |

## 先に決めるべき open questions

- `using-marketplace-agent-skills` の後継は 1 つの generic meta skill にするか、example overlay のみにするか
- rename 対象 skill に prefix を付けるか、自然名だけにするか
- `Stripeイベントマトリクス.md` を generic reference に昇格させるか、example へ残すか
- `deployment-for-nextjs-supabase-stripe` を stack skill として維持するか、より広い deploy skill に吸収するか

## 初手でやる変更

1. `using-marketplace-agent-skills` の責務を split する
2. pattern skill 3 本を generic 名へ寄せる
3. example 固有 references を別ディレクトリへ移す
4. `README.md` を generic / example 二層の入口に書き直す
