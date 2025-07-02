# ドメイン名（Domain）
注文ドメイン（Order Domain）  
- 顧客からの注文の管理を担当するドメイン。注文の作成、更新、状態管理を行う。

---

# ユビキタス言語（Ubiquitous Language）
| 用語       | 定義・説明                          |
|------------|-----------------------------------|
| 注文（Order） | 顧客が購入を希望する商品の集まり   |
| 注文明細（OrderItem） | 注文内の個別商品と数量           |
| 注文状態（OrderStatus） | 注文の現在の状態（例：新規、発送済）|

---

# コンテキスト境界（Bounded Context）
- 注文管理コンテキスト  
- 注文の作成から配送完了までの一連の処理を管理。  
- 支払い管理コンテキスト、在庫管理コンテキストと連携。

---

# ドメインモデル概要図（Optional）
- Order — 1..* — OrderItem  
- Order 状態管理（OrderStatus）

---

# エンティティ（Entity）
| 名称     | 説明                             |
|----------|--------------------------------|
| Order    | 顧客の注文を表し、一意なIDで識別|

### エンティティ詳細

#### Order
- 識別子（ID）: OrderID (UUID)  
- 属性一覧:  
  | 属性名       | 型           | 説明                | 制約               |
  |--------------|--------------|---------------------|--------------------|
  | customerId   | UUID         | 顧客ID              | 必須               |
  | orderDate    | DateTime     | 注文日時            | 必須               |
  | status       | OrderStatus  | 注文状態            | 初期値: 新規注文    |
- 振る舞い（メソッド・操作）:  
  - addOrderItem(item: OrderItem)  
  - cancelOrder()  
  - changeStatus(newStatus: OrderStatus)  
- 不変条件（Invariant）:  
  - 注文状態は「キャンセル済」以降は変更不可  
  - 注文明細は必ず1件以上保持  
- 備考:  

---

# 値オブジェクト（Value Object）
| 名称          | 説明                             |
|---------------|--------------------------------|
| OrderItem     | 注文内の商品の情報と数量       |

### 値オブジェクト詳細

#### OrderItem
- 属性一覧:  
  | 属性名      | 型       | 説明             | 制約       |
  |-------------|----------|------------------|------------|
  | productId   | UUID     | 商品ID           | 必須       |
  | quantity    | int      | 注文数量         | 1以上      |
  | price       | decimal  | 単価             | 0以上      |
- 振る舞い（メソッド・操作）:  
  - getTotalPrice() : price * quantity  
- 不変条件（Immutable）:  
  - 作成後、属性は変更不可  
- 備考:  

---

# アグリゲート（Aggregate）
| 名称     | 説明                             |
|----------|--------------------------------|
| Order    | 注文明細（OrderItem）を集約し、一貫性を保つ|

### アグリゲート詳細

#### Order
- ルートエンティティ: Order  
- 関連エンティティ・値オブジェクト: OrderItem（複数）  
- 不変条件（集約ルール）:  
  - Orderの状態に応じて、OrderItemの追加・変更可否を制御  
- 備考:  

---

# リポジトリ（Repository）
| 名称       | 説明                             |
|------------|--------------------------------|
| OrderRepository | Orderの永続化操作を管理       |

### リポジトリ詳細

#### OrderRepository
- インターフェース定義（主な操作）:  
  - save(order: Order): void  
  - findById(orderId: UUID): Order  
  - findByCustomerId(customerId: UUID): List<Order>  
- 実装の注意点・技術選定:  
  - RDB（PostgreSQL）を使用予定。ORMで実装。  
- 備考:  

---

# ドメインサービス（Domain Service）
| 名称         | 説明                             |
|--------------|--------------------------------|
| OrderValidationService | 注文のビジネスルール検証を提供 |

### ドメインサービス詳細

#### OrderValidationService
- 主な操作・メソッド:  
  - validateOrder(order: Order): boolean  
- 入力・出力:  
  - 入力：Orderオブジェクト  
  - 出力：検証結果（真偽値）  
- 不変条件:  
  - 注文内容の整合性チェックを担う  
- 備考:  

---

# アプリケーションサービス（Application Service）
| 名称           | 説明                             |
|----------------|--------------------------------|
| OrderAppService | 注文関連ユースケース実装        |

### アプリケーションサービス詳細

#### OrderAppService
- 主な操作・メソッド:  
  - createOrder(customerId, items)  
  - cancelOrder(orderId)  
- 使用するドメインモデル・リポジトリ:  
  - Order, OrderRepository, OrderValidationService  
- トランザクション境界:  
  - ユースケース単位でトランザクション開始・終了  
- 備考:  

---

# イベント（Domain Event）
| 名称           | 説明                             |
|----------------|--------------------------------|
| OrderPlaced    | 注文が確定したことを通知         |

### イベント詳細

#### OrderPlaced
- 発生タイミング:  
  - 注文作成完了時  
- 発生条件:  
  - 注文内容が有効であることの検証後  
- ペイロード（イベントデータ）:  
  - orderId, customerId, orderDate  
- 備考:  

---

# 画面・API仕様（オプション）
- APIエンドポイント例  
  - POST /orders  
  - GET /orders/{orderId}  
- 入力例:  
```json
{
  "customerId": "uuid-string",
  "items": [
    {"productId": "uuid-string", "quantity": 2, "price": 1000}
  ]
}
```

- 出力例:
```json
{
  "orderId": "uuid-string",
  "status": "NEW",
  "orderDate": "2025-07-02T12:34:56Z"
}

```

---

# 備考・補足

- 注文のキャンセルや状態変更は業務ルールに厳格に従うこと。
- 将来的に配送管理サービスと連携するためのイベント設計も検討中。

---
