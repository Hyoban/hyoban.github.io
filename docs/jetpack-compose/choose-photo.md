# 如何选择相册图片并展示

关于在非 compose 界面中的写法，参考另外一篇 [ActivityResultContracts](https://note.hyoban.fun/android/activity-result-contracts/) 即可。

## 快进到正题

但是现在我们使用的是 compose ，没办法和 activity 直接打交道。实际上在 compose 中提供了对应的函数 `rememberLauncherForActivityResult` ，写法上完全一致。

```kotlin
val context = LocalContext.current

var bitmap by remember { mutableStateOf<Bitmap?>(null) }

val launcherFromPhotos =
    rememberLauncherForActivityResult(ActivityResultContracts.GetContent()) {
        bitmap = BitmapFactory.decodeStream(context.contentResolver.openInputStream(it))
    }
```

需要注意的是，我们需要在 compose 中使用 `LocalContext.current` 获取 context 。

然后我们将 bitmap 加载到 compose 中的 `Image` 组件即可，注意不要使用 `Icon` 这个组件。

```kotlin
bitmap?.let {
    Image(bitmap = it.asImageBitmap(), contentDescription = "")
}
```
