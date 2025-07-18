---
title: "你真的需要一个 CDN 吗"
date: 2019-07-28T10:00:00+08:00
toc: true
draft: true
tags:
  - 技术
  - 建站
---

在上一篇文章（ [静态网站访问速度优化技巧](static-website-access-speed-optimization.md) ）中，我提到了一种叫做 CDN 的技术。它通过缓存，和使用更接近边缘的服务器进行内容分发而达到加速静态网站访问速率的作用。

一直以来，我都是就简单地直接去使用它，仅仅凭感觉和理论去衡量它的效果，从来没有尝试细致一点地去量化它的效果，更没有尝试去针对性地优化配置。

所以这篇文章，我来尝试先量化比较在不同场景下有 CDN 和没有 CDN 的情况，能有怎样的性能表现。然后再尝试给出在某些场景下使用 CDN 的建议。特别的，这篇文章讨论的场景全部都是基于你正在浏览的这个博客的部署情况的，如果你的需求和这样的场景不同，那么你可以参考去测试你所在场景的情况，然后得到适合你的配置方案。

同时，这篇文章不会详细介绍 CDN 的工作方式，也不会介绍相关的部署、配置过程，关于这些问题的讨论，我或许会另外写一篇文章，反正不会是今天。

---

## 测试场景和环境

这篇文章所讨论的场景，全部都是基于你正在浏览的这个博客的部署情况的。所以我先介绍一下这个博客的部署情况。

这个博客所包含的静态资源基本上来自于两个源。

首先是每个页面的 HTML 文件是部署在 **腾讯云 广东广州 的云服务器，1 核、 2G 内存、 1Mbps 固定带宽 使用 Nginx 部署** ，这样的一个文件的大小大概 50KB ，每个页面只有一个这样的文件。

然后是页面上所有图片、 CSS 、 JS 、字体文件都存储在 **阿里云 浙江杭州 的 OSS 服务器上** ，每个文件大小不一，但是我只选取一些 1440 * 1920 的 JPEG 格式的图片进行测试，它们的大小大概 1MB ，每个页面上这种文件的数目不等，一般十几个。

由于这是两个不同的源，不方便配置在同一个 CDN 上，所以我会分别讨论在上面配置 CDN 的效果。

同时，还有一种情况，虽然我现在没有使用这种方式部署我的博客，但是我过去使用过，而且很多同学现在仍然使用这种方式部署自己的静态网站，那就是 GitHub Pages 。 GitHub Pages 本身不慢，但是在国内访问很慢，我们可以把它当作是一个很慢的源站，看看 CDN 能不能帮助我们改善访问 GitHub Pages 上站点的体验。

这样我们相当于按照源站的类型不同分成了三种场景

|     | 往返时延高        | 往返时延低 |
|-----|--------------|-------|
| 带宽低 | GitHub Pages | 云服务器  |
| 带宽高 |              | OSS   |

对于 CDN 的配置方式，所有测试都使用阿里云的 CDN 服务，加速区域选择“全球加速”。对我的云服务器和 GitHub Pages 服务加速时，都使用源站域名来标识源站。对 OSS 加速时，由于所用 OSS 和 CDN 都是阿里云的，它们内部有标识方式，具体如何回源的我就不得而知了。全部测试中访问所有资源都使用 HTTPS ，回源也使用 HTTPS 。

测试方法主要就是用脚本连续加载一定数量的文件，计算总用时。在加载文件时，可以指定一定数量的 url ，指定加载次数（比如每个文件加载10次），指定加载时使用的线程数。每个线程会被均匀分配一定数量的请求任务，它们会分别使用一个 session 完成给定数量的请求，也就是说每个线程在请求多个资源时会复用 TLS 连接。多个线程在这里可以模拟多个用户或者一个浏览器的多个线程。

全部测试在广东深圳进行，测试期间所使用 CDN 给我分配的边缘节点基本都在 **广东云浮** 。

测试脚本参考下面这段代码（实际用于测试的脚本会根据测试项目不同做少量调整）

