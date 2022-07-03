# Hexo框架+NexT主题博客美化篇

> 前文中，我们成功借助Hexo框架搭建了博客；那理所当然的是要作一些些美化操作来使博客看着更加舒服一些咯，此文就记录一下从网上荡来的一些美化博客的操作。
>
> > 后续看到好的美化会及时抄袭更新哟:smirk:

### 0.前置说明

首先需要说明的博客采用的主题的NexT主题。

其次说明两个配置文件：

1. Hexo框架配置文件：即Hexo跟路径下的 `/_config.yml`文件
2. HexT主题配置文件：即 `/themes/next/_config.yml` 文件

用NexT主题即在框架配置文件中设置了`theme: next`

还有就是修改完主题配置后都需要重新布署即：`hexo g && hexo s`，先在本地布署看看效果对不对比较合适。

### 1.NexT中Scheme修改

NexT中提供了4种可供修改的Schema，如下：

```yml
# Schemes
scheme: Muse
#scheme: Mist
#scheme: Pisces
#scheme: Gemini
```

默认就是 `Muse`，启动后是这样子的：

![MuseScheme](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251724532.PNG)

比较了一些其他的主题，我更喜欢`Gemini`这个Scheme，因此打开这个注释并把`Muse`注上，重新启动Hexo，如下：

![GeminiScheme](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251725142.PNG)

### 2.中文设置

上面可知显示的都是英文，比如这个Home和Archives；

在框架配置文件中修改如下：

```
language: zh-CN
```

> 原先是`language: en`

修改完成后重启即可，此时可见侧边栏都变中文了，如下：（可以对比上面看出差别）

![中文修改](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251725791.PNG)

> 当然也支持其他语言，但没必要....

### 3.站点信息的设置

从上面的文中设置中可以看到，现在博客的侧边状态栏显示的信息都不是自己的，需要进行修改，原先为：

```yml
# Site
title: Hexo
subtitle: ''
description: ''
keywords:
author: John Doe
language: zh-CN  # 这个上面刚刚改的
timezone: ''
```

我这里就修改为如下：

```yml
# Site
title: 贤余超  # 博客名
subtitle: 一位底层码农 :-)  # 副标题
description: 不急不躁。  # 描述
keywords:
author: ZhuCC  # 作者
language: zh-CN  # 显示语言
timezone: Asia/Shanghai  # 时区
```

修改完后如下：

![侧边栏修改](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251725520.PNG)

### 4.个人头像设置

在 `/source/images/` 下放置需要显示的头像，然后修改主题配置文件，原文件如下：

> 没有上述文件夹的话建一个就行

```yml
# Sidebar Avatar
avatar:
  # Replace the default image and set the url here.
  url: #/images/avatar.gif
  # If true, the avatar will be dispalyed in circle.
  rounded: false
  # If true, the avatar will be rotated with the cursor.
  rotated: false
```

修改后：

```yml
# Sidebar Avatar
avatar:
  # Replace the default image and set the url here.
  url: /images/头像.jpg  # 指出头像图片路径
  # If true, the avatar will be dispalyed in circle.
  rounded: true  # 头像以圆形的方式显示
  # If true, the avatar will be rotated with the cursor.
  rotated: false  # 是不是旋转，不须换把自己转起来就没开
```

修改效果如下：

![添加头像](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251725519.PNG)

### 5.扩充侧边栏内容

从上面可以看到此时侧边栏就首页和归档两个选项，默认如下：

```yml
# Usage: `Key: /link/ || icon`
# Key is the name of menu item. If the translation for this item is available, the translated text will be loaded, otherwise the Key name will be used. Key is case-senstive.
# Value before `||` delimiter is the target link, value after `||` delimiter is the name of Font Awesome icon.
# When running the site in a subdirectory (e.g. yoursite.com/blog), remove the leading slash from link value (/archives -> archives).
# External url should start with http:// or https://
menu:
  home: / || fa fa-home
  #about: /about/ || fa fa-user
  #tags: /tags/ || fa fa-tags
  #categories: /categories/ || fa fa-th
  archives: /archives/ || fa fa-archive
  #schedule: /schedule/ || fa fa-calendar
  #sitemap: /sitemap.xml || fa fa-sitemap
  #commonweal: /404/ || fa fa-heartbeat
```

