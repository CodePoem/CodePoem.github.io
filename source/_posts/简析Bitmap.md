---
title: 简析Bitmap
date: 2019-10-16 11:25:23
updated: 2019-10-16 11:25:27
categories:
- 图片
tags:
- Android
- 图片
- Bitmap
---

# 简析Bitmap

基于9.0（29）简析Bitmap类。

我们只关注Bitmap类暴露出来的（即访问权限为public）东西。

* 枚举类Config
* 枚举类CompressFormat
* 若干 createBitmap 静态方法
* 其他方法

## 枚举类Config

可能的 bitmap 配置。bitmap 配置描述像素的存储方式。这会影响质量（颜色深度）以及显示透明/半透明颜色的能力。

| Config    | 每个像素占用字节数                           | 表示颜色种数          | 说明                                                                   |
| --------- | -------------------------------------------- | --------------------- | ---------------------------------------------------------------------- |
| ALPHA_8   | 1个字节，A分量占8位，不存储颜色信息          | 0                     | 单透明通道                                                             |
| RGB_565   | 两个字节，R分量占5位，G分量占6位，B分量占5位 | 2^16(65536)           | 简易RGB色调                                                            |
| RGB_888   | 三个字节，R、G、B分量各占8位                 | 2^24(16777216)        | RGB色调                                                                |
| ARGB_4444 | 两个字节，A、R、G、B分量各占4位              | 2^12(4096)            | 已弃用，成像效果比较差，并且v4.4+后如果使用了它会自动转成用ARGB_8888。 |
| ARGB_8888 | 四个字节，A、R、G、B分量各占8位              | 2^24(16777216)        | 24位真彩色，Android中默认的配置                                        |
| RGBA_F16  | 八个字节，A、R、G、B分量各占16位             | 2^48(281474976710656) | 特别适合于宽色域和HDR内容，在8.0（api 26）引入。                       |
| HARDWARE  | -                                            | -                     | 特殊配置。bitmap始终存储在图形内存中，在8.0（api 26）引入。            |

## 枚举类CompressFormat

指定可以将 bitmap 压缩为的已知格式。

* JPEG
* PNG
* WEBP

## 若干 createBitmap 静态方法

### public static Bitmap wrapHardwareBuffer(@NonNull HardwareBuffer hardwareBuffer, @Nullable ColorSpace colorSpace)

创建由 HardwareBuffer 支持的硬件位图。

传递的 HardwareBuffer 使用标志必须包含 HardwareBuffer.USAGE_GPU_SAMPLED_IMAGE。

bitmap 将保留对缓冲区的引用，以便调用者可以安全地关闭 HardwareBuffer 而不会影响 bitmap 。但是，在包装 bitmap 访问硬件缓冲区时，不能对其进行修改。这样做将导致不确定的行为。

* HardwareBuffer hardwareBuffer：包装的 HardwareBuffer。
* ColorSpace colorSpace：bitmap 的颜色空间。必须是 ColorSpace.Rgb 颜色空间。如果为 null ，则假定为SRGB。

### public static Bitmap createScaledBitmap(@NonNull Bitmap src, int dstWidth, int dstHeight, boolean filter)

如果可能，创建一个新的 bitmap ，从现有的 bitmap 缩放。如果指定的宽度和高度与源位图的当前宽度和高度相同，则返回源位图，并且不创建新的位图。

* Bitmap src：源 bitmap。
* int dstWidth：新 bitmap 的所需宽度
* int dstHeight：新 bitmap 的所需高度
* boolean filter：缩放 bitmap 时是否使用双线性滤波。如果为 true ，则在缩放时将使用双线性滤波，从而以较差的性能为代价来获得更好的图像质量。如果为 false ，则使用邻近采样缩放，这将使图像质量较差，但速度更快。推荐的默认值是将设置为 true，因为双线性滤镜的成本通常很小，并且改善的图像质量非常重要。

### public static Bitmap createBitmap(@NonNull Bitmap src)

重载 Bitmap createBitmap(@NonNull Bitmap source, int x, int y, int width, int height)

createBitmap(src, 0, 0, src.getWidth(), src.getHeight());

### public static Bitmap createBitmap(@NonNull Bitmap source, int x, int y, int width, int height)