```python
"""
该脚本用于进行 单站点 连续 多文件 加载测试
以加载总时长作为量度
"""

import requests
import json
import threading
import time


def files_loading(thread_id, target_urls, result):
    s = requests.session()
    for url in target_urls:
        try:
            response = s.get(url)
        except Exception as err:
            print(f'{thread_id}: {url}, loading fail: {err}')
            continue
        size = len(response.content) // 1024
        print(f'{thread_id}: {url}, {size}KB, {response.status_code}')
    s.close()


def files_loading_controller(target_urls, times=1, threads_num=0):
    threads_num = len(target_urls) if threads_num == 0 else threads_num
    target_urls = target_urls * times
    threads = []
    results = []
    for i in range(threads_num):
        results.append(dict())
        threads.append(threading.Thread(target=files_loading, args=(i + 1, target_urls[i::threads_num], results[i])))

    start_time = time.time()
    for thread in threads:
        thread.start()

    for thread in threads:
        thread.join()
    total_time = time.time() - start_time

    return {
        'total_time': total_time
    }


def main():
    # 加载配置文件
    with open('./targets2.json', 'rt', encoding='utf-8') as fp:
        targets_config = json.load(fp)

    host = targets_config.get('host')
    targets_url = [
        f'{host}{url}'
        for url in targets_config.get('urls')
    ]
    times = targets_config.get('times', 1)
    threads_num = targets_config.get('threads_num', 0)
    result = files_loading_controller(target_urls, times, threads_num)
    print(result)


if __name__ == '__main__':
    main()

```

## 对带宽较低的云服务上部署的静态资源使用 CDN

腾讯云的学生主机，虽然带宽很低，性能很差，但是单纯做个静态文件服务器还是很可以的，单个用户访问体验是不差的，人多的时候有点撑不住。如果对它加 CDN ，会有什么效果呢。

### 基准测试

首先看不加 CDN 的情况。我选取了我博客总大小 850KB 的 16 个页面。由于一个页面只有一个文件，一个用户一次不可能同时加载两个文件，所以这里测试使用 n 个线程加载 n 个文件的方式，比如说 32 线程，就要加载这 16 个文件两遍，每个线程分一个文件。我从 16 开始，每次增加 16 的倍数个线程和文件，最高到 256 个线程和文件。考察加载总时长（表示加载时间最长的线程所用时间）和加载成功率这两个指标。测试重复 8 遍。

不过以上只是我的设计，实际测试时我加到 96 个线程加载时间就接近 2 分钟了，数量级都加了两个，再增加也没有多少参考价值了，就没有继续增加线程数。至于成功率， 48 次测试中 47 次都是 100% 成功，剩下那次可能是网络波动 96.88% ，基本都是 100% ，就不进行比较了，只作为参考。

测试结果如下

| Threads | 1st   | 2nd   | 3rd    | 4th   | 5th   | 6th    | 7th    | 8th    | avg   |
| :------ | ----: | ----: | -----: | ----: | ----: | -----: | -----: | -----: | ----: |
| 16t     | 3.46  | 3.62  | 3.51   | 3.65  | 3.56  | 2.05   | 3.56   | 3.52   | 3.37  |
| 32t     | 7.14  | 7.07  | 7.18   | 7.31  | 6.89  | 7.44   | 7.71   | 7.72   | 7.31  |
| 48t     | 13.90 | 14.00 | 14.89  | 14.38 | 14.59 | 14.72  | 16.03  | 9.35   | 13.98 |
| 64t     | 27.81 | 15.67 | 56.34  | 28.95 | 27.85 | 16.73  | 27.76  | 27.76  | 28.61 |
| 80t     | 28.42 | 56.73 | 28.41  | 30.97 | 56.10 | 59.42  | 55.52  | 28.41  | 43.00 |
| 96t     | 56.59 | 56.51 | 110.85 | 31.02 | 57.31 | 110.59 | 110.62 | 111.65 | 80.64 |

