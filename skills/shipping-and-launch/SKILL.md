---
name: shipping-and-launch
description: production launch の準備をするスキルです。production へ deploy する準備をするときに使ってください。pre-launch checklist、monitoring 設定、段階 rollout 計画、rollback 戦略が必要なときに使います。
---

# Shipping と Launch

## 概要

自信を持って ship します。目標は単に deploy することではなく、monitoring が入り、rollback plan が準備され、何を成功とみなすかが明確な状態で、安全に deploy することです。すべての launch は、reversible、observable、incremental であるべきです。

## 使う場面

- feature を初めて production に deploy するとき
- 大きな変更を user に release するとき
- data や infrastructure を migration するとき
- beta や early access program を開始するとき
- リスクを伴う deploy 全般（実質すべて）

## pre-launch checklist

### Code Quality

- [ ] すべての test が通る（unit、integration、e2e）
- [ ] warning なしで build が成功する
- [ ] lint と type check が通る
- [ ] code review と承認が済んでいる
- [ ] launch 前に消すべき TODO comment が残っていない
- [ ] production code に `console.log` debugging が残っていない
- [ ] error handling が想定 failure mode をカバーしている

### Security

- [ ] code や version control に secret がない
- [ ] `npm audit` に critical / high 脆弱性が出ていない
- [ ] すべての user-facing endpoint で input validation をしている
- [ ] authentication と authorization check が入っている
- [ ] security header が設定されている（CSP、HSTS など）
- [ ] authentication endpoint に rate limiting がある
- [ ] CORS が特定 origin に設定されている（wildcard ではない）

### Performance

- [ ] Core Web Vitals が "Good" 閾値内にある
- [ ] critical path に N+1 query がない
- [ ] image が最適化されている（圧縮、responsive size、lazy loading）
- [ ] bundle size が予算内である
- [ ] database query に適切な index がある
- [ ] static asset と繰り返し query に cache が設定されている

### Accessibility

- [ ] すべての操作要素で keyboard navigation が動く
- [ ] screen reader が page content と構造を伝えられる
- [ ] color contrast が WCAG 2.1 AA を満たす（text は 4.5:1）
- [ ] modal や動的 content の focus management が正しい
- [ ] error message が具体的で form field に関連付いている
- [ ] axe-core や Lighthouse に accessibility warning がない

### Infrastructure

- [ ] production に環境変数が設定されている
- [ ] database migration を適用済み（または適用準備済み）
- [ ] DNS と SSL が設定されている
- [ ] static asset 用 CDN が設定されている
- [ ] logging と error reporting が設定されている
- [ ] health check endpoint があり、応答する

### Documentation

- [ ] 新しい setup 要件があれば README に反映されている
- [ ] API documentation が最新である
- [ ] architectural decision があれば ADR が書かれている
- [ ] changelog が更新されている
- [ ] user-facing documentation が必要に応じて更新されている

## feature flag 戦略

deploy と release を分離するため、feature flag の裏で ship します。

```typescript
// feature flag の確認
const flags = await getFeatureFlags(userId);

if (flags.taskSharing) {
  // 新機能: task sharing
  return <TaskSharingPanel task={task} />;
}

// デフォルト: 既存の挙動
return null;
```

**feature flag の lifecycle:**

```
1. flag OFF で DEPLOY       → code は production にあるが無効
2. team / beta に ENABLE    → production 環境で内部テスト
3. GRADUAL ROLLOUT          → user の 5% → 25% → 50% → 100%
4. 各段階で MONITOR         → error rate、performance、user feedback を見る
5. CLEAN UP                 → full rollout 後に flag と dead code path を消す
```

**ルール:**
- すべての feature flag に owner と expiration date を持たせる
- full rollout 後 2 週間以内に flag を片付ける
- feature flag を入れ子にしない（組み合わせが指数的に増える）
- CI で flag の on / off 両状態を test する

## 段階 rollout

### rollout の順番

```
1. staging へ DEPLOY
   └── staging 環境でフル test suite
   └── critical flow の手動 smoke test

2. production へ DEPLOY（feature flag OFF）
   └── deploy 成功を確認（health check）
   └── error monitoring を確認（新規 error がない）

3. team 向けに ENABLE（内部 user のみ flag ON）
   └── team が production で機能を使う
   └── 24 時間の監視期間

4. CANARY rollout（user の 5% だけ flag ON）
   └── error rate、latency、user behavior を監視する
   └── metric を比較する: canary vs baseline
   └── 24-48 時間の監視期間
   └── すべての threshold を通過したときだけ次へ進む（下表）

5. GRADUAL increase（25% -> 50% -> 100%）
   └── 各段階で同じ監視を行う
   └── いつでも前の割合へ戻せるようにする

6. FULL rollout（全 user に flag ON）
   └── 1 週間監視する
   └── feature flag を片付ける
```

### rollout 判断 threshold

各段階で advance、hold、rollback を決める目安です。

