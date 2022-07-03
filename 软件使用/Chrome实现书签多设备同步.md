# Chrome实现书签多设备同步

> 无需翻墙；借助GitHub来实现Chrome收藏书签的多设备同步；
>
> 是这样的，前些日子将浏览器从`360极速浏览器`换成了`Google Chrome`，其实是很早之间就想换的，但因为很多书签都在原先的浏览器就搁置了，然后前些日子进行了书签的整理导出完成了替换；但是存在一个不翻墙Chrome书签无法很好同步的问题，现给出解决方案。

**步骤1：**去应用商店下载插件：`书签同步 1.0`，如下：

![书签同步插件](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206252036816.PNG)

> 地址如下：https://chrome.google.com/webstore/detail/%E4%B9%A6%E7%AD%BE%E5%90%8C%E6%AD%A5/fbcbemgibdnpboehnfcnkegefaomnlbk?hl=zh-CN
>
> 啊这...翻墙应该会把:smirk:

**步骤2：**按照要求先在GitHub上创建一个仓库，这里我创建了名为`ChromeBookmarkds`仓库，如下：

![GitHub仓库](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206252036124.PNG)

> 上面的Readme.md是自己家的哈

**步骤3：**生成一个token

> 不知道从什么时候开始秘钥好像不好用了，只能用 token 进行同步，第一次发现困扰了一会儿 :cry:

进入 `Setting→Developer settings→Personal access tokens`中，`Generate new token`新建立一个 token，这里需要选上 repo的权限（时间上可以设置一个永不过期？），如下：

![token生成](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206252036626.PNG)

> **注意：**生成完token后生成的token码只会出现一次！需要保存下来哦。

**步骤4：**进行仓库关联并上传内容，打开刚刚下载的插件关联上这个仓库，如下：

![同步书签内容](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206252036298.PNG)

> **注意**：这里打开一下 Remember Me 选线（太隐蔽了第一次还没发现）

查看一下仓库是否成提交，成提交的话这个 `bookmarks.json` 文件会被更新滴

**步骤5：**此时书签已经同步完成咯，当你在新的浏览器需要同步书签时，配置还是和上述相同，然后仅需要选择 `Download` 即可完成同步操作 。

OVER:smile: