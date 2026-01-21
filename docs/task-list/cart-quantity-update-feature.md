# カート画面数量変更機能 実行可能な改修計画

## 前提：現在の実装状況

### フロントエンド（React）

現在のカート画面（`react-ui/frontend/src/components/Cart/index.js`）では、商品の数量表示と「Remove」ボタンのみが実装されています。

```javascript
// react-ui/frontend/src/components/Cart/index.js (74-79行目)
<div className="pricing">
  <h6>${product.price.toFixed(2)}</h6> x {this.props.cart.data[product.id]}
</div>
<div className="actions">
  <Button className="btn-cart-remove" onClick={() => this.props.removeItemFromCart(product)}  size="meduim">Remove</Button>
</div>
```

数量は `{this.props.cart.data[product.id]}` で表示されていますが、直接変更するUIは存在しません。

### バックエンド（cart-microservice）

現在のAPIエンドポイント（`cart-microservice/src/main/java/com/yugabyte/app/yugastore/cart/controller/ShoppingCartController.java`）：

```java
// ShoppingCartController.java
@RequestMapping(method = RequestMethod.GET, value = "/shoppingCart/addProduct")
public String addProductToCart(@RequestParam("userid") String userId, 
        @RequestParam("asin") String asin)

@RequestMapping(method = RequestMethod.GET, value = "/shoppingCart/removeProduct")
public String removeProductFromCart(@RequestParam("userid") String userId, 
        @RequestParam("asin") String asin)

@RequestMapping(method = RequestMethod.GET, value = "/shoppingCart/productsInCart")
public Map<String, Integer> getProductsInCart(@RequestParam("userid") String userId)

@RequestMapping(method = RequestMethod.GET, value = "/shoppingCart/clearCart")
public String clearCart(@RequestParam("userid") String userId)
```

`addProduct` は数量を1増加、`removeProduct` は数量を1減少させる実装です。任意の数量を直接設定するAPIは存在しません。

### データモデル

`ShoppingCart` エンティティ（`cart-microservice/src/main/java/com/yugabyte/app/yugastore/cart/domain/ShoppingCart.java`）：

```java
// ShoppingCart.java
@Entity(name = "shopping_cart")
@Table(name = "shopping_cart")
public class ShoppingCart {
    @Id
    @Column(name = "cart_key")
    private String cartKey;

    @Column(name = "user_id")
    private String userId;
    
    @Column(name = "asin")
    private String asin;

    @Column(name = "time_added")
    private String time_added;
    
    @Column(name = "quantity")
    private int quantity;
}
```

`quantity` フィールドは既に存在しており、データモデルの変更は不要です。

---

## Phase 1: 開発タスク

### 1.1 バックエンド開発（cart-microservice）

#### 1.1.1 数量更新APIの追加

**ファイル**: `cart-microservice/src/main/java/com/yugabyte/app/yugastore/cart/controller/ShoppingCartController.java`

新規エンドポイントを追加：

```java
@RequestMapping(method = RequestMethod.PUT, value = "/shoppingCart/updateQuantity", produces = "application/json")
public ResponseEntity<String> updateProductQuantity(
        @RequestParam("userid") String userId, 
        @RequestParam("asin") String asin,
        @RequestParam("quantity") int quantity) {
    
    if (quantity < 1 || quantity > 99) {
        return ResponseEntity.badRequest().body("Quantity must be between 1 and 99");
    }
    
    boolean success = shoppingCart.updateProductQuantity(userId, asin, quantity);
    if (success) {
        return ResponseEntity.ok("Quantity updated successfully");
    } else {
        return ResponseEntity.notFound().build();
    }
}
```

**工数**: 0.5人日

#### 1.1.2 サービス層の実装

**ファイル**: `cart-microservice/src/main/java/com/yugabyte/app/yugastore/cart/service/ShoppingCartImpl.java`

新規メソッドを追加：

```java
public boolean updateProductQuantity(String userId, String asin, int quantity) {
    String shoppingCartKeyStr = userId + "-" + asin;
    Optional<ShoppingCart> cartItem = shoppingCartRepository.findById(shoppingCartKeyStr);
    
    if (cartItem.isPresent()) {
        shoppingCartRepository.setQuantityForShoppingCart(userId, asin, quantity);
        System.out.println("Updated product: " + asin + " quantity to: " + quantity);
        return true;
    }
    return false;
}
```

