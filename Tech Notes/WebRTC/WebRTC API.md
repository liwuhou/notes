
WebRTC 的基础 API 都有一个使用前提，那就是要在[[Secure Origins|安全源]]的情况下访问，否则调用会出错。

### getUserMedia
以前的版本中我们经常使用 `navigator.getUserMedia` 来获取计算机的摄像头或者麦克风权限，但是现在这个接口废弃，变更为 `navigator.mediaDevices.getUserMedia`，因此后面我们均使用新的`API`来完成代码编写。

`getUserMedia` 这个方法如其名，可以获取到用户层面的媒体，并且提供了一些可选项。常用做获取摄像头或者麦克风权限集合之前的**探路函数**。

```js
const userMediaList = await navigator.mediaDevices.getUserMedia({video: true, audio: true})
```

![](http://cdn.liwuhou.cn/blog/202302042254768.png)

```js
try {
  const userMediaList = await navigator.mediaDevices.getUserMedia({video: true, audio: true})
  userMediaList.getTracks().forEach((track) => track.stop())
} catch(e) {
  alert('Permission denied')
}
```

如果没有授权，就会出现这个授权的提示框，如果用户拒绝授权或者授权失败就会触发 `catch`。

这个函数的作用在于探路，在我们真正使用用户的媒体设备之前，先访问一下服务，如果在此时用户同意使用设备，那么后续我们就能在 `enumerateDevices` 函数中获取到设备信息。

> `track.stop()`那行代码是停止去某些流的捕获，如果没有清除的话，在浏览器标签的右侧是可以看到有个红色的小点一直在闪烁的。当在会议中对摄像头等设备使用完毕后，除了强制刷新，可以这样清除一下当前正在使用的 stream。

### enumerateDevices
获取当前用户的设备列表信息。是一个对象数组，其中有 `kind` 字段来标识媒体设备的类型，还有 `deviceId` 来标识设备，后续指定使用具体的哪个媒体设备就是通过传入这个字段值。

![](http://cdn.liwuhou.cn/blog/202302042309593.png)

```ts
interface EnumerateDevices {
  kind: 'audioIn' | 'audioOut' | 'videoIn',
  deviceId: string
  label: string
  groundId?: string
}
```

### constraints 参数
`getUserMedia` 方法可以传入一个参数，用来控制计算机的摄像头和麦克风的媒体流。

#### 同时获取视频输入和音频输入
`navigator.getUserMedia({video: true, audio: true})`。
当遇到计算机没有摄像头的时候，直接调用上述代码就会报错，这个时候就需要使用 `enumerateDevices` 返回的结果主动判断有无视频输入源，没有的话就将 `video` 设置为 `false`。

#### 获取指定分辨率
可以通过 `{width: number, height: number}` 指定设备捕获视频的分辨率。

还有更详细的配置，能让视频分辨率在一个范围内。其中 `ideal` 字段拥有最高的权重，也就是说，浏览器会最先尝试最接近理想值的设定或摄像头（可能不只一个摄像头）。

```js
await navigator.getUserMedia({
  audio: true,
  video: {
    width: {min: 320, ideal: 1280, max: 1920},
    height: {min: 240, ideal: 720, max: 1080}
  }
})
```

#### 指定视频轨道：获取移动设备的前置或后置摄像头
`facingMode`属性，可接受的值有 `user` 前置摄像头、`environment` 后置摄像头；值得注意的是，这个属性在移动端可用，在移动端利用这个属性可以达到切换前后摄像头的效果。

```
{audio: true, video: {facingMode: 'user'}} // 前置摄像头
{audio: true, video: {facingMode: 'environment'}} // 后置摄像头
```

#### 指定帧速率
`frameRate`，[[Video Basis#Frame Rate|帧率]]。顾名思义，就是对捕获视频的帧率进行设置，这个选项不仅对视频质量，还对带宽有着影响，所以在我们通话过程中，如果判定网络的状态不好，那么可以限制帧速率来提升通话的稳定性。

```js
const constraints = {
  width: 1920,
  height: 1080,
  frameRate: {ideal: 10, max: 15}
}
```

在某个特定的场合，选择 `frameRate` 再搭配前面提到的分辨率，可以提高 WebRTC 通话的质量。例如：
- 屏幕分享的过程中，我们应当重视分辨率而不是帧率，这时稍微卡点也没事
- 在普通会议中，我们应当重视的就是画面的流畅性，即重视帧率而不是分辨率
- 在开会人数多但宽带又受限的时候，就要重视会议的流程性，可以降低画面的分辨率来优化体验

#### 使用特定的网络摄像头或麦克风
这里就要用到前面 `enumerateDevices` 时说的列表和其中的 `deviceId`了。
我们可以通过传入特定的 `deviceId`，来使用指定的设备。

```js
const getTargetStream = async (videoId, audioId) => {
  const stream = await navigator.getUserMedia({
    video: {
      deviceId: videoId ? { exact: videoIdl } : undefined,
      width: 1920,
      height: 1080,
      frameRate: { ideal: 10, max: 15 }
    },
    audio: {
      deviceId: audioId ? { exact: audioId } : undefined
    }
    
  })
  return stream
}
```

### getDisplayMedia
这个 api 可以用来分享自己的屏幕，或是仅分享桌面上固定的应用程序。详细在的 W3C 的[Screen Capture](https://w3c.github.io/mediacapture-screen-share/)中有说明。

```js
const constraints = {
  audio: true,
  video: true
}

const stream = await navigator.mediaDevices.getDisplayMdeia(constraints)

```

这个时候会出现一个选项框给到用户选择要分享的浏览器 tab、屏幕或者是应用程序窗口。
如果 `constraints` 中设置了 `audio` 为 `true`的话，也有会对应的询问用户是否分享音频的选项。

![](http://cdn.liwuhou.cn/blog/202302051534265.png)

这里的 `constraints`与 `getUserMedia` 不同的是，`video` 是不能设置为 `false`的，也可以指定分辨率。

```js
getDisplayMedia({
  audio: false,
  video: {width: 1920, height: 1080}
})
```

