# Git基本使用

> Git 其实很早就用了，此处整合一下之前的笔记做个简单的记录。

**索引**

* Git的常用命令
* GitHub的使用
* Git版本创建和回退
* Git的工作区和暂存区
* Git分支管理

## 1. Git的常用命令

### 1.1 创建版本库

进入相应的目录

```bash
git init
```

> 可以看到目录下多了一个.git隐藏目录，这就是版本库目录

### 1.2 版本创建

```bash
git add filename
git commit -m '版本信息描述'
```

### 1.3 版本记录查看

```bash
git log
git log --pretty=oneline
git log --graph --pretty=oneline

# 查看操作记录
git reflog
```

### 1.4 版本回退

```bash
git reset --hard HEAD^
git reset --hard HEAD~1
git reset --hard 版本号
```

### 1.5 撤销修改

```bash
# 丢弃工作区(你的文件夹目录)的修改
git checkout -- filename
# 放弃暂存区(已经add了)中的修改
git reset HEAD filename
```

### 1.6 对不文件不同

```bash
# 查看工作区状态
git status
# 比较工作区中的版本和HEAD版本中文件的不同
git diff HEAD -- filename
# 比较两个区别
git diff 版本1 版本2 -- filename
```

### 1.7 删除文件

```bash
# 删除当前目录下的文件
# 如果的确要从版本库中删除
git rm
git commit
```

### 1.8 分支管理

```bash
# 查看分支
git branch
# 创建分支
git checkout -b 分支名
# 切换分支
git checkout 分支名
# 合并分支，能用fast forward，就用fast forward
git merge 分支名
# 不采用fast forward模式，会保存分支信息
git merge --no-ff -m '说明信息' 分支名
# 删除分支
git branch -d 分支名
```

### 1.9 保护工作现场

```bash
# 存储现场，
git stash
# 恢复现场
git stash pop
```

### 1.10 GitHub

```bash
# 克隆项目
git clone 复制过来的地址
# 上传分支
git push origin 分支名称
# 将本地分支跟踪服务器分支
git branch --set-upstream-to=origin/远程分支名称 本地分支名称
# 从远程分支上拉取代码
git pull origin 分支名称

```

## 2. GitHub使用

### 2.1 创建仓库

注意点：

- 可以勾选上：**Initialize this repository with a README**
- 可以在**Add.gitignore:**中选择你的编程语言，这样可以忽略掉一些相应的临时文件

### 2.2 添加ssh账户

- 在**setting**下选择**SSH and GPG keys**,再选择**New SSH key**
- 如果某台机器需要与github上的仓库交互，那么就要**把这台机器的ssh公钥添加到这个github账户上**

**Ubuntu下：**

- 在ubuntu的命令行中，回到用户的主目录下，**编辑文件.gitconfig，修改某台机器的git配置**
- 使用如下命令生成ssh密钥

```bash
ssh-keygen -t rsa -C "邮箱地址"
```

- 进入主目录下的.ssh文件件，下面有两个文件。
  - 公钥为id_rsa.pub
  - 私钥为id_rsa
- 查看公钥内容，复制此内容
- 回到浏览器中，填写标题，粘贴公钥

**Windows下**

- 打开Git Bash Here
- 输入命令

```bash
$ ssh-keygen -t rsa -C "邮箱地址"
# 后面的邮箱即为 github 注册邮箱
```

- 回车后让你选择路径建立相应的.ssh文件夹，直接回车建立到默认的用户目录下即可；

- 中途会让你输入密码

  - 该密码是在push文件时要输入的密码；
  - 可以不输入直接回车，那么你在push文件时就不需要在输入密码了，而可以直接push到github上

  > 推荐就别输密码了...容易忘，而且每次提交都需要输入，很是麻烦


- 根据路径提示，到用户目录下的这个.ssh文件夹下，下面有两个文件。

  - 公钥为id_rsa.pub
  - 私钥为id_rsa

- 然后GitHub网上相应位置添加（公钥`id_rsa.pub`）一下就可以了

- 验证是否添加成功，输入`$ ssh -T git@github.com`

  - 第一次输入时会提示如下；

  ```bash
  The authenticity of host 'github.com (13.229.188.59)' can't be established.
  RSA key fingerprint is SHA256:nThbg6kXUpJWGl7E1IGOCspRomTxdCARLviKw6E5SY8.
  Are you sure you want to continue connecting (yes/no/[fingerprint])?
  ```

  - 输入yes即可，若出现如下提示，表示添加成功

  ```bash
  Hi zhuChengChao! You've successfully authenticated, but GitHub does not provide shell access.
  ```

> 这方面内容，一下博客写的非常好：
>
> https://www.cnblogs.com/ayseeing/p/3572582.html

### 2.3 克隆项目

- 复制git地址
- 在命令行中复制仓库中的内容

```bash
git clone 复制过来的地址
```

- `https`和`SSH`克隆：
  - `https`可以随意克隆项目，但仅限与克隆；
  - `SSH`你必须是要克隆项目的管理员或拥有者，且需要添加`SSH Key`

