# Flutter Coding Style

本章定義 Flutter 專案的預設目錄、模組邊界與依賴方向。中型以上專案原則上採用 **Feature-first**；小型專案則依實際複雜度漸進拆層，不為了形式建立沒有責任的空資料夾。

## 本 repository 的預設規範

1. 新功能優先放在 `lib/features/<feature_name>/`，依產品能力而不是頁面型態切分。
2. Feature 內部可依複雜度採用 `presentation`、`application`、`domain`、`data` 分層，但不強制每層都存在。
3. Feature 外部只依賴其公開 API，不直接 import 其他 Feature 的 `data/`、內部 Controller、DTO 或實作類別。
4. 無產品語意的基礎設施放在 `core/`；確實被多個 Feature 共用的 App 元件放在 `shared/`。
5. 不因為兩處看起來相似就立即抽共用；確認責任相同且有穩定重用需求後再移入 `shared/`。
6. 測試、assets 與 i18n 應保留 Feature 所有權，讓功能可獨立理解、測試、替換或移除。

## Feature-first 架構

### Feature-first 是什麼？

**Feature-first 是依照「產品功能／使用者行為」來組織程式碼，而不是依照技術類型分類。**

例如一個購物 App 有：

- 登入
- 商品
- 購物車
- 訂單
- 會員資料

Feature-first 會先依功能切：

```text
lib/
└── features/
    ├── auth/
    ├── products/
    ├── cart/
    ├── orders/
    └── profile/
```

每個 feature 裡，再放這個功能自己的：

- UI
- State
- Domain model
- Repository
- API
- Database
- Tests

核心概念是：

> 修改「購物車」功能時，大部分需要閱讀和修改的檔案，都應該集中在 `features/cart/` 裡。

---

## Feature-first 與 Layer-first 的差異

### 傳統 Layer-first

很多 Flutter 專案一開始會這樣分：

```text
lib/
├── models/
├── pages/
├── widgets/
├── controllers/
├── repositories/
├── services/
└── utils/
```

購物車功能可能分散在：

```text
models/cart_item.dart
pages/cart_page.dart
widgets/cart_item_tile.dart
controllers/cart_controller.dart
repositories/cart_repository.dart
services/cart_api_service.dart
```

這種分類對小專案很直覺，但專案變大後，問題會逐漸出現：

1. 修改一個功能，要在很多資料夾之間跳來跳去。
2. 很難知道哪些檔案屬於同一個功能。
3. 刪除功能時，很容易遺漏檔案。
4. 團隊成員容易同時修改相同的大型資料夾。
5. `services`、`utils`、`common` 最後容易變成垃圾場。
6. 功能邊界不清楚，任何地方都可能互相 import。

---

### Feature-first

相同功能改成：

```text
lib/
└── features/
    └── cart/
        ├── cart_page.dart
        ├── cart_controller.dart
        ├── cart_state.dart
        ├── cart_item.dart
        ├── cart_repository.dart
        ├── cart_api.dart
        └── widgets/
            └── cart_item_tile.dart
```

所有與購物車相關的內容集中在一起。

這會讓開發者很容易回答：

- 購物車功能在哪裡？
- 哪些檔案屬於購物車？
- 購物車依賴什麼？
- 如何刪除或替換購物車？
- 哪些測試涵蓋購物車？

---

## Feature-first 不等於不分層

這是最容易誤解的地方。

Feature-first 解決的是：

> 最外層資料夾如何分類。

Clean Architecture、MVVM、BLoC、Riverpod 等則處理：

> Feature 裡面的責任如何分層。

所以兩者可以同時存在。

例如：

```text
lib/
└── features/
    └── cart/
        ├── presentation/
        ├── application/
        ├── domain/
        └── data/
```

這可以稱為：

> Feature-first + Layered Architecture

或：

> Feature-first Clean Architecture

---

## 一個實用的 Flutter Feature-first 結構

本規範建議以下結構：

```text
lib/
├── app/
│   ├── app.dart
│   ├── router/
│   │   └── app_router.dart
│   ├── bootstrap/
│   │   └── bootstrap.dart
│   └── dependencies/
│       └── app_dependencies.dart
│
├── core/
│   ├── network/
│   ├── database/
│   ├── error/
│   ├── logging/
│   ├── analytics/
│   ├── config/
│   └── extensions/
│
├── shared/
│   ├── widgets/
│   ├── theme/
│   ├── localization/
│   └── models/
│
└── features/
    ├── auth/
    ├── products/
    ├── cart/
    ├── orders/
    └── profile/
```

