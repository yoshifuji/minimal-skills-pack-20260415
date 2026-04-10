---
name: security-and-hardening
description: 脆弱性に対して code を harden するスキルです。ユーザー入力、認証、データ保存、外部連携を扱うときに使ってください。非信頼データを受け取る機能、user session を管理する機能、third-party service と連携する機能を作るときに使います。
---

# セキュリティと堅牢化

## 概要

web application 向けの security-first な開発実践です。すべての外部入力を hostile とみなし、すべての secret を厳重に扱い、すべての authorization check を必須として扱います。security は phase ではなく、user data、authentication、external system に触れるすべての行にかかる制約です。

## 使う場面

- ユーザー入力を受け取る機能を作るとき
- authentication や authorization を実装するとき
- 機微データを保存または送信するとき
- external API や service と連携するとき
- file upload、webhook、callback を追加するとき
- payment や PII を扱うとき

## 3 層の boundary system

### Always Do（例外なし）

- **すべての外部入力を検証する** システム境界（API route、form handler）で行う
- **すべての database query をパラメータ化する** user input を SQL に連結しない
- **出力を encode する** XSS を防ぐ（framework の auto-escaping を使い、迂回しない）
- **すべての外部通信に HTTPS を使う**
- **password を bcrypt / scrypt / argon2 で hash する** 平文保存しない
- **security header を設定する**（CSP、HSTS、X-Frame-Options、X-Content-Type-Options）
- **session には httpOnly / secure / sameSite cookie を使う**
- **毎回の release 前に `npm audit` を回す**（または同等）

### Ask First（human 承認が必要）

- 新しい authentication flow を足す、または auth logic を変える
- 新しい種類の機微データ（PII、payment info）を保存する
- 新しい external service integration を追加する
- CORS 設定を変更する
- file upload handler を追加する
- rate limiting や throttling を変更する
- 高い権限や role を付与する

### Never Do

- **secret を version control に commit しない**（API key、password、token）
- **機微データを log に出さない**（password、token、完全なカード番号）
- **client-side validation を security boundary とみなさない**
- **便宜のために security header を無効化しない**
- **user 提供データで `eval()` や `innerHTML` を使わない**
- **client から読める storage に session を保存しない**（auth token を localStorage に置かない）
- **stack trace や内部 error detail を user に見せない**

## OWASP Top 10 への対策

### 1. Injection（SQL、NoSQL、OS command）

```typescript
// 悪い例: 文字列連結による SQL injection
const query = `SELECT * FROM users WHERE id = '${userId}'`;

// 良い例: パラメータ化 query
const user = await db.query('SELECT * FROM users WHERE id = $1', [userId]);

// 良い例: パラメータ化 input を使う ORM
const user = await prisma.user.findUnique({ where: { id: userId } });
```

### 2. Broken Authentication

```typescript
// password hashing
import { hash, compare } from 'bcrypt';

const SALT_ROUNDS = 12;
const hashedPassword = await hash(plaintext, SALT_ROUNDS);
const isValid = await compare(plaintext, hashedPassword);

// session management
app.use(session({
  secret: process.env.SESSION_SECRET,  // From environment, not code
  resave: false,
  saveUninitialized: false,
  cookie: {
    httpOnly: true,     // JavaScript から触れない
    secure: true,       // HTTPS のみ
    sameSite: 'lax',    // CSRF 対策
    maxAge: 24 * 60 * 60 * 1000,  // 24 時間
  },
}));
```

### 3. Cross-Site Scripting（XSS）

```typescript
// 悪い例: user input を HTML として描画
element.innerHTML = userInput;

// 良い例: framework の auto-escaping を使う（React は標準でそうなる）
return <div>{userInput}</div>;

// どうしても HTML を描画するなら先に sanitize する
import DOMPurify from 'dompurify';
const clean = DOMPurify.sanitize(userInput);
```

### 4. Broken Access Control

