# テーブル一覧設計書

## 1. テーブル一覧表

| No. | テーブル名 | テーブル概要 |
|-----|-----------|-------------|
| 1 | orders | 注文情報を管理する |
| 2 | product_inventory | 商品の在庫数量を管理する |
| 3 | product_rankings | 商品のカテゴリ別売上ランキングを管理する |
| 4 | products | 商品の基本情報を管理する |
| 5 | shopping_cart | ショッピングカートの内容を管理する |

## 2. ER図

```mermaid
erDiagram
    products {
        text asin PK
        text title
        text description
        double price
        text imurl
        list also_bought
        list also_viewed
        list bought_together
        list buy_after_viewing
        text brand
        set categories
        int num_reviews
        int num_stars
        double avg_stars
    }

    product_rankings {
        text asin PK
        text category PK
        int sales_rank
        text title
        double price
        text imurl
        int num_reviews
        int num_stars
        double avg_stars
    }

    product_inventory {
        text asin PK
        int quantity
    }

    orders {
        text order_id PK
        text user_id
        text order_details
        text order_time
        double order_total
    }

    shopping_cart {
        text cart_key PK
        text user_id
        text asin
        text time_added
        int quantity
    }
```
