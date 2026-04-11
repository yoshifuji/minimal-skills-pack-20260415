# Route Handler テンプレート

```ts
export async function POST(request: Request) {
  try {
    const currentUser = await requireUser();
    const body = await request.json();
    const input = CreateItemSchema.parse(body);

    const item = await createItem({
      ...input,
      actorId: currentUser.id,
    });

    return Response.json(item, { status: 201 });
  } catch (error) {
    return toErrorResponse(error);
  }
}
```

## 見るポイント

- 認証確認が先にあるか
- parse と validation が入口で行われているか
- route が thin か
- error response が統一されているか
