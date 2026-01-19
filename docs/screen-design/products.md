# Products画面設計書

## 1. 画面項目定義
| No. | 項目名 | 項目名(英語) | 桁数 | 属性 | 必須 | 入力 | 項目種別 | 初期値 | 備考 |
| ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- |
| 1 | 画面タイトル | highlights-title | - | - | - | - | h1 | "Our bestsellers" | props.name、category、またはデフォルト値を表示 |
| 2 | 商品画像リンク | product-img | - | - | - | - | Link | - | リスト項目。商品詳細画面（/item/{asin}）へ遷移 |
| 3 | 星評価アイコン1 | review-star | - | - | - | - | Icon | star_border | リスト項目。avg_starsに応じてstar/star_half/star_borderを表示 |
| 4 | 星評価アイコン2 | review-star | - | - | - | - | Icon | star_border | リスト項目。avg_starsに応じてstar/star_half/star_borderを表示 |
| 5 | 星評価アイコン3 | review-star | - | - | - | - | Icon | star_border | リスト項目。avg_starsに応じてstar/star_half/star_borderを表示 |
| 6 | 星評価アイコン4 | review-star | - | - | - | - | Icon | star_border | リスト項目。avg_starsに応じてstar/star_half/star_borderを表示 |
| 7 | 星評価アイコン5 | review-star | - | - | - | - | Icon | star_border | リスト項目。avg_starsに応じてstar/star_half/star_borderを表示 |
| 8 | レビュー情報 | reviews-add | - | - | - | - | div | - | リスト項目。"{num_stars} stars from {num_reviews} reviews"形式で表示 |
| 9 | 商品名 | product-name | - | - | - | - | div | - | リスト項目。product.titleを表示 |
| 10 | 商品価格 | product-price | - | - | - | - | div | - | リスト項目。"${product.price}"形式で表示 |
| 11 | カートに追加ボタン | price-add | - | - | - | - | button | - | リスト項目。クリックでaddItemToCart関数を呼び出し |
| 12 | カートアイコン | add-icon | - | - | - | - | Icon | add_shopping_cart | リスト項目。カートに追加ボタン内のアイコン |
| 13 | 前ページボタン | Previous page | - | - | - | - | Button | - | offset=0の場合は非活性。ページネーション用 |
| 14 | 次ページボタン | Next page | - | - | - | - | Button | - | 商品数が12未満または0件で更新中でない場合は非活性 |

## 2. 画面処理設計
| No. | 項目名 | 処理タイミング | 処理内容 | 備考 |
| ---- | ---- | ---- | ---- | ---- |
| 1 | 画面 | Load（componentDidMount） | 1. props.categoryまたはthis.props.match.params.categoryからカテゴリを取得<br>2. linkEncode関数でカテゴリ名をURLエンコード<br>3. setStateでcategoryを設定<br>4. fetchProducts関数を呼び出して商品データを取得 | 初期表示時の処理 |
| 2 | 画面 | PropsChange（componentWillReceiveProps） | 1. nextProps.categoryまたはnextProps.match.params.categoryから新しいカテゴリを取得<br>2. 現在のcategoryと異なる場合、setStateでcategoryを更新しproductsを空配列にリセット<br>3. fetchProducts関数を呼び出して新しいカテゴリの商品データを取得 | URLパラメータ変更時の処理 |
| 3 | 商品画像リンク | Click | 1. React Router Linkによる画面遷移<br>2. /item/{product.id.asin}へ遷移 | 商品詳細画面への遷移 |
| 4 | カートに追加ボタン | Click | 1. props.addItemToCart(product)を呼び出し<br>2. 親コンポーネント（App）のaddItemToCart関数が実行される<br>3. POST /cart/add?asin={asin}へリクエスト送信<br>4. レスポンスでカート状態を更新 | 親コンポーネントで定義された関数を呼び出し |
| 5 | 前ページボタン | Click | 1. setStateでproductsを空配列にリセット<br>2. fetchProducts関数を呼び出し（offset - limit） | offset=0の場合は非活性 |
| 6 | 次ページボタン | Click | 1. setStateでproductsを空配列にリセット、isUpdatingをtrueに設定<br>2. fetchProducts関数を呼び出し（offset + limit） | 商品数が12未満の場合は非活性 |
| 7 | fetchProducts | 内部処理 | 1. カテゴリの有無でURLを決定（/products/category/{category}?または/products?）<br>2. limit、offsetパラメータを追加<br>3. current_queryと異なる場合のみfetch実行<br>4. GET {url}limit={limit}&offset={offset}へリクエスト送信<br>5. レスポンスJSONをproducts stateに設定<br>6. isUpdatingをfalseに設定 | API: GET /products または GET /products/category/{category} |

