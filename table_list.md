# テーブル一覧設計書

## 1. テーブル一覧表

| No. | テーブル名 | テーブル概要 |
|-----|-----------|-------------|
| 1   | orders    | 注文情報を管理する |
| 2   | product_inventory | 商品の在庫数量を管理する |
| 3   | product_rankings | 商品のカテゴリ別売上ランキングを管理する |
| 4   | products  | 商品の基本情報を管理する |
| 5   | shopping_cart | ユーザーのショッピングカート情報を管理する |

## 2. ER図

```mermaid
erDiagram
    orders {
        order_id text PK
        user_id text
        order_details text
        order_time text
        order_total double
    }

    product_inventory {
        asin text PK
        quantity int
    }

    product_rankings {
        asin text PK
        category text PK
        sales_rank int
        title text
        price double
        imurl text
        num_reviews int
        num_stars int
        avg_stars double
    }

    products {
        asin text PK
        title text
        description text
        price double
        imurl text
        also_bought frozen_list_text
        also_viewed frozen_list_text
        bought_together frozen_list_text
        buy_after_viewing frozen_list_text
        brand text
        categories set_text
        num_reviews int
        num_stars int
        avg_stars double
    }

    shopping_cart {
        cart_key TEXT PK
        user_id TEXT
        asin TEXT
        time_added TEXT
        quantity INT
    }
```