从上面可以看出，此时打开了 home和archives，就是能看见的首页和归档；

在仔细看看上面的注释内容，使用方式：`Key: /link/ || icon`

* key：菜单名，是会被翻译的；
* `||`：之前的为`link`，就是这个类别的菜单的连接咯；之后的是图标

> 关于上面的图标，指的是Font Awesome图标；后续有需求在详细了解一下暂时没需求，默认的就很好用了

从上可以看出，你要什么就取消什么注释即可咯，位置也可以互换，目前我选择了这么几项：

```yml
menu:
  home: /|| fa fa-home  # 主页
  about: /about/|| fa fa-user  # 关于页面
  tags: /tags/|| fa fa-tags  # 标签页
  categories: /categories/|| fa fa-th  # 分类页
  archives: /archives/|| fa fa-archive  # 归档页
  #schedule: /schedule/ || fa fa-calendar  
  #sitemap: /sitemap.xml || fa fa-sitemap  
  #commonweal: /404/ || fa fa-heartbeat 
```

修改完之后如下：

![侧边栏标签添加](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251725655.PNG)

至此侧边栏基本是搞的符合我的审美了。

但此时有个问题就是，点击新出现的关于和标签你会发现是空的连接...此时就需要创建对应的index页面了，操作如下：

1. 创建标签页

   在命令行模式下执行命令：`hexo new page tags`

   可以看到在`/source/` 目录下多了 tags文件夹，内部的 `index.md` 文件就是到时候会显示的内容

   再次访问就可以别访问到了，但是默认创建的 index.md 文件不太符合阅读习惯，修改如下：

   ```
   title: 标签
   date: 2021-11-09 21:30:10
   type: "tags"
   comments: false  # 这个是后续添加评论功能是为了使当然页面不支持评论而设定的
   ```

2. 创建标签页

   在命令行模式下执行命令：`hexo new page categories`

   可以看到在`/source/` 目录下多了 categories 文件夹，内部的`index.md`文件就是到时候会显示的内容

   再次访问就可以别访问到了，但是默认创建的 index.md 文件不太符合阅读习惯，修改如下：

   ```
   title: 分类
   date: 2021-11-09 21:34:20
   type: categories
   comments: false
   ```

3. 创建关于页

   在命令行模式下执行命令：`hexo new page about`

   可以看到在`/source/` 目录下多了 about 文件夹，内部的`index.md`文件就是到时候会显示的内容

   再次访问就可以别访问到了，但是默认创建的 index.md 文件不太符合阅读习惯，修改如下：

   ```
   title: 关于
   date: 2021-11-09 21:36:02
   type: about
   comments: false
   ```

至此侧边栏上的所有内容都可以正常的跳转了。

### 6.设置博客的图标

博客的图标挺重要的，默认图标没啥好说的，现在就把默认的图标给修改了，操作如下：

找个`.ico`的图标放到文件夹 `/source/images/`下，这里我找了一个马里奥的图标，命名为：`Mario.ico`，在主题配置文件中，修改如下：

```yml
# 原内容：
favicon:
  small: /images/favicon-16x16-next.png
  medium: /images/favicon-32x32-next.png
  apple_touch_icon: /images/apple-touch-icon-next.png
  safari_pinned_tab: /images/logo.svg
  #android_manifest: /images/manifest.json
  #ms_browserconfig: /images/browserconfig.xml
  
# 修改为：
favicon:
  small: /images/Mario.ico
  medium: /images/Mario.ico
  apple_touch_icon: /images/apple-touch-icon-next.png
  safari_pinned_tab: /images/logo.svg
  #android_manifest: /images/manifest.json
  #ms_browserconfig: /images/browserconfig.xml
```

重启发现可以发现标签页上的图标已经变了。



### 7.添加Git传送门

网址如下：https://tholman.com/github-corners/，然后选择一个你喜欢的，我选择了最经典的第一个黑色的；