重载 Bitmap createBitmap(@NonNull Bitmap source, int x, int y, int width, int height, @Nullable Matrix m, boolean filter)

createBitmap(source, x, y, width, height, null, false);

### public static Bitmap createBitmap(@NonNull Bitmap source, int x, int y, int width, int height, @Nullable Matrix m, boolean filter)

从源 bitmap 返回一个 bitmap 。新的bitmap 可能与源 bitmap 是同一对象，或者可能已复制。使用与源 bitmap 相同的密度和色彩空间进行初始化。

如果源 bitmap 是不可变的，并且所请求的子集与源 bitmsp 本身相同，则返回源 bitmap ，并且不会创建新 bitmap 。

除非在以下情况下，否则返回的 bitmap 将始终是可变的：
（1）在返回源 bitmap 且源 bitmap 不可变的情况下
（2）源 bitmap 是硬件 bitmap 。即 getConfig() = Config.HARDWARE 。

* Bitmap source：源 bitmap 。
* int x：源中第一个像素的x坐标
* int y：源中第一个像素的y坐标
* int width：每行像素数
* int height：行数
* Matrix m：应用于像素的可选矩阵
* boolean filter：是否使用双线性滤波，则为true。仅在矩阵包含的不仅仅是平移时适用。

### public static Bitmap createBitmap(int width, int height, @NonNull Config config)

重载 Bitmap createBitmap(int width, int height, @NonNull Config config, boolean hasAlpha)

createBitmap(null, width, height, config, hasAlpha)

### public static Bitmap createBitmap(@Nullable DisplayMetrics display, int width, int height, @NonNull Config config)

重载 Bitmap createBitmap(@Nullable DisplayMetrics display, int width, int height, @NonNull Config config, boolean hasAlpha)

createBitmap(display, width, height, config, true);

### public static Bitmap createBitmap(int width, int height, @NonNull Config config, boolean hasAlpha)

重载 Bitmap createBitmap(@Nullable DisplayMetrics display, int width, int height, @NonNull Config config, boolean hasAlpha)

createBitmap(null, width, height, config, hasAlpha);

### public static Bitmap createBitmap(int width, int height, @NonNull Config config, boolean hasAlpha, @NonNull ColorSpace colorSpace)

重载 Bitmap createBitmap(@Nullable DisplayMetrics display, int width, int height, @NonNull Config config, boolean hasAlpha, @NonNull ColorSpace colorSpace)

createBitmap(null, width, height, config, hasAlpha, colorSpace);

### public static Bitmap createBitmap(@Nullable DisplayMetrics display, int width, int height, @NonNull Config config, boolean hasAlpha)

重载 Bitmap createBitmap(@Nullable DisplayMetrics display, int width, int height, @NonNull Config config, boolean hasAlpha, @NonNull ColorSpace colorSpace)

createBitmap(display, width, height, config, hasAlpha, ColorSpace.get(ColorSpace.Named.SRGB));

### public static Bitmap createBitmap(@Nullable DisplayMetrics display, int width, int height, @NonNull Config config, boolean hasAlpha, @NonNull ColorSpace colorSpace)

返回具有指定宽度和高度的可变 bitmap。它的初始密度由给定的 DisplayMetrics 确定。新创建的 bitmap 位于 ColorSpace.Named.SRGB 颜色空间中。

* DisplayMetrics display：将被绘制的 bitmap 的显示的显示度量。
* int width：bitmap 的宽度。
* int height：bitmap 的高度。
* Config config：创建的 bitmap 的配置。
* boolean hasAlpha：如果 bitmap 是 ARGB_8888 或 RGBA_16F ，则此标志可用于将 bitmap 标记为不透明。这样做将以黑色而不是透明的方式清除 bitmap。
* ColorSpace colorSpace：bitmap的颜色空间。如果配置为 Config.RGBA_F16 和 ColorSpace.Named.SRGB sRGB 或ColorSpace.Named.LINEAR_SRGB ，则假定为相应的扩展范围变体。

### public static Bitmap createBitmap(@NonNull @ColorInt int[] colors, int offset, int stride, int width, int height, @NonNull Config config)

