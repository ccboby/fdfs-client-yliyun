# Nodejs Client for FastDFS

项目fork自 chenboxiang/fdfs-client
https://github.com/weilaihui/fdfs_client

原作者停止维护，自己做了一点小更改（如修复无后缀名上传文件出错的bug）


# 安装
```shell
npm install fdfs-client-yliyun
```

# 使用（使用方法完全摘自 https://github.com/weilaihui/fdfs_client）
```javascript
var fdfs = new FdfsClient({
    // tracker servers
    trackers: [
        {
            host: 'tracker.fastdfs.com',
            port: 22122
        }
    ],
    // 默认超时时间10s
    timeout: 10000,
    // 默认后缀
    // 当获取不到文件后缀时使用
    defaultExt: 'txt',
    // charset默认utf8
    charset: 'utf8'
})
```
以上是一些基本配置，你还可以自定义你的日志输出工具，默认是使用console
例如你要使用[debug](https://github.com/visionmedia/debug)作为你的日志输出工具，你可以这么做：
```javascript
var debug = require('debug')('fdfs')
var fdfs = new FdfsClient({
    // tracker servers
    trackers: [
        {
            host: 'tracker.fastdfs.com',
            port: 22122
        }
    ],
    logger: {
        log: debug
    }
})
```

### 上传文件
注：以下fileId为group + '/' + filename，以下的所有操作使用的fileId都是一样

通过本地文件名上传
```javascript
fdfs.upload('test.gif', function(err, fileId) {
    // fileId 为 group + '/' + filename
})
```

上传Buffer
```javascript
var fs = require('fs')
// 注意此处的buffer获取方式只为演示功能，实际不会这么去构建buffer
var buffer = fs.readFileSync('test.gif')
fdfs.upload(buffer, function(err, fileId) {
    
})
```

ReadableStream
```javascript
var fs = require('fs')
var rs = fs.createReadStream('test.gif')
fdfs.upload(rs, function(err, fileId) {
    
})
```

其他一些options，作为第2个参数传入
```js
fdfs.upload('test.gif', {
    // 指定文件存储的group，不指定则由tracker server分配
    group: 'group1',
    // file bytes, file参数为ReadableStream时必须指定
    size: 1024,
    // 上传文件的后缀，不指定则获取file参数的后缀，不含(.)
    ext: 'jpg'
    
}, function(err, fileId) {

})
```

### 下载文件

下载到本地
```js
fdfs.download(fileId, 'test_download.gif', function(err) {
    
})
```

下载到WritableStream
```js
var fs = require('fs')
var ws = fs.createWritableStream('test_download.gif')
fdfs.download(fileId, ws, function(err) {

})

```

下载文件片段
```js
fdfs.download(fileId, {
    target: 'test_download.part',
    offset: 5,
    bytes: 5
}, function(err) {

})
```

### 删除文件

```js
fdfs.del(fileId, function(err) {

})
```

### 获取文件信息

```js
fdfs.getFileInfo(fileId, function(err, fileInfo) {
    // fileInfo有4个属性
    // {
    //   // 文件大小
    //   size:
    //   // 文件创建的时间戳，单位为秒
    //   timestamp:
    //   // 校验和
    //   crc32:
    //   // 最初上传到的storage server的ip
    //   addr:
    // }
    console.log(fileInfo)
})
```

### 文件的Meta Data

设置Meta Data, 我只贴出来文件签名信息吧，flag字段如果不传则默认是O
```js
/**
 * @param fileId
 * @param metaData  {key1: value1, key2: value2}
 * @param flag 'O' for overwrite all old metadata (default)
                'M' for merge, insert when the meta item not exist, otherwise update it
 * @param callback
 */
fdfs.setMetaData(fileId, metaData, flag, callback)
```

获取Meta Data
```js
fdfs.getMetaData(fileId, function(err, metaData) {
    console.log(metaData)
})
```


### 错误处理

当无tracker可用时会触发error事件
```javascript
fdfs.on('error', function(err) {
    // 在这里处理错误
})
```

# 测试
测试时需要用到co，所以需要node版本0.11+。测试前请确保配置好FastDFS的Server地址，为tracker.fastdfs.com:22122，或者修改test/fdfs_test.js中的client配置，然后执行如下命令：
```shell
make test
```

# 帮助
有任何问题请提交到Github Issue里

# 授权协议
MIT