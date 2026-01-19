# Cart画面設計書

## 1. 画面項目定義
| No. | 項目名 | 項目名(英語) | 桁数 | 属性 | 必須 | 入力 | 項目種別 | 初期値 | 備考 |
| ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- |
| 1 | 商品タイトルリンク | product.title | - | - | - | - | Link | - | リスト項目。商品詳細ページへ遷移 |
| 2 | 削除ボタン | btn-cart-remove | - | - | - | - | Button | - | リスト項目。カートから商品を削除 |
| 3 | チェックアウトボタン | Checkout | - | - | - | - | Button | - | カートに商品がある場合のみ表示 |

## 2. 画面処理設計
| No. | 項目名 | 処理タイミング | 処理内容 | 備考 |
| ---- | ---- | ---- | ---- | ---- |
| 1 | 画面 | Load | 1. props.cart.dataからproduct_idを取得<br>2. 各product_idに対してfetchProductDetails()を呼び出し<br>3. `/products/details?asin={product_id}`にGETリクエスト送信<br>4. 取得した商品詳細をstate.productsに追加 | componentDidMount |
| 2 | 商品タイトルリンク | Click | 1. `/item/{product.id.asin \|\| product.id}`へ画面遷移 | react-router-domのLinkコンポーネント |
| 3 | 削除ボタン | Click | 1. props.removeItemFromCart(product)を呼び出し<br>2. 親コンポーネント(App)で`/cart/remove/?asin={product.id}`にPOSTリクエスト送信<br>3. レスポンスでApp.state.cartを更新 | 親コンポーネントの関数を呼び出し |
| 4 | チェックアウトボタン | Click | 1. submitCheckout()を呼び出し<br>2. `/cart/checkout`にPOSTリクエスト送信<br>3. レスポンスをstate.resultに保存<br>4. state.isCompletedをtrueに設定<br>5. props.fetchCart()を呼び出してカート情報を再取得 | カートが空の場合はdisabled |

## 3. 画面レイアウト
![Cart画面](./cart_screenshot.png)

## 4. データマッピング
| No. | 項目名 | bindKey | Ownerコンポーネント | ファイルパス | state種別 | 変数名/パス | 更新API | 初期値 | 永続化 | 備考 |
| ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- |
| 1 | カート情報 | this.props.cart | App | react-ui/frontend/src/components/App/index.js | props | cart | fetchCart | {data: {}, total: 0} | API | 親コンポーネントから受け取り |
| 2 | カートデータ | this.props.cart.data | App | react-ui/frontend/src/components/App/index.js | props | cart.data | fetchCart | {} | API | 商品ID:数量のオブジェクト |
| 3 | カート合計数 | this.props.cart.total | App | react-ui/frontend/src/components/App/index.js | props | cart.total | fetchCart | 0 | API | カート内商品の合計数量 |
| 4 | カート取得関数 | this.props.fetchCart | App | react-ui/frontend/src/components/App/index.js | props | fetchCart | - | - | - | `/cart/get`にPOSTリクエスト |
| 5 | 商品削除関数 | this.props.removeItemFromCart | App | react-ui/frontend/src/components/App/index.js | props | removeItemFromCart | - | - | - | `/cart/remove/?asin={id}`にPOSTリクエスト |
| 6 | 商品詳細リスト | this.state.products | CartProducts | react-ui/frontend/src/components/Cart/index.js | local | products | setState | [] | なし | fetchProductDetailsで取得 |
| 7 | チェックアウト完了フラグ | this.state.isCompleted | CartProducts | react-ui/frontend/src/components/Cart/index.js | local | isCompleted | setState | false | なし | チェックアウト完了時にtrue |
| 8 | チェックアウト結果 | this.state.result | CartProducts | react-ui/frontend/src/components/Cart/index.js | local | result | setState | undefined | なし | チェックアウトAPIのレスポンス |
| 9 | 商品画像 | product.imUrl | CartProducts | react-ui/frontend/src/components/Cart/index.js | local | products[].imUrl | - | - | なし | fetchProductDetailsで取得 |
| 10 | 商品タイトル | product.title | CartProducts | react-ui/frontend/src/components/Cart/index.js | local | products[].title | - | - | なし | fetchProductDetailsで取得 |
| 11 | 商品価格 | product.price | CartProducts | react-ui/frontend/src/components/Cart/index.js | local | products[].price | - | - | なし | fetchProductDetailsで取得 |
| 12 | 商品ID | product.id | CartProducts | react-ui/frontend/src/components/Cart/index.js | local | products[].id | - | - | なし | fetchProductDetailsで取得 |
| 13 | 注文番号 | this.state.result.orderNumber | CartProducts | react-ui/frontend/src/components/Cart/index.js | local | result.orderNumber | - | - | API | チェックアウト完了時に表示 |
| 14 | 注文詳細 | this.state.result.orderDetails | CartProducts | react-ui/frontend/src/components/Cart/index.js | local | result.orderDetails | - | - | API | チェックアウト完了時に表示 |
