# 七牛云 MLS API 鉴权指南

本文档介绍三种常用的 MLS API 鉴权方式：

1. AK/SK 签名鉴权（QiniuToken）
2. IAM 子账号鉴权
3. Bearer Token 鉴权

每种方式都提供签名生成规则和请求示例。

---

## 1 AK/SK 签名鉴权（QiniuToken）

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
> 注意：1.<Method> 与 <Path> 中间有一个空格

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

### 1.5 请求示例
ak = test1
sk = test2
QiniuToken = "Qiniu test1:KI-VgUTKszBmF2b0r3ssQMbnA5Q="

```bash
curl -v -XPOST 'http://mls.cn-east-1.qiniumiku.com/?apikey' \
-d '{"name": "test"}' \
-H 'Authorization: Qiniu test1:KI-VgUTKszBmF2b0r3ssQMbnA5Q=' \
-H 'Host: mls.cn-east-1.qiniumiku.com' \
-H 'Content-Type: application/json'
```
> 

**Go 语言示例：**

```go
package main

import (
	"crypto/hmac"
	"crypto/sha1"
	"encoding/base64"
	"fmt"
	"io"
	"net/http"
	"net/url"
	"os"
	"strings"
	"time"
)

func main() {
	// 从环境变量获取Access Key和Secret Key
	ak := os.Getenv("Access_key")
	sk := os.Getenv("Secret_key")

	// 请求体
	body := `{"data":{"uri":"http://oayjpradp.bkt.clouddn.com/Audrey_Hepburn.jpg"},"params":{"scenes":["pulp","terror","politician","ads"]}}`

	// 构建HTTP请求
	targetURL := "http://ai.qiniuapi.com/v3/image/censor"

	// 生成签名并发送请求
	signature := generateSignature("POST", targetURL, body, ak, sk)
	fmt.Printf("生成的签名: %s\n", signature)

	response, err := sendHttpRequest(targetURL, "POST", body, signature, 2)
	if err != nil {
		fmt.Printf("请求失败: %v\n", err)
		return
	}
	fmt.Printf("响应内容: %s\n", response)
}

// generateSignature 生成七牛签名
func generateSignature(method, urlStr, body, ak, sk string) string {
	parsedURL, err := url.Parse(urlStr)
	if err != nil {
		panic(err)
	}

	// 构建签名数据
	var data strings.Builder
	data.WriteString(method)
	data.WriteString(" ")
	data.WriteString(parsedURL.Path)

	if parsedURL.RawQuery != "" {
		data.WriteString("?")
		data.WriteString(parsedURL.RawQuery)
	}

	data.WriteString("\nHost: ")
	data.WriteString(parsedURL.Host)
	data.WriteString("\nContent-Type: application/json")

	if body != "" {
		data.WriteString("\n\n")
		data.WriteString(body)
	}

	// 使用HMAC-SHA1进行签名
	hmacSha1 := hmac.New(sha1.New, []byte(sk))
	hmacSha1.Write([]byte(data.String()))
	hmacResult := hmacSha1.Sum(nil)

	sign := "Qiniu " + ak + ":" + base64UrlSafeEncode(hmacResult)
	return sign
}

// sendHttpRequest 发送HTTP请求
func sendHttpRequest(urlStr, method, data, signature string, timeout int) (string, error) {
	client := &http.Client{
		Timeout: time.Duration(timeout) * time.Second,
	}

	var bodyReader io.Reader
	if data != "" {
		bodyReader = strings.NewReader(data)
	}

	req, err := http.NewRequest(method, urlStr, bodyReader)
	if err != nil {
		return "", err
	}

	req.Header.Set("Content-Type", "application/json")
	req.Header.Set("Authorization", signature)

	resp, err := client.Do(req)
	if err != nil {
		return "", err
	}
	defer resp.Body.Close()

	bodyBytes, err := io.ReadAll(resp.Body)
	if err != nil {
		return "", err
	}

	return fmt.Sprintf("HTTP %d: %s", resp.StatusCode, string(bodyBytes)), nil
}

// base64UrlSafeEncode URL安全的base64编码
func base64UrlSafeEncode(data []byte) string {
	encoded := base64.StdEncoding.EncodeToString(data)
	encoded = strings.ReplaceAll(encoded, "+", "-")
	encoded = strings.ReplaceAll(encoded, "/", "_")
	return encoded
}

```

## 2 Bearer Token 鉴权（API Key）

通过管理员 AK/SK 生成 API Key，然后使用 **Bearer Token** 调用 API。

### 2.1 创建 API Key

```bash
curl -v -XPOST 'http://mls.cn-east-1.qiniumiku.com/?apikey' \
-d '{"name": "name"}' \
-H 'Authorization: Qiniu <AK/SK签名生成的Token>' \
-H 'Content-Type: application/json'
```

> 注意：name 自定义，长度 1~20，内容不做校验。

### 2.2 使用 API Key 调用接口

```bash
curl https://mls.cn-east-1.qiniumiku.com/stream?info=test \
-H "Authorization: Bearer <Miku API Key>"
```
> 可以通过调用 api 获取创建的 apikey 列表或者通过 [miku portal](https://portal.qiniu.com/mikustream/api-key) 创建对应的 apikey


## 3 IAM 子账号鉴权

IAM 子账号可访问 Miku 的 S3 风格接口，签名方式与 AK/SK 相同。

### 3.1 S3 风格接口示例

- 普通接口：  
```
https://domain/{bucket}/{streamtitle}/path?params={params}&params2={params2}
```

- S3 风格接口：  
```
https://{bucket}.domain/{streamtitle}?path&params={params}&params2={params2}
```

### 3.2 请求示例

```bash
curl -v 'http://mls.cn-east-1.qiniumiku.com/?trafficStats&begin=20240101000000&end=20240129105148&g=5min&select=flow&flow=downflow' \
-H 'Content-Type: application/json' \
-H 'Authorization: Qiniu <IAM-AccessKey>:<signToken>'
```

> 注意：签名生成方式与 AK/SK QiniuToken 完全一致，只是 AccessKey 和 SecretKey 使用子账号凭证。

---

### 总结

1. **AK/SK 签名**：管理类 API 使用，生成 QiniuToken  
2. **IAM 子账号**：适用于 S3 风格接口  
3. **Bearer Token**：基于 API Key 调用接口  

**关键点**：

- Method / Path / Query / Host / Content-Type / Body 必须与签名一致  
- HMAC-SHA1 输出必须 URL-safe Base64 编码