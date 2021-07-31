---
title: 简析BitmapFactory
date: 2019-10-14 16:59:44
updated: 2019-10-14 16:59:46
categories:
  - Android
tags:
  - Android
  - 图片
  - Bitmap
---

基于 9.0（29）简析 BitmapFactory 类。

我们只关注 BitmapFactor 类暴露出来的（即访问权限为 public）东西。

- 静态内部类 Options
- 若干 decode 方法

## 静态内部类 Options

Options，顾名思义，是 BitmapFactory 用于 decode 方法的选项参数。让我们看看有哪些：

### Bitmap inBitmap

如果使用此参数，decode 方法会在加载内容时尝试重用此 bitmap ，如果编码操作不能使用此 bitmap （有限制条件），则会抛出 java.lang.IllegalArgumentException 异常。当前的重用实现方式要求 bitmap 必须是可变的，并且即使一个资源在 decode 后通常会得到不可变的 bitmap，在重用成功后所得到的重用 bitmap 也将保持可变性。任何可变的 bitmap 都可以被 BitmapFactory 重用，以解码任何其他 bitmap，只要解码后的 bitmap （待分配内存）的字节数（getByteCount 方法，解码后待分配内存状态的大小）小于或等于分配的字节数（getAllocationByteCount 方法，占用内存的实际大小）。这可能是因为待分配内存的 bitmap 固有尺寸较小，或者其缩放后的尺寸（对于密度/样本尺寸）较小。

在 android.os.Build.VERSION_CODES.KITKAT（Android4.4，19）之前，使用有更多限制：

- 解码的图像（无论是作为资源还是作为流）必须为 jpeg 或 png 格式。
- 仅支持大小相等的 bitmap，并且 inSampleSize 设置为 1。
- 另外，重用 bitmap 的 Bitmap.Config 配置 inPreferredConfig 将被覆盖（如果已设置）。

BitmapRegionDecoder 会将其请求的内容绘制到提供的 bitmap 中，如果输出内容大小（缩放后）大于提供的 bitmap ，则进行裁剪。提供的 bitmap 的宽度，高度和 Bitmap.Config 不会更改。
android.os.Build.VERSION_CODES.JELLY_BEAN（Android 4.1，16）中引入了对 inBitmap 的 BitmapRegionDecoder 支持。 BitmapRegionDecoder 支持的所有格式 通过 inBitmap 支持 bitmap 重用。

### boolean inMutable

在 native 代码中使用。

如果开启该设置，则 decode 方法将始终返回可变的 bitmap。这可以用于以编程方式将效果应用于通过 BitmapFactory 加载的 Bitmap。不能与 inPreferredConfig = Bitmap.Config.HARDWARE 同时设置，因为硬件 bitmap 始终是不可变的。

### boolean inJustDecodeBounds

如果开启该设置， 解码器将会返回 null （不返回 bitmap ），但是 out... 字段（如 outWidth 、 outHeight 、 outMimeType ）会被赋值，从而允许调用者查询 bitmap ，而不必为其像素分配内存。

### int inSampleSize

如果设置的值大于 1，将会请求解码器对原始图像进行二次采样，返回较小的图像以节省内存。样本大小是任一维度中与已解码 bitmap 中单个像素对应的像素数。例如，设置 inSampleSize == 4 ，将会返回一张是宽高为原始宽度和高度的 1/4 的图像，像素为原来的 1/16。任何小于等于 1 的值都与 1 相同。注意：解码器使用基于 2 的幂的最终值，任何其他值将四舍五入为最接近的 2 的幂。

### Bitmap.Config inPreferredConfig

如果该配置不为 null ，解码器将会尝试解析 bitmap 成此内部配置（bitmap 的像素存储方式）。如果该配置为 null，或者无法满足要求，解码器将会基于系统屏幕深度和原始图像的特征（如是否有每像素透明度）来选择最合适的配置（要求该配置也满足要求）。

图像默认以 Bitmap.Config.ARGB_8888 的配置加载。