打开配置文件：`/themes/next/layour/_layout.swig`

找到标签：`<div class="headband"></div>`，然后复制上述内容到下方：

```xml
<a href="https://your-url" class="github-corner" aria-label="View source on GitHub">
    <svg width="80" height="80" viewBox="0 0 250 250" style="fill:#151513; color:#fff; position: absolute; top: 0; border: 0; right: 0;" aria-hidden="true">
        <path d="M0,0 L115,115 L130,115 L142,142 L250,250 L250,0 Z"></path>
        <path d="M128.3,109.0 C113.8,99.7 119.0,89.6 119.0,89.6 C122.0,82.7 120.5,78.6 120.5,78.6 C119.2,72.0 123.4,76.3 123.4,76.3 C127.3,80.9 125.5,87.3 125.5,87.3 C122.9,97.6 130.6,101.9 134.4,103.2" fill="currentColor" style="transform-origin: 130px 106px;" class="octo-arm"></path>
        <path d="M115.0,115.0 C114.9,115.1 118.7,116.5 119.8,115.4 L133.7,101.6 C136.9,99.2 139.9,98.4 142.2,98.6 C133.8,88.0 127.5,74.4 143.8,58.0 C148.5,53.4 154.0,51.2 159.7,51.0 C160.3,49.4 163.2,43.6 171.4,40.1 C171.4,40.1 176.1,42.5 178.8,56.2 C183.1,58.6 187.2,61.8 190.9,65.4 C194.5,69.0 197.7,73.2 200.1,77.6 C213.8,80.2 216.3,84.9 216.3,84.9 C212.7,93.1 206.9,96.0 205.4,96.6 C205.1,102.4 203.0,107.8 198.3,112.5 C181.9,128.9 168.3,122.5 157.7,114.1 C157.9,116.9 156.7,120.9 152.7,124.9 L141.0,136.5 C139.8,137.7 141.6,141.9 141.8,141.8 Z" fill="currentColor" class="octo-body"></path>
    </svg>
</a>
<style>.github-corner:hover .octo-arm{animation:octocat-wave 560ms ease-in-out}@keyframes octocat-wave{0%,100%{transform:rotate(0)}20%,60%{transform:rotate(-25deg)}40%,80%{transform:rotate(10deg)}}@media (max-width:500px){.github-corner:hover .octo-arm{animation:none}.github-corner .octo-arm{animation:octocat-wave 560ms ease-in-out}}</style>
```

需要修改的就是上的URL地址修改为你的GitHub地址，此处我是：`https://github.com/zhuChengChao`；

修改成功后效果就是右上角多了个一个传送链接。

### 8.修改博文页面宽度

默认状态下页面两边的宽度比较宽，可以通过如下操作修改：

打开文件：`/themes/next/source/css/_schemes/Pisces/_layout.styl`

在文件最后添加如下内容：

```stylus
// 以下为新增代码！！修改post宽度
header{ width: 80% !important; }
header.post-header {
	width: auto !important;
}
.container .main-inner { width: 80%; }
.content-wrap { width: calc(100% - 260px); }

.header {
	+tablet() {
	  width: auto !important;
	}
	+mobile() {
	  width: auto !important;
	}
}

.container .main-inner {
	+tablet() {
	  width: auto !important;
	}
	+mobile() {
	  width: auto !important;
	}
}

.content-wrap {
	+tablet() {
	  width: 100% !important;
	}
	+mobile() {
	  width: 100% !important;
	}
}
```

> 具体差别就不展示了，你可以删除再加上看看变化

### 9.页面整体圆角呈现

现在页面的一个个板块都是方形的不是很好看，故将其修改为圆角的；

打开文件：`themes/next/source/css/_variables/Gemini.styl`，在文件的最后添加如下内容：

```stylus
// 修改主题页面布局为圆角
$border-radius-inner           = 10px 10px 10px 10px;
$border-radius                 = 10px;
```

> 具体差别就不展示了，你可以删除再加上看看变化

### 10.去掉顶部黑线

