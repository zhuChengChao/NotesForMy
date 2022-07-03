# Notepad直连Linux

> 号称编辑器之神的Vim对于只会用几个基础操作的本人而言，在编辑一些大型文本有那么些力不从心；
>
> 平时都是通过Xftp拖到本地，修改完后再覆盖回去；
>
> 无意间发现Notepad++可以通过插件直接远程连接Linux，觉得很实用，记录一下配置过程:-)

## 下载插件

* 下载地址，下载插件：https://sourceforge.net/projects/nppftp/
* 下载解压后，将其中的`NppFTP.dll`文件放置到`Notepad++`安装路径下的`plugins`文件夹下

## 配置

* 打开`Notepad++`，进行如下选择

![插件选择](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206252046224.JPG)

* 进行IP配置
  * 首先选择如下：![插件配置1](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206252046947.JPG)
  * 再进行IP地址与端口的配置![插件配置2](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206252046238.JPG)
* 测试连接，首次连接会提示是否对其进行授权信息

![连接成功](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206252046874.JPG)

* 通过绿色的刷新按钮可以对文件列表进行更新操作

## 使用

连接成功后，就可以在熟悉的Notepad++中对远程的Linux文件进行修改了：

* 找到需要修改的文件，双击，即自动下载到本地后，用Notepad++打开；

* 编辑完成后，保存即可自动上传修改。

## 参考

https://blog.csdn.net/qq_22317977/article/details/100038692

https://jingyan.baidu.com/article/d8072ac4791d0cec94cefd7c.html