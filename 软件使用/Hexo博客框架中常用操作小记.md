# Hexo博客框架中常用操作小记

> 本文就记录一些日常使用Hexo框架写博文过程中常用的操作等。

先给出一个官方文档：https://hexo.io/zh-cn/docs/

### 日常使用

> 这部分就记录一下日常写文章的一些操作吧

#### 创建文章

> **日常其实更多的是手动创建**

使用命令：`hexo new [layout] <title>`

> 当没有指定选择layout的话，会默认使用框架配置文件中`default_layout`的参数，即：`default_layout: post`
>
> 其他参数：
>
> | 参数              | 描述                                          |
> | :---------------- | :-------------------------------------------- |
> | `-p`, `--path`    | 自定义新文章的路径                            |
> | `-r`, `--replace` | 如果存在同名文章，将其替换                    |
> | `-s`, `--slug`    | 文章的 Slug，作为新文章的文件名和发布后的 URL |

常用的话就是创建正常的博客（就是在`/source/_posts`目录下的内容）：

```bash
$ hexo new -p 软件使用/Hexo博客框架中常用操作小记 Hexo博客框架中常用操作小记
INFO  Validating config
INFO  Created: ~\Desktop\test\Blog\source\_posts\软件使用\Hexo博客框架中常用操作小记.md
```

上述操作就会在 `/source/_posts/软件使用/`的目录下创建文件`Hexo博客框架中常用操作小记.md`，文件内容如下：（**不过日常使用更多的还是直接新建一下然后给MD文件加个头就好了:smirk:**）

```
---
title: Hexo博客框架中常用操作小记
date: 2021-12-20 18:34:55
tags:
---
```

#### 创建草稿

基本和上述一致就是在多加一个`[layout]`参数，如下：

```bash
ZhuCC@LAPTOP-F6OAPAAB MINGW64 ~/Desktop/test/Blog (master)
$ hexo new draft 测试草稿
INFO  Validating config
INFO  Created: ~\Desktop\test\Blog\source\_drafts\测试草稿.md
```

上述命令就会在`/source/_drafts/`文件夹下创建一个名为`测试草稿.md`的文件，同时的话创建的草稿文件是不会显示在页面上的（可以放一些临时文件），当需要发布适，可以通过`hexo publish [layout] <title>`命令，这里把上述创建的草稿给发布一下，如下：

```
ZhuCC@LAPTOP-F6OAPAAB MINGW64 ~/Desktop/test/Blog (master)
$ hexo publish 测试草稿
INFO  Validating config
INFO  Published: ~\Desktop\test\Blog\source\_posts\测试草稿.md
```

然后在`/source/_drafts/测试草稿.md`就到了`/source/_post/测试草稿.md`

#### 图片支持

图片可以放到`/source/images`下，这样在文章中通过`[](/images/图片名)`就能在发布的时候被访问到了，但是存在两点问题：

1. 本地的 Typora 中无法看到；
2. 全部放到`/source/images`下的话就太杂了，计划的是这个文件夹下仅放一些通用的图片

**解决**：

1. 修改框架配置文件，设置`post_asset_folder: true`

2. 安装插件：`npm install https://github.com/CodeFalling/hexo-asset-image --save`

