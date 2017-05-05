# ExoPlayer分析  基于 2.4.0 release #
很早就想看看Player相关的源码了，(MediaPlayer已经看过)

## 0) 从使用开始 ##
   demo中的PlayerActivity.java
   onStart()或onResume()中init,对应在onStop()或onPause()中release()--根据是否是android-7.0

   ```java
   void initializePlayer() {
       UUID drm... //数字版权相关/加密相关 目前忽略这部分
	   int extensionRendererMode 0-1-2//是否使用扩展库 如ffmpeg vp9等
	   new DefaultRenderersFactory(this, drm, extensionRendererMode); //渲染
	   //下面是轨道选择， 比如demo中很多video都有多种分辨率和声音 即清晰度和音轨的选择
	   TrackSelection.Factory videoTrackSelectionFactory =new AdaptiveTrackSelection.Factory(BANDWIDTH_METER);
	   new DefaultTrackSelector(videoTrackSelectionFactory);
	   new TrackSelectionHelper(trackSelector, videoTrackSelectionFactory);//选择Button的辅助
	   player = ExoPlayerFactory.newSimpleInstance(renderersFactory, trackSelector);
	   ...... //各种监听listener
	   view.setPlayer(player); //设置surface
	   ......
	   player.prepare(mediaSource);
   }
   ```

## 1)player实例化 ##
  ExoPlayerFactory.newSimpleInstance()时
  ```java
  new SimpleExoPlayer(){ // 先创建若干
    Renderer[] renderers = renderersFactory.createRenderers(...)
	  buildVideoRenderers() -> new MediaCodecVideoRenderer() 可选LibvpxVideoRenderer
	  buildAudioRenderers() -> new MediaCodecAudioRenderer() 可选opus,flac,ffmpeg
	  buildTextAudioRenderers()  -> new TextRenderer()
	  buildMetadataRenderers()    -> new MetadataRenderer()
	  buildMiscellaneousRenderers() //nothing
	
	player = new ExoPlayerImpl(renderers, trackSelector, loadControl);
  }
  ```
  ```java
  class ExoPlayerImpl {
    ExoPlayerImpl(){
      eventHandler = new Handler()... ExoPlayerImpl.this.handleEvent(msg);
      internalPlayer = new ExoPlayerImplInternal();
    }
  
    void prepare(MediaSource s){
	  internalPlayer.prepare(mediaSource, resetPosition);
	}
    ...
	...
    //主线程，主要是回调EventListener方法
    void handleEvent(Message msg){
	  /* External messages
      MSG_PREPARE_ACK = 0                       pendingPrepareAcks--;
      MSG_STATE_CHANGED = 1;                 onPlayerStateChanged(playWhenReady, playbackState);
      MSG_LOADING_CHANGED = 2;               onLoadingChanged(isLoading);
      MSG_TRACKS_CHANGED = 3;                onTracksChanged(trackGroups, trackSelections);
      MSG_SEEK_ACK = 4;                      onPositionDiscontinuity();
      MSG_POSITION_DISCONTINUITY = 5;        onPositionDiscontinuity();
      MSG_SOURCE_INFO_REFRESHED = 6;         onTimelineChanged(timeline, manifest);
      MSG_PLAYBACK_PARAMETERS_CHANGED = 7;   onPlaybackParametersChanged(playbackParameters);
      MSG_ERROR = 8;                         onPlayerError(exception);
	  */
	}
  }
  ```
  ```java
  class ExoPlayerImplInternal {
    ExoPlayerImplInternal(){
	  ...
	  internalPlaybackThread = new HandlerThread("ExoPlayerImplInternal:Handler",
        Process.THREAD_PRIORITY_AUDIO); //优先级极高
      internalPlaybackThread.start();
      handler = new Handler(internalPlaybackThread.getLooper(), this); //这里this指Handler.Callback接口
	}
	...
	//工作线程
	public boolean handleMessage(Message msg){
	  /*
	  MSG_PREPARE = 0                                prepareInternal((MediaSource) msg.obj, msg.arg1 != 0);
      MSG_SET_PLAY_WHEN_READY = 1
      MSG_DO_SOME_WORK = 2                           doSomeWork();
      MSG_SEEK_TO = 3
      MSG_SET_PLAYBACK_PARAMETERS = 4
      MSG_STOP = 5
      MSG_RELEASE = 6
      MSG_REFRESH_SOURCE_INFO = 7
      MSG_PERIOD_PREPARED = 8
      MSG_SOURCE_CONTINUE_LOADING_REQUESTED = 9
      MSG_TRACK_SELECTION_INVALIDATED = 10
      MSG_CUSTOM = 11
      */
	}
  }
  ```
  到此player实例化就完成了
  