三個主要區域的責任：

| 區域 | 用途 |
|---|---|
| `app/` | App 啟動、路由、全域 DI、App shell |
| `core/` | 網路、資料庫、Log、Error 等基礎設施 |
| `features/` | 實際產品功能 |
| `shared/` | 確實被多個 feature 共用的元件 |

---

## Feature 內部怎麼分？

這要看專案規模。

### 小型 Feature：不要過度分層

例如設定頁面只有幾個 API 和狀態：

```text
features/
└── settings/
    ├── settings_page.dart
    ├── settings_controller.dart
    ├── settings_state.dart
    ├── settings_repository.dart
    └── widgets/
        └── theme_selector.dart
```

這樣就夠了。

不需要為了「架構看起來漂亮」，硬拆成二十個檔案：

```text
data/
domain/
application/
presentation/
usecases/
entities/
repositories/
datasources/
mappers/
```

Feature-first 的目的本來就是提高凝聚度，不是增加樣板程式碼。

---

### 中大型 Feature：再進行分層

例如訂單功能包含：

- 訂單列表
- 訂單詳情
- 取消訂單
- 退款
- 訂單狀態同步
- 本地快取

可以分成：

```text
features/
└── orders/
    ├── presentation/
    │   ├── pages/
    │   │   ├── order_list_page.dart
    │   │   └── order_detail_page.dart
    │   ├── widgets/
    │   └── controllers/
    │       ├── order_list_controller.dart
    │       └── order_detail_controller.dart
    │
    ├── application/
    │   ├── get_orders.dart
    │   ├── cancel_order.dart
    │   └── observe_order_status.dart
    │
    ├── domain/
    │   ├── entities/
    │   │   └── order.dart
    │   ├── repositories/
    │   │   └── order_repository.dart
    │   └── failures/
    │       └── order_failure.dart
    │
    ├── data/
    │   ├── datasources/
    │   │   ├── order_remote_data_source.dart
    │   │   └── order_local_data_source.dart
    │   ├── dtos/
    │   │   └── order_dto.dart
    │   ├── mappers/
    │   │   └── order_mapper.dart
    │   └── repositories/
    │       └── order_repository_impl.dart
    │
    └── orders.dart
```

---

## 各層的責任

### Presentation

放 Flutter UI 和畫面狀態：

```text
presentation/
├── pages/
├── widgets/
├── controllers/
├── providers/
└── states/
```

可以使用：

- Flutter Widget
- Riverpod
- BLoC
- ChangeNotifier
- ValueNotifier

例如：

```dart
class OrderListPage extends ConsumerWidget {
  const OrderListPage({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final state = ref.watch(orderListControllerProvider);

    return state.when(
      loading: () => const CircularProgressIndicator(),
      error: (error, stackTrace) => Text(error.toString()),
      data: (orders) => ListView.builder(
        itemCount: orders.length,
        itemBuilder: (context, index) {
          return OrderTile(order: orders[index]);
        },
      ),
    );
  }
}
```

---

### Application

放使用案例、流程協調、操作意圖。

例如：

```dart
class CancelOrder {
  const CancelOrder(this._repository);

  final OrderRepository _repository;

  Future<void> call(String orderId) {
    return _repository.cancelOrder(orderId);
  }
}
```

Application 層適合處理：

- 取消訂單
- 建立訂單
- 送出付款
- 驗證登入狀態
- 組合多個 repository
- 執行完整使用流程

不應該依賴具體 Widget。

---

### Domain

放純業務規則：

```dart
class Order {
  const Order({
    required this.id,
    required this.status,
    required this.totalAmount,
  });

  final String id;
  final OrderStatus status;
  final int totalAmount;

  bool get canCancel {
    return status == OrderStatus.pending ||
        status == OrderStatus.confirmed;
  }
}
```

Repository interface 也可以放 Domain：

```dart
abstract interface class OrderRepository {
  Future<List<Order>> getOrders();

  Future<Order> getOrder(String orderId);

  Future<void> cancelOrder(String orderId);
}
```

Domain 最理想的狀態是：

- 不依賴 Dio
- 不依賴 Firebase
- 不依賴 Flutter Widget
- 不知道資料來自 REST、SQLite 或 Mock
- 可以單獨進行 unit test