3. 修改文件：`/node_modules/hexo-asset-image/index.js`文件内容为：

   ```js
   'use strict';
   var cheerio = require('cheerio');
   
   // http://stackoverflow.com/questions/14480345/how-to-get-the-nth-occurrence-in-a-string
   function getPosition(str, m, i) {
     return str.split(m, i).join(m).length;
   }
   
   var version = String(hexo.version).split('.');
   hexo.extend.filter.register('after_post_render', function(data){
     var config = hexo.config;
     if(config.post_asset_folder){
       	var link = data.permalink;
   	if(version.length > 0 && Number(version[0]) == 3)
   	   var beginPos = getPosition(link, '/', 1) + 1;
   	else
   	   var beginPos = getPosition(link, '/', 3) + 1;
   	// In hexo 3.1.1, the permalink of "about" page is like ".../about/index.html".
   	var endPos = link.lastIndexOf('/') + 1;
       link = link.substring(beginPos, endPos);
   
       var toprocess = ['excerpt', 'more', 'content'];
       for(var i = 0; i < toprocess.length; i++){
         var key = toprocess[i];
    
         var $ = cheerio.load(data[key], {
           ignoreWhitespace: false,
           xmlMode: false,
           lowerCaseTags: false,
           decodeEntities: false
         });
   
         $('img').each(function(){
   		if ($(this).attr('src')){
   			// For windows style path, we replace '\' to '/'.
   			var src = $(this).attr('src').replace('\\', '/');
   			if(!/http[s]*.*|\/\/.*/.test(src) &&
   			   !/^\s*\//.test(src)) {
   			  // For "about" page, the first part of "src" can't be removed.
   			  // In addition, to support multi-level local directory.
   			  var linkArray = link.split('/').filter(function(elem){
   				return elem != '';
   			  });
   			  var srcArray = src.split('/').filter(function(elem){
   				return elem != '' && elem != '.';
   			  });
   			  if(srcArray.length > 1)
   				srcArray.shift();
   			  src = srcArray.join('/');
   			  $(this).attr('src', config.root + link + src);
   			  console.info&&console.info("update link as:-->"+config.root + link + src);
   			}
   		}else{
   			console.info&&console.info("no src attr, skipped...");
   			console.info&&console.info($(this));
   		}
         });
         data[key] = $.html();
       }
     }
   });
   ```

通过上述设置，则每次在创建新文章时，会同时创建一个和文章同名的文件夹，所有的图片就放到这里去好了，在应用是直接通过如下：`[图片名](文章名/图片名)`，用本文举个例子：

> 本文的题目为：`Hexo博客框架中常用操作小记`，在和`Hexo博客框架中常用操作小记.md`同级的目录下创建文件夹：Hexo博客框架中常用操作小记，然后在里面放文件，如下：
>
> ![图片引用](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206252045783.PNG)
>
> 然后在需要插入的图片的地方：`![图片引用](image/图片引用.PNG)`完成图片插入（这里的话设置一下 Typora 的偏好在图片选项中选择优先相对路径，这样图片放好后直接复制粘贴一下就可以了）

> 参考：https://blog.csdn.net/xjm850552586/article/details/84101345

#### 发布

1. 生成：`hexo generate / hexo g`；
2. 本地调试：`hexo server / hexo s`;
3. 布署：`hexo deploy / hexo d`;
4. 清空缓存和静态文件：`hexo clean / hexo c`、

日常使用就是：

* 写完内容后首先：`hexo g && hexo s`，进行本地查看效果如何；
* 然后发布一下：`hexo g && hexo d`；
* 最后清空一下缓存：`hexo c`；
* 最后用TortoiseGit同步博客内容到Gitee中。

### Front-matter配置

> 详见：https://hexo.io/zh-cn/docs/front-matter

正常文章一般采用如下文章头而不是默认的是：

```
---
title: 这里就是文章标题
date: xxx 10:24:00
updated: xxx 10:24:00
categories: 
		- xxx
tags:
		- xxx
password: mima
mathjax: true
---

前言内容（会显示在主页上的，而在`<!--more-->`的内容就是正文不会显示在首页上

<!--more-->
```

一般的话就是采用上述的内容，然后按需删减比如说这个密码和是否启用公式渲染；

其他需要注意的是，如果有多级分类，应该这样配置：

```
categories: 
		- [分类1]
		- [分类2]
```

这样的话`分类1`和`分类2`是同一级别的分类（目前我是没有上下级分类的需求，如需要的话详见上方的官方文档。

### 博文压缩(存在问题废弃:exclamation:)