### ColorSpace inPreferredColorSpace

如果改配置不为 null ，则解码器将尝试解码 bitmap 到此颜色空间中。如果该配置为 null ，或者无法满足要求，则解码器将选择嵌在图像的颜色空间或最适合请求的图像配置的颜色空间（例如，ColorSpace.Named.SRGB 对应 Bitmap.Config.ARGB_8888 ; ColorSpace.Named.EXTENDED_SRGB 对应 Bitmap.Config.RGBA_F16）。

目前只支持 ColorSpace.Model.RGB 颜色空间，如果设置了非 RGB 颜色空间（例如 ColorSpace.Named.CIE_LAB）， decode 方法将会抛出异常（IllegalArgumentException）。

指定的色彩空间的传递函数必须是 ColorSpace.Rgb.TransferParameters() ICC 参数曲线。如果在指定的色彩空间调用方法 ColorSpace.Rgb.getTransferParameters() 返回 null ， decode 方法将会抛出异常（IllegalArgumentException）。

解码之后，bitmap 的色彩空间存储在 BitmapFactory.Options.outColorSpace 字段。

### boolean inPremultiplied

如果开启此配置（默认开启），则生成的 bitmap 的颜色通道将被 alpha 通道预乘。

由视图系统或通过 Canvas 直接绘制的图像，不应将其设置为 false。视图系统和 Canvas 假定所有绘制的图像都已预乘以简化绘制时融合，并且在绘制未预乘时将抛出 RuntimeException 。

仅当您要处理原始编码的图像数据（例如，使用 RenderScript 或自定义 OpenGL）时才是合适的。

不会影响没有 Alpha 通道的 bitmap 。

将 inScaled 设置为 true 时将此标志设置为 false 可能会导致颜色错误。

### boolean inDither

@deprecated 自 android.os.Build.VERSION_CODES.N （Android7.0，24）起，此配置被忽略。

在 android.os.Build.VERSION_CODES.M (Android6.0，23)及以下版本中，如果设置为 true，则解码器将尝试对解码的图像进行抖动。

### int inDensity

用于 bitmap 像素密度。这将使得在返回的位图中始终有为其设置的密度（请参阅 Bitmap.setDensity(int)）。此外，如果开启了 inScaled（默认情况下开启），并且此密度与 inTargetDensity 不匹配，则 bitmap 在返回之前将被缩放为目标密度。

如果设置为 0， BitmapFactory.decodeResource(Resources, int) 、 BitmapFactory.decodeResource(Resources, int, android.graphics.BitmapFactory.Options) 、 BitmapFactory.decodeResourceStream 这些 decode 方法将填充与资源关联的密度。其他 decode 方法将保持原样，并且不会应用任何密度。

### int inTargetDensity

此 bitmap 将被绘制到的目标像素密度。 与 inDensity 和 inScaled 结合使用，以确定在返回 bitmap 之前是否以及如何缩放位图。

如果设置为 0， BitmapFactory.decodeResource(Resources, int) 、 BitmapFactory.decodeResource(Resources, int, android.graphics.BitmapFactory.Options) 、 BitmapFactory.decodeResourceStream 这些 decode 方法将填充与资源对象的 DisplayMetrics 相关的密度。其他 decode 方法将保持原样，并且不会对密度进行缩放。

### int inScreenDensity

正在使用的实际屏幕的像素密度。 仅用于以密度兼容代码运行的应用程序，其中 inTargetDensity 实际上是应用程序看到的密度而不是实际的屏幕密度。

通过设置此选项，允许加载代码避免将当前的屏幕密度的 bitmap 缩放到/降低到兼容密度。相反的，如果 inDensity 与 inScreenDensity 相同，则 bitmap 将保持不变。任何使用生成的 bitmap 的对象，还必须使用 Bitmap.getScaledWidth 和 Bitmap.getScaledHeight 来说明 bitmap 的密度与目标密度之间的任何差异。

