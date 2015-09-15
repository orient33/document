##Palette 调色板
作用：从bitmap中提取突出的颜色(主题色)  
用法：同步或异步，耗时依据bitmap大小和取色位相关  
  
    // Synchronous  
    Palette p = Palette.from(bitmap).generate();  
  
    // Asynchronous  
    Palette.from(bitmap).generate(new PaletteAsyncListener() {  
       public void onGenerated(Palette p) {  
           // Use generated instance  
       }  
    });   

Palette默认生成6种颜色：
>  
Vibrant  （有活力/鲜艳的）  
Vibrant dark（有活力/鲜艳的 暗色）  
Vibrant light（有活力/鲜艳的 亮色）  
Muted  （柔和）  
Muted dark（柔和 暗色）  
Muted light（柔和 亮色）  

关于颜色的HSL：  
RGB是以红绿蓝三种基本色混合。  
HSL是以 色相，饱和度，明亮度，取值范围0-360度、0%-100%、0%-100% 。    
RGB和HSL可以以一定的公式转换。   

##Palette源码解析 -- 取色步骤 
默认计算 16 色  
1 缩小图片到默认尺寸(100px，新版为192px)。宽或高较小者为100  

2 实例化ColorHistogram，即颜色表  
  将图片的每个px的RGB颜色放到数组pixels[]，排序，计算颜色数目以及每种颜色的出现频率，保存 mColors[]和mColorCounts[], mNumberColors。    

3 实例化ColorCutQuantizer, 即颜色剪切转换  
  遍历mColors[]中的每个颜色，过滤掉一些颜色(黑，白，红)  
  若此时mColors[]的颜色length不大于  16， 则将每种颜色和对应频率new一个Swatch加到mQuantizedColors.  
  否则 用 quantizePixels(length-1, 16 )转换/剪切颜色组
  
    构造优先队列，PriorityQueue pq 大小为 16  
    pq.offer(new Vbox(0, maxColorIndex))先添加0-最大索引的Vbox  
    splitBoxs(pq, 16){
        while(pq.size < 16) {
            final Vbox vbox = pq.poll();
            if(vbox != null && vbox.canSplit()) {
                pq.offer(vbox.splitBox());
                pq.offer(vbox);
            } else {
                return;
            }
        }
    }    


4 实例化palette，new Palette(List<Swatch> s)  
  在取样Swatch列表中找出最大频率，并找出最适合做默认主题色的6中Swatch，当然，可能找不到一些主题色。这里会根据颜色的 HSL 中的S，L 来匹配

     private Swatch findColor(float targetLuma, float minLuma, float maxLuma,
         float targetSaturation, float minSaturation, float maxSaturation)

  在 [min, max]的范围下，找出最接近target的Swatch；若此范围下无，则null。

5 最后Palette对象包括6种样品Swatch，包括颜色和频率，及titleText、bodyText的颜色。