```typescript
// authentication だけでなく authorization も必ず確認する
app.patch('/api/tasks/:id', authenticate, async (req, res) => {
  const task = await taskService.findById(req.params.id);

  // 認証済み user がこの resource を所有していることを確認する
  if (task.ownerId !== req.user.id) {
    return res.status(403).json({
      error: { code: 'FORBIDDEN', message: 'この task を変更する権限がありません' }
    });
  }

  // 更新を続行
  const updated = await taskService.update(req.params.id, req.body);
  return res.json(updated);
});
```

### 5. Security Misconfiguration

```typescript
// security header（Express では helmet を使う）
import helmet from 'helmet';
app.use(helmet());

// Content Security Policy
app.use(helmet.contentSecurityPolicy({
  directives: {
    defaultSrc: ["'self'"],
    scriptSrc: ["'self'"],
    styleSrc: ["'self'", "'unsafe-inline'"],  // 可能ならさらに絞る
    imgSrc: ["'self'", 'data:', 'https:'],
    connectSrc: ["'self'"],
  },
}));

// CORS: 既知の origin に制限する
app.use(cors({
  origin: process.env.ALLOWED_ORIGINS?.split(',') || 'http://localhost:3000',
  credentials: true,
}));
```

### 6. Sensitive Data Exposure

```typescript
// API response に機微 field を返さない
function sanitizeUser(user: UserRecord): PublicUser {
  const { passwordHash, resetToken, ...publicFields } = user;
  return publicFields;
}

// secret は環境変数から取る
const API_KEY = process.env.STRIPE_API_KEY;
if (!API_KEY) throw new Error('STRIPE_API_KEY not configured');
```

## 入力バリデーションのパターン

### 境界での schema validation

```typescript
import { z } from 'zod';

const CreateTaskSchema = z.object({
  title: z.string().min(1).max(200).trim(),
  description: z.string().max(2000).optional(),
  priority: z.enum(['low', 'medium', 'high']).default('medium'),
  dueDate: z.string().datetime().optional(),
});

// route handler で検証する
app.post('/api/tasks', async (req, res) => {
  const result = CreateTaskSchema.safeParse(req.body);
  if (!result.success) {
    return res.status(422).json({
      error: {
        code: 'VALIDATION_ERROR',
        message: '入力が不正です',
        details: result.error.flatten(),
      },
    });
  }
  // result.data は型付きかつ検証済み
  const task = await taskService.create(result.data);
  return res.status(201).json(task);
});
```

### file upload の安全性

```typescript
// file type と size を制限する
const ALLOWED_TYPES = ['image/jpeg', 'image/png', 'image/webp'];
const MAX_SIZE = 5 * 1024 * 1024; // 5MB

function validateUpload(file: UploadedFile) {
  if (!ALLOWED_TYPES.includes(file.mimetype)) {
    throw new ValidationError('許可されていないファイル形式です');
  }
  if (file.size > MAX_SIZE) {
    throw new ValidationError('ファイルサイズが大きすぎます（最大 5MB）');
  }
  // 拡張子は信頼しない。重要なら magic bytes も確認する
}
```

## `npm audit` 結果の triage

すべての audit 結果が即時対応を要するわけではありません。次の判断木を使います。

```
`npm audit` が脆弱性を報告した
├── Severity: critical または high
│   ├── 脆弱 code path は app から到達可能か?
│   │   ├── YES --> 直ちに修正する（update、patch、置換）
│   │   └── NO（dev-only 依存、未使用 code path） --> 早めに直すが blocker ではない
│   └── 修正は利用可能か?
│       ├── YES --> 修正版へ更新する
│       └── NO --> workaround を探す、dependency 置換を検討する、または review 日付きで allowlist に入れる
├── Severity: moderate
│   ├── production から到達するか? --> 次の release cycle で修正
│   └── dev-only か? --> 都合のよい時に修正し、backlog で追跡
└── Severity: low
    └── 通常の dependency update 時に追跡して修正
```

