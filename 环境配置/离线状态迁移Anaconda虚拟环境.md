# 离线状态迁移Anaconda虚拟环境

> 同样是项目需求，需要布署的服务器上的Anaconda安装到了普通账户下；
>
> 而后续所有的内容都需要通过root账户进行操作，而服务器已经在实地布署，联网比较麻烦；
>
> 本文提出，在无需联网的状态下，转移一下Anaconda的环境。

## 1. 安装 Anaconda

> 所有的操作最好都用root账户进行

1. 找到之前下载的`Anaconda3-2019.10-Linux-x86_64.sh`文件，最好移动到root目录下；

2. 运行`bash Anaconda3-2019.10-Linux-x86_64.sh` 进行安装；

   > 1. 根据相应提示回车即可
   >
   > ```bash
   > Welcome to Anaconda3 2019.10
   > 
   > In order to continue the installation process, please review the license
   > agreement.
   > Please, press ENTER to continue
   >    >>> 
   > ```
   >
   > 2. 接受许可：yes
   >
   > ```bash
   > Do you accept the license terms? [yes|no]
   >    >>> yes
   > ```
   >
   > 3. 确定安装路径：回车，选择默认安装路径即可
   >
   > ```bash
   > Anaconda3 will now be installed into this location:
   > /root/anaconda3
   > 
   >      - Press ENTER to confirm the location
   >      - Press CTRL-C to abort the installation
   >      - Or specify a different location below
   > 
   > [/root/anaconda3] >>> 
   > ```
   >
   > 4. 初始化环境：yes
   >
   > ```bash
   > Do you wish the installer to initialize Anaconda3
   > by running conda init? [yes|no]
   > [no] >>> yes
   > ```
   >
   > 5. 等待安装完成
   >
   > ```bash
   > Thank you for installing Anaconda3!
   > ```
   >
   > 此时，在默认安装路径下出现`anaconda3`这个文件夹

## 2. 环境拷贝

> 所有的操作最好都用root账户进行
>
> 假定之前安装的anaconda在`/home/username/anaconda3`这个目录下
>
> 上一步root下安装的anaconda3在`/root/anaconda3`这个目录下

1. 进入之前username账户中的安装anaconda3的相应位置，拷贝环境：

   `cd /home/username/anaconda3/envs/`

2. 对其中的pytorch目录进行拷贝：

   `tar -cvf pytorch.tar pytorch/`

3. 将生成的`pytorch.tar`文件拷贝至root下安装的anaconda3对应的目录：

   `mv ./pytorch.tar /root/anaconda3/envs/`

4. 回到`/root/anaconda3/envs/`相应目录，对移动到此的`pytorch.tar`文件进行解压操作

   ```bash
   cd /root/anaconda3/envs/
   tar -xvf pytorch.tar
   # 解压成功，此时envs下多了一个pytorch/目录
   ```

5. 修改root目录下的`.bashrc`文件

   ```bash
   cd ~
   vim .bashrc
   # 进入.bashrc文件最下方，添加如下内容
   export PATH="/root/anaconda3/envs/pytorch/bin:$PATH"
   # 添加完成后保存退出
   
   # 在终端中运行以下命令，更新
   source ~/.bashrc
   
   # 若成功的话应该能看到：
   # 原先为root@...
   # 变为(base)root@...
   ```

6. 测试：

   ```bash
   # 在终端中依次输入：
   conda activate pytorch
   # 进入python环境
   python
   >>> import torch
   >>> torch.cuda.is_available()
   True  # 结果为true则说明迁移成功
   >>> exit()
   ```

> **注意**：（补充内容，后续发现了新的问题，补充说明）
>
> 迁移之后，若输入 `pip3 list / pip list`，即查看安装包命令时，会出现如下错误：
>
> ```bash
> (torch) zcc@ubuntu-C246-WU4:~/anaconda3/envs$ pip list
> -bash: /home/zcc/anaconda3/envs/torch/bin/pip: /home/sxj/miniconda3/envs/torch/bin/python: 解释器错误: 没有那个文件或目录
> ```
>
> 解决方式：
>
> ```bash
> # 1.进入文件
> (torch) zcc@ubuntu-C246-WU4:~$ vim /home/zcc/anaconda3/envs/torch/bin/pip
> 
> # 2.将文件第一行修改为自己的地址
> # 原先：
> #!/home/sxj/miniconda3/envs/torch/bin/python
> # 修改为：
> #!/home/zcc/anaconda3/envs/torch/bin/python
> # 修改成功后保存退出
> 
> # 3. 修改完后你会发现无法clear了，如下
> (torch) zcc@ubuntu-C246-WU4:~$ clear
> terminals database is inaccessible
> # 解决：
> (torch) zcc@ubuntu-C246-WU4:~$ export TERMINFO=/usr/share/terminfo
> # 然后重连一下ssh，即可，当然最好是直接加到.barshrc中
> ```
>
> > 同理，修改pip3的话将pip改为pip3即可

## 3. 修改对应的配置文件

> 假定目前的root下安装的anaconda路径为：`/root/anaconda3`

1. 修改CMakeLists.txt文件

   ```bash
   # 第一处修改：
   # 原：
   include_directories(/home/username/anaconda3/envs/pytorch/include/python3.6m)
   # 修改为：
   include_directories(/root/anaconda3/envs/pytorch/include/python3.6m)
   
   # 第二处修改：在CmakeLists文件的最后
   # 原：
   /home/username/anaconda3/envs/pytorch/lib/libpython3.6m.so)
   # 修改为：
   /root/anaconda3/envs/pytorch/lib/libpython3.6m.so)
   ```

2. 删除工程目录中的build文件夹的内容，重新`cmake ..` 后 `make` 一下

