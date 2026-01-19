# Products画面設計書

## 1. 画面項目定義
| No. | 項目名 | 項目名(英語) | 桁数 | 属性 | 必須 | 入力 | 項目種別 | 初期値 | 備考 |
| ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- |
| 1 | 商品詳細リンク | product-link | - | - | - | - | Link | - | リスト項目 |
| 2 | カートに追加ボタン | add-to-cart-button | - | - | - | - | button | - | リスト項目 |
| 3 | 前のページボタン | previous-page-button | - | - | - | - | Button | disabled | offset=0の場合は非活性 |
| 4 | 次のページボタン | next-page-button | - | - | - | - | Button | - | 商品数が12未満の場合は非活性 |

## 2. 画面処理設計
| No. | 項目名 | 処理タイミング | 処理内容 | 備考 |
| ---- | ---- | ---- | ---- | ---- |
| 1 | 画面 | Load | 1. URLパラメータまたはpropsからcategoryを取得<br>2. stateにcategoryを設定<br>3. fetchProducts()を呼び出して商品一覧を取得 | componentDidMount |
| 2 | 画面 | PropsChange | 1. categoryが変更された場合、stateを更新<br>2. fetchProducts()を呼び出して商品一覧を再取得 | componentWillReceiveProps |
| 3 | 商品詳細リンク | Click | 1. `/item/{asin}`に画面遷移 | React Router Link |
| 4 | カートに追加ボタン | Click | 1. props.addItemToCart(product)を呼び出し<br>2. 親コンポーネント（App）のaddItemToCart関数が実行される<br>3. `/cart/add?asin={asin}`にPOSTリクエスト<br>4. レスポンスでcart stateを更新 | 親コンポーネントの関数を呼び出し |
| 5 | 前のページボタン | Click | 1. productsを空配列にリセット<br>2. fetchProducts()をoffset - limitで呼び出し<br>3. レスポンスでproducts stateを更新 | offset=0の場合は非活性 |
| 6 | 次のページボタン | Click | 1. productsを空配列にリセット<br>2. isUpdatingをtrueに設定<br>3. fetchProducts()をoffset + limitで呼び出し<br>4. レスポンスでproducts stateを更新 | 商品数が12未満の場合は非活性 |

## 3. 画面レイアウト
![Products画面](./products_screenshot.png)

## 4. データマッピング
| No. | 項目名 | bindKey | Ownerコンポーネント | ファイルパス | state種別 | 変数名/パス | 更新API | 初期値 | 永続化 | 備考 |
| ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- |
| 1 | 商品一覧 | this.state.products | Products | react-ui/frontend/src/components/Products/index.js | local | products | fetchProducts (setState) | [] | API | /products または /products/category/{category} |
| 2 | カテゴリ | this.state.category | Products | react-ui/frontend/src/components/Products/index.js | local | category | componentDidMount, componentWillReceiveProps | undefined | なし | URLパラメータまたはpropsから取得 |
| 3 | URLカテゴリ | this.props.match.params.category | React Router | - | url | category | - | - | なし | URLパスパラメータ |
| 4 | propsカテゴリ | this.props.category | App | react-ui/frontend/src/components/App/index.js | props | category | - | - | なし | 親コンポーネントから渡される |
| 5 | カートに追加関数 | this.props.addItemToCart | App | react-ui/frontend/src/components/App/index.js | props | addItemToCart | addItemToCart | - | API | /cart/add?asin={asin} |
| 6 | 表示件数 | this.state.limit | Products | react-ui/frontend/src/components/Products/index.js | local | limit | fetchProducts (setState) | 12 | なし | propsで上書き可能 |
| 7 | オフセット | this.state.offset | Products | react-ui/frontend/src/components/Products/index.js | local | offset | fetchProducts (setState) | 0 | なし | ページネーション用 |
| 8 | 更新中フラグ | this.state.isUpdating | Products | react-ui/frontend/src/components/Products/index.js | local | isUpdating | fetchProducts (setState) | true | なし | ローディング状態管理 |
