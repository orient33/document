# Dagger2学习记录. (不是总结)
## 1.简述
1.Dagger2是Google在 square的dagger基础上维护,扩展的.
2.Dagger2是基于jsr330的javax.inject. (JSR是java规范), 是javax.inject的一种实现
3.构造/提供依赖; 使用依赖; 以及两者的桥梁
## 2.基础 Inject和Component
1.Inject 注解在类的属性时, 为使用依赖.(需求方)
2.Inject 注解在类的构造方法时, 为提供依赖.(供给)
3.Component是链接供需的纽带, 用于注解在一个接口interface上, 由于apt生成实现类.  

  使用上面2个注解基本上可以实现了DI. 但是为了解决三方库的问题, 又需要使用下面2个:

4.Provider 即提供依赖,是为了解决三方库的class构造方法,不能加Inject注解的问题.
5.Module 是注解在 interface 或 abstract class上, 其method可以使用Provider注解.

Provider和Module必须一块使用,才可以提供依赖.
那么Module必须声明在Component注解的参数中,才可以被识别

## 3.Android中的Dagger2
类似MainActivity的属性,可以用Inject注解表示使用依赖. 那么MainActivity的构造是由framework实现的,
Dagger2是如何处理的这类注入呢?
由DaggerApplication, DaggerAppcompatActivity,DaggerFragment,等实现的了.
DaggerApplication的applicationInjector() 需要一个继承自AndroidInjector的AppComponent 接口,
和AppComponent的参数中包含的XXModule, 可查看生成的DaggerAppComponent代码.  

从代码执行逻辑上看, DaggerAppCompatActivity.onCreate()
 -> AndroidInjection.inject(this);
 -> DaggerAppComponent.MainActivitySubcomponentImpl.inject()
 -> injectMainActivity()
 -> MainActivity_MembersInjector.injectMembers()


(这里跟参考1的不太一样. 根据参考2/3的代码阅读,)

## 4.其他注解
### Singleton 单例
用 @Singleton 标注在目标单例上，然后用 @Singleton 标注在 Component 对象上。
如果要以 @Provides 方式提供单例的话，需要用 @Singleton 注解依赖提供的方法.
(费解的话可以运行下,通过log看看object的hashCode是否单例)
### Scope
Singleton 是一个注解，但是它被一个元注解Scope 注解了,
而Scope的作用范围是由其所在的Component决定的,
### Qualifiers和Name
Name是由Qualifier元注解修饰的, 一般的是由Name亦可.
如果provider的依赖为一个类的2个实例, 那么单Inject无法区分, 那么可以是由Name来区分.
自定义一个注解,(Qualifiers),只是方便一点,跟Name("xx")效果一样.

### Lazy延迟加载
Lazy<T> name; 只会在首次name.get()时才构造T,并返回
### Subcomponent
@Subcomponent(modules=AModule.class)  相当于继承,
Subcomponent同时具备了parent的scope. 一般的,使用dependencies更好.
即, **组合由于继承**


## 5.其他
### Component之间的依赖
使用dependencies参数标明依赖即可,如下,BComponent就可以使用AComponent中的依赖元素了
```java
@Module
public class AModule{
    @Provides
    public A provideA(){
        return new A();
    }
}
@Component(modules=AModule.class)
public interface AComponent{
    A provideA();
}
@Module
public class BModule{
    @Provides
    public B provideB(){
        return new B();
    }
}
@Component(modules=BModule.class, dependencies=AComponent.class)
public interface BComponent{
    void inject(C c);
}
```

## 参考
1. CSDN博客资料 https://blog.csdn.net/briblue/article/details/75578459
2. Google的mvp+dagger2样例https://github.com/googlesamples/android-architecture/tree/todo-mvp-dagger/
3. 我的demo https://github.com/orient33/daggerDemo
4. 官方GitHub https://github.com/google/dagger
5. 官方文档 https://dagger.dev/
6. 官方文档,Android特例 <https://dagger.dev/android>