**工数**: 0.5人日

#### 1.1.3 リポジトリ層の実装

**ファイル**: `cart-microservice/src/main/java/com/yugabyte/app/yugastore/cart/repositories/ShoppingCartRepository.java`

新規クエリメソッドを追加：

```java
@Modifying
@Query("UPDATE shopping_cart SET quantity = :quantity WHERE user_id = :userId AND asin = :asin")
void setQuantityForShoppingCart(@Param("userId") String userId, 
                                 @Param("asin") String asin, 
                                 @Param("quantity") int quantity);
```

**工数**: 0.25人日

### 1.2 バックエンド開発（api-gateway-microservice）

#### 1.2.1 Feignクライアントの更新

**ファイル**: `api-gateway-microservice/src/main/java/com/yugabyte/app/yugastore/rest/clients/ShoppingCartRestClient.java`

新規メソッドを追加：

```java
@RequestMapping(method = RequestMethod.PUT, value = "/cart-microservice/shoppingCart/updateQuantity")
ResponseEntity<String> updateProductQuantity(
    @RequestParam("userid") String userId,
    @RequestParam("asin") String asin,
    @RequestParam("quantity") int quantity);
```

**工数**: 0.25人日

#### 1.2.2 コントローラーの更新

**ファイル**: `api-gateway-microservice/src/main/java/com/yugabyte/app/yugastore/controller/ShoppingCartController.java`

新規エンドポイントを追加：

```java
@RequestMapping(method = RequestMethod.PUT, value = "/cart/updateQuantity", produces = "application/json")
public ResponseEntity<String> updateProductQuantity(
        @RequestParam("asin") String asin,
        @RequestParam("quantity") int quantity,
        HttpServletRequest request) {
    
    String userId = getUserId(request);
    return shoppingCartRestClient.updateProductQuantity(userId, asin, quantity);
}
```

**工数**: 0.5人日

### 1.3 在庫チェック機能の実装（オプション）

#### 1.3.1 在庫確認APIの呼び出し

数量変更時に在庫数を超えないようチェックする機能。`checkout-microservice` の在庫テーブル（`cronos.inventory`）を参照。

**ファイル**: 新規作成 `api-gateway-microservice/src/main/java/com/yugabyte/app/yugastore/rest/clients/InventoryRestClient.java`

```java
@FeignClient("checkout-microservice")
public interface InventoryRestClient {
    @RequestMapping("/checkout-microservice/inventory/check")
    InventoryStatus checkInventory(@RequestParam("asin") String asin);
}
```

**工数**: 1.5人日（在庫チェックAPI新規作成含む）

### 1.4 フロントエンド開発（React）

#### 1.4.1 数量変更UIコンポーネントの作成

**ファイル**: `react-ui/frontend/src/components/Cart/index.js`

数量表示部分を入力可能なコンポーネントに変更：

```jsx
<div className="quantity-control">
  <button 
    className="qty-btn minus" 
    onClick={() => this.handleQuantityChange(product, -1)}
    disabled={this.props.cart.data[product.id] <= 1}
  >
    -
  </button>
  <input 
    type="number" 
    className="qty-input"
    value={this.props.cart.data[product.id]}
    onChange={(e) => this.handleQuantityInput(product, e.target.value)}
    min="1"
    max="99"
  />
  <button 
    className="qty-btn plus" 
    onClick={() => this.handleQuantityChange(product, 1)}
    disabled={this.props.cart.data[product.id] >= 99}
  >
    +
  </button>
</div>
```

**工数**: 1.5人日

#### 1.4.2 数量変更ハンドラーの実装

```javascript
handleQuantityChange = (product, delta) => {
  const currentQty = this.props.cart.data[product.id];
  const newQty = currentQty + delta;
  if (newQty >= 1 && newQty <= 99) {
    this.updateQuantity(product.id, newQty);
  }
}

handleQuantityInput = (product, value) => {
  const newQty = parseInt(value, 10);
  if (!isNaN(newQty) && newQty >= 1 && newQty <= 99) {
    this.updateQuantity(product.id, newQty);
  }
}

updateQuantity = (productId, quantity) => {
  fetch(`/cart/updateQuantity?asin=${productId}&quantity=${quantity}`, {
    method: 'PUT',
    headers: {
      'Accept': 'application/json',
      'Content-Type': 'application/json'
    }
  })
    .then(res => res.json())
    .then(() => this.props.fetchCart())
    .catch(err => console.error('Failed to update quantity:', err));
}
```

