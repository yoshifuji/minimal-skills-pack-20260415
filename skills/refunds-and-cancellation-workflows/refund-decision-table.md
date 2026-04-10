# 返金判定テーブル

| 条件 | 結果 |
|---|---|
| role が customer ではない | deny |
| reservation の所有者が一致しない | deny |
| reservation status が `paid` ではない | deny |
| `scheduled_for` が過去 | deny |
| payment intent がない / latest charge がない | deny |
| アクティブ状態で refund が作成された | reservation → `cancelled`、refund 関連フィールドを保存 |
| refund が即時に failed / canceled になった | エラーを返し、成功扱いしない |
