## 解决Git克隆慢问题

> 记录两种解决国内Git克隆慢的解决方案（第二种有效）

关于Git克隆或是上传代码龟速的问题真是让人很恼火:angry:，在网上找的关于修改host的方法，或许是出于我是校园网的缘故，并不起作用。

### 方式1：通过修改host解决

* 进入网站：https://www.ipaddress.com/ 分别搜索如下两个域名对于的IP地址

```
github.global.ssl.fastly.net
github.com
```

> * **`github.global.ssl.fastly.net`如下：**
>
> ![ip1](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206252035394.PNG)
>
> * **`github.com`如下：**
>
> ![ip2](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206252035705.PNG)

* 打开hosts文件
  * Windows上的hosts文件路径在`C:\Windows\System32\drivers\etc\hosts`
  * Linux的hosts文件路径在：`sudo vim /etc/hosts`
* 在打开的hosts文件末尾添加上上述查询出的IP地址和域名，如下

```
...
# localhost name resolution is handled within DNS itself.
#	127.0.0.1       localhost
#	::1             localhost

199.232.5.194 github.global.ssl.fastly.Net
140.82.114.4 github.com
```

* 保存后刷新DNS
  * Windows下的做法为：打开CMD控制终端，输入`ipconfig /flushdns`
  * Linux端下的做法为：在终端输入`sudo /etc/init.d/networking restart`

> 说明：
>
> 此方法对我而言并没有卵用，可能是由于我校园网的缘故:-(

[摘录自：git clone速度太慢的解决办法](https://www.jianshu.com/p/3f6477049ece)

### 方式2：利用码云克隆github项目

* 进入码云，新建一个仓库；

* 在创建的最后选择**导入已有仓库**，输入仓库地址后创建；

![mayun](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206252036062.PNG)

* 等待仓库创建完成...
* 之后就可通过git的bash愉快的clone代码了

**GIF演示：**

![gitclone](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206252036667.gif)

[参考：解决GitHub下载慢问题，不用修改HOSTS文件](https://blog.csdn.net/github_37847975/article/details/86477343)