仔细观察，网站的顶部是有条黑线的！下拉后就没了，驱除方法如下：

创建文件：`/source/_data/styles.styl`

添加内容：

```stylus
// 去掉顶部黑线：
.headband {display:none;}
```

> 之前看的教程是在一个`themes/next/source/css/_custom/custom.styl`文件夹下修改，我这个版本没上述文件了，在主题配置文件中如下：
>
> ```yml
> # Define custom file paths.
> # Create your custom files in site directory `source/_data` and uncomment needed files below.
> custom_file_path:
>   #head: source/_data/head.swig
>   #header: source/_data/header.swig
>   #sidebar: source/_data/sidebar.swig
>   #postMeta: source/_data/post-meta.swig
>   #postBodyEnd: source/_data/post-body-end.swig
>   #footer: source/_data/footer.swig
>   #bodyEnd: source/_data/body-end.swig
>   #variable: source/_data/variables.styl
>   #mixin: source/_data/mixins.styl
>   #style: source/_data/styles.styl
> ```
>
> 打开上述配置：`style: source/_data/styles.styl`，然后把部分关于`custom.styl`的内容写到这里即可

### 11.添加博文结束标记

在每篇文章结束后添加结束标志：

1. 首先在路径：`themes/next/layout/_marcro/` 下新建文件：`passage-end-tag.swig`

   文件内容如下：（这里我填的就是下面的内容）

   ```xml
   <link rel="stylesheet" href="https://cdn.staticfile.org/font-awesome/4.7.0/css/font-awesome.css">
   
   <div>
       {% if not is_index %}
          <div style="text-align:center;color:#bfbfbf;font-size:16px;">
                <span>--- 本文结束 </span>
                <i class="fa fa-thumbs-o-up"></i>
                <span> 感谢阅读 ---</span>
          </div>
       {% endif %}
   </div>
   ```

   > 这里用到了Font Awesome图标，上述有提

2. 然后打开`themes/next/layout/_marcro/post.swig`文件，在`post-body`结束之后添加如下内容：

   ```xml
   {#####################}
   {### END POST BODY ###}
   {#####################}
   # post-body标记结束
   <div>
       {% if not is_index %}
       {% include 'passage-end-tag.swig' %}
       {% endif %}
   </div>
   ```

3. 最后打开主题配置文件，在最后添加以下代码：

   ```yml
   # 文本结束标记开启
   post_end_tag:
       enabled: true  # 是否开启文末的本文结束标记
   ```

添加完成后，效果如下：

![文章结束标记](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251725481.PNG)

### 12.修改博文标签符号

博文结束后，若有标签默认是这样的，如下：

![默认结束标签](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251725320.PNG)

将 `#` 修改一下，采用Font Awesome图标好看一些，修改如下：

打开文件：`/themes/next/layout/_macro/post.swig`，找到如下内容：

```xml
{%- if post.tags and post.tags.length %}
    {%- if theme.tag_icon %}
        {%- set tag_indicate = '<i class="fa fa-tag"></i>' %}  # 判断条件
        {% else %}
        {%- set tag_indicate = '#' %}
    {%- endif %}
    <div class="post-tags">
        {%- for tag in post.tags.toArray() %}
        <a href="{{ url_for(tag.path) }}" rel="tag">{{ tag_indicate }} {{ tag.name }}</a>
        {%- endfor %}
    </div>
{%- endif %}
```

上面的代码我品了一下，这里显示的话是：`{{ tag_indicate }} {{ tag.name }}`，意思就是前面的标签符号+标签名，然后这个标签的符号 `tag_indicate`，上面是一个判断来的，但好像一直满足不了第一个判断条件，这时候如果不想用默认的`#`话，就把下面的内容该了就好了，我之类选了一个Font Awesome

> 使用方式见：http://www.fontawesome.com.cn/，后续不再提了

上面的网址里选了个好看的如下：

```xml
{%- if theme.tag_icon %}
	{%- set tag_indicate = '<i class="fa fa-tag"></i>' %}
{% else %}
	{%- set tag_indicate = '<i class="fa fa-hand-peace-o" aria-hidden="true"></i>' %}
{%- endif %}
```

