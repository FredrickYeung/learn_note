# Web前端优化

<!--- [Cache-Control](https://varvy.com/pagespeed/cache-control.html)

- [progressive web app - HTTP 缓存](https://developers.google.com/web/fundamentals/performance/optimizing-content-efficiency/http-caching)

- [How caching and CDNs work](https://docs.fastly.com/guides/basic-concepts/how-caching-and-cdns-work)
-->
[Best Practices for Speeding Up Your Web Site](https://developer.yahoo.com/performance/rules.html)

## **Content**

### **1. 尽量减少HTTP请求(Make Fewer HTTP Requests)**

主要有以下方法：
- 合并文件
- CSS Sprites 雪碧图
- 内联图像。使用data:URL scheme的方法把图像数据加载到页面中，例如base64

### **2. 减少DNS查找次数(Reduce DNS Lookups)**

DNS提供了域名和IP的对应关系。缓存DNS查找可以改变性能。这种缓存需要一个特定的缓存服务器，这种服务器一般属于用户的 ISP 提供商或者本地局域网控制，但是它同样会在用户使用的计算机上产生缓存。浏览器也会进行DNS缓存。

### **3. 避免重定向(Avoid Redirects)**

URL末尾斜杠（/）被忽略时会返回301状态码的跳转，在Apache服务器中使用Alias或者mod_rewrite或者the DirectorySlash来避免。

### **4. 可缓存的Ajax(Make Ajax Cacheable)**

Ajax响应通过Expire或者Cache-Control来实现缓存。其他规则也适用于Ajax：Gzip压缩、减少DNS查找次数、精简JavaScript、避免重定向、配置ETags

### **5. 延迟加载组件(Post-load Components)**

如果有用于实现拖放和动画的JavaScript，可以等待稍后加载。YUI Image Loader可以延迟加载图片。

### **6. 预加载组件(Preload Components)**

预加载是在浏览器空闲时请求将来可能会用到的页面内容（如图像、样式表和脚本）。

下面提供了几种预加载的方法：

- **无条件加载**：触发onload事件时，直接加载额外的页面内容。以[Google.com](www.google.com)为例，你可以看一下它的spirit image图像是怎样在onload中加载的。这个spirit image图像在google.com主页中是不需要的，但是却可以在搜索结果页面中用到它。
- **有条件加载**：根据用户的操作来有根据地判断用户下面可能去往的页面并相应的预加载
页面内容。
- **有预期加载**：载入重新设计过的页面时使用预加载。这种情况经常出现在页面经过重新设计后用户抱怨“新的页面看起来很酷，但是却比以前慢”。问题可能出在用户对于你的旧站建立了完整的缓存，而对于新站点却没有任何缓存内容。因此你可以在访问新站之前就加载一部内容来避免这种结果的出现。在你的旧站中利用浏览器的空余时间加载新站中用到的图像的和脚本来提高访问速度。

### **7. 减少DOM元素数量(Reduce the Number of DOM Elements)**

YUI CSS utilities可以给你的布局带来巨大帮助：grids.css可以帮你实现整体布局，font.css和reset.css可以帮助你移除浏览器默认格式。它提供了一个重新审视你页面中标签的机会，比如只有在语意上有意义时才使用`<div>`，而不是因为它具有换行效果才使用它。

通过以下代码计算DOM的数量：

```JavaScript
document.getElementsByTagName('*').length
```
### **8. 根据域名划分组件(Split Components Across Domains)**

把页面内容划分成若干部分可以使你最大限度地实现平行下载。由于DNS查找带来的影响你首先要确保你使用的域名数量在2个到4个之间（TODO:为什么）。例如，你可以把用到的HTML内容和动态内容放在www.example.org上，而把页面各种组件（图片、脚本、CSS)分别存放在statics1.example.org和statics.example.org上。

### **9. 最小化iframe数量(Minimize the Number of iframes)**

`<iframe>`的优点：
- 解决加载缓慢的第三方内容如图标和广告等的加载问题
- Security sandbox
- 并行加载脚本（TODO：跨域？）

`<iframe>`的缺点：
- 即时内容为空，加载也需要时间
- 会阻止页面加载
- 没有语意，影响SEO

### **10. 不要出现404错误(No 404s)**

对页面链接的充分测试加上对 Web 服务器 error 日志的不断跟踪能有效减少 404 错误，亦能提升用户体验。HTTP 请求时间消耗是很大的， 因此使用 HTTP 请求来获得一个没有用处的响应（例如404没有找到页面）是完全没有必要的，它只会降低用户体验而不会有一点好处。

## **Server**

### **1. 使用CDN(Use a Content Delivery Network)**

内容分发网络（Content Delivery Network，CDN）是由一系列分散到各个不同地理位置上的Web服务器组成的，它提高了网站内容的传输速度。用于向用户传输内容的服务器主要是根据和用户在网络上的靠近程度来指定的。例如，拥有最少网络跳数（network hops）和响应速度最快的服务器会被选定。

### **2. 添加 Expires 或 Cache-Control 信息头(Add an Expires or a Cache-Control Header)**

这条守则包括两方面的内容：
- 对于静态内容：设置文件头过期时间Expires的值为“Never expire”（永不过期）
- 对于动态内容：使用恰当的Cache-Control文件头来帮助浏览器进行有条件的请求

Apache

```
ExpiresActive On
ExpiresByType image/gif "modification plus 1 weeks"
```

Lighttpd 启用 mod_expire 模块 后：

```
$HTTP["url"] =~ "\.(jpg|gif|png)$" {
    expire.url = ( "" => "access 1 years" )
}
``` 

Nginx 例子参考:

```
location ~* \.(jpg|gif|png)$ {
    if (-f $request_filename) {
        expires max;
        break;
    }
}
```

### **3. Gzip压缩文件内容(Gzip Components)**

从HTTP/1.1 开始，web客户端都默认支持HTTP请求中有Accept-Encoding文件头的压缩格
式：

```
Accept-Encoding: gzip, deflate
```

如果web服务器在请求的文件头中检测到上面的代码，就会以客户端列出的方式压缩响
应内容。Web服务器把压缩方式通过响应文件头中的Content-Encoding来返回给浏览器。

```
Content-Encoding: gzip
```

[Nginx/IIS 配置Gzip压缩](http://dbanotes.net/web/best_practices_for_speeding_up_your_web_site_server.html)

### **4. 设置ETags (Configure ETags)**

Entity tags（ETags）是web服务器和浏览器用于判断浏览器缓存中的内容和服务器中的原始内容是否匹配的一种机制（“实体”就是所说的“内容”，包括图片、脚本、样式表等）。Etag是一个识别内容版本号的唯一字符串。唯一的格式限制就是它必须包含在双引号内。

原始服务器通过含有ETag文件头的响应指定页面内容的ETag。

```
HTTP/1.1 200 OKLast-Modified: Tue, 12 Dec 2006 03:03:59 GMT
ETag: “10c24bc-4ab-457e1c1f”
Content-Length: 12195
```

稍后，如果浏览器要验证一个文件，它会使用If-None-Match文件头来把ETag传回给原始服务器。在这个例子中，如果ETag匹配，就会返回一个304状态码，这就节省了12195字节的响应。

```
GET /i/yahoo.gif HTTP/1.1
Host: us.yimg.com
If-Modified-Since: Tue, 12 Dec 2006 03:03:59 GMT
If-None-Match: “10c24bc-4ab-457e1c1f”
HTTP/1.1 304 Not Modified
```
[TODO: 未完]

### **5. 尽早刷新Buffer (Flush the Buffer Early)**

[TODO: 不太明白]

### **6. 使用GET来完成AJAX请求(Use GET for AJAX Requests)**

当使用XMLHttpRequest时，浏览器中的POST方法是一个“两步走"的过程：首先发送文件头，然后才发送数据。因此使用GET最为恰当，因为它只需发送一个TCP包（除非你有很多cookie）。IE中URL的最大长度为 2K，因此如果你要发送一个超过2K的数据时就不能使用GET了。

[TODO:理解不够彻底]

## **Cookie**

### **1. 减少Cookie体积(Reduce Cookie Size)**

[TODO: 有意思]

### **2. 对于Web组件使用无coockie域名(Use Cookie-free Domains for Components)**

对于静态内容的请求采用无Cookie的请求，可通过创建子域名来存放静态内容。

## **CSS/JavaScript**

### **1. 把样式表置于顶部(Put Stylesheets at the Top)**

按照HTML规范在文档`<head />`内加载样式表。

### **2. 避免使用CSS表达式(Avoid CSS Expressions)**

一个减少CSS表达式计算次数的方法就是使用一次性的表达式，它在第一次运行时将结果赋给指定的样式属性，并用这个属性来代替CSS表达式。如果样式属性必须在页面周期内动态地改变，使用事件句柄来代替CSS表达式是一个可行办法。如果必须使用CSS表达式，一定要记住它们要计算成千上万次并且可能会对你页面的性能产生影响。

### **3. 使用外部JavaScript和CSS(Make JavaScript and CSS External)**

剥离后，能够有针对性的对其进行单独的处理策略，比如压缩或者缓存策略。

### **4. 精简JavaScript与CSS(Minify JavaScript and CSS)**

### **5. 用`<link>`代替@import**

在 IE 中 @import 指令等同于把 link 标记写在 HTML 的底部。而这与第一条相违背。

### **6. 避免使用Filter(Avoid Filters)**

IE独有属性AlphaImageLoader用于修正7.0以下版本中显示PNG图片的半透明效果。这个滤镜的问题在于浏览器加载图片时它会终止内容的呈现并且冻结浏览器。在每一个元素（不仅仅是图片）它都会运算一次，增加了内存开支，因此它的问题是多方面的。完全避免使用 AlphaImageLoader的最好方法就是使用PNG8格式来代替，这种格式能在IE中很好地工作。如果你确实需要使用AlphaImageLoader，请使用下划线_filter又使之对IE7以上版本的用户无效。

[TODO:不懂]

### **7. 脚本放到HTML代码页底部(Put Scripts at the Bottom)**

### **8. 移除重复脚本(Remove Duplicate Scripts)**

### **9. 减少DOM访问**

有以下建议：

- 缓存已经访问过的元素(Cache references to accessed elements)
- “离线”更新节点, 再将它们添加到树中(Update nodes “offline” and then add them to the tree)
- 避免使用 JavaScript 输出页面布局(Avoid fixing layout with JavaScript)

## **开发智能事件处理程序(Develop Smart Event Handlers)**

如果你在一个 div 中有 10 个按钮，你只需要在 div 上附加一次事件句柄就可以了，而不用去为每一个按钮增加一个句柄。

关于 Ajax 优化指导原则，可以参见[提高 Ajax 应用程序性能，避开 Web 服务漏洞](https://www.ibm.com/developerworks/cn/web/wa-aj-speed.html)


### **1. 优化图像(Optimize Images)**

你可以检查一下你的GIF图片中图像颜色的数量是否和调色板规格一致。 使用imagemagick中下面的命令行很容易检查：

```
identify -verbose image.gif
```

如果你发现图片中只用到了 4 种颜色，而在调色板的中显示的 256 色的颜色槽，那么这张图片就还有压缩的空间。

尝试把 GIF 格式转换成 PNG 格式，看看是否节省空间。下面这条简单的命令可以安全地把 GIF 格式转换为 PNG 格式：

```
convert image.gif image.png
```

在所有的PNG图片上运行pngcrush（或者其它PNG优化工具）。例如：

- [pngcrush](http://pmt.sourceforge.net/pngcrush/)
- [pngrewrite](http://www.pobox.com/~jason1/pngrewrite/)
- [OptiPNG](http://www.cs.toronto.edu/~cosmin/pngtech/optipng/)
- [PNGOut](http://advsys.net/ken/utils.htm)


```
pngcrush image.png -rem alla -reduce -brute result.png
```

在所有的 JPEG 图片上运行 [jpegtran]((http://jpegclub.org/))。 这个工具可以对图片中的出现的锯齿等做无损操作，同时它还可以用于优化和清除图片中的注释以及其它无用信息（如EXIF 信息）：

```
jpegtran -copy none -optimize -perfect src.jpg dest.jpg
```

### **2. 优化 CSS Sprites (Optimize CSS Sprites)**

- 在 Spirite 中水平排列你的图片，垂直排列会稍稍增加文件大小；
- Spirite 中把颜色较近的组合在一起可以降低颜色数， 理想状况是低于 256 色以便适用 PNG8 格式；
- 便于移动，不要在 Spirite 的图像中间留有较大空隙。这虽然不大会增加文件大小但对于用户代理来说它需要更少的内存来把图片解压为像素地图。100×100 的图片为 1 万像素，而 1000×1000 就是 100 万像素。

### **3. 不要在HTML中缩放图像 (Don’t Scale Images in HTML)**

不要为了在 HTML 中设置长宽而使用比实际需要大的图片。如果你需要：

```html
<img width=”100″ height=”100″ src=”mycat.jpg” alt=”My Cat” />
```

那么你的图片（mycat.jpg）就应该是 100×100 像素而不是把一个 500×500 像素的图片缩小使用

### **4. favicon.ico 要小而且可缓存 (Make favicon.ico Small and Cacheable)**

favicon.ico 是位于服务器根目录下的一个图片文件。它是必定存在的，因为即使你不关心它是否有用，浏览器也会对它发出请求，因此最好不要返回一个 404 Not Found 的响应。由于是在同一台服务器上，它每被请求一次 coockie 就会被发送一次。这个图片文件还会影响下载顺序，例如在 IE 中当你在 onload 中请求额外的文件时，favicon 会在这些额外内容被加载前下载。

因此，为了减少 favicon.ico 带来的弊端，要做到：

- 文件尽量地小，最好小于 1K
- 在适当的时候（也就是你不要打算再换 favicon.ico 的时候，因为更换新文件时不能对它进行重命名）为它设置 Expires 文件头。你可以很安全地把 Expires 文件头设置为未来的几个月。你可以通过核对当前 favicon.ico 的上次编辑时间来作出判断

## **Mobile**

### **1. 保持单个组件小于25k (Keep Components under 25K)**

主要是因为iPhone不能缓存大于 25K的文件。

### **2. 打包组件成复合文本(Pack Components into a Multipart Document)**

把页面内容打包成复合文本就如同带有多附件的 Email，它能够使你在一个 HTTP 请求中取得多个组件（切记：HTTP 请求是很奢侈的）。当你使用这条规则时，首先要确定用户代理是否支持（iPhone 就不支持）。

[TODO: 不懂]