但這不是絕對規定。小型 Flutter 專案不一定要追求完全純 Dart。

---

### Data

放具體資料來源：

```dart
class ApiOrderRepository implements OrderRepository {
  const ApiOrderRepository(this._api);

  final OrderApi _api;

  @override
  Future<List<Order>> getOrders() async {
    final dtos = await _api.getOrders();

    return dtos.map((dto) => dto.toDomain()).toList();
  }

  @override
  Future<void> cancelOrder(String orderId) {
    return _api.cancelOrder(orderId);
  }

  @override
  Future<Order> getOrder(String orderId) async {
    final dto = await _api.getOrder(orderId);
    return dto.toDomain();
  }
}
```

Data 層可能包含：

- REST API
- GraphQL
- Firebase
- SQLite
- SharedPreferences
- Secure Storage
- DTO
- JSON serialization
- Cache
- Repository implementation

---

## 最重要的是 Feature 邊界

Feature-first 不是只把資料夾名稱改掉。

真正重要的是：

> 每個 Feature 都要有明確的邊界與責任。

例如：

### Auth Feature

負責：

- 登入
- 登出
- Token
- Session
- 身分驗證狀態

```text
features/auth/
```

### Profile Feature

負責：

- 姓名
- 頭像
- 個人資料編輯

```text
features/profile/
```

不要因為「會員資料跟登入都有 User」，就把所有東西塞進：

```text
features/user/
```

最後 `user` 很容易變成超大型模組，包含：

- 登入
- 訂閱
- 帳務
- 個人資料
- 權限
- 通知
- 設定

這樣就失去 Feature-first 的價值。

---

## Feature 怎麼切？不要只看頁面

Feature 不一定等於一個 Screen。

例如：

```text
features/
├── authentication/
├── shopping_cart/
├── checkout/
├── order_history/
└── notification_settings/
```

這些是使用者可以理解的產品能力。

比較不推薦：

```text
features/
├── home_page/
├── detail_page/
├── list_page/
└── dialog/
```

因為這是用 UI 形態切，不是用業務能力切。

可以用下面三個問題判斷：

1. 使用者會怎麼稱呼這個功能？
2. 產品經理會如何描述這項需求？
3. 這個功能是否有相對獨立的資料、狀態和流程？

例如產品需求是：

> 使用者可以把商品加入購物車、調整數量並移除商品。

這就是明確的 `cart` feature。

---

## Feature 之間如何溝通？

這是實踐 Feature-first 時最需要控制的部分。

假設 Checkout 需要知道購物車內容，最直接但不理想的方式是：

```dart
import 'package:app/features/cart/presentation/controllers/cart_controller.dart';
import 'package:app/features/cart/data/cart_repository_impl.dart';
```

這代表 Checkout 直接知道 Cart 的內部實作。

更好的做法是讓 Feature 提供公開介面。

例如：

```text
features/
└── cart/
    ├── src/
    │   ├── presentation/
    │   ├── domain/
    │   └── data/
    └── cart.dart
```

`cart.dart`：

```dart
library;

export 'src/domain/entities/cart.dart';
export 'src/domain/entities/cart_item.dart';
export 'src/application/cart_service.dart';
```

其他 Feature 只能：

```dart
import 'package:app/features/cart/cart.dart';
```

不要直接：

```dart
import 'package:app/features/cart/src/data/cart_repository_impl.dart';
```

這種概念與 Dart package 的 `lib/src` 很接近。

---

## 建議的依賴方向

可以遵循：

```text
Presentation
     ↓
Application
     ↓
Domain
     ↑
Data
```

更精確地說：

```text
presentation → application
presentation → domain
application → domain
data → domain
domain → 不依賴其他層
```

Repository interface：

```text
domain/repositories/order_repository.dart
```

Repository implementation：

```text
data/repositories/order_repository_impl.dart
```

因此 Domain 不需要知道資料來源是 Dio、Firebase 還是本地資料庫。

---

## Riverpod 專案如何實踐

假設有商品 Feature：

```text
features/
└── products/
    ├── data/
    │   ├── product_api.dart
    │   └── product_repository_impl.dart
    ├── domain/
    │   ├── product.dart
    │   └── product_repository.dart
    ├── application/
    │   └── get_products.dart
    └── presentation/
        ├── products_page.dart
        ├── product_tile.dart
        └── products_controller.dart
```