> Hexo发布的文章其实都为静态网页资源，可以通过 gulp 进行一个压缩来提升访问速度
>
> gulp GitHub链接如下：https://github.com/gulpjs/gulp

1. 安装 gulp，如下：

   ```
   npm install gulp --save
   npm install gulp-minify-css --save
   npm install gulp-uglify --save
   npm install gulp-htmlmin --save
   npm install gulp-htmlclean --save
   npm install gulp-imagemin --save  （这个先别装了有问题，后面有说明）
   ```

   安装完成后在`package.json`中多了一下内容：

   ```json
   "dependencies": {
       "gulp": "^4.0.2",
       "gulp-htmlclean": "^2.7.22",
       "gulp-htmlmin": "^5.0.1",
       "gulp-imagemin": "^8.0.0",
       "gulp-minify-css": "^1.2.4",
       "gulp-uglify": "^3.0.2",
   	// ...
   }
   ```

2. 然后在根目录下创建`gulpfile.js`文件，内容如下：

   ```js
   // 引入需要的模块
   var gulp = require('gulp');
   var minifycss = require('gulp-minify-css');
   var uglify = require('gulp-uglify');
   var htmlmin = require('gulp-htmlmin');
   var htmlclean = require('gulp-htmlclean');
   var imagemin = require('gulp-imagemin');
   
   // 压缩public目录下所有html文件, minify-html是任务名, 设置为default，启动gulp压缩的时候可以省去任务名
   gulp.task('minify-html', function() {
       return gulp.src('./public/**/*.html') // 压缩文件所在的目录
           .pipe(htmlclean())
           .pipe(htmlmin({
               removeComments: true,
               minifyJS: true,
               minifyCSS: true,
               minifyURLs: true,
           }))
           .pipe(gulp.dest('./public')) // 输出的目录
   });
   
   // 压缩css
   gulp.task('minify-css', function() {
       return gulp.src(['./public/**/*.css','!./public/js/**/*min.css'])
           .pipe(minifycss({
               compatibility: 'ie8'
           }))
           .pipe(gulp.dest('./public'));
   });
   // 压缩js
   gulp.task('minify-js', function() {
       return gulp.src(['./public/**/.js','!./public/js/**/*min.js'])
           .pipe(uglify())
           .pipe(gulp.dest('./public'));
   });
   // 压缩图片
   gulp.task('minify-images', function() {
       return gulp.src(['./public/**/*.png','./public/**/*.jpg','./public/**/*.gif'])
           .pipe(imagemin(
           [imagemin.gifsicle({'optimizationLevel': 3}), 
           imagemin.mozjpeg({'progressive': true}), 
           imagemin.optipng({'optimizationLevel': 5}), 
           imagemin.svgo()],
           {'verbose': true}))
           .pipe(gulp.dest('./public'))
   });
   // gulp 4.0 适用的方式
   gulp.task('default', gulp.parallel('minify-html','minify-css','minify-js','minify-images'));
   ```

3. 使用，即在原先的`hexo g`命令后添加命令`gulp default / gulp`，之后在部署`hexo d`

**上述gulp-imagemin内容存在一些问题：（感觉是版本太高了）**

1. `gulp-imagemin` 插件安装有问题，解决如下：

   > 设置npm为cnpm：`npm install -g cnpm --registry=https://registry.npm.taobao.org`
   >
   > 下面操作完之后还是再修改回来吧：`npm uninstall cnpm -g`，即卸载 cnpm

2. gulp报错，也是`gulp-imagemin`导致的，暂时没解决，若要用到话把上面`gulpfile.js`文件中关于 `gulp-imagemin` 都删除就行了

   **已经解决了（果然是版本高了）：**

   用安装的 cnpm 安装指定版本的 `gulp-imagemin`，不过要卸载一下之前版本的 `gulp-imagemin`；

**总结解决问题操作如下（即安装gulp-imagemin）：**

