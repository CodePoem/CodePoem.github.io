---
title: Android图片压缩
date: 2019-10-14 16:59:44
updated: 2019-10-14 16:59:46
categories:
- Android
tags:
- Android
- 图片
- 压缩
---

# Android图片压缩

## 为什么要压缩

### 减少内存占用

内存占用大小 = Bitmap大小 = 总像素点数 x 一个像素点占用的字节数

#### Bitmap

**Android中图片在内存中的表达形式是Bitmap，即位图。**

##### 总像素点数

**总像素点数由什么决定呢？**

总像素点数 = 图片源长度 x 缩放比例 x 图片源宽度 x 缩放比例

缩放比例可以大致表示为：（为什么说大致呢？因为不是绝对的，实际还有其他因素可以影响实际缩放比例，如inScreenDensity）
缩放比例 = 1/inSampleSize x (inTargetDensity/inDensity)

* int inSampleSize

如果设置的值大于1，将会请求解码器对原始图像进行二次采样，返回较小的图像以节省内存。样本大小是任一维度中与已解码 bitmap 中单个像素对应的像素数。例如，设置 inSampleSize == 4 ，将会返回一张是宽高为原始宽度和高度的 1/4 的图像，像素为原来的 1/16。任何小于等于1的值都与1相同。注意：解码器使用基于2的幂的最终值，任何其他值将四舍五入为最接近的2的幂。

* int inDensity

用于 bitmap 像素密度。这将使得在返回的位图中始终有为其设置的密度（请参阅 Bitmap.setDensity(int)）。此外，如果开启了 inScaled（默认情况下开启），并且此密度与 inTargetDensity 不匹配，则 bitmap 在返回之前将被缩放为目标密度。

如果设置为0， BitmapFactory.decodeResource(Resources, int) 、 BitmapFactory.decodeResource(Resources, int, android.graphics.BitmapFactory.Options) 、 BitmapFactory.decodeResourceStream 这些 decode 方法将填充与资源关联的密度。其他 decode 方法将保持原样，并且不会应用任何密度。

* int inTargetDensity

此 bitmap 将被绘制到的目标像素密度。 与 inDensity 和 inScaled 结合使用，以确定在返回 bitmap 之前是否以及如何缩放位图。

如果设置为0， BitmapFactory.decodeResource(Resources, int) 、 BitmapFactory.decodeResource(Resources, int, android.graphics.BitmapFactory.Options) 、 BitmapFactory.decodeResourceStream 这些 decode 方法将填充与资源对象的 DisplayMetrics 相关的密度。其他 decode 方法将保持原样，并且不会对密度进行缩放。

* int inScreenDensity

正在使用的实际屏幕的像素密度。 仅用于以密度兼容代码运行的应用程序，其中 inTargetDensity 实际上是应用程序看到的密度而不是实际的屏幕密度。

通过设置此选项，允许加载代码避免将当前的屏幕密度的 bitmap 缩放到/降低到兼容密度。相反的，如果 inDensity 与 inScreenDensity 相同，则 bitmap 将保持不变。任何使用生成的 bitmap 的对象，还必须使用 Bitmap.getScaledWidth 和 Bitmap.getScaledHeight 来说明 bitmap 的密度与目标密度之间的任何差异。

BitmapFactory 自身永远不会为调用者自动设置。必须明确设置它，因为调用者必须以密度感知的方式处理结果位图。

* boolean inScaled

如果设置了该配置，并且 inDensity 和 inTargetDensity 不为 0，bitmap 在加载时将缩放以匹配 inTargetDensity，而不是每次绘制到 Canvas 时都依赖于图形系统对其进行缩放。

BitmapRegionDecoder 会忽略这个配置，并且不会根据密度缩放输出。（尽管支持 inSampleSize ）

此配置默认开启，如果需要 bitmap 的非缩放版本，则应将其关闭。.9 bitmaps（Nine-patch bitmaps）会忽略此配置，并且始终会缩放。

如果 inPremultiplied 设置为 false ，并且图像具有 Alpha透明度，将此配置设置为 true 可能会导致颜色错误。

##### 一个像素点占用的字节数

**一个像素点占用的字节数由什么决定呢？**