Provider 可以放在 Feature 裡：

```dart
final productApiProvider = Provider<ProductApi>((ref) {
  return ProductApi(ref.watch(dioProvider));
});

final productRepositoryProvider = Provider<ProductRepository>((ref) {
  return ProductRepositoryImpl(
    ref.watch(productApiProvider),
  );
});

final productsControllerProvider =
    AsyncNotifierProvider<ProductsController, List<Product>>(
  ProductsController.new,
);
```

Controller：

```dart
class ProductsController extends AsyncNotifier<List<Product>> {
  @override
  Future<List<Product>> build() {
    final repository = ref.watch(productRepositoryProvider);
    return repository.getProducts();
  }

  Future<void> refresh() async {
    state = const AsyncLoading();

    state = await AsyncValue.guard(() {
      final repository = ref.read(productRepositoryProvider);
      return repository.getProducts();
    });
  }
}
```

所有商品相關 provider 仍然放在商品 Feature 裡，不需要集中到：

```text
lib/providers/
```

否則又退回 Layer-first。

---

## 路由怎麼處理？

有兩種常見做法。

### 方式一：路由集中管理

適合中小型專案：

```text
app/
└── router/
    └── app_router.dart
```

```dart
GoRoute(
  path: '/orders',
  builder: (_, __) => const OrderListPage(),
),
```

優點是完整路由一眼可見。

---

### 方式二：每個 Feature 提供自己的 Routes

適合大型專案：

```text
features/orders/
├── orders.dart
└── presentation/
    └── routes/
        └── order_routes.dart
```

```dart
abstract final class OrderRoutes {
  static final routes = <RouteBase>[
    GoRoute(
      path: '/orders',
      builder: (_, __) => const OrderListPage(),
      routes: [
        GoRoute(
          path: ':orderId',
          builder: (_, state) {
            return OrderDetailPage(
              orderId: state.pathParameters['orderId']!,
            );
          },
        ),
      ],
    ),
  ];
}
```

App Router 只做組合：

```dart
final router = GoRouter(
  routes: [
    ...AuthRoutes.routes,
    ...ProductRoutes.routes,
    ...OrderRoutes.routes,
  ],
);
```

---

## Shared 和 Core 怎麼區分？

這兩個資料夾最容易失控。

### Core

通常是沒有產品語意的基礎設施：

```text
core/
├── network/
│   ├── dio_client.dart
│   └── auth_interceptor.dart
├── database/
├── error/
├── logging/
├── analytics/
├── config/
└── platform/
```

判斷方式：

> 就算把產品從購物 App 換成新聞 App，這個東西還可能繼續使用嗎？

若答案是「可能」，它比較像 Core。

例如：

- HTTP client
- Logger
- Crash reporting
- Secure storage wrapper
- Date formatter
- App exception

---

### Shared

通常是多個 Feature 共同使用，而且帶有一些 App 層語意：

```text
shared/
├── widgets/
│   ├── app_button.dart
│   ├── error_view.dart
│   └── loading_view.dart
├── theme/
├── localization/
└── models/
```

但不要一看到兩個地方使用，就馬上移到 Shared。

比較好的原則是：

> 重複三次，而且責任真的相同，再抽出共用。

因為看起來相同的兩個 Widget，未來可能走向不同需求。

---

## 不要讓 Shared 成為垃圾場

以下名稱要特別小心：

```text
shared/
common/
utils/
helpers/
services/
managers/
```

不是不能用，而是每個檔案必須有清楚責任。

不推薦：

```text
utils/
└── app_utils.dart
```

裡面同時有：

```dart
formatDate()
showDialog()
saveToken()
calculatePrice()
trackEvent()
```

推薦拆成：

```text
core/
├── formatting/
│   └── date_formatter.dart
├── storage/
│   └── token_storage.dart
├── analytics/
│   └── analytics_service.dart
└── pricing/
    └── price_calculator.dart
```

其中 `pricing` 如果是特定購物業務規則，更應該放在：

```text
features/checkout/domain/services/price_calculator.dart
```

---

## Feature 是否可以依賴其他 Feature？

可以，但需要節制。

例如：

```text
checkout → cart
checkout → payment
checkout → address
```

這在業務上很合理。

但不建議直接依賴對方內部的：

