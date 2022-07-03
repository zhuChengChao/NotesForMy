# 服务器安装Anaconda&指定版本TensorFlow

> 由于疫情影响，原先使用的服务器已断电，故重选了一台服务器对环境重选进行搭建，正好补上这篇博文。

## 01. 下载`Anaconda3`

1. 进入清华开源软件镜像站，其网址如下：https://mirrors.tuna.tsinghua.edu.cn/anaconda/archive/

2. 下载相应的版本信息：

![下载](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251729271.JPG)

> 此处可通过命令 `uname -a`查看系统的版本等信息：
>
> ```bash
> zcc@new-gpu:/home$ uname -a
> Linux new-gpu 4.4.0-78-generic #99-Ubuntu SMP Thu Apr 27 15:29:09 UTC 2017 x86_64 x86_64 x86_64 GNU/Linux
> ```

3. 通过`Xftp`将下载好的软件上传到服务器上

## 02. 安装

1. 定位到`Anaconda3-2019.10-Linux-x86_64.sh`所在路径

2. 在命令窗口中输入：`bash Anaconda3-2019.10-Linux-x86_64.sh `进行安装

   * 安装过程中根据相应提示，输入enter/yes等确认操作

3. 安装过程很快速，安装完成后，提示：`Thank you for installing Anaconda3!`

4. **环境激活**：输入命令`source ~/.bashrc `，激活环境后，默认进入`base`环境下

   * 如：`(base) zcc@new-gpu:~$`

   > 若无法通过该命令激活环境，则采取下列操作：
   >
   > 1. `vim ~/.bashrc`
   > 2. 在最后一行加上：`export PATH=$PATH:/你的路径/anaconda3/bin`
   > 3. 保存后再运行`source ~/.bashrc`

5. 验证是否安装成功：输入命令`conda --version`，查看是否能查看安装版本

   ```bash
   (base) zcc@new-gpu:~$ conda --version
   conda 4.7.12
   ```


> 上述版本的`Anaconda`安装后，默认安装了`python3`