>像素信息:
>
>* 像素不是一个具体的物理量，是一种抽象的数据结构。
>
>如果把一张图片看成是一堆信息元素的集合，那么为了描述一张图片，我们要先建模，用一个数据结构来表示信息元素。从而，建模后图像就成了一堆数据结构（结构体）的集合，现在给这种数据结构起个名字就叫像素。
>
>* 像素这种数据结构中可以记录颜色信息（因为图像就是由不同颜色组成的）。


>色彩空间（ColorSpace），对色彩的组织方式，RGB、YUV、CMYK等。
>
>* RGB：一个像素划分Red、Green、Blue分量来表示，面向硬件，适合显示系统,不适合图像处理。
>* YUV：Y表示明亮度（Luminance，Luma，黑白信号），UV表示色度、浓度（Chrominance，Chroma，色彩信息）。
>* CMYK：C：Cyan ＝ 青色，常被误称为“天蓝色”或“湛蓝”，M：Magenta ＝ 洋红色，又称为“品红色”，Y：Yellow ＝ 黄色，K：blacK ＝ 黑色，是彩色印刷时采用的一种套色模式，利用色料的三原色混色原理，加上黑色油墨，共计四种颜色混合叠加，形成所谓“全彩印刷”。
>
>色彩模式是数字世界中表示颜色的一种算法。

**因此一个像素点占用的字节由存储像素信息的方案决定。**

假设我们采用ARGB色彩模式，即在RGB色彩空间的基础上加上Alpha（透明度）通道。每个通道的取值范围在[0,255]，即有256个值，刚好可以用一个字节（8bit）表示。ARGB四个通道，需要四个字节表示一个像素信息。

事实上，在保证必要的像素信息的同时追求更少的内存占用，可以减少通道的位数，甚至可以考虑去掉Alpha（透明度）通道。

**Bitmap.Config是一个枚举类，它表示的就是每个像素点信息的存储方案。**
| Config    | 每个像素占用字节数                           | 表示颜色种数          | 说明                                                                   |
| --------- | -------------------------------------------- | --------------------- | ---------------------------------------------------------------------- |
| ALPHA_8   | 1个字节，A分量占8位，不存储颜色信息          | 0                     | 单透明通道                                                             |
| RGB_565   | 两个字节，R分量占5位，G分量占6位，B分量占5位 | 2^16(65536)           | 简易RGB色调                                                            |
| RGB_888   | 三个字节，R、G、B分量各占8位                 | 2^24(16777216)        | RGB色调                                                                |
| ARGB_4444 | 两个字节，A、R、G、B分量各占4位              | 2^12(4096)            | 已弃用，成像效果比较差，并且v4.4+后如果使用了它会自动转成用ARGB_8888。 |
| ARGB_8888 | 四个字节，A、R、G、B分量各占8位              | 2^24(16777216)        | 24位真彩色，Android中默认的配置                                                             |
| RGBA_F16  | 八个字节，A、R、G、B分量各占16位             | 2^48(281474976710656) | 特别适合于宽色域和HDR内容，在8.0（api 26）引入。                       |
| HARDWARE  | -                                            | -                     | 特殊配置。bitmap始终存储在图形内存中，在8.0（api 26）引入。            |

### 减少物理空间占用

物理空间占用 = 簇（操作系统所使用的逻辑概念）的整数倍 >= 文件大小

图片在物理空间的表现形式是File，具体有GIF、JPEG、BMP、PNG和WebP等格式。