- Controller
- Widget
- DTO
- API implementation
- Database table

應依賴對方公開提供的：

- Domain entity
- Application service
- Public provider
- Feature API

例如 Checkout 不該直接讀：

```dart
cartController.state.items
```

可以改成依賴：

```dart
abstract interface class CartReader {
  Future<Cart> getCurrentCart();
}
```

或更簡單：

```dart
final cart = ref.read(currentCartProvider);
```

前提是 `currentCartProvider` 是 Cart Feature 刻意公開的 API，而不是內部實作細節。

---

## Feature-first 的測試結構

測試也可以照 Feature 分：

```text
test/
└── features/
    └── orders/
        ├── domain/
        │   └── order_test.dart
        ├── application/
        │   └── cancel_order_test.dart
        ├── data/
        │   └── order_repository_impl_test.dart
        └── presentation/
            └── order_list_page_test.dart
```

或者直接把測試放在 Feature 附近，若團隊工具和規範允許：

```text
features/
└── orders/
    ├── lib/
    └── test/
```

大型 monorepo、Melos 或多 package 架構更適合後者。

---

## Assets 與 i18n 也可以 Feature 化

例如訂單專屬圖片：

```text
assets/
└── features/
    └── orders/
        ├── empty_orders.webp
        └── order_success.webp
```

翻譯 key 可以加 Feature namespace：

```json
{
  "orders": {
    "title": "我的訂單",
    "empty": "目前沒有訂單",
    "cancel": "取消訂單"
  }
}
```

避免所有 key 混成：

```text
title
emptyText
buttonText
cancelTitle
```

---

## Feature-first 常見錯誤

### 1. 一個頁面就是一個 Feature

例如：

```text
features/
├── home_page/
├── product_list_page/
├── product_detail_page/
└── cart_page/
```

Product list 與 Product detail 很可能都屬於：

```text
features/products/
```

應該是：

```text
features/
└── products/
    └── presentation/
        └── pages/
            ├── product_list_page.dart
            └── product_detail_page.dart
```

---

### 2. Feature 切得太細

例如：

```text
features/
├── login_button/
├── password_field/
├── email_validator/
└── login_form/
```

這些不是 Feature，而是 Auth Feature 的內部元件：

```text
features/
└── auth/
    └── presentation/
        └── widgets/
            ├── login_button.dart
            ├── password_field.dart
            └── login_form.dart
```

---

### 3. Feature 切得太大

例如：

```text
features/
└── user/
```

裡面有登入、訂閱、通知、設定、付款、個人資料。

這種應該拆成：

```text
features/
├── auth/
├── profile/
├── subscription/
├── notification_settings/
└── payment_methods/
```

---

### 4. 所有 Entity 都放 Shared

例如：

```text
shared/models/
├── user.dart
├── product.dart
├── order.dart
└── cart.dart
```

這會模糊模型所有權。

通常應該是：

```text
features/auth/domain/auth_session.dart
features/products/domain/product.dart
features/orders/domain/order.dart
features/cart/domain/cart.dart
```

只有真正跨 Feature、語意穩定的模型才適合抽出。

---

### 5. 每個 Feature 都強制四層

一個只有靜態說明頁的 Feature：

```text
features/about/
├── presentation/
├── application/
├── domain/
└── data/
```

可能完全沒必要。

可以只有：

```text
features/about/
└── about_page.dart
```

等複雜度出現後再拆。

---

### 6. Feature 之間任意 import

例如：

```dart
import '../../cart/data/cart_repository_impl.dart';
import '../../profile/presentation/profile_controller.dart';
```

這會使功能高度耦合。

應建立公開入口：

```dart
import 'package:app/features/cart/cart.dart';
import 'package:app/features/profile/profile.dart';
```

---

## 實際導入步驟

假設目前專案是：

```text
lib/
├── pages/
├── widgets/
├── models/
├── repositories/
└── services/
```

不要一次全部重構。可以從下一個功能開始。

### 第一步：列出產品功能

例如：

```text
auth
home
products
cart
checkout
orders
profile
settings
```

---

### 第二步：選一個邊界清楚的 Feature

例如 `orders`：

```text
lib/features/orders/
```

先把新訂單功能放進去。

---

### 第三步：搬移該 Feature 的相關檔案

從：

```text
pages/order_list_page.dart
models/order.dart
repositories/order_repository.dart
services/order_api.dart
widgets/order_tile.dart
```

