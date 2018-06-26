#Android同屏方案
##常见的同屏方案
###1 View.getDrawingCache()
最常见的**应用内**截屏方法，这个函数的原理就是通过view的Cache来获取一个bitmap对象，然后保存成图片文件，这种截屏方式非常的简单，但是局限行也很明显，首先它只能截取应用内部的界面，甚至连状态栏都不能截取到。其次是对某些view的兼容性也不好，比如webview内的内容也无法截取.

###2 读取/dev/graphics/fb0
Android是基于linux内核，所以我们也能在android中找到framebuffer这个设备，我们可以通过读取/dev/graphics/fb0这个帧缓存文件中的数据来获取屏幕上的内容，但是这个文件是**system权限**的，所以只有通过root才能读取到其中的内容，并且直接通过framebuffer读取出来的画面还需要**转换成rgb**才能正常显示
###3 反射调用SurfaceControl.screenshot()/Surface.screenshot()
SurfaceControl.screenshot()(低版本是Surface.screenshot())是系统内部提供的截屏函数，但是这个函数是**@hide**的，所以无法直接调用，需要反射调用。我尝试反射调用这个函数，但是函数返回的是null，后面发现SurfaceControl这个类也是隐藏的，所以从用户代码中无法获取这个类。也有一些方法能够调用到这个函数，比如重新编译一套sdk，或者在源码环境下编译apk，但是这种方案兼容性太差，只能在**特定ROM**下成功运行。
PS: 据Android P (9.0 preview)看, 隐藏api后续马上会加保护. 普通app无法调用.
###4 screencap -p xxx.png/screenshot xxx.png
这两个是在shell下调用的命令，通过adb shell可以直接截图，但是在代码里调用则需要**系统权限**，所以无法调用。可以看到要实现类似vysor的同步操作，可以使用这两个命令来截取屏幕然后传到电脑显示，但是这种方式非常的卡，因为这两个命令不能压缩图片，所以导致获取和生成图片的时间非常长。

###5 MediaProjection,VirtualDisplay (>=5.0)
在5.0以后，google开放了截屏的接口，可以通过”虚拟屏幕”来录制和截取屏幕，不过因为这种方式会弹出确认对话框，并且只在5.0上有效...需要demo验证.

PS:考虑到Google开放的接口,这个会比较靠谱.
##vysor原理
使用上述3,并通过adb绕过了system权限 即使用了adb shell的权限.
1 手机安装vysor的app, 然后通过adb执行指定class
```shell
adb shell export CLASSPATH=/data/app/com.zkele.andcast-1/base.apk
adb shell exec app_process /system/bin com.zkele.andcast.Main '$@'
adb forward tcp:53516 tcp: 53516
```
便会执行Main的main方法, 监听指定端口..然后再PC端连接,显示...控制是通过inputManager实现的,构造MotionEvent.




##参考
> 1 http://makaidong.com/dongweiq/13671_1501995.html  vysor原理以及同屏方案介绍
> 2 https://www.jianshu.com/p/8b313692ac85  Android录屏的方案
> 3 http://blog.csdn.net/xifei66/article/details/53816532 Android使用USB实现通信
> 4 https://github.com/fyhertz/libstreaming  Android上流媒体的一个方案

