# Data Access パターン

## 推奨の分け方

- route handler
  - request / response を扱う
- service helper
  - 業務ルールを扱う
- data access
  - DB 読み書きを扱う

## 例

```ts
// route
const input = CreatePostSchema.parse(body);
const post = await createPost(input, currentUser.id);
return NextResponse.json(post, { status: 201 });
```

```ts
// service
export async function createPost(input: CreatePostInput, userId: string) {
  return insertPost({
    ...input,
    authorId: userId,
  });
}
```

```ts
// data access
export async function insertPost(input: InsertPostInput) {
  return db.post.create({ data: input });
}
```

## 原則

- query を UI に散らさない
- null と error の扱いを決める
- server-only access を browser に混ぜない
