##使用Volley传输网络数据
Volley 是一个HTTP库，它能够帮助Android apps更方便的执行网络操作，最重要的是，它更快速高效。可以通过开源的 AOSP 仓库获取到Volley 。  
Volley 有如下的优点：

    自动调度网络请求。
    高并发网络连接。
    通过标准的HTTP的cache coherence(高速缓存一致性)使得磁盘与内存缓存不可见(Transparent)。
    支持指定请求的优先级。
    支持取消已经发出的请求。你可以取消单个请求，或者指定取消请求队列中的一个区域。
    框架容易被定制，例如，定制重试或者回退功能。
    强大的指令(Strong ordering)可以使得异步加载网络数据并显示到UI的操作更加简单。
    包含了Debugging与tracing工具。
Volley擅长执行用来显示UI的RPC操作， 例如获取搜索结果的数据。它轻松的整合了任何协议，并输出操作结果的数据，可以是raw strings，也可以是images，或者是JSON。通过提供内置你可能使用到得功能，Volley可以使得你免去重复编写样板代码，使你可以把关注点放在你的app的功能逻辑上。

Volley不适合用来下载大的数据文件。因为Volley会在解析的过程中保留持有所有的响应数据在内存中。对于下载大量的数据操作，请考虑使用`DownloadManager`。

Volley框架的核心代码是托管在AOSP仓库的frameworks/volley中，相关的工具放在toolbox下。把Volley添加到你的项目中的最简便的方法是Clone仓库然后把它设置为一个library project：  
通过下面的命令来Clone仓库：

    git clone https://android.googlesource.com/platform/frameworks/volley

以一个Android library project的方式导入下载的源代码到你的项目中。(如果你是使用Eclipse，请参考Managing Projects from Eclipse with ADT)，或者编译成一个.jar文件。
###发送简单的网络请求
1 实例化一个请求队列(RQ)   
2 实例化一个请求(R)    
3 然后将请求(Q)加到请求队列(RQ).  
    
    RequestQueue queue = Volley.newRequestQueue(this);
    String url ="http://www.google.com";

    // Request a string response from the provided URL.
    StringRequest stringRequest = new StringRequest(Request.Method.GET, url,
            new Response.Listener() {
      @Override
      public void onResponse(String response) {
        // Display the first 500 characters of the response string.
        mTextView.setText("Response is: "+ response.substring(0,500));
      }
    }, new Response.ErrorListener() {
        @Override
        public void onErrorResponse(VolleyError error) {
          mTextView.setText("That didn't work!");
        }
    });
    // Add the request to the RequestQueue.
    queue.add(stringRequest);
  
每个请求(R)可以设置一个tag(Object),可以用RQ取消这个tag的所有R.
###建立请求队列
可参考AOSP的源码Volley   
1 提供了默认的网络实现   
API>=9则使用基于`HttpURLConnection`的`HurlStack`;  
API<=8则用基于`AndroidHttpClient`的`HttpClientStack`    
2 缓存  
默认目录 data/data/pkg../cache/volley,默认大小5MB  
需要的话可以在`new DiskBasedCache(dir, cacheSize)`时指定  
另外RQ一般放到一个SingleTon Pattern中(单例)
###创建标准的网络请求
Volley框架实现了常用的请求:  
`StringRequest`, `ImageRequest`,`JsonObjectRequest`,`JsonArrayRequest`  
1 ImageRequest
 a 可以像上面的StringRequest一样在RQ中
 b 也能用一个ImageLoader
  
    new ImageLoader(RequestQueue rq, Cache imageCache)
    ImageLoader.get(url, ImageLoader.getImageListener(imageView, defaultImg, errorImg))
或者使用 `com.android.volley.toolbox.NetworkImageView`  
2 JsonXXXRequest

    JsonObjectRequest jsObjRequest = new JsonObjectRequest
        (Request.Method.GET, url, null, new Response.Listener() {

      @Override
      public void onResponse(JSONObject response) {
        mTxtDisplay.setText("Response: " + response.toString());
      }
    }, new Response.ErrorListener() {

      @Override
      public void onErrorResponse(VolleyError error) {
        // TODO Auto-generated method stub
      }
    });
###创建自定义的请求
继承`Request<T>` 然后实现方法`parseNetworkResponse()`和`deliverResponse()`
其中`parseNetworkResponse()`用来解析网络的返回数据,是在work线程执行的  
而`deliverResponse()`用来分发上面解析后的数据,是在UI线程执行的  
###参考:  
http://developer.android.com/training/volley/index.html  
http://hukai.me/android-training-course-in-chinese/connectivity/volley/index.html       