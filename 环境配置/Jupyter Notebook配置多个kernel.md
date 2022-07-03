## Jupyter Notebook配置多个kernel

> 在 Anaconda 下配置了多个环境，而 Jupiter Notebook 只是安装在 `base` 环境下，为了能在 Jupiter Notebook 中使用不同的环境，故进行如下配置。
>
> 此次配置在windows10系统下

### 步骤

1. 打开`Anaconda Prompt`；

2. 查看现有的环境`conda info -e`，我此时的环境如下：

   ```base
   # conda environments:
   #
   base                  *  C:\Software\Anaconda3
   tensorflow-cpu           C:\Software\Anaconda3\envs\tensorflow-cpu
   tensorflow-gpu           C:\Software\Anaconda3\envs\tensorflow-gpu
   ```

3. 转到相应的环境`conda activate tensorflow-cpu`

4. 在激活的环境中安装`ipykernel`

   > 此处，若我使用`conda install ipykernel`都安装失败了；
   >
   > 因此采用pip安装：`pip3 install ipykernel -i https://pypi.tuna.tsinghua.edu.cn/simple/`

5. 选择环境写入Jupiter Notebook中：

   ```base
   python -m ipykernel install --user --name 要添加的环境 --display-name "展示的名字"
   ```

6. 打开Jupiter Notebook，此时就能进行kernel 的选择。

### 参考

https://blog.csdn.net/tong_he/article/details/78813494

https://blog.csdn.net/u011606714/article/details/77741324

