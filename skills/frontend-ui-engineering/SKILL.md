---
name: frontend-ui-engineering
description: production 品質の UI を作るスキルです。user-facing interface を作る、または変更するときに使ってください。component 作成、layout 実装、state 管理、または AI 生成風ではなく production 品質の見た目と操作感が必要なときに使います。
---

# Frontend UI Engineering

## 概要

アクセシブルで、高速で、見た目まで磨かれた production 品質の user interface を作ります。目標は、AI が生成したように見える UI ではなく、上位企業の design-aware な engineer が作ったように見える UI です。つまり、実際の design system に従い、適切な accessibility を持ち、よく考えられた interaction pattern を備え、ありがちな「AI aesthetic」を避けるということです。

## 使う場面

- 新しい UI component や page を作るとき
- 既存の user-facing interface を変更するとき
- responsive layout を実装するとき
- interactivity や state management を足すとき
- visual / UX の問題を直すとき

## component アーキテクチャ

### file 構成

component に関係するものは colocate します。

```
src/components/
  TaskList/
    TaskList.tsx          # component 実装
    TaskList.test.tsx     # test
    TaskList.stories.tsx  # Storybook story（使う場合）
    use-task-list.ts      # custom hook（state が複雑な場合）
    types.ts              # component 専用型（必要な場合）
```

### component パターン

**configuration より composition を優先する:**

```tsx
// 良い例: 合成しやすい
<Card>
  <CardHeader>
    <CardTitle>Tasks</CardTitle>
  </CardHeader>
  <CardBody>
    <TaskList tasks={tasks} />
  </CardBody>
</Card>

// 避ける: 設定過多
<Card
  title="Tasks"
  headerVariant="large"
  bodyPadding="md"
  content={<TaskList tasks={tasks} />}
/>
```

**component は焦点を絞る:**

```tsx
// 良い例: 1 つのことだけをする
export function TaskItem({ task, onToggle, onDelete }: TaskItemProps) {
  return (
    <li className="flex items-center gap-3 p-3">
      <Checkbox checked={task.done} onChange={() => onToggle(task.id)} />
      <span className={task.done ? 'line-through text-muted' : ''}>{task.title}</span>
      <Button variant="ghost" size="sm" onClick={() => onDelete(task.id)}>
        <TrashIcon />
      </Button>
    </li>
  );
}
```

**data fetching と presentation を分ける:**

```tsx
// Container: data を扱う
export function TaskListContainer() {
  const { tasks, isLoading, error } = useTasks();

  if (isLoading) return <TaskListSkeleton />;
  if (error) return <ErrorState message="Failed to load tasks" retry={refetch} />;
  if (tasks.length === 0) return <EmptyState message="No tasks yet" />;

  return <TaskList tasks={tasks} />;
}

// Presentation: 描画だけを担当
export function TaskList({ tasks }: { tasks: Task[] }) {
  return (
    <ul role="list" className="divide-y">
      {tasks.map(task => <TaskItem key={task.id} task={task} />)}
    </ul>
  );
}
```

## state management

**動く中で最も単純な方法を選ぶ:**

```
Local state (`useState`)          → component 固有の UI state
Lifted state                      → 2-3 個の sibling component で共有
Context                           → theme、auth、locale（read 多め、write 少なめ）
URL state (`searchParams`)        → filter、pagination、共有可能な UI state
Server state（React Query、SWR）   → cache 付き remote data
Global store（Zustand、Redux）     → app 全体で共有する複雑な client state
```

**prop drilling を 3 段より深くしない。** 使わない component を経由して props を渡しているなら、context を導入するか component tree を組み替えてください。

## design system への準拠

### AI っぽい見た目を避ける

AI 生成 UI には分かりやすい癖があります。全部避けてください。

| AI のありがち | 問題点 | Production 品質の代替 |
|---|---|---|
| 全部紫 / indigo | model が視覚的に無難な palette に寄り、どの app も同じ見た目になる | project 本来の color palette を使う |
| gradient の多用 | gradient は視覚ノイズを増やし、多くの design system と衝突する | design system に合うフラット、または控えめな gradient |
| 何でも丸い（`rounded-2xl`） | 最大丸角は「親しみやすさ」は出るが、実デザインの corner radius 階層を無視する | design system にある一貫した border-radius |
| 汎用 hero section | template 駆動で、実 content や user need と結び付かない | content-first layout |
| Lorem ipsum 風 copy | 仮文は、実文で起きる長さ、折り返し、overflow 問題を隠す | 現実的な placeholder content |
| 過大な padding | 一律で大きい余白は視覚階層を壊し、画面を浪費する | 一貫した spacing scale |
| 量産型 card grid | 均一 grid は情報優先度や視線走査を無視した近道 | 目的に合わせた layout |
| 影だらけの design | 重ねた shadow は content と競合し、低性能端末の描画も遅くする | design system 指定がない限り、控えめまたはなし |

### spacing と layout

一貫した spacing scale を使います。思い付きの値を作らないでください。

```css
/* 0.25rem 刻みなど、project の scale を使う */
/* 良い例 */  padding: 1rem;      /* 16px */
/* 良い例 */  gap: 0.75rem;       /* 12px */
/* 悪い例 */  padding: 13px;      /* scale にない */
/* 悪い例 */  margin-top: 2.3rem; /* scale にない */
```

### typography

type hierarchy を守ります。

