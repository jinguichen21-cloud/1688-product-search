---
name: 1688-product-search
description: >-
 1688商品搜索SKILL：提供完整的1688商品搜索能力，包括类目查询、关键词搜索、图片搜索、商品详情等9个核心接口。
 支持多语言搜索和商品推荐，使用1688开放平台官方API，统一鉴权，Token全局缓存共享。
metadata:
  version: 1.0.2
  label: 1688商品搜索
  author: 1688官方技术团队
  openclaw:
    primaryEnv: ALI1688_APP_KEY, ALI1688_APP_SECRET, ALI1688_REFRESH_TOKEN
    requires:
      env:
        - ALI1688_APP_KEY
        - ALI1688_APP_SECRET
        - ALI1688_REFRESH_TOKEN
---

# 1688商品搜索SKILL

通过1688开放平台官方API提供完整的商品搜索能力，包含9个核心接口。

## 鉴权说明

每个 Skill 内置独立的鉴权模块（`scripts/auth.py`），**不依赖任何外部 Skill**。

所有 1688 Skill 的 Token 缓存指向同一个固定路径，实现"独立运行 + 鉴权只发生一次"。

- Token 缓存路径: `skills/.1688_token_cache.json`（所有 1688 Skill 共用）
- 任意一个 Skill 首次请求完成鉴权后，其他 Skill 直接复用缓存
- Token 过期前自动用 refresh_token 刷新
- 支持 `ALI1688_REFRESH_TOKEN`（自动刷新）和 `ALI1688_ACCESS_TOKEN`（直接使用）两种模式

### 配置

在 OpenClaw config 中设置环境变量：

```json5
{
 skills: {
 entries: {
 "1688-product-search": {
 env: {
 ALI1688_APP_KEY: "your-app-key",
 ALI1688_APP_SECRET: "your-app-secret", 
 ALI1688_REFRESH_TOKEN: "your-refresh-token"
 }
 }
 }
 }
}
```

### 如何获取 AppKey / AppSecret / Token

如果遇到 Token 相关错误（如 401、签名失败、Token 过期），按以下步骤操作：

#### Step 1：注册开发者 & 创建应用 → 获取 AppKey + AppSecret

