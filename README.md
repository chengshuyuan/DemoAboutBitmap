# Bitmap的加载
## 1 Bitmap的加载
- 1 由于bitmap的特殊性以及Android对于单个应用所施加的内存限制，这导致在加载Bitmap的时候很容易出现内存溢出，因此如何高效地加载Bitmap是一个很重要的问题。
- 2 如何加载一张图片，Android的BitmapFactory提供了四种方法
    - 1 decodeFile 从文件系统加载一个Bitmap对象
    - 2 decodeResource 从资源中加载一个Bitmap对象
    - 3 decodeStream 从输入流中加载一个Bitmap对象
    - 4 decodeByteArray 从字节数组中加载一个Bitmap对象
- 3 如何高效加载Bitmap呢？就是采用BitmapFactory.Options来加载所需尺寸的图片
    - 通过BitmapFactory.Options就可以按照一定的采样率来加载缩小后的图片，将缩小后的图片在ImageView中显示，这样就会降低内存占用率而一定程度上避免OOM,提高了Bitmap加载时的性能
    - BitmapFactory提供的加载图片的四类方法都支持BitmapFactory.Options参数，通过它们就可以很方便地对一个图片进行采样缩放
- 4 当BitmapFactory.Options来缩放图片，主要用到了它的inSampleSize参数，即采样率
    - 当inSampleSize为1时，采样后的图片大小为图片的原始大小(小于1时，无缩放效果)
    - 当inSampleSize大于1，如为2时，采样后的图片的宽高均为原图的1/2,像素为原图的1/4,所占用的内存大小也为原图的1/4
    - inSampleSize应为2的指数，如果不是2的指数，那系统会向下取整并选择一个最接近的2的指数来代替（单并非所有的Android版本都成立）
- 5 采用采样率的步骤
    - 1 将BitmapFactory.Options的inJustDecodeBounds参数设为true并加载图片
    - 2 从BitmapFactory.Options中取出图片的原始大小，对应于outWith和outHeight参数
    - 3 根据采样率的规则并结合目标View的大小计算出采样率inSampleSize
    - 4 将BitmapFactory.Options的inJustDecodeBounds参数为false，然后重新加载图片
- 6 注意：
    - 1 inJustDecodeBounds的参数，当此参数为true时，BitmapFactory只会解析图片的原始宽高信息，而并不会真正的去加载图片，所以这个操作是轻量级的
    - 2 这个时候获得的图片宽高信息与图片的位置以及程序运行的设备有关，如一张图片放在不同的drawable目录下会获取不同的结果



## 2 Bitmap的缓存策略
- 1 实际开发中经常用到Bitmap做缓存，通过缓存策略，我们不需要每次都从此那个网络上请求图片或者从存储设备中加载图片，这样就极大地提高了图片的加载效率
- 2 目前常用的缓存策略有LruCache和DiskLruCache,其中LruCache被用作内存缓存，DiskLruCache常被用作存储缓存