6. 镜像源的修改：[各种镜像源的更换](https://www.cnblogs.com/zhuchengchao/p/11622065.html)

## 03. 安装指定版本`TensorFlow`

### 版本解决

考虑到项目中需要使用的`TensorFlow`与`python`版本和服务器端现有的`CUDA``版本，故重新创建一个新的conda`环境进行各个内容的安装，此处我需要安装的版本为：`python3.6`,`tensorflow-gpu 1.12.0`

1. 查看服务器端的`cuda`与`cudnn`版本， 通过输入命令：`cat /usr/local/cuda/version.txt`与`cat /usr/local/cuda/include/cudnn.h | grep CUDNN_MAJOR -A 2`，如下：

```bash
(base) zcc@new-gpu:~$ cat /usr/local/cuda/version.txt
CUDA Version 8.0.61
(base) zcc@new-gpu:~$ cat /usr/local/cuda/include/cudnn.h | grep CUDNN_MAJOR -A 2
#define CUDNN_MAJOR      5
#define CUDNN_MINOR      1
#define CUDNN_PATCHLEVEL 10
--
#define CUDNN_VERSION    (CUDNN_MAJOR * 1000 + CUDNN_MINOR * 100 + CUDNN_PATCHLEVEL)

#include "driver_types.h"

```

2. 在以下网址查看需要安装的`TensorFlow`是否匹配`cuda`，`cudnn`和`Python`版本，如下：https://tensorflow.google.cn/install/source

![版本匹配](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251729383.JPG)

3. 非常不幸的是，这台服务器上的`CUDA`版本和我的需要安装的`TensorFlow`版本并不匹配:-(

> 该不匹配的情况不会影响你后续的安装，因此先进行后续的安装，完成后再来解决cuda版本匹配问题

#### 修改`CUDN`和`CUDNN`版本

**再次说明**：因为之前我遇到过一次该问题，也的确是会有版本冲突的问题，去了英伟达的官方网下载了相应的`cudnn`，但是这次虽然版本也没有匹配上，但是程序居然跑通了，弄的我一脸懵逼。

故在此只贴出上次解决是参考的博客：https://blog.csdn.net/weixin_42567692/article/details/80844696

亲测有效

> 后来在路径`usr/local`一查，原来服务器下CUDA8.0和CUDA9.0的版本都有

### 创建新`Conda`环境

1. 创建一个新的`conda`环境，并指定`Python`的版本号未3.6

```bash
# conda create -n 环境名 python=制定版本号
conda create -n tensorflow1.12 python=3.6

# ....创建环境，安装各种包....

# To activate this environment, use
#
#     $ conda activate tensorflow1.12
#
# To deactivate an active environment, use
#
#     $ conda deactivate
```

> 由于并没有进行镜像的更换，可以在命令后方制定临时的镜像源

2. 安装成后，通过命令`conda info -e`查看目前现有环境，通过命令`conda activate tensorflow1.12`切换到指定环境

   > 当出现如下错误时：
   >
   > ```bash
   > CommandNotFoundError: Your shell has not been properly configured to use 'conda activate'.
   > To initialize your shell, run
   > 
   >     $ conda init <SHELL_NAME>
   > ```
   >
   > 解决：
   >
   > 1. 输入命令`source activate`进入base环境
   > 2. 再输入`conda activate 环境名`即可
   > 3. 退出环境命令：`source deactivate`

3. 输入`python`，查看是`python`是否是你指定安装的`Python`版本

### 安装`tensorflow`

#### 方式一：直接安装

通过pip工具对`tensorflow-gpu`进行安装

```bash
# 指定临时镜像源对指定版本的tensorflow进行安装
pip install tensorflow-gpu==1.12.0 -i https://pypi.tuna.tsinghua.edu.cn/simple/

# ...漫长的安装tensorflow+依赖包...
```

> 若出现下载链接超时`pip._vendor.urllib3.exceptions.ReadTimeoutError: HTTPSConnectionPool(host='pypi.tuna.tsinghua.edu.cn', port=443): Read timed out.`的异常
>
> 可设定超时时间：`pip install --default-timeout=1000 tensorflow-gpu==....`
>
> 若还是下载缓慢，比如我这次安装时就遇见了，那就采用下载方式二：

#### 方式二：本地下载好再装

1. 根据`pip`在下载时提示的网站地址，自行下载

例如我下载的`tensorflow1,12-gpu`的地址为：` https://pypi.tuna.tsinghua.edu.cn/packages/55/7e/bec4d62e9dc95e828922c6cec38acd9461af8abe749f7c9def25ec4b2fdb/tensorflow_gpu-1.12.0-cp36-cp36m-manylinux1_x86_64.whl`

2. 下载完成后，将文件`tensorflow_gpu-1.12.0-cp36-cp36m-manylinux1_x86_64.whl`移动到服务器端

3. 通过命令`pip install 包路径+包名`指定刚刚上传的包路径进行安装

> 此方法安装也会去网上下载所依赖的包，因此最好也指定下路径，最好还指定下超时时间
>
> 我的完整命令如下：
>
> `pip install --default-time=1000 tensorflow_gpu-1.12.0-cp36-cp36m-manylinux1_x86_64.whl -i https://pypi.tuna.tsinghua.edu.cn/simple/`

#### 验证

经过漫长的安装：控制端提示安装成功，验证如下：

```bash
(tensorflow1.12) zcc@new-gpu:~$ python
Python 3.6.10 |Anaconda, Inc.| (default, Jan  7 2020, 21:14:29) 
[GCC 7.3.0] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> import tensorflow as tf
>>> tf.__version__
'1.12.0'
```

可见，我们`tensorflow1.12`安装成功

> 在导入包的时候会报一串`FutureWarning`警告，这是由于numpy的版本问题，不影响使用，网上有很多解决方法，此处就不贴解决方案了。

## 参考

https://blog.csdn.net/weixin_41519463/article/details/89373643

https://blog.csdn.net/qq_33221533/article/details/100150534

https://blog.csdn.net/weixin_38705903/article/details/86533863

> 就在我刚刚装完环境后，居然告诉我电力恢复，我真是晕了:-(