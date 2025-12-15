# 从 Pili 迁移到 Miku（MikuStream）指南

## 迁移前注意事项

在开始迁移之前，请注意以下重要事项：

- **海外域名限制**：当前暂不支持覆盖海外区域的域名迁移，请联系客服获取解决方案
- **推流 SDK 兼容性**：如果需要使用 PILI 推流 SDK 请联系技术支持
- **控制台操作限制**：目前云导播和 Pub 转推功能尚未开放 API，需通过控制台手动操作

> **操作指引**：
> - 导播台：https://portal.qiniu.com/mikustream/caster
> - Pub 转推：https://portal.qiniu.com/mikustream/pub/tasks

# 迁移步骤详解

## 1. 创建直播空间

1. 登录 [Miku 控制台](https://portal.qiniu.com/mikustream/buckets)
2. 新建直播空间

![新建直播空间](https://docs.qnsdk.com/miku_8f3c91b2e7d04a8fbe4a2ddc9c1d57af.png)

## 2. 域名绑定

将原 Pili 推拉流域名迁移至 Miku 平台

![绑定域名](https://docs.qnsdk.com/miku_d1f47c9ab2384e79a4c6e1f50b8f94e2.png)

**Miku 推流协议支持：**
```
# RTMP 推流
rtmp://<domain>/<bucket>/<stream>

# SRT 推流
srt://<domain>:1935?streamid=#!::h=<bucket>/<stream>,m=publish,domain=<domain>

# WHIP 推流
https://<domain>/<bucket>/<stream>.whip
```

**Miku 拉流协议支持：**
```
# RTMP 拉流
rtmp://<domain>/<bucket>/<stream>

# WHEP 拉流
http://<domain>/<bucket>/<stream>.whep

# HLS 拉流
http://<domain>/<bucket>/<stream>.m3u8

# FLV 拉流
http://<domain>/<bucket>/<stream>.flv

# SRT 拉流
srt://<domain>:1935?streamid=#!::h=<bucket>/<stream>,m=request,domain=<domain>

# DASH 拉流
http://<domain>/<bucket>/<stream>.mpd
```

> **性能优化建议**：经测试，Miku 使用 FLV 格式播放性能最优

> **备案要求**：直播域名需完成 ICP 备案和公安备案。如不确定备案流程，请参考[备案帮助文档](https://developer.qiniu.com/af/kb/3987/how-to-make-website-and-inquires-the-police-put-on-record-information)

## 3. CNAME 配置

1. 获取 Miku 平台分配的域名和 CNAME
2. 在您的 DNS 管理平台配置该 CNAME 记录

![cname](https://docs.qnsdk.com/miku_7ab39d84c2ef4a46b1da3f9c0e5d17bf.png)

## 4. 录制配置

![录制设置](https://docs.qnsdk.com/miku_3f9a7be214df4962a8730bc59d42e8aa.png)

> **重要提示**：与 Pili 不同，Miku 在控制台中暂不支持存储过期时间设置，需要调用 [修改空间配置](https://developer.qiniu.com/mikustream/13096/mikustream-live-bucket-config-update-api#recording-object) API 进行设置

## 5. 推流鉴权迁移

如原 Pili 配置了推流鉴权，迁移至 Miku 时需调整签名算法。具体方法请参考：[Miku 鉴权文档](https://developer.qiniu.com/mikustream/13172/mikustream-live-authentication)

## 6. 拉流鉴权迁移

如原 Pili 配置了拉流鉴权，仅需将过期时间参数从 16 进制调整为 10 进制，其余逻辑保持不变。

**PILI:**
![PILI](https://docs.qnsdk.com/miku_ae49c0d13f7b45c0b82e5f4d09a8c73d.png)

**MIKU:**
![MIKU](https://docs.qnsdk.com/miku_59e4c3ab7fdc49f88a3b15d607e1c4fb.png)

## 7. 创建直播流

![创建直播流](https://docs.qnsdk.com/miku_7d9b2e14c0fa4c0db8135ff92a6d3b8c.png)

生成推流地址和拉流地址用于直播推流和观看

## 高级功能说明

### URL 重写规则

Pili 采用 break 模式（匹配即终止），而 Miku 采用 continue 模式（匹配后继续），使用 URL 重写时需特别注意此差异

### 直播流管理
MIKU 默认开启 **自动创建流**，默认关闭 **自动删除过期流** 

![直播流管理](https://docs.qnsdk.com/miku_d4b1f3a9c2e6471b9f0a2e5c7d8b3f6a.png)

在 MIKU 内目前不支持设置开关

# 服务端 API 迁移指南

API 详细文档请参考：[Miku API 概览](https://developer.qiniu.com/mikustream/12890/mikustream-live-an-overview-of-the-api)

> 与 PILI 相比，MIKU API 的接口路径、入参、出参存在较大差异。建议在调用前参考官方文档。以下为整体调用差异对比

## 接口域名替换

- **直播空间管理**：`pili.qiniuapi.com` → `mls.cn-east-1.qiniumiku.com`
- **功能接口**：`pili.qiniuapi.com` → `<bucket>.mls.cn-east-1.qiniumiku.com`

## 鉴权方案选项

### 方案 A：AK/SK（QiniuToken）

- 可复用现有 Pili 签名代码
- 修改签名串和 Host 为 `mls.<region>.qiniumiku.com`
- [AK/SK 认证文档](https://developer.qiniu.com/mikustream/12893/mikustream-live-http-requests-authentication#2)

### 方案 B：Bearer API Key

1. 创建 API Key：`POST http://mls.<region>.qiniumiku.com/?apikey`
2. 请求鉴权：`Authorization: Bearer <Miku API Key>`
- [Bearer Token 认证文档](https://developer.qiniu.com/mikustream/12893/mikustream-live-http-requests-authentication#1)

### 方案 C：IAM 子账号

- 使用 `AccessKey: IAM-xxxx` + `SecretKey`
- 保持 QiniuToken 签名方式
- [IAM 认证文档](https://developer.qiniu.com/mikustream/12893/mikustream-live-http-requests-authentication#3)

**变更要点：**
- IAM 子账号不再需要专用域名 `pili-iam.qiniuapi.com`
- AK/SK 鉴权与 Pili 保持一致
- 新增 Bearer Token 支持
- 详见 [API 请求鉴权](https://developer.qiniu.com/mikustream/12893/mikustream-live-http-requests-authentication)

# 快直播适配说明

**原 Pili 快直播用户：**
- 如需使用 whep 拉流，可升级至最新版 [快直播 SDK](https://developer.qiniu.io/mikustream/12973/miku-fast-live-sdk-download-experience)
- 其余功能可顺畅迁移

---

# 技术支持服务

如有任何迁移问题或技术疑问，欢迎通过右下角联系我们的技术支持团队获取专业协助。
