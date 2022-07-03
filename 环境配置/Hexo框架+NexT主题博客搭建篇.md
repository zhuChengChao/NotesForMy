# Hexo框架+NexT主题博客搭建篇

> 之前为了图个方便，将学习笔记同步到了**博客园**中，方便之处就是无需再关心样式之类的内容，不方便之处就是，每次本地更新都无法进行实时同步。
>
> 现在将要毕业步入工作，于是乎就打算重新搭建一个个人博客，将所有之前做的笔记都迁移过来，该篇博文就记录一下搭建过程，包括初始搭建，更换主题，布署得到GitPages，申请域名替换原有域名等...
>
> yssy，命令行同步上传的感觉太香了...

## 1. 环境配置

关于 Hexo + NexT 主题所需要的环境这里不再过多说明，网上教程真是一大坨...这里只说明一下本机所安装及采用的版本号：

1. `Node.js`  --- `v14.17.3`
2. `npm` --- `v6.14.13`
3. `git` --- `git version 2.23.0.windows.1`

这里需要一个 GitHub 的账号，因为后续需要把网站布署到 GitPages 上；

关于 Git 的相关内容可以见：[Git基本使用](https://xianyuchao.cn/2019/11/28/%E8%BD%AF%E4%BB%B6%E4%BD%BF%E7%94%A8/Git%E6%B5%85%E6%9E%90/)

## 2. 博客搭建

1. 创建一个需要存放你博客内容的文件夹，这里暂定为 Blog；

2. 进入新建立的 Blog 文件夹内，打开 Git 控制台；

3. 使用 npm 指令进行 Hexo 的初始安装，命令如下：`npm install -g hexo-cli`

   ```bash
   ZhuCC@DESKTOP-3AB85C8 MINGW64 ~/Desktop/test/Blog
   $ npm install -g hexo-cli
   C:\Users\ZhuCC\AppData\Roaming\npm\hexo -> C:\Users\ZhuCC\AppData\Roaming\npm\node_modules\hexo-cli\bin\hexo
   npm WARN optional SKIPPING OPTIONAL DEPENDENCY: fsevents@~2.3.2 (node_modules\hexo-cli\node_modules\chokidar\node_modules\fsevents):
   npm WARN notsup SKIPPING OPTIONAL DEPENDENCY: Unsupported platform for fsevents@2.3.2: wanted {"os":"darwin","arch":"any"} (current: {"os":"win32","arch":"x64"})
   
   + hexo-cli@4.3.0
   added 57 packages from 51 contributors in 7.89s
   ```

   > `-g`其实说明的是全局安装，从上述控制台的输出也可知，安装到了npm默认的路径中

4. 继续输入命令 `hexo init` 进行 hexo 的初始化操作，如下：

   ```bash
   ZhuCC@DESKTOP-3AB85C8 MINGW64 ~/Desktop/test/Blog
   $ hexo init
   INFO  Cloning hexo-starter https://github.com/hexojs/hexo-starter.git
   INFO  Install dependencies
   added 242 packages from 207 contributors in 22.211s
   
   15 packages are looking for funding
     run `npm fund` for details
   
   INFO  Start blogging with Hexo!
   ```

   安装完成后的目录结构如下：

   ```bash
   Blog
   	- .github
   	- scaffolds
   	- source  # 我们写的博客就放在这里
   	- themes  # 放主题文件
   	- .gitignore
   	- _config.landscape.yml
   	- _config.yml  # 站点配置文件
   	- package.json  # npm安装的版本号相关
   	- package-lock.json
   ```

5. 初始化完成后安装组件，即输入命令`npm install`，如下：

   ```bash
   ZhuCC@DESKTOP-3AB85C8 MINGW64 ~/Desktop/test/Blog
   $ npm install
   npm WARN optional SKIPPING OPTIONAL DEPENDENCY: fsevents@2.3.2 (node_modules\fsevents):
   npm WARN notsup SKIPPING OPTIONAL DEPENDENCY: Unsupported platform for fsevents@2.3.2: wanted {"os":"darwin","arch":"any"} (current: {"os":"win32","arch":"x64"})
   
   up to date in 1.304s
   
   15 packages are looking for funding
     run `npm fund` for details
   ```

6. 此时已经完成基本的搭建了，通过命令 `hexo g`生成初始页面，查看是否安装成功，如下：

   ```bash
   ZhuCC@DESKTOP-3AB85C8 MINGW64 ~/Desktop/test/Blog
   $ hexo g  # 生成页面
   INFO  Validating config
   INFO  Start processing
   INFO  Files loaded in 145 ms
   # node的一些警告 略...
   INFO  Generated: archives/index.html
   INFO  Generated: archives/2021/index.html
   INFO  Generated: archives/2021/11/index.html
   INFO  Generated: index.html
   INFO  Generated: fancybox/jquery.fancybox.min.css
   INFO  Generated: js/script.js
   INFO  Generated: 2021/11/16/hello-world/index.html
   INFO  Generated: css/style.css
   INFO  Generated: css/fonts/FontAwesome.otf
   INFO  Generated: css/fonts/fontawesome-webfont.woff
   INFO  Generated: fancybox/jquery.fancybox.min.js
   INFO  Generated: css/fonts/fontawesome-webfont.woff2
   INFO  Generated: css/fonts/fontawesome-webfont.eot
   INFO  Generated: js/jquery-3.4.1.min.js
   INFO  Generated: css/fonts/fontawesome-webfont.ttf
   INFO  Generated: css/images/banner.jpg
   INFO  Generated: css/fonts/fontawesome-webfont.svg
   INFO  17 files generated in 504 ms
   ```

7. 启动页面，通过命令：`hexo s`，如下：

   ```bash
   ZhuCC@DESKTOP-3AB85C8 MINGW64 ~/Desktop/test/Blog
   $ hexo s
   INFO  Validating config
   INFO  Start processing
   INFO  Hexo is running at http://localhost:4000 . Press Ctrl+C to stop.
   ```

   此时即可通过地址http://localhost:4000进行访问啦，访问结果如下：

![搭建成功](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251724226.PNG)

## 3. 配合 GitPages 完成布署

1. 进入 GitHub 中创建一个新的仓库，这里需要注意的是因为要用到 GitPages，**所以创建的仓库名必须是** `用户名.github.io`，只有这样后续才能顺利访问（坑死我了！！！）

2. 继续回到 Blog 文件下打开的 Git 控制台，输入命令：`npm install hexo-deployer-git --save`，安装成功后在 `Blog/node_modules` 文件夹下可以看到多了 `hexo-deployer-git`

3. 修改博客配置文件 `_config.yml`

   ```yml
   # 原内容
   # Deployment
   ## Docs: https://hexo.io/docs/one-command-deployment
   deploy:
     type: ''
     
   # 修改为：
   deploy:
     type: git
     repository: https://github.com/zhuChengChao/zhuChengChao.github.io.git
     branch: main
   ```

4. 开始进行部署了，通过命令 `hexo d` 进行部署，如下：

   ```bash
   ZhuCC@DESKTOP-3AB85C8 MINGW64 ~/Desktop/Blog (master)
   $ hexo d  # 进行部署
   INFO  Validating config
   INFO  Deploying: git
   INFO  Clearing .deploy_git folder...
   INFO  Copying files from public folder...
   INFO  Copying files from extend dirs...
   On branch master
   nothing to commit, working tree clean
   Enumerating objects: 1512, done.
   Counting objects: 100% (1512/1512), done.
   Delta compression using up to 4 threads
   Compressing objects: 100% (1137/1137), done.
   Writing objects: 100% (1512/1512), 47.49 MiB | 3.30 MiB/s, done.
   Total 1512 (delta 279), reused 0 (delta 0)
   remote: Resolving deltas: 100% (279/279), done.
   To https://github.com/zhuChengChao/zhuChengChao.github.io.git
    + 96df8cf...2d5bee2 HEAD -> main (forced update)
   Branch 'master' set up to track remote branch 'main' from 'https://github.com/zhuChengChao/zhuChengChao.github.io.git'.
   INFO  Deploy done: git  # 部署成功啦！
   ```

5. 上传成功后进入GitHub仓库的Setting，找到对应的 Pages 栏，显示如下：

   ![gitpages](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251723628.PNG)

   点击：`Your site is published at https://xianyuchao.cn/` 即可实现访问

   > 这里需要注意的是，因为我已经申请了私有域名，也完成了配置，故显示的是 `https://xianyuchao.cn/`，后续会写，正常的话地址为：`https://用户名.github.io`

至此已经成功完成了 Git Pages 的布署啦。

## 4. 使用 NexT 主题

初始建立的 hexo 博客中，有一个 themes 文件夹，内部用于放主题文件，此处我选择使用最为广泛的 NexT主题，参考文档如下：http://theme-next.iissnan.com/getting-started.html，这里做一些简答的概括：

1. 进入搭建博客的文件夹下，即：Blog，开启 Git 终端；

2. 通过命令 git clone 主题文件到 themes/next 下，如下：

   ```bash
   $ git clone https://github.com/theme-next/hexo-theme-next themes/next  # 输入以下命令
   Cloning into 'themes/next'...
   remote: Enumerating objects: 12037, done.
   remote: Total 12037 (delta 0), reused 0 (delta 0), pack-reused 12037
   Receiving objects: 100% (12037/12037), 13.05 MiB | 30.00 KiB/s, done.
   Resolving deltas: 100% (6964/6964), done.
   ```

   > 这里需要注意一点，教程中的地址已经过期啦...现在next的github地址为：`https://github.com/theme-next/hexo-theme-next`

3. 下载完成后，在 themes 文件夹下多了一个 next 文件夹，这里放的就是主题文件夹；

4. 打开 站点配置文件 `_config.yml`，修改如下：

   ```yml
   # Extensions
   ## Plugins: https://hexo.io/plugins/
   ## Themes: https://hexo.io/themes/
   theme: next  # 这里添加以下next
   ```

5. 运行命令重新布署一下：`hexo clean && hexo g && hexo s`

6. 访问本地站点：http://localhost:4000，查看到主题已经替换成功，如下：

![主题替换成功](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251724232.PNG)

## 5. 私人域名申请

目前，已经成功搭建了博客，并成功布署到了GitHub中，同时更换了主题；即现在可通过 `https://用户名.github.io` 来访问你搭建的博客了，现在申请一个自己的域名替换上述的访问路径。

1. 申请一个域名，我是在阿里云上申请了一个 .cn 的域名，一年29，如何申请不再展开；

2. 申请完成后首先ping一下原先网址的 IP 地址，如下：

   ```bash
   C:\Users\ZhuCC>ping zhuchengchao.github.io
   
   正在 Ping zhuchengchao.github.io [185.199.111.153] 具有 32 字节的数据:
   ```

3. 然后进入购买域名的控制台中，解析域名，如下：

   ![域名配置1](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251724724.PNG)

4. 添加两条记录，主机记录、记录类型、解析路线、记录值（上述ping出来的IP地址/`用户名.github.io`），如下：

   ![域名配置2](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251724426.PNG)

5. 添加 `CNAME` 文件，在 Blog 路径的 source 路径下，添加名为 CNAME 的文件（无后缀），内容即为你申请的域名，如此处我的为：`xianyuchao.cn`

   > 当然你想要在地址栏中显示输入 www 的话，那就输入 www.你申请的域名 即可

6. 执行命令 `hexo clean && hexo g && hexo d` 进行一个布署；

7. 进入GitHub所建立仓库的 Setting 中的 pages 时，可以看到在 Custom domain 栏中已经自动识别出了你的域名，如下：

   ![gitpages](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251724339.PNG)

每次布署上传之后都需要等待一会后才能访问到最新的内容哦...

## 6. 待完善（Gitee托管 + 能被检索到）

由于 GitHub 服务器在国外，因此后续打算再加上 Gitee 双托管；

然后为了博文能被检索到后续也要添加到百度 & 谷歌的搜索引擎中；

等博客迁移完成后再来实现相应功能！

## 参考

[公众号：管家小e——网站搭建](https://mp.weixin.qq.com/mp/homepage?__biz=MzU4NDcxNjQ2Ng==&hid=1&sn=debf3376e6c934da259097b1886297d7&scene=18#wechat_redirect)

[hexo史上最全搭建教程](https://blog.csdn.net/sinat_37781304/article/details/82729029)