```bash
# 替换npm为cnmp
$ npm install -g cnpm --registry=https://registry.npm.taobao.org
# 卸载已经安装的gulp-imagemin
$ npm uninstall gulp-imagemin
# 但凡遇到npm uninstall的情况，顺带清理一下包：npm prune，因为uninstall不会清理node_module中内容
$ npm prune
# cnpm 安装指定版本的gulp-imagemin
$ cnpm install gulp-imagemin@7.1.0 --save
# 安装完成后即可成功的进行gulp
$ hexo clean && hexo g && gulp && hexo d
# 成功部署后恢复npm
npm uninstall cnpm -g
```

> https://blog.csdn.net/GerZhouGengCheng/article/details/106074545
>
> https://blog.csdn.net/qq_36691432/article/details/106994577
>
> https://blog.csdn.net/qq_42984640/article/details/103936738

**废弃原因**：在装了`gulp-imagemin@7.1.0`后，`node_modules`中的内容炸了，出现了巨多的东西...不太好同步，而且每次 gulp 的时间挺长的...以上两点原因让我一怒之下把上面的内容全删了！

### 文件加密访问

是这样，因为有些文章写了一些个人的经历；但又不是很想被人瞧见，就打算给文章上个锁这样的，操作如下：

首先安装插件：`hexo-blog-encrypt`，通过如下命令：

```bash
$ npm install --save hexo-blog-encrypt
```

**方式一**：在需要被加密的文件头中加入 `password` 内容，如下：

```
---
title: 求学之路：小学篇
date: 2021-11-07 10:24:10
password: hello
categories: 
		- 个人经历
tags:
		- 个人经历
		- 成长史
---
```

> 上面配置的比较基础，没啥提示（其实也够了，如果要人性化一些就如下）
>
> ```yml
> ---
> title: Hello World
> tags:
> 	- 作为日记加密
> date: 2016-03-30 21:12:21
> password: mikemessi
> abstract: 有东西被加密了, 请输入密码查看.
> message: 您好, 这里需要密码.
> wrong_pass_message: 抱歉, 这个密码看着不太对, 请再试试.
> wrong_hash_message: 抱歉, 这个文章不能被校验, 不过您还是能看看解密后的内容.
> ---
> ```
>
> 下面是个人的配置：
>
> ```yml
> title: 求学之路：小学篇
> date: 2021-11-07 10:24:10
> categories: 
> 		- 个人经历
> tags:
> 		- 个人经历
> 		- 成长史
> password: mikemessi
> message: 加密文章的啦~~~
> wrong_pass_message: 错啦错啊，密码错啦~~~
> wrong_hash_message: Sorry啦，这个文章不能被校验的哟~~~
> ```

**方式二（我用的）**：在框架配置文件中加入以下内容，然后在文章就只要加入 `password: xxx` 就可以有提示等功能咯

```yml
# Security
encrypt: # hexo-blog-encrypt
  abstract: 有东西被加密了, 请输入密码查看.
  message: 您好, 这里需要密码.
  tags:  # 标签加密功能
  - {name: tagName, password: 密码A}
  - {name: tagName, password: 密码B}
  wrong_pass_message: 抱歉, 这个密码看着不太对, 请再试试.
  wrong_hash_message: 抱歉, 这个文章不能被校验, 不过您还是能看看解密后的内容.
```

> 我的使用如下，在框架配置文件中加入以下内容：
>
> ```yml
> # Security文章加密配置
> encrypt: # hexo-blog-encrypt
>     # abstract: 有东西被加密了, 请输入密码查看.
>     message: 加密文章的啦~~~
>     tags:  # 标签加密功能，好像有点问题说我密码错误
>   - {name: 加密文章, password: ???}
>   # - {name: tagName, password: 密码B}
>   wrong_pass_message: 错啦错啊，密码错啦~~~
>   wrong_hash_message: Sorry啦，这个文章不能被校验的哟~~~
> # -----------------------------------------------------
> ```
>
> 然后在需要加密的文章头上加：`password: xxx`；即可实现文章加密功能（仅用tags就说我密码错误很奇怪）