现在的效果如下：

![修改结束标签](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251726453.PNG)

### 13.当前浏览进度显示

修改主题配置文件，默认如下：

```yml
back2top:
  enable: true
  # Back to top in sidebar.
  sidebar: false
  # Scroll percent label in b2t button.
  scrollpercent: false
```

这里采用的是通过目录的侧边栏+返回顶部功能

![无返回顶部](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251725328.PNG)

修改上述配置：

* sidebar：设定为true，支持返回顶部；
* scrollpercent：设置为true，表示显示阅读百分比；

修改完成后如下：（有了阅读进度+返回顶部）

![有返回顶部](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251725720.PNG)

### 14.添加站内搜索

通过安装插件来实现，即先安装一下两个插件：

```bash
npm install hexo-generator-search --save
npm install hexo-generator-searchdb --save
```

在框架配置文件末尾加入如下内容：

```yml
# 搜索配置
search:
  path: search.xml
  field: post
  format: html
  limit: 10000
```

在主题配置文件中修改如下：

```yml
local_search:
    enable: true
```

重启服务即可支持搜索功能，如下：

![支持搜索](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251725941.PNG)

### 15.添加评论

此处使用Valine评论系统，使用参考网址如下：https://valine.js.org/

反正就是创建一个应用，我这里的创建如下：

![评论：添加应用](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251725961.PNG)

然后在进入新创建的应用，找到这个应用凭证，这里的AppID和AppKey是需要用到的，如下：

![评论：应用信息](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251725351.PNG)

在主题配置文件中找到valine项，默认如下：

```
# Valine
# For more information: https://valine.js.org, https://github.com/xCss/Valine
valine:
  enable: false
  appid: # Your leancloud application appid
  appkey: # Your leancloud application appkey
  notify: false # Mail notifier
  verify: false # Verification code
  placeholder: Just go go # Comment box placeholder
  avatar: mm # Gravatar style
  guest_info: nick,mail,link # Custom comment header
  pageSize: 10 # Pagination size
  language: # Language, available values: en, zh-cn
  visitor: false # Article reading statistic
  comment_count: true # If false, comment count will only be displayed in post page, not in home page
  recordIP: false # Whether to record the commenter IP
  serverURLs: # When the custom domain name is enabled, fill it in here (it will be detected automatically by default, no need to fill in)
  #post_meta_order: 0
```

然后我们需要修改的就是appid和appkey，把刚刚应用的AppID和AppKey填上去，还有就是placeholder就是评论框中默认的内容，这里的我修改成了`placeholder: 请多多指教~ # Comment box placeholder`，最终效果如下：

![评论：最终效果](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251725946.PNG)

> 关于这个评论内容的话现在先放上再说仅为了美化，后续肯定还是要再回来好好整整的；真正的让其具有实用性！
>
> 还有一点about/tags/category页面中不应该是有评论页面的，如果出息了的话在相应的index.md文件头中加入：`comments: false`，具体见上面[5.扩充侧边栏内容]()

### 16.博文字数统计功能

安装插件：`npm install hexo-symbols-count-time --save`

主题配置文件中修改如下：

```yml
symbols_count_time:
  separated_meta: true
  item_text_post: true
  item_text_total: false
  symbols: true # 统计单篇文章字数(我就开了这个)
  time: false # 估算单篇文章阅读时间(又不是小说...这个估算没意义的)
  total_symbols: false # 统计站点总字数
  total_time: false  # 估算站点总阅读时间
  exclude_codeblock: false  #是否包括代码
```

效果如下：

![字数统计](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251725595.PNG)

还有在博文的底部也会变化，如下：

![底部字数统计](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251725875.PNG)

> 这里我虽然吧阅读时间关了但还是会显示，不知道为啥...
>
> > **补充**：发现了原因，显示是都会显示的，要是选择了false的话第一次首次布署时间上显示的就是Na很难开，因而还是开启吧，即上面的：`time: true`

### 17.3D动态标签云

同样需要安装插件：`npm install hexo-tag-cloud@^2.* --save`