BitmapFactory 自身永远不会为调用者自动设置。必须明确设置它，因为调用者必须以密度感知的方式处理结果位图。

### boolean inScaled

如果设置了该配置，并且 inDensity 和 inTargetDensity 不为 0，bitmap 在加载时将缩放以匹配 inTargetDensity，而不是每次绘制到 Canvas 时都依赖于图形系统对其进行缩放。

BitmapRegionDecoder 会忽略这个配置，并且不会根据密度缩放输出。（尽管支持 inSampleSize ）

此配置默认开启，如果需要 bitmap 的非缩放版本，则应将其关闭。.9 bitmaps（Nine-patch bitmaps）会忽略此配置，并且始终会缩放。

如果 inPremultiplied 设置为 false ，并且图像具有 Alpha 透明度，将此配置设置为 true 可能会导致颜色错误。

### boolean inPurgeable

@deprecated 自 android.os.Build.VERSION_CODES.LOLLIPOP（Android5.0，21） 起，此配置被忽略。

在 android.os.Build.VERSION_CODES.KITKAT（Android4.4，19）及以下版本中，如果将此配置设置为 true ，那么生成的 bitmap 将分配其像素，以便在系统需要回收内存时可以将其清除。在那种情况下，当需要再次访问像素时（例如绘制 bitmap，调用 getPixels（）），它们将被自动重新解码。

为了进行重新解码，bitmap 必须通过共享对输入的引用或对其进行复制来访问编码的数据。此区别由 inInputShareable 控制。如果 inInputShareable 为 true，则 bitmap 可能会保留对输入的浅引用。如果这 inInputShareable 为 false，则 bitmap 将显式地复制输入数据，并将其保留。即使允许共享，实现仍可以决定对输入数据进行深拷贝。

尽管 inPurgeable 可以帮助避免大的 Dalvik 堆分配（从 API 级别 11 开始），但是它牺牲了性能可预测性，因为视图系统尝试绘制的任何图像都可能会导致解码延迟，从而导致帧丢失。因此，大多数应用应避免使用 inPurgeable 来提供快速流畅的 UI。为了最小化 Dalvik 堆分配，请使用 inBitmap 配置。

该配置会被 decodeResource(Resources, int, android.graphics.BitmapFactory.Options) 或者 decodeFile(String,
android.graphics.BitmapFactory.Options)

### boolean inInputShareable

@deprecated 自 android.os.Build.VERSION_CODES.LOLLIPOP（Android5.0，21） 起，此配置被忽略。

在 android.os.Build.VERSION_CODES.KITKAT（Android4.4，19）及以下版本中，此配置与 inPurgeable 结合使用。如果 inPurgeable 为 false ，则会忽略此字段。如果 inPurgeable 为 true ，则配置确定 bitmap 是否可以共享对输数据（inputstream，数组等）的引用，或者是否必须进行深拷贝。

### boolean inPreferQualityOverSpeed

@deprecated 自 android.os.Build.VERSION_CODE.N （Android7.0，24）起，此配置被忽略。输出将始终是高质量的。

在 android.os.Build.VERSION_CODES.M (Android6.0，23)及以下版本中，如果将 inPreferQualityOverSpeed 设置为 true ，则解码器将尝试以牺牲解码速度为代价，将重构图像解码为更高质量的图像。当前该字段仅影响 JPEG 解码，在这种情况下，将使用更准确但稍慢的 IDCT 方法代替。

### int outWidth

bitmap 的解码结果宽度。如果 inJustDecodeBounds 设置为 false，这将是应用所有缩放后输出 bitmap 的宽度。如果为 true，它将是输入图像的宽度，不考虑缩放。

如果尝试解码时发生错误，outWidth 将设置为-1。

### int outHeight

bitmap 的解码结果高度。如果 inJustDecodeBounds 设置为 false，这将是应用所有缩放后输出 bitmap 的高度。如果为 true，它将是输入图像的高度，不考虑缩放。

如果尝试解码时发生错误，outHeight 将设置为-1。