重载 Bitmap createBitmap(@NonNull DisplayMetrics display, @NonNull @ColorInt int[] colors, int offset, int stride, int width, int height, @NonNull Config config)

createBitmap(null, colors, offset, stride, width, height, config)

### public static Bitmap createBitmap(@NonNull DisplayMetrics display, @NonNull @ColorInt int[] colors, int offset, int stride, int width, int height, @NonNull Config config)

返回具有指定宽度和高度的不可变 bitmap，每个像素值设置为 colors 数组中的相应值。它的初始密度由给定的 DisplayMetrics 确定。新创建的 bitmap 位于 ColorSpace.Named.SRGB 颜色空间中。

* DisplayMetrics display：将被绘制的 bitmap 的显示的显示度量。
* @ColorInt int[] colors：sRGB Color数组，用于初始化像素。
* int offset：颜色数组中第一个颜色之前要跳过的值数。
* int stride：行之间数组中的颜色数（必须为 >= width 或 <= -width）。
* int width：bitmap 的宽度。
* int height：bitmap 的高度。
* Config config：创建的 bitmap 的配置。如果配置不支持每像素的alpha（例如RGB_565），那么colors []中的alpha 字节将被忽略（假定为FF）

### public static Bitmap createBitmap(@NonNull @ColorInt int[] colors, int width, int height, Config config)

重载 Bitmap createBitmap(@NonNull DisplayMetrics display, @NonNull @ColorInt int[] colors, int offset, int stride, int width, int height, @NonNull Config config)

createBitmap(null, colors, 0, width, width, height, config);

### public static Bitmap createBitmap(@Nullable DisplayMetrics display, @NonNull @ColorInt int colors[], int width, int height, @NonNull Config config)

重载 Bitmap createBitmap(@NonNull DisplayMetrics display, @NonNull @ColorInt int[] colors, int offset, int stride, int width, int height, @NonNull Config config)

createBitmap(display, colors, 0, width, width, height, config);

### public static @NonNull Bitmap createBitmap(@NonNull Picture source)

重载 Bitmap createBitmap(@NonNull Picture source, int width, int height, @NonNull Config config)

createBitmap(source, source.getWidth(), source.getHeight(), Config.HARDWARE);

### public static @NonNull Bitmap createBitmap(@NonNull Picture source, int width, int height, @NonNull Config config)

从记录的绘图命令中给定 Picture 源创建 bitmap。

在给定的宽度和高度下，bitmap 将不可变。如果宽度和高度与图片的宽度和高度不同，则图片将缩放以适应给定的宽度和高度。

* Picture source：记录的绘图命令 Picture 将会被绘制到返回的 bitmap 中
* int width：要创建的 bitmap 的宽度。图片的宽度将根据需要缩放以匹配。
* int height：要创建的 bitmap 的高度。图片的高度将根据需要缩放以匹配。
* Config config：创建的 bitmap 的配置。

## 其他方法

### int getDensity()

返回此 bitmap 的密度。

默认密度与当前显示的密度相同，除非当前应用程序不支持不同的屏幕密度，这种情况下默认密度为 android.util.DisplayMetric.DENSITY_DEFAULT = DENSITY_MEDIUM = 160。请注意，兼容性模式由最初加载到进程中的应用程序确定-共享同一进程的应用程序应该全部具有相同的兼容性，或者确保它们适当地显式设置其 bitmap 的密度。

### void setDensity(int density)

指定此 bitmap 的密度。当 bitmap 绘制到也具有密度的Canvas时，它将适当缩放。

* int densit：该 bitmap 使用的密度缩放因子；如果密度未知，则使用DENSITY_NONE = 0。

### void reconfigure(int width, int height, Config config)

将 bitmap 修改为具有指定的宽度，高度和 Config ，而不影响该 bitmap 的底层分配。 对于新配置，bitmap 像素数据未重新初始化。

此方法可用于避免分配新的 bitmap，而是将现有 bitmap 的分配重用于大小等于或小于的新配置。如果 bitmap 的分配不足以支持新配置，则将抛出IllegalArgumentException，并且 bitmap 将不会被修改。

getByteCount（）的结果将反映新的配置，而getAllocationByteCount（）的结果将反映初始的配置。