> * Gif
> Gif是一种基于LZW算法的无损压缩格式，其压缩率一般在50％左右。Gif可插入多帧，从而实现动画效果。因此Gif图片分为静态GIF和动画GIF两种GIF格式。由于Gif以8位颜色压缩存储单个位图，所以它最多只能用256种颜色来表现物体，对于色彩复杂的物体它就力不从心了。因此Gif不适合用于色彩非常丰富的图片的压缩存储，比如拍摄的真彩图片等。
> * BMP
> BMP是标准图形格式，它是包括Windows在内多种操作系统图像展现的终极形式。其本质就是Bitmap对象直接持久化保存的位图文件格式，由于没有进行压缩存储，因此体积非常大，故而不适合在网络上传输。同时也是因为这种格式是对Bitmap对象的直接存储而没有进行压缩，因此我们在讨论压缩格式时往往忽略这一种。
> * PNG
> PNG格式本身的设计目的是替代GIF格式，所以它与GIF 有更多相似的地方。PNG格式也属于无损压缩，其位深为32位，也就是说它支持所有的颜色类型。同样是无损压缩，PNG的压缩率高于Gif格式，而且PNG支持的颜色数量也远高于Gif，因此：如果是对静态图片进行无损压缩，优先使用PNG取代Gif，因为PNG压缩率高、色彩好；但是PNG不支持动画效果。所以Gif仍然有用武之地。
> PNG缺点是：由于是无损压缩，因此PNG文件的体积往往比较大。如果在项目中多处使用PNG图片文件，那么在APP瘦身时需要对PNG文件进行优化以减少APP体积大小。具体做法后面会详细介绍。
> * JPEG
> JPEG是一种有损压缩格式，JPEG图片以24位颜色压缩存储单个位图。也就是说，JPEG不支持透明通道。JPEG也不支持多帧动画。因为是有损压缩，所以需要注意控制压缩率以免图片质量太差。
> JPG和JPEG没有区别，全名、正式扩展名是JPEG。但因DOS、Windows95等早期系统采用的8.3命名规则只支持最长3字符的扩展名，为了兼容采用了.jpg。也因历史习惯和兼容性的考虑，.jpg目前更流行。JPEG2000作为JPEG的升级版，其压缩率比JPEG高约30％左右，同时支持有损和无损压缩。
> JPEG2000格式有一个极其重要的特征在于它能实现渐进传输，即先传输图像的轮廓，然后逐步传输数据，不断提高图像质量，让图像由朦胧到清晰显示。此外，JPEG2000还支持所谓的“感兴趣区域”特性，也就是可以任意指定影像上感兴趣区域的压缩质量；另外，JPEG2000还可以选择指定的部分先解压缩来加载到内存中。JPEG2000和JPEG相比优势明显，且向下兼容，因此可取代传统的JPEG格式。
> * WebP
> WebP 是 Google 在 2010 年发布的图片格式，希望以更高的压缩率替代 JPEG。它用 VP8 视频帧内编码作为其算法基础，取得了不错的压缩效果。WebP支持有损和无损压缩、支持完整的透明通道、也支持多帧动画，并且没有版权问题，是一种非常理想的图片格式。WebP支持动图，基本取代gif。
> WebP不仅集成了PNG、JPEG和Gif的所有功能，而且相同质量的无损压缩WebP图片体积比PNG小大约26%；如果是有损压缩，相同质量的WebP图片体积比JPEG小25%-34%。
> 很多人会认为，既然WebP功能完善、压缩率更高，那直接用WebP取代上述所有的图片压缩格式不就行了吗？其实不然，WebP也有其缺点：我们知道JPEG是有损压缩而PNG是无损压缩，所以JPEG的压缩率高于PNG；但是有损压缩的算法决定了其压缩时间一定是高于无损压缩的，也就是说JPEG的压缩时间高于PNG。而WebP无论是无损还是有损压缩，压缩率都分别高于PNG和JPEG；与其相对应的是其压缩时间也比它们长的多。经测试，WebP图片的编码时间比JPEG长8倍。可以看出，时间和空间是一对矛盾；如果想要节省更多的空间，必然要付出额外的时间；如果想要节省时间，那么必然要付出空间的代价。这取决于我们在实际中对于时空不同的需求程度来做出选择。
> 不管怎么说，WebP还是一种强大的、理想的图片压缩格式，并且借由 Google 在网络世界的影响力，WebP 在几年的时间内已经得到了广泛的应用。看看你手机里的 App：微博、微信、QQ、淘宝等等，每个 App 里都有 WebP 的身影。
> 另外，WebP是Android4.0才引入的一种图片压缩格式，如果想要在Android4.0以前的版本支持WebP格式的图片，那么需要借助于第三方库来支持WebP格式图片，例如：[webp-android-backport](https://github.com/alexey-pelykh/webp-android-backport )函数库，当然考虑到一般的Android开发中只需要向下兼容到Android4.0即可，所以也可以忽略这个问题。
>
> 目前来说，以上所述的五种格式，Android操作系统都提供了原生支持；但是在上层能直接调用的编码方式只有 JPEG、PNG、WebP 这三种。具体的，可以查看Bitmap类的枚举内部类CompressFormat类的枚举值来获取上层能调用的图片编码方式。你会发现枚举值也是JPEG、PNG和WEBP三种。
> 如果我们想要在应用层使用Gif格式图片，需要自行引入第三方函数库来提供对Gif格式图片的支持。不过一般我们用WebP取代Gif。

## 如何压缩

### 压缩分类

#### 有损压缩

>有损压缩的基本依据是：人的眼睛对光线的敏感度远高于对颜色的敏感度，光线对景物的作用比颜色的作用更为重要。有损压缩的原理是：保持颜色的逐渐变化，删除图像中颜色的突然变化。生物学中的大量实验证明，人类大脑会自发地利用与附近最接近的颜色来填补所丢失的颜色。有损压缩的具体实现方法就是删除图像中景物边缘的某些颜色部分。当在屏幕上看这幅图时，大脑会利用在景物上看到的颜色填补所丢失的颜色部分。利用有损压缩技术，某些数据被有意地删除了，并且在图片重新加载至内存中时这些数据也不会还原，因此被称为是“有损”的。有损压缩技术可以灵活地设置压缩率。
>无可否认，利用有损压缩技术可以在位图持久化存储的过程中大大地压缩图片的存储大小，但是会影响图像质量，这一点在压缩率很高时尤其明显。所以需要选择恰当的压缩率。

#### 无损压缩

>无损压缩的基本原理是：相同的颜色信息只需保存一次。具体过程是：首先会确定图像中哪些区域是相同的，哪些是不同的。包括了重复数据的区域就可以被压缩，只需要记录该区域的起始点即可。
>从本质上看，无损压缩的方法通过删除一些重复数据，也能在位图持久化存储的过程中减少要在磁盘上保存的图片大小。但是，如果将该图片重新读取到内存中，重复数据会被还原。因此，无损压缩的方法并不能减少图片的内存占用量，如果要减少图片占用内存的容量，就必须使用有损压缩方法。
>无损压缩方法的优点是能够比较好地保存图像的质量，但是相对来说这种方法的压缩率比较低。
>对比分析：有损压缩压缩率高而且可以灵活设置压缩率，并且删除的数据不可还原，因此可以减少图片的内存占用，但是对图片质量会有一定程度的影响；无损压缩可以很好地保存图片质量，也能保证一定的压缩率虽然没有有损压缩那么高，并且无损压缩删除的数据在重新加载至内存时会被还原，因此不可以减少图片的内存占用。

### Android中压缩

#### 编译时压缩

AAPT打包时默认对PNG进行三个优化检查（[Android源码](http://androidxref.com/)中搜索analyze_image），默认使用的是libpng库进行无损压缩（修改色彩模式）：

1. 每个像素都是 R == G == B (grayscale 灰度)
2. 每个像素都是 A == 255 (opaque 全透明)
3. 是否不超过256种不同的RGBA颜色

判断它是否可以被转成灰度格式的图片，判断它是否是全透明的，或判断它是否可以被转成一张索引图。

如果想压缩地更多，禁用掉AAPT的默认压缩PNG（一般二次压缩反而会增大体积），采用第三方压缩工具来提升压缩效果。

```禁用AAPT默认PNG处理
android {
        buildTypes {
            release {
                // Disables PNG crunching for the release build type.
                crunchPngs false
            }
        }

    // If you're using an older version (Android Gradle plugin < 3.0.0) of the plugin, use the
    // following:
    //  aaptOptions {
    //      cruncherEnabled false
    //  }
    }
```

第三方工具：

* [ImageOptim](https://imageoptim.com) 无损压缩
* [ImageAlpha](https://pngmini.com) 有损压缩
* [TinyPNG](https://tinypng.com) 有损压缩
* [Pngquant](https://pngquant.org/) 有损压缩
* [Pngout](http://advsys.net/ken/utils.htm) 无损压缩
* [OptiPNG](http://optipng.sourceforge.net/) 无损压缩

#### 运行时压缩

##### 质量压缩

质量压缩在不改变像素的大小的前提下，降低图像的质量（改变图片的位深及透明度等），从而降低存储大小，进而达到压缩的目的。

**质量压缩对内存大小占用无影响。**

> 位深度指的是存储每个像素所用的位数，主要用于存储。
> 色深指的是每一个像素点用多少bit存储颜色，属于图片自身的一种属性。
> 位深一般小于或等于色深。
> 举个例子：某张图片100像素*100像素 色深32位(ARGB_8888)，保存时位深度为24位，那么：
> 该图片在内存中所占大小为：100 x 100 x (32 / 8) Byte
> 在文件中所占大小为 100 x 100 x ( 24/ 8 ) x 压缩率 Byte

Android中的质量压缩API ：

```Android中的质量压缩API
public boolean compress(CompressFormat format, int quality, OutputStream stream)
```

Java 层函数 → Native 函数 → Skia函数 → 对应第三库函数（例如 libjpeg、libpng、libjwebp）

* CompressFormat format：压缩格式,它有JPEG、PNG、WEBP三种选择，JPEG是有损压缩，PNG是无损压缩，WEBP是Google推出的图像格式.
* int quality：0~100可选，数值越大，质量越高，图像越大。
* OutputStream stream：压缩后图像的输出流。

针对JPEG的压缩库：

* [libjpeg-turbo](https://github.com/libjpeg-turbo/libjpeg-turbo)

> libjpeg-turbo是用于x86和x86-64处理器的libjpeg的高速版本，它使用SIMD指令（MMX，SSE2等）来加速基线JPEG压缩和解压缩。 libjpeg-turbo的速度通常是未修改版本的libjpeg的2-4倍，其他所有条件都相同（对于非灰度JPEG压缩和解压缩，libjpeg-turbo的速度是libjpeg v6b的1.8倍至4.5倍）。

* [mozilla/mozjpeg](https://github.com/mozilla/mozjpeg)

> 基于libjpeg-turbo，依靠三种技术（渐进JPEG编码，jpgcrush和网格量化）来减小JPEG图像的大小。libjpeg-turbo支持渐进式JPEG，但不支持jpgcrush和网格量化。mozjpeg的唯一目的是减少网络上提供的JPEG文件的大小，因此它牺牲了一些性能为代价。

##### 尺寸压缩

尺寸压缩即减少图片长宽，或者说减少像素点，又称采样压缩（上采样为放大，下采样缩小，因此采样压缩为下采样）。

###### 邻近采样

邻近采样（Nearest Neighbour Resampling）采用邻近点插值算法，用一个像素点代替邻近的像素点。

[官方高效加载Bitmap指南](https://developer.android.com/topic/performance/graphics/load-bitmap?hl=zh-cn)采用的方式就是进行邻近采样。

做法是先将 BitmapFactory.Options 中的 inJustDecodeBounds 设置为true，这样 BitmapFactory 在 decode 的时候能避免内存分配，但能对 outWidth，outHeight 和 outMimeType 赋值。然后通过获取到的outWidth，outHeight 以及 加载的目标宽高计算出合适的采样率赋值给 BitmapFactory.Options 中的 inSamleSize。最后 BitmapFactory.Options 中的 inJustDecodeBounds 设置回false，并进行相应的decode。

关键之处就是计算出合适的采样率，对应 BitmapFactory.Options 中的 inSamleSize（inSamleSize会是四舍五入后最接近2的幂的值）。
关于如何计算出合适的采样率，普遍的算法是：

```java
public static int calculateInSampleSize(
            BitmapFactory.Options options, int reqWidth, int reqHeight) {
    // Raw height and width of image
    final int height = options.outHeight;
    final int width = options.outWidth;
    int inSampleSize = 1;

    if (height > reqHeight || width > reqWidth) {

        final int halfHeight = height / 2;
        final int halfWidth = width / 2;

        // Calculate the largest inSampleSize value that is a power of 2 and keeps both
        // height and width larger than the requested height and width.
        while ((halfHeight / inSampleSize) >= reqHeight
                && (halfWidth / inSampleSize) >= reqWidth) {
            inSampleSize *= 2;
        }
    }

    return inSampleSize;
}
```

> 第三个开源库[Luban](https://github.com/Curzibn/Luban)的[计算采样率算法思路](https://github.com/Curzibn/Luban/blob/master/DESCRIPTION.md)

###### 双线性采样

双线性采样（Bilinear Resampling）采用双线性插值算法，相比邻近采样简单粗暴的选择一个像素点代替其他像素点，双线性采样参考源像素相应位置周围2x2个点的值，根据相对位置取对应的权重，经过计算得到目标图像。

Android中双线性采样有两个API：

* public static Bitmap createScaledBitmap(@NonNull Bitmap src, int dstWidth, int dstHeight, boolean filter)
* public static Bitmap createBitmap(@NonNull Bitmap source, int x, int y, int width, int height, @Nullable Matrix m, boolean filter)
  
事实上，createScaledBitmap(@NonNull Bitmap src, int dstWidth, int dstHeight, boolean filter) 方法最终也是调用 createBitmap(@NonNull Bitmap source, int x, int y, int width, int height, @Nullable Matrix m, boolean filter) 方法。


* Bitmap source：源图像
* int x：目标图像第一个像素的x坐标
* int y：目标图像第一个像素的y坐标
* int width：目标图像的宽度（像素点个数）
* int height：目标图像的高度（像素点个数）
* Matrix m：变换矩阵
* boolean filter：是否开启双线性滤波

###### 双立方/双三次采样

[双立方/双三次采样（Bicubic Resampling）](https://en.wikipedia.org/wiki/Bicubic_interpolation)，邻近点插值算法的目标像素值由源图上单个像素决定，双线性內插值算法由源像素某点周围 2x2 个像素点按一定权重获得，而双立方／双三次插值算法更进一步参考了源像素某点周围 4x4 个像素。

Android中对于双立方/双三次采样没有支持，可以通过手动编写算法或者引用第三方算法库，幸运的是这个算法在 ffmpeg 中已经给到了支持，具体的实现在 libswscale/swscale.c 文件中：[FFmpeg Scaler Documentation](http://www.ffmpeg.org/ffmpeg-scaler.html)。

###### Lanczos 采样

[Lanczos 采样](https://en.wikipedia.org/wiki/Lanczos_resampling)和 Lanczos 过滤是 Lanczos 算法的两种常见应用，它可以用作低通滤波器或者用于平滑地在采样之间插入数字信号，Lanczos 采样一般用来增加数字信号的采样率，或者间隔采样来降低采样率。
Lanczos 采样使用的 Lanczos 算法也可以用来作为图片的缩放，Lanczos 算法和双三次插值算法都是使用卷积核来通过输入像素计算输出像素。
同样的，Lanczos 算法在 ffmpeg 的 libswscale/swscale.c 中也有实现。其实不光 Lanczos 和上面的三种算法，ffmpeg 还提供了其他的图像重采样方法，诸如 area averaging、Gaussian 等等。

##### 色彩模式压缩

BitmapFactory.Options中inPreferredConfig可以指定色彩模式，采用色位更少的色彩模式。

___

参考链接：

* [Android Bitmap（位图）详解](https://yunxi.vkucloud.com/article/e53f10f5-0f45-41a3-45dd-08d6f1abedd9)
* [Android中Bitmap内存优化](https://juejin.im/entry/5ad0213f6fb9a028df2306cd)
* [也谈图片压缩](https://juejin.im/entry/58fc75e0ac502e0063aa433b)
* [Android平台图像压缩方案](https://juejin.im/post/5a1bd6595188254cc067981f)
* QQ音乐技术文章[Android 中图片压缩分析（上）](https://cloud.tencent.com/developer/article/1006307)
* QQ音乐技术文章[Android 中图片压缩分析（下）](https://cloud.tencent.com/developer/article/1006352)
* [Android 性能优化（五）之细说 Bitmap](https://juejin.im/post/58c3b29761ff4b005d906730#heading-14)
* [Loading Large Bitmaps Efficiently](https://developer.android.com/topic/performance/graphics/load-bitmap.html)
* [Caching Bitmaps](https://developer.android.google.cn/topic/performance/graphics/cache-bitmap.html)
* [Managing Bitmap Memory](https://developer.android.com/topic/performance/graphics/manage-memory)
* [Handling bitmaps](https://developer.android.com/topic/performance/graphics)

后记：可研读核心类Bitmap、BitmapFactory。