```
h1 → page title（1 page に 1 つ）
h2 → section title
h3 → subsection title
body → 通常本文
small → 補助 / 二次テキスト
```

heading level を飛ばさないでください。heading でない content に heading style を流用しないでください。

### color

- raw hex 値ではなく semantic color token を使う: `text-primary`, `bg-surface`, `border-default`
- 十分な contrast を確保する（通常 text は 4.5:1、大きい text は 3:1）
- 色だけで情報を伝えない（icon、text、pattern も使う）

## Accessibility（WCAG 2.1 AA）

すべての component は次の基準を満たす必要があります。

### keyboard navigation

```tsx
// すべての操作要素は keyboard でアクセス可能でなければならない
<button onClick={handleClick}>Click me</button>        // ✓ Focusable by default
<div onClick={handleClick}>Click me</div>               // ✗ Not focusable
<div role="button" tabIndex={0} onClick={handleClick}    // ✓ ただし <button> を優先
     onKeyDown={e => e.key === 'Enter' && handleClick()}>
  Click me
</div>
```

### ARIA label

```tsx
// 見える text がない操作要素には label を付ける
<button aria-label="Close dialog"><XIcon /></button>

// form input に label を付ける
<label htmlFor="email">Email</label>
<input id="email" type="email" />

// 見える label がないなら aria-label を使う
<input aria-label="Search tasks" type="search" />
```

### focus management

```tsx
// content 変更時に focus を移す
function Dialog({ isOpen, onClose }: DialogProps) {
  const closeRef = useRef<HTMLButtonElement>(null);

  useEffect(() => {
    if (isOpen) closeRef.current?.focus();
  }, [isOpen]);

  // dialog が開いている間は内側に focus を閉じ込める
  return (
    <dialog open={isOpen}>
      <button ref={closeRef} onClick={onClose}>Close</button>
      {/* dialog content */}
    </dialog>
  );
}
```

### 意味のある empty / error state

```tsx
// 真っ白な画面にしない
function TaskList({ tasks }: { tasks: Task[] }) {
  if (tasks.length === 0) {
    return (
      <div role="status" className="text-center py-12">
        <TasksEmptyIcon className="mx-auto h-12 w-12 text-muted" />
        <h3 className="mt-2 text-sm font-medium">No tasks</h3>
        <p className="mt-1 text-sm text-muted">Get started by creating a new task.</p>
        <Button className="mt-4" onClick={onCreateTask}>Create Task</Button>
      </div>
    );
  }

  return <ul role="list">...</ul>;
}
```

## responsive design

mobile first で設計し、その後に広げます。

```tsx
// Tailwind: mobile-first responsive
<div className="
  grid grid-cols-1      /* mobile: 1 列 */
  sm:grid-cols-2        /* small: 2 列 */
  lg:grid-cols-3        /* large: 3 列 */
  gap-4
">
```

320px、768px、1024px、1440px で確認してください。

## loading と transition

```tsx
// skeleton loading（content に spinner だけを使わない）
function TaskListSkeleton() {
  return (
    <div className="space-y-3" aria-busy="true" aria-label="Loading tasks">
      {Array.from({ length: 3 }).map((_, i) => (
        <div key={i} className="h-12 bg-muted animate-pulse rounded" />
      ))}
    </div>
  );
}

// 体感速度のための optimistic update
function useToggleTask() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: toggleTask,
    onMutate: async (taskId) => {
      await queryClient.cancelQueries({ queryKey: ['tasks'] });
      const previous = queryClient.getQueryData(['tasks']);

      queryClient.setQueryData(['tasks'], (old: Task[]) =>
        old.map(t => t.id === taskId ? { ...t, done: !t.done } : t)
      );

      return { previous };
    },
    onError: (_err, _taskId, context) => {
      queryClient.setQueryData(['tasks'], context?.previous);
    },
  });
}
```

## 関連資料

詳細な accessibility 要件と test tool は `references/アクセシビリティチェックリスト.md` を参照してください。

## よくある正当化

| Rationalization | Reality |
|---|---|
| 「accessibility は nice-to-have」 | 多くの法域で法的要件であり、engineering quality の基準でもあります。 |
| 「responsive は後でやる」 | responsive design の後付けは、最初から組み込むより 3 倍大変です。 |
| 「design はまだ固まっていないから styling は後でいい」 | design system の default を使ってください。unstyled UI は reviewer に壊れた第一印象を与えます。 |
| 「これは prototype だから」 | prototype は production code になります。土台から正しく作ってください。 |
| 「今は AI っぽい見た目でもいい」 | それは低品質の信号です。最初から project 本来の design system を使ってください。 |

## レッドフラグ

- 200 行を超える component（分割する）
- inline style や任意の pixel 値
- error state、loading state、empty state の欠落
- keyboard navigation test がない
- 色だけで state を表している（text や icon のない赤 / 緑など）
- ありがちな「AI look」（紫 gradient、巨大 card、量産 layout）

## 検証

UI を作ったら、次を確認します。

- [ ] component が console error なしで描画される
- [ ] すべての操作要素に keyboard でアクセスできる（Tab で一巡する）
- [ ] screen reader が page の内容と構造を伝えられる
- [ ] responsive である（320px、768px、1024px、1440px で動く）
- [ ] loading、error、empty state をすべて扱っている
- [ ] project の design system（spacing、color、typography）に従っている
- [ ] dev tools や axe-core に accessibility warning がない
