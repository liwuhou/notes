推拉流是一个将实况场景信息实时编码压缩后，进行网络输出，用户侧播放经过颁发后的流媒体数据的过程。

![](http://cdn.liwuhou.cn/blog/202302122239544.png)

整个过程有以下几步：
1. 经过输入设备等到原始的采样数据（[[Video Basis|视频数据]]和[[Audio Basis|音频数据]]）
2. 使用硬编码（系统中对应的 API）或软编码（FFMpeg 等）来编码压缩音视频数据
3. 然后根据不同的封装格式（如 FLV、TS、MPEG-TS）进行封装压缩音视频数据
4. 通过不同的传输协议（如 RTMP、RTP 等）将流上传到服务器
5. 服务器进行节点分发（CDN）
6. 用户侧通过不同的传输协议（如 RTMP、HLS、HTTP-FLV 等）获取流数据
7. 与步骤 3、2相反，通过反向解封和解码音视频数据
8. 进行渲染播放

上面的每一步都会影响最终推流和拉流的效果，比如采用优质的采集设备就能获取到优质的原始数据。用高配置的电脑并采用最先进的编码策略（如 HEVC，当然播放端也需要配合支持），推流协议选取延迟相对较低的协议（如 RTMP、HTTP-FLV 等）

> RTMP 是 Adobe 公司的提案，依托于 Flash 技术，所以并没有成为行业标准，存在着兼容性问题，故在 Web 中，HTTP-FLV 应用更为广泛。更新的推流协议也可以使用 [[Index|WebRTC]]。