注意：这可能会更改hasAlpha（）的结果。当转换为565时，新的 bitmap 将始终被视为不透明的。从565转换时，新的 bitmap 将被认为是不透明的，并且将遵循由setPremultiplied（）设置的值。

警告：不应在当前正在 view 系统、Canvas 或 AndroidBitmap NDK API 使用的 bitmap 上调用此方法。它不能保证基础像素缓冲区如何重新映射到新配置，而只是保证分配已被重用。此外，view 系统无法确定使用过程中正在修改的 bitmap 属性，例如何时附着在 drawables 上。

为了安全地确保 view 系统不再使用 bitmap，必须等待正在invalidate（）的任何在最后的绘制过程中由于硬件加速对绘制命令的缓存已经在之前绘制过此 bitmap 的 view 完成绘制过程。举一个例子，以下是以 ImageView 完成的：

```ImageView bitmap reconfigure
ImageView myImageView = ...;
final Bitmap myBitmap = ...;
myImageView.setImageDrawable(null);
myImageView.post(new Runnable() {
    public void run() {
        // myBitmap is now no longer in use by the ImageView
        // and can be safely reconfigured.
        myBitmap.reconfigure(...);
    }
});
```

### void setWidth(int width)

使用当前已有高度和配置调用 reconfigure（int，int，Config）的便捷方法。

警告：此方法不应在 view 系统当前使用的 bitmap 上使用，有关更多详细信息，请参见 reconfigure（int，int，Config）。

### void setHeight(int height)

使用当前已有宽度和配置调用 reconfigure（int，int，Config）的便捷方法。

警告：此方法不应在 view 系统当前使用的 bitmap 上使用，有关更多详细信息，请参见 reconfigure（int，int，Config）。

### void setConfig(Config config)

使用当前已有宽度和高度调用 reconfigure（int，int，Config）的便捷方法。

警告：此方法不应在 view 系统当前使用的 bitmap 上使用，有关更多详细信息，请参见 reconfigure（int，int，Config）。

### void setNinePatchChunk(byte[] chunk)

设置.9图数据块。

* byte[] chunk .9图相关定义数据块

### void recycle()

释放与此 bitmap 关联的 native 对象，并清除对像素数据的引用。这将不会同步释放像素数据。 如果没有其他引用，它只是允许对其进行垃圾回收。bitmap 被标记为“死亡”，这意味着如果 getPixels（）或setPixels（）被调用，它将抛出异常，并且不会绘制任何内容。 此操作不能撤消，因此只有在确定 bitmap 没有进一步用途时才应调用它。这是一个高级调用，通常不需要，因为当没有更多对该 bitmap 的引用时，正常的GC进程将释放此内存。

### boolean isRecycled()

如果此 bitmap 已被回收，则返回true。如果是被回收，则尝试访问其像素是一个错误，并且 bitmap 将不会绘制。

### int getGenerationId()

返回此 bitmap 的生成ID。只要修改 bitmap，生成ID就会改变。这是一个可用于检查 bitmap 是否已更改的高效方法。

### void copyPixelsToBuffer(Buffer dst)

将 bitmap 的像素复制到指定的缓冲区（由调用者分配）。如果缓冲区的大小不足以容纳所有像素（考虑到每个像素的字节数），或者如果Buffer子类不是支持类型之一（ByteBuffer，ShortBuffer，IntBuffer），则抛出异常。

bitmap 的内容按原样复制到缓冲区中。这意味着如果该 bitmap 存储了预乘的像素（请参见 isPremultiplied()），则缓冲区中的值也将被预乘。像素保留在 bitmap 的颜色空间中）。

返回此方法后，将更新缓冲区的当前位置：位置将增加写入缓冲区的元素数量。

### void copyPixelsFromBuffer(Buffer src)

从当前位置开始，从缓冲区复制像素，覆盖 bitmap 的像素。缓冲区中的数据不会以任何方式更改（不同于setPixels（），后者会从未预乘的32bit转换为 bitmap 的 native 格式。源缓冲区中的像素假定位于 bitmap 的色彩空间中）。

返回此方法后，将更新缓冲区的当前位置：该位置将增加从缓冲区读取的元素数。如果需要再次从缓冲区读取 bitmap，则必须首先倒带缓冲区。

### Bitmap copy(Config config, boolean isMutable)