然后修改配置文件：`/themes/next/layout/_marco/sidebar.swig`

```xml
	...上文略...
    {% if site.tags.length > 1 %}
        <script type="text/javascript" charset="utf-8" src="/js/tagcloud.js"></script>
        <script type="text/javascript" charset="utf-8" src="/js/tagcanvas.js"></script>
        <div class="widget-wrap">
            <div id="myCanvasContainer" class="widget tagcloud">
                <canvas width="250" height="250" id="resCanvas" style="width=100%">
                    {{ list_tags() }}
                </canvas>
            </div>
        </div>
    {% endif %}

    </div>
  </aside>
  <div id="sidebar-dimmer"></div>
{% endmacro %}
```

最终效果如下：

![动态标签云](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251725519.PNG)

### 19.设置背景图片

这个我没采用，感觉纯纯的就很好看，非要加的话如下：

```stylus
// 背景图像
// body {
//     background:url(https://source.unsplash.com/random/1600x900);
//     background-repeat: no-repeat;
//     background-attachment:fixed;
//     background-size: cover;
//     background-position:50% 50%;
// }
```

这个文件是 `/source/_data/style.styl`

### 20. 文章底部修改

文章默认底部如下：

![默认底部](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251725137.PNG)

现做如下两点修改：

1. 不显示：由Hexo&NexT.Gemini强力驱动

   修改配置文件：`/themes/next/layout/_partials/footer.swig`，注释掉下述代码：

   ```xml
   <!-- 
   {%- if theme.footer.powered %}
     <div class="powered-by">
       {%- set next_site = 'https://theme-next.org' %}
       {%- if theme.scheme !== 'Gemini' %}
         {%- set next_site = 'https://' + theme.scheme | lower + '.theme-next.org' %}
       {%- endif %}
       {{- __('footer.powered', next_url('https://hexo.io', 'Hexo', {class: 'theme-link'}) + ' & ' + next_url(next_site, 'NexT.' + theme.scheme, {class: 'theme-link'})) }}
     </div>
   {%- endif %}
   -->
   ```

2. 让上面的红心跳动起来，通过修改主题配置文件，修改项animated如下：

   ```yml
   # Icon between year and copyright info.
   icon:
       # Icon name in Font Awesome. See: https://fontawesome.com/icons
       name: fa fa-heart
       # If you want to animate the icon, set it to true.
       animated: true  # 修改为true即可
       # Change the color of icon, using Hex Code.
       color: "#ff0000"
   ```

最终效果如下（红心是会跳动的）：

![修改底部后结果](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251725407.PNG)

### 21. 取消目录编号

大部分写文章时其实都已经编号完成了，和自动编号叠加在一起会比较乱，这里就取消一下：

修改主题配置文件如下：

```yml
# Table of Contents in the Sidebar
# Front-matter variable (unsupport wrap expand_all).
toc:
  enable: true
  # Automatically add list number to toc.
  number: true  # 这里为true表示会自动编号，改为false即可
  # If true, all words will placed on next lines if header width longer then sidebar width.
  wrap: false
  # If true, all level of TOC in a post will be displayed, rather than the activated part of it.
  expand_all: false
  # Maximum heading depth of generated toc.
  max_depth: 6
```

因此只要将上面的`number`修改为false即可，效果如下：

![自动编号](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251725883.PNG)

### 22. 代码复制按钮

修改主题配置文件如下

```yml
codeblock:
  # Code Highlight theme
  # Available values: normal | night | night eighties | night blue | night bright | solarized | solarized dark | galactic
  # See: https://github.com/chriskempson/tomorrow-theme
  highlight_theme: normal
  # Add copy button on codeblock
  copy_button:
    enable: true  # 开启支持
    # Show text copy result.
    show_result: true  # 复制成功给反馈
    # Available values: default | flat | mac
    style:
```

即可实现支持，如下：

![代码复制](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251725253.PNG)

### 23.图片灯箱功能

默认情况下图片好像是不能全屏查看，而图片灯箱就大大简化了博客中图片的查看方式，其作为主题的依赖，安装方式如下：