然后重新布署一下即可。

> 更多内容可参考作者文档：https://github.com/D0n9X1n/hexo-blog-encrypt

### 支持公式+Emoji

> 之前的写的博客中存在很多的公式，在迁移过程中遇到了各种渲染问题，换了各种渲染器...真的搞了我很久差点放弃了！
>
> 目前网上很多的修改方式其实都比较老了基本都是不能用的...纯属坑爹玩意儿！！！

打开Hexo更目录下的 `package.json` 文件，看此时你的dependencies下的渲染依赖(带renderer的)默认如下：

```json
"hexo-renderer-ejs": "^1.0.0",
"hexo-renderer-marked": "^4.0.0",
"hexo-renderer-stylus": "^2.0.0",
```

按照主题配置文件中的要求（下面有写公式依赖`hexo-renderer-pandoc`这个渲染器），因此的话删除掉渲染器`hexo-renderer-marked`，通过如下命令：

```bash
$ npm uninstall hexo-renderer-marked -- save
```

> 卸载成功后可以看到在 `package.json` 中的 `hexo-renderer-marked` 已经没了

然后再次安装上述渲染器：`hexo-renderer-pandoc`，通过以下命令

```bash
$ npm install hexo-renderer-pandoc --save
```

成功后可以看到在`package.json`中的渲染器为如下：

```json
"hexo-renderer-ejs": "^1.0.0",
"hexo-renderer-pandoc": "^0.3.0",
"hexo-renderer-stylus": "^2.0.0",
```

然后在主题配置文件中有以下默认内容：

```yml
# Math Formulas Render Support
math:
  # Default (true) will load mathjax / katex script on demand.
  # That is it only render those page which has `mathjax: true` in Front-matter.
  # If you set it to false, it will load mathjax / katex srcipt EVERY PAGE.
  per_page: true  # true表示只有在需要在页面手动开启公式渲染

  # hexo-renderer-pandoc (or hexo-renderer-kramed) required for full MathJax support.
  mathjax:  # 从上面的package.json中可以看到，这里已经有包hexo-renderer-pandoc了，因此支持公式
    enable: false  # 是否开启公式，这里要选择开启即设定为true，这时候其实就可以正常使用了！
    # See: https://mhchem.github.io/MathJax-mhchem/
    mhchem: false

  # hexo-renderer-markdown-it-plus (or hexo-renderer-markdown-it with markdown-it-katex plugin) required for full Katex support.
  katex:  # 并没有用这个插件，不用过管他（之前被这个坑了很久）
    enable: false
    # See: https://github.com/KaTeX/KaTeX/tree/master/contrib/copy-tex
    copy_tex: false
```

上面的注释中我已经清楚的说明了，在配置完成后，在每个需要启用公式的页面的中开启公式支持`mathjax: true`，如下：

![公式支持](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206252045940.PNG)

这时候原先不支持公式渲染失败的部分就成功了，如下：

![公式渲染](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206252046759.PNG)

> 渲染成功，不需要改TM的其他配置文件！不会出现渲染一半的情况​ :rage:

支持Emoji表情，默认的渲染器是不支持Emoji表情的，现在换了 `hexo-renderer-pandoc` 之后即可支持，在框架配置文件末加上如下内容：

```
# hexo-renderer-pandoc (https://github.com/wzpan/hexo-renderer-pandoc)
pandoc:
  extra:
    # - toc: # will be passed as `--toc`. Note the colon，如果不需要自动生成目录无需添加此行
  extensions:
    - '+hard_line_breaks'
    - '+emoji'
    - '-implicit_figures'
  template:
    # ./pandoc_template.html # 如果不需要自动生成目录无需添加此行
# -----------------------------------------------------
```

这时原先的 `:smile:` 就变成了 :smile: 了

> 给出一个 Emoji 参考网址：https://www.webfx.com/tools/emoji-cheat-sheet/



