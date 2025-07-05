---
title: "键盘客制化 | Monon R2 装配"
date: 2020-06-23T10:00:00+08:00
toc: true
tags:
  - DIY
  - 键盘
images:
  - https://blog-assets-1253422097.file.myqcloud.com/images/2020-06-23-monon-r2/final-1.jpg
---

19 年 10 月份的时候我写过一篇关于我第一把客制化键盘的装配的文章 [客制化键盘入坑笔记](my-first-custom-keyboard.md) 。 8 个月后的今天，我又组了第二把客制化键盘，不过今天这把不是我的，是我打算送给一个朋友的。

这把键盘前后准备了几个月，前几天才弄完。用这把键盘写了篇文章记录一下。

---

## 外壳、定位板

这把键盘最早确定的部件就是外壳了。 6 月初的时候在 [zFrontier 装备前线](https://www.zfrontier.com/) 看到一个不错的 60% 套件的发车贴 [Monon r2盲团已先行发车！](https://www.zfrontier.com/app/flow/18296) 。刚好还是我之前看到的一个视频里面我挺喜欢的套件（ Monon SE）的作者出的，感觉他们家 60% 套件设计得都挺合我审美。但是我看到这个帖子时这个套件已经开团很久了，我看到的是更新说快要发货了。不过加群之后发现还有现货，于是赶上了晚班车。

我选的这个是 PC 材质的，包含外壳、铝定位板和铝铭牌，我上车的价格是 699 。这个套件还有各种颜色的阳极氧化和烤漆的铝材质外壳可以选择，铝壳好像还便宜点。

![monon-r2](https://blog-assets-1253422097.file.myqcloud.com/images/2020-06-23-monon-r2/monon-r2.jpg)

因为我订这个外壳时还没有确定用什么颜色的键帽，所以定位板和铭牌都选择了烤漆白色，比较好搭配。

### 铭牌

铭牌纯白的不太好看。群里一起团购的小伙伴一般都是用丙烯颜料或者马克笔涂一下中间的字。但是我觉得千篇一律没意思，所以试着用水转印上色。

刚开始用的是田宫的绿色和白色的珐琅漆，有一种陶瓷的光泽。但是珐琅漆没法在水里扩散成薄薄一层，因为太厚了，漆干得很慢，干的过程中还在流动，所以弄的效果很差。下面是一些翻车案例：

![fail-printing](https://blog-assets-1253422097.file.myqcloud.com/images/2020-06-23-monon-r2/fail-printing.jpg)

幸好珐琅漆可以用稀释液擦掉，可以试多次。不过试了五六次就放弃了。

后来尝试只用绿色珐琅漆涂一下字：

![nameplate-1](https://blog-assets-1253422097.file.myqcloud.com/images/2020-06-23-monon-r2/nameplate-1.jpg)

效果还可以，但是感觉没有什么特色。于是又用田宫的绿色和白色亮光漆（喷漆）试了一下水转印。

![nameplate-2](https://blog-assets-1253422097.file.myqcloud.com/images/2020-06-23-monon-r2/nameplate-2.jpg)

其实效果也比较差强人意。不过算了，就这样吧。中间的字用珐琅漆又涂了一遍，然后整个喷了一遍透明的半消光漆作为保护。

### 定位板

虽然买外壳送了一块定位板，但是仔细观察后发现送的定位板不支持阶梯大小写、 ISO 回车和分裂退格。

![layout](https://blog-assets-1253422097.file.myqcloud.com/images/2020-06-23-monon-r2/layout.jpg)

而我刚好想弄成阶梯大小写和ISO回车的。

所以换了一块：

![plate](https://blog-assets-1253422097.file.myqcloud.com/images/2020-06-23-monon-r2/plate.jpg)

看起来好像和送的形状一样，但是回车、退格、大小写位置的孔位和间隔有一点点差别，这使它支持除了左移 64 以外的几乎所有配列。

## PCB

PCB 的选择有点头疼，我本来想要找一块热插拔 + 蓝牙/有线双模 + RGB + 多种配列的。但是这里本来就有很多矛盾，热插拔的一般都不会多配列，因为热插拔轴座有固定位置， RGB 同理。蓝牙和 RGB 一般不会在一起，因为 RGB 耗电，无线的话续航不行。满足最多条件的应该是 GK61S ，也就是我组的第一个键盘用的， GK61S 只有多配列不满足。不选择 GK61S 的另一个原因是 GK61S 只能用定位板卫星轴或者平衡杆，不能用 PCB 卫星轴，而适用定位板卫星轴和平衡杆的定位板不好买。手感上应该是平衡杆 > PCB 卫星轴 > 定位板卫星轴，而适用平衡杆的定位板尤其难买，定位板卫星轴我感觉太差了点，所以我还是希望找一个支持 PCB 卫星轴的 PCB 。

还有一个比较奇怪的事情就是淘宝有卖热插拔轴座，但是没有卖没焊热插拔座的适用热插拔座的 PCB 。我找这样的板子是因为如果要多佩列，板子上应该不能有焊与键位相关的元件，因为多配列的板子有些键位离得非常近，只能二选一焊接，而不能同时焊上。

最后看来看去都没有满意的，干脆就选了了比较多人用的 KBDfans 的 DZ60 。 DZ60 主要特性如下：

- TypeC 有线单模
- 焊接（即非热插拔）
- 支持多配列
- 没有 RGB 轴灯
- 有 RGB 底灯
- 轴正装（下灯位）
- 有两脚轴灯脚位
- 适用 PCB 卫星轴
- 支持 QMK 固件，支持 VIA

![dz60](https://blog-assets-1253422097.file.myqcloud.com/images/2020-06-23-monon-r2/dz60.jpg)

## 轴体

因为这把键盘是送给朋友的，这位朋友用过的键盘里面比较喜欢 Cherry 青轴和 Cherry 茶轴这种带段落的轴，不太喜欢线性轴。但是刚好我比较喜欢线性轴，对段落轴和声音轴没什么了解。

所以顺便买了下面这块 72 轴试轴器，包括了 Cherry 、凯华和佳达隆的几乎所有轴。

![switches-tester](https://blog-assets-1253422097.file.myqcloud.com/images/2020-06-23-monon-r2/switches-tester.jpg)

试了一圈感觉凯华的 Violet 轴和 Royal 轴不错。它们都和 Cherry 的白轴（ Clear 轴）有点像，都是一个大段落。只不过白轴段落在中间， Violet 轴和 Royal 轴在刚开始，而且 Violet 轴和 Royal 轴都比 Cherry 白轴更轻。

因为 Violet 轴比较轻， Royal 轴比较重，所以用 Violet 轴作为字母、数字和符号键，用 Royal 轴作其它键。

![violet-and-royal](https://blog-assets-1253422097.file.myqcloud.com/images/2020-06-23-monon-r2/violet-and-royal.jpg)

它们都属于凯华的 Box 轴，比起 Cherry 等的 MX 轴轴心更稳，不容易晃，不容易有卡涩感，据说还有防水溅的好处。

因为整体做工不错，听不出轴心与轴盖的摩擦声和弹簧声，而且我拆开看了一下，轴体段落处是有润滑的，所以我就不一一拆轴润滑了。

## 键帽

键帽也是挑了很久。本来想搞一套 JTK 的 ABS 二色键帽试试，但是这玩意儿基本都没有现货，要买得等个几个月，而且颜色不能随便选，得看最近开什么。那些奇奇怪怪的牌子的一两百的键帽我又不想买。而且我想找一套 ABS 二色成型的原厂高度键帽，这个条件的键帽不太多。

最后选了菜菜（ EnjoyPBT ）的绿白键帽， ABS 二色，原厂高度。

![keycaps](https://blog-assets-1253422097.file.myqcloud.com/images/2020-06-23-monon-r2/keycaps.jpg)

字符、颜色和大键都还可以。之前听说菜菜的空格是弯的，但我这套问题不大，可能是因为这是 ABS 材质的缘故吧，变形的一般都是 PBT 材质的。

比起 JTK 也算不上便宜吧，但主要是有现货，可以直接买到。

## 组装

组装可就，把我坑惨了。

首先是我忘了在焊轴之前装好卫星轴，焊了一大堆轴才想起。

![assembling-1](https://blog-assets-1253422097.file.myqcloud.com/images/2020-06-23-monon-r2/assembling-1.jpg)

用的是 PCB 卫星轴，定位板会阻碍卫星轴的安装，所以应该先安装卫星轴再焊轴。

没啥办法，只能全部都拆了，一个下午没了。焊接很容易但是拆焊有点麻烦，按吸锡器的左手累到抽筋。

![assembling-2](https://blog-assets-1253422097.file.myqcloud.com/images/2020-06-23-monon-r2/assembling-2.jpg)

装好卫星轴之后重新把轴都焊好。

焊好之后测试了一下，发现不能用。但是在拆轴之前我还试过是可以的，仅仅拆装一下怎么就不行了呢。

因为这个 PCB 是支持刷 QMK 固件的，默认用的是 VIA 的固件， VIA 固件其实也是一种特殊的 QMK 固件，只是 VIA 可以用一个图形界面实时修改键位配置，不用像一般的 QMK 固件那样要重新编译、重新刷写固件。

我试着重刷了 VIA 固件，发现还是不行。然后试着刷默认布局的普通 QMK 固件，成功了，键盘可以使用，说明 PCB 没问题。但是刷 VIA 固件就不能用，而且刷固件要按 PCB 上的 Reset 键进入 DFU 模式，刷完一般都会自动退出 DFU 模式，而每次刷完 VIA 固件之后都还是会进入 DFU 模式。我开始以为是固件版本不对，因为是开源的，我试着在仓库找找历史版本刷刷看，还是一样。然后又刷了其它型号 PCB 的 VIA 固件，发现可以识别为键盘了， VIA 软件也能够识别，但是键盘不能用，这很正常，因为固件和 PCB 型号不匹配。

最后我发现刷入非 VIA 的 QMK 固件后键盘虽然能用，但是左 Ctrl 键是一直被按住的，因为在浏览器滚动滚轮会放大缩小而不是上下滚动。测了一下键位触发发现最左侧那一列键全部都是一直被按住的。这就说得通了，因为 VIA 固件设定了如果开机时按住 ESC 键就会进入 DFU 模式。

然后排查 PCB 上的焊接情况，发现 ESC 键的一个脚因为焊盘和 TypeC 插口外壳的一个固定脚很近，我给焊到一起了，结果这一整条线都是对地短路的。把焊点处理一下让它们分开就正常了。

然后装键帽时发现右下角的键焊错了位置，应该往左边一点点，因为 PCB 支持多配列，同一个键可以焊在特别临近的好几个地方，没看清楚就给弄错了。又调整了一下。

然后是装轴灯。我本来想给所有键装上灯，虽然键帽不透光，但是有灯的话以后想换透光键帽可以换。用的是白色的 234 方灯，比圆头的灯光线要更均匀、柔和一点。

![led](https://blog-assets-1253422097.file.myqcloud.com/images/2020-06-23-monon-r2/led.jpg)

正当我要装时发现 Violet 轴和 Royal 轴结构是不一样的

![violet-and-royal-compare](https://blog-assets-1253422097.file.myqcloud.com/images/2020-06-23-monon-r2/violet-and-royal-compare.jpg)

Violet 轴上有安装两脚灯珠的孔位，而 Royal 轴上没有，奇了怪了，不知道为啥这样设计。所以 Royal 轴的灯只能装在轴下面，但是轴已经焊好了，只能打开轴上盖或者把轴整个拆下来。因为有定位板阻挡，拆轴盖很难，所以基本只能把轴整个拆下来，要拆的话得拆 14 个轴。后来我考虑了一下，干脆就不不装 Royal 轴的灯了，只要字母、数字、符号有灯就好。但是大小写灯不能没有，所以还是要拆一个的。

![assembling-3](https://blog-assets-1253422097.file.myqcloud.com/images/2020-06-23-monon-r2/assembling-3.jpg)

装好大小写灯之后就开始装其它灯。因为焊接的时候 PCB 要反过来，插在上面的灯珠如果没有固定会掉下来，所以我一次焊一行，全部插上用胶带固定。

![assembling-4](https://blog-assets-1253422097.file.myqcloud.com/images/2020-06-23-monon-r2/assembling-4.jpg)

然后焊完一行之后有一种预感，装上键帽试试，发现果然灯珠会卡键帽。因为用的是原厂高度，本来就比 OEM 高度矮，然后二色成型键帽里面还有一层，整个键帽比较厚，所以卡到灯珠了。没办法，只能全拆了。就，灯不装了吧。

最后装进外壳，装上键帽。

![final-1](https://blog-assets-1253422097.file.myqcloud.com/images/2020-06-23-monon-r2/final-1.jpg)

跟我的掌托颜色还有点配。

PCB 底部有 16 个 RGB 灯珠，光线可以透过半透明的 PC 外壳透出来，起到一点氛围灯的作用。

![final-2](https://blog-assets-1253422097.file.myqcloud.com/images/2020-06-23-monon-r2/final-2.jpg)

最后听听它的打字声吧

**因为我操作失误，这个视频暂时丢失了，所以播放不了，等我有时间再重新弄一个吧**

<video id="video" controls="" preload="metadata" width="100%">
    <source src="https://video-1253422097.file.myqcloud.com/blog/2020-06-23-monon-r2/sound.mp4" type="video/mp4">
    <p>Your user agent does not support the HTML5 Video element.</p>
</video>
