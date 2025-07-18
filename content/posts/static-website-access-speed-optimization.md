---
title: "静态网站访问速度优化技巧"
date: 2019-07-21T10:00:00+08:00
toc: true
draft: true
tags:
  - 技术
  - 学习笔记
  - 建站
images:
  - https://blog-assets-1253422097.file.myqcloud.com/images/2019-07-21-static-website-access-speed-optimization/banner.jpg
---

我发现现在很多计算机相关专业的人博客里面写的文章满满的都是算法、机器学习、大数据、云计算什么的，感觉像我这么菜的人实在是看不下去。于是打算写点接地气的专业相关的文章，我发现我对博客搭建情有独钟，写了不少相关的文章，等用一段时间 Hexo 我可能会再写一篇 Hexo 相关的，这篇文章我们先来聊聊如果你有了一个博客，你可以怎么优化它的访问速度。

就这个问题，即使是对 Web 有一定了解的初学者，都不一定回答得上来。因为很多人学 Web 嘛，就是什么前端、后端，套几个框架，按着文档的示例，就能写起来，我以前也是这样学习的。至于一个网站，是怎么部署的，写的东西是怎么变成浏览器里面可以看到的东西的，可能完全没有概念。对于中间有什么可以优化的环节，可能也完全没有概念。这其实很正常，你看有哪个学校会教你怎么搭建网站的，有哪个研究生选题是搭建网站相关的，一大堆的不都是什么机器学习、大数据、云计算方向的。就像我一个老师说的 “很多人觉得，搭建网站这个东西呢，不够理论，不够系统，不够抽象。” ，实际上呢，是不是可以很理论、很系统、很抽象，我也不知道，但至少，这个东西不是很简单的。

> 静态网站就是网站上所有资源都是静态的的网站

这篇文章对静态网站访问速度优化这个问题的讨论，其实是基于博客访问速度优化的。因为说 “个人博客访问速度优化” 好像太局限，而实际上个人博客和其他静态网站又没有太多不同，所以将标题定为 “静态网站访问速度优化技巧” 。

很多人可能不知道 “静态网站” 是什么意思，即使是对 Web 有一定了解的初学者，也不一定正确认识了这个概念。静态网站并不是不会动的网站，简单来说静态网站就是网站上所有资源都是静态的的网站，比如你正在浏览的这个网站就是一个静态网站，你所访问的几乎所有资源，包括这里的图片、文章都是在你访问之前就已经存在的文件，当你访问时，服务器只需要将你需要的文件发送给你，你就看到了这样的页面。说几乎，是因为这里的留言和浏览量信息并不是静态资源，他们是需要服务器在请求到来时才动态生成的。类似的还有搜索引擎，搜索引擎并不会提前准备好每一个可能被搜索的关键字对应的页面，这也不可能做到。相反，它会在你输入关键字，敲下回车后，服务器拿着你的关键字去检索应该展示给你的网站名字以及摘要、URL等，然后生成一个页面，返回给你，也就是你随后看到的页面。你输入不同的关键字将会得到不同的页面，你能输入的关键字的可能情况近于无穷多，服务器也不可能提前准备好无穷多的页面，所以这些页面都是动态生成的。但是本站不一样，本站文章数量是有限的，而且不多，你能看到的页面跳转来跳转去就那么几页，数量有限而且不多，而且你不管什么时候看都是一样的（除非我更新了我的博客），所以就可以一次性生成好全部页面，放在服务器，供需要的人取阅。

所以其实静态网站访问速度的优化，就是文件加载速度的优化。

作为这个话题的开始，我们先看几张图片。

首先是我所处网络环境的测速结果

