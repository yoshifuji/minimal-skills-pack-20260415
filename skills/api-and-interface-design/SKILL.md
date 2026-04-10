---
name: api-and-interface-design
description: 安定した API とインターフェース設計を導くスキルです。API、モジュール境界、または公開インターフェースを設計するときに使ってください。REST / GraphQL エンドポイントの作成、モジュール間の型契約の定義、フロントエンドとバックエンドの境界設定で使います。
---

# API とインターフェース設計

## 概要

誤用しにくく、安定していて、十分に文書化されたインターフェースを設計します。良いインターフェースは、正しい使い方を簡単にし、間違った使い方を難しくします。これは REST API、GraphQL スキーマ、モジュール境界、コンポーネント props、そしてコード同士がやり取りするあらゆる面に当てはまります。

## 使う場面

- 新しい API エンドポイントを設計するとき
- モジュール境界やチーム間の契約を定義するとき
- コンポーネントの prop interface を作るとき
- API の形を決めるデータベーススキーマを設計するとき
- 既存の公開インターフェースを変更するとき

## 基本原則

### ハイラムの法則

> API の利用者が十分多ければ、契約で何を約束しているかに関係なく、システムの観測可能な振る舞いはすべて、誰かに依存される。

つまり、文書化されていない癖、エラーメッセージ文言、タイミング、順序を含むすべての公開挙動は、利用者が依存した瞬間に事実上の契約になります。設計上の含意は次の通りです。

- **何を露出するかを意図的に決める。** 観測可能な挙動はすべて将来の約束になり得ます。
- **実装詳細を漏らさない。** 利用者に観測できるなら、そこに依存されます。
- **設計時点で廃止計画を持つ。** 利用者が依存しているものを安全に外す方法は `deprecation-and-migration` を参照してください。
- **テストだけでは足りない。** 契約テストが完璧でも、文書化されていない挙動に依存する実ユーザーは壊れます。

### 単一バージョン原則

同じ依存関係や API の複数バージョンを利用者に選ばせないでください。異なる利用者が同じものの異なるバージョンを必要とすると、ダイヤモンド依存問題が起きます。同時に存在するのは 1 バージョンだけ、という前提で設計し、fork ではなく拡張で進めます。

### 1. まず契約を定義する

実装より先にインターフェースを定義します。契約こそが仕様であり、実装はその後です。

```typescript
// 先に契約を定義する
interface TaskAPI {
  // タスクを作成し、サーバー生成フィールドを含む作成済みタスクを返す
  createTask(input: CreateTaskInput): Promise<Task>;

  // フィルターに一致するタスク一覧をページネーション付きで返す
  listTasks(params: ListTasksParams): Promise<PaginatedResult<Task>>;

  // 単一タスクを返す。存在しなければ NotFoundError を投げる
  getTask(id: string): Promise<Task>;

  // 部分更新。渡されたフィールドだけを変更する
  updateTask(id: string, input: UpdateTaskInput): Promise<Task>;

  // 冪等な削除。すでに削除済みでも成功する
  deleteTask(id: string): Promise<void>;
}
```

### 2. 一貫したエラー意味論

1 つのエラー戦略を選び、全体で統一します。

```typescript
// REST: HTTP ステータスコード + 構造化エラーボディ
// すべてのエラーレスポンスを同じ形にする
interface APIError {
  error: {
    code: string;        // 機械可読: "VALIDATION_ERROR"
    message: string;     // 人間向け: "メールアドレスは必須です"
    details?: unknown;   // 必要なら追加コンテキスト
  };
}

// ステータスコード対応表
// 400 → クライアントの入力が不正
// 401 → 未認証
// 403 → 認証済みだが権限なし
// 404 → リソースが見つからない
// 409 → 競合（重複、バージョン不一致）
// 422 → バリデーション失敗（意味的に不正）
// 500 → サーバーエラー（内部情報は絶対に出さない）
```

**パターンを混在させない。** あるエンドポイントは throw、別のものは null、さらに別のものは `{ error }` を返すようでは、利用側が挙動を予測できません。

### 3. 境界でバリデーションする

内部コードは信頼し、外部入力が入るシステム境界でバリデーションします。

```typescript
// API 境界で検証する
app.post('/api/tasks', async (req, res) => {
  const result = CreateTaskSchema.safeParse(req.body);
  if (!result.success) {
    return res.status(422).json({
      error: {
        code: 'VALIDATION_ERROR',
        message: 'タスクデータが不正です',
        details: result.error.flatten(),
      },
    });
  }

  // 検証後は、内部コードでその型を信頼する
  const task = await taskService.create(result.data);
  return res.status(201).json(task);
});
```

バリデーションが必要な場所:
- API ルートハンドラ（ユーザー入力）
- フォーム送信ハンドラ（ユーザー入力）
- 外部サービス応答の解析（サードパーティーデータは **常に非信頼** として扱う）
- 環境変数読み込み（設定）

> **サードパーティー API の応答は非信頼データです。** ロジック、描画、意思決定に使う前に、形と内容を検証してください。侵害された外部サービスや壊れたサービスは、予期しない型、悪意ある内容、命令文のような文字列を返すことがあります。

バリデーションすべきでない場所:
- すでに型契約を共有している内部関数間
- すでに検証済みコードから呼ばれるユーティリティ関数
- 自分のデータベースから直前に読んだデータ

### 4. 変更より追加を優先する

既存利用者を壊さずにインターフェースを拡張します。

