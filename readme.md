# Laravel Shop

基于 Laravel 5.7 的 B2C 电商系统，支持支付宝/微信双支付通道，包含完整的前台购物流程和后台管理面板。

## 功能特性

### 前台（用户端）

- **用户系统** — 注册、登录、邮箱验证、密码重置
- **商品浏览** — 商品列表、搜索、排序（价格/销量/评分）、分页
- **商品详情** — SKU 选择、库存展示、用户评价展示
- **购物车** — 加入购物车、数量修改、移除商品、全选/取消
- **订单管理** — 创建订单、订单列表、订单详情、确认收货
- **支付系统** — 支付宝网页支付、微信扫码支付（二维码）、支付回调
- **退款系统** — 用户申请退款、退款状态跟踪
- **商品评价** — 订单评价（评分+文字）、评价后自动更新商品评分
- **商品收藏** — 收藏/取消收藏、收藏列表
- **收货地址** — 省市区三级联动、地址增删改查

### 后台（管理端，基于 laravel-admin）

- **Dashboard** — 系统环境、扩展、依赖信息面板
- **用户管理** — 用户列表查看
- **商品管理** — 商品 CRUD、SKU 管理（一对多嵌套表单）、富文本编辑器、图片上传
- **订单管理** — 订单列表、订单详情查看、发货（物流公司+单号）、退款审批

### 系统特性

- **队列任务** — 超时未支付自动关闭订单（`CloseOrder` Job），自动释放库存
- **事件/监听** — 支付成功后更新商品销量、发送邮件通知；评价后更新商品平均评分
- **权限策略** — 订单归属校验（`OrderPolicy`）、地址归属校验（`UserAddressPolicy`）
- **表单验证** — 自定义 Request 类，含复杂业务逻辑校验（库存检查、权限校验等）
- **自定义异常** — `InvalidRequestException`（400 用户端错误）、`InternalException`（500 系统内部错误）
- **服务层** — `CartService`、`OrderService` 封装核心业务逻辑

## 技术栈

| 层面 | 技术 |
|------|------|
| 后端框架 | Laravel 5.7 / PHP 7.1+ |
| 前端 | Bootstrap 4 + jQuery + Vue.js |
| 后台管理 | encore/laravel-admin 1.6.8 |
| 支付 | yansongda/pay 2.6（支付宝 + 微信支付） |
| 二维码 | endroid/qr-code 3.5 |
| 省市区数据 | china-area-data 4.0 |
| 数据库 | MySQL |
| 缓存/队列 | Redis（predis） |
| 构建工具 | Laravel Mix (Webpack) |
| 中文语言包 | overtrue/laravel-lang 3.0 |

## 项目结构

```
laravel-shop/
├── app/
│   ├── Admin/                    # 后台管理模块（laravel-admin）
│   │   ├── Controllers/          #   管理后台控制器（订单/商品/用户）
│   │   ├── routes.php            #   后台路由
│   │   └── bootstrap.php         #   Admin 启动配置
│   ├── Events/                   # 事件
│   │   ├── OrderPaid.php              订单支付成功
│   │   └── OrderReviewed.php          订单评价完成
│   ├── Exceptions/                # 自定义异常
│   │   ├── InvalidRequestException.php  无效请求 (400)
│   │   └── InternalException.php       内部异常 (500)
│   ├── Http/
│   │   ├── Controllers/          # 前台控制器
│   │   │   ├── CartController.php       购物车
│   │   │   ├── OrdersController.php     订单
│   │   │   ├── PaymentController.php    支付宝+微信支付+回调
│   │   │   ├── ProductsController.php   商品列表/详情/收藏
│   │   │   ├── UserAddressesController.php 收货地址
│   │   │   └── PagesController.php      静态页面
│   │   ├── Requests/               # 表单验证
│   │   │   ├── AddCartRequest.php         加入购物车
│   │   │   ├── OrderRequest.php           创建订单
│   │   │   ├── ApplyRefundRequest.php     申请退款
│   │   │   ├── SendReviewRequest.php      发送评价
│   │   │   ├── UserAddressRequest.php     地址表单
│   │   │   └── Admin/HandleRefundRequest.php 后台退款处理
│   │   └── Middleware/              # 中间件（Laravel 内置）
│   ├── Jobs/                      # 队列任务
│   │   └── CloseOrder.php              超时未支付自动关闭订单
│   ├── Listeners/                 # 事件监听器
│   │   ├── SendOrderPaidMail.php       支付成功 → 发邮件
│   │   ├── UpdateProductSoldCount.php  支付成功 → 更新销量
│   │   └── UpdateProductRating.php     评价完成 → 更新评分
│   ├── Models/                    # Eloquent 模型
│   │   ├── User.php                   用户（含收藏/购物车关联）
│   │   ├── Product.php                商品（SKU 一对多）
│   │   ├── ProductSku.php             商品 SKU（含加减库存方法）
│   │   ├── Order.php                  订单（含状态常量/订单号生成）
│   │   ├── OrderItem.php              订单商品
│   │   ├── CartItem.php               购物车商品
│   │   └── UserAddress.php            收货地址
│   ├── Notifications/             # 通知
│   │   └── OrderPaidNotification.php   支付成功邮件通知
│   ├── Policies/                  # 授权策略
│   │   ├── OrderPolicy.php             订单归属校验
│   │   └── UserAddressPolicy.php       地址归属校验
│   └── Services/                  # 业务逻辑服务层
│       ├── CartService.php             购物车逻辑
│       └── OrderService.php            订单创建逻辑
├── bootstrap/
│   └── helpers.php               # 全局辅助函数（route_class）
├── config/
│   └── pay.php                   # 支付宝/微信支付配置
├── database/
│   ├── factories/                # 模型工厂（6 个）
│   ├── migrations/               # 数据库迁移（10 个）
│   └── seeds/                    # 数据填充（商品 Seeder）
├── resources/
│   ├── js/
│   │   ├── app.js                    # Vue 应用入口
│   │   └── components/
│   │       ├── SelectDistrict.js      省市区三级联动组件
│   │       └── UserAddressesCreateAndEdit.js 地址编辑组件
│   ├── sass/                     # SASS 样式
│   └── views/                    # Blade 模板（21 个）
│       ├── layouts/                  页面布局
│       ├── products/                 商品页面
│       ├── cart/                     购物车页面
│       ├── orders/                   订单页面
│       ├── user_addresses/           地址页面
│       ├── auth/                     认证页面
│       ├── pages/                    静态/结果页面
│       └── admin/                    后台视图
├── routes/
│   ├── web.php                   # 前台路由
│   └── api.php                   # API 路由
└── tests/                        # 测试文件
```