## 2 player.prepare(mediaSource) ##
    -> ExoPlayerImp.prepare()
	   -> ExoPlayerImplInternal.prepare(){
	         handler.obtainMessage(0,0,source).sendToTarget();
	      }
  即 在ExoPlayerImplInternal发送prepare消息，然后在工作线程
  ```java
  prepareInternal(source) {
    source.prepareSource(player...)
	setState(BUFFERING);
	handler.sendEmptyMessage(DO_SOME_WORK);
  }
  ```
  
## 3 doSomeWork(){ ##
    ```java
    //更新position
	for(Renderer renderer : enabledRenderers) {
	  renderer.render(posUs, realtimeUs);
	  ...
	}
    ...
	if (allRenderEnded ..&& isLast) {
	  setState(END)
	  stopRenderers();
	} else if (BUFFERING) {
	  boolean isNewlyReady = ...;
	  if (isNewlyReady) {
	    setState(READY);
	  }
	} else if (READY) {
	  boolean isStillReady = ...;
	  if (!isStillReady) {
	    setState(BUFFERING);
		stopRenderers();
	  }
	}
	
	if (BUFFERING) {
	   for (Renderer render : enabledRenderers) {
	     render.maybeThrowStreamError();
	   }
	}

	if ( (playWhenReady&&READY) || BUFFERING) {
	  scheduleNextWork(startTime, 10ms);
	} else if (enabledRenderers.length != 0) {
	  scheduleNextWork(startTime, 1000ms);
	} else {
	  handler.removeMessage(DO_SOME_WORK);
	}
  }
 
  scheduleNextWork(){
    handler.removeMessages(MSG_DO_SOME_WORK);
    long nextOperationStartTimeMs = thisOperationStartTimeMs + intervalMs;
    long nextOperationDelayMs = nextOperationStartTimeMs - SystemClock.elapsedRealtime();
    if (nextOperationDelayMs <= 0) {
      handler.sendEmptyMessage(MSG_DO_SOME_WORK);
    } else { //大多数情况是延迟 xx毫秒 循环发送DO_SOME_WORK
      handler.sendEmptyMessageDelayed(MSG_DO_SOME_WORK, nextOperationDelayMs);
    }
  }
  ```

## 4 renderer.render ## 
  以Video为例， MediaCodecVideoRenderer
  render()执行其父类 MediaCodecRender的方法
  ```java
  public void render(long pos,long elapsedRealtime){
    ...
	maybeInitCodec(); //若codec空，则实例化它
	if (codec != null) {
	  while (drainOutputBuffer(positionUs, elapsedRealtimeUs)) {} //消费outputBuffer,更新surface
      while (feedInputBuffer()) {}                                //生产 inputBuffer
	} else {
	  skipSource(pos);
	  ...
	}
	decoderCounters.ensureUpdated();
  }
  ```
## 5 maybeInitCodec() { //根据格式实例化MediaCoderc ##
   ```java {
    ...
	decoderInfo = getDecoderInfo(mediaCodecSelector, format, drmSessionRequiresSecureDecoder);
	codec = MediaCodec.createByCodecName(decoderInfo.name); //根据name实例化MediaCodec
	configureCodec(decoderInfo, codec, format, cryto);   //配置
	codec.start();
	inputBuffer = codec.getInputBuffer();
	outputBuffer = codec.getOutputBuffer();
	...
  }
  ```
  
## 6 drainOutputBuffer(positionUs, elapsedRealtimeUs) ##
	```java
	{...
    processOutputBuffer(.......); //抽象方法，实际调用 MediaCodecVideoRenderer
	...
    }
    ```

## 7 MediaCodecVideoRenderer.processOutputBuffer()  ##
  ```java
  {  
    ...
	if (Util.SDK_INT >= 21) {
      renderOutputBufferV21(codec, bufferIndex, System.nanoTime());
    } else {
      renderOutputBuffer(codec, bufferIndex);
    }
  }
  根据SDK_INT 执行 {
    codec.releaseOutputBuffer(bufferIndex, true);//android-5 以下
	codec.releaseOutputBuffer(bufferIndex, releaseTimeNs); // 5.0以及以上
	//这里会将bufferIndex对应的帧 渲染到 Surface上
  }
  ```
## 8 对于音频MediaCodecAudioRender.processOutputBuffer()  ##
  ```java
  {
    ...
	if (audioTrack.handleBuffer(buffer, bufferPresentationTimeUs)) {//使用AudioTrack播放
	  codec.releaseOutputBuffer(bufferIndex, false);//这里只释放不渲染surface
	}
  }
  ```
  
  AudioTrack最终调用 android.media.AudioTrack.write(buffer, size, .); //不同SDK_INT调用有差异
-----------------------------------------------------------------------------
目前看到这里，发现还需要看下 MediaCodec相关的API。
还没明白的地方：
  虽然看到了 渲染 发生在 线程"ExoPlayerImplInternal:Handler" 上，
  但缓存的IO呢？ 应该不是在同一线程的，是在哪里呢？(尤其网络视频)
