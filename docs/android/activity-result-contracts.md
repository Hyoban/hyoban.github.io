# 使用 Intent 启动系统组件

安卓开发中，我们经常需要调用一些系统的组件，比如相机，选择文件等。下面我们以选择相册中的图片并展示这个例子来讲解。

## 从前我们是怎么做的

基本步骤是

1. 定义一个获取从系统文件选择器中获取图片的 Intent
2. 使用 `startActivityForResult` 来启动 Activity
3. 在 `onActivityResult` 的回调中处理返回的 `uri` 数据

当然我们需要将拿到的 `uri` 转成 bitmap 才能加载到 imageview 上

```kotlin
const val IMAGE_MIME_TYPE = "image/*"
const val IMAGE_PICKER_REQUEST_CODE = 1

private fun selectImage() {
    val intent = Intent(Intent.ACTION_GET_CONTENT).apply {
        type = IMAGE_MIME_TYPE
    }

    startActivityForResult(intent, IMAGE_PICKER_REQUEST_CODE)
}

override fun onActivityResult(requestCode: Int, resultCode: Int, resultData: Intent?) {
    super.onActivityResult(requestCode, resultCode, resultData)

    if (requestCode == IMAGE_PICKER_REQUEST_CODE && resultCode == RESULT_OK) {
        resultData?.data?.let { uri ->
            activity?.let {
                binding.preview.setImageBitmap(
                    BitmapFactory.decodeStream(it.contentResolver.openInputStream(uri))
                )
            }
        }
    }
}
```

不过由于耦合严重和一大堆 `requestcode` ，现在 `startActivityForResult` 已经被废弃，我们需要使用 `registerForActivityResult`。

## 新的写法

```kotlin
private val processingPictures =
    registerForActivityResult(ActivityResultContracts.StartActivityForResult()) {
        if (it.resultCode == RESULT_OK) {
            it.data?.data.let { uri ->
            
            }
        }
    }

fun selectImage() {
    val intent = Intent(Intent.ACTION_GET_CONTENT).apply {
        type = IMAGE_MIME_TYPE
    }
    processingPictures.launch(intent)
}
```

我们创建一个启动器，指定启动需要的协议，然后使用 launch 函数启动即可。

```kotlin
abstract class ActivityResultContract<I, O>
```

值得一提的是， `ActivityResultContract` 是一个抽象类，它的范型参数分别为，我们启动的时需要指定的参数和在回调中我们能够获取到的数据。我们需要实现自己的协议类，而这里采用了系统预置的通用的 `StartActivityForResult` ，整体写法和之前还是很类似的。

很棒，我们不用定义 `requestcode` 了，也不用在混着各种内容的 `onActivityResult` 中进行操作了。但还可以更加简单一点。

```kotlin
private val processingPictures =
    registerForActivityResult(ActivityResultContracts.GetContent()) { uri ->
        
    }

fun selectImage() {
    processingPictures.launch(IMAGE_MIME_TYPE)
}
```

从文件管理器中获取内容是系统提供给我们的预置协议，我们将 `Contract` 指定为 `GetContent`

```kotlin
public static class GetContent extends ActivityResultContract<String, Uri>
```

我们需要在 `launch` 函数中指定选择文件的类型，这里和之前一样指定为 `"image/*"` 即可，在回调中拿到 `uri` 。

## 从相机拍一张？

将协议指定为 `TakePicture` 即可，`launch` 函数中传递保存图片地址的 uri ，请注意回调中我们只能拿到是否拍照成功的数据，所以我们一般需要记录下传递的 uri 便于使用拍摄的照片。

对于回调中的处理是这样的，我们可以拿到是否拍摄成功的信息，如果拍摄成功，之前记录下的 uri 就能加载出图片了。

```kotlin
val filename = "${System.currentTimeMillis()}.jpg"

val imageUri =
    if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.Q) {
        MediaStore.Images.Media.getContentUri(
            MediaStore.VOLUME_EXTERNAL_PRIMARY
        )
    } else {
        MediaStore.Images.Media.EXTERNAL_CONTENT_URI
    }

val imageDetails = ContentValues().apply {
    put(MediaStore.Images.Media.DISPLAY_NAME, filename)
}

context.contentResolver.insert(imageUri, imageDetails).let {
    takenPictureUri = it
    launcherFromCamera.launch(takenPictureUri)
}
```

对于传递的 uri ，我们需要先自己构建出，这里我们指定出文件名并填充即可。

## 参考

[再见！onActivityResult！你好，Activity Results API！](https://segmentfault.com/a/1190000037601888)