## 数据库设计

```
users                        # 用户表
├── user_addresses            # 收货地址（1:N）
├── cart_items                # 购物车（1:N）
├── user_favorite_products    # 收藏（M:N）
└── orders                    # 订单（1:N）
    └── order_items           # 订单商品（1:N）

products                     # 商品表
└── product_skus              # 商品 SKU（1:N）
```

核心表说明：

| 表名 | 说明 |
|------|------|
| `users` | 用户（name, email, email_verified_at） |
| `user_addresses` | 收货地址（省/市/区/详细地址/邮编/联系人/电话） |
| `products` | 商品（标题/描述/封面图/上架状态/评分/销量/评价数/价格） |
| `product_skus` | 商品 SKU（标题/描述/价格/库存） |
| `cart_items` | 购物车项（用户+SKU+数量） |
| `orders` | 订单（订单号/地址/金额/支付信息/退款状态/物流状态/关闭标记） |
| `order_items` | 订单商品（SKU+数量/价格/评分/评价） |
| `user_favorite_products` | 用户收藏商品关联表 |

## 快速开始

### 环境要求

- PHP >= 7.1.3
- MySQL >= 5.7
- Redis（队列驱动可选）
- Composer
- Node.js + NPM

### 安装步骤

```bash
# 1. 克隆项目
git clone <repository-url>
cd laravel-shop

# 2. 安装 PHP 依赖
composer install

# 3. 安装前端依赖
npm install

# 4. 环境配置
cp .env.example .env
php artisan key:generate

# 5. 配置 .env 中的数据库连接
# DB_DATABASE=laravel_shop
# DB_USERNAME=root
# DB_PASSWORD=your_password

# 6. 运行数据库迁移
php artisan migrate

# 7. 填充测试数据（可选）
composer dump-autoload
php artisan db:seed --class=ProductsSeeder

# 8. 编译前端资源
npm run dev

# 9. 启动开发服务器
php artisan serve
```

### 访问地址

| 入口 | URL |
|------|-----|
| 前台首页 | `http://localhost:8000` |
| 后台管理 | `http://localhost:8000/admin` |

### 支付配置

支付相关配置位于 `config/pay.php`，需要填入实际的支付凭证：

- **支付宝**：`app_id`、`ali_public_key`、`private_key`
- **微信支付**：`app_id`、`mch_id`、`key`、`cert_client`、`cert_key`

> 微信支付回调地址需配置为外网可访问的 URL，开发环境可使用 ngrok 等内网穿透工具。

### 队列配置

默认队列驱动为 `sync`（同步），如需使用异步队列自动关闭超时订单：

```env
QUEUE_CONNECTION=redis
```

订单超时时间默认 300 秒（5 分钟），可在 `config/app.php` 中修改 `order_ttl`。

## 事件机制

```
OrderPaid (订单支付成功)
  ├── UpdateProductSoldCount → 更新商品销量
  └── SendOrderPaidMail      → 发送支付成功邮件

OrderReviewed (订单评价完成)
  └── UpdateProductRating    → 更新商品平均评分和评价数

CloseOrder (队列任务，延迟执行)
  └── 关闭未支付订单 + 释放 SKU 库存
```

## License

MIT

<p align="center">Made with ❤️ by PanNinan</p>
