# Migration チェックリスト

- [ ] table / column / index / constraint の意図を説明できる
- [ ] breaking change を 1 本に詰め込んでいない
- [ ] backfill が必要か判断している
- [ ] app code と DB 変更の順番が決まっている
- [ ] rollback が難しい変更を把握している
- [ ] schema 変更後に型や生成物を更新している
- [ ] lookup で使う列に index がある
