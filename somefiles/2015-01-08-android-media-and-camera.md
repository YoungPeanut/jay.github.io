---
layout: post
title: android-[API Guides]8:视频－音频－拍照
tags: android API 视频 音频 拍照
categories: android
---


<div class="toc"></div>


利用多媒体相关API，给你的app添加视频／音频／图像等功能。
##第一部分 － 播放多媒体 Media Playback   
</br> 
    
 [MediaPlayer](https://developer.android.com/reference/android/media/MediaPlayer.html) 可以播放视频和音频，包括应用资源文件 (raw resources)／手机磁盘文件／网络远程文件。
 [AudioManager](https://developer.android.com/reference/android/media/AudioManager.html) 音频输入输出管理类   
 
####Manifest声明
- 需要播放网络流媒体

```
<uses-permission android:name="android.permission.INTERNET" />
```

- 防止屏幕变暗／防止手机睡眠／MediaPlayer的setScreenOnWhilePlaying()和MediaPlayer.setWakeMode()方法

```
<uses-permission android:name="android.permission.WAKE_LOCK" />
```

####使用MediaPlayer

android支持的多媒体文件格式  
[https://developer.android.com/guide/appendix/media-formats.html](https://developer.android.com/guide/appendix/media-formats.html)

播放 `res/raw/` 目录下的音频

```
MediaPlayer mediaPlayer = MediaPlayer.create(context, R.raw.sound_file_1);
mediaPlayer.start(); // no need to call prepare(); create() does that for you
```

通过本地uri播放文件(比如Content Resolver得到的uri)

```
Uri myUri = ....; // initialize Uri here
MediaPlayer mediaPlayer = new MediaPlayer();
mediaPlayer.setAudioStreamType(AudioManager.STREAM_MUSIC);
mediaPlayer.setDataSource(getApplicationContext(), myUri);
mediaPlayer.prepare();
mediaPlayer.start();
```

播放http流文件

```
String url = "http://........"; // your URL here
MediaPlayer mediaPlayer = new MediaPlayer();
mediaPlayer.setAudioStreamType(AudioManager.STREAM_MUSIC);
mediaPlayer.setDataSource(url);
mediaPlayer.prepare(); // might take long! (for buffering, etc)
mediaPlayer.start();
```

####关于prepare

不要在UI线程调用 [prepare()](https://developer.android.com/reference/android/media/MediaPlayer.html) ，因为获取和解析数据需要时间。  
可以使用 `prepareAsync()]`,配合  setOnPreparedListener()。

####管理状态

![state ](https://developer.android.com/images/mediaplayer_state_diagram.gif)  

####释放MediaPlayer  
以Activity为例，onStop()时应该释放，resumed or restarted时重新创建一个新的 MediaPlayer。

```
mediaPlayer.release();
mediaPlayer = null;
```
如果想在后台播放，请用Service。  

####MediaPlayer + Service

1	异步
[ Service](https://developer.android.com/reference/android/app/Service.html) 本身也是在主线程中执行的，所以最好使用prepareAsync()  

```
public class MyService extends Service implements MediaPlayer.OnPreparedListener {
    private static final String ACTION_PLAY = "com.example.action.PLAY";
    MediaPlayer mMediaPlayer = null;

    public int onStartCommand(Intent intent, int flags, int startId) {
        ...
        if (intent.getAction().equals(ACTION_PLAY)) {
            mMediaPlayer = ... // initialize it here
            mMediaPlayer.setOnPreparedListener(this);
            mMediaPlayer.prepareAsync(); // prepare async to not block main thread
        }
    }

    /** Called when MediaPlayer is ready */
    public void onPrepared(MediaPlayer player) {
        player.start();
    }
}

```

2	 监听异常  

```
public class MyService extends Service implements MediaPlayer.OnErrorListener {
    MediaPlayer mMediaPlayer;

    public void initMediaPlayer() {
        // ...initialize the MediaPlayer here...

        mMediaPlayer.setOnErrorListener(this);
    }

    @Override
    public boolean onError(MediaPlayer mp, int what, int extra) {
        // ... react appropriately ...
        // The MediaPlayer has moved to the Error state, must be reset!
    }
}
```

####设备保持唤醒 wake locks

让CPU保持唤醒

```
mMediaPlayer = new MediaPlayer();
// ... other initialization here ...
mMediaPlayer.setWakeMode(getApplicationContext(), PowerManager.PARTIAL_WAKE_LOCK);
```

获取[ WifiLock](https://developer.android.com/reference/android/net/wifi/WifiManager.WifiLock.html) 

```
WifiLock wifiLock = ((WifiManager) getSystemService(Context.WIFI_SERVICE))
    .createWifiLock(WifiManager.WIFI_MODE_FULL, "mylock");

wifiLock.acquire();
//解锁
//wifiLock.release();
```

####foreground service

比如音乐应用，用户与播放器不断交互，这时service 应该用作 "foreground service."。前台service比后台service有更高的优先级。"foreground service"一定要有一个 [ Notification](https://developer.android.com/reference/android/app/Notification.html) 给用户交互。在Service中调用 startForeground()。  

```
String songName;
// assign the song name to songName
PendingIntent pi = PendingIntent.getActivity(getApplicationContext(), 0,
                new Intent(getApplicationContext(), MainActivity.class),
                PendingIntent.FLAG_UPDATE_CURRENT);
Notification notification = new Notification();
notification.tickerText = text;
notification.icon = R.drawable.play0;
notification.flags |= Notification.FLAG_ONGOING_EVENT;
notification.setLatestEventInfo(getApplicationContext(), "MusicPlayerSample",
                "Playing: " + songName, pi);
startForeground(NOTIFICATION_ID, notification);
```

当不再给用户知道的时候

```
stopForeground(true);
```

####处理audio focus

从android2.2开始(API level 8才有audio focus的API)，防止多个app同时大声播放，推出了audio focus机制。该机制认为，音频app要一直请求音频焦点，得到焦点后进行播放动作，同时还继续监听焦点变化。当失去焦点的时候，应该关掉音频或减小音量到一定程度。  

```
AudioManager audioManager = (AudioManager) getSystemService(Context.AUDIO_SERVICE);
int result = audioManager.requestAudioFocus(this, AudioManager.STREAM_MUSIC,
    AudioManager.AUDIOFOCUS_GAIN);

if (result != AudioManager.AUDIOFOCUS_REQUEST_GRANTED) {
    // could not get audio focus.
}
```

你的activity或service监听焦点。  

```
class MyService extends Service
                implements AudioManager.OnAudioFocusChangeListener {
    // ....
    public void onAudioFocusChange(int focusChange) {
        // Do something based on focus change...
    }
}
```

返回的参数 focusChange，   

- [AUDIOFOCUS_GAIN](https://developer.android.com/reference/android/media/AudioManager.html#AUDIOFOCUS_GAIN),得到焦点。  
- AUDIOFOCUS_LOSS,长时间失去焦点，可以release MediaPlayer 了。
- AUDIOFOCUS_LOSS_TRANSIENT，暂时失去焦点，停止音频播放，但可以保留资源。
- AUDIOFOCUS_LOSS_TRANSIENT_CAN_DUCK，暂时失去焦点，但不必关闭，可以小声播放。  

```
public void onAudioFocusChange(int focusChange) {
    switch (focusChange) {
        case AudioManager.AUDIOFOCUS_GAIN:
            // resume playback
            if (mMediaPlayer == null) initMediaPlayer();
            else if (!mMediaPlayer.isPlaying()) mMediaPlayer.start();
            mMediaPlayer.setVolume(1.0f, 1.0f);
            break;

        case AudioManager.AUDIOFOCUS_LOSS:
            // Lost focus for an unbounded amount of time: stop playback and release media player
            if (mMediaPlayer.isPlaying()) mMediaPlayer.stop();
            mMediaPlayer.release();
            mMediaPlayer = null;
            break;

        case AudioManager.AUDIOFOCUS_LOSS_TRANSIENT:
            // Lost focus for a short time, but we have to stop
            // playback. We don't release the media player because playback
            // is likely to resume
            if (mMediaPlayer.isPlaying()) mMediaPlayer.pause();
            break;

        case AudioManager.AUDIOFOCUS_LOSS_TRANSIENT_CAN_DUCK:
            // Lost focus for a short time, but it's ok to keep playing
            // at an attenuated level
            if (mMediaPlayer.isPlaying()) mMediaPlayer.setVolume(0.1f, 0.1f);
            break;
    }
}
```

另外，由于2.2之前不支持，所以可以这样，  

```
public class AudioFocusHelper implements AudioManager.OnAudioFocusChangeListener {
    AudioManager mAudioManager;

    // other fields here, you'll probably hold a reference to an interface
    // that you can use to communicate the focus changes to your Service

    public AudioFocusHelper(Context ctx, /* other arguments here */) {
        mAudioManager = (AudioManager) mContext.getSystemService(Context.AUDIO_SERVICE);
        // ...
    }

    public boolean requestFocus() {
        return AudioManager.AUDIOFOCUS_REQUEST_GRANTED ==
            mAudioManager.requestAudioFocus(mContext, AudioManager.STREAM_MUSIC,
            AudioManager.AUDIOFOCUS_GAIN);
    }

    public boolean abandonFocus() {
        return AudioManager.AUDIOFOCUS_REQUEST_GRANTED ==
            mAudioManager.abandonAudioFocus(this);
    }

    @Override
    public void onAudioFocusChange(int focusChange) {
        // let your service know about the focus change
    }
}

//
if (android.os.Build.VERSION.SDK_INT >= 8) {
    mAudioFocusHelper = new AudioFocusHelper(getApplicationContext(), this);
} else {
    mAudioFocusHelper = null;
}

```

####释放资源

用完请释放，不要依赖GC

```
public class MyService extends Service {
   MediaPlayer mMediaPlayer;
   // ...

   @Override
   public void onDestroy() {
       if (mMediaPlayer != null) mMediaPlayer.release();
   }
}
```

如果你隔一段时间再用MediaPlayer，那先释放，到时候再重新创建；如果你等会就用，你可以持有MediaPlayer，不必再创建准备。（23333333333）

####Intent  AUDIO_BECOMING_NOISY

这种场景：用耳机听歌，突然耳机失联了。

```
<receiver android:name=".MusicIntentReceiver">
   <intent-filter>
      <action android:name="android.media.AUDIO_BECOMING_NOISY" />
   </intent-filter>
</receiver>
```

```
public class MusicIntentReceiver extends android.content.BroadcastReceiver {
   @Override
   public void onReceive(Context ctx, Intent intent) {
      if (intent.getAction().equals(
                    android.media.AudioManager.ACTION_AUDIO_BECOMING_NOISY)) {
          // signal your service to stop playback
          // (via an Intent, for instance)
      }
   }
}
```

####Content Resolver得到数据

```
ContentResolver contentResolver = getContentResolver();
Uri uri = android.provider.MediaStore.Audio.Media.EXTERNAL_CONTENT_URI;
Cursor cursor = contentResolver.query(uri, null, null, null, null);
if (cursor == null) {
    // query failed, handle error.
} else if (!cursor.moveToFirst()) {
    // no media on the device
} else {
    int titleColumn = cursor.getColumnIndex(android.provider.MediaStore.Audio.Media.TITLE);
    int idColumn = cursor.getColumnIndex(android.provider.MediaStore.Audio.Media._ID);
    do {
       long thisId = cursor.getLong(idColumn);
       String thisTitle = cursor.getString(titleColumn);
       // ...process entry...
    } while (cursor.moveToNext());
}
```

```
long id = /* retrieve it from somewhere */;
Uri contentUri = ContentUris.withAppendedId(
        android.provider.MediaStore.Audio.Media.EXTERNAL_CONTENT_URI, id);

mMediaPlayer = new MediaPlayer();
mMediaPlayer.setAudioStreamType(AudioManager.STREAM_MUSIC);
mMediaPlayer.setDataSource(getApplicationContext(), contentUri);

// ...prepare and start...
```

</br>
##第二部分 － 多媒体路由 Media Router    
</br> 

</br>
##第四部分 － 定制播放器 ExoPlayer
</br> 

[ MediaPlayer](https://developer.android.com/reference/android/media/MediaPlayer.html) 是一种封装好的解决方案，帮你用少量代码播放视频音频。  
[ MediaCodec](https://developer.android.com/reference/android/media/MediaCodec.html) 和 [MediaExtractor](https://developer.android.com/reference/android/media/MediaExtractor.html) 可以用来定制播放器。  
然后谷歌开源了一个播放器项目，ExoPlayer，可以进行扩展。ExoPlayer具有一些MediaPlayer没有的功能，比如基于http的动态自适应流(DASH)，SmoothStreaming，Common Encryption。ExoPlayer也可以作为library引入你的项目。  
1. [http://google.github.io/ExoPlayer/](http://google.github.io/ExoPlayer/) 项目主页  
2. [http://google.github.io/ExoPlayer/guide.html](http://google.github.io/ExoPlayer/guide.html) 开发者文档  
3. [https://github.com/google/ExoPlayer](https://github.com/google/ExoPlayer) Demo源码

</br>
##第五部分 － 格式 Supported Media Formats
</br> 

android支持的编码解码器包括android平台支持的和设备硬件支持的。  

android支持基于以下网络协议的音频视频播放(Android 3.1以前不支持HTTPS)：  
<li> RTSP (RTP, SDP)  
<li> HTTP/HTTPS progressive streaming  
<li> HTTP/HTTPS live streaming [draft protocol](http://tools.ietf.org/html/draft-pantos-http-live-streaming):  
~~ MPEG-2 TS media files only  
~~ Protocol version 3 (Android 4.0 and above)  
~~ Protocol version 2 (Android 3.x)  
~~ Not supported before Android 3.0  

</br>
表1:媒体格式和编解码器

视频相关  
表2：H.264 Baseline Profile编解码器支持的视频参数  

表3：VP8编解码器支持的视频参数  

除了这两个表，硬件兼容的媒体参数可以通过[ CamcorderProfile](https://developer.android.com/reference/android/media/CamcorderProfile.html) 查询。
另外，基于HTTP或RTSP的视频流有额外要求：
如果是3GPP和MPEG-4内容，`moov`原子必须在`mdat`原子前，且在`ftyp`原子后；
如果是3GPP，MPEG-4和WebM内容，每相同时间间隔内的音视频样本不应该大于500kb。
  
</br>
##第六部分 － 音频采集 Audio Capture
</br> 
1. 创建对象 [ android.media.MediaRecorder](https://developer.android.com/reference/android/media/MediaRecorder.html)  
2. 设置音频源，一般是`MediaRecorder.AudioSource.MIC` [ MediaRecorder.setAudioSource()](https://developer.android.com/reference/android/media/MediaRecorder.html#setAudioSource(int))  
3. 设置输出文件格式 [ MediaRecorder.setOutputFormat()](https://developer.android.com/reference/android/media/MediaRecorder.html#setOutputFormat(int))  
4. 指定输出文件 [MediaRecorder.setOutputFile()](https://developer.android.com/reference/android/media/MediaRecorder.html#setOutputFile(java.io.FileDescriptor))  
5. 音频编码器 [ MediaRecorder.setAudioEncoder()](https://developer.android.com/reference/android/media/MediaRecorder.html#setAudioEncoder(int))  
6. 准备 [MediaRecorder.prepare()](https://developer.android.com/reference/android/media/MediaRecorder.html#prepare())  
7. 开始录音 [ MediaRecorder.start()](https://developer.android.com/reference/android/media/MediaRecorder.html#start())  
8. 停止录音 [ MediaRecorder.stop()](https://developer.android.com/reference/android/media/MediaRecorder.html#stop())  
9. 用完请释放 [ MediaRecorder.release() ](https://developer.android.com/reference/android/media/MediaRecorder.html#release())  
 示例：

```
/*
 * The application needs to have the permission to write to external storage
 * if the output file is written to the external storage, and also the
 * permission to record audio. These permissions must be set in the
 * application's AndroidManifest.xml file, with something like:
 *
 * <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
 * <uses-permission android:name="android.permission.RECORD_AUDIO" />
 *
 */
package com.android.audiorecordtest;

import android.app.Activity;
import android.widget.LinearLayout;
import android.os.Bundle;
import android.os.Environment;
import android.view.ViewGroup;
import android.widget.Button;
import android.view.View;
import android.view.View.OnClickListener;
import android.content.Context;
import android.util.Log;
import android.media.MediaRecorder;
import android.media.MediaPlayer;

import java.io.IOException;


public class AudioRecordTest extends Activity
{
    private static final String LOG_TAG = "AudioRecordTest";
    private static String mFileName = null;

    private RecordButton mRecordButton = null;
    private MediaRecorder mRecorder = null;

    private PlayButton   mPlayButton = null;
    private MediaPlayer   mPlayer = null;

    private void onRecord(boolean start) {
        if (start) {
            startRecording();
        } else {
            stopRecording();
        }
    }

    private void onPlay(boolean start) {
        if (start) {
            startPlaying();
        } else {
            stopPlaying();
        }
    }

    private void startPlaying() {
        mPlayer = new MediaPlayer();
        try {
            mPlayer.setDataSource(mFileName);
            mPlayer.prepare();
            mPlayer.start();
        } catch (IOException e) {
            Log.e(LOG_TAG, "prepare() failed");
        }
    }

    private void stopPlaying() {
        mPlayer.release();
        mPlayer = null;
    }

    private void startRecording() {
        mRecorder = new MediaRecorder();
        mRecorder.setAudioSource(MediaRecorder.AudioSource.MIC);
        mRecorder.setOutputFormat(MediaRecorder.OutputFormat.THREE_GPP);
        mRecorder.setOutputFile(mFileName);
        mRecorder.setAudioEncoder(MediaRecorder.AudioEncoder.AMR_NB);

        try {
            mRecorder.prepare();
        } catch (IOException e) {
            Log.e(LOG_TAG, "prepare() failed");
        }

        mRecorder.start();
    }

    private void stopRecording() {
        mRecorder.stop();
        mRecorder.release();
        mRecorder = null;
    }

    class RecordButton extends Button {
        boolean mStartRecording = true;

        OnClickListener clicker = new OnClickListener() {
            public void onClick(View v) {
                onRecord(mStartRecording);
                if (mStartRecording) {
                    setText("Stop recording");
                } else {
                    setText("Start recording");
                }
                mStartRecording = !mStartRecording;
            }
        };

        public RecordButton(Context ctx) {
            super(ctx);
            setText("Start recording");
            setOnClickListener(clicker);
        }
    }

    class PlayButton extends Button {
        boolean mStartPlaying = true;

        OnClickListener clicker = new OnClickListener() {
            public void onClick(View v) {
                onPlay(mStartPlaying);
                if (mStartPlaying) {
                    setText("Stop playing");
                } else {
                    setText("Start playing");
                }
                mStartPlaying = !mStartPlaying;
            }
        };

        public PlayButton(Context ctx) {
            super(ctx);
            setText("Start playing");
            setOnClickListener(clicker);
        }
    }

    public AudioRecordTest() {
        mFileName = Environment.getExternalStorageDirectory().getAbsolutePath();
        mFileName += "/audiorecordtest.3gp";
    }

    @Override
    public void onCreate(Bundle icicle) {
        super.onCreate(icicle);

        LinearLayout ll = new LinearLayout(this);
        mRecordButton = new RecordButton(this);
        ll.addView(mRecordButton,
            new LinearLayout.LayoutParams(
                ViewGroup.LayoutParams.WRAP_CONTENT,
                ViewGroup.LayoutParams.WRAP_CONTENT,
                0));
        mPlayButton = new PlayButton(this);
        ll.addView(mPlayButton,
            new LinearLayout.LayoutParams(
                ViewGroup.LayoutParams.WRAP_CONTENT,
                ViewGroup.LayoutParams.WRAP_CONTENT,
                0));
        setContentView(ll);
    }

    @Override
    public void onPause() {
        super.onPause();
        if (mRecorder != null) {
            mRecorder.release();
            mRecorder = null;
        }

        if (mPlayer != null) {
            mPlayer.release();
            mPlayer = null;
        }
    }
}
```
</br>
##第七部分 －  JetPlayer
</br> 
JET：一个在嵌入式设备上的音乐播放器（JET is an interactive music player for small embedded devices, including the those running the Android platform）。  
JET engine：一个控制游戏声音特效的引擎，其使用MIDI格式，并可以控制游戏的时间进度（一个精确的时钟是一个游戏必不可少）。  
注，这两句话引自 [http://blog.sina.com.cn/s/blog_61f4999d0100ktex.html](http://blog.sina.com.cn/s/blog_61f4999d0100ktex.html)。  
</br>
[JetPlayer](https://developer.android.com/reference/android/media/JetPlayer.html) 类可以用来播放JET内容。播放SD卡上的.jet 文件    

```
JetPlayer jetPlayer = JetPlayer.getJetPlayer();
jetPlayer.loadJetFile("/sdcard/level1.jet");
byte segmentId = 0;

// queue segment 5, repeat once, use General MIDI, transpose by -1 octave
jetPlayer.queueJetSegment(5, -1, 1, -1, 0, segmentId++);
// queue segment 2
jetPlayer.queueJetSegment(2, -1, 0, 0, 0, segmentId++);

jetPlayer.play();
```
一个叫JetCreator的工具：[JetCreator User Manual](https://developer.android.com/guide/topics/media/jet/jetcreator_manual.html)  
一个样例：[JetBoy](https://android.googlesource.com/platform/development/+/master/samples/JetBoy/)  





* 原文 [http://andraskindler.com/blog/2013/create-viewpager-transitions-a-pagertransformer-example/](http://andraskindler.com/blog/2013/create-viewpager-transitions-a-pagertransformer-example/)

* young [YoungPeanut.github.io](http://youngpeanut.github.io/)

* 博客  [http://youngpeanut.github.io/2016-01-17/android-viewpager-transform/](http://youngpeanut.github.io/2016-01-17/android-viewpager-transform/)
