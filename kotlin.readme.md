﻿﻿﻿﻿﻿﻿﻿﻿﻿﻿﻿﻿﻿﻿﻿﻿﻿﻿﻿﻿﻿﻿﻿﻿﻿﻿#﻿kotlin for android
##1 为何使用kotlin/kotlin的优势
###1.1 数据类

```
data class Person(var age: Int, var name: String)
```
###1.2NPE空指针

```
var name: String ?= null
name.length()//编译Error,\
name?.length() //只在非空时执行length()
name!!.length()// 编译通过, 但运行时可能有NPE异常
```
###1.3扩展函数

```java
fun Fragment.toast(msg:String, dur:Int = Toast.LENGTH_SHORT){
    Toast.makeText(getActivity(), msg, dur).show();
} //之后可以视 toast()为Fragment的方法了.
```
###1.4函数式编程 lambda
......
##2 
### 2.1配置gradle
1 [anko](https://github.com/Kotlin/anko) kotlin出的用于Android开发的工具 

2 一个 相当于了 butter knife的工具. 直接把id转为对应的控件.由 build.gradle中的apply plugin: 'kotlin-android-extensions' 提供支持

```
import kotlinx.android.synthetic.main.activity_main.*
```
###2.2类的定义

```java
class Person(var age:Int, var name:String) {
    init{ //...其中 {}和init{}可以省略如果不需要
    }
}
```
默认类都继承自Any (等同于java的Object)
kotlin的类默认是final, 除非声明为 open或abstract 才能有子类.
类的实例化,不需要 new关键字了.
###2.3基本类型
int,float,boolean等在kotlin是以对象存在的, 不能自动转换了, 需要手动转,如

```java
val i : Int = 7
val d:Double = i.toDouble()
```
位运算也从 java的 | & ^  >> << 转换为了 and or xor shl  shs等了.
String 可以像数组一样访问, 迭代

```java
val s = "Example"
val c = s[2] //即表示a
for (var a in s) {
}
```
Anko 提供的DSL: doAsync{} . uiThread{}
kotlin提供的with(){} 展开对象 
###2.4操作符重载
###2.5lambda

```java
view.setOnClickListener() {toast("clicked")}
view.setOnClickListener {toast("clicked")}
```
###2.6 kotlin Android Extensions
当前包括view的绑定, 这个插件自动创建了很多属性来方便直接访问XML中的view,
优点是不需要依赖其他库,如butterknife..
###2.7委托属性
记T为委托属性的类型,

```java
class Delegate<T> : ReadWriteProperty<Any?, T> {
    fun getValue(thisRef: Any?, property: KProperty<*>) : T {
        return ...
    }
    fun setValue(thisRef: Any?, property: KProperty<*>, value: T) {
        ...
    }
}
//下面展示 属性委托的设置
class Example {
    var p: String by Delegate()
}
```
###2.8 标准委托
Kotlin标准库中提供了一系列标准委托. Lazy
####Lazy
**lazy**包含一个lambda, 当第一次执行getValue时候,才执行这个lambda,之后都会返回一个值.

```java
class App : Application() {
    val database : SQLiteOpenHelper by lazy {
        MyDatabaseHelper(applicationContext)
    }
    override fun onCreate() {
        super.onCreate()
        val db = database.writableDatabase
    }
}
```
上面例子中, 直到第一次调用onCreate时才初始化...
lazy操作符是线程安全的. 如果不想使用线程安全,可以 lazy(LazyThreadSafeMode.NONE){}
####Observable
这个委托会检测属性的变化.

```java
class ViewModel(val db: MyDatabase) {
    var myProperty by Delegates.observable("") {
        d, old, new -> db.saveChanges(this, new)
    }
}
```

####Vetoable 
这是个特殊的observable, 有你(的lambda)决定新值是否被保存.
#### **NotNull**

```java
class App : Application() {
    componion object {
        var ins: App by Delegate.notNull()
//     var ins: App by Delegate.notNullSingleValue() 则只能修改一次.
    }
    override fun onCreate() {
        super.onCreate()
        ins = this
    }
}
```
##Anko的ManagedSqliteOpenHelper
##集合
###总数操作符
**any** 如果至少有一个元素符合条件,则为true  
**all** 所有的元素都符合条件,则为true  
**count** 返回符合条件的元素总数  
**fold** 在一个初始值的基础上从第一项到最后一项通过一个函数累计所有元素  
**foldRight**同上,但顺序是逆序  
**forEach** 遍历所有元素并执行给定的操作/函数  
**forEachIndexed**同上,但给定了index  
**max** 返回最大一项, 若没有则返回null  
**maxBy**根据给定的函数返回最大项, 若没则返回null  
**min**   **minBy**类似 上面  
**none** 如果没有元素和函数匹配则返回true  
**reduce**与fold一样,但没初始值.通过函数累计第一个到最后一个.  
**reduceRight**   
**sumBy** 返回每一项通过函数转换后的数据总和  

```java
val list = listOf(1, 2, 3, 4, 5, 6)
assertTrue(list.any { it % 2 == 0 })
assertTrue(list.all { it < 7})
assertEquals(3, list.count {it % 2 == 0})
assertEquals(22, list.fold(1) { total, next -> total + next} )
assertEquals(23, list.foldRight(2) { total, next -> total + next})
list.forEach { println(it) }
list.forEachIndexed { index, value -> println("index $index, value $value")}
assertEquals(6, list.max())
assertEquals(1, list.maxBy{-it})
//min类似max就不再写了
assertTrue(list.none { it % 8 ==0})
assertEquals(21, list.reduce { total, next -> total+next})
assertEquals(21, list.reduceRight {total,next->total+next})
assertEquals(3, list.sumBy{it %2})
```

###过滤操作符
**drop** 返回去掉前n个元素(包含)的 列表  
**dropWhile** 返回根据函数从第一项开始去掉指定元素的列表  
**dropLastWhile** 同上,但从后面执行  
**filter** 过滤所有符合条件的元素  
**filterNot** 过滤所有 不符合条件的元素  
**filterNotNull** 过滤所有非null的元素  
**slice** 过滤一个list中指定index的元素  
**take** 返回从第一个开始的n个元素  
**takeLast**返回从最后一个开始的n个元素  
**takeWhile**返回从第一个开始符合条件的元素  

```java
assertEquals(listOf(5,6),  list.drop(4))
assertEquals(listOf(5,6),  list.dropWhile{i <5})
//dropLastWhile貌似和dropWhile效果一样,只是删除顺序不一样
assertEquals(listOf(2, 4, 6), list.slice(listOf(1, 3, 5)) )
```
###映射操作符
**flatMap** 遍历所有元素, 为每个创建一个集合,最后把所有集合放一起 (并集)  
**groupBy**返回一个根据函数分组后的map  
**map** 返回一个 每个元素根据函数转换所组成的List  
**mapIndexed** 同上,但函数可用index  
**mapNotNull** 返回一个 每个非null元素根据函数转换 所组成的list  
###元素操作符
**contains** 包含
**elementAt** 返回指定的元素, 若越界,则抛 IndexOutofBoundsException  
**elementAtOrElse**返回指定元素,若越界则返回函数的返回值  
**elementAtOrNull**返回指定元素,若越界则返回null  
**first**返回第一个符合条件的元素  
**firstOrNull**同上,若没有符合的则返回Null  
**indeOf**  
**indexOfFirst**  
**indexOfLast**  
**last**  返回符合条件的最后一个元素  
**lastIndexOf**  
**lastOrNull**  
**single**  返回符合条件的单个元素,若没有或超过一个,则抛异常  
**singleOrNull**  返回符合条件的单个元素,若没有或超过一个,则null  
###生产操作符
**merge** 合并集合, 相同index的元素, 通过函数合并为新的元素..新集合大小由最小集合大小决定  
**partition** 分割集合, 第一个集合由符合条件的元素组成,第二个集合由不符合条件的元素组成  
**plus** 并集,, 可以使用操作符 + 代替  
**zip** 返回由pair组成的list, 每个pair是2个集合中相同index元素组成.. 集合size由最小集合决定  
**unzip** 从包含pair的list中生成包含list的pair  
###顺序操作符
**reverse** 反序  
**sort** 排序(升序)  
**sortBy** 根据指定函数排序的list   
**sortDescending** 排序(降序)   
**sortDescendingBy** 根据指定函数(降序)排序后的list  
##空指针的避免
指定一个变量可能null是 在类型后增加一个问号码. Kotlin中一切皆是对象,甚至java中的原始数据类型,一切都是可null的. 如果一个可null变量在不检查null时使用它 编译时不通的.

##接口和委托
##泛型 变体

#参考
1.  https://github.com/wangjiegulu/kotlin-for-android-developers-zh/blob/master/SUMMARY.md
```