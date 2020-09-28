# meta 标签

### 开始我们先说说 meta 标签的作用：

meta 标签是 HTML 标记 head 区的一个辅助标签，它位于 HTML 文档的`<head>`和`<title>`之间（有些也不是在`<head>`和`<title>`之间）。**它提供的信息虽然对用户不可见，但是于浏览器可见**。可以说 meta 标签就是写给浏览器看的

HTML`<meta>`除了提供文档字符集、使用语言、作者等基本信息外，还可以定义页面编码语言、**搜索引擎优化**、**自动刷新并指向新的页面**、**控制页面缓冲**、
**响应式视窗** 等！

```
在查阅 w3school 中，第一句话中的 “元数据” 就让我开始了 Google 之旅。然后很顺利的在英
文版的 w3school 找到了想要的结果。（中文 w3school 说的是元信息，Google 和百度都没有相关的词条。但元数据在 Google 就有详细解释。所以这儿采用英文版 W3school 的解释。）

不难看出，其中的关键是 metadata，中文名叫元数据，是用于描述数据的数据。它不会显示在页面上，但是机器却可以识别。这么一来 meta 标签的作用方式就很好理解了。
```

meta 常用于定义页面的说明，关键字，最后修改日期（缓存），和其它的元数据。这些元数据将服务于浏览器（如何布局或重载页面），搜索引擎和其它网络服务。

我们先来瞅瞅 meta 的几个属性标签：

<div align="center" style="margin: 15px">

|       属性       |                                             值                                             |                    描述                    |
| :--------------: | :----------------------------------------------------------------------------------------: | :----------------------------------------: |
| charset( H5 New) |                                       character_set                                        |             定义文档的字符编码             |
|     content      |                                            text                                            | 定义与 http-equiv 或 name 属性相关的元信息 |
|    http-equiv    |                         content-type<br> default-style<br> refresh                         |      把 content 属性关联到 HTTP 头部       |
|       name       | application-name<br>author<br>description<br>generator<br>keywords<br>copyright<br> robots |       把 content 属性关联到一个名称        |
| scheme(H5 删除)  |                                  format/URI HTML5 不支持                                   |     定义用于翻译 content 属性值的格式      |

<p>meta标签属性attributes</p>
</div>

## HTML 中 meta 标签的使用方法介绍:

## 1. charset

meta 最常见的无疑是定义 charset Unicode 字符集啦

```html
<!-- 默认UTF-8编码 -->
<meta charset="UTF-8" />
<!--如果 HTML5 网页使用不同于 UTF-8 的字符，则需要在 <meta /> 标签中指定，如下：-->
<meta charset="ISO-8859-1" />
```

为什么是 UTF-8，不能是 UTF-16 吗?

<div align="center">

| 字符集 |                                                                           描述                                                                            |
| :----: | :-------------------------------------------------------------------------------------------------------------------------------------------------------: |
| UTF-8  |           UTF8 中的字符可以是 1 到 4 字节长。UTF-8 可以代表 Unicode 标准中的任何字符。UTF-8 向后兼容 ASCII。UTF-8 是电子邮件和网页的首选编码。            |
| UTF-16 | 16 位 Unicode 转换格式是一种可变长度的 Unicode 字符编码，能够编码整个 Unicode 指令表。UTF-16 主要用于操作系统和环境，如 Microsoft Windows、Java 和 .NET。 |

<p>unicode常见字符集</p>
</div>
提示：Unicode 的前 128 个字符（与 ASCII 一一对应）使用一个与 ASCII二进制值相同的八位组进行编码，使有效的 ASCII 文本在进行 UTF-8 编码时也是有效的。

提示：所有的 HTML 4 处理器支持 UTF-8，所有的 HTML 5 和 XML 处理器支持 UTF-8 和 UTF-16！

## 2. name

name 属性主要用于描述网页，对应属性是 content ，以便于搜索引擎机器人查找、分类（目前几乎所有的搜索引擎都使用网上机器人自动查找 meta 值来给网页分类）。

```html
<!--页面关键词--->
<meta
  name="keywords"
  content="个人活动发布，会办app，活动管理，会议管理，社群管理"
/>

<!--页面描述-->
<meta
  name="description"
  content="发布个人会议，发布公司会议，我们都可以帮你找到合适的会议地点和参会观众"
/>

<!--网页作者-->
<meta name="author" content="nzreal" />

<!--搜索引擎抓取 robots， 是一组使用逗号（，）分割的值，通常有如下几种取值：none，noindex，nofollow，all，index和 follow。默认值是all-->
<meta name="robots" content="all" />

<!--禁止 Chrome 浏览器中自动提示翻译-->
<meta name="google" value="notranslate" />

<!--自定义标签：app 版本号说明-->
<meta name="app-version" content="1.13.3" />

<!--版权-->
<meta name="copyright" content="本网站版权归XX所有" />

<!--一般由编译器自动生成-->
<meta name="generator" content="你所用的编辑器" />

<!--网站重访 让搜索引擎每隔7天访问一次我们的网页-->
<meta name="revisit-after" content="7 days" />
<!--一般来说，网站并不需要使用revisit-after来限制搜索引擎的访问频率。
而只有当网站数据量非常大，被搜索引擎频繁的抓取，会占用过多的服务器资源，这会影响网站的访问反应速度。在这种情况下，我们并不希望搜索引擎过于频繁抓取网页，一般设置成每隔5—7天来访问抓取一次网页即可
同时限制抓取频率，但不能通过设置它来提高抓取频率
-->
```

