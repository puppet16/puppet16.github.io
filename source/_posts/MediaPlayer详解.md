---
title: MediaPlayer 详解
date: 2020-05-22 15:50:15
tags:
---

# 前言

> MediaPlayer 是 Android 系统自带的播放器，支持播放各种常见媒体类型，包含了 Audio 和 Video 的播放功能，可以播放存储在应用资源（原始资源）内的媒体文件、文件系统中的独立文件或者通过网络连接获得的数据流中的音频或视频。

# 一、支持媒体格式

 下表介绍了 Android 平台中内置的媒体格式支持。括号中注明了不能保证在所有 Android 平台版本上均可用的编解码器，例如：（Android 3.0 及更高版本）。请注意，任何给定的移动设备均可能支持该表中未列出的其他格式或文件类型。

***参考：[https://developer.android.com/guide/topics/media/media-formats](https://developer.android.com/guide/topics/media/media-formats)***

## 音频支持

<table>
    <tr>
        <th>格式/编解码器</th>
        <th>编码器</th>
        <th>解码器</th>
        <th>详细信息</th>
        <th>支持的文件类型/容器格式</th>
    </tr>
    <tr>
        <td>AAC LC</td>
        <td style="text-align: center;"><big>•</big></td>
        <td style="text-align: center;"><big>•</big></td>
        <td rowspan="2">支持单声道/立体声/5.0/5.1 内容，标准采样率为 8-48 kHz。</td>
        <td rowspan="4">
        • 3GPP (.3gp)<br>
        <nobr>• MPEG-4（.mp4、.m4a）</nobr>
        • ADTS 原始 AAC（.aac、在 Android 3.1 及更高版本中解码、在 Android 4.0 及更高版本中编码、不支持 ADIF）
        • MPEG-TS（.ts、不可查找、Android 3.0 及更高版本）</td>
    </tr>
    <tr>
        <td>HE-AACv1 (AAC+)</td>
        <td style="text-align: center;"><big>•</big><br><small>（Android 4.1 及更高版本）</small></td>
        <td style="text-align: center;"><big>•</big></td>
    </tr>
    <tr>
        <td>HE-AACv2（增强型 AAC+）</td>
        <td>&nbsp;</td>
        <td style="text-align: center;"><big>•</big></td>
        <td>支持立体声/5.0/5.1 内容，标准采样率为 8-48 kHz。</td>
    </tr>
    <tr>
        <td>AAC ELD（增强型低延迟 AAC）</td>
        <td style="text-align: center;"><big>•</big><br><small>（Android 4.1 及更高版本）</small></td>
        <td style="text-align: center;"><big>•</big><br><small>（Android 4.1 及更高版本）</small></td>
        <td>支持单声道/立体声内容，标准采样率为 16-48 kHz</td>
    </tr>
    <tr>
        <td>AMR-NB</td>
        <td style="text-align: center;"><big>•</big></td>
        <td style="text-align: center;"><big>•</big></td>
        <td>4.75-12.2 kbps，采样率为 8 kHz</td>
        <td>
        3GPP (.3gp)</td>
    </tr>
    <tr>
        <td>AMR-WB</td>
        <td style="text-align: center;"><big>•</big></td>
        <td style="text-align: center;"><big>•</big></td>
        <td>有 9 个比特率（介于 6.60-23.85 kbit/s 之间）可供选择，采样率为 16 kHz</td>
        <td>
        3GPP (.3gp)</td>
    </tr>
    <tr>
        <td>FLAC</td>
        <td style="text-align: center;" nowrap=""><big>•</big><br><small>（Android 4.1 及更高版本）</small></td>
        <td style="text-align: center;" nowrap=""><big>•</big><br><small>（Android 3.1 及更高版本）</small></td>
        <td>单声道 / 立体声（非多声道）。采样率最高可达 48 kHz（但对于输出为 44.1 kHz 的设备，则建议最高不超过 44.1 kHz，因为 48-44.1 kHz 的降采样器不包含低通滤波器）。建议使用 16 位；对于 24 位，不会应用任何抖动。
        </td>
        <td>
        仅支持 FLAC (.flac)</td>
    </tr>
    <tr>
        <td>GSM</td>
        <td>&nbsp;</td>
        <td style="text-align: center;"><big>•</big></td>
        <td>Android 支持在电话设备上进行 GSM 解码</td>
        <td>GSM (.gsm)</td>
    </tr>
    <tr>
        <td>MIDI</td>
        <td>&nbsp;</td>
        <td style="text-align: center;"><big>•</big></td>
        <td>MIDI 类型 0 和 1。DLS 版本 1 和 2。XMF 和 Mobile XMF。支持铃声格式 RTTTL/RTX、OTA 和 iMelody</td>
        <td>
        • 类型 0 和 1（.mid、.xmf、.mxmf）<br>
        • RTTTL/RTX（.rtttl、.rtx）
        • OTA (.ota)
        • iMelody (.imy)</td>
    </tr>
    <tr>
        <td>MP3</td>
        <td>&nbsp;</td>
        <td style="text-align: center;"><big>•</big></td>
        <td>单声道 / 立体声 8-320 Kbps 恒定 (CBR) 或可变比特率 (VBR)
        </td>
        <td>
        MP3 (.mp3)</td>
    </tr>
    <tr>
        <td>Opus</td>
        <td style="text-align: center;"></td>
        <td style="text-align: center;"><big>•</big><br><small>（Android 5.0 及更高版本）</small></td>
        <td></td>
        <td>
        Matroska (.mkv)</td>
    </tr>
    <tr>
        <td>PCM/WAVE</td>
        <td style="text-align: center;"><big>•</big><br><small>（Android 4.1 及更高版本）</small></td>
        <td style="text-align: center;"><big>•</big></td>
        <td>8 位和 16 位线性 PCM（比特率最高可达到硬件上限）。以 8000、16000 和 44100 Hz 录制原始 PCM 所需的采样率。</td>
        <td>
        WAVE (.wav)</td>
    </tr>
    <tr>
        <td>Vorbis</td>
        <td>&nbsp;</td>
        <td style="text-align: center;"><big>•</big></td>
        <td>&nbsp;</td>
        <td>
        • Ogg (.ogg)<br>
        • Matroska（.mkv、Android 4.0 及更高版本）</td>
    </tr>
</table>

## 视频支持

<table>
    <tr>
        <th>格式/编解码器</th>
        <th>编码器</th>
        <th>解码器</th>
        <th>详细信息</th>
        <th>支持的文件类型/容器格式</th>
    </tr>
    <tr>
        <td>H.263</td>
        <td style="text-align: center;"><big>•</big></td>
        <td style="text-align: center;"><big>•</big></td>
        <td>对 H.263 的支持在 Android 7.0 及更高版本中并非必需</td>
        <td>
        • 3GPP (.3gp)<br>
        • MPEG-4 (.mp4)</td>
    </tr>
    <tr>
        <td>H.264 AVC<br>Baseline&nbsp;Profile&nbsp;(BP)</td>
        <td style="text-align: center;" nowrap=""><big>•</big><br><small>（Android 3.0 及更高版本）</small></td>
        <td style="text-align: center;" nowrap=""><big>•</big></td>
        <td></td>
        <td>
        • 3GPP (.3gp)<br>
        • MPEG-4 (.mp4)
        • MPEG-TS（.ts、仅限 AAC 音频、不可查找、Android 3.0 及更高版本）</td>
    </tr>
    <tr>
        <td>H.264 AVC<br>Main&nbsp;Profile&nbsp;(MP)</td>
        <td style="text-align: center;" nowrap=""><big>•</big><br><small>（Android 6.0 及更高版本）</small></td>
        <td style="text-align: center;" nowrap=""><big>•</big></td>
        <td>解码器为必需项，编码器为推荐项。</td>
        <td>
        </td>
    </tr>
    <tr>
        <td>H.265 HEVC</td>
        <td style="text-align: center;" nowrap=""></td>
        <td style="text-align: center;" nowrap=""><big>•</big><br><small>（Android 5.0 及更高版本）</small></td>
        <td>适用于移动设备的 Main Profile Level 3 和适用于 Android TV 的 Main Profile Level 4.1</td>
        <td>
        • MPEG-4 (.mp4)<br>
        </td>
    </tr>
    <tr>
        <td>MPEG-4 SP</td>
        <td>&nbsp;</td>
        <td style="text-align: center;"><big>•</big></td>
        <td>&nbsp;</td>
        <td>
        3GPP (.3gp)</td>
    </tr>
    <tr>
        <td>VP8</td>
        <td style="text-align: center;" nowrap=""><big>•</big><br><small>（Android 4.3 及更高版本）</small></td>
        <td style="text-align: center;" nowrap=""><big>•</big><br><small>（Android 2.3.3 及更高版本）</small></td>
        <td>只能在 Android 4.0 及更高版本中流式传输</td>
        <td>
        • <a href="http://www.webmproject.org/">WebM</a> (.webm)<br>
        • Matroska（.mkv、Android 4.0 及更高版本）</td>
    </tr>
    <tr>
        <td>VP9</td>
        <td style="text-align: center;" nowrap=""></td>
        <td style="text-align: center;" nowrap=""><big>•</big><br><small>（Android 4.4 及更高版本）</small></td>
        <td></td>
        <td>
        • <a href="http://www.webmproject.org/">WebM</a> (.webm)<br>
        • Matroska（.mkv、Android 4.0 及更高版本）</td>
    </tr>
</table>

## 图片支持

格式 / 编解码器|编码器|解码器|详细信息|支持的文件类型 / 容器格式
:-:|:-:|:-:|:-:|:--|
BMP| |•|  |BMP (.bmp)
GIF| |•|  |GIF (.gif)
JPEG|•|•| 基准式 + 渐进式|JPEG (.jpg)
PNG|•|•| |PNG (.png)
WebP|•<br>（Android 4.0 及更高版本）<br>无损、透明度、Android 4.2.1 及更高版本）|•<br>Android 4.0 及更高版本）<br>无损、透明度、Android 4.2.1 及更高版本）| | WebP (.webp)
HEIF| | •<br>（Android 8.0 及更高版本）| |HEIF（.heic；.heif）

## 网络协议

音频和视频播放支持以下网络协议：

* RTSP（RTP、SDP）
* HTTP/HTTPS 渐进式流式传输
* HTTP/HTTPS 实时流式传输草案协议：
    * 仅限 MPEG-2 TS 媒体文件
    * 协议版本 3（Android 4.0 及更高版本）
    * 协议版本 2 (Android 3.x)
    * 在 Android 3.0 之前的版本中不支持

* **注意：Android 3.1 之前的版本不支持 HTTPS。**

----

# 二、常用方式  

***参考：[https://developer.android.com/reference/android/media/MediaPlayer](https://developer.android.com/reference/android/media/MediaPlayer)***

返回结果|方法名|说明
:-:|:--|:--
static MediaPlayer | create(Context context, Uri uri, SurfaceHolder holder) | 指定从Uri对应的资源文件中来装载文件，同时指定了SurfaceHolder对象并返回MediaPlyaer对象，不需要再调用准备方法。
static MediaPlayer | create(Context context, int resid) | 指定从资源ID对应的资源文件中来装载文件，并返回新创建的MediaPlyaer对象，不需要再调用准备方法。
static MediaPlayer|create(Context context, Uri uri) | 指定从Uri对应的资源文件中来装载文件，并返回新创建的MediaPlayer对象，不需要再调用准备方法。
int | getCurrentPosition() | 获取当前播放的位置，**单位毫秒**
int | getDuration() | 获取音频的时长，**单位毫秒**
int | getVideoHeight() | 获取视频的高度。
int | getVideoWidth() | 获取视频的宽度。
boolean | isLooping() | 判断MediaPlayer是否正在循环播放。
boolean | isPlaying() | 判断MediaPlayer是否正在播放。
void | pause() | 暂停播放。
void | prepare() | 准备播放（装载文件），调用此方法会使MediaPlayer进入Prepared状态。
void | prepareAsync() | 异步准备播放,需要与setOnPreparedListener联合使用
void | release() | 释放媒体资源。
void | reset() | 重置MediaPlayer进入IDLE状态。
void | seekTo(int msec) | 定位到指定的时间位置。
void | setAudioStreamType(int streamtype) | 设置音频流的类型。
void | setDataSource(String path) | 指定装载path路径所代表的文件。
void | setDataSource(Context context, Uri uri, Map<String, String headers) | 指定装载uri所代表的文件。
void | setDataSource(Context context, Uri uri) | 指定装载uri所代表的文件。
void | setDataSource(FileDescriptor fd, long offset, long length) | 指定装载fd所代表的文件中从offset开始长度为length的文件内容。
void | setDataSource(FileDescriptor fd) | 指定装载fd所代表的文件。
void | setDisplay(SurfaceHolder sh) | 设置显示方式。
void | setLooping(boolean looping) | 设置是否循环播放。
void | setNextMediaPlayer(MediaPlayer next) | 设置当前流媒体播放完毕,下一个播放的MediaPlayer。
void | setOnBufferingUpdateListener(MediaPlayer.OnBufferingUpdateListener listener) | 注册一个回调函数,在网络视频流缓冲变化时调用。
void | setOnCompletionListener(MediaPlayer.OnCompletionListener listener) | 为MediaPlayer的播放完成事件绑定事件监听器。
void | setOnErrorListener(MediaPlayer.OnErrorListener listener) | 为MediaPlayer的播放错误事件绑定事件监听器。
void | setOnPreparedListener(MediaPlayer.OnPreparedListener listener) | 当MediaPlayer已经准备完成时触发该监听器。
void | setOnSeekCompleteListener(MediaPlayer.OnSeekCompleteListener listener) | 当MediaPlayer里一个定位操作完成时触发该监听器。
void | setOnVideoSizeChangedListener(MediaPlayer.OnVideoSizeChangedListener listener) | 注册一个用于监听视频大小改变的监听器。
void | setScreenOnWhilePlaying(boolean screenOn) | 设置是否使用SurfaceHolder来显示。
void | setSurface(Surface surface) | 设置Surface。
void | setVideoScalingMode(int mode) | 设置视频缩放的模式。
void | setVolume(float leftVolume, float rightVolume) | 设置播放器的音量。
void | setWakeMode(Context context, int mode) | 为MediaPlayer设置低级电源管理行为。
void | start() | 开始或恢复播放。
void | stop() | 停止播放。

# 三、使用

# 四、 状态转换  

<br>  

![mediaPlayer生命周期](Mediaplayer详解/mediaplayer.gif)

上面这张状态转换图清晰的列举了主要的方法的调用时序，每种方法只能在一些特定的状态下使用，如果MediaPlayer的状态不正确则会引发`IllegalStateException`异常。

### **Idle状态：** 
当使用`new()`方法创建一个MediaPlayer对象或者调用了其`reset()`方法时，该MediaPlayer对象就会处于`idle`状态。在处于`Idle`状态时，调用`getCurrentPosition()`, `getDuration()`, `getVideoHeight()`,`getVideoWidth()`,`setAudioStreamType(int)`, `setLooping(boolean)`, `setVolume(float, float)`, `pause()`, `start()`, `stop()`, `seekTo(int)`, `prepare()`或者 `prepareAsync()`方法都会出现相应错误。*因为没有装裁资源嘛。*  
使用`new()`和调用`reset()`置成`Idle`状态有一个重要差别：当一个MediaPlayer对象刚被构建的时候，内部的播放引擎和对象的状态都没有改变，在这个时候调用以上的那些方法，框架将无法回调客户端程序注册的`OnErrorListener.onError()`方法，所以不会进入`Error`状态；但若这个MediaPlayer对象调用了`reset()`方法之后，再调用以上的那些方法，内部的播放引擎就会回调客户端程序注册的`OnErrorListener.onError()`方法，这时MediaPlayer会进入`Error`状态。

**提示：** 使用`new`操作符创建的MediaPlayer对象处于`Idle`状态，而那些通过重载的`create()`方法创建的MediaPlayer对象却不是处于`Idle`状态。如果成功调用了重载的`create()`方法，则MediaPlayer已经是`prepared`状态；若调用重载的`create()`方法失败，则MediaPlayer返回`null`。

### **End状态：** 
通过`release()`方法可以进入`End`状态，只要MediaPlayer对象不再被使用，就应当尽快将其通过`release()`方法释放掉，以释放相关资源。如果MediaPlayer进入了`End`状态，则不会再进入任何其他状态。

### **Initialized状态：**  
MediaPlayer调用`setDataSource()`方法成功就进入Initialized状态，表示此时要播放的文件已经设置好了。

**提示：** 若当此MediaPlayer **不在`Idle`** 的状态下，调用了`setDataSource()`方法，会抛出`IllegalStateException`异常。

### **Prepared状态：**  

载入资源成功之后还需要通过调用`prepare()`或`prepareAsync()`方法，这两个方法一个是同步的一个是异步的。其中，`prepareAsync()`方法需要与`setOnPreparedListener`配合使用。只有进入`Prepared`状态，才可以调用`start()`方法进行文件播放。

**提示：** 当MediaPlayer对象处于`Prepared`状态的时候，可以调整音频/视频的属性，如音量，播放时是否一直亮屏，循环播放等。

### **Preparing状态：** 
这个状态比较好理解，主要是和prepareAsync()配合，如果异步准备完成，会触发OnPreparedListener.onPrepared()，进而进入Prepared状态。

 

Started状态：显然，MediaPlayer一旦准备好，就可以调用start()方法，这样MediaPlayer就处于Started状态，这表明MediaPlayer正在播放文件过程中。可以使用isPlaying()测试MediaPlayer是否处于了Started状态。如果播放完毕，而又设置了循环播放，则MediaPlayer仍然会处于Started状态，类似的，如果在该状态下MediaPlayer调用了seekTo()或者start()方法均可以让MediaPlayer停留在Started状态。

 

Paused状态：Started状态下MediaPlayer调用pause()方法可以暂停MediaPlayer，从而进入Paused状态，MediaPlayer暂停后再次调用start()则可以继续MediaPlayer的播放，转到Started状态，暂停状态时可以调用seekTo()方法，这是不会改变状态的。

 

Stop状态：Started或者Paused状态下均可调用stop()停止MediaPlayer，而处于Stop状态的MediaPlayer要想重新播放，需要通过prepareAsync()和prepare()回到先前的Prepared状态重新开始才可以。

 

PlaybackCompleted状态：文件正常播放完毕，而又没有设置循环播放的话就进入该状态，并会触发OnCompletionListener的onCompletion()方法。此时可以调用start()方法重新从头播放文件，也可以stop()停止MediaPlayer，或者也可以seekTo()来重新定位播放位置。

 

Error状态：在一般情况下，由于种种原因一些播放控制操作可能会失败，如不支持的音频/视频格式，缺少隔行扫描的音频/视频，分辨率太高，流超时等原因，等等会触发会触发OnErrorListener.onError()事件，此时MediaPlayer会进入Error状态，及时捕捉并妥善处理这些错误是很重要的，可以帮助我们及时释放相关的软硬件资源，也可以改善用户体验。

开发者可以通过setOnErrorListener(android.media.MediaPlayer.OnErrorListener设置监听器来监听MediaPlayer是否进入Error状态。如果MediaPlayer进入了Error状态，可以通过调用reset()来恢复，使得MediaPlayer重新返回到Idle状态。