尝试根据此 bitmap 的尺寸制作一个新的 bitmap，将新 bitmap 的配置设置为指定的配置，然后将该 bitmap 的像素复制到新的位图中。如果不支持转换，或者分配器失败，则返回 NULL 。返回的 bitmap 具有与原始 bitmap 相同的密度和色彩空间，以下情况除外。复制到 Config.ALPHA_8 时，颜色空间将被删除。复制到 Config.RGBA_F16 或从 Config.RGBA_F16 复制时， EXTENDED 或 non-EXTENDED 变体可能会适当调整。

* Config config：生成的 bitmap 所需的配置。
* boolean isMutable：是否结果 bitmap 是可变的（即像素可以修改）。

### byte[] getNinePatchChunk()

返回一个可选的私有数据数组，UI系统用于一些 bitmap 。不应由应用程序调用。

### boolean compress(CompressFormat format, int quality, OutputStream stream)

将 bitmap 的压缩版本写入指定的输出流。如果返回true，则可以通过将相应的输入流传递给 BitmapFactory.decodeStream（）来重构位图。注意：不是所有格式都直接支持所有 bitmap 配置，因此从 BitmapFactory 返回的 bitmap 可能处于不同的位深中，并且/或者 丢失了每个像素的alpha（例如JPEG仅支持不透明像素）。

* CompressFormat format：压缩图像的格式
* int qualit：给压缩器的提示，0-100。 0表示压缩已获得最低质量，100表示​​压缩以获得最高质量。某些格式（例如无损的PNG）将忽略该质量设置。
* OutputStream stream：写入压缩数据的输出流。

### final boolean isMutable()

返回 bitmap 是否为可变的。

### final boolean isPremultiplied()

指示是否存储在此 bitmap 中的像素被预乘。 当像素预乘时，RGB 分量已乘以 alpha分量。例如，如果原始颜色为50％半透明红色（128，255，0，0），则预乘形式为（128，128，0，0）。

如果 getConfig() 为 Bitmap.Config.RGB_565 ，则方法始终返回false。

如果 getConfig() 为 Bitmap.Config.ALPHA_8 ，则方法返回值不确定。

仅当 hasAlpha() 返回 true 时，此方法才返回 true 。 没有alpha通道的 bitmap 既可以用作预乘位，也可以用作非预乘 bitmap 。

view 系统或 Canvas 只能绘制预乘的 bitmap。如果将带有 Alpha 通道的非预乘 bitmap 绘制到了 Canvas 上，则会抛出 RuntimeException。

### final int getWidth()

返回 bitmap 的宽度。

### final int getHeight()

返回 bitmap 的高度。

### int getScaledWidth(Canvas canvas)

使用给定 Canva 的目标密度调用 getScaledWidth(int) 的便捷方法。

### int getScaledHeight(Canvas canvas)

使用给定 Canva 的目标密度调用 getScaledWidth(int) 的便捷方法。

### int getScaledWidth(DisplayMetrics metrics)

使用给定 DisplayMetrics 的目标密度调用 getScaledWidth(int) 的便捷方法。

### int getScaledHeight(DisplayMetrics metrics)

使用给定 DisplayMetrics 的目标密度调用 getScaledWidth(int) 的便捷方法。

### int getScaledWidth(int targetDensity)

返回此 bitmap 宽度除以密度比例因子的便捷方法。

返回 bitmap 的宽度乘以目标密度与 bitmap 的源密度的比率。

* int targetDensity：bitmap 的目标 canvas 的密度。

### int getScaledHeight(int targetDensity)

返回此 bitmap 高度除以密度比例因子的便捷方法。

返回 bitmap 的高度乘以目标密度与 bitmap 的源密度的比率。

* int targetDensity：bitmap 的目标 canvas 的密度。

### final int getRowBytes()

返回 bitmap 像素中 行之间的字节数。请注意，指的是由 bitmap 由 native 存储的像素。如果调用 getPixels() 或 setPixels() ，则像素将被统一视为 32 位值，并根据Color类打包。

从 android.os.Build.VERSION_CODES.KITKAT （Android 4.4，19）开始，此方法不应用于计算 bitmap 的内存使用情况。而应使用 getAllocationByteCount() 。