**工数**: 1人日

#### 1.4.3 CSSスタイリング

**ファイル**: `react-ui/frontend/src/components/Cart/index.css`

```css
.quantity-control {
  display: flex;
  align-items: center;
  gap: 8px;
}

.qty-btn {
  width: 32px;
  height: 32px;
  border: 1px solid #ddd;
  background: #f5f5f5;
  cursor: pointer;
  font-size: 18px;
  border-radius: 4px;
}

.qty-btn:hover:not(:disabled) {
  background: #e0e0e0;
}

.qty-btn:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

.qty-input {
  width: 50px;
  height: 32px;
  text-align: center;
  border: 1px solid #ddd;
  border-radius: 4px;
  font-size: 14px;
}

.qty-input::-webkit-inner-spin-button,
.qty-input::-webkit-outer-spin-button {
  -webkit-appearance: none;
  margin: 0;
}
```

**工数**: 0.5人日

### 1.5 エラーハンドリング・バリデーション

#### 1.5.1 バックエンドバリデーション

- 数量範囲チェック（1〜99）
- 商品存在チェック
- ユーザー認証チェック

**工数**: 0.5人日

#### 1.5.2 フロントエンドバリデーション

- 入力値の即時バリデーション
- エラーメッセージ表示
- ローディング状態の表示

**工数**: 0.5人日

---

## Phase 2: テスト（単体テスト）

### 2.1 バックエンド単体テスト

#### 2.1.1 ShoppingCartControllerTest

**ファイル**: `cart-microservice/src/test/java/com/yugabyte/app/yugastore/cart/controller/ShoppingCartControllerTest.java`

```java
@WebMvcTest(ShoppingCartController.class)
public class ShoppingCartControllerTest {
    
    @Test
    public void testUpdateQuantity_Success() {
        // 正常系：数量更新成功
    }
    
    @Test
    public void testUpdateQuantity_InvalidQuantity() {
        // 異常系：範囲外の数量
    }
    
    @Test
    public void testUpdateQuantity_ProductNotFound() {
        // 異常系：商品が存在しない
    }
}
```

**工数**: 1人日

#### 2.1.2 ShoppingCartImplTest

**ファイル**: `cart-microservice/src/test/java/com/yugabyte/app/yugastore/cart/service/ShoppingCartImplTest.java`

```java
@SpringBootTest
public class ShoppingCartImplTest {
    
    @Test
    public void testUpdateProductQuantity_Success() {
        // 正常系：数量更新成功
    }
    
    @Test
    public void testUpdateProductQuantity_NotFound() {
        // 異常系：カートに商品がない
    }
}
```

**工数**: 1人日

### 2.2 フロントエンド単体テスト

#### 2.2.1 Cartコンポーネントテスト

**ファイル**: `react-ui/frontend/src/components/Cart/Cart.test.js`

```javascript
describe('Cart Component', () => {
  test('renders quantity controls', () => {
    // 数量変更UIが表示されることを確認
  });
  
  test('handles quantity increment', () => {
    // +ボタンクリックで数量が増加
  });
  
  test('handles quantity decrement', () => {
    // -ボタンクリックで数量が減少
  });
  
  test('validates quantity input', () => {
    // 入力値のバリデーション
  });
  
  test('disables minus button at quantity 1', () => {
    // 数量1で-ボタンが無効化
  });
});
```

**工数**: 1.5人日

### 2.3 統合テスト

#### 2.3.1 E2Eテスト（Cypress/Selenium）

**ファイル**: `react-ui/frontend/cypress/integration/cart.spec.js`

```javascript
describe('Cart Quantity Update', () => {
  it('should update quantity using plus button', () => {
    // +ボタンで数量更新
  });
  
  it('should update quantity using minus button', () => {
    // -ボタンで数量更新
  });
  
  it('should update quantity using direct input', () => {
    // 直接入力で数量更新
  });
  
  it('should update total price when quantity changes', () => {
    // 数量変更時に合計金額が更新される
  });
});
```

**工数**: 2人日

---

## Phase 3: 設計書のリバースエンジニアリング

