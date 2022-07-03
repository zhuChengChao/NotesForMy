# 非root账户通过终端连入校园网

> 想要连入校园网的步骤：
>
> 1. 浏览器输入对应的IP地址，输入账号密码连网；
>
> 2. 下载对应软件，输入账号密码连网；
>
> 而面对没有界面的服务器，而你又没有root权限，如何上网？
>
> 这里参考zxh师兄给的一个脚本，进行以下记录

## 1. 找到登录时浏览器发送的post请求

1. 进入登录页面，比如的上网登录页面为`http://192.168.6.1/`

2. `F12` 打开浏览器的调试页面，找到对应的 NetWork 栏，并选上 Preserver Log

   ![登录post获取1](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206252034301.PNG)

   > 这里需要注意的是，中文版的火狐浏览器这个选项为`持续记录`，如下：
   >
   > ![登录post获取2](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206252035703.PNG)

3. 然后登录界面输入账号密码后，点击登录，获取POST请求，有如下

   ![登录post获取3](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206252035944.PNG)

   > 这里吐槽一下：学校登录注销界面在windows下采用Chrome/360浏览器 无法注销！找了个有界面的Linux用了火狐才能注销

4. 查看POST请求的对应的**请求头**与**表单请求**内容，如下：

   * 请求头：

     ![登录post获取4](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206252035349.PNG)

   * 请求内容：

     ![登录post获取5](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206252035322.PNG)

## 2. 编写脚本

在知道了发送的post的请求后就把相应的内容写入Python脚本，后续运行该脚本即可实现与网页登录相同的效果：

1. 找到上述的请求头内容：

   ```json
   POST /a70.htm HTTP/1.1
   Host: 192.168.6.1
   User-Agent: Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:84.0) Gecko/20100101 Firefox/84.0
   Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
   Accept-Language: zh-CN,zh;q=0.8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.3,en;q=0.2
   Accept-Encoding: gzip, deflate
   Content-Type: application/x-www-form-urlencoded
   Content-Length: 72
   Origin: http://192.168.6.1
   Connection: keep-alive
   Referer: http://192.168.6.1/a70.htm
   Upgrade-Insecure-Requests: 1
   ```

2. 把请求头加入变量中

   ```json
   headers = {
   	"Host": "192.168.6.1", 
   	"User-Agent": "Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:84.0) Gecko/20100101 Firefox/84.0", 
   	"Accept": "text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8", 
   	"Accept-Language": "zh-CN,zh;q=0.8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.3,en;q=0.2", 
   	"Accept-Encoding": "gzip, deflate", 
   	"Content-Type": "application/x-www-form-urlencoded", 
   	"Content-Length": "72", 
   	"Origin": "http://192.168.6.1", 
   	"Connection": "keep-alive", 
   	"Referer": "http://192.168.6.1/a70.htm", 
   	"Upgrade-Insecure-Requests": "1"
   }
   ```

3. 再根据表单数据构造登录内容

   ```json
   data = {
   	"DDDDD": 账号,
   	"upass": 密码,
   	"R1": "0",
   	"R2": "",
   	"R6": "0",
   	"para": "00",
   	"0MKKey": "123456"
   }
   ```

## 3. 完整脚本

```bash
#encoding=utf-8、
# 运行命令： python3 login.py --account 校园网账号 --password 校园网密码
import requests
import argparse

def parse_args():
    """
    Parse input arguments
    """
    parser = argparse.ArgumentParser(description='Train a chinese')
    parser.add_argument('--account', dest='account',
                        help='account', type=str)
    parser.add_argument('--password', dest='password',
                        help='password', type=str)
    args = parser.parse_args()
    return args

args = parse_args()

# 请求头
headers = {
	"Host": "192.168.6.1", 
	"User-Agent": "Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:84.0) Gecko/20100101 Firefox/84.0", 
	"Accept": "text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8", 
	"Accept-Language": "zh-CN,zh;q=0.8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.3,en;q=0.2", 
	"Accept-Encoding": "gzip, deflate", 
	"Content-Type": "application/x-www-form-urlencoded", 
	"Content-Length": "72", 
	"Origin": "http://192.168.6.1", 
	"Connection": "keep-alive", 
	"Referer": "http://192.168.6.1/a70.htm", 
	"Upgrade-Insecure-Requests": "1"
}

# 请求表单数据
data = {
   "DDDDD":args.account,
   "upass":args.password,
	"R1": "0",
	"R2": "",
	"R6": "0",
	"para": "00",
	"0MKKey": "123456"
}

# 请求的url地址
url ='http://192.168.6.1/a70.htm'

session = requests.Session()
session.post(url, headers = headers, data = data)
print("ok")
```

运行该脚本：`python3 login.py --account 校园网账号 --password 校园网密码`

即可实现登录