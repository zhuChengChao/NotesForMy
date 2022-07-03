# 提升Linux使用效率小操作

> 本文记录了个人在使用Linux时觉得好用的一些快捷方式/功能；
>
> 即为那种知道了能提高效率，但是的不知道也并没有影响的操作。
>
> **保存更新**:triumph:

## 历史命令

该操作用于快速查看已使用过的命令

**history**

1. `history`：查看历史执行过的命令

2. `!num`：重新执行history命令显示的第num行命令

**快捷键**

1. `ctrl + r`，进入命令查找模式；
2. 输入关键字查找曾经执行过的命令；
3. 继续`ctrl + r`，匹配上一次相关命令；
4. 匹配成功：
   * 回车：直接执行
   * `ctrl + j`：将命令赋值到shell中

## VIM 中的操作

vim中其实只需要知道`esc`进入命令模式；`:wq`退出并保存；`i/o`进入编辑模型编辑，上述三种操作就完全可以了

![vim模式转换](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206252036604.PNG)

纪录一些个人常用的其他操作：

* 剪切/复制/粘贴操作：

  * 剪切光标所在当前行：`dd`
  * 剪切从光标开始的n行：`ndd`

  * 复制光标所在当前行：`yy`
  * 复制从光标开始的n行：`nyy`

  * 将之前剪切/复制的行粘贴在光标行后：`p`

* 撤销操作：`u`

> 上述操作在命令模式下执行

* 关于行号：
  * 设置行号`：set nu`；取消行号显示`:set nonu`
  * 调到指定行`:行数`
* 查找操作：
  * 从上至下查找字符：`\字符串`
  * 从下至上查找字符：`?字符串`
  * 输入查找指令后，`n`——查找下一个；`N`——查找上一个；
  * 支持正则表达式
  * 查找时高亮显示：`:set hlsearch`；取消：`noh`
* 替换指令
  * 替换光标所在行的第一个匹配内容：`:s/原内容/新内容`
  * 替换光标所在行的所有匹配内容：`:s/原内容/新内容/g`
  * 替换全文中所有匹配内容：`:%s/原内容/新内容/g`

> 上述操作在末行模式下进行

* 不退出vim临时执行shell命令：`:!command`

## scp 命令

> Linux scp 命令用于 Linux 之间复制文件和目录
>
> scp 是 secure copy 的缩写, scp 是 linux 系统下基于 ssh 登陆进行安全的远程文件拷贝命令
>
> scp 是加密的，[rcp](https://www.runoob.com/linux/linux-comm-rcp.html) 是不加密的，scp 是 rcp 的加强版
>
> 摘自：[菜鸟教程：Linux scp 命令](https://www.runoob.com/linux/linux-comm-scp.html)，更加详细内容也可参考该网站
>
> 我的用途主要是同一局域网间服务器数据的拷贝

1. 正在操作服务器A，把**服务器A的数据拷贝到服务器B**

   ```bash
   # 命令格式：scp -r -P 22 服务器A的目录/文件 服务器B接受的用户名@服务器B的IP地址:放到服务器B的对应路径
   # -r:递归复制整个目录
   # -P port:指定数据传输用到的端口，即服务器B的端口号，若是22端口可不写
   
   # 整合：把服务器A中的UseGpu文件夹中的所有数据拷贝到服务器B中，如下
   # 服务器A IP:port   10.12.11.48:10033
   # 服务器B IP:port   10.12.11.192:22
   (base) zcc@e0c362370fb8:~$ scp -r  ./UseGPU/  zcc@10.12.11.192:/home/zcc/
   zcc@10.12.11.192's password:   # 输入服务器B的密码
   workspace.xml           100% 3284     3.2KB/s   00:00    
   modules.xml             100%  271     0.3KB/s   00:00    
   misc.xml                100%  205     0.2KB/s   00:00    
   profiles_settings.xml   100%  174     0.2KB/s   00:00    
   UseGPU.iml              100%  458     0.5KB/s   00:00    
   cifar-demo.py           100%  861     0.8KB/s   00:00    
   occupy.sh               100%  718     0.7KB/s   00:00    
   mnist.npz               100%   11MB  11.0MB/s   00:01  
   ```

2. 正在操作服务器A，把**服务器B的数据拷贝到服务器A**

   ```bash
   # 命令格式：scp -r -P 22 服务器B的用户名@服务器B的IP地址:服务器B需要传到A的文件/目录 服务器A放置的路径
   # -r:递归复制整个目录
   # -P port:指定数据传输用到的端口，即服务器B的端口号，若是22端口可不写
   
   # 整合：把服务器B中的UseGpu文件夹中的所有数据拷贝到服务器A中，如下
   # 服务器A IP:port   10.12.11.48:10033
   # 服务器B IP:port   10.12.11.192:22
   (base) zcc@e0c362370fb8:~$ scp -r zcc@10.12.11.192:/home/zcc/UseGPU/ /home/zcc/
   zcc@10.12.11.192's password:   # 输入服务器B的密码
   mnist.npz                100%   11MB  11.0MB/s   00:01    
   cifar-demo.py            100%  861     0.8KB/s   00:00    
   workspace.xml            100% 3284     3.2KB/s   00:00    
   misc.xml                 100%  205     0.2KB/s   00:00    
   modules.xml              100%  271     0.3KB/s   00:00    
   profiles_settings.xml    100%  174     0.2KB/s   00:00    
   UseGPU.iml               100%  458     0.5KB/s   00:00    
   occupy.sh                100%  718     0.7KB/s   00:00 
   ```