### final int getByteCount()

返回可用于存储该 bitmap 像素的最小字节数。

从 android.os.Build.VERSION_CODES.KITKAT （Android 4.4，19）开始，此方法不应用于计算 bitmap 的内存使用情况。而应使用 getAllocationByteCount() 。

### final int getAllocationByteCount()

返回用于存储此 bitmap 像素的已分配内存的大小。

如果 bitmap 被重用以解码其他较小尺寸的 bitmap，或者通过手动重新配置，则此结果可能大于 getByteCount() 的结果。请参见 reconfigure(int，int，Config)，setWidth(int) ，setHeight(int) ，setConfig(Bitmap.Config) 和 BitmapFactory.Options.inBitmap 。如果未以这种方式修改 bitmap ，则此值将 getByteCount() 返回的值相同。

### final Config getConfig()

如果 bitmap 的内部配置采用一种公共格式，则返回该配置，否则返回 null 。

### final boolean hasAlpha()

如果 bitmap 的配置支持每个像素的 alpha ，则返回true；如果像素可能包含非透明的 alpha 值，则返回。对于某些配置，始终为false（例如RGB_565），因为它们不支持按像素 alpha。但是，对于需要这样做的配置，可以将 bitmap 标记为知道其所有像素都是不透明的。在这种情况下，hasAlpha（）也将返回 false 。如果未对ARGB_8888之类的配置进行标记，则默认情况下将返回true。

### final boolean hasMipMap()

指示负责绘制此 bitmap 的渲染器是否应按比例缩小 bitmap 图时尝试使用 mipmaps 。

如果知道要以小于其原始大小的 50％绘制此 bitmap ，则可能可以获得更高的质量。

此属性只是一个渲染器可以忽略的建议。不能保证有任何效果。

### final void setHasMipMap(boolean hasMipMap)

指示负责绘制此 bitmap 的渲染器是否应按比例缩小 bitmap 图时尝试使用 mipmaps 。

如果知道要以小于其原始大小的 50％绘制此 bitmap ，则可能可以获得更高的质量。

此属性只是一个渲染器可以忽略的建议。不能保证有任何效果。

* boolean hasMipMap；指示渲染器是否应尝试使用mipmaps。

### final ColorSpace getColorSpace()

返回与此 bitmap 关联的色彩空间。如果颜色空间未知，则此方法返回 null 。

### void setColorSpace(@NonNull ColorSpace colorSpace)

将 bitmap 识别为具有指定的 ColorSpace ，而不影响 bitma 的底层分配。

这影响 framework 层如何解释每个像素的颜色。具有 Config.ALPHA_8 的 bitmap 永远不会有颜色空间，因为颜色空间不会影响 a​​lpha 通道。其他 Config 一定具有非 null 的 ColorSpace。

### void eraseColor(@ColorInt int c)

用指定的 Color 填充 bitmap 的像素。

### void eraseColor(@ColorLong long color)

用指定的 ColorLong 填充 bitmap 的像素。

### int getPixel(int x, int y)

返回指定位置的 Color 。如果 x 或 y 超出范围（负值或者分别大于等于宽度或高度，则抛出异常）。返回的颜色是 ColorSpace.Named.SRGB 颜色空间中的非预乘ARGB值。

* int x：要返回像素的x坐标（0 ... width-1）。
* int y：要返回像素的y坐标（0 ... height-1）。

### Color getColor(int x, int y)

返回指定位置的 Color 。如果 x 或 y 超出范围（负值或者分别大于等于宽度或高度，则抛出异常）。返回的颜色是 ColorSpace.Named.SRGB 颜色空间中的非预乘ARGB值。

* int x：要返回像素的x坐标（0 ... width-1）。
* int y：要返回像素的y坐标（0 ... height-1）。

### void getPixels(@ColorInt int[] pixels, int offset, int stride, int x, int y, int width, int height)

以 pixel[] 数组的形式返回 bitmap 中数据的副本。每个值都是表示 Color 的包装 int 。 stride 参数允许调用方在行之间的返回像素数组中设置步长。对于正常的打包结果，只需通过宽度作为步长即可。 返回的颜色是 ColorSpace.Named.SRGB 颜色空间中的非预乘ARGB值。

