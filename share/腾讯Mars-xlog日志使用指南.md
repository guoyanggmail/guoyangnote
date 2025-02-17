## Android本地So库编译
> 需要准备`JDK`<font style="color:rgb(77, 77, 77);">、</font>`<font style="color:rgb(77, 77, 77);">NDK</font>`<font style="color:rgb(77, 77, 77);">、</font>`<font style="color:rgb(77, 77, 77);">CMake</font>`环境
>

### 下载[mars](https://github.com/Tencent/mars)<font style="color:rgb(77, 77, 77);">源码</font>
### 使用Python2进行编译
找到`mars`项目`mars`目录下的`build_android.py`，使用`python2`进行编译

```python
python2 build_android.py
```

<font style="color:rgb(36, 41, 47);">执行命令后，会让选择:</font>

```python
Enter menu:
1. Clean && build mars.
2. Build incrementally mars.
3. Clean && build xlog.
4. Exit
```

我们这里选择3，进行`xlog`的编译。

<font style="color:rgb(36, 41, 47);">编译成功后将</font>`<font style="color:rgb(36, 41, 47);">com.tencent.mars.xlog</font>`<font style="color:rgb(36, 41, 47);">下面的</font>`<font style="color:rgb(36, 41, 47);">Java</font>`<font style="color:rgb(36, 41, 47);">文件和</font>`<font style="color:rgb(36, 41, 47);">lib</font>`<font style="color:rgb(36, 41, 47);">下面的</font>`<font style="color:rgb(36, 41, 47);">libc++_shared.so、libmarsxlog.so</font>`<font style="color:rgb(36, 41, 47);">导入到自己的项目中，然后就可以进行愉快的编码了。</font>

## <font style="color:rgb(36, 41, 47);">Xlog 加密使用指引</font>
### 环境准备
> 需要准备`python2`环境
>

### <font style="color:rgb(77, 77, 77);">安装</font>`<font style="color:rgb(77, 77, 77);">pyelliptic1.5.10</font>`
[https://github.com/mfranciszkiewicz/pyelliptic/archive/1.5.10.tar.gz#egg=pyelliptic](https://github.com/mfranciszkiewicz/pyelliptic/archive/1.5.10.tar.gz#egg=pyelliptic)

<font style="color:rgb(77, 77, 77);">文档中写的是</font>`<font style="color:rgb(77, 77, 77);">1.5.7</font>`<font style="color:rgb(77, 77, 77);">，如果在新版本的</font>`<font style="color:rgb(77, 77, 77);">macos</font>`<font style="color:rgb(77, 77, 77);">下不可用，已经有人提交了</font>`<font style="color:rgb(77, 77, 77);">Issues</font>`<font style="color:rgb(77, 77, 77);">，并给出了解决方案</font>  
[Exception: Couldn't load OpenSSL lib , 升级到MACOS 11.5.1后出现 · Issue #969 · Tencent/mars (github.com)](https://github.com/Tencent/mars/issues/969)

<font style="color:rgb(77, 77, 77);">将</font>`<font style="color:rgb(77, 77, 77);">pyelliptic1.5.10</font>`<font style="color:rgb(77, 77, 77);">进行解压后，修改</font>`<font style="color:rgb(77, 77, 77);">pyelliptic-1.5.10/pyelliptic.py</font>`<font style="color:rgb(77, 77, 77);">文件中的内容</font>

```python
def find_crypto_lib():
    if sys.platform != 'win32':
        # 注释掉下面路径,写绝对路径
        # return ctypes.util.find_library('crypto')
        return '/usr/lib/libcrypto.dylib'
```

<font style="color:rgb(77, 77, 77);">在</font>`<font style="color:rgb(77, 77, 77);">Python3</font>`<font style="color:rgb(77, 77, 77);">的环境下进行安装，到</font>`<font style="color:rgb(77, 77, 77);">pyelliptic-1.5.10</font>`<font style="color:rgb(77, 77, 77);">目录 执行</font>

```shell
python3 setup.py install
```

### <font style="color:rgb(77, 77, 77);">生成公私钥</font>
<font style="color:rgb(77, 77, 77);">到</font>`<font style="color:rgb(77, 77, 77);">Mars</font>`<font style="color:rgb(77, 77, 77);">目录下</font>`<font style="color:rgb(77, 77, 77);">mars-master/mars/log/crypt/gen_key.py</font>`

<font style="color:rgb(77, 77, 77);">执行</font>`<font style="color:rgb(77, 77, 77);">gen_key.py</font>`<font style="color:rgb(77, 77, 77);">文件 </font>

```shell
python gen_key.py

save private key：#私钥
xxxxxxxxxxxx

appender_open's parameter:#公钥
xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

  
<font style="color:rgb(77, 77, 77);">将私钥和公钥配置到</font>`<font style="color:rgb(77, 77, 77);">decode_mars_crypt_log_file.py</font>`<font style="color:rgb(77, 77, 77);">中</font>

```python
PRIV_KEY = "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
PUB_KEY = "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
```

### xlog文件解析
切换到`<font style="color:rgb(77, 77, 77);">decode_mars_crypt_log_file.py</font>`<font style="color:rgb(77, 77, 77);">目录，使用</font>`<font style="color:rgb(77, 77, 77);">python2</font>`<font style="color:rgb(77, 77, 77);">命令进行解密。</font>

```python
python2 decode_mars_crypt_log_file.py xxx.xlog
```

最终会在该目录下生成`xxx.xlog.log`文件

## Android端封装Xlog使用
### 引入依赖
```groovy
dependencies {
    implementation 'com.github.GuoYangGit:XlogUtils:1.0.0'
}
```

### NDK添加支持so库类型
```groovy
ndk {
    // 选择要添加的对应 cpu 类型的 .so 库，多个abi以“,”分隔。
    abiFilters 'armeabi-v7a', 'arm64-v8a'
    // 可指定的值为 'armeabi-v7a', 'arm64-v8a', 'armeabi', 'x86', 'x86_64'，
}
```

### 初始化
```groovy
// 初始化日志打印，Application进行初始化
// 日志保存路径
val logPath = ""
// 公钥
val xLogPubKey = ""
LogHelper.init(application, BuildConfig.DEBUG, logPath) {
    this.pubKey = xLogPubKey
}
```

### 使用
```groovy
xLogV("测试一下")
xLogD("测试一下")
xLogI("测试一下")
xLogW("测试一下")
xLogE("测试一下")
xLogF("测试一下")
```