### String outMimeType

如果明确，则将该字符串设置为解码图像的媒体类型。 如果未知或有错误，则将其设置为 null 。

### Bitmap.Config outConfig

如果明确，为解码 bitmap 具有的像素存储方式配置。 如果未知或有错误，则将其设置为 null 。

### ColorSpace outColorSpace

如果明确，为解码后的 bitmap 具有的色彩空间。注意，输出颜色空间不保证是 bitmap 编码的颜色空间。如果未知（例如，配置为 Bitmap.Config.ALPHA_8）或存在错误，则将其设置为 null 。

### byte[] inTempStorage

用于解码的临时存储。建议 16K 左右。

### boolean mCancel

@deprecated 自 android.os.Build.VERSION_CODES.N （Android7.0，24）起。

指示已在此对象上调用 cancel 的标志。如果有一个中间人想要首先解码边界然后解码图像，则此配置很有用。在那种情况下，中间人可以在边界解码和图像解码之间检查是否取消了该操作。

### public void requestCancelDecode()

@deprecated 自 android.os.Build.VERSION_CODES.N （Android7.0，24）起，不影响解码，尽管设置了 mCancel 的值为 true。

在 android.os.Build.VERSION_CODES.M (Android6.0，23)及以下版本。调用此命令将通知解码器应该取消其操作。这不能保证取消解码，但是如果成功取消解码，则解码器操作结果将返回 null ，或者如果 inJustDecodeBounds 为 true ，则将 outWidth / outHeight 设置为-1。

## decode 方法

### decode 方法简析

#### Bitmap decodeFile(String pathName, Options opts)

将文件路径解码为 bitmap 。如果指定的文件名为 null ，或无法解码为 bitmap，则该函数返回 null 。

- String pathName：待解码文件的完整路径名
- Options opts：控制下采样以及是否应完全解码图像或仅返回尺寸的选项参数，可为 null。

实际内部调用了另外一个 decode 方法：
Bitmap decodeStream(@Nullable InputStream is, @Nullable Rect outPadding, @Nullable Options opts)
（参数 String pathName -> InputStream is, Rect null , Options opts）

#### Bitmap decodeFile(String pathName)

重载 Bitmap decodeFile(String pathName, Options opts)

decodeFile(pathName, null);

#### Bitmap decodeResourceStream(@Nullable Resources res, @Nullable TypedValue value, @Nullable InputStream is, @Nullable Rect pad, @Nullable Options opts)

从 InputStream 解码新的 bitmap 。此 InputStream 是从资源获得的，我们通过传递这些资源可以相应地缩放位图。

#### Bitmap decodeResource(Resources res, int id, Options opts)

打开给定的资源并且调用 decodeResourceStream 方法的代名词。

- Resources res：包含图像数据的资源对象。
- int id：图像数据的资源 ID。
- Options opts：控制下采样以及是否应完全解码图像或仅返回尺寸的选项参数，可为 null。

实际内部调用了另外一个 decode 方法：
decodeResourceStream(@Nullable Resources res, @Nullable TypedValue value, @Nullable InputStream is, @Nullable Rect pad, @Nullable Options opts)
（参数 Resources res, TypedValue value = new TypedValue(), InputStream res.openRawResource(id, value), Rect null, Options opts）

#### Bitmap decodeResource(Resources res, int id)

重载 Bitmap decodeResource(Resources res, int id, Options opts)

decodeResource(res, id, null);

#### Bitmap decodeByteArray(byte[] data, int offset, int length, Options opts)

从指定的字节数组解码不可变的 bitmap。

- byte[] data：压缩图像数据的字节数组。
- int offset：解码器应开始解析的 imageData 的偏移量。
- int length：从偏移量开始解析的字节数。
- Options opts：控制下采样以及是否应完全解码图像或仅返回尺寸的选项参数，可为 null。

调用 native 解码方法：
private static native Bitmap nativeDecodeByteArray(byte[] data, int offset, int length, Options opts, long inBitmapHandle, long colorSpaceHandle)

