# Home画面設計書

## 1. 画面項目定義

| No. | 項目名 | 項目名(英語) | 桁数 | 属性 | 必須 | 入力 | 項目種別 | 初期値 | 備考 |
| ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- |
| 1 | ロゴリンク | logo-link | - | - | - | - | NavLink | - | トップページへ遷移 |
| 2 | Booksナビリンク | nav-books | - | - | - | - | NavLink | - | Booksカテゴリページへ遷移 |
| 3 | Musicナビリンク | nav-music | - | - | - | - | NavLink | - | Musicカテゴリページへ遷移 |
| 4 | Beautyナビリンク | nav-beauty | - | - | - | - | NavLink | - | Beautyカテゴリページへ遷移 |
| 5 | Electronicsナビリンク | nav-electronics | - | - | - | - | NavLink | - | Electronicsカテゴリページへ遷移 |
| 6 | カートリンク | nav-cart | - | - | - | - | NavLink | - | カートページへ遷移 |
| 7 | Booksセクションリンク | books-section-link | - | - | - | - | Link | - | Booksカテゴリページへ遷移 |
| 8 | Musicセクションリンク | music-section-link | - | - | - | - | Link | - | Musicカテゴリページへ遷移 |
| 9 | Beautyセクションリンク | beauty-section-link | - | - | - | - | Link | - | Beautyカテゴリページへ遷移 |
| 10 | Electronicsセクションリンク | electronics-section-link | - | - | - | - | Link | - | Electronicsカテゴリページへ遷移 |
| 11 | 商品画像リンク | product-link | - | - | - | - | Link | - | 商品詳細ページへ遷移、リスト項目 |
| 12 | カートに追加ボタン | add-to-cart-button | - | - | - | - | button | - | 商品をカートに追加、リスト項目 |
| 13 | メールアドレス入力 | email | - | - | - | ○ | FormControl | - | ニュースレター購読用 |
| 14 | 購読ボタン | subscribe-button | - | - | - | - | Button | - | ニュースレター購読 |
| 15 | yugabyte.comリンク | yugabyte-link | - | - | - | - | NavLink | - | 外部サイトへ遷移 |
| 16 | フッターカテゴリリンク | footer-category-link | - | - | - | - | NavLink | - | 各カテゴリページへ遷移、リスト項目 |

## 2. 画面処理設計

| No. | 項目名 | 処理タイミング | 処理内容 | 備考 |
| ---- | ---- | ---- | ---- | ---- |
| 1 | Home画面 | Load | 1. Homeコンポーネントがマウントされる<br>2. 子コンポーネント（Products）が各カテゴリの商品を取得する | - |
| 2 | Productsコンポーネント | Load | 1. componentDidMountでfetchProducts()を呼び出す<br>2. `/products/category/{category}?limit=4&offset=0`にGETリクエストを送信<br>3. レスポンスをthis.state.productsにsetState | Books、Music、Beauty、Electronicsの4カテゴリ分実行 |
| 3 | カートに追加ボタン | Click | 1. this.props.addItemToCart(product)を呼び出す<br>2. App.jsのaddItemToCart関数が実行される<br>3. `/cart/add?asin={asin}`にPOSTリクエストを送信<br>4. レスポンスでApp.state.cartを更新 | - |
| 4 | 商品画像リンク | Click | 1. `/item/{product.id.asin}`に画面遷移 | React Routerによる遷移 |
| 5 | ナビゲーションリンク | Click | 1. 対応するカテゴリページに画面遷移 | React Routerによる遷移 |
| 6 | カートリンク | Click | 1. `/cart`に画面遷移 | React Routerによる遷移 |

## 3. 画面レイアウト

![Home画面](./home_screenshot.png)

## 4. データマッピング

| No. | 項目名 | bindKey | Ownerコンポーネント | ファイルパス | state種別 | 変数名/パス | 更新API | 初期値 | 永続化 | 備考 |
| ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- |
| 1 | カート情報 | this.state.cart | App | react-ui/frontend/src/components/App/index.js | local | cart | fetchCart, addItemToCart | {data: {}, total: 0} | API | Navbarに渡される |
| 2 | スクロール状態 | this.state.scrolled | App | react-ui/frontend/src/components/App/index.js | local | scrolled | - | false | なし | Navbarのスタイル制御用 |
| 3 | カートに追加関数 | this.props.addItemToCart | App | react-ui/frontend/src/components/App/index.js | props | addItemToCart | - | - | - | Homeからproductsに渡される |
| 4 | 商品リスト（Books） | this.state.products | Products | react-ui/frontend/src/components/Products/index.js | local | products | fetchProducts | [] | API | category="Books", limit=4 |
| 5 | 商品リスト（Music） | this.state.products | Products | react-ui/frontend/src/components/Products/index.js | local | products | fetchProducts | [] | API | category="Music", limit=4 |
| 6 | 商品リスト（Beauty） | this.state.products | Products | react-ui/frontend/src/components/Products/index.js | local | products | fetchProducts | [] | API | category="Beauty", limit=4 |
| 7 | 商品リスト（Electronics） | this.state.products | Products | react-ui/frontend/src/components/Products/index.js | local | products | fetchProducts | [] | API | category="Electronics", limit=4 |
| 8 | カート内アイテム数 | this.props.cart.total | Navbar | react-ui/frontend/src/components/Main/components/NavBar/index.js | props | cart.total | - | 0 | - | Appから渡される |
