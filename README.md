# 真实地址生成器

中文 / [English](README_EN.md)

这是一个基于 Cloudflare Workers 的真实地址生成器。项目当前采用**单文件 Worker 架构**，同时提供两套访问方式：

- **公开页面 `/`**：面向普通访问者，无需授权，由 Worker 在服务端生成数据并直接渲染 HTML。
- **受保护 API `/api/v1/address`**：面向开发者、服务端程序或外部受控客户端，必须通过 Bearer Token 访问。

> 说明：公开页面与受保护 API 是两套访问方式，不应混用。不要在 HTML 模板中内嵌 `API_TOKEN`。如果未来确实需要浏览器端直接调用受保护接口，应单独设计登录态或会话方案，而不是复用静态 API Key。

## 演示地址
![](https://github.com/Adonis142857/Real-Address-Generator/blob/main/example.png)

在线使用：[https://realaddress.fuyiran.com/](https://realaddress.fuyiran.com/)

## 功能

- 生成地图上存在的随机真实地址
- 支持多国家选项
- 使用谷歌地图显示地址位置
- 生成随机的手机号、姓名和性别，并可以直接点击复制
- 支持保存生成的信息并添加备注
- 同时提供公开页面与受保护开发者接口

## 访问方式说明

### 1. 公开页面 `/`

- **路径**：`/`
- **授权要求**：不需要
- **使用场景**：普通用户在浏览器中访问页面
- **渲染方式**：由 Worker 服务端生成地址数据并输出 HTML
- **可选参数**：`country`

示例：

```text
GET /?country=US
```

### 2. 受保护 API `/api/v1/address`

- **路径**：`/api/v1/address`
- **授权要求**：需要 Bearer Token
- **适用对象**：服务端、自动化程序、外部受控客户端
- **不建议**：在公开网页前端直接暴露静态 Token 调用

#### 请求头格式

```http
Authorization: Bearer <API_TOKEN>
```

#### 示例请求

```bash
curl "https://your-worker.example.workers.dev/api/v1/address?country=US" \
  -H "Authorization: Bearer your-api-token"
```

#### 成功返回示例

```json
{
  "success": true,
  "data": {
    "country": "US",
    "name": "John Smith",
    "gender": "Male",
    "phone": "+1 (415) 555-1234",
    "address": "123 Main St, San Francisco, 94105, US"
  }
}
```

#### 失败返回示例

未提供或提供了错误的 Token：

```json
{
  "success": false,
  "error": {
    "code": "unauthorized",
    "message": "Missing or invalid Bearer token"
  }
}
```

`country` 参数不合法：

```json
{
  "success": false,
  "error": {
    "code": "invalid_country",
    "message": "country must be one of the supported ISO-like country codes",
    "supportedCountries": ["US", "UK", "FR"]
  }
}
```

#### `country` 参数说明

`country` 为可选于公开页面、必填于受保护 API 的国家代码参数。当前支持以下值：

`US`、`UK`、`FR`、`DE`、`CN`、`TW`、`HK`、`JP`、`IN`、`AU`、`BR`、`CA`、`RU`、`ZA`、`MX`、`KR`、`IT`、`ES`、`TR`、`SA`、`AR`、`EG`、`NG`、`ID`

说明：

- 公开页面未传 `country` 时，Worker 会随机选择国家。
- 受保护 API 未传或传入不支持的 `country` 时，会返回错误。
- 建议调用方始终显式传递 `country`，以便获得稳定结果。

## 部署方法

要部署此网站，您需要使用 Cloudflare Workers 部署提供的 JavaScript 代码。

1. 注册或登录您的 [Cloudflare](https://www.cloudflare.com/) 账户。
2. 创建一个新的 Cloudflare Worker。
3. 将 `worker.js` 复制到 Cloudflare Worker 脚本编辑器中。
4. 为受保护 API 配置 Worker Secret / Variable：`API_TOKEN`。
5. 保存并部署您的 Worker。

### 部署建议

- 把 `/` 保持为公开页面，继续使用服务端渲染 HTML。
- 把 `/api/v1/address` 视为开发者接口，仅允许受控调用方通过 Bearer Token 访问。
- 不要在 HTML 模板、前端脚本或静态资源中暴露 `API_TOKEN`。
- 如果未来需要浏览器端调用受保护接口，请单独引入登录态、会话或签名机制，不要直接复用静态 API Key。

## 反馈

如果您在使用过程中遇到任何问题，请提交 [issues](https://github.com/Adonis142857/Real-Address-Generator/issues)。

## 许可证

此项目使用 MIT 许可证。

## 版权声明

初代版本来自 chatgpt.org.uk，由 Adonis142857 进行修改。
