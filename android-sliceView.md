# SliceView

## 1.简述
- 能够显示丰富,动态,可交互的UI模板.
- 便于在Google搜索,Google助手等显示.
- 可以增强App Actions.
- 支持API 19+.
个人理解:
- 用于在搜索中展示结果,(跨应用的),
- 相对于AppWidget来说,是临时显示.
- 支持的UI模板有限! HeaderBuilder RowBuilder GridBuilder RangeBuilder InputRangeBuilder

## 2.源码分析
依赖库的关系如下:
```bash
+--- androidx.slice:slice-view:1.0.0
|    +--- androidx.lifecycle:lifecycle-livedata-core:2.0.0 (*)
|    +--- androidx.slice:slice-core:1.0.0
|    \--- androidx.recyclerview:recyclerview:1.0.0

+--- androidx.slice:slice-builders-ktx:1.0.0-alpha6
|    +--- androidx.slice:slice-builders:1.0.0
|    +--- androidx.slice:slice-core:1.0.0 (*)
|    \--- androidx.core:core:1.0.0 (*)
```
提供方-实现/继承 androidx.slice.SliceProvider
通过 provider.call()方法,最终调onBindSlice,将Slice返回给显示方.

显示方-通过查询uri,确定provider,
SliceLiveData.observer().
拿到Slice后, 直接SliceView.setSlice(s);//来更新UI.
//更新UI时,通过动态的addView,设置RecyclerView等来完成.
//所以,不适合显示动态更新的slice.--显示搜索结果,所以不太需要动态更新,差分化更新.


## 3.参考
Google文档:     https://developer.android.com/guide/slices
Google代码样例: https://github.com/googlesamples/android-SliceViewer.git