# API一覧

## 1. API一覧表

| No. | APIパス | 機能 | 使用画面 |
| ---- | ---- | ---- | ---- |
| 1 | GET /products | - 商品一覧を取得する | - / (ホーム画面) |
| 2 | GET /products/category/{category} | - カテゴリ別商品一覧を取得する<br>- limit、offsetパラメータでページネーション | - / (ホーム画面)<br>- /:category (商品一覧画面) |
| 3 | GET /products/details | - 商品詳細情報を取得する<br>- asinパラメータで商品を指定 | - /item/:id (商品詳細画面)<br>- /cart (カート画面) |
| 4 | POST /cart/add | - 商品をカートに追加する<br>- asinパラメータで商品を指定 | - / (ホーム画面)<br>- /:category (商品一覧画面)<br>- /item/:id (商品詳細画面) |
| 5 | POST /cart/get | - カート内の商品一覧を取得する | - 全画面（App.jsで初期化時に呼び出し） |
| 6 | POST /cart/getCart | - カート内の商品一覧を取得する | - 全画面（App.jsで初期化時に呼び出し） |
| 7 | POST /cart/remove | - カートから商品を削除する<br>- asinパラメータで商品を指定 | - /cart (カート画面) |
| 8 | POST /cart/checkout | - チェックアウト処理を実行する<br>- 注文を確定し、注文番号を返却 | - /cart (カート画面) |
| 9 | GET /registration | - ユーザー登録画面を表示する | - /registration (登録画面) |
| 10 | POST /registration | - ユーザー登録処理を実行する | - /registration (登録画面) |
| 11 | GET /login | - ログイン画面を表示する | - /login (ログイン画面) |
| 12 | GET / | - ウェルカム画面を表示する<br>- 設定されたリダイレクトURLへ遷移 | - / (ウェルカム画面) |
| 13 | GET /welcome | - ウェルカム画面を表示する<br>- 設定されたリダイレクトURLへ遷移 | - /welcome (ウェルカム画面) |