#### Bitmap decodeByteArray(byte[] data, int offset, int length)

重载 Bitmap decodeByteArray(byte[] data, int offset, int length, Options opts)

decodeByteArray(data, offset, length, null);

#### Bitmap decodeStream(@Nullable InputStream is, @Nullable Rect outPadding, @Nullable Options opts)

将输入流解码为 bitmap 。如果输入流为 null ，或者不能用于解码 bitmap ，则该函数返回 null 。 流的位置将是读取编码数据之后的位置。

- InputStream is：输入流，其中包含要解码为 bitmap 的原始数据。
- Rect outPadding：如果不为 null ，则返回 bitmap 的填充矩形（如果存在），否则将填充设置为[-1，-1，-1，-1]。如果没有 bitmap 返回（ null ），则填充是不变的。
- Options opts：控制下采样以及是否应完全解码图像或仅返回尺寸的选项参数，可为 null。

如果输入流是 AssetManager.AssetInputStrea，调用 native 解码方法：
private static native Bitmap nativeDecodeAsset(long nativeAsset, Rect padding, Options opts,
long inBitmapHandle, long colorSpaceHandle)
否则调用私有内部方法：
Bitmap decodeStreamInternal(@NonNull InputStream is, @Nullable Rect outPadding, @Nullable Options opts)

#### decodeStream(InputStream is)

重载 Bitmap decodeStream(@Nullable InputStream is, @Nullable Rect outPadding, @Nullable Options opts)

decodeStream(is, null, null);

#### Bitmap decodeFileDescriptor(FileDescriptor fd, Rect outPadding, Options opts)

从文件描述符解码 bitmap 。如果 bitmap 无法解码返回 null 。当返回时，描述符中的位置不会更改，因此可以按原样再次使用描述符。

- FileDescriptor fd：包含要解码的 bitmap 数据的文件描述符。
- Rect outPadding：如果不为 null ，则返回 bitmap 的填充矩形（如果存在），否则将填充设置为[-1，-1，-1，-1]。如果没有 bitmap 返回（ null ），则填充是不变的。
- Options opts：控制下采样以及是否应完全解码图像或仅返回尺寸的选项参数，可为 null。

#### Bitmap decodeFileDescriptor(FileDescriptor fd)

重载 Bitmap decodeFileDescriptor(FileDescriptor fd, Rect outPadding, Options opts)

decodeFileDescriptor(fd, null, null);

### decode 方法总结

所有的 decode 方法最后都会走下面三个 decode 方法：

Bitmap decodeStream(@Nullable InputStream is, @Nullable Rect outPadding, @Nullable Options opts)

Bitmap decodeByteArray(byte[] data, int offset, int length, Options opts)

Bitmap decodeFileDescriptor(FileDescriptor fd, Rect outPadding, Options opts)

其中 decodeStream 和 decodeFileDescriptor 方法 有可能走以下这个方法：

Bitmap decodeStreamInternal(@NonNull InputStream is, @Nullable Rect outPadding, @Nullable Options opts)

实质是调用了 native 解码方法：
Bitmap nativeDecodeStream(InputStream is, byte[] storage, Rect padding, Options opts, long inBitmapHandle, long colorSpaceHandle)

因此 native 以下四个方法是 decode 的最终归宿：

- private static native Bitmap nativeDecodeStream(InputStream is, byte[] storage, Rect padding, Options opts, long inBitmapHandle, long colorSpaceHandle);
- private static native Bitmap nativeDecodeFileDescriptor(FileDescriptor fd, Rect padding, Options opts, long inBitmapHandle, long colorSpaceHandle);
- private static native Bitmap nativeDecodeAsset(long nativeAsset, Rect padding, Options opts, long inBitmapHandle, long colorSpaceHandle);
- private static native Bitmap nativeDecodeByteArray(byte[] data, int offset, int length, Options opts, long inBitmapHandle, long colorSpaceHandle);
