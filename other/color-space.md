android上的颜色
# android 8.0 (api 26)开始
支持了颜色空间 ColorSpace, 以广色域P3为例,如果要开启使用, 需要设备软硬件的支持.
但是 启用广色域后, Activity需要更多内存及GPU资源。全屏显示图片的Activity更适合
广色域模式,显示较小图的Activity则不适合开启广色域。

# android.graphics.Color
## 1-int类型
使用最多的sRGB格式(从android 1.0 开始), 32bit的int代表一个颜色
0x12345678 其中12是alpha,34/56/78分别是RGB
由aRGB转换int的方法如下
int color = (A & 0xff) << 24 | (R & 0xff) << 16 | (G & 0xff) << 8 | (B & 0xff)

## 2-long类型
android 8引入的 共计64bit代表一个颜色
其中
0x0123456789abcdef
若是XYZ模型 其中
 cdef代表 Z
 89ab代表 Y
 4567代表 X
 0123这16bit分2部分 前6bit代表颜色空间index ColorSpace
                    后10bit代表alpha
## 3-Color对象实例

# android.graphics.ColorSpace
根据索引 从[0,63]范围, 目前android定义了[0,15]共计16种颜色空间
由枚举ColorSpace.Named 定义
ColorSpace.Model 定义了4中颜色模型:
CMYK, LAB, RGB, XYZ
最终 一个ColorSpace由 name Model Named 三个值定义/决定的.



# 参考如下官方文档
https://developer.android.google.cn/training/wide-color-gamut?hl=zh-cn
https://developer.android.google.cn/reference/android/graphics/ColorSpace.Named
https://developer.android.google.cn/reference/android/graphics/Color

