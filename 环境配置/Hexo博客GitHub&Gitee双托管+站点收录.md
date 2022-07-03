# Hexo博客GitHub&Gitee双托管+站点收录

> 之前的博客是托管到了GitHub的Pages服务上，但考虑到网速无法被百度收录等原因，因此想把博客同时托管到Gitee；这样的话布署到GitHub上的内容被Google收录，Gitee上的内容被百度收录。

## 1. GitHub&Gitee双托管

#### GitHub Pages托管

该部分内容见：[Hexo框架+NexT主题博客搭建篇：3. 配合 GitPages 完成布署](https://xianyuchao.cn/2021/11/14/%E7%8E%AF%E5%A2%83%E9%85%8D%E7%BD%AE/Hexo%E6%A1%86%E6%9E%B6+NexT%E4%B8%BB%E9%A2%98%E5%8D%9A%E5%AE%A2%E6%90%AD%E5%BB%BA%E7%AF%87/)

#### Gitee Page托管

在GitHub Pages托管的基础上，在把博客的内容托管到 Gitee 上，操作如下：

**步骤1：**在Gitee中创建仓库，**这里仓库名是需要是Gitee的用户名**

**步骤2**：修改Hexo搭建的博客中的配置文件`_config.yml`，使得每次推送时实现GitHub与Gitee双推送，如下：

```yml
# Deployment
## Docs: https://hexo.io/docs/one-command-deployment
deploy:
  type: git
  repository: 
    github: https://github.com/用户名/用户名.github.io.git,main
    gitee: https://gitee.com/用户名/用户名.git,master
```

> 因为GitHub是用main做主分支，而Gitee中还是用master做主分支的，需要注意区分。

**步骤3**：为创建的仓库申请Gitee Pages服务，如下图：（需要一个工作日时间）

![申请GiteePages](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251723010.PNG)

**步骤4**：申请成功后需要启用Gitee Pages服务，如下图：

![启用GiteePages服务](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251723865.PNG)

**步骤5：**推送`hexo g && gulp && hexo d`，输出如下

```bash
Enumerating objects: 2075, done.
Counting objects: 100% (2075/2075), done.
Delta compression using up to 12 threads
Compressing objects: 100% (793/793), done.
Writing objects: 100% (1241/1241), 1.94 MiB | 257.00 KiB/s, done.
Total 1241 (delta 539), reused 0 (delta 0)
remote: Resolving deltas: 100% (539/539), completed with 267 local objects.
To https://github.com/zhuChengChao/zhuChengChao.github.io.git
   cbbf0a51..13dd45a7  HEAD -> main
Branch 'master' set up to track remote branch 'main' from 'https://github.com/xxx/xxx.github.io.git'.
On branch master
nothing to commit, working tree clean  # GitHub推送成功
Enumerating objects: 2075, done.
Counting objects: 100% (2075/2075), done.
Delta compression using up to 12 threads
Compressing objects: 100% (793/793), done.
Writing objects: 100% (1241/1241), 1.95 MiB | 263.00 KiB/s, done.
Total 1241 (delta 539), reused 0 (delta 0)
remote: Resolving deltas: 100% (539/539), completed with 267 local objects.
remote: Powered by GITEE.COM [GNK-6.2]
To https://gitee.com/zhuChengChao/zhuChengChao.git
   cbbf0a51..13dd45a7  HEAD -> master
Branch 'master' set up to track remote branch 'master' from 'https://gitee.com/xxx/xxx.git'.
INFO  Deploy done: git  # Gitee推送成功
```

> 参考：https://gitee.com/help/articles/4136

**步骤6**：Gitee手动更新页面

> GitHub Pages是不需要的会自动更新，Gitee其实也有办法设置成自动，但因为后面Gitee都无法支持自定义域名就不想搞了

![手动更新Pages](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251723035.PNG)

#### Gitee Pages自定义域名（官方不支持）

本来的话是想着Gitee Page托管的内容也设定一下自定义域名，然后国内访问就直接走 Gitee Pages托管的内容，从外网访问博客就走GitHub Pages托管的内容，但是现在的**Gitee Pages是不支持自定义域名**了，官网上有说明如下：

https://gitee.com/help/articles/4228

这里还是记录一下配置过程吧，如果今后能突然支持了就过来再搞下：

其实的话是只要在修改一下之前通过阿里云所申请域名的解析设置，如下：

![解析路线配置](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251723893.PNG)

> 关于上面Gitee的IP，只要ping一下就行，如下：
>
> ```bash
> C:\Users\ZhuCC>ping zhuchengchao.gitee.io
> 
> 正在 Ping fn0wz54v.dayugslb.com [212.64.62.183] 具有 32 字节的数据:
> 来自 212.64.62.183 的回复: 字节=32 时间=301ms TTL=49
> 来自 212.64.62.183 的回复: 字节=32 时间=213ms TTL=49
> 来自 212.64.62.183 的回复: 字节=32 时间=277ms TTL=49
> 
> 212.64.62.183 的 Ping 统计信息:
>     数据包: 已发送 = 3，已接收 = 3，丢失 = 0 (0% 丢失)，
> 往返行程的估计时间(以毫秒为单位):
>     最短 = 213ms，最长 = 301ms，平均 = 263ms
> ```

**但由于现在Gitee官网就不支持自定义域名**，因此很无奈的就只要再改回去咯：

![改回解析路线配置](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251723294.PNG)

## 2.  站点收录

由于上述无法使用 Gitee Page 服务:angry:，故后续打算买一个阿里云的ECS服务器重新搞一下；等打折了再入手，然后再来记录一下。