## 3. 画面レイアウト
![Products画面](./products_screenshot.png)

※注記：上記スクリーンショットはフロントエンドのみ起動した状態で撮影しています。バックエンドAPI（products-microservice）が起動している場合、商品一覧（商品画像、星評価、商品名、価格、カートに追加ボタン）およびページネーションボタンが表示されます。

## 4. データマッピング
| No. | 項目名 | bindKey | Ownerコンポーネント | ファイルパス | state種別 | 変数名/パス | 更新API | 初期値 | 永続化 | 備考 |
| ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- |
| 1 | 画面タイトル | this.props.name / category | Products / App | react-ui/frontend/src/components/Products/index.js | props / local / url | name / category / match.params.category | setState | undefined | なし | props.name優先、次にcategory、デフォルトは"Our bestsellers" |
| 2 | 商品リスト | this.state.products | Products | react-ui/frontend/src/components/Products/index.js | local | products | fetchProducts | [] | API | GET /products または GET /products/category/{category}で取得 |
| 3 | 商品画像 | product.imUrl | Products | react-ui/frontend/src/components/Products/index.js | local | products[].imUrl | fetchProducts | - | API | 商品データの一部 |
| 4 | 星評価 | product.avg_stars | Products | react-ui/frontend/src/components/Products/index.js | local | products[].avg_stars | fetchProducts | - | API | 0-5の値で星アイコンを決定 |
| 5 | 星数 | product.num_stars | Products | react-ui/frontend/src/components/Products/index.js | local | products[].num_stars | fetchProducts | - | API | レビュー情報表示用 |
| 6 | レビュー数 | product.num_reviews | Products | react-ui/frontend/src/components/Products/index.js | local | products[].num_reviews | fetchProducts | - | API | レビュー情報表示用 |
| 7 | 商品名 | product.title | Products | react-ui/frontend/src/components/Products/index.js | local | products[].title | fetchProducts | - | API | 商品データの一部 |
| 8 | 商品価格 | product.price | Products | react-ui/frontend/src/components/Products/index.js | local | products[].price | fetchProducts | - | API | 商品データの一部 |
| 9 | 商品ID | product.id.asin / product.id | Products | react-ui/frontend/src/components/Products/index.js | local | products[].id.asin | fetchProducts | - | API | 商品詳細リンクとカート追加に使用 |
| 10 | カテゴリ | this.state.category | Products | react-ui/frontend/src/components/Products/index.js | local | category | setState | undefined | なし | URLエンコードされた値を保持 |
| 11 | 現在のクエリ | this.state.current_query | Products | react-ui/frontend/src/components/Products/index.js | local | current_query | setState | "" | なし | 重複リクエスト防止用 |
| 12 | 表示件数 | this.state.limit | Products | react-ui/frontend/src/components/Products/index.js | local | limit | fetchProducts | 12 | なし | ページネーション用 |
| 13 | オフセット | this.state.offset | Products | react-ui/frontend/src/components/Products/index.js | local | offset | fetchProducts | 0 | なし | ページネーション用 |
| 14 | 更新中フラグ | this.state.isUpdating | Products | react-ui/frontend/src/components/Products/index.js | local | isUpdating | setState | true | なし | 次ページボタンの活性制御に使用 |
| 15 | カート追加関数 | this.props.addItemToCart | App | react-ui/frontend/src/components/App/index.js | props | addItemToCart | - | - | API | POST /cart/add?asin={asin}を呼び出す関数 |
| 16 | ソートキー | this.props.sort | App | react-ui/frontend/src/components/App/index.js | props | sort | - | undefined | なし | /sort/:queryルートで使用 |
