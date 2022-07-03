# Hexo博客内容同步Gitee私人仓库

> 博客目前已经从博客园迁移了，通过Hexo进行了部署，但存在一个多设备同步的问题，即：台式机和笔记本都需要一个可以对博文进行编辑的环境，即Hexo的配置文件、主题文件都需要能进行同步。
>
> 之前的话可以很方便[利用实验室的服务器搭建一个私人仓库](https://xianyuchao.cn/2020/10/18/%E7%8E%AF%E5%A2%83%E9%85%8D%E7%BD%AE/%E6%90%AD%E5%BB%BA%E4%B8%80%E4%B8%AA%E6%9C%AC%E5%9C%B0%E7%A7%81%E4%BA%BAGit%E4%BB%93%E5%BA%93/)进行同步，由于马上要毕业了...因此服务器同步的方案就使用不了咯，故就想着利用Gitee搭建一个仅用于同步的私人仓库。
>
> > 这里推荐使用 TortoiseGit，会方便很多

### 步骤1：创建仓库

> 第一次搭建的时候没记录本文是后续记录的，因而此处仓库名随意起了blog-test，推荐的话是和你Hexo布署的文件名一致最好，当然不一样也可以反正在clone的时候可以指定文件夹

在 Gitee 上建立仓库，最好的话还是设置为私有，如下（就是一个正常的仓库就行）

![新建仓库](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206252046860.PNG)

### 步骤2：Gitee仓库同步本地博客

> 仅靠步骤2是不行的，要结合步骤3！！！

首先：进入布署博客的根目录中，可以先删除了主题文件的 `.git`，即进入`/themes/next/`后，显示隐藏的文件，删除名为`.git`的文件，如下：

![删除next的git](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206252046783.PNG)

> 因为这时候主题内容以及被你修改掉了，不需要一起同步

回到根目录，使用Git Bash 进行 Git 仓库初始化，如下：

```bash
ZhuCC@LAPTOP-F6OAPAAB MINGW64 ~/Desktop/test/Blog
$ git init
Initialized empty Git repository in C:/Users/ZhuCC/Desktop/test/Blog/.git/
```

然后：先运行一下 `hexo clean` 后，将所有的内容添加到工作区并提交，如下：

```bash
# 先清空一下临时文件啥的
ZhuCC@LAPTOP-F6OAPAAB MINGW64 ~/Desktop/test/Blog (master)
$ hexo clean
INFO  Validating config
INFO  Deleted database.
INFO  Deleted public folder.

# 添加内容到工作区
ZhuCC@LAPTOP-F6OAPAAB MINGW64 ~/Desktop/test/Blog (master)
$ git add *
The following paths are ignored by one of your .gitignore files:
# ...需要被同步的文件...

# 提交一下
ZhuCC@LAPTOP-F6OAPAAB MINGW64 ~/Desktop/test/Blog (master)
$ git commit -m "首次提交"
[master (root-commit) a37a079] 首次提交
 323 files changed, 23004 insertions(+)
# ....
```

其次：和刚刚创建的Gitee仓库连接上后推送：

```bash
# 关联远程仓库
ZhuCC@LAPTOP-F6OAPAAB MINGW64 ~/Desktop/test/Blog (master)
$ git remote add origin https://gitee.com/zhuChengChao/blog-test.git

# 推送
ZhuCC@LAPTOP-F6OAPAAB MINGW64 ~/Desktop/test/Blog (master)
$ git push -u origin master
Enumerating objects: 393, done.
Counting objects: 100% (393/393), done.
Delta compression using up to 12 threads
Compressing objects: 100% (375/375), done.
Writing objects: 100% (393/393), 1.16 MiB | 1.45 MiB/s, done.
Total 393 (delta 22), reused 0 (delta 0)
remote: Resolving deltas: 100% (22/22), done.
remote: Powered by GITEE.COM [GNK-6.2]
To https://gitee.com/zhuChengChao/blog-test.git
 * [new branch]      master -> master
Branch 'master' set up to track remote branch 'master' from 'origin'.
```

首次推送成功，查看Gitee仓库可见如下：

![首次提交](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206252046839.PNG)

### 步骤3：忽略文件的同步(old)

> 推荐用 TortoiseGit，不会出现漏掉文件的情况，第一次就是因为漏掉了，通过TortoiseGit发现后出现了三次提交:angry:

由于存在文件`.gitignore`，因此关于hexo框架的支持被直接忽略掉了，如下当我clone仓库后运行命令会是无法被知道的：

```bash
ZhuCC@DESKTOP-3AB85C8 MINGW64 ~/Desktop/test/blog-test (master)
$ hexo clean
ERROR Cannot find module 'hexo' from 'C:\Users\ZhuCC\Desktop\test\blog-test'
ERROR Local hexo loading failed in ~\Desktop\test\blog-test
ERROR Try running: 'rm -rf node_modules && npm install --force'
```

那怎么把hexo相关的内容全部都给同步到Gitee上呢？

那就直接把 `.gitignore` 中的 `/node_modules/`内容给删了，让 `/node_modules/` 中的内容也同时被同步，删除后运行命令如下：

> 记得之后把删除的 `/node_modules/`给添加回来，这是考虑到每次通过 npm 卸载相应的依赖时，其实是不会卸载 `/node_modules/` 中的内容的！这就会导致`/node_modules/`中的内容爆炸；
>
> **这里再提供一个命令：`npm prune`，自动从node_modules中删除未使用的软件包。**

```bash
ZhuCC@LAPTOP-F6OAPAAB MINGW64 ~/Desktop/test/Blog (master)
$ git add *  # 吧/node_modules/内容添加当工作区
# 输出内容略...
```

其实还有几个文件是没有被管理的，有如下：

* 文件 `/.github/dependabot.yml` 

* 文件`/.gitignore`

* 如下图所示4个文件：

  ![未被添加的文件](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206252046821.PNG)

对于上述文件的话就需要手动提添加了，如下：

> **或者直接过通过TortoiseGit添加**，直接在根目录右键通过TortoiseGit的添加功能勾选上包括忽略文件，如上图红框框，就不需要下面的步骤了！直接一次性把所有已忽略的文件都添加上去（连上面的`.gitignore`都无需再去修改了）

```bash
ZhuCC@LAPTOP-F6OAPAAB MINGW64 ~/Desktop/test/Blog (master)
$ git add .gitignore .git
warning: LF will be replaced by CRLF in .gitignore.
The file will have its original line endings in your working directory

ZhuCC@LAPTOP-F6OAPAAB MINGW64 ~/Desktop/test/Blog (master)
$ git add .github/*
warning: LF will be replaced by CRLF in .github/dependabot.yml.
The file will have its original line endings in your working directory

# 这里列出两个文件，后面我是直接用TortoiseGit添加的...（因为第一次漏了）
```

提交一下然后推送，如下：

```bash
ZhuCC@LAPTOP-F6OAPAAB MINGW64 ~/Desktop/test/Blog (master)
$ git commit -m "提交配置文件"
# ...输出略...
ZhuCC@LAPTOP-F6OAPAAB MINGW64 ~/Desktop/test/Blog (master)
ZhuCC@LAPTOP-F6OAPAAB MINGW64 ~/Desktop/test/Blog (master)
$ git push -u origin master
Enumerating objects: 5205, done.
Counting objects: 100% (5205/5205), done.
Delta compression using up to 12 threads
Compressing objects: 100% (4985/4985), done.
Writing objects: 100% (5204/5204), 7.28 MiB | 498.00 KiB/s, done.
Total 5204 (delta 1198), reused 0 (delta 0)
remote: Resolving deltas: 100% (1198/1198), completed with 1 local object.
remote: Powered by GITEE.COM [GNK-6.2]
To https://gitee.com/zhuChengChao/blog-test.git
   a37a079..3410dfb  master -> master
Branch 'master' set up to track remote branch 'master' from 'origin'.
```

同步成功后，查看此时的Gitee仓库，如下：

![二次提交](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206252046984.PNG)

### 步骤3：忽略文件的同步(new)

上面搞的有点复杂了，其实就是在需要上传的文件夹处直接用TortoiseGit添加文件功能，并选上**包括忽略文件**就可以了，如下：

![未被添加的文件](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206252046417.PNG)

然后再提交一下就可以了！

### 步骤4：测试

此时已经把搭建博客的所有内容都传到了Gitee上了，那么测试一下，在另一台电脑上，进行测试；

> 当然这个电脑是也是要有相应环境的，即可以成功重头用Hexo框架搭建博客的

首先从Gitee仓库中进行内容的拉取，如下：

```bash
ZhuCC@DESKTOP-3AB85C8 MINGW64 ~/Desktop/test
$ git clone https://gitee.com/zhuChengChao/blog-test.git ./Blog/
Cloning into './Blog'...
remote: Enumerating objects: 5615, done.
remote: Counting objects: 100% (5615/5615), done.
remote: Compressing objects: 100% (4158/4158), done.
remote: Total 5615 (delta 1236), reused 5581 (delta 1220), pack-reused 0
Receiving objects: 100% (5615/5615), 8.46 MiB | 1.78 MiB/s, done.
Resolving deltas: 100% (1236/1236), done.
Updating files: 100% (5804/5804), done.

```

之后进入Blog下，测试是否能正常的部署提交：

```bash
ZhuCC@DESKTOP-3AB85C8 MINGW64 ~/Desktop/test
$ cd ./Blog/
(base)
ZhuCC@DESKTOP-3AB85C8 MINGW64 ~/Desktop/test/Blog (master)
$ hexo clean
INFO  Validating config
(base)
ZhuCC@DESKTOP-3AB85C8 MINGW64 ~/Desktop/test/Blog (master)
$ hexo g && hexo s
INFO  Validating config
INFO  Start processing
# 忽略输出
INFO  55 files generated in 1.1 s
INFO  Validating config
INFO  Start processing
INFO  ---- START COPYING TAG CLOUD FILES ----
INFO  ---- END COPYING TAG CLOUD FILES ----
INFO  Hexo is running at http://localhost:4000 . Press Ctrl+C to stop.
```

OK，大功告成！这回你博客的所有的内容都同步到了云端，可以在不同的电脑上都进行操作了。

> 补：以下就是最近的一些真实博客最近的同步内容
>
> ![最近提交](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206252046266.PNG)