![CVM N 线程加载 N 文件](https://blog-assets-1253422097.file.myqcloud.com/images/2019-07-28-do-you-need-a-cdn/cvm-N线程加载N文件.png)

其中，每一次测试结果一个点绘制在图中，中间折线表示 8 次重复测试的平均值。

看起来，随着用户增加，最长响应时间差不多是呈指数函数增长。用户稍微多一些，加载一个页面就需要等待几十秒到几分钟。这个性能，像我的小破博客还勉强可以接受，一般我博客同时也不会超过一个人访问，除非我在朋友圈推文章链接。但是如果是对于稍微多人访问一些的网站，这个结果就不可以接受了。

看到这个结果其实我想起了 SS::STA 在 2017 年招新的情况，当时没有经验，就简单把迎新网站部署在一个便宜的云服务器上，然后主席在新生级会上刚宣讲完，迎新网站就炸了。临时给 Nginx 加了几个 worker 才勉强缓解，而且还是很炸。其实 CDN 也要不了几个钱，比升级服务器配置和网络带宽便宜多了。

### CDN 测试

下面来看看 CDN 做同样测试时的表现吧。由于 CDN 表现不俗，我把线程数最高加到 512 个，结果如下

| Threads | 1st   | 2nd   | 3rd  | 4th   | 5th   | 6th   | 7th  | 8th  | avg  |
| :------ | ----: | ----: | ---: | ----: | ----: | ----: | ---: | ---: | ---: |
| 16t     | 10.03 |  0.37 | 0.30 |  0.39 |  0.53 |  0.42 | 0.30 | 0.30 | 1.58 |
| 32t     |  0.64 |  0.43 | 0.41 |  0.39 |  0.60 |  0.52 | 0.32 | 0.35 | 0.46 |
| 48t     |  0.62 |  0.61 | 0.55 |  0.44 |  0.63 |  0.96 | 0.70 | 0.76 | 0.66 |
| 64t     |  0.81 |  0.71 | 0.98 |  0.85 |  0.74 |  1.02 | 0.67 | 0.78 | 0.82 |
| 80t     |  1.18 |  1.00 | 1.04 |  1.04 |  1.07 |  1.31 | 0.88 | 1.07 | 1.07 |
| 96t     |  1.33 |  1.42 | 1.26 |  1.15 |  1.58 |  1.28 | 1.09 | 1.41 | 1.31 |
| 112t    |  1.31 |  1.56 | 1.74 |  1.55 |  1.06 |  1.91 | 1.32 | 1.28 | 1.47 |
| 128t    |  1.63 |  1.60 | 1.50 |  1.42 |  1.49 |  1.68 | 1.43 | 1.77 | 1.56 |
| 144t    |  1.61 |  1.59 | 1.76 |  2.21 |  1.50 |  2.13 | 1.55 | 1.97 | 1.79 |
| 160t    |  1.97 |  1.91 | 2.09 |  1.79 |  1.94 |  2.36 | 1.83 | 1.93 | 1.98 |
| 176t    |  1.80 |  1.98 | 2.10 |  2.19 | 16.93 |  2.05 | 2.06 | 2.24 | 3.92 |
| 192t    |  1.97 |  2.13 | 2.27 |  2.16 |  1.91 |  2.25 | 2.43 | 2.62 | 2.22 |
| 208t    |  2.56 |  2.70 | 2.37 |  2.51 |  2.09 |  2.42 | 2.59 | 2.50 | 2.47 |
| 224t    |  3.36 |  2.28 | 2.47 |  3.20 |  3.85 |  2.73 | 2.91 | 2.81 | 2.95 |
| 240t    |  2.30 |  2.72 | 2.65 |  3.88 |  2.20 |  2.87 | 3.46 | 2.60 | 2.84 |
| 256t    |  3.34 | 17.34 | 3.04 |  3.31 |  3.72 |  3.45 | 3.44 | 2.75 | 5.05 |
| 272t    |  3.43 |  2.33 | 3.07 |  3.38 |  3.48 |  3.77 | 3.34 | 3.28 | 3.26 |
| 288t    |  3.80 |  5.29 | 3.60 | 18.14 |  3.94 |  4.56 | 3.48 | 3.91 | 5.84 |
| 304t    |  3.83 |  2.88 | 3.66 |  2.69 |  3.98 |  3.75 | 3.69 | 4.08 | 3.57 |
| 320t    |  4.41 |  4.47 | 4.00 |  6.08 |  4.25 | 18.38 | 3.38 | 4.89 | 6.23 |
| 336t    |  5.82 |  4.08 | 4.03 |  3.16 |  4.31 |  3.53 | 4.04 | 4.21 | 4.15 |
| 352t    | 11.57 |  4.78 | 4.08 |  9.65 |  4.42 |  5.08 | 4.50 | 4.25 | 6.04 |
| 368t    |  3.56 |  4.67 | 4.49 | 15.31 |  4.20 |  3.58 | 4.70 | 4.17 | 5.59 |
| 384t    |  6.38 |  5.00 | 4.79 |  4.31 |  4.33 |  4.90 | 4.99 | 4.67 | 4.92 |
| 400t    |  3.81 |  5.07 | 4.48 |  6.00 |  4.40 |  5.19 | 4.56 | 5.07 | 4.82 |
| 416t    |  5.49 |  4.38 | 5.48 |  4.40 |  5.65 |  5.34 | 4.62 | 5.33 | 5.09 |
| 432t    |  5.27 |  5.65 | 5.26 |  5.32 |  5.82 |  6.95 | 4.83 | 5.58 | 5.58 |
| 448t    |  5.68 |  5.11 | 5.20 |  5.71 | 18.15 |  5.72 | 6.74 | 5.23 | 7.19 |
| 464t    |  5.39 |  6.08 | 5.37 |  5.79 |  6.02 |  5.56 | 6.01 | 5.23 | 5.68 |
| 480t    |  5.44 |  6.11 | 6.00 |  5.37 |  5.32 |  5.43 | 6.08 | 6.15 | 5.74 |
| 496t    |  5.67 |  6.30 | 5.72 | 16.84 |  5.71 |  6.14 | 5.70 | 6.98 | 7.38 |
| 512t    |  6.58 |  5.81 | 6.29 |  6.68 |  6.83 |  6.36 | 5.91 | 6.53 | 6.37 |

![CDN N 线程加载 N 文件](https://blog-assets-1253422097.file.myqcloud.com/images/2019-07-28-do-you-need-a-cdn/cdn-N线程加载N文件.png)

将两组测试平均值画在同一张图中

![N 线程加载 N 文件](https://blog-assets-1253422097.file.myqcloud.com/images/2019-07-28-do-you-need-a-cdn/N线程加载N文件.png)

会发现加了 CDN 之后性能有明显的提高。而且我猜测，这个最长响应时间与线程数之所以是一次函数，是因为测试所在网络带宽限制。测试过程中，基本上都是跑满 50Mbps 的带宽。所以其实用多线程模拟多用户还是有问题的，但是也没有办法了。

这里面的第一个数据， 10.03 秒，之所以这么慢是因为相关资源没有缓存在 CDN 边缘服务器上，所请求每个资源还需要回源，所以比较慢。

### 结论

首先，在这个情况下，加 CDN 肯定比不加 CDN 性能要好很多。而且云服务器带宽很贵，加 CDN 比升级服务器带宽要划算很多。

但是不是我就应该无脑加 CDN 呢？其实我是很纠结的。除了上面的测试，我还做了一个规模小得多的测试，如果只加载 1 个 52KB 的文件，重复 64 次。使用 CDN 时平均 273.85ms ，不使用 CDN 时平均 171.51ms ，明显不使用 CDN 更快。但是当把线程数和文件数加到 4 ，使用 CDN 是 384.70ms ，不使用 CDN 是 741.01ms ，就 CDN 更快了。

什么意思呢。就是说，如果像我博客平常时候，只偶尔有一个用户来看看，那么对这个用户来说，加了 CDN 比不加 CDN 体验更差。但如果是我发个朋友圈说我发了一篇新文章，然后可能就会出现好几个用户同时访问，这时如果没有 CDN ，那么加载时间随用户数量是成指数函数增长的。然后就很容易给人一种，我的博客，平时还好，人稍微多一点点就不行了的感觉。

如果仅仅考虑这个情况，似乎用 CDN 更稳健，最差时差 100ms 并不会被人感知，发挥作用时能好几十上百倍就很有用。但是 CDN 还有一个问题就是缓存，上面加载单个文件差 100ms 的结果是在 CDN 已经缓存了资源的情况下，如果缓存失效，这个时间是 1s 左右，就差了非常多。缓解缓存失效问题就只能增加缓存超时时间，如果按照我博客的情况，一天只有一两个访问的话，缓存超时时间可能要设置 1 周或者 1 个月才比较有效。但是设置特别长的缓存超时时间就会给我博客更新带来麻烦，我将需要在更新文章后手动刷新缓存。

或者我可能可以在服务器配置一些规则，访问多时重定向到 CDN ，这样或许比较合适。这个问题我可能还需要好好研究一下。

还有一个问题可能会干扰我的判断，那就是我在广东深圳进行测试，云服务器在广东广州，CDN 给我分配的边缘服务器在广东云浮，明显源站离我更近。但如果我在西安（我的很多同学就在西安）或者北京（我认识的不少强者在北京工作），源站对比 CDN 还会不会有那么强的竞争力呢？这也是一个有待确认的问题。

## 对 OSS 使用 CDN

OSS 其实已经很快了，为什么还要加 CDN 呢。一个是为了省钱， CDN 的流量费比 OSS 要便宜，第二个是 CDN 边缘服务器更靠近用户，数量更多，可能会更快或者在有大量访问的情况下体验更好。

### 基准测试

首先看一下基准测试，也就是不加 CDN 的情况，我选取了总大小 9.09MB 的 16 张图片，使用两种方案测试。

一个是文件总数 16 不变，不断增加线程数目（从 1 到 16 ），模拟一个用户访问一个页面，页面上有 16 张图片，浏览器使用多线程（或多进程）加载资源的情况。看看增加线程能不能明显提高加载速率。

第二个方案是，文件还是这 16 个，不断增加线程数，同时文件倍数和线程数同步增加，比如 4 个线程，就要加载 16 * 4 个文件。模拟的是多个用户同时访问同一个页面，各自都需要加载一份这 16 张图片。看看用户数目增加时 ta 们的体验有多大幅度的下降。

每项测试重复 8 次

其中第一项，多个线程加载同一组资源的结果如下

![OSS 多线程加载同一组资源](https://blog-assets-1253422097.file.myqcloud.com/images/2019-07-28-do-you-need-a-cdn/OSS-多线程加载同一组资源.png)

纵轴是加载的总时长，单位是秒，横轴是线程数目。每个结果一个点绘制在图上，中间折线是 8 次测试结果的平均值。

可以看到，在 1 到 4 个线程时，增加线程有比较明显的速率提升，继续增加线程，效果基本保持不变，结果上下波动。看平均值折线可能不明显，可以看看散点的分布。

第二项，多个线程各自加载一组资源的结果如下

![OSS 多线程各自加载一组资源](https://blog-assets-1253422097.file.myqcloud.com/images/2019-07-28-do-you-need-a-cdn/OSS-多线程各自加载一组资源.png)

同样，纵轴是加载的总时长，单位是秒，横轴是线程数目。每个结果一个点绘制在图上，中间折线是 8 次测试结果的平均值。

可以看到，这基本上就是一个一次函数，每个用户的等待时间与同时访问的用户数量成正比。而且注意到，如果有 16 个用户同时加载同一个有 16 张图片的页面时，在这个模拟的情况中，每个用户需要等待超过半分钟才能完成加载，这个时间几乎不能接受。不过幸好，我的博客同一时间不会有 16 个用户访问，超过 1 个都已经很夸张了。

### CDN 测试

使用 CDN 时，我会进行跟上面完全一样的测试，但是，在开始正式测试之前，我会先跑一遍单线程加载16个图片的测试。一方面是为了让 CDN 准备好缓存，另一方面是比较一下没有缓存时从 CDN 加载比直接从 OSS 加载慢多少。 CDN 我设置了 30 分钟的缓存时间，足够覆盖后面所有测试过程。

在单线程加载 16 张图片的测试中，第一次跑了 6.23 秒，但是第二次和第三次都是 2 秒左右，说明 CDN 已经缓存了这些资源，效果是比较明显的。但是即使是缓存后，好像也没有比基准测试中第一项第一组的平均 2.32 秒快很多。

两次测试的结果如下

![CDN 多线程加载同一组资源](https://blog-assets-1253422097.file.myqcloud.com/images/2019-07-28-do-you-need-a-cdn/CDN-多线程加载同一组资源.png)

![CDN 多线程各自加载一组资源](https://blog-assets-1253422097.file.myqcloud.com/images/2019-07-28-do-you-need-a-cdn/CDN-多线程各自加载一组资源.png)

看起来好像和基准测试结果差不多。

将两组图表中的平均值放在一张图中比较一下

![多线程加载同一组资源 比较](https://blog-assets-1253422097.file.myqcloud.com/images/2019-07-28-do-you-need-a-cdn/both-多线程加载同一组资源.png)

![多线程各自加载一组资源 比较](https://blog-assets-1253422097.file.myqcloud.com/images/2019-07-28-do-you-need-a-cdn/both-多线程各自加载一组资源.png)

实际上就是差不多。而且我尝试把线程加到 256 个，效果还是差不多。或许在特别大量用户访问时， CDN 优势会更明显，但是我的博客肯定不可能会出现超过我测试限度的负载。在这种情况下，除了省钱，好像 CDN 没有任何优势，加载未缓存的资源明显变慢，加载缓存的资源速度没有显著加快。

### 关于省钱

关于省钱，我今天也稍微计算一下，毕竟图片还是很耗流量的，我为了进行这个测试也花费了几十GB的流量。

在我查询时，阿里云对于 OSS 产品的定价明细如下

![oss定价](https://blog-assets-1253422097.file.myqcloud.com/images/2019-07-28-do-you-need-a-cdn/oss-price.png)

存储和上传的流量是固定要给的了，我们就比较直接从 OSS 下载和通过 CDN 下载的流量费和请求费用相差多少。如果按照一张图片 512KB 计算， 1 GB 的图片有 2048 张。也就是要消耗外网流出流量 1GB 外加 2048 次请求，也就是 0.5 * 1 + 0.01 * 0.2048 = 0.52048 元。

CDN 的费用，包含基础服务和增值服务。基础服务可以按两种方式计费 “按峰值带宽计费” 和 “按流量计费” 两种，为了方便比较，我们就按照默认的计费方式 “按流量计费” 来说明。按流量计费的定价如下

![cdn基础服务定价](https://blog-assets-1253422097.file.myqcloud.com/images/2019-07-28-do-you-need-a-cdn/cdn-base-price.png)

虽然我开通了全球加速，但是肯定没有什么外国人看我的中文博客，所以我们全部按照中国大陆的流量来算。我从来没有试过一个月超过 10TB 流量，所以就按最低的 0GB - 10TB 档来计算。 1GB 流量费是 0.24 元。

由于我还开启了 HTTPS ，这作为增值服务需要另外计费

![cdn增值服务定价](https://blog-assets-1253422097.file.myqcloud.com/images/2019-07-28-do-you-need-a-cdn/cdn-extra-price.png)

全部都是静态 HTTPS 请求， 0.05 * 0.2048 = 0.01024 ，总共 0.25024 元。回源的费用我就不计算了，忽略不计。也就是说，使用 CDN 比只使用 OSS 要节省差不多一半的流量费，虽然只有几毛钱，但是看起来也足够诱人。调大缓存时间可以减少缓存失效的情况，同时，如果 CDN 和 OSS 都是阿里云的，那么可以在 OSS 中开启 “CDN 缓存自动刷新” 功能来避免缓存不一致的问题。这样的话， CDN 就还是一个可以考虑的选择。

### 结论

上面的测试展示了 CDN 对比 OSS 没有太大的性能优势，但是便宜一半，这已经值得使用 CDN 了。

这个测试还有一个很严重的问题局限了它的参考价值。那就是测试环境的带宽不高，测试环境带宽大概是 50Mbps ，实在不能模拟多个用户，即使是 3G/4G 用户也最多相当于两三个。这与上一节云服务器与CDN的比较不一样，我的云服务器只有 1Mbps 的带宽，很容易触到天花板，而且单个文件只有 50KB 左右，同时加载多个文件也不容易触到我 50Mbps 的上限。但是在这个 OSS 与 CDN 的比较中， OSS 和 CDN 带宽都非常高，肯定高于我自己的 50Mbps ，而且单个文件 500KB 到 1MB ，同时加载几十个文件就很容易达到我的带宽上限。所以这也是为什么上面的结果中加载时间随文件数目增加成一次函数增加。

但是这个测试起码说明，用户数量不多的时候，直连 OSS 和使用 CDN 是差不多的。

## 对 GitHub Pages 使用 CDN

我已经很少用 GitHub Pages 建站了，用也是用在一些小项目展示或者临时应急上，肯定也不会费工夫去优化访问速度。但是为什么我还是写了这一节呢？这要从我写这个系列的文章的动机说起，如果你看了上一篇文章 [静态网站访问速度优化技巧](static-website-access-speed-optimization.md) ，你会发现我拿了我友链里面一个人的博客作为优化的反面例子，他就是将所有文件扔到 GitHub Pages 上的部署方式，在那篇文章中你也能看到这导致整个网站加载速度非常非常过分地缓慢。要说他博客文章质量也还可以，但是这个加载速度非常劝退，如果网不好，加载五分钟也未必能加载好一页，导致我并不经常去看。

他肯定也意识到这个问题了。只是一直没有去试图改善，或者被这里面的一点点小门槛挡住了。很多人开始想要搭建自己的博客，记录自己的学习、生活时，都会被这些小问题劝退。然后表现就是嫌麻烦，懒得管，然后我就很不理解，这有什么麻烦的嘛，不是很简单的操作嘛。所以我就觉得有必要写一些建站相关的文章，起码能够让不了解这些知识的人能够有一些直观的认识，能够解决一些会遇到的问题。而且这些人有不少都依赖 GitHub Pages 等静态网站托管服务，所以有必要了解一下加 CDN 这种简单的方法能否改善。

好，那这一节我们就简单介绍一下。

GitHub Pages 虽然看起来很慢，但其实它也是有 CDN 的，只不过服务器基本在境外，而且我们网络普遍不是特别好，造成了 GitHub Pages 特别菜的感觉。如果有正常的网络，加载速度其实和你正在浏览的这个博客速度差不多。

因为 GitHub Pages 也有 CDN ，所以在面对大量访问时，表现跟我现在要测试的 CDN 是差不多的，只是平均比较慢而已。为了节省时间，我们就进行加载单个文件的测试，为了比较有代表性，我选取了一个 52KB 的 HTML 文件和一个 1386KB 的图片分别测试，测试重复 64 次。结果如下

![加载52KB的HTML文件](https://blog-assets-1253422097.file.myqcloud.com/images/2019-07-28-do-you-need-a-cdn/加载52KB的HTML文件.png)

![加载1386KB的图片文件](https://blog-assets-1253422097.file.myqcloud.com/images/2019-07-28-do-you-need-a-cdn/加载1386KB的图片文件.png)

因为前面两节测试所在网络环境都非常不错，访问 GitHub Pages 也非常快，不符合中国大陆一般的情况，所以这一部分使用的是我手机热点，测试了一下，带宽大概是 60Mbps 。

两张图片中深蓝色的都是直接访问部署在 GitHub Pages 上的源站，橙色的都是访问阿里云的 CDN 。线条是所谓 “ 2 周期移动平均” ，实际上就是将相邻两次测试值的平均值连成一条折线，大致能反映波动情况。

可以看到，加载图片时， GitHub Pages 最高可以飙到令人发指的 102 秒，这还只是单张图片。即使是小文件，加载时间也是使用 CDN 的 2 到 30 倍，而且非常不稳定。但是使用 CDN 之后，访问速度明显改善，而且稳定得多，只要 CDN 缓存了相关资源。

这 4 组测试的平均值如下

|              | 52KB HTML | 1386KB PNG |
| :----------- | --------: | ---------: |
| GitHub Pages |    1.1549 |    29.0130 |
| CDN          |    0.2416 |     0.6080 |

---

好了，就研究到这里吧。结论嘛，没有，方法论嘛，也没有，就是讲一下我自己在考虑是否加 CDN 时的测试方法和考虑的问题，如果觉得跟你的情况比较接近，或许可以参考一下。而我自己的情况嘛，我原本以为经过测试我会很明朗，但是似乎更纠结了，我应该还需要更仔细考虑一下。有空或许会再写一篇关于 CDN 配置的文章（等我去再仔细了解一下再说）
