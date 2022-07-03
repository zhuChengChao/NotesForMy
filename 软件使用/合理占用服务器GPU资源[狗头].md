# 合理占用服务器GPU资源[狗头]

> 场景：当你想进行模型训练时，发现GPU全被占用，怎么办？
>
> 本文给出一个合理的方式:dog2:，对此进行解决

## 解决方案1

在终端输入如下命令：`watch -n 设定刷新时间(s) nvidia-smi`

然后记起来了回来看看有空下来的没？

## 解决方案2

> 首先说明一下我的环境：
>
> * ubuntu 16.04
>* anaconda3
> * python3.6
>* tensorflow1.13
>   
>    再说明一下对应的脚本与占用GPU程序路径
>    
>    ```
>    - 用户目录
>    	- UseGPU
>    		- cifar-demo.py  # 占用GPU程序
>    		- mnist.npy      # 手写字数据
>    		- occupy.sh      # 占用GPU的sh脚本
>    ```

### `occupy.sh`脚本内容：

```bash
#!/bin/bash
# 变量定义
index=0  # GPU号
free=1600 # 空闲的GPU显存量(MB)
sleeptime=1  # 查询间隔时间 
while :
do
	# 需要执行的获取占用GPU显存的命令
	STRING=$(nvidia-smi -q -i $index | grep -m 1 'Free' | tr -cd '[0-9]')
	# echo $STRING
	
	if [ "$STRING" -gt "$free" ]; then
		# 当所占资源小于设定值时，执行相应程序
		echo `date -d today +"%Y-%m-%d %H:%M:%S"`
		echo "Find Free GPU:$index!!!"	
		# 运行对应程序 占TMD
		$(CUDA_VISIBLE_DEVICES=$index python3 ./cifar-demo.py)
		break
	else
		# 没找到，寻找下一块GPU
		echo "GPU:$index Not satisfied!"
		sleep $sleeptime
		
		# GPU号自增
		let index++
		if [ $index -ge 4 ]; then
			index=0
		fi 
	fi
done
```

### `cifar-demo.py`对应程序

> 为tensorflow的官方教程：https://github.com/tensorflow/docs/tree/master/site/en/r1/tutorials

```python
import numpy as np
import tensorflow as tf

# set train epoch
epochs = 10000000

# suppose the data in the ./mnist.npz
try:
    data = np.load('./mnist.npz')
    x_train, y_train, x_test, y_test = data['x_train'], data['y_train'], data['x_test'], data['y_test']

    # Normalized data
    x_train, x_test = x_train / 255.0, x_test / 255.0
except Exception as e:
    print('%s' % e)


model = tf.keras.models.Sequential([
  tf.keras.layers.Flatten(input_shape=(28, 28)),
  tf.keras.layers.Dense(512, activation=tf.nn.relu),
  tf.keras.layers.Dropout(0.2),
  tf.keras.layers.Dense(10, activation=tf.nn.softmax)
])

model.compile(optimizer='adam',
              loss='sparse_categorical_crossentropy',
              metrics=['accuracy'])

# start train
model.fit(x_train, y_train, epochs=epochs)
model.evaluate(x_test, y_test)
```

### `mnist.npy` 数据获取

1. 下载地址：https://storage.googleapis.com/tensorflow/tf-keras-datasets/
2. 网盘链接：链接：https://pan.baidu.com/s/1E_w_wJbvseU9I6kkShJahQ  提取码：5kdn 

### 可有可无的一点点说明

> 对`occupy.sh`脚本进行一点点说明：
>
> 1. 查看指定GPU的所有信息：`nvidia-smi -q -i 第几块GPU`
>2. 查看指定GPU的显存空闲`nvidia-smi -q -i 第几块GPU | grep "Free"`
> 
>```bash
>    # 如，此处我查看第2块GPU的显存占用：
>    (base) zcc@e0c362370fb8:~$ nvidia-smi -q -i 1 | grep "Free"
>            Free           : 15839 MiB	# FB Memory Usage (以此为准)
>            Free           : 16376 MiB	# BAR1 Memory Usage
>    
>    # 考虑到上面有两个输出，
>    # FB Memory Usage (对比后发现是以此为准)
>    #     Total                       : 16160 MiB
>    #     Used                        : 321 MiB
>    #     Free                        : 15839 MiB
>    # BAR1 Memory Usage
>    #     Total                       : 16384 MiB
>    #     Used                        : 8 MiB
>    #     Free                        : 16376 MiB
>    
>    # 修改命令如下，只输出一条
>    (base) zcc@e0c362370fb8:~$ nvidia-smi -q -i 1 | grep -m 1 "Free"
>            Used                        : 15839 MiB
>    
>    # 最后对数字进行截取
>    (base) zcc@e0c362370fb8:~$ nvidia-smi -q -i 1 | grep -m 1 "Free" | tr -cd "[0-9]"
>    15839
> ```

## 参考

https://blog.csdn.net/geng333abc/article/details/107481364

https://github.com/tensorflow/docs/tree/master/site/en/r1/tutorials

https://www.cnblogs.com/ypzhai/p/9997856.html

