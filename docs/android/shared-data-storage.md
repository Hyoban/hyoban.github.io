# 共享存储空间

## 如何交互获取到的 uri

我们可以通过 contentResolver 来打开 uri 的输入流，然后对于流进行操作。

```kotlin
val resolver = applicationContext.contentResolver

resolver.openInputStream(content-uri).use { stream ->
    
}
```

## 查询存储空间对应的文件信息

通过 contentResolver 按需获取到对应的 Cursor

我们已经拿到 uri 的话并且只希望查询对应的，可以直接传递 uri 作为第一个参数，否则传递 `MediaStore.Images.Media.EXTERNAL_CONTENT_URI` 来查询全部数据（这里以查询图像为示例）

第二个参数传递我们希望查询的数据类型的数组（这里假设我们只需要拿到文件名）

第三四个参数对应查询语句和语句中的变量，第五个参数为排序方式。只是查询对应文件的话，我们就留为空即可

```kotlin
val cursor = resolver.query(
    uri, 
    arrayOf(MediaStore.Images.Media.DISPLAY_NAME), 
    null, 
    null, 
    null
)
```

然后我们就可以开始查询了，我们先缓存下数据库中对应的行，然后获取对应行的值。这里我们确定查询对应的文件信息，如果是获取多个文件数据的话，就改为 `while (cursor.moveToNext())` 即可，持续获取数据

```kotlin
cursor?.use {
    val columnIndex = 
        cursor.getColumnIndexOrThrow(MediaStore.Images.Media.DISPLAY_NAME)
    if (cursor.moveToFirst()) {
        if (columnIndex > -1) {
            fileName = cursor.getString(columnIndex)
        }
    }
}
```

如果我们是在无 uri 的情况下，查询所需的多个文件，那我们应该需要拿到文件的 uri 。要实现这个操作，我们需要先查询到 id ，然后将 id 附加到内容 uri 来获取到文件的 uri

```kotlin
val contentUri = ContentUris.withAppendedId(
    MediaStore.Images.Media.EXTERNAL_CONTENT_URI, 
    id
)
```

> 在应用中执行此类查询时，请注意以下几点：
> 
> 1. 在工作线程中调用 query() 方法。
> 2. 缓存列索引，以免每次处理查询结果中的行时都需要调用 getColumnIndexOrThrow()。
> 3. 将 ID 附加到内容 URI，如代码段所示。
> 4. 搭载 Android 10 及更高版本的设备需要在 MediaStore API 中定义的列名称。如果应用中的某个依赖库需要 API 中未定义的列名称（例如 "MimeType"），请使用 CursorWrapper 在应用的进程中动态转换列名称。
