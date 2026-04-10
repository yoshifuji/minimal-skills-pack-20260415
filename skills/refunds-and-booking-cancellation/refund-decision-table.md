# 返金判定テーブル

| 条件 | 結果 |
|---|---|
| role が student ではない | deny |
| booking の所有者が一致しない | deny |
| booking status が `paid` ではない | deny |
| `scheduled_for` が過去 | deny |
| payment intent がない / latest charge がない | deny |
| アクティブ状態で refund が作成された | booking → `cancelled`、refund 関連フィールドを保存 |
| refund が即時に failed / canceled になった | エラーを返し、成功扱いしない |
