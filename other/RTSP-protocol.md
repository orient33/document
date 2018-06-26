##RTSP (Real Time Streaming Protocol)

**实时流协议**（Real Time Streaming Protocol）是一种网络应用协议.

流数据本身的传输不是RTSP的任务。大多数RTSP服务器使用实时传输协议（RTP）和实时控制协议（RTCP）结合媒体流传输。

RTSP由RealNetworks公司，Netscape公司和哥伦比亚大学开发，第一稿于1996年提交给IETF。由互联网工程任务组（IETF ）的多方多媒体会话控制工作组（MMUSIC WG）进行了标准化），并于1998年发布为RFC 2326。 RTSP 2.0 于2016年发布为RFC 7826，作为RTSP 1.0的替代品。RTSP 2.0基于RTSP 1.0，但不是在基本版本协商机制之外的向后兼容。

在某些方面与HTTP类似，RTSP定义了控制多媒体播放控制顺序。虽然HTTP是无状态的，但RTSP具有状态; 当需要跟踪并发会话时使用标识符。像[HTTP](https://zh.wikipedia.org/wiki/HTTP)一样，RTSP使用TCP来维护端到端连接，而大多数RTSP控制消息由客户端发送到服务器，一些命令沿着另一个方向（即从服务器到客户端）传播。

###基本的RTSP请求
#### OPTIONS请求
请求返回服务器将接受的请求类型。  (C 代表客户端 S 代表服务端)
```
C->S:  OPTIONS rtsp://example.com/media.mp4 RTSP/1.0
       CSeq: 1
       Require: implicit-play
       Proxy-Require: gzipped-messages

S->C:  RTSP/1.0 200 OK
       CSeq: 1
       Public: DESCRIBE, SETUP, TEARDOWN, PLAY, PAUSE
```
####DESCRIBE 请求
DESCRIBE请求包括RTSP URL（rtsp:// ...）以及可以处理的回复数据类型。该回复包括呈现描述，通常以会话描述协议（SDP）格式。其中，演示文稿描述列出了使用汇总网址控制的媒体流。在典型的情况下，每个音频和视频都有一个媒体流。
```
C->S: DESCRIBE rtsp://example.com/media.mp4 RTSP/1.0
      CSeq: 2

S->C: RTSP/1.0 200 OK
      CSeq: 2
      Content-Base: rtsp://example.com/media.mp4
      Content-Type: application/sdp
      Content-Length: 460

      m=video 0 RTP/AVP 96
      a=control:streamid=0
      a=range:npt=0-7.741000
      a=length:npt=7.741000
      a=rtpmap:96 MP4V-ES/5544
      a=mimetype:string;"video/MP4V-ES"
      a=AvgBitRate:integer;304018
      a=StreamName:string;"hinted video track"
      m=audio 0 RTP/AVP 97
      a=control:streamid=1
      a=range:npt=0-7.712000
      a=length:npt=7.712000
      a=rtpmap:97 mpeg4-generic/32000/2
      a=mimetype:string;"audio/mpeg4-generic"
      a=AvgBitRate:integer;65790
      a=StreamName:string;"hinted audio track"
```
####SETUP 请求
SETUP请求指定如何传输单个媒体流。这必须在发送PLAY请求之前完成。请求包含媒体流URL和传输说明符。该说明符通常包括用于接收RTP数据（音频或视频）的本地端口，另一个用于RTCP数据（元信息））。服务器回复通常会确认所选参数，并填写缺少的部分，例如服务器选择的端口。必须在发送聚合播放请求之前，使用SETUP配置每个媒体流。
```
C->S: SETUP rtsp://example.com/media.mp4/streamid=0 RTSP/1.0
      CSeq: 3
      Transport: RTP/AVP;unicast;client_port=8000-8001

S->C: RTSP/1.0 200 OK
      CSeq: 3
      Transport: RTP/AVP;unicast;client_port=8000-8001;server_port=9000-9001;ssrc=1234ABCD
      Session: 12345678
```
####Play 播放请求
Play 播放请求 将导致播放一个或所有媒体流。可以通过发送多个播放请求来堆叠播放请求。URL可以是聚合URL（播放所有媒体流）或单个媒体流URL（仅播放该流）。可以指定范围。如果没有指定范围，流将从头开始播放，并播放到最后，或者如果流暂停，则在暂停点恢复播放。
```
C->S: PLAY rtsp://example.com/media.mp4 RTSP/1.0
      CSeq: 4
      Range: npt=5-20
      Session: 12345678

S->C: RTSP/1.0 200 OK
      CSeq: 4
      Session: 12345678
      RTP-Info: url=rtsp://example.com/media.mp4/streamid=0;seq=9810092;rtptime=3450012
```
####PAUSE 暂停请求
PAUSE 暂停请求 暂时停止一个或所有媒体流，因此稍后可以通过播放请求恢复。请求包含聚合或媒体流URL。PAUSE请求中的范围参数指定何时暂停。当省略范围参数时，暂停会立即无限期地发生。
...
####RECORD 记录请求
RECORD 该方法根据呈现描述开始记录一系列媒体数据。时间戳反映开始和结束时间（UTC）。如果没有给定时间范围，请使用演示文稿描述中提供的开始或结束时间。如果会话已经开始，请立即开始录制。服务器决定是否将记录的数据存储在请求URI或其他URI下。如果服务器不使用请求URI，则响应应为201，并包含描述请求状态并引用新资源的实体以及Location头。
...
####ANNOUNCE 发布请求
ANNOUNCE 发布方法有两个目的：
从客户端发送到服务器时，ANNOUNCE将请求URL标识的演示文稿或媒体对象的描述发布到服务器。当从服务器发送到客户端时，ANNOUNCE会实时更新会话描述。如果新的媒体流被添加到演示文稿中（例如，在实时演示中），则应该再次发送整个演示文稿描述，而不仅仅是附加的组件，以便可以删除组件。(下面邮箱'#'号替换成'@')
...
####TEARDOWN 停止发布流请求
TEARDOWN 请求用于终止会话。它停止所有媒体流，并释放所有与会话相关的数据在服务器上。
```
C->S: TEARDOWN rtsp://example.com/media.mp4 RTSP/1.0
      CSeq: 8
      Session: 12345678

S->C: RTSP/1.0 200 OK
      CSeq: 8
```

###参考
https://zh.wikipedia.org/wiki/即時串流協定
http://blog.csdn.net/gg_simida/article/details/72678561

###RTSP Client
#### android 上.
按 https://developer.android.google.cn/guide/topics/media/media-formats.html#network看, Android支持RTSP协议的. 使用 android.media.MediaPlayer播放部分RTSP流可以成功. 
可播放  RTSP://wowzaec2demo.streamlock.net/vod/mp4:BigBuckBunny_115k.mov
不播放 RTSP://192.168.33.28:1234
AOSP中RTSP的解析 ::??
frameworks/av/media/libmediaplayerservice/nuplayer/

