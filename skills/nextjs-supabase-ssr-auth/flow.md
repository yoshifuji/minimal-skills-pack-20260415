# 推奨ファイル分割

```text
lib/supabase/client.ts   # ブラウザクライアント
lib/supabase/server.ts   # SSR / ルートハンドラ用クライアント
lib/supabase/admin.ts    # サーバー専用の管理クライアント
lib/auth.ts              # getCurrentUserAndProfile, requireRole
middleware.ts            # セッション更新、webhook のバイパス
components/LoginForm.tsx
components/SignupForm.tsx
components/SignOutButton.tsx
```

## リダイレクト方針

- ログイン成功時:
  - tutor → `/dashboard/tutor`
  - student → `/dashboard/student`
  - fallback → `/`
- すでに認証済みの状態で `/login` または `/signup` を開いた場合:
  - tutor → `/dashboard/tutor`
  - student → `/dashboard/student`
