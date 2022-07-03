# VS2015+OpenCV+Qt安装

> 记录一次安装 VS2015+OpenCV+Qt的过程
>
> > 好早之前的笔记了，突然发现那就同步一下

## 01.OpenCV

### 下载

* 进入官网链接: https://opencv.org，下载所需要的版本；

* 下载完成后直接双击，选择解压路径，解压到响应的文件夹中；

> 若之后需要把OpenCV的库配置到Qt中，则选择3.1.0或者3.0.0的版本，亲测可用

### 添加环境变量

一顿操作进入环境变量配置对话框，向其中添加进相应的路径，

eg：`C:\03ProgrammingSoftware\OpenCV\opencv\build\x64\vc14\bin`

> 注意：VS2015版本用vc14\bin，VS2017用户用vc15\bin

## 02.VS2015

### 下载安装

* 双击软件的光盘镜像文件`.iso`

> 来源：同学处拷贝了一个2015的社区版

* 运行安装程序，进行相应的路径选择等操作；

* 选择**自定义安装**只需要选择直接需要的组件即可；

> 例如我只需进行C++的代码编写，则选择：**编程语言->Visual C++**

* 等待安装完成

### 配置环境

* 建立一个新的空项目
* 打开：**视图 -> 属性管理器 -> Debug|x64 -> 右击选择：添加新项目属性表**

 ![001](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251727378.PNG)

* 取名后添加后缀名为`.props`，例如我取名为：`OpenCVProperty.props`

* 在新建的项目属性表中，选择刚刚建立的`OpenCVProperty.props`，右击选择属性后：

  * **VC++目录 -> 包含目录 -> 编辑**  加上：
    * C:\03ProgrammingSoftware\OpenCV\opencv\build\include
    * C:\03ProgrammingSoftware\OpenCV\opencv\build\include\opencv
    * C:\03ProgrammingSoftware\OpenCV\opencv\build\include\opencv2
  * **库目录 -> 编辑** 加上：
    
    * C:\03ProgrammingSoftware\OpenCV\opencv\build\x64\vc14\lib
  * **链接器 -> 输入 -> 附加依赖项 -> 编辑** 加上：
    
    * opencv_world310d.lib
    
    > 该文件在C:\03ProgrammingSoftware\OpenCV\opencv\build\x64\vc14\lib的目录下
    >
    > 如需要配置release版本的则输入不需要加d的lib

* 应用 -> 确定
* **在你工程的目录下找你的配置文件`OpenCVProperty.props`，将该文件保存在相应的位置，之后需要用到时再通过属性管理器添加即可**

### 验证

* 解决方案平台选择**x64**，而不是x86
* 输入程序

```c++
#include <opencv2/opencv.hpp>  //头文件

using namespace cv;  //包含cv命名空间

void main()
{
	//载入图片
	Mat image = imread("lena.jpg");
	//显示图片
	imshow("图片", image);
	waitKey(0);
}
```

若不报错，且出来图片(图片记得放到和.cpp)相同的路径下，则证明已经成功配置。

## 03.Qt + OpenCV

### 软件环境

#### 软件版本

- CMake          V3.9.0
- OpenCV       V3.1.0

> 版本3.0.0和版本3.0.0都配置成功
>
> 版本3.4.5配置失败

- Qt V5.8.0

#### 软件下载

- cmake：https://cmake.org/files/

> 下载比较慢,可通过科学上网：）

- opencv

> 参照上述01.

- Qt

> 软件安装包：百度云
>
> 自定义的安装过程过：需要选上**MinGW**这个编译器

### 系统变量添加

- cmake环境变量：
  - C:\03ProgrammingSoftware\CMake\bin
- qt环境变量：
  - C:\03ProgrammingSoftware\Qt5.8.0\Tools\mingw530_32\bin
  - C:\03ProgrammingSoftware\Qt5.8.0\5.8\mingw53_32\bin
- **特别注意：**
  - 若你安装了anaconda，并且配置的环境变量，那么**暂时删除anaconda的环境变量！**

### 用Cmake进行编译

- 在cmake的文件夹中找到**cmake-gui**
- 选择路径如下：

![002](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251727998.PNG)

- 点击configure，后续如下图：Next——Finish

![003](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251727484.PNG)

![004](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251727416.PNG)

- 配置完成后，**勾选中：“WITH_OPENGL”、“WITH_QT”，再次点击Configure**
- 上步运行完毕后，点**Generate**
- CMake成功

### 编译OpenCV

> 此处是最容易出问题的步骤
>
> 请默默祈祷：）
>
> 出了问题就百度，希望能找到解决方法：）

- 在刚刚cmake构造的文件夹中：Shift+右键，选择：在此处打开命令窗口
- 输入`mingw32-make`命令，按回车进行编译
- 当编译完成后，在当前目标下输入 `mingw32-make install `，按回车进行安装
- 这样可用于Qt的opencv库文件就生成了！！！

#### 再次添加环境

- 把编译完成后的文件夹中的的bin目录，再次添加到环境变量中
  - 我的文件是：C:\03ProgrammingSoftware\QtWithOpenCV\bin

### 在Qt中测试

- 新建一个Qt工程

- 在.pro文件中添加如下路径：

  ```c++
  INCLUDEPATH += C:\03ProgrammingSoftware\OpenCV\opencv\build\include
                 C:\03ProgrammingSoftware\OpenCV\opencv\build\include\opencv
                 C:\03ProgrammingSoftware\OpenCV\opencv\build\include\opencv2
  
  LIBS += C:\03ProgrammingSoftware\QtWithOpenCV\lib\libopencv_*.a
  ```

- 在main.cpp文件中输入

  ```c++
  #include "mainwindow.h"
  #include <QApplication>
  #include <opencv2/opencv.hpp>
  
  int main(int argc, char *argv[])
  {
      QApplication a(argc, argv);
  
      MainWindow w;
      w.show();
  
      cv::Mat image = cv::imread("C:/Users/Zhucc/Desktop/QtTest/testOpencv/lena.png");
      cv::imshow("lena", image);
  
      return a.exec();
  }
  ```

- 若出图像了，那么恭喜你，成功啦：）

------

### 参考文件

https://blog.csdn.net/Home_Wood/article/details/88162977

https://blog.csdn.net/biaobro/article/details/79141868

https://blog.csdn.net/naozhuo0615/article/details/81568881

https://blog.csdn.net/t980832453/article/details/79396972