![speedtest](https://blog-assets-1253422097.file.myqcloud.com/images/2019-07-21-static-website-access-speed-optimization/speedtest.jpg)

![keybrl_loading](https://blog-assets-1253422097.file.myqcloud.com/images/2019-07-21-static-website-access-speed-optimization/keybrl_loading.jpg)

![dapao_loading](https://blog-assets-1253422097.file.myqcloud.com/images/2019-07-21-static-website-access-speed-optimization/dapao_loading.jpg)

图一是我所处网络环境在 [speedtest.cn](http://www.speedtest.cn/) 的测速结果，表明我测试时有 20Mbps 左右的下行带宽。图二是我博客某篇文章加载的时序图，图三是某个朋友博客首页加载的时序图。

图二、图三其实就是浏览器的开发者工具，你也可以现在按一下 F12 看看。这个图怎么看呢，首先看右上角，勾选了 “禁用缓存” 说明下面所有资源都是从服务器加载的，并不是本地缓存。然后看左下角，有 “61个请求 已传输16.31 MB/16.04 MB 完成 9.04秒 DOMContentLoaded 400毫秒 load 9.04秒” 的字样，说明加载这个页面一共需要 61 个资源，总共下载了 16.31MB 的资源，总用时 9.04 秒，DOM内容加载用了 400 毫秒，这是加载我博客某篇文章的情况（其实就是 [软路由与 NAS (3) - 外壳](router3-shell.md) （我们后面就说 “我博客的情况” ））。

作为对比，图三表示的是加载 [大炮's Blog](https://blog.dapaostudio.com/) 这个页面的情况（我们后面就说 “大炮的情况” ）。可以看到加载了 7.13MB 花了 2.99 分钟。也就是说，我博客的加载速度大概是他的 45 倍。一般来说 1 秒内的响应速度是可以接受的， 3 秒以上的加载就需要进度条、加载动画等安慰剂来忽悠，几十上百秒的话，几乎是不能接受的。我的博客虽然花了 9 秒，但是你访问一下你就会知道，你看到页面加载完不到 1 秒，因为后面都在加载图片，而你屏幕并不能一次显示完整个页面，你能看到的部分很快就加载完成了。

是什么产生了如此悬殊的差距呢？有什么办法可以改善这种情况呢？特别是对于一个有 20 兆带宽的用户来说，加载一个不大的页面需要 3 分钟是不可能接受的。

## 页面总大小

最直观的加快访问速度的方法就是减小页面的总大小嘛。对于一般人来说，最简单的方法就是缩小和压缩图片。

写博客肯定少不了很多图片，如果是直接插入从手机、相机传过来的原图，那一张图片就有几兆到十几兆。但其实插在文章中的图片不需要那么大。按照正常人的屏幕分辨率和系统界面缩放大小，全屏浏览器的等效宽度一般不会超过 2000px 。即使是电视，用了 4K ，没有缩放，人坐在沙发上也很难看出上面全屏显示的 1440p 图像和 4K 图像有什么区别，实际上上面放的电视节目的分辨率都很少能达到 1080p 。

所以我一般都将我博客中的图片缩小为宽度 1920px 左右的，如果是 16:9 就 1920 * 1080 ,如果是 4:3 的就 1920 * 1440 。就用 ps 批量处理一下。我这是按照图片全屏显示来算的，如果图标这种小图，要根据实际情况缩小。作为参考，我 [照片墙](/gallery/) 里面的图片，小图短边 720px ，点开的大图短边 1440px 。其实我觉得已经挺奢侈了。

还有就是压缩质量，特别是 jpeg 格式的图片，同样的分辨率，大小可以差很大。jpeg的有损压缩，会把位置和颜色都比较相近的色块变成一样的颜色，压缩质量高的话就细节保留更多，质量低的话照片细节就损失多一点。我一般如果不重要的图片都是在 ps 中调成质量是 8 ，如果要求高一些的风景照、人像什么的就调成 10 。再往上加的话图片大小会翻几番，而人眼又不怎么看得出差别。

这样处理之后一张图片大概是 100KB ，从原图的十兆左右，相差 100 倍左右。

## 大文件扔 OSS

实际上，上面提到的大炮的情况并不是因为页面总大小很大，他非常非常尽力地压缩了图片的大小和质量，但是你会发现速度慢根本不是页面大小的问题，大炮一个 7MB 的页面加载要 3 分钟，而我 16MB 的页面只需要 9 秒。所以这里显然还有别的问题。

很多人个人博客嘛，就用丢到 GitHub 上，的 GitHub Pages 服务来部署。像 Jekyll 用户一般就是把源文件丢给 GitHub Pages ，让它来构建、部署，因为 GitHub Pages 支持对 Jekyll 项目的构建。而 Hexo 用户么一般那就是在本地构建好再部署到 GitHub 仓库的 `gh-pages` 分支，这时 GitHub Pages 就仅仅相当于一个静态文件服务器。或者大家可能还有别的类似平台，我就单拿 GitHub Pages 来说了。

GitHub Pages 是免费的，带宽肯定不用想能有多高，而且国内经常还网络不好，你想从里面加载好几 MB 的文件肯定是非常缓慢的。但是自己搭服务器吧，好像也很麻烦，而且还贵，而且带宽也是相当昂贵，即使是腾讯云学生优惠，也要 120 一年，还只有 1Mbps 的带宽（实际一般会比标称的大）。

那有什么办法可以像原来那样可以方便部署在 GitHub Pages 上，但是又能有效加快访问速度呢。那就是将大文件扔到别的地方。这里推荐的是 OSS （或者腾讯云叫 COS ）。OSS 就是 对象 存储 服务 （COS 就是 云 对象 存储，反正就是一个意思），什么是对象存储呢，就是存储对象，这玩意儿什么对象都可以存储，因为对象实际上就是内存里面的二进制数据，通过 OSS 提供的编程接口可以很方便地存取。它和数据库不同，数据库存储的是关系型数据，数据需要组织成合适的结构，而且对存储的数据类型是有限制的。还有一些非关系型数据库，就是数据的结构和类型不一样的。但是对象存储可以说是一种，没有结构的或者说只有很简单结构的，类型没有限制的存储方式。一个整数、一个结构体、一个字符串，什么都可以。

所以当然啦，文件系统上存储的文件也可以，因为他们也是一堆二进制数据而已嘛。而且如果你只是存文件的话，你也不需要编程与之交互，阿里云和腾讯云都为它们的对象存储服务提供了基于 Web 的图形界面可以让你上传文件。

而且 OSS 还有一个好处是，里面的对象通过 url 标识，所以你就可以像下载一个文件一样通过一个 url 从里面得到一个文件。

而且 OSS 一般只是按照存储和流量计费的，至于带宽，是 “尽力而为” 服务，不过这一般都能跑满你的带宽了，就不需要再纠结从里面下载文件具体能有多快了。

下面我们以阿里云的 OSS 来演示一下具体操作。注册一个阿里云账号，登录控制台，点开左上角菜单，就能看到 “对象存储 OSS”

![aliyun_console](https://blog-assets-1253422097.file.myqcloud.com/images/2019-07-21-static-website-access-speed-optimization/aliyun_console.jpg)

点击进入对象存储的控制台，第一次进去应该会有确认开通服务的提示。左侧 bucket 列表，点一下 + 号，新建一个 bucket

![oss_console](https://blog-assets-1253422097.file.myqcloud.com/images/2019-07-21-static-website-access-speed-optimization/oss_console.jpg)

![create_bucket](https://blog-assets-1253422097.file.myqcloud.com/images/2019-07-21-static-website-access-speed-optimization/create_bucket.jpg)

起个名字，选个地域，存储类型选 “标准存储” ，读写权限选 “公共读” ，无服务端加密，日志看需要开吧。

如果有买了某个地区的流量包或者存储包，就把地域选择在那个区域，如果没有这方面的限制的话就选个离目标用户比较近的吧，就是哪里的人看你的网站多就选哪里。或者也可以喜欢哪个城市就选哪里，这个其实对实际体验，没有太大影响。

开完之后对象存储控制台左侧 bucket 列表里就会有新创建的 bucket 。点进去，点文件管理

![bucket_console](https://blog-assets-1253422097.file.myqcloud.com/images/2019-07-21-static-website-access-speed-optimization/bucket_console.jpg)

里面的操作就和网盘差不多了，自己创建目录，上传文件

然后怎么访问里面的文件呢，回到 bucket 的 “概览”  ，可以看到 bucket 的外网访问域名

![bucket_host](https://blog-assets-1253422097.file.myqcloud.com/images/2019-07-21-static-website-access-speed-optimization/bucket_host.jpg)

比如说你的得到的是 `whatever.oss-cn-hangzhou.aliyuncs.com` 你 bucket 里面有个文件夹叫 `images` 里面有张图片叫做 `hhh.png` ，那么这张图片的 url 就是 `https://whatever.oss-cn-hangzhou.aliyuncs.com/images/hhh.png` 。实际上就是 bucket 域名后面加文件路径， http 和 https 都可以，看需要。

然后你在文章中就可以用这个地址来引入图片了。

如果用的是 jekyll ， jekyll 可以在文章中使用配置文件的的字段，那你可以在配置文件 `_config.yml` 里加一行

```yml
oss_url: https://whatever.oss-cn-hangzhou.aliyuncs.com
```

然后在文章中用

![jekyll_template](https://blog-assets-1253422097.file.myqcloud.com/images/2019-07-21-static-website-access-speed-optimization/jekyll_template.png)

就可以访问到刚刚那张图片（为什么我要用截图，而不直接写出来呢，因为智障 Hexo 会试图解析双大括号，即使他放在行内式代码块中）。这样以后要换 bucket ，或者要给这个 bucket 加 cdn ，用自己的域名，都可以很方便修改。

如果你用的是 Hexo ，那很可惜，目前我看过的对这方面的支持的插件，都不能完全正常地工作，建议还是直接在文章中写完整 url ，如果需要替换域名的话，用编辑器的 “搜索/替换” 功能吧。

除了图片，页面使用的 CSS 、 JS 、字体文件都可以扔到 OSS ，如果你知道去哪里修改这些资源的 url 的话。而且如果你要这样做，建议你修改 bucket 的同源策略，在 bucket 控制台上方导航栏的 “基础设置” -> “跨域设置” ，添加一条这样的规则

![cors_setting](https://blog-assets-1253422097.file.myqcloud.com/images/2019-07-21-static-website-access-speed-optimization/cors_setting.jpg)

他会在响应头加上 `Origin: *` 以确保资源可以被任何来源的请求获取。如果你确定你的资源只会用于你一个 `blog.keybrl.com` 域名的博客的话，你也可以把 `*` 换成 `blog.keybrl.com` 。

如果是只有图片，那不需要设置跨域规则，因为这种直接在页面上写 url 请求的资源，都不算跨域。但如果要引入 js 或者 css ，他们可能会要引入其他 js 脚本或者字体文件，这时就会被浏览器同源策略限制，需要加上面说的设置才能正常使用。

除了 OSS ，也可以用一下图床什么的，原理都是类似的。但我觉得 OSS 会快一些，特别是比某些免费的图床。而且 OSS 能放的不只是图片，任何文件都可以放进去。一般文件放进去，在页面上放个下载链接，点一下就能下载，速度和便利性都比某度网盘好。视频文件，直接在文章中用 `<video>` 标签引入，可以在页面上直接播放。总之很适合放各种页面上的大文件就是了。

最彻底的重构之后，比如你正在浏览的这个博客，只有一个几十 kB 的 .html 文件是单独部署的，其他所有资源都在 OSS 上。这可以保证页面的加载时间不会很过分。

试着这么去重新布置一下，你会发现能极大提高博客访问速度。

进一步的优化可以看下面关于 CDN 的介绍，不过我们先来关注一下那个剩下的 .html 文件的问题。

## DOMContent

在文章开头的两张加载时序图中，左下角显示的时间，有一项单独的 “DOMContentLoaded: 400毫秒” 。而大炮的情况在这一项上是 14.53 秒，相差 36 倍。这一项是什么呢，这个差别影响很大吗？

DOMContent 直接翻译一下就是 文档 对象 模型 内容，但其实熟悉 Web 的同学都知道， DOM 嘛，就是 DOM 嘛，没必要翻译。具体是什么呢，简单来说就是加载时序图第一项的那个 .html 文件（但其实不是，应该是根据这个文件生成的对象模型，只是简单这么理解而已）。一般博客里面一个页面的 HTML 也就几十 kB 。它是由 Jekyll 、 Hexo 这种静态网站生成器根据你写的 .md 文件和一些模板，生成的。你如果右键这个页面点 “查看页面源代码” （或者类似描述）你就能看到它，都是纯文本。里面的图片和其他静态资源，都是通过在里面标注 url ，由浏览器另外加载的。

都是纯文本而已，就几十 kB 大，那在一个几 MB 到十几 MB 大小的页面里面应该无足轻重啊。事实当然不是这样的。

你看大炮页面加载的时序图，你会发现前十几秒，其他所有文件都没有开始加载，仅仅在等待一个几十 kB 的文件的加载。因为其他所有资源的 url 都写在这个几十 kB 的 HTML 文件里，浏览器不知道 url 怎么加载其他资源呢，是吧。

在大炮的情况中，这个 HTML 文件加载慢似乎影响也不大，无非就是在 2 分 40 秒左右的基础上加上了前面等待的 14 秒，达到 2.99 分钟而已。

但如果你想象一下是我的博客页面，其他所有资源加载只需要 9 秒，但是开始这 9 秒之前要先等待 14 秒加载一个只有几十 kB 大小的文件，这完全不合理。而且如果你经过了上面那个 “大文件扔 OSS” 的步骤之后，如果 HTML 仍然部署在 GitHub Pages 上，你就会遇到这种情况。虽然从几分钟削减到 23 秒已经是一个巨大的进步，但是你肯定想要更进一步。

方法么也很简单粗暴，主要就是两种思路。

第一种就是不用 GitHub Pages 部署了，全部文件都扔到 OSS 上。用 Hexo 的话执行 `hexo g` 就可以进行一次本地构建，将结果放到 `public/` 文件夹下，用其他工具的话也都有类似操作，实际上你部署在 GitHub Pages 上的文件就是这些，你只要把他们全部扔到 OSS 上，然后再在 OSS 给你的 bucket 设置一个自定义的域名（域名需要是你自己的，需要按照指引设置解析规则，域名需要经过备案），就可以了。

实际上我上一个博客就是这样部署的。全部文件都在 OSS 上，然后外面再加 CDN ， CDN 我们后面说，我们先说说 OSS 的问题。

如果你看看构建完成的 `public/` 目录，你会发现里面有很多文件，比如我的大概有 “94 个文件， 94 个文件夹” ，根据不同的静态网站生成器，和生成的 url 风格的不同，文件和文件夹数目可能相差很大，但是反正都很多就是了。然后你会发现 OSS 不能上传文件夹，你只能逐一新建文件夹，上传文件。图片还可以接受，一般文件夹不多，一个文件夹内可以批量上传，而且图片一般不需要改变。但是其他文件，每一次构建都需要更新一次，手动上传真的吃不消。

所以我的解决方案就是写了一个脚本，自动同步，他会检查本地目录和 OSS Bucket 上文件名是否有不同的，然后执行删除或者增加文件的操作，如果是同名文件，对它们哈希校验，看看文件是否有修改，有修改的就将本地的再上传一次，覆盖 OSS 上的。它可以自动将本地一个目录下的文件全部同步到 OSS 一个 Bucket 上，包括里面所有的嵌套文件夹。一次根据需要同步的文件数目不同大概需要十几秒到几分钟不等。这就是 OSS 比图床、网盘的优势了， OSS 有丰富的接口可以通过编程与之交互。这个脚本我就不细说了，感兴趣的可以看看这个项目 [keybrl/oss_sync](https://github.com/keybrl/oss_sync) 。我现在虽然不用这种 OSS 建站的方法了，但我其实仍然在使用这个脚本，我用它同步我 OSS 上的图片。

![upload_images](https://blog-assets-1253422097.file.myqcloud.com/images/2019-07-21-static-website-access-speed-optimization/upload_images.jpg)

我一般写博客就会打开这么一个终端，一边写一边上传需要的图片。

除了与 OSS 交互比较麻烦以外， OSS 还有一个问题。 OSS 没有目录的概念，它的存储模型其实就是一个对象名加一个对象（一堆二进制）。你说，哎，不是啊，这不是可以 “新建文件夹” 吗，这路径不是可以 `/images/hhh.png` 吗？嗯，这是可以的，但是实际上那张图片的名字就叫做 `/images/hhh.png` 他跟另一个叫做 `/images/hhh2.png` 的图片没有任何关系。那空文件夹是什么，空文件夹就是一个对象名是 `/images/` 的 0 大小的对象。这个东西仅仅是为了给你在图形界面好看而存在的。实际上没有 `/images/` 也可以有 `/images/hhh.png` ，因为图片并不是在 `/images/` 里面的，仅仅是名字有一样的前缀而已。

没有目录会有什么问题呢？你抬头看看这个页面的地址 `https://blog.keybrl.com/professional-2019-07-21-static-website-access-speed-optimization/` 你会发现这个 url 看起来是一个目录，但是目录并不能访问，目录并不会写那么多字，你看到的页面是一个文件。那访问目录为什么会得到文件呢，因为我在我的静态文件服务器里配置了 `autoindex on` ，所以当你访问 `/hhh/` 它会给你返回 `/hhh/index.html` ，你也可以在地址栏上试试，后面多加一个 `index.html` 看看，你会得到同一个页面。这么做有什么特殊好处呢，其实也没有什么好处，主要就是 url 比较好看。但是这种 url 在访问 OSS 上会有问题，因为 OSS 没有目录的概念，它自然也不会去找你要的目录里面的 `index.html` ，它只会给你返回 404 。

这个问题是可以解决的，如果你用 MkDocs 你可以选择一篇叫做 `hhh` 的文章是被生成 `hhh/index.html` 还是 `hhh.html` ，并且处理好网站中所有的 url 是形如 `hhh/` 还是 `hhh.html` 。后者这种风格的 url 是可以用于访问 OSS 的。

但是 Hexo 和 Jekyll 虽然都是表面上中立地支持生成这两种风格的页面和 url ，但是实际上他们都偏向目录风格。 Jekyll 太久没用我就不说了， Hexo ，虽然你可以生成文件风格的 url 和页面，但是某些默认页面，比如所有分类页面的 url 是 `/categories/` ，我也不知道是因为我使用的主题问题还是 Hexo 的问题，反正这个东西我没法调整。所以如果一个静态网站生成器不能生成完全文件风格 url 的网站，那么这个网站就不能正常部署在 OSS 上。

所以我们来看第二个方法，就是自建静态文件服务器，使用 Nginx ，非常简单，具体可以看看这篇文章 [Nginx常用配置套路](/professional-2018-07-02-nginx-configure/) 。本站就是这样部署的，你会发现即使是部署本站的云服务器只有 1Mbps 的带宽，本站访问速度依然很不错，因为一个 HTML 只有几十 kB 嘛，其他大文件都在 OSS 上，这时服务器响应速度和建立连接的速度比实际传输速度更重要。但是频繁地将构建好的网站上传到自己服务器上很麻烦，我是怎么做的呢？

Hexo 已经有了一些自动部署的方法，比如使用 `hexo deploy -g` 他会自动将构建好的网站按照设置的部署方法部署好。比如我设置了 git 方法的部署，设置好仓库地址和分支名（比如说 `deploy` ），那么 Hexo 会自动帮我将构建好的网站 push 到我仓库的 `deploy` 分支。然后我在服务器 pull 一下这个分支就可以了，这个方法很多人都会，也是常见的往云服务器部署服务的方法。但是，既然就是一个简单的 pull ，我为什么不能让他自己 pull 呢？

GitHub 有一个东西叫做 Webhook ，设置好一个 Webhook 之后， GitHub 会在仓库发生某个事件时，比如一个 push 事件，将事件相关信息 POST 到某个设定的 url 。我在我服务器写一个 Django 应用，接收这个 Webhook 的 POST 请求，校验身份、检查是否是关于我博客仓库的 `deploy` 分支的 push 事件，如果是就执行一段 bash 脚本，自动 pull 一下分支。这样我虽然没有部署在 GitHub Pages 上，但是我还是跟原来一样，只需要 `hexo deploy -g` 就可以完成博客更新部署。

这些其实都是自动化部署的技巧了，跟优化访问没什么关系。

我上面说， “两种思路” ，其实我只讲了第一种思路的两个方法，第一种思路就是加快源站的访问速度。第二种思路就是用 CDN 。

## CDN

CDN 翻译就是 “内容分发网络” ，所以就是什么呢。 CDN 的基本工作原理，如果要说明白，似乎有点复杂，我就简单介绍一下。

比如说你的网站部署在 GitHub Pages 上，我们不知道它具体是为什么那么慢，我们就假设它是单一服务器，而且这个服务器带宽很低，只有 100kbps ，那访问的人多了就完全应付不过来了嘛，毕竟一个页面如果 10MB ，单是传输时间都至少需要 819 秒，还没计算往返时延、丢包、建立连接的时间，人多的话就更困难了。

这里面访问慢主要就几个问题，带宽不够、往返时延长、访问的人多时服务端维护大量连接负载重。那解决方法是什么呢，就是分布式嘛。如果每个城市都有一个服务器（就是 CDN 服务器），每个服务器都有一个这个网站的副本，每个服务器都有很高的带宽和性能。当你请求获取一个页面时，你没有直接向源站（就是 GitHub Pages 的小水管服务器）发送请求，而是向你附近的那些很强的服务器发送请求，他们可以快速响应你，并且用他的大带宽向你发送文件，这样你访问的速度就会很快了嘛。但是这里还是有问题，比如说，如果要更新网站，那些 CDN 服务器怎么知道要更新呢？还有就是，客户端怎么通过同一个域名，能够访问到不同的CDN服务器呢（要就近访问嘛）？

第一个问题我简单介绍一下吧，第二个问题了解 Web 的同学都知道，不了解的也没有必要知道，也跟这篇文章的内容没什么关系，我就不说了。 CDN 服务器相当于是有一份网站的缓存，他是要符合缓存策略的。缓存策略可以设置，比如说设定缓存超时时间。如果设置了 24 小时的缓存超时时间，那么同步了一次之后，一整天，都不需要向源站请求更新。如果一个请求到来，CDN 服务器发现所请求的资源没有缓存在服务器上（比如说是新发布的文章），或者缓存超时了，那么 CDN 服务器会请求源站获取最新的资源，这个叫回源，其他情况下 CDN 服务器都可以直接将缓存的资源直接发送给客户端。除了超时时间，缓存策略还有不同的，我就不详细说了。那么一般来说，除了需要回源的情况比直接访问源站慢，其他情况下，通过 CDN 都应该比直接访问源站快，特别是源站比较弱的情况下。

所以通过 CDN 可以很容易提升静态网站访问速度，无论是对于 OSS 还是 GitHub Pages ， CDN 都很有效。

我突然觉得我这篇文章写了很长了，如果还是像 OSS 那样一步步写的话好像有点无聊。那我就不细说了，大家自己可以去看看，按照指引添加 CDN ，然后按照指引设置 CNAME 解析，设置回源策略，设置证书什么的，。主要要注意的是，用于配置 CDN 的域名要先备案。

---

上面提到的东西，全套搞下来，网站访问速度就会快很多。而且上面提到的很多产品也不贵。

OSS 的话建议买一个 9元一年的存储包（应该是有 40GB 的存储空间），然后平时就只需要给流量费，我没有仔细阅读流量计费规则，反正，充个 10 块钱余额，一个月也就几分到几毛钱的支出（相信我，你博客没什么人看的）

CDN的话我也没有买流量包，就让它按量扣费，一个月也就几毛钱。

云服务器是最贵的了，但是如果你平时有用来做别的事情，顺便搭一个静态文件服务器也不麻烦。