1. 打开 [1688开放平台](https://open.1688.com)，用1688账号登录
2. 进入 [控制中心](https://open.1688.com/console)
3. 点击「我的应用」→「创建应用」
4. 填写应用信息，提交审核
5. 审核通过后，在应用详情页可以看到 **AppKey** 和 **AppSecret**

#### Step 2：订购解决方案 → 获取 API 调用权限

1. 打开 [跨境ERP/独立站SaaS数字化解决方案](https://open.1688.com/solution/solutionDetail.htm?solutionKey=1697015308755)
2. 点击「立即订购」，将解决方案绑定到你的应用
3. 订购成功后，应用才有权限调用方案内的 API

#### Step 3：用户授权 → 获取 access_token + refresh_token

1. 在浏览器中访问授权页面（替换 YOUR_APPKEY 和 YOUR_REDIRECT_URI）：
 ```
 https://auth.1688.com/oauth/authorize?client_id=YOUR_APPKEY&site=1688&redirect_uri=YOUR_REDIRECT_URI
 ```
2. 用1688账号登录并同意授权
3. 页面会跳转到你的回调地址，URL 中带有 `code` 参数
4. 用 code 换取 Token（有效期短，需在10分钟内使用）：
 ```bash
 curl -X POST "https://gw.open.1688.com/openapi/param2/1/system.oauth2/getToken/YOUR_APPKEY" \
 -d "grant_type=authorization_code" \
 -d "need_refresh_token=true" \
 -d "client_id=YOUR_APPKEY" \
 -d "client_secret=YOUR_APPSECRET" \
 -d "redirect_uri=YOUR_REDIRECT_URI" \
 -d "code=授权码"
 ```
5. 返回结果中包含：
 - `access_token` — 用于调用 API（有效期约10小时）
 - `refresh_token` — 用于刷新 access_token（有效期约半年）

#### Step 4：配置到环境变量

- `ALI1688_APP_KEY` = 应用的 AppKey
- `ALI1688_APP_SECRET` = 应用的 AppSecret
- `ALI1688_REFRESH_TOKEN` = 上一步获得的 refresh_token（推荐，支持自动刷新）
- `ALI1688_ACCESS_TOKEN` = 上一步获得的 access_token（备用，过期需手动换）

#### 常见 Token 错误及解决

| 错误 | 原因 | 解决方案 |
|------|------|---------|
| `HTTP 400` 刷新失败 | refresh_token 无效或已过期 | 重新走 Step 3 授权，获取新的 refresh_token |
| `HTTP 401` 未授权 | access_token 过期或无效 | 设置 ALI1688_REFRESH_TOKEN 启用自动刷新 |
| `签名错误(code=25)` | AppSecret 不正确 | 检查 ALI1688_APP_SECRET 是否与应用详情页一致 |
| `无权限调用` | 未订购解决方案 | 回到 Step 2 订购对应解决方案 |
| `refresh_token 半年后过期` | Token 自然过期 | 重新走 Step 3 授权 |

#### 参考链接

- [1688开放平台 - 控制中心](https://open.1688.com/console)
- [API 调用说明](https://open.1688.com/doc/apiInvoke.htm)
- [签名规则](https://open.1688.com/doc/signature.htm)
- [授权说明](https://open.1688.com/doc/apiAuth.htm)
- [解决方案订购](https://open.1688.com/solution/solutionDetail.htm?solutionKey=1697015308755)

## 使用方法

### 1. 类目查询
```bash
# 查询所有一级类目（英语）
python3 scripts/product_search.py category 0

# 查询中文类目
python3 scripts/product_search.py category 0 --language en
```

### 2. 多语言关键词搜索
```bash
# 英文关键词搜索
python3 scripts/product_search.py keyword-search "dress" --country en

# 中文关键词搜索
python3 scripts/product_search.py keyword-search "连衣裙" --country en

# 带筛选条件的搜索
python3 scripts/product_search.py keyword-search "dress" --country en --filter "shipIn48Hours,shipIn24Hours" --sort '{"price":"asc"}'
```

### 3. 多语言图片搜索
```bash
# 使用imageId进行图片搜索
python3 scripts/product_search.py image-search "your_image_id" --country en

# 上传图片获取imageId
python3 scripts/product_search.py upload-image "/path/to/your/image.jpg"
```

### 4. 多语言商品详情
```bash
# 查询单个商品详情
python3 scripts/product_search.py product-detail "offer_id"

# 查询多个商品详情
python3 scripts/product_search.py product-detail "offer_id1,offer_id2,offer_id3"
```

### 5. 多语言商品店铺搜索
```bash
# 根据商家ID搜索商品
python3 scripts/product_search.py shop-search "seller_open_id" --country en
```

### 6. 多语言商品推荐
```bash
# 基于关键词的商品推荐
python3 scripts/product_search.py offer-recommend "keyword" --country en
```

### 7. 多语言搜索导航
```bash
# 获取搜索导航信息
python3 scripts/product_search.py search-navigation "keyword" --country en
```

### 8. 相关性商品推荐
```bash
# 基于商品ID的相关推荐
python3 scripts/product_search.py related-recommend "offer_id" --country en
```

### 9. 上传图片获取imageId
```bash
# 上传本地图片获取imageId
python3 scripts/product_search.py upload-image "/path/to/image.jpg"
```

**智能图片压缩功能**：当上传的图片文件大于300KB时，系统会自动进行智能压缩，确保图片大小符合1688 API的要求。压缩过程会：
- 自动检测图片格式并转换为JPEG（如果需要）
- 保持最佳画质的同时将文件大小控制在300KB以内
- 临时生成压缩后的图片用于上传，完成后自动清理临时文件
- 输出详细的压缩日志（原大小 → 压缩后大小）

这确保了无论用户提供的图片大小如何，都能成功获取有效的imageId用于后续的图片搜索操作。

## 参数说明

### 通用参数
| 参数 | 说明 | 默认值 | 可选值 |
|------|------|--------|--------|
| `--country` / `--language` | 语言代码 | `en` | `en` / `ja` / `ko` / `ru` / `vi` / `es` 等 |
| `--beginPage` | 起始页码 | `1` | 数字 |
| `--pageSize` | 每页数量 | `20` | 数字，最大50 |

**注意**：当接口参数中包含 `beginPage` 时，默认传 `1`；包含 `pageSize` 时，默认传 `20`；包含 `country` 或 `language` 时，默认传 `en`。

### 筛选条件 (filter)
支持多种筛选条件，多个条件用英文逗号分割：
- `shipIn24Hours` - 24小时发货
- `shipIn48Hours` - 48小时发货  
- `certifiedFactory` - 认证工厂
- `isOnePsale` - 支持一件代发
- `new7` - 7天上新
- `1688Selection` - 1688严选

示例：`--filter "shipIn48Hours,certifiedFactory,isOnePsale"`

### 排序参数 (sort)
支持按不同维度排序：
- `price` - 批发价
- `rePurchaseRate` - 复购率  
- `monthSold` - 月销量

示例：`--sort '{"price":"asc"}'` 或 `--sort '{"monthSold":"desc"}'`

## 输出格式

JSON 格式，直接返回1688 API 的原始响应数据。

**重要提示：所有商品查询结果都会包含商品ID（offerId字段），这是商品的唯一标识符，可用于后续的商品详情查询或其他操作。**

### 错误处理
- **失败时不会返回mock数据**：当API调用失败、参数错误或网络异常时，会直接返回错误信息JSON并退出
- **错误格式**：`{"error": "具体的错误信息"}`
- **退出码**：失败时返回退出码1，成功时返回0

### 商品结果字段说明
- `offerId` - **商品ID（必显）**：商品的唯一标识符
- `subject` - 商品标题（中文）
- `subjectTrans` - 商品标题（英文翻译）
- `imageUrl` - 商品主图URL
- `priceInfo` - 价格信息（包含批发价和代发价）
- `monthSold` - 月销量
- `repurchaseRate` - 复购率
- `promotionURL` - 商品详情页链接

## API 接口地址

| 接口 | 完整URL |
|------|---------|
| 类目查询 | `POST https://gw.open.1688.com/openapi/param2/1/com.alibaba.fenxiao.crossborder/category.translation.getById/${APPKEY}` |
| 关键词搜索 | `POST https://gw.open.1688.com/openapi/param2/1/com.alibaba.fenxiao.crossborder/product.search.keywordQuery/${APPKEY}` |
| 图片搜索 | `POST https://gw.open.1688.com/openapi/param2/1/com.alibaba.fenxiao.crossborder/product.search.imageQuery/${APPKEY}` |
| 商品详情 | `POST https://gw.open.1688.com/openapi/param2/1/com.alibaba.fenxiao.crossborder/product.search.queryProductDetail/${APPKEY}` |
| 店铺搜索 | `POST https://gw.open.1688.com/openapi/param2/1/com.alibaba.fenxiao.crossborder/product.search.shopSearch/${APPKEY}` |
| 商品推荐 | `POST https://gw.open.1688.com/openapi/param2/1/com.alibaba.fenxiao.crossborder/product.search.offerRecommend/${APPKEY}` |
| 搜索导航 | `POST https://gw.open.1688.com/openapi/param2/1/com.alibaba.fenxiao.crossborder/product.search.keywordSNQuery/${APPKEY}` |
| 相关推荐 | `POST https://gw.open.1688.com/openapi/param2/1/com.alibaba.fenxiao.crossborder/product.related.recommend/${APPKEY}` |
| 图片上传 | `POST https://gw.open.1688.com/openapi/param2/1/com.alibaba.fenxiao.crossborder/product.image.upload/${APPKEY}` |

## 1688接口通用说明

### API接入要点
- **语言支持**: 默认使用 `country=en`，但返回字段包含中英双语
  - 中文字段示例: `subject`  
  - 英文字段示例: `subjectTrans`
- **Access Token**: 当前解决方案产生的access_token是**长久有效**的
- **筛选条件**: 支持多种商品筛选条件（发货时效、认证工厂、一件代发等）
- **排序功能**: 支持按价格/复购率/月销量排序，但仅对当前页有效

### 重要限制
- **数据量限制**: 每个查询最多返回2000个商品
- **图片搜索**: 仅推荐使用1688图片地址，其他域名成功率不稳定  
- **价格显示**: API返回的是原价，下单时会享受营销价格

### 服务列表映射
- `sendGoods24H` → **24小时发货**
- `sendGoods48H` → **48小时发货**

## API 参考文档

完整的 API 接口和数据结构文档请参阅 [references/api.md](references/api.md)。