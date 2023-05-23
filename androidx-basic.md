# 1-lifecycle系列

https://developer.android.com/topic/libraries/architecture/livedata?hl=zh-cn

https://developer.android.com/topic/libraries/architecture/viewmodel?hl=zh-cn

## 1-1 LiveData

LiveData 是一种可观察的数据存储器类。可确保 LiveData 仅更新处于活跃生命周期状态的应用组件观察者。

```java
//基于
public abstract class LiveData<T> {}
public class MutableLiveData<T> extends LiveData<T>{}
//postValue(){} 子线程
//setValue(){} 主线程
```

## 1-2 ViewModel

目的:以注重生命周期的方式存储和管理界面相关的数据。ViewModel 类让数据可在发生屏幕旋转等配置更改后继续留存。

ViewModel经常与LiveData一块使用,

**注意**：[`ViewModel`](https://developer.android.com/reference/androidx/lifecycle/ViewModel?hl=zh-cn) 绝不能引用视图、[`Lifecycle`](https://developer.android.com/reference/androidx/lifecycle/Lifecycle?hl=zh-cn) 或可能存储对 Activity 上下文的引用的任何类。

[`ViewModel`](https://developer.android.com/reference/androidx/lifecycle/ViewModel?hl=zh-cn) 对象存在的时间范围是获取 [`ViewModel`](https://developer.android.com/reference/androidx/lifecycle/ViewModel?hl=zh-cn) 时传递给 [`ViewModelProvider`](https://developer.android.com/reference/androidx/lifecycle/ViewModelProvider?hl=zh-cn) 的 [`Lifecycle`](https://developer.android.com/reference/androidx/lifecycle/Lifecycle?hl=zh-cn)(Activity or Fragment),

ViewModel得以在Activity销毁重建可复用在于
Activity.onRetainNonConfigurationInstance() -- 销毁前在onStop后-onDestory前调用了这个
Activity.getLastNonConfigurationInstance()  -- 第二个Activity实例创建时调用返回了viewModelStore,包含VM的map