搬成：

```text
features/orders/
├── presentation/
│   ├── order_list_page.dart
│   └── widgets/
│       └── order_tile.dart
├── domain/
│   ├── order.dart
│   └── order_repository.dart
└── data/
    ├── order_api.dart
    └── order_repository_impl.dart
```

---

### 第四步：建立公開 API

```text
features/orders/orders.dart
```

```dart
library;

export 'presentation/order_list_page.dart';
export 'domain/order.dart';
```

只 export 其他模組真的需要知道的內容。

不要直接 export 整個 Feature：

```dart
export 'data/order_api.dart';
export 'data/order_repository_impl.dart';
export 'presentation/controllers/internal_order_state.dart';
```

---

### 第五步：限制跨 Feature import

可以訂團隊規範：

1. Feature 外部只能 import 該 Feature 的 public barrel。
2. 不可 import 其他 Feature 的 `data/`。
3. 不可 import 其他 Feature 的內部 Controller。
4. 共用程式碼必須有明確 owner。
5. 新增到 `shared/` 前必須確認至少兩至三個真實使用者。

例如：

```dart
// 建議
import 'package:app/features/orders/orders.dart';

// 不建議
import 'package:app/features/orders/data/repositories/order_repository_impl.dart';
```

大型專案可以使用：

- `import_lint`
- `custom_lint`
- Melos package boundaries
- Dart package 的 `lib/src`
- Dependency graph check

來強制依賴規則。

---

## 建議的漸進式結構

一般 Flutter 商業 App 不建議一開始就套用最完整的 Clean Architecture。

可以從：

```text
features/
└── cart/
    ├── cart_page.dart
    ├── cart_controller.dart
    ├── cart_state.dart
    ├── cart_repository.dart
    └── widgets/
```

隨複雜度成長後，再演進成：

```text
features/
└── cart/
    ├── presentation/
    ├── application/
    ├── domain/
    └── data/
```

判斷是否需要拆層，可以看：

- Repository 是否已有多種實作。
- API DTO 與畫面 Model 是否開始不同。
- 業務規則是否逐漸複雜。
- Controller 是否變成巨大類別。
- 測試是否很難撰寫。
- Feature 是否由多位工程師共同維護。

---

## 語音學習 App 範例

例如語音學習 App，可以考慮：

```text
lib/
├── app/
│   ├── app.dart
│   ├── router/
│   └── bootstrap/
│
├── core/
│   ├── network/
│   ├── audio/
│   ├── permissions/
│   ├── storage/
│   └── ai/
│
├── shared/
│   ├── design_system/
│   ├── widgets/
│   └── localization/
│
└── features/
    ├── authentication/
    ├── onboarding/
    ├── speaking_practice/
    ├── speech_recording/
    ├── pronunciation_analysis/
    ├── conversation_history/
    ├── subscription/
    └── settings/
```

需要注意：

`speech_recording` 是否應獨立成 Feature，要看它是產品功能還是基礎能力。

如果它只是一個底層錄音引擎，可能更適合：

```text
core/audio/
```

如果使用者真的有：

- 開始錄音
- 暫停
- 重錄
- 播放
- 查看錄音記錄
- 管理錄音檔

那它就可能是完整 Feature：

```text
features/speech_recording/
```

判斷原則是：

> 它是使用者可感知的業務能力，還是支援其他功能的技術能力？

---

## 最實用的判斷口訣

可以記成四句：

> 先依業務功能切資料夾。  
> 再依複雜度決定是否分層。  
> Feature 內高凝聚，Feature 間低耦合。  
> 只有真正共用的東西才放 Core 或 Shared。

---

## 建議的標準版本

一般中型 Flutter 專案建議採用：

```text
lib/
├── app/
├── core/
├── shared/
└── features/
    └── feature_name/
        ├── data/
        ├── domain/
        ├── application/
        ├── presentation/
        └── feature_name.dart
```

但不強制每個資料夾都存在。例如某 Feature 沒有 application use case，就不要建立空資料夾。

最終目標不是追求固定目錄模板，而是做到：

1. 看到路徑就知道程式碼屬於哪個功能。
2. 修改功能時，影響範圍集中。
3. Feature 不依賴其他 Feature 的內部細節。
4. 功能可以被獨立測試、替換或移除。
5. 專案成長後仍能清楚找到責任歸屬。