### 3.1 画面設計書の更新

**ファイル**: `docs/screen-design/cart-screen-design.md`

更新内容：
- 数量変更UIの追加（+/-ボタン、入力フィールド）
- 画面レイアウト図の更新
- UI要素の仕様（サイズ、色、フォント）
- 操作フロー図

**工数**: 1人日

### 3.2 API設計書の作成・更新

**ファイル**: `docs/api-design/cart-api-design.md`

新規API仕様：

```yaml
PUT /cart/updateQuantity
  Parameters:
    - asin: string (required) - 商品ID
    - quantity: integer (required) - 新しい数量 (1-99)
  Response:
    - 200: 更新成功
    - 400: バリデーションエラー
    - 404: 商品が見つからない
```

**工数**: 0.5人日

### 3.3 データベース設計書の確認

**ファイル**: `docs/table_design/` 配下

確認内容：
- `shopping_cart` テーブルの `quantity` カラムの仕様確認
- 変更不要であることの確認・記載

**工数**: 0.25人日

### 3.4 シーケンス図・アーキテクチャ図の作成

**ファイル**: `docs/architecture/cart-quantity-update-sequence.md`

```
User -> React UI: 数量変更操作
React UI -> API Gateway: PUT /cart/updateQuantity
API Gateway -> Cart Microservice: PUT /shoppingCart/updateQuantity
Cart Microservice -> YugabyteDB: UPDATE shopping_cart
YugabyteDB -> Cart Microservice: Success
Cart Microservice -> API Gateway: 200 OK
API Gateway -> React UI: 200 OK
React UI -> User: 画面更新
```

**工数**: 0.5人日

---

## 工数サマリー

| フェーズ | タスク | 工数（人日） |
|---------|--------|-------------|
| **Phase 1** | **開発タスク** | |
| 1.1 | バックエンド（cart-microservice） | 1.25 |
| 1.2 | バックエンド（api-gateway） | 0.75 |
| 1.3 | 在庫チェック機能（オプション） | 2.5 |
| 1.4 | フロントエンド（React） | 3.0 |
| 1.5 | エラーハンドリング | 1.0 |
| | **Phase 1 小計** | **8.5** (在庫チェックあり) / **6.0** (なし) |
| **Phase 2** | **テスト** | |
| 2.1 | バックエンド単体テスト | 2.0 |
| 2.2 | フロントエンド単体テスト | 1.5 |
| 2.3 | 統合テスト | 2.0 |
| | **Phase 2 小計** | **5.5** |
| **Phase 3** | **設計書作成** | |
| 3.1 | 画面設計書 | 1.0 |
| 3.2 | API設計書 | 0.5 |
| 3.3 | DB設計書確認 | 0.25 |
| 3.4 | シーケンス図 | 0.5 |
| | **Phase 3 小計** | **2.25** |
| | | |
| | **合計（在庫チェックあり）** | **16.25人日** |
| | **合計（在庫チェックなし）** | **13.75人日** |

### バッファを含めた見積もり

| 項目 | 工数 |
|------|------|
| 基本工数（在庫チェックあり） | 16.25人日 |
| 基本工数（在庫チェックなし） | 13.75人日 |
| バッファ（20%） | 3.25人日 / 2.75人日 |
| コードレビュー・修正 | 2.0人日 |
| デプロイ・動作確認 | 1.0人日 |
| **最終見積もり（在庫チェックあり）** | **22.5人日** |
| **最終見積もり（在庫チェックなし）** | **19.5人日** |

---

## Notes

### 実装上の重要ポイント

1. **既存の `addProduct` / `removeProduct` との整合性**
   - 新規APIは既存APIと共存
   - 既存の+1/-1操作は引き続き利用可能

2. **トランザクション管理**
   - `@Transactional` アノテーションによる整合性確保
   - 楽観的ロックの検討（同時更新対策）

3. **パフォーマンス考慮**
   - 数量変更時のAPI呼び出し頻度制限（デバウンス処理）
   - フロントエンドでの楽観的UI更新

4. **セキュリティ**
   - ユーザー認証の確認
   - 他ユーザーのカート操作防止

5. **在庫チェック機能について**
   - オプション機能として分離
   - 実装する場合は `checkout-microservice` との連携が必要
   - 在庫不足時のユーザーフィードバック設計が重要