> 当添加了SSH账户后，通过ssh克隆时发现，无法clone，问题如下：
>
> ```bash
> Cloning into 'ML-LinearRegrssion'...
> ssh_dispatch_run_fatal: Connection to 13.250.177.223 port 22: Software caused connection abort
> fatal: Could not read from remote repository.
> 
> Please make sure you have the correct access rights
> and the repository exists.
> ```
>
> 解决：
>
> 在bash中输入`eval "$(ssh-agent -s)"`即可
>
> > 网上找了一圈没人解释为什么原因，希望有人能告知：）

### 2.4 分支管理

#### 上传分支

- 项目克隆到本地之后，一般性都会创建一个你自己的分支
- 在分支上进行开发，开发完成后添加到暂存区提交
- **推送分支到github上，就是把该分支上的所有本地提交推送到远程库**

```bash
git push origin 分支名称
```

#### 将本地分支跟踪服务器分支

```bash
git branch --set-upstream-to=origin/远程分支名称 本地分支名称
```

#### 从远程分支上拉取代码

```bash
git pull origin 分支名称
# 使用上述命令会把远程分支上的代码下载并合并到本地所在分支
```

## 3. Git版本创建与回退

### 3.1 安装与配置（Linux下）

- 安装命令

```bash
sudo apt-get install git
```

- 安装成功后，运行如下命令

```bash
git
```

### 3.2 创建一个版本库

- 在新建的目录下输入终端命令

```bash
git init
```

> 可以看到目录下多了一个.git隐藏目录，这就是版本库目录

### 3.3 版本创建与回退

#### 版本创建

```bash
git add filename
git commit -m '版本信息描述'
```

> 问题，若上传文件时出现`warning: LF will be replaced by CRLF`的警告：
>
> 有个写的很好的详细说明如下：https://www.jianshu.com/p/450cd21b36a4

#### 版本记录查看

```bash
git log
git log --pretty=oneline
git log --graph --pretty=oneline
```

> 第一条命令会显示版本的详细信息
>
> 第二条命令会显示版本的简略信息
>
> 第二条命令会显示版本的简略信息 且会有提交的顺序图像（推荐使用）

#### 版本回退

- 若要回到上一个版本

```bash
git reset --hard HEAD^
```

> 其中**HEAD表示当前最新版本**，HEAD^表示当前版本的前一个版本,HEAD^^表示当前版本的前前个版本，也可以使用HEAD~1表示当前版本的前一个版本,HEAD~100表示当前版本的前100版本

- 回到特定的版本号

```
git reset --hard 版本号
```

> 版本号通过`git log` 或者 `git log --pretty=oneline`

#### 查看操作记录

```bash
git reflog
```

## 4. Git的工作区和暂存区

### 4.1 工作区

- Working Directory
- 就你工作的目录文件夹

查看工作区状态

```bash
git status
```

### 4.2 版本库

- Repository
- 工作区中的.git影藏目录，为git的版本库
- 版本库中存在**暂存区**和**分支**

`git add`把文件添加到暂存区

`git commit`把暂存区中的所有内容提交到当前的分支

### 4.3 撤销修改

#### 丢弃工作区的改动

```bash
git checkout -- filename
```

#### 放弃暂存区中的修改

```bash
git reset HEAD filename
```

### 4.4 对比文件的不同

```bash
# 比较工作区中的版本和HEAD版本中文件的不同
git diff HEAD -- filename
# 比较两个区别
git diff 版本1 版本2 -- filename
```

### 4.5 删除文件

- 当我们删除工作区的某些文件时，git会知道你删除了这些文件
- 此时工作区的版本和版本库中的版本已经不同了
- 选择一：

```bash
# 如果的确要从版本库中删除
git rm filename
gir commit -m 'describe infomation'
```

- 选择二

```bash
# 若要恢复，则丢弃工作区中的操作
git checkout -- filename
```

## 5. Git分支管理

### 5.1 创建分支

- 查看当前的工作分支

```bash
#查看当前在哪个分支下工作
git branch
```

- 主分支`master`，开始是`HEAD`指向`master`分支
- 创建新分支，再把`HEAD`指向新的分支

```bash
# 创建名为dev的分支
git checkout -b dev
# 接下来进行的提交都是在dev这个分支上了
```

- 切换回`master`分支

```bash
git checkout master
```

### 5.2 合并分支

- 当需要合并分支时

```bash
git merge 分支名
```

> Fast-forward为“快速合并”模式，也就是直接把master指向dev的当前提交

- 合并完成后可以删除分支

```bash
git branch -d 分支名
```

### 5.3 解决冲突

- 当不同分支对于同一个文件都修改了，**此时无法采用“快速合并”**
- 必须**手动解决冲突**后在进行提交
- git用<<<<<<<，=======，>>>>>>>标记出不同分支的内容
- 修改冲突内容后保存，再进行提交，之后就可以删除相应分支了

#### 分支管理策略

- 合并分支时，git可以用fast forward就用；
- 但是FF模式下，删除分支后，会丢掉分支的信息
- 禁用Fast forward模式

```bash
git merge --no-ff -m '说明信息' 分支名
# 这样操作不采用fast forward模式，会保存分支信息
```

### 5.4 BUG分支

- 修复bug时，我们会通过创建新的bug分支进行修复，然后合并，最后删除；
- 当手头工作没有完成时，先把工作现场git stash一下，然后去修复bug
- 修复后，再git stash pop，恢复工作现场。

```bash
# 存储现场，
git stash
# 恢复现场
git stash pop
```