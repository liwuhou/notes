信令服务器的作用
使用 Nodejs+socket.io 实现信令服务器

设备管理
音视频采集

p2p 穿越

RTP/RTCP

网络限流

媒体协商

音视频数据的传输

非音视频数据的传输

统计网络质量

Track 
不相交 轨

MediaStream
流，流里面包含了很多轨

### WebRTC 里重要的类
MediaStream

RTCPeerConnecction

RTCDataChannel
文本、文件、二进制数据

![](http://cdn.liwuhou.cn/tmp/20220319180208.png)

![时序图](http://cdn.liwuhou.cn/tmp/20220319180537.png)

#### web 服务器
（信令服务器）

### 设备管理

**enumerateDevices**
获取到设备中的音频和视频设备

```js
var ePromise = navigator.mediaDevices.enumberateDevices()
```

返回的是一个 `promise`

其中有一个 `MediaDevicesInfo` 字段，包含了

| 属性     | 说明                                         |
| -------- |:-------------------------------------------- |
| deviceId | 设备 ID                                      |
| groupID  | 两个设备 `groupID`相同，说明是同一个物理设备 |
| label    | 设备的名字                                   |
| kind     | 设备的各类                                   |

![](http://cdn.liwuhou.cn/tmp/20220321233718.png)

### 音视频采集 API

```js
const promise = navigator.mediaDevices.getUserMedia(constraints)
```

**MediaStreamConstraints**
``` js
dictionary MediaStreamConstraints {
  (boolean or MediaTrackConstraints) video = false
  (boolean or MediaTrackConstraints) audio = false
}
```

如果是简单的 `boolean`类型的值，那么就可以控制音视频的采集开关，而`MediaTrackConstraints`配置则是更精细化的配置，可以控制视频的分辨率、帧率、音频声道、延时等信息。

一个简单的捕获音视频并展示在页面 Video 标签的 demo

```js
(async() => {
  const constraints = {
    video: true,
    audio: true
  }
  const stream = await navigator.mediaDevices.getUserMedia(constraints)
  const video = document.createElement('video')
  video.srcObject = stream;
  video.autoplay = true
  document.appendChild(video)
})()
```

#### getUserMedia 兼容性适配
方法一：都监测下
```js
const getUserMedia = navigator.getUserMedia ||
      navigator.webkitGetUserMedia ||
      navigator.mozGetUserMedia
```

方法二：使用 google 开源库：[adapter.js](https://github.com/webrtchacks/adapter)

### 采集约束
#### 视频采集约束
width
height
aspectRatio  比例 宽/高 小数
frameRate 帧率 可以控制码流
facingMode 摄像头模式
  - user 前置
  - environment 后置
  - left 前置左侧
  - right 前置右侧
resizeMode 是否裁剪
  - none 不裁剪