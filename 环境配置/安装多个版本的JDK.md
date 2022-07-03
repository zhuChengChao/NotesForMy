# 安装多个版本的JDK

> 刚刚开始学 Java 的时候安装了 JDK9 版本，后续发现还是 JDK8 使用的多些，而又不想删除原先版本，于是安装了两个版本的 JDK 在需要时切换一下即可:smile:

## 1. 安装第一个版本 JDK

1. 进入Oracle官网下载相应版本：https://www.oracle.com/java/technologies/javase-downloads.html

   > 我之前下的是 JDK9，但应该不是长期支持版本；
   >
   > 官网上能下到 JDK11 LTS这个版本

2. 选择 `JDK Download` 根据你机子下载对应版本即可；

   > 我安装在windows下的，则选择了：`jdk-9.0.4_windows-x64_bin.exe`

3. 双击进行安装，路径啥的根据需求修改即可：

   > **注意**：如下，没有必要勾选上 `公共JRE`，因为在JDK中已经包含了JRE，当然可以装咯
   >
   > ![安装JDK](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251727616.PNG)
   >
   > 即：JDK(Java Development Kit)包含了JRE(Java Runtime Environment)，JRE包含了JVM(Java Virtual Machine)

4. 安装完成后配置环境变量

   1. 在环境变量配置栏中，选择`系统变量/用户变量`，选择`新建`

      ![添加java环境变量1](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251727863.PNG)

   2. 在`系统变量/用户变量`的`path`中添加刚刚添加的变量

      ![添加java环境变量2](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251727406.PNG)

5. 配置完成，在cmd中进行测试

   ```bash
   C:\Users\ZhuCC>javac -version
   javac 9.0.4  # 成功安装了JDK9
   ```

## 2. 安装第二个版本的 JDK

1. 同样还是去Oracle的官网中下载，这里我下了 JDK8 

   在如下网址：https://www.oracle.com/java/technologies/javase/javase-jdk8-downloads.html

2. 选择对应的版本进行安装：`jdk-8u241-windows-x64.exe`

3. 安装步骤和上述相同，略

4. 安装成功后，添加环境变量：

   1. 此时你有两个 JDK 版本了，因此你可以设置不同的环境变量名对其进行区分，如：

      ![添加java环境变量3](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251727382.PNG)

   2. 在上一步 `1->4->2`中 你已经成功的在环境变量中的path设置了对应的变量：

      ```
      %JAVA_HOME%\bin
      ```

   3. 因此此处，你只需要把你需要的JDK版本改个名字即可，如次数我需要用 JDK8，则将 `JAVA_HOME_8` 修改为 `JAVA_HOME`即可

   4. 测试，还是打开cmd

      ```bash
      C:\Users\ZhuCC>javac -version
      javac 1.8.0_241  # 已经变成jdk8了
      ```

## 3. 在 IDEA 中设置对应的 JDK 环境

**现存工程修改JDK环境**

1. 在 IDEA 中依次选择 `File -> Project Structure`

2. 在 `Project Structure` 中选择 `SDKs`，添加安装的JDK环境即可，如下：

   ![添加java环境变量4](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251727134.PNG)

**新建工程设置JDK环境**

在新建立工程后，也会出现 `Project Structure`界面，操作与上方相同

