# TortoiseGit使用

> 其实作为一名学生，还未接触过企业级开发项目，基本都是一个人在本地敲代码，对于项目管理工具使用的并不多，最常用的命令也就是`git clone`了，hhh；
>
> 前些日子了解了一下SVN，发现对于SVN有一款很好用的第三方管理客户端，叫TortoiseSVN；
>
> 然后顺带还发现了他的兄弟TortoiseGit，使用起来虽然没有黑乎乎的命令窗口来的酷炫，但...架不住其界面的友好，故此对其进行了略微的了解。
>
> > 这名字起的也很有趣，Tortoise=乌龟，Git=饭桶，加起来就是饭桶龟？

## 1. 下载安装

### 1.1 下载

进入如下网址：https://tortoisegit.org/download/

![下载海龟](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206252047567.JPG)

根据电脑的版本进行下载

> 正常下载速度有点慢，怎么办？小飞机开起来！

在可以在下方顺带下载一下语言包

![下载语言包](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206252047440.JPG)

### 1.2 安装

#### Tortoise安装

双击下载完的安装包开始安装`TortoiseGit-2.10.0.0-64bit.msi`

基本按照默认配置进行即可，其中有一个选项我修改了，如下

![配置](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206252047657.png)

> 由于我之前安装过一遍2.4版本的，卸载后再装最新的该配置界面没有再出现，应该还是原先的配方

安装完成后，可以选择进入向导配置，包括语言的设置，与你安装的`Git.exe`路径的确认

![向导](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206252047872.JPG)

> 其中还由于我的Git版本过老，还顺带额外手动升级了一下GIt

#### 语言支持

双击安装完的语言包`TortoiseGit-LanguagePack-2.10.0.0-64bit-zh_CN.msi`，下一步即可；

安装完成后，在空白处右击，进入TortoiseGit的Settings中进行语言的选择

![语言设置](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206252047262.JPG)

选择中文即可：

![语言设置2](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206252047652.JPG)

## 2. 基本使用

由于是图形界面，类似于**创建仓库**，**添加文件到暂存区**，**提交代码**，**还原**等这些基础操作**直接在需要操作的文件或是文件夹上右击进行相应的选择即可一目了然**，故不再展开说明，只是说明一些特殊的情况。

### 2.1 提交时忽略文件

在工程项目中，有些临时文件是不需要提交的，可采用对其进行忽略操作；

将文件添加到暂存区后，可选择

![忽略文件1](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206252047998.JPG)

后就根据需求选择忽略文件的相应配置即可

![忽略文件2](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206252047397.JPG)

之后会自动生成`.gitignore`文件，内有选择忽略文件的表达式

### 2.2 差异比较

乌龟的差异比较是真的很人性化，直接在修改过的文件上右击选择比较差异，即可看到和版本库文件有差异的部分

![差异比较](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206252047923.JPG)

当**存在冲突**时，就需要用到改功能进行差异的比较，在解决冲突后，**手动标记冲突解决后**才可以进行代码的提交操作。

### 2.3 查看版本库信息

在仓库中，鼠标右击选择`版本库浏览器`，即可查看当前版本库中的文件信息

![版本库浏览器](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206252047477.JPG)

### 2.4 查看修改历史

通过右键可选择`显示日志`，可查看提交的日志信息，非常的方便。

### 2.5 分支管理

使用乌龟，能很方便的创建分支，切换分支，合并分支，解决冲突等等；

对于分支，目前的我对其的使用为0，也不进行阐述。

## 3 远程仓库

将本地仓库和远程仓库进行对接当然也是必要的一步，而关于在GitHub上创建仓库等步骤略；

此时我已经在GitHub上创建了一个仓库，用于与本地仓库的对接操作；

关于如何配置SSH，可见如下[Git基本使用](https://xianyuchao.cn/2019/11/28/%E8%BD%AF%E4%BB%B6%E4%BD%BF%E7%94%A8/Git%E6%B5%85%E6%9E%90/)

### 3.1 同步到远程仓库

#### 通过SSH

1. 在本地仓库中右击选择`Git同步`，后进行设置

![推送配置](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206252047920.JPG)

2. 首先确保SSH客户端的位置正确

![推送配置](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206252047013.JPG)

3. 进行仓库地址，私钥配置

![推送配置](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206252047507.JPG)

![推送配置](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206252047760.JPG)

4. 选择刚刚的配置，进行推送

![推送配置](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206252047287.JPG)

> 这里的推送和拉取都是用于和GitHub上的仓库进行同步处理

#### 通过HTTPS

在进行`Git-远端`配置时，`URL：`选项中，复制如`https`的即可

![推送配置](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206252047977.JPG)

该方式需要填入GitHub的账号和密码

![推送配置](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206252047605.JPG)

### 3.2 从远程仓库拉取

右击选择Git克隆即可

![拉取1](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206252047001.JPG)

## 4. 在`IntelliJ IDEA`中配置Git

日常使用应该也GIt应该也是会配合各种`IDEA`进行git的各个操作，如今已经转向java后端，故用`IntelliJ IDEA`对git进行相应的配置

**配置流程**

1. 打开`IEDA`在`Settings`中配置`git.exe`的路径，可通过test进行测试是否正确

![IDEA1](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206252047501.JPG)

2. 将建立的工程添加至git

![IDEA1](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206252047990.JPG)

选择工程所在的上级目录，较合理

![IDEA3](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206252047342.JPG)

3. 添加完成后，可进行工程的commit，update等操作

![IDEA](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206252047138.JPG)

> 在commit时，可以选择需要忽略的文件
>
> ![IDEA](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206252047492.JPG)

4. 将工程推送到远程仓库中，其中URL:**只能选择Https协议的才能推送成功**，配置完成后，点击push将仓库推送到远程仓库中

![IDEA](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206252047844.JPG)

5. 从远程仓库中克隆，在IDEA的首页可选择从版本库中克隆代码，如下

![IDEA](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206252047213.JPG)

![IDEA](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206252047234.JPG)



## **参**考

https://jingyan.baidu.com/article/f3ad7d0f37a75d09c2345b6f.html

https://www.cnblogs.com/anayigeren/p/10177027.html