## 3. http-equiv

```html
<!--页面重定向和刷新-->
<!--数字代表几秒后进行跳转-->
<meta http-equiv="refresh" content="5; url=https://github.com/nzreal" />

<!--指定网页在缓存中的过期时间，一旦网页过期，必须到服务器上重新传输。-->
<meta http-equiv="expires" content="Wed, 26 Feb 1997 08:21:57 GMT" />
<!--注意：必须使用GMT的时间格式，或者直接设为0（数字表示多久后过期）-->

<!--禁止浏览器从本地计算机的缓存中访问页面内容。-->
<meta http-equiv="Pragma" content="no-cache" />
<!--注意：网页不保存在缓存中，每次访问都刷新页面。这样设定，访问者将无法脱机浏览。-->

<!-- Set-Cookie（cookie设定）浏览器访问某个页面时会将它存在缓存中，下次再次访问时就可从缓存中读取，以提高速度。当你希望访问者每次都刷新你广告的图标，或每次都刷新你的计数器，就要禁用缓存了。
如果网页过期，那么存盘的cookie将被删除。-->
<meta
  http-equiv="Set-Cookie"
  content="cookievalue=xxx; expires=Wednesday, 21-Oct-98 16:14:21 GMT; path=/"
/>
<!--注意：必须使用GMT的时间格式-->

<!---禁止百度转码-->
<meta http-equiv="Cache-Control" content="no-siteapp" />
```

以上都是关于 HTML meta 标签的作用和使用方法，还有很多属性，不过那些对现在的我们来说学这些还有点早。

再来分享几个著名网站的首页的 meta 文件：

### 京东首页的 meta 设置：

```html
<meta charset="gbk" />

<meta
  name="description"
  content="京东 JD.COM-专业的综合网上购物商城,销售家电、数码通讯、电脑、家居百货、服装服饰、母婴、图书、食品等数万个品牌优质商品.便捷、诚信的服务，为您提供愉悦的网上购物体验!"
/>

<meta
  name="Keywords"
  content="网上购物,网上商城,手机,笔记本,电脑,MP3,CD,VCD,DV,相机,数码,配件,手表,存储卡,京东"
/>
```

### 淘宝首页的 meta 设置：

```html
<meta charset="utf-8" />

<meta http-equiv="X-UA-Compatible" content="IE=edge,chrome=1" />

<meta name="renderer" content="webkit" />

<meta name="spm-id" content="a21bo" />

<meta
  name="description"
  content="淘宝网-亚洲最大、最安全的网上交易平台，提供各类服饰、美容、家居、数码、话费/点卡充值…8 亿优质特价商品，同时提供担保交易(先收货后付款)、先行赔付、假一赔三、七天无理由退换货、数码免费维修等安全交易保障服务，让你全面安心享受网上购物乐趣！"
/>

<meta name="keyword" content="" />
```

### 优酷首页的 meta 设置：

```html
<meta charset="utf-8" />

<meta http-equiv="X-UA-Compatible" content="IE=Edge" />

<meta
  name="title"
  content="优酷-中国领先视频网站,提供视频播放,视频发布,视频搜索-优酷视频"
/>

<meta name="keywords" content="视频,视频分享,视频搜索,视频播放,优酷视频" />

<meta
  name="description"
  content="视频服务平台,提供视频播放,视频发布,视频搜索,视频分享"
/>
```

除了上面的解释的那些，这在补一个也是常用的 meta 标签里的属性

X-UA-CompatibleIE8 的专用标记，用来指定 IE8 浏览器去模拟某个特定版本的 IE 浏览器的渲染方式，以此来解决部分兼容问题。

```html
<meta http-equiv="X-UA-Compatible" content="IE=7" />
```

以上代码告诉 IE 浏览器，无论是否用 DTD 声明文档标准，IE8/9 都会以 IE7 引擎来渲染页面。

通过看到大网站对于 meta 的设置，我们可以看到常用的有：X-UA-Compatible、keywords、description 这三种，我们要好好学会用。

### 参考修改整理自：https://blog.csdn.net/zhangank/article/details/94014629 及 http://www.divcss5.com/html5/h54802.shtml

### 这两篇文章写得较全，故转载补充以 mark 来学习，大家可以去看看原文