* int[] pixels：接收 bitmap 颜色的数组。
* int offset：写入像素的第一个索引[]。
* int stride：要在行之间跳过的像素数[]的条目数（必须 >= bitmap的宽度）。可以为负。
* int x：从 bitmap 读取的第一个像素的 x 坐标。
* int y：从 bitmap 读取的第一个像素的 y 坐标。
* int width：每行要读取的像素数。
* int height：要读取的行数。

### void setPixel(int x, int y, @ColorInt int color)

在 x，y 坐标处将指定的 Color 写入 bitmap（假设 bitmap 是可变的）。颜色必须是 ColorSpace.Named.SRGB 颜色空间中的非预乘ARGB值。

* int x：要替换的像素的 x 坐标（0 ... width-1）。
* int y：要替换的像素的 y 坐标（0 ... height-1）。
* int color：写入 bitmap 的ARGB颜色。

### void setPixels(@ColorInt int[] pixels, int offset, int stride, int x, int y, int width, int height)

用数组中的颜色替换 bitmap 中的像素。数组中的每个元素是一个包装 int ，表示 ColorSpace.Named.SRGB 颜色空间中未预乘的ARGB Color。

* int[] pixels：写入 bitma 的颜色数组。
* int offset：从像素读取的第一种颜色的索引[]。
* int stride：以像素为单位的颜色数，以在行之间跳过。通常，此值将与 bitmap 的宽度相同，但是可以更大（或为负）。
* int x：bitmap 中要写入的第一个像素的 x 坐标。
* int y：bitmap 中要写入的第一个像素的 y 坐标。
* int width：每行要从 pixel[] 复制的颜色数。
* int height：写入 bitmap 的行数。

### Bitmap extractAlpha()

返回捕获原始图像的 Alpha 值的新 bitmap。 可以使用Canvas.drawBitmap() 进行绘制，其中颜色将从传递给绘图调用的 paint 中获取。

### Bitmap extractAlpha(Paint paint, int[] offsetXY)

返回捕获原始图像的 Alpha 值的新 bitmap。 这些值可能会受到可选的Paint参数的影响，该参数可以包含自己的 alph 值，还可以包含 MaskFilter ，后者可以更改结果 bitmap 的实际尺寸（例如模糊 maskfilter 可能会放大结果 bitmap）。如果 offsetXY 不为 null ，则返回偏移返回 bitmap 的量，以便在逻辑上与原始对齐。例如，如果绘画包含半径为2的模糊，则 offsetXY[] 将包含 -2，-2，这样绘制 alpha 的 bitmap 偏移量为（-2，-2），然后绘制原图将使得模糊效果与原图在视觉上对齐。

* Paint paint：用于修改结果 bitmap 中的 alpha 值的可选画笔。为默认传递 null 。
* int[] offsetXY：可选数组，该数组返回放置返回的 bitmpa 所需的 X（索引0）和 Y（索引1）偏移，以使其在视觉上与原始行对齐。

### boolean sameAs(Bitmap other)

给定另一个 bitmap ，如果它具有与此 bitmap 相同的尺寸，配置，和像素数据，则返回true。如果其中任何一个不同，则返回false。 如果 other 为 null ，则返回 false 。

### void prepareToDraw()

构建与用于绘制 bitmap 的 bitmap 关联的缓存。

从 android.os.Build.VERSION_CODES.N（Android 7.0，24）开始，如果尚未上传 bitmap，则此调用会在 RenderThread 上启动异步上传到GPU。使用硬件加速时，必须将 bitmap 上传到GPU才能进行渲染。默认情况下，这是在第一次绘制 bitmap 时完成的，但是此过程可能需要几毫秒的时间，具体取决于 bitmap 的大小。每次修改并再次绘制 bitmap 时，都必须重新上传。

提前调用此方法可以节省使用第一帧的时间。例如，建议在解码的 bitmap 即将显示时在图像解码工作线程上调用此方法。建议在调用此方法之前对 bitmap 进行任何预绘制修改，以便可以重新使用缓存的上传副本，而无需重新上传。

在 android.os.Build.VERSION_CODES.KITKAT （Android 6.0，23）及以下版本中，对于可清除的 bitmap ，此调用将尝试确保像素已解码。