将依赖安装到：`/themes/next/source/lib`下，可以在博客根目录下运行命令：

```bash
$ git clone https://github.com/theme-next/theme-next-fancybox3 themes/next/source/lib/fancybox
Cloning into 'themes/next/source/lib/fancybox'...
remote: Enumerating objects: 34, done.
remote: Total 34 (delta 0), reused 0 (delta 0), pack-reused 34
Unpacking objects: 100% (34/34), done.
```

安装成功后可以在`/themes/next/source/lib`下查看到了多了 `fancybox` ，然后在主题配置文件中开启对其的支持，如下即可

```
fancybox: true
```

### 24. 看板娘添加

**步骤1：**安装相应模块：`npm install hexo-helper-live2d --save`

**步骤2：**选择看板娘安装

可供选择的看板娘有如下：`https://github.com/xiazeyu/live2d-widget-models`

预览网站：`https://huaji8.top/post/live2d-plugin-2.0/`

我这里选择了一个黑色的小猫hijiki（之前的博客用的就是这个），相对应的是：`live2d-widget-model-hijiki`，然后安装一下：`npm install live2d-widget-model-hijiki`

**步骤3：**修改框架配置文件`_config.yml`

```yml
# Live2D
## https://github.com/EYHN/hexo-helper-live2d
live2d:
  enable: true
  scriptFrom: local # 默认
  pluginRootPath: live2dw/  # 插件在站点上的根目录(相对路径)
  pluginJsPath: lib/        # 脚本文件相对与插件根目录路径
  pluginModelPath: assets/  # 模型文件相对与插件根目录路径
  # scriptFrom: jsdelivr # jsdelivr CDN
  # scriptFrom: unpkg # unpkg CDN
  # scriptFrom: https://cdn.jsdelivr.net/npm/live2d-widget@3.x/lib/L2Dwidget.min.js # 你的自定义 url
  tagMode: false # 标签模式, 是否仅替换 live2d tag标签而非插入到所有页面中
  debug: false   # 调试, 是否在控制台输出日志
  model:
    use: live2d-widget-model-hijiki
    scale: 1
    hHeadPos: 0.5
    vHeadPos: 0.618
    # use: live2d-widget-model-wanko # npm-module package name
    # use: wanko # 博客根目录/live2d_models/ 下的目录名
    # use: ./wives/wanko # 相对于博客根目录的路径
    # use: https://cdn.jsdelivr.net/npm/live2d-widget-model-wanko@1.0.5/assets/wanko.model.json # 你的自定义 url
  display:  # 关于显示的一些参数
    superSample: 2
    width: 150
    height: 300
    position: right
    hOffset: 0
    vOffset: -20
  mobile:
    show: true # 是否在移动设备上显示
    scale: 0.5 # 移动设备上的缩放       
  react:
    opacityDefault: 0.7
    opacityOnHover: 0.8
# -----------------------------------------------------
```

**步骤4：**`hexo g && hexo s`，即重新布署一下即可。

> 最为基础的使用，感觉已经够了，后续如果要加强的话再来完善一下

### 参考

https://mp.weixin.qq.com/s?__biz=MzU4NDcxNjQ2Ng==&mid=2247485623&idx=1&sn=edd7a6d3c69d09493b3920edbbf6b223&scene=19#wechat_redirect

https://mp.weixin.qq.com/s?__biz=MzU4NDcxNjQ2Ng==&mid=2247485676&idx=1&sn=2f921b74c98e736748c7296ccda6eb14&scene=19#wechat_redirect

https://mp.weixin.qq.com/s?__biz=MzU4NDcxNjQ2Ng==&mid=2247485737&idx=1&sn=adfce04503d6e758a00281657972fa62&scene=19#wechat_redirect

> 前23个内容基本是参考了[公众号：管家小e——网站搭建](https://mp.weixin.qq.com/mp/homepage?__biz=MzU4NDcxNjQ2Ng==&hid=1&sn=debf3376e6c934da259097b1886297d7&scene=18#wechat_redirect)，因为起初就是跟着这个教程搭起来的:smile:

