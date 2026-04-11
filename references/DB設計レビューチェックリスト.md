# Schema Review チェックリスト

- [ ] nullable の理由を説明できる
- [ ] unique 制約が必要な列に入っている
- [ ] 外部キーが必要な関係に入っている
- [ ] status の allowed values が曖昧でない
- [ ] timestamps が必要な table にある
- [ ] ownership を code だけに頼っていない
- [ ] 将来の cleanup や archive 方針を考えている
