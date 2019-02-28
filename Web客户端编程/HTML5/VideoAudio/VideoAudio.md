# html5多媒体播放

html5之前，播放视频/音频有两种方法：安装指定播放器插件或使用flash（也需要安装flash插件）。html5引入了video和audio标签，用于播放视频和音频，播放器由浏览器实现，同时提供一套API，用于对播放器进行定制。

浏览器多媒体支持格式可能不同，firefox的支持格式可以在MDN中查看。

## 处理不同浏览器

由于不同浏览器支持多媒体编码格式可能不同，因此video/audio标签允许指定多个多媒体资源地址。

```html
<video controls>
  <source src="somevideo.webm" type="video/webm">
  <source src="somevideo.mp4" type="video/mp4">
  I'm sorry; your browser doesn't support HTML5 video in WebM with VP8/VP9 or MP4 with H.264.
  <!-- You can embed a Flash player here, to play your mp4 video in older browsers -->
</video>
```

## 属性

* src：媒体资源url地址
* autoplay：是否在页面加载后自动播放
* preload：预加载选项
  * none：不进行预加载
  * metadata：只加载元数据（字节数，第一帧，播放列表，持续时间等）
  * auto：预加载全部
* poster：video独有，视频播放前展示给用户，值是一个图片的url
* loop：是否循环播放
* controls：是否显示浏览器自带的播放控制条
* width/height：video独有，指定视频宽度高度
* error：正常情况下error为null，出错时error被指定MediaError对象，该对象code表示错误状态。code取值：
  * MEDIA_ERR_ABORTED 1 下载过程中被用户终止
  * MEDIA_ERR_NETWORK 2 由于网络原因下载过程终止
  * MEDIA_ERR_DECODE 3 解码过程中出错
  * MEDIA_ERR_SRC_NOT_SUPPORTED 4 媒体格式不被支持
* networkState：用于读取当前网络状态，取值
  * NETWORK_EMPTY 0 元素处于初始化状态
  * NETWORK_IDLE 1 浏览器已经选择好用什么编码格式播放多媒体数据了，但还未下载
  * NETWORK_LOADING 2 媒体数据加载中
  * NETWORK_NOSOURCE 3 没有支持的编码格式，不执行加载
* currentSrc：只读，读取当前媒体数据URL
* buffered：实现了TimeRanges的对象，只读，表示缓存的媒体数据的时间范围。
* readyState：媒体当前播放位置的就绪状态，只读
  * HAVE_NOTHING 0 当前播放位置没有可播放数据
  * HAVE_METADATA 1 只有元数据
  * HAVE_CURRENT_DATA 2 当前帧数据已获得，但还没获得能够继续播放的数据
  * HAVE_FUTURE_DATA 3 当前帧和下一帧数据已获得
  * HAVE_ENOUGH_DATA 4 已经获得能持续播放的数据
* seeking：返回布尔值，表示浏览器是否正在请求某一特定位置的播放数据
* vseekable：返回TimeRanges对象，表示请求倒的数据的时间范围
* currentTime/startTime/duration：分别是读取/修改当前播放位置，读取播放开始时间（通常为0），读取播放总时间，单位均为秒
* played/paused/ended：played返回TimeRanges对象，表示已播放时段，paused返回暂停还是正在播放，ended表示是否播放完毕
* defaultPlaybackRate/playbackRate：读取或修改默认播放速率/当前播放速率。
* volume：音量，取值0-1，0为静音
* muted：静音状态

## 方法

* play()：播放
* pause()：暂停
* load()：重新加载，playbackRate改为defaultPlaybackRate，error归为null
* canPlayType(type)：测试浏览器是否支持指定的媒体类型，参数type和source标签中的type相同，返回值为字符串：
  * ""：不支持
  * "maybe"：可能支持
  * "probably"：支持

## 事件


* loadStart：浏览器开始在网上寻找媒体数据
* progress：正在获取媒体数据
* suspend：媒体数据被阻止加载
* abort：用户终止了媒体数据加载
* error：加载媒体数据出错
* emptied：载入媒体过程中发生致命错误，或浏览器正在选择播放格式时，又调用了load方法重新载入媒体
* stalled：尝试获取媒体数据失败
* play：开始播放，执行play()时触发，或autoplay时触发
* pause：暂停，执行pause()时触发
* loadedmetadata：浏览器已获取元数据
* loadeddata：播放数据已加载完毕
* wating：播放过程因得不到下一帧而暂停（下一帧未加载完毕）
* playing：正在播放
* canplay：能够播放，但还需要缓冲
* canplayenough：能够播放全部
* seeking：正在请求特定位置数据
* seeked：停止请求特定位置数据
* timeupdate：当前播放位置被改变，可能是人为改变
* ended：播放结束后停止播放
* ratechange：播放速率被改变
* durationchange：播放时长被改变
* volumechange：音量或静音状态被改变
