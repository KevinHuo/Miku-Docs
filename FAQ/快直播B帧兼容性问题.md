# 什么是 B 帧？
B 帧（Bidirectional Predictive Frame）是 H.264/H.265 编码中的双向预测帧，通过参考前后帧实现更高的压缩效率，但解码需要依赖前后帧，因此会带来额外的延迟和复杂度。

**特点：**
- 压缩率高
- 解码依赖更多参考帧
- 存在 PTS/DTS 重排序

# 为什么 RTC 播放器可能不兼容 B 帧？
RTC（实时通信）场景对延迟极度敏感，而 B 帧会引入帧重排和解码等待，会影响实时性。

可能带来的问题包括：

- **延迟增加**：需要等待后续帧才能解码 B 帧  
- **丢帧风险更高**：参考帧丢失导致 B 帧无法恢复  
- **RTC 解码器不支持 B 帧**：WebRTC 默认要求无 B 帧

# 快直播 RTC 播放器中的不兼容表现
当推流中包含 B 帧时，可能出现：

- 黑屏  
- 卡顿、跳帧  
- 花屏、画面错乱  
- 延迟明显增加

# 如何解决 B 帧兼容问题？

## 方案 1（推荐方案）：推流端禁用 B 帧
以 OBS 为例：

### 步骤 1：进入设置
OBS → 设置
![OBS设置页面](https://dn-odum9helk.qbox.me/FjqAtnQAaJgqdLd66TrZTbaeZ9C7)

### 步骤 2：输出配置
根据下图配置OBS的输出配置。

![输出配置](https://dn-odum9helk.qbox.me/FhW7PA_yBvY6ARNIQxGFOp-f4Ihg)

### 步骤 3：推流配置
进入推流设置，服务选择自定义，将七牛云的推流地址复制粘贴到服务器和串流密钥上，点击确定。
![输出配置](https://dn-odum9helk.qbox.me/FhW7PA_yBvY6ARNIQxGFOp-f4Ihg)

### 步骤 4：开始推流
点击开始推流，即可推不含B帧的直播流到七牛云直播服务上。

> 此方案是最推荐方式，零成本且从源头规避 B 帧问题。

## 方案 2：使用云端转码去除 B 帧（无法控制推流端时）
若推流设备无法关闭 B 帧，可在 MikuStream 控制台使用“去 B 帧”转码模板。

### 步骤 1
打开[直播转码控制台](https://portal.qiniu.com/mikustream/templates)

### 步骤 2
创建转码模板并勾选“去 B 帧”。
![](https://docs.qnsdk.com/miku_e47c1a92fb8f4d5cbe3c0f7a91d2e6f3.png)

### 步骤 3
直播空间绑定转码模版。
![转码模板绑定](https://docs.qnsdk.com/miku_b13fa9c7e2d44f0a9c58d73fe904c2ab.png)

> 注意：使用转码模板会产生额外费用，具体价格请参考 [转码计费规则](https://www.qiniu.com/prices/mikustream)

# 如何检查视频流是否包含 B 帧？
使用 ffprobe（推荐）：

```bash
ffprobe -v error -show_frames -select_streams v:0 <流地址>
```

检查输出中的 pict_type：
- I = 关键帧
- P = 前向预测帧
- B = 双向预测帧（即 B 帧）

若看到 pict_type=B，说明流中存在 B 帧。