**重要な質問:**
- 脆弱な関数は実際に自分の code path から呼ばれているか?
- その dependency は runtime 依存か dev-only か?
- その脆弱性は、今の deployment context で悪用可能か?（例: client-only app における server-side 脆弱性）

修正を先送りするなら、理由を文書化し、review 日を設定してください。

## rate limiting

```typescript
import rateLimit from 'express-rate-limit';

// 一般 API の rate limit
app.use('/api/', rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100,                   // 100 requests per window
  standardHeaders: true,
  legacyHeaders: false,
}));

// auth endpoint はより厳しくする
app.use('/api/auth/', rateLimit({
  windowMs: 15 * 60 * 1000,
  max: 10,  // 10 attempts per 15 minutes
}));
```

## secret 管理

```
.env files:
  ├── .env.example  → commit する（placeholder 値だけの template）
  ├── .env          → commit しない（実 secret を含む）
  └── .env.local    → commit しない（local override）

.gitignore must include:
  .env
  .env.local
  .env.*.local
  *.pem
  *.key
```

**commit 前に必ず確認する:**
```bash
# 誤って stage された secret を確認
git diff --cached | grep -i "password\|secret\|api_key\|token"
```

## security review checklist

```markdown
### Authentication
- [ ] password を bcrypt / scrypt / argon2 で hash している（salt rounds ≥ 12）
- [ ] session token が httpOnly / secure / sameSite である
- [ ] login に rate limiting がある
- [ ] password reset token に期限がある

### Authorization
- [ ] すべての endpoint で user permission を確認している
- [ ] user は自分の resource にしか access できない
- [ ] admin action に admin role 検証がある

### Input
- [ ] すべての user input を境界で検証している
- [ ] SQL query がパラメータ化されている
- [ ] HTML 出力が encode / escape されている

### Data
- [ ] code や version control に secret がない
- [ ] API response から機微 field を除外している
- [ ] 必要なら PII を保存時暗号化している

### Infrastructure
- [ ] security header を設定している（CSP、HSTS など）
- [ ] CORS を既知 origin に制限している
- [ ] dependency の脆弱性監査をしている
- [ ] error message が内部情報を露出しない
```
## 関連資料

詳細な security checklist と pre-commit の確認手順は `references/セキュリティチェックリスト.md` を参照してください。

## よくある正当化

| Rationalization | Reality |
|---|---|
| 「これは internal tool だから security は重要ではない」 | internal tool も侵害されます。攻撃者は最も弱い環を狙います。 |
| 「security は後で足す」 | security の後付けは最初から組み込むより 10 倍難しくなります。今入れてください。 |
| 「こんなものを誰も攻撃しない」 | 自動スキャナが見つけます。security by obscurity は security ではありません。 |
| 「framework が security を面倒見てくれる」 | framework は道具を提供するだけで、保証はしません。正しく使う必要があります。 |
| 「ただの prototype だ」 | prototype は本番になります。初日から security の癖を付けてください。 |

## レッドフラグ

- user input を database query、shell command、HTML render に直接渡している
- source code や commit history に secret がある
- authentication / authorization check のない API endpoint
- CORS 設定がない、または wildcard (`*`) origin を使っている
- auth endpoint に rate limiting がない
- stack trace や内部 error を user に出している
- 既知の critical 脆弱性を持つ dependency がある

## 検証

security 関連 code を実装したら、次を確認します。

- [ ] `npm audit` に critical / high 脆弱性が出ていない
- [ ] source code や git history に secret がない
- [ ] すべての user input を system boundary で検証している
- [ ] すべての保護 endpoint で authentication / authorization を確認している
- [ ] response に security header が入っている（browser DevTools で確認）
- [ ] error response が内部 detail を露出していない
- [ ] auth endpoint に rate limiting が有効である
