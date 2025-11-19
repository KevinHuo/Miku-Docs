# 从 Pili 迁移到 Miku（MikuStream）指南

## 背景
Pili 接口与签名方式在 Miku 中均有等价实现；Miku 新增了 Bearer API Key 与 IAM 子账号等鉴权能力，并采用新的服务域名与更统一的直播地址格式。

##  ⚠️ 注意事项
- **海外域名覆盖**：域名覆盖到海外的客户暂不支持切换 Miku，请联系客服确认解决方案
- **推流 SDK 兼容性**：之前使用 Pili 推流 SDK 进行推流的用户，Miku 目前暂不支持，请确认兼容性

##  📋 Pili 有而 Miku 暂无的接口

- **云导播 API**（Cloud Director）
- **Pub 转推 API**（任务创建、编辑、启动、停止、日志、历史记录等）

>   目前只能通过登录门户进行手动操作
> - 导播台：https://portal.qiniu.com/mikustream/caster
> - Pub 转推：https://portal.qiniu.com/mikustream/pub/tasks

##  🔧 基本信息

- **公共入口**：`https://mls.cn-east-1.qiniumiku.com`（用于列举空间等公共操作）
- **空间域名**：`https://<bucket>.mls.cn-east-1.qiniumiku.com`

具体如何使用 Miku 可见文档：[Miku 快速入门](https://developer.qiniu.com/mikustream/12712/mikustream-the-console-quick-start)

##  🚀 迁移步骤

### 1. 推拉流地址

**Miku 支持的推流格式：**
```
# RTMP 推流
rtmp://<domain>/<bucket>/<stream>

# SRT 推流
srt://<domain>:1935?streamid=#!::h=<bucket>/<stream>,m=publish,domain=<domain>

# WHIP 推流
https://<domain>/<bocket>/<stream>.whip
```

**Miku 支持的拉流格式：**
```
# RTMP 拉流
rtmp://<domain>/<bucket>/<stream>

# WHIP 拉流
http://<domain>/<bucket>/<stream>.whep

# HLS 拉流
http://<domain>/<bucket>/<stream>.m3u8

# FLV 拉流
http://<domain>/<bucket>/<stream>.flv

# SRT 拉流
srt://<domain>:1935?streamid=#!::h=<bucket>/<stream>,m=request,domain=<domain>
```

> 💡 **性能建议**：经内部测试及线上反馈，Miku 使用 FLV 格式进行播放整体效果最佳

### 2. 鉴权配置

#### 🔐 推流时间防盗链
如果之前开启了推流鉴权，需要更新鉴权生成逻辑。具体配置请参考：[Miku 鉴权相关文档](https://developer.qiniu.com/mikustream/12893/mikustream-live-http-requests-authentication)

####  🔒 拉流防盗链

##### Referer 防盗链
**配置路径：**
1. 直播空间 → 域名管理 → 具体域名 → 防盗链设置 → Referer 防盗链
2. 直播空间 → 基础设置（全局）→ Referer 防盗链

**说明：**
- 通过 HTTP 头部的 referer 信息进行访问控制
- 仅支持 HTTP 协议的流媒体播放域名

##### 时间戳防盗链
- **算法兼容**：与 Pili 算法一致，可沿用原有逻辑
- **密钥设置**：在直播空间设置中配置主密钥和副密钥

### 3. 服务端 API

####  🔄 域名替换
- **查询直播空间列表**：`pili.qiniuapi.com` → `mls.cn-east-1.qiniumiku.com`
- **其他 API 接口**：`pili.qiniuapi.com` → `<bucket>.mls.cn-east-1.qiniumiku.com`

####  🔑 鉴权方案选择

**方案 A：AK/SK（QiniuToken）**
- 复用 Pili 签名代码
- 修改签名串和请求 Host 为 `mls.<region>.qiniumiku.com`
- [AK/SK 签名鉴权文档](https://developer.qiniu.com/mikustream/12893/mikustream-live-http-requests-authentication#2)

**方案 B：Bearer API Key**
1. 创建 API Key：`POST http://mls.<region>.qiniumiku.com/?apikey`
2. 业务请求：`Authorization: Bearer <Miku API Key>`
- [Bearer Token 鉴权文档](https://developer.qiniu.com/mikustream/12893/mikustream-live-http-requests-authentication#1)

**方案 C：IAM 子账号**
- 使用 `AccessKey: IAM-xxxx` + `SecretKey`
- 按 QiniuToken 方式签名
- [IAM 鉴权文档](https://developer.qiniu.com/mikustream/12893/mikustream-live-http-requests-authentication#3)

>  **重要变化**：与 Pili 不同，IAM 子账号不再需要使用 `pili-iam.qiniuapi.com` 专用域名

#### 📚 API 接口调整
详细接口调整请参考：[Miku Live API 文档](https://s.apifox.cn/83029e40-7850-4d8e-9839-e331d0687253)

### 4. 播放器适配

**Pili 快直播用户：**
- 升级到最新版 SDK
- 修改播放地址格式
- 其他功能无缝切换

## 📞 技术支持
如有迁移问题，请联系技术支持团队获得帮助。