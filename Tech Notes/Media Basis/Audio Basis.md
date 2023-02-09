## Bit Rate
同视频码率[[Video Basis#Bit Rate]]，音频中指单位时间内取样率(Sampling Rate)的大小，越高则说明精度越大，处理出来的文件就越接近原始文件，一般使用的单位是 kbps，即千位每秒

### 音频中的码率计算公式
$$
比特率(kbps) = 量化采样点(kHz) \times 位深(\frac{bit}{采样点}) \times 声道数量(一般为2)
$$

### 比特率对应音频质量
32 kbps — MW(AM) 质量
96 kbps — FM 质量
128 ~ 160 kbps — 相当好的质量，有时有明显差别
192 kbps — 优良质量，偶尔有差别
224 ~ 320 kbps — 高质量
800 bps — 能够分辨的语音所需最低码率（需使用专用的 FS-1015 语音编解码器）
8 kbps — 电话质量（使用语音编码）
8 ~ 500 kbps — Ogg Vorbis 和MPEG1 Player1/2/3 中使用的有损音频模式
500 kbps ~ 1.4 Mbps —44.1KHz 的无损音频，解码器为 FLAC Audio, WavPack 或 Monkey's Audio
1411.2 ~ 2822.4 Kbps —脉冲编码调制(PCM)声音格式 CD 光碟的数字音频
5644.8 kbps — SACD使用的 Direct Stream Digital 格式

## Sampling Rate
音频采样率，是指在一秒钟内，收音设备对声音信号的采样次数，采样的频率越高，对声音的还原度就越高，效果就越真实。一般的单位是 `Hz`和 `KHz`

![](http://cdn.liwuhou.cn/tmp/20230130212616.png)

### 常用的采样率

- 8,000 Hz - 电话所用采样率
- 11,025 Hz-AM调幅广播所用采样率
- 22,050 Hz和24,000 Hz- FM调频广播所用采样率
- 32,000 Hz - miniDV 数码视频 camcorder、DAT (LP mode)所用采样率
- 44,100 Hz - 音频 CD, 也常用于 MPEG-1 音频（VCD, SVCD, MP3）所用采样率
- 47,250 Hz - 商用 PCM 录音机所用采样率
- 48,000 Hz - miniDV、数字电视、DVD、DAT、电影和专业音频所用的数字声音所用采样率
- 50,000 Hz - 商用数字录音机所用采样率
- 96,000 或者 192,000 Hz - DVD-Audio、一些 LPCM DVD 音轨、BD-ROM（蓝光盘）音轨、和 HD-DVD （高清晰度 DVD）音轨所用所用采样率
- 2.8224 MHz - Direct Stream Digital 的 1 位 sigma-delta modulation 过程所用采样率。

## Sampling Size
用来衡量声音波动变化的参数，是指声卡在采集和播放声音文件时所使用数字声音信号的二进制位数，有8位，16位，24位等。这个数值越大，解析度就越高，录制和回放的声音就越真实。

![](http://cdn.liwuhou.cn/tmp/20230130213842.png)

### 计算公式

$$
Bits = 采样率 \times 位深 \times 通道 \times 时长(s)
$$


## 应用场景

-   16Bit：动态范围大概是96dB，适用于普通流行歌曲。
-   24Bit：动态范围大概是144dB，一般用于电影配乐，交响乐团等等大动态的音频信号。

## Compression Ratio
音频压缩率：原始音频数据与通过PCM等压缩编码技术压缩后的数据大小的比率

压缩率一般用10:1来表示，这种表示也称为压缩系数。10代表未压缩的数据大小

[常见容器格式压缩率](https://www.yuque.com/webmedia/handbook/audio-compression-ratio)
