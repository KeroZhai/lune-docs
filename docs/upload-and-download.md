# 文件上传与下载

## 文件上传

和普通的请求参数类似，上传后的文件也是通过[参数绑定](#参数绑定)来传入到接口方法中的。

对于单文件上传，在接口方法中使用`app\core\request\UploadedFile`类型的参数来接收，参数名必须和指定的`name`一致。例如在表单中：

``` html
<form action="XXX" method="POST" enctype="multipart/form-data">
    <input type="file" name="file"/>
    <input type="button" value="上传" onclick="doUpload()" />
</form>
```

则对应的接口方法应为：

``` php
function upload(UploadedFile $file) {
    // Do something with the file.
}
```

`UploadedFile`类提供了以下几个实例方法：

| 方法名 | 返回值类型 | 含义 |
| :------------: | :------------: | :------------: |
| `getName()` |  `string` | 返回文件名（包含扩展名） |
|  `getMediaType()` | `string`  | 返回文件类型 |
|  `getSize()` | `int`  | 返回文件大小，以字节为单位 |
|  `getError()` | `int`  | 返回错误代码，没有错误则为0 |
| `saveTo(string $targetLocation)`  | `bool`  | 将上传后的文件保存到指定位置，成功返回`true` |

!> 上传的文件最终必须要通过`saveTo()`方法保存，否则本次请求结束后，临时文件将会被自动清理。

对于多文件上传，区别是需要用`app\core\request\UploadedFiles`类型的参数来接收。该类具有类似于数组的行为，其中的每个元素都是一个`UploadedFile`类对象，可以使用`count()`方法统计文件数，也可以进行遍历操作。例如：

``` php
function upload(UploadedFiles $files) {
    foreach ($files as $file) {
        // Do something with the file.
    }
}
```

此外，不管是哪种情况，**所有**上传的文件都可以通过请求对象的`getUploadedFiles()`方法获取到，其返回值即是一个`UploadedFiles`类对象。

## 文件下载

文件作为一种特殊的响应数据，需要进行一些额外的配置。

首先需要给定文件路径来获取一个指定的`app\core\io\File`类对象，如：

``` php
$file = new File("lune.png");
```

`File`类是对几个 php 内置文件操作 API 的简单封装，提供了以下几个实例方法：

| 方法名 | 返回值类型 | 含义 |
| :------------: | :------------: | :------------: |
| `getPath()` |  `string` | 返回文件路径 |
|  `getName()` | `string`  | 返回文件名 |
|  `getSize()` | `int`  | 返回文件大小，以字节为单位 |
|  `exists()` | `bool`  | 判断文件是否存在，存在则返回`true` |
| `delete()`  | `bool`  | 删除文件，成功返回`true` |
| `canRead()`  | `bool`  | 判断文件是否可读，可读则返回`true` |
| `canWrite()`  | `bool`  | 判断文件是否可写，可写则返回`true`|

有了要下载的文件之后，需要添加一些额外的响应头，然后将`File`对象设置为响应 body 即可。例如：

``` php
function download() {
    $file = new File("lune.png");
    $response = Response::builder()
            ->header("Cache-Control", "no-cache", "no-store", "must-revalidate")
            ->header("Pragma", "no-cache")
            ->header("Expires", "0")
            ->header("Content-Disposition", "attachment;filename=" . $file->getName())
            ->body($file)
            ->build();
    return $resposne;
}

```

此时在浏览器中访问响应的接口，将会触发下载指定的文件。