```typescript
// 良い例: 任意フィールドを追加する
interface CreateTaskInput {
  title: string;
  description?: string;
  priority?: 'low' | 'medium' | 'high';  // 後から追加、任意
  labels?: string[];                     // 後から追加、任意
}

// 悪い例: 既存フィールドの型変更や削除
interface CreateTaskInput {
  title: string;
  // description: string;  // 削除は既存利用者を壊す
  priority: number;       // string から変更すると既存利用者を壊す
}
```

### 5. 予測可能な命名

| Pattern | Convention | Example |
|---------|-----------|---------|
| REST endpoints | 複数名詞、動詞なし | `GET /api/tasks`, `POST /api/tasks` |
| Query params | camelCase | `?sortBy=createdAt&pageSize=20` |
| Response fields | camelCase | `{ createdAt, updatedAt, taskId }` |
| Boolean fields | `is` / `has` / `can` 接頭辞 | `isComplete`, `hasAttachments` |
| Enum values | UPPER_SNAKE | `"IN_PROGRESS"`, `"COMPLETED"` |

## REST API パターン

### リソース設計

```
GET    /api/tasks              → List tasks (with query params for filtering)
POST   /api/tasks              → Create a task
GET    /api/tasks/:id          → Get a single task
PATCH  /api/tasks/:id          → Update a task (partial)
DELETE /api/tasks/:id          → Delete a task

GET    /api/tasks/:id/comments → List comments for a task (sub-resource)
POST   /api/tasks/:id/comments → Add a comment to a task
```

### ページネーション

一覧系エンドポイントはページネーションします。

```typescript
// リクエスト
GET /api/tasks?page=1&pageSize=20&sortBy=createdAt&sortOrder=desc

// レスポンス
{
  "data": [...],
  "pagination": {
    "page": 1,
    "pageSize": 20,
    "totalItems": 142,
    "totalPages": 8
  }
}
```

### フィルタリング

フィルターには query parameter を使います。

```
GET /api/tasks?status=in_progress&assignee=user123&createdAfter=2025-01-01
```

### 部分更新（PATCH）

部分オブジェクトを受け取り、渡されたものだけ更新します。

```typescript
// title だけ変更し、他は保持する
PATCH /api/tasks/123
{ "title": "Updated title" }
```

## TypeScript インターフェースパターン

### バリアントには discriminated union を使う

```typescript
// 良い例: 各バリアントが明示的
type TaskStatus =
  | { type: 'pending' }
  | { type: 'in_progress'; assignee: string; startedAt: Date }
  | { type: 'completed'; completedAt: Date; completedBy: string }
  | { type: 'cancelled'; reason: string; cancelledAt: Date };

// 利用側で型が絞り込まれる
function getStatusLabel(status: TaskStatus): string {
  switch (status.type) {
    case 'pending': return '保留中';
    case 'in_progress': return `進行中 (${status.assignee})`;
    case 'completed': return `${status.completedAt} に完了`;
    case 'cancelled': return `キャンセル: ${status.reason}`;
  }
}
```

### 入力と出力を分ける

```typescript
// 入力: 呼び出し側が渡すもの
interface CreateTaskInput {
  title: string;
  description?: string;
}

// 出力: システムが返すもの（サーバー生成フィールドを含む）
interface Task {
  id: string;
  title: string;
  description: string | null;
  createdAt: Date;
  updatedAt: Date;
  createdBy: string;
}
```

### ID には branded type を使う

```typescript
type TaskId = string & { readonly __brand: 'TaskId' };
type UserId = string & { readonly __brand: 'UserId' };

// UserId を TaskId の代わりに誤って渡すことを防ぐ
function getTask(id: TaskId): Promise<Task> { ... }
```

## よくある正当化

| Rationalization | Reality |
|---|---|
| 「API の文書は後で書けばいい」 | 型そのものが文書です。先に定義してください。 |
| 「今はページネーション不要」 | 100 件を超えた瞬間に必要になります。最初から入れてください。 |
| 「PATCH は複雑だから PUT でいい」 | PUT は毎回フルオブジェクトが必要です。利用者が本当に欲しいのは PATCH です。 |
| 「必要になったら API を versioning すればいい」 | versioning なしの破壊的変更は利用者を壊します。最初から拡張前提で設計してください。 |
| 「その undocumented behavior は誰も使っていない」 | ハイラムの法則では、観測可能なら誰かが依存します。公開挙動はすべて約束として扱ってください。 |
| 「2 バージョン維持すればいい」 | 複数バージョンは保守コストを増やし、ダイヤモンド依存問題を生みます。単一バージョン原則を優先してください。 |
| 「内部 API に契約は不要」 | 内部利用者も利用者です。契約が疎結合と並行開発を支えます。 |

## レッドフラグ

- 条件によって返却 shape が変わるエンドポイント
- エンドポイントごとに異なるエラーフォーマット
- 境界ではなく内部コード全体に散らばったバリデーション
- 既存フィールドへの破壊的変更（型変更、削除）
- ページネーションのない一覧系エンドポイント
- REST URL に動詞が入っている (`/api/createTask`, `/api/getUsers`)
- バリデーションやサニタイズなしで使われるサードパーティー API 応答

## 検証

API を設計したら、次を確認します。

- [ ] すべてのエンドポイントに型付きの入出力 schema がある
- [ ] エラーレスポンスが単一の一貫した形式に従っている
- [ ] バリデーションはシステム境界でのみ行っている
- [ ] 一覧系エンドポイントがページネーションをサポートしている
- [ ] 新しいフィールドは追加的かつ任意である（後方互換）
- [ ] すべてのエンドポイントで命名規約が一貫している
- [ ] API ドキュメントまたは型定義が実装と一緒にコミットされている
