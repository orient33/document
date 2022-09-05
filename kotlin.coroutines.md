# 协程
参考
1. https://blog.csdn.net/Jason_Lee155/article/details/107895920
2. https://developer.android.google.cn/kotlin/coroutines/coroutines-adv?hl=zh-cn
3. https://developer.android.google.cn/kotlin/flow?hl=zh-cn
## 创建/开启 协程

全局的, launch { } , runBlocking{} 前者是不阻塞, 后者会阻塞
 前者是CoroutineScope的扩展方法, 
 public fun CoroutineScope.launch(
    context: CoroutineContext = EmptyCoroutineContext,
    start: CoroutineStart = CoroutineStart.DEFAULT,
    block: suspend CoroutineScope.() -> Unit
): Job 
 后者是 public actual fun <T> runBlocking(context: CoroutineContext, block: suspend CoroutineScope.() -> T): T 

可返回结果的 withContext 与 async
  前者在多个时,为串行的, ,后者是并行的,见 参考1

or
可以通过以下两种方式来启动协程：
launch 可启动新协程而不将结果返回给调用方。任何被视为“一劳永逸”的工作都可以使用 launch 来启动。
async 会启动一个新的协程，并允许您使用一个名为 await 的挂起函数返回结果。

## 协程概念
### CoroutineScope
CoroutineScope 会跟踪它使用 launch 或 async 创建的所有协程。您可以随时调用 scope.cancel() 以取消正在进行的工作（即正在运行的协程）。
在 Android 中，某些 KTX 库为某些生命周期类提供自己的 CoroutineScope。例如，ViewModel 有 viewModelScope，Lifecycle 有 lifecycleScope。
不过，与调度程序不同，CoroutineScope 不运行协程。
创建scope的方法如下 
  val scope = CoroutineScope(Job() + Dispatchers.Main) 
作业
Job 是协程的句柄。使用 launch 或 async 创建的每个协程都会返回一个 Job 实例，该实例是相应协程的唯一标识并管理其生命周期
### CoroutineContext
- Job：控制协程的生命周期。
- CoroutineDispatcher：将工作分派到适当的线程。
- CoroutineName：协程的名称，可用于调试。
- CoroutineExceptionHandler：处理未捕获的异常。

如下,创建scope时需要一个CoroutineContext 
public fun CoroutineScope(context: CoroutineContext): CoroutineScope =
    ContextScope(if (context[Job] != null) context else context + Job())


## 在 Android 中使用协程的最佳做法

### 注入调度程序
在创建新协程或调用 withContext 时，请勿对 Dispatchers 进行硬编码。

### 挂起函数应该能够安全地从主线程调用
挂起函数应该是主线程安全的，这意味着，您可以安全地从主线程调用挂起函数。如果某个类在协程中执行长期运行的阻塞操作，那么该类负责使用 withContext 将执行操作移出主线程。这适用于应用中的所有类，无论其属于架构的哪个部分都不例外。
### ViewModel 应创建协程
ViewModel 类应首选创建协程，而不是公开挂起函数来执行业务逻辑。如果只需要发出一个值，而不是使用数据流公开状态，ViewModel 中的挂起函数就会非常有用。
### 不要公开可变类型
最好向其他类公开不可变类型。这样一来，对可变类型的所有更改都会集中在一个类中，便于在出现问题时进行调试。
class LatestNewsViewModel : ViewModel() {
    private val _uiState = MutableStateFlow(LatestNewsUiState.Loading)
    val uiState: StateFlow<LatestNewsUiState> = _uiState
}
### 避免使用 GlobalScope
这类似于“注入调度程序”最佳做法。

### 层和业务层应公开挂起函数和数据流
数据层和业务层中的类通常会公开函数以执行一次性调用，或接收数据随时间变化的通知。这些层中的类应该针对一次性调用公开挂起函数，并公开数据流以接收关于数据更改的通知。
### 将协程设为可取消
协程取消属于协作操作，也就是说，在协程的 Job 被取消后，相应协程在挂起或检查是否存在取消操作之前不会被取消。如果您在协程中执行阻塞操作，请确保相应协程是可取消的。
例如，如果您要从磁盘读取多个文件，请先检查协程是否已取消，然后再开始读取每个文件。若要检查是否存在取消操作，有一种方法是调用 ensureActive 函数。
someScope.launch {
    for(file in files) {
        ensureActive() // Check for cancellation
        readFile(file)
    }
}

## 协程与线程池
### 线程池 -> CoroutineDispater

public fun ExecutorService.asCoroutineDispatcher(): ExecutorCoroutineDispatcher = ExecutorCoroutineDispatcherImpl(this)
public fun Executor.asCoroutineDispatcher(): CoroutineDispatcher =
    (this as? DispatcherExecutor)?.dispatcher ?: ExecutorCoroutineDispatcherImpl(this)

android中 Handler

public fun Handler.asCoroutineDispatcher(name: String? = null): HandlerDispatcher =
    HandlerContext(this, name)

### CoroutineDispater -> 线程池

public fun CoroutineDispatcher.asExecutor(): Executor =
    (this as? ExecutorCoroutineDispatcher)?.executor ?: DispatcherExecutor(this)

# 数据流
1. https://developer.android.google.cn/kotlin/flow?hl=zh-cn
## Flow