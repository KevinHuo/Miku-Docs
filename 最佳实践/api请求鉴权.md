# 七牛云 MLS API 鉴权指南

本文档介绍三种常用的 MLS API 鉴权方式：

1. AK/SK 签名鉴权（QiniuToken）
2. IAM 子账号鉴权
3. Bearer Token 鉴权

每种方式都提供签名生成规则和请求示例。

---

## 1️⃣ AK/SK 签名鉴权（QiniuToken）

七牛云管理 API 请求必须经过 AK/SK 签名生成 **QiniuToken**，并在 `Authorization` 头中使用。  

### 1.1 获取密钥

1. 登录 [七牛开发者平台](https://portal.qiniu.com/)  
2. 打开 **个人中心 → 密钥管理**  
3. 获取 **AccessKey** 和 **SecretKey**  

> 注意：密钥非常重要，请勿放在客户端或公开仓库。

### 1.2 构造待签名字符串

签名字符串规则：

```text
data = <Method> + " " + <Path> + "?" + <RawQuery> + "\nHost: " + <Host> + "\nContent-Type: " + <contentType> + "\n\n" + <bodyStr>
```
> 注意：<Method> 与 <Path> 中间有一个空格

**参数说明：**

| 参数 | 必填 | 说明 |
|------|------|------|
| Method | 是 | HTTP 方法：GET/POST/PUT/DELETE，大写 |
| Path | 是 | 请求路径，如 `/v2/hubs/PiliSDKTest/streams/xxx` |
| RawQuery | 否 | URL 查询参数，如无可不加 `?` |
| Host | 是 | 域名，如 `mls.cn-east-1.qiniumiku.com` |
| Content-Type | 否 | 请求体类型，如为空可不加该字段 |
| bodyStr | 否 | 请求体内容，如有且 Content-Type 不为空且不为 `application/octet-stream`，需加入 |

> 注意：无论 Content-Type 或 body 是否为空，中间都必须保留 `\n\n`。

**Go 语言示例：**

```go
data = "<Method> <Path>"
if "<RawQuery>" != "" {
    data += "?" + "<RawQuery>"
}
data += "\nHost: <Host>"
if "<Content-Type>" != "" {
    data += "\nContent-Type: <Content-Type>"
}
data += "\n\n"
if "<Body>" != "" && "<Content-Type>" != "" && "<Content-Type>" != "application/octet-stream" {
    data += "<Body>"
}
```

### 1.3 计算签名

使用 **HMAC-SHA1** 算法并进行 URL-safe Base64 编码：

```go
sign = hmac_sha1(data, "Your_Secret_Key")
encodedSign = urlsafe_base64_encode(sign)
```

### 1.4 生成 QiniuToken

```text
QiniuToken = "Qiniu " + "<AccessKey>" + ":" + "<encodedSign>"
```

**示例：**

```
Authorization: Qiniu 7O7hf7Ld1RrC_fpZdFvU8aCgOPuhw2K4eapYOdII:PGTUV-oRxWAIl6mdneJPSJieyyQ=
```

### 1.5 请求示例

```bash
curl -v -XPOST 'http://mls.cn-east-1.qiniumiku.com/?apikey' \
-d '{"name": "yydounaui"}' \
-H 'Authorization: Qiniu <YourToken>' \
-H 'Host: mls.cn-east-1.qiniumiku.com' \
-H 'Content-Type: application/json'
```


## 2️⃣ IAM 子账号鉴权

IAM 子账号可访问 Miku 的 S3 风格接口，签名方式与 AK/SK 相同。

### 2.1 S3 风格接口示例

- 普通接口：  
```
https://domain/{bucket}/{streamtitle}/path?params={params}&params2={params2}
```

- S3 风格接口：  
```
https://{bucket}.domain/{streamtitle}?path&params={params}&params2={params2}
```

### 2.2 请求示例

```bash
curl -v 'http://mls.cn-east-1.qiniumiku.com/?trafficStats&begin=20240101000000&end=20240129105148&g=5min&select=flow&flow=downflow' \
-H 'Content-Type: application/json' \
-H 'Authorization: Qiniu <IAM-AccessKey>:<signToken>'
```

> 注意：签名生成方式与 AK/SK QiniuToken 完全一致，只是 AccessKey 和 SecretKey 使用子账号凭证。


## 3️⃣ Bearer Token 鉴权（API Key）

通过管理员 AK/SK 生成 API Key，然后使用 **Bearer Token** 调用 API。

### 3.1 创建 API Key

```bash
curl -v -XPOST 'http://mls.cn-east-1.qiniumiku.com/?apikey' \
-d '{"name": "name"}' \
-H 'Authorization: Qiniu <AK/SK签名生成的Token>' \
-H 'Content-Type: application/json'
```

### 3.2 使用 API Key 调用接口

```bash
curl https://mls.cn-east-1.qiniumiku.com/stream?info=test \
-H "Authorization: Bearer <Miku API Key>"
```

> 注意：name 长度 1~20，内容不做校验。

---

### 总结

1. **AK/SK 签名**：管理类 API 使用，生成 QiniuToken  
2. **IAM 子账号**：适用于 S3 风格接口  
3. **Bearer Token**：基于 API Key 调用接口  

**关键点**：

- Method / Path / Query / Host / Content-Type / Body 必须与签名一致  
- HMAC-SHA1 输出必须 URL-safe Base64 编码