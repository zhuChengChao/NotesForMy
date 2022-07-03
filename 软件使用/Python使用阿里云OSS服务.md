## Python使用阿里云OSS服务

> 由于项目需要，在远程搭建了一个平台，通过该远程平台进行数据的采集，需要将数据内容传送至本地进行处理；为了实现该功能，采用了阿里云OSS对象储存的服务。
>
> > 40G包月只需1元:yum:
> >
> > 甚至还有客服致电给你，说有问题可直接通过电话联系对方，15星好评

### OSS安装

>  关于账号注册，开通服务等等功能直接去阿里云的官方进行相应操作即可

* 安装`python-devel`

  * win：此过程不需要，在安装Python时已经包含了；
  * Debian/Ubuntu：`apt-get install python-dev`

* 安装OSS：`pip3 install oss2 -i https://pypi.tuna.tsinghua.edu.cn/simple/`

  * 安装完成后可用python进行验证：

  ```python
  In [1]: import oss2
  
  In [2]: oss2.__version__
  Out[2]: '2.8.0'
  ```

### OSS使用

#### 创建存储空间

* 方式一：直接去网站页面进行创建即可；

* 方式二：通过代码创建

  ```python
  import oss2
  auth = oss2.Auth('yourAccessKeyId', 'yourAccessKeySecret')
  # Endpoint以杭州为例，其它Region请按实际情况填写。yourBucketName就是你要创建的Bucket
  bucket = oss2.Bucket(auth, 'http://oss-cn-hangzhou.aliyuncs.com', 'yourBucketName')
  bucket.create_bucket(oss2.models.BUCKET_ACL_PRIVATE)
  ```

  > `yourBucketName`为你的Bucket名称，若已存在则会报错；
  >
  > `yourAccessKeySecret`，别告诉别哈：）

#### 连接OSS

```python
auth = oss2.Auth('yourAccessKeyId', 'yourAccessKeySecret')
bucket = oss2.Bucket(auth, 'http://oss-cn-hangzhou.aliyuncs.com', 'yourBucketName')
```

#### 上传文件

```python
# 此处上传了图片文件
image_path = './test.jpg'
put_result = bucket.put_object_from_file('test.jpg', image_path)
if put_result.status == 200:
    # 若此时的status状态为200，则说明上传成功；在你选择的Bucket中已经存在文件名为'test.jpg'的文件了
    print('put success')
```

#### 下载文件

```python
# param1:oss上bucket中的文件名
# param2:保存在当地的文件路径+文件名
get_result = bucket.get_object_to_file('test01.jpg', './get/test.jpg')
if get_result == 200:
    print("get sucess")
```

### 整体代码

* 根据触发信号获取摄像头的图片；
* 获取摄像头的**实时**图片；
* 将先图片保存到本地->上传阿里云->在本地进行删除

```python
import oss2
import time
import cv2
import os
import RPi.GPIO as GPIO

class Camera(object):
    """摄像的初始化，配置"""
    def __init__(self, channel):
        self.capture = cv2.VideoCapture(channel)

        self.fps = int(self.capture.get(cv2.CAP_PROP_FPS))
        self.video_height = int(self.capture.get(cv2.CAP_PROP_FRAME_HEIGHT))
        self.video_width = int(self.capture.get(cv2.CAP_PROP_FRAME_WIDTH))

        self.capture.set(cv2.CAP_PROP_FRAME_WIDTH, self.video_width)
        self.capture.set(cv2.CAP_PROP_FRAME_HEIGHT, self.video_height)
        self.capture.set(cv2.CAP_PROP_FPS, self.fps)
        # 设置摄像头的缓存图片为1，为了能获取实时的图像
        self.capture.set(cv2.CAP_PROP_BUFFERSIZE, 1)

    def get_pic(self, image_dir=None, image_name=None, is_show=False, is_write=False):
        """
        写入照片
        """
        if self.capture.isOpened():
            # 读取缓存中的图片，起到清空缓存的作用
            abandon_ret, abandon_frame = self.capture.read()
            # 用于处理/上传的图片
            ret, frame = self.capture.read()
            # 判断图片信息是否读取成功
            if ret is False:
                print('get picture failed')
                return None
            # 是否需要显示
            if is_show is True:
                cv2.imshow('image', frame)
                cv2.waitKey(200)
            # 是否需要写入到文件中
            if is_write is True:
                cv2.imwrite(image_dir + '/' + image_name, frame)
            print('get picture success')
            return frame

    def release_camera(self):
        """
        释放摄像机资源
        """
        self.capture.release()
        cv2.destroyAllWindows()


class oss(object):
    """对象存储类，将图片传至阿里云端"""
    def __init__(self):
        self.auth = oss2.Auth('yourAccessKeyId', '**********************')
        self.bucket = oss2.Bucket(self.auth, 'http://oss-cn-hangzhou.aliyuncs.com', '20190731')

    def put_image(self, image_dir, image_name):
        put_result = self.bucket.put_object_from_file(image_name, image_dir + '/' + image_name)
        # 状态为200表示成功的上传
        if put_result.status == 200:
            print('put success')

    def get_image(self):
        pass


class gpio(object):
    """触发信号的控制"""
    event_flag = False  # 判断是否有电平触发信号

    def __init__(self, pin):
        """GPIO初始化"""
        self.pin = pin
        # 设置引脚应用模式
        GPIO.setmode(GPIO.BCM)
        GPIO.setup(self.pin, GPIO.IN, pull_up_down=GPIO.PUD_DOWN)
        # 设计引脚触发模式为：上升沿触发模式，防抖的时间为1000ms
        GPIO.add_event_detect(self.pin, GPIO.RISING, callback=gpio.callback, bouncetime=1000)

    @staticmethod
    def callback(flag):
        """回调函数"""
        gpio.event_flag = True

    def stop_event(self):
        GPIO.remove_event_detect(self.pin)

    def start_event(self):
        GPIO.add_event_detect(self.pin, GPIO.RISING, callback=gpio.callback, bouncetime=1000)

    @staticmethod
    def release():
        """GPIO释放引脚资源"""
        GPIO.cleanup()


if __name__ == "__main__":
    picture = None
    sleep_time = 0
    picture_path = './put'
    picture_name = '.jpg'
    gpio_pin = 18

    # oss配置
    oss_server = oss()
    # 开启摄像头
    camera = Camera(0)
    # GPIO初始化
    event = gpio(gpio_pin)

    while True:
        if gpio.event_flag is True:
            gpio.event_flag = False  # 标志位置位
            event.stop_event()  # 关闭事件触发
            picture_name = time.strftime('%Y-%m-%d-%H-%M-%S', time.localtime(time.time())) + '.jpg'
            # 获取图片
            picture = camera.get_pic(picture_path, picture_name, is_show=True, is_write=True)

            if picture is not None:
                pass
                # oss_server.put_image(picture_path, picture_name)
            else:
                camera.release_camera()
                camera = Camera(0)
            # 删除照片，防止内存爆炸
            os.remove(picture_path + '/' + picture_name)

            event.start_event()  # 开启事件触发
            # time.sleep(sleep_time)

    gpio.release()
```

### 参考

https://ptorch.com/news/209.html