| Metric | Advance (green) | Hold and investigate (yellow) | Roll back (red) |
|--------|-----------------|-------------------------------|-----------------|
| Error rate | Within 10% of baseline | 10-100% above baseline | >2x baseline |
| P95 latency | Within 20% of baseline | 20-50% above baseline | >50% above baseline |
| Client JS errors | No new error types | New errors at <0.1% of sessions | New errors at >0.1% of sessions |
| Business metrics | Neutral or positive | Decline <5% (may be noise) | Decline >5% |

### rollback すべきとき

次のときは即 rollback します。
- error rate が baseline の 2 倍超に増える
- P95 latency が 50% 超増える
- user report が急増する
- data integrity issue が見つかる
- security vulnerability が見つかる

## monitoring と observability

### 何を監視するか

```
Application metrics:
├── error rate（全体と endpoint ごと）
├── response time（p50、p95、p99）
├── request volume
├── active user 数
└── 主要 business metric（conversion、engagement）

Infrastructure metrics:
├── CPU / memory 使用率
├── database connection pool 使用量
├── disk space
├── network latency
└── queue depth（該当する場合）

Client metrics:
├── Core Web Vitals（LCP、INP、CLS）
├── JavaScript error
├── client 観点の API error rate
└── page load time
```

### error reporting

```typescript
// reporting 付き error boundary を設定する
class ErrorBoundary extends React.Component {
  componentDidCatch(error: Error, info: React.ErrorInfo) {
    // error tracking service に送る
    reportError(error, {
      componentStack: info.componentStack,
      userId: getCurrentUser()?.id,
      page: window.location.pathname,
    });
  }

  render() {
    if (this.state.hasError) {
      return <ErrorFallback onRetry={() => this.setState({ hasError: false })} />;
    }
    return this.props.children;
  }
}

// server-side error reporting
app.use((err: Error, req: Request, res: Response, next: NextFunction) => {
  reportError(err, {
    method: req.method,
    url: req.url,
    userId: req.user?.id,
  });

  // user に内部情報を出さない
  res.status(500).json({
    error: { code: 'INTERNAL_ERROR', message: '問題が発生しました' },
  });
});
```

### launch 直後の検証

launch 後 1 時間は次を確認します。

```
1. health endpoint が 200 を返す
2. error monitoring dashboard を確認する（新しい error type がない）
3. latency dashboard を確認する（regression がない）
4. critical user flow を手動で試す
5. log が流れ、読めることを確認する
6. rollback mechanism が動くことを確認する（可能なら dry run）
```

## rollback 戦略

すべての deploy は、実行前に rollback plan を持っていなければなりません。

```markdown
## [Feature / Release] の rollback plan

### 発火条件
- Error rate > 2x baseline
- P95 latency > [X]ms
- User reports of [specific issue]

### rollback 手順
1. Disable feature flag (if applicable)
   OR
1. Deploy previous version: `git revert <commit> && git push`
2. Verify rollback: health check, error monitoring
3. Communicate: notify team of rollback

### database 上の考慮
- Migration [X] has a rollback: `npx prisma migrate rollback`
- Data inserted by new feature: [preserved / cleaned up]

### rollback 所要時間
- Feature flag: < 1 minute
- Redeploy previous version: < 5 minutes
- Database rollback: < 15 minutes
```
## 関連資料

- security の pre-launch check は `references/セキュリティチェックリスト.md` を参照
- performance の pre-launch checklist は `references/パフォーマンスチェックリスト.md` を参照
- launch 前の accessibility verification は `references/アクセシビリティチェックリスト.md` を参照

## よくある正当化

| Rationalization | Reality |
|---|---|
| 「staging で動くなら production でも動く」 | production には別の data、traffic pattern、edge case があります。deploy 後に監視してください。 |
| 「この変更に feature flag は要らない」 | どんな feature にも kill switch の価値があります。単純な変更でも壊れます。 |
| 「monitoring はオーバーヘッド」 | monitoring がないと、dashboard ではなく user 苦情で問題を知ることになります。 |
| 「monitoring は後で足す」 | launch 前に入れてください。見えないものは debug できません。 |
| 「rollback は失敗を認めることだ」 | rollback は責任ある engineering です。壊れた feature を出し続けるほうが失敗です。 |

## レッドフラグ

- rollback plan なしで deploy する
- production に monitoring や error reporting がない
- big-bang release（全部一気に、staging なし）
- expiration や owner のない feature flag
- 最初の 1 時間 deploy を見ている人がいない
- production 環境設定を code ではなく記憶で行っている
- 「金曜の夕方だが ship しよう」

## 検証

deploy 前:

- [ ] pre-launch checklist が完了している（全 section 緑）
- [ ] feature flag が設定されている（必要な場合）
- [ ] rollback plan が文書化されている
- [ ] monitoring dashboard が用意されている
- [ ] team に deploy が周知されている

deploy 後:

- [ ] health check が 200 を返す
- [ ] error rate が通常範囲である
- [ ] latency が通常範囲である
- [ ] critical user flow が動く
- [ ] log が流れている
- [ ] rollback がテスト済み、またはすぐ使える状態である
