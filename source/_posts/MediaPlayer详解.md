---
title: MediaPlayer 详解
date: 2020-05-22 15:50:15
tags: [Android, MediaPlayer]
summary: MediaPlayer详细介绍，及使用中遇到的问题汇总
academia: true
---
<!-- toc -->

# 前言

> MediaPlayer 是 Android 系统自带的播放器，支持播放各种常见媒体类型，包含了 Audio 和 Video 的播放功能，可以播放存储在应用资源（原始资源）内的媒体文件、文件系统中的独立文件或者通过网络连接获得的数据流中的音频或视频。

# 一、支持媒体格式

 下表介绍了 Android 平台中内置的媒体格式支持。括号中注明了不能保证在所有 Android 平台版本上均可用的编解码器，例如：（Android 3.0 及更高版本）。请注意，任何给定的移动设备均可能支持该表中未列出的其他格式或文件类型。

***参考：[https://developer.android.com/guide/topics/media/media-formats](https://developer.android.com/guide/topics/media/media-formats)***

## 1. 音频支持

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

## 2. 视频支持

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
        • WebM (.webm)<br>
        • Matroska（.mkv、Android 4.0 及更高版本）</td>
    </tr>
    <tr>
        <td>VP9</td>
        <td style="text-align: center;" nowrap=""></td>
        <td style="text-align: center;" nowrap=""><big>•</big><br><small>（Android 4.4 及更高版本）</small></td>
        <td></td>
        <td>
        • WebM (.webm)<br>
        • Matroska（.mkv、Android 4.0 及更高版本）</td>
    </tr>
</table>

## 3. 图片支持

格式 / 编解码器|编码器|解码器|详细信息|支持的文件类型 / 容器格式
:-:|:-:|:-:|:-:|:--|
BMP| |•|  |BMP (.bmp)
GIF| |•|  |GIF (.gif)
JPEG|•|•| 基准式 + 渐进式|JPEG (.jpg)
PNG|•|•| |PNG (.png)
WebP|•<br>（Android 4.0 及更高版本）<br>无损、透明度、Android 4.2.1 及更高版本）|•<br>Android 4.0 及更高版本）<br>无损、透明度、Android 4.2.1 及更高版本）| | WebP (.webp)
HEIF| | •<br>（Android 8.0 及更高版本）| |HEIF（.heic；.heif）

## 4. 网络协议

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

# 二、常用方法  

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


# 三、 状态转换  

<br>  

![mediaPlayer生命周期](Mediaplayer详解/mediaplayer.gif)

上面这张状态转换图清晰的列举了主要的方法的调用时序，每种方法只能在一些特定的状态下使用，如果MediaPlayer的状态不正确则会引发`IllegalStateException`异常。

<font size="5"> **Idle状态：**</font> 
当使用`new()`方法创建一个MediaPlayer对象或者调用了其`reset()`方法时，该MediaPlayer对象就会处于`idle`状态。在处于`Idle`状态时，调用`getCurrentPosition()`, `getDuration()`, `getVideoHeight()`,`getVideoWidth()`,`setAudioStreamType(int)`, `setLooping(boolean)`, `setVolume(float, float)`, `pause()`, `start()`, `stop()`, `seekTo(int)`, `prepare()`或者 `prepareAsync()`方法都会出现相应错误。*因为没有装裁资源嘛。*  
使用`new()`和调用`reset()`置成`Idle`状态有一个重要差别：当一个MediaPlayer对象刚被构建的时候，内部的播放引擎和对象的状态都没有改变，在这个时候调用以上的那些方法，框架将无法回调客户端程序注册的`OnErrorListener.onError()`方法，所以不会进入`Error`状态；但若这个MediaPlayer对象调用了`reset()`方法之后，再调用以上的那些方法，内部的播放引擎就会回调客户端程序注册的`OnErrorListener.onError()`方法，这时MediaPlayer会进入`Error`状态。

**提示：** 使用`new`操作符创建的MediaPlayer对象处于`Idle`状态，而那些通过重载的`create()`方法创建的MediaPlayer对象却不是处于`Idle`状态。如果成功调用了重载的`create()`方法，则MediaPlayer已经是`prepared`状态；若调用重载的`create()`方法失败，则MediaPlayer返回`null`。

<font size="5"> **End状态：** </font>
通过`release()`方法可以进入`End`状态，只要MediaPlayer对象不再被使用，就应当尽快将其通过`release()`方法释放掉，以释放相关资源。如果MediaPlayer进入了`End`状态，则不会再进入任何其他状态。

<font size="5"> **Initialized状态：**  </font>
MediaPlayer调用`setDataSource()`方法成功就进入Initialized状态，表示此时要播放的文件已经设置好了。

**提示：** 若当此MediaPlayer **不在`Idle`** 的状态下，调用了`setDataSource()`方法，会抛出`IllegalStateException`异常。

<font size="5"> **Prepared状态：** </font> 

载入资源成功之后还需要通过调用`prepare()`或`prepareAsync()`方法，这两个方法一个是同步的一个是异步的。其中，`prepareAsync()`方法需要与`setOnPreparedListener`配合使用。只有进入`Prepared`状态，才可以调用`start()`方法进行文件播放。

**提示：** 当MediaPlayer对象处于`Prepared`状态的时候，可以调整音频/视频的属性，如音量，播放时是否一直亮屏，循环播放等。

<font size="5"> **Preparing状态：**</font> 
这个状态主要是和`prepareAsync()`配合，如果异步准备完成，会触发`OnPreparedListener.onPrepared()`，进而进入Prepared状态。

<font size="5"> **Started状态：** </font>
在 `prepared`状态下，就可以调用`start()`方法，这样MediaPlayer就处于`Started`状态，这表明MediaPlayer正在播放文件过程中。可以使用`isPlaying()`测试MediaPlayer是否处于了`Started`状态。如果播放完毕，而又设置了循环播放，则MediaPlayer仍然会处于`Started`状态。

<font size="5"> **Paused状态：** </font>
`Started`状态下MediaPlayer调用`pause()`方法可以暂停MediaPlayer，从而进入`Paused`状态，MediaPlayer暂停后再次调用`start()`则可以继续MediaPlayer的播放，转到`Started`状态，暂停状态时可以调用`seekTo()`方法，且不会改变状态。

<font size="5"> **Stop状态：** </font>
`Started`或者`Paused`状态下均可调用`stop()`停止MediaPlayer，而处于`Stop`状态的MediaPlayer要想重新播放，需要通过`prepareAsync()`或  `prepare()`回到先前的`Prepared`状态重新开始才可以。

<font size="5"> **PlaybackCompleted状态：** </font>
文件正常播放完毕，而又没有设置循环播放的话就进入该状态，并会触发`OnCompletionListener`的`onCompletion()`方法。此时可以调用`start()`方法重新从头播放文件，也可以`stop()`停止MediaPlayer，或者也可以`seekTo()`来重新定位播放位置。

<font size="5"> **Error状态：** </font>
在一般情况下，由于种种原因一些播放控制操作可能会失败，如不支持的音频/视频格式，缺少隔行扫描的音频/视频，分辨率太高，流超时等原因，等等会触发会触发`OnErrorListener.onError()`事件，此时MediaPlayer会进入`Error`状态，及时捕捉并妥善处理这些错误是很重要的，可以帮助我们及时释放相关的软硬件资源，也可以改善用户体验。  
可以通过`setOnErrorListener()`设置监听器来监听MediaPlayer是否进入`Error`状态。如果MediaPlayer进入了`Error`状态，可以通过调用`reset()`来恢复，使得MediaPlayer重新返回到`Idle`状态。


# 四、使用注意事项  

## 1. 使用流媒体时准备方法使用prepareAsync()
使用`start()`播放流媒体之前，需要装载流媒体资源。这种情况应该使用`prepareAsync()`用异步的方式装载流媒体资源。因为流媒体资源的装载是会消耗系统资源的，在一些硬件不理想的设备上，如果使用`prepare()`同步的方式装载资源，可能会造成UI界面的卡顿，非常影响用户体验的。使用异步方式时，为了避免还没有装载完成就调用`start()`而报错的问题，需要绑定`MediaPlayer.setOnPreparedListener()`事件，它将在异步装载完成之后回调。异步装载还有一个好处就是避免装载超时引发ANR（Application Not Responding）错误。

```
MediaPlayer mediaPlayer = new MediaPlayer();
mediaPlayer.setDataSource(UriPath);
// 通过异步的方式装载媒体资源
mediaPlayer.prepareAsync();
mediaPlayer.setOnPreparedListener(new OnPreparedListener() {                    
    @Override
    public void onPrepared(MediaPlayer mp) {
        // 装载完毕回调
        mediaPlayer.start();
    }
});
```
## 2. 回收资源

使用完MediaPlayer需要回收资源。在使用完MediaPlayer，不要等待系统自动回收，最好是主动回收资源。

```
if (mediaPlayer != null) {
    mediaPlayer.stop();
    mediaPlayer.release();
    mediaPlayer = null;
}
```
## 3. setOnCompletionListener的使用  
setOnCompletionListener监听器在MediaPlayer播放完成后触发。但是，在使用`prepareAsync()`方法准备资源时出错也会调用该方法，可根据播放器总时长是否为0来判断是成功还是失败，即`getDuration() == 0`。

# 五、播放进度展示  
SeekBar可以用来显示播放进度，用户也可以利用SeekBar的滑块来控制音乐的播放。
## SeekBar与MediaPlayer所需要方法  
* SeekBar:
    * setProgress（int value）：设置滑块中心的位置。

    * setMax(int value)：设置进度条的最大长度,可以设置为播放器的总时长，这样seeKTo时不需要再转换。

    * setOnSeekBarChangeListener(OnSeekBarChangeListener l)：设置SeekBar的进度改变监听器。
* MediaPlayer:
    * getDuration()：获得音乐长度,单位毫秒。

    * getCurrentPosition()：获得现在播放的位置，单位毫秒。

    * seekTo(int msec)：调用seekTo()方法可以调整播放的位置。

## 进度条展示步骤

### 1. 创建SeekBar

```
<android.support.constraint.ConstraintLayout
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:orientation="horizontal">

    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:minWidth="29dp"
        android:textColor="#FF999999"
        android:textSize="11sp"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintLeft_toLeftOf="parent"
        app:layout_constraintTop_toTopOf="parent"
        tools:text="06:52" />

    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="--:--"
        android:textColor="#FF999999"
        android:textSize="11sp"
        android:minWidth="29dp"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintRight_toRightOf="parent"
        app:layout_constraintTop_toTopOf="parent"
        tools:text="06:52" />

    <SeekBar
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:max="0"
        android:maxHeight="2dp"
        android:paddingStart="9dp"
        android:paddingEnd="9dp"
        android:progressDrawable="@drawable/seekbar_style_audio_work_detail"
        android:splitTrack="false"
        android:thumb="@mipmap/ico_audio_work_seek_dot"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintLeft_toRightOf="@id/audio_tv_current_duration"
        app:layout_constraintRight_toLeftOf="@id/audio_tv_total_duration"
        app:layout_constraintTop_toTopOf="parent" />

</android.support.constraint.ConstraintLayout>
```
>**注意：**  
>1. 若滑块与进度条之间有间隔，则需要添加属性`android:splitTrack="false"`。
>2. 若要设置SeekBar左右间距则必须使用`android:paddingStart`和`android:paddingEnd`。  

### 2. 添加进度条的改变监听器

```
mSeekBar.setOnSeekBarChangeListener(new SeekBar.OnSeekBarChangeListener() {
    @Override
    public void onProgressChanged(SeekBar seekBar, int progress, boolean fromUser) {

    }

    @Override
    public void onStartTrackingTouch(SeekBar seekBar) {
        stopAudioProgressSync();
    }

    @Override
    public void onStopTrackingTouch(SeekBar seekBar) {
        MediaPlayer.seekTo(seekBar.getProgress());
    }
});
```
>**说明：**  
>1. onProgressChanged (SeekBar seekBar, int progress, boolean fromUser)
    * 进度已经被修改。客户端可以使用fromUser参数区分用户触发的改变还是编程触发的改变
    * *progress*      当前的进度值。此值的取值范围为0到max之间。Max为用户通过setMax(int)设置的值，默认为100
    * *fromUser* 如果是用户触发的改变则返回True  
>2. onStartTrackingTouch (SeekBar seekBar)
    * 通知用户已经开始一个触摸拖动手势。客户端可能需要使用这个来禁用seekbar的滑动功能，或是禁止进度条更新。
>3. onStopTrackingTouch (SeekBar seekBar)  
    * 通知用户触摸手势已经结束。户端可能需要使用这个来启用seekbar的滑动功能，或是再次开启进度条更新。  

### 3. 设置seekBar最大值
``` 
mSeekBar.setMax(mediaPlayer.getDuration());//设置SeekBar的长度
```
>**注意:**  
>`getDuration()`方法要在`prepare()`方法成功之后，否则会出现`Attempt to call getDuration without a valid mediaplayer`异常  

### 4. 更新进度条  
```
private DisposableObserver<Long> mAudioProgressSyncObserver;

/**
 * 关闭音频进度更新计时器
 */
public void stopAudioProgressSync() {
    if (mAudioProgressSyncObserver != null)
        mAudioProgressSyncObserver.dispose();
}

/**
 * 打开音频进度更新计时器
 */
public void startAudioProgressSync() {
    stopAudioProgressSync();
    mAudioProgressSyncObserver = Observable.interval(0, 500, TimeUnit.MILLISECONDS)
            .subscribeOn(Schedulers.io())
            .observeOn(AndroidSchedulers.mainThread())
            .subscribeWith(new DisposableObserver<Long>() {
                @Override
                public void onNext(Long count) {
                    //如果当前是播放状态，则更新进度条及当前播放时间
                }

                @Override
                public void onError(Throwable throwable) {
                }

                @Override
                public void onComplete() {
                }
            });

}
```
