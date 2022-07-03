## 远程服务器使用Tensorboard

> 由于服务器端是纯命令行模式，且在远端。因此在训练模型时，需要通过相应的配置，才能在本地浏览器使用Tensorboard查看训练情况。

### 配置方式：采用了Xshell隧道

#### 配置

* 打开Xshell，右击相应的会话，在弹出的对话框中选择`属性`->`连接`->`SSH`->`隧道`->`添加`
* 在`侦听端口`和`目标端口`中填入相同的端口，此处填写了`6006`;
* 如下图所示：

![服务器使用tensorboard](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251731416.PNG)

> 服务器上tensorboard使用的端口是6006

#### 使用

* 在服务的命令窗口中使用：

  ```bash
  tensorboard --logdir=./training/model --port=6006
  ```

  来打开tensorboard服务

* 在本来的浏览器中输入：`localhost:6006`/`127.0.0.1:6006`

* 即可通过tensorboard查看目前训练的情况。

### 参考：

https://blog.csdn.net/a314688122a/article/details/81505082