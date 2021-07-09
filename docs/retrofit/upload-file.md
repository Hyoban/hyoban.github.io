# 上传文件

## Http 中是什么样子

```http
POST {{base_url}}/sys/common/upload HTTP/1.1
Content-Type:  multipart/form-data; boundary=----WebKitFormBoundary0Xlfh1skUlQI1Kqe

------WebKitFormBoundary0Xlfh1skUlQI1Kqe
Content-Disposition: form-data; name="biz"

temp

------WebKitFormBoundary0Xlfh1skUlQI1Kqe
Content-Disposition: form-data; name="file"; filename="1_tDFPLaEDlaW5dtsfv4sd0A.png"
Content-Type: image/png

< /Users/hyoban/Downloads/1_tDFPLaEDlaW5dtsfv4sd0A.png

------WebKitFormBoundary0Xlfh1skUlQI1Kqe--
```

我们需要指定 `Content-Type` 为 `multipart/form-data` ，如果内容包含多段的话，我们需要指定分割符，每段分隔符之间就是一段数据。注意分隔符之前都有多余的两个 -- ，就和最后一段数据最后一样。

这里我们第一段是普通键值对数据，第二段是上传的文件， filename 指定文件名。后面主要内容的实际数值为上传文件的二进制数据，这里是使用 vscode 的 rest client 插件的功能。

## 定义 retrofit api 接口

```kotlin
@Multipart
@POST("/sys/common/upload")
suspend fun uploadFile(
    @Part("biz") biz: RequestBody,
    @Part file: MultipartBody.Part,
): UploadFileResponse
```

使用 `@Multipart` 注解表示请求体的内容格式为 multi-part ，请注意，这时我们的参数就都要加上 `@Part` 的注解了。

对于第一个参数为普通字符串，我们指定为 `RequestBody` 类型即可，而文件的话，就需要 `MultipartBody.Part` ，稍后我们将使用 `createFormData` 方法来创建 formdata 的格式。

## Repository 层调用接口

```kotlin
suspend fun uploadFile(inputStream: InputStream, fileName: String) = flow {
    val requestFile =
        RequestBody.create(MediaType.parse("multipart/form-data"), inputStream.readBytes())
    val body = MultipartBody.Part.createFormData("file", fileName, requestFile)
    val biz = RequestBody.create(MediaType.parse("multipart/form-data"), "temp")
    emit(api.uploadFile(biz, body))
}
```

对于文件类型， `createFormData` 需要我们传递一个 `RequestBody` 类型的对象，也就是我们文件的具体内容。

```kotlin
Part createFormData(String name, @Nullable String filename, RequestBody body)
```

我们使用 `create` 方法来创建，这里我们有两种方式。一种是通过输入流读到 byte 数组，一种是直接传递文件。由于我通过 SAF 拿到用户选择的公用文件，只能拿到 uri ，不方便转化为文件，所以通过输入流来创建。

```kotlin
RequestBody create(final @Nullable MediaType contentType, final byte[] content)

RequestBody create(final @Nullable MediaType contentType, final File file)
```

我们需要将请求体的格式都指定为 `multipart/form-data` 。

## 后续

后面我们就依据需要的参数来调用方法即可，对于图片相关可以看我其他的文章。

## 参考

[How to Upload Image file in Retrofit 2](https://stackoverflow.com/questions/39953457)
