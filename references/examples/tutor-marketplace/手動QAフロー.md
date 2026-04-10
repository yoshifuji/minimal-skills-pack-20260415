# 手動 QA フロー

## フロー 1: Tutor onboarding

1. `/signup` で tutor 登録
2. `/dashboard/tutor` に遷移
3. Connect account を作成
4. onboarding に進む
5. `設定状況を更新` を押す
6. `charges_enabled` と `payouts_enabled` が反映されることを確認
7. profile と offering を作成する

## フロー 2: Student booking

1. 別ブラウザで student 登録
2. `/tutors/[id]` へ移動
3. offering を選ぶ
4. `scheduled_for` と `notes` を入れる
5. Checkout へ進む
6. 決済完了後、`/checkout/success` と student dashboard で `paid` を確認

## フロー 3: Checkout cancel recovery

1. Checkout に進む
2. 戻る / cancel を実行
3. `/checkout/cancel?booking_id=...` に戻る
4. 対象 booking が `pending` → `cancelled` になっていることを確認

## フロー 4: Refund

1. 未来日時の `paid` booking を用意
2. student dashboard からキャンセルを確定
3. refund が作成され、refund status が出ることを確認
4. refund webhook 到達後に state が整うことを確認

## フロー 5: webhook retry 挙動

1. webhook 内で意図的に DB 更新失敗を再現
2. endpoint が 500 を返すことを確認
3. Stripe 側で再送されることを確認
4. 問題解消後に再送で最終状態が整うことを確認
