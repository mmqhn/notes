---
typora-copy-images-to: img
---

通常网站受到的攻击方式有：sql注入（在服务端防范，目前都已经做得很好了）、XSS、CSRF、DDOS、HTTP挟持等


##一、XSS（跨站脚本攻击）
###1. XSS 简介

![XSS](O:\notes\img/QQ截图20180417123622.png)

**之所以会发生 XSS 攻击，就是由于用户的输入被当做代码来执行，导致出现意料之外的运行结果**。XSS攻击的本质就是由于浏览器执行攻击者插入的恶意脚本数据所致。

根据往网页里注入脚本的方式和攻击形式的不同，XSS攻击可分为：

* 反射型（非持久型）XSS
* 存储型（持久型）XSS
* DOM-Based型XSS

#### 1.1 反射型（非持久型）XSS

![反射型XSS](O:\notes\img/QQ截图20180417124753.png)

反射型XSS是指诱导 **用户点击一些带有恶意脚本参数的链接** 而引起的 **一次性** XSS攻击。

反射型XSS归根结底就是由于不可信的用户输入被服务器在没有安全防范处理，然后就直接使用到相应页面中，然后反射给用户而导致代码在浏览器执行的一种XSS漏洞。

事实上由于反射型XSS因为url特征导致很容易被防御。很多浏览器（如chrome）都内置了相应的XSS过滤器，来防范用户点击了反射型XSS的恶意链接。



####1.2 存储型（持久型）XSS

![存储型XSS](O:\notes\img/QQ截图20180417125000.png)

存储型XSS是指将恶意代码存储到漏洞服务器中，用户浏览相关页面发起攻击。

持久型 XSS 其实就是由于不可信的用户输入在没有任何验证的情况下保存在服务端的文件或者数据库中，并且取出不可信的数据时也没有做相关安全处理就返回响应，导致了存储的恶意脚本数据在浏览器中执行的一种 XSS 漏洞。



**反射型XSS  VS  存储型XSS：**

* 非持久化  /  持久化（存储在服务器）
* 需要用户点击链接  /  不需要交互也可以触发



#### 1.3 DOM Based型XSS

![DOM Based型XSS](O:\notes\img/QQ截图20180417125155.png)

DOM-Based 型 XSS 是由于客户端 JavaScript 脚本修改页面 DOM 结构时引起浏览器 DOM 解析所造成的一种漏洞攻击。因此如果页面 JavaScript 脚本不存在着漏洞的话，则不会发送 DOM-Based 型 XSS 攻击啦。

特点：

1. 基于浏览器DOM解析的攻击
2. 并不依赖服务端，其数据来源在客户端中
3. 需要诱导用户打开恶意链接才能发送XSS攻击
4. 常触发的场景：innerHTML、outerHTML、document.write



### 2. XSS payload

**能实现XSS攻击的恶意脚本** 称为XSS payload。

XSS payload可以做到：

* **窃取用户cookie**（通过`document.cookie`可获取。cookie是表示网站中用户登录态的重要信息，窃取了cookie便能模仿用户的登录态）
* **识别用户浏览器**（通过`navigator.userAgent`可获取。识别到浏览器的类型和版本就可以采取相对应的攻击方式）
* **伪造请求**（通过Ajax、脚本、<img>、.src、表单等发起GET/POST请求来发起XSS攻击）
* **XSS钓鱼**（XSS payload + 钓鱼网站。比方说在评论区通过XSS payload发表了指向钓鱼网站的恶意链接 `<a href='www.fish.com'>王者荣耀周年活动</a>`，用户点击后跳转到钓鱼网站，一旦进行了输入用户名密码的操作，你的用户名密码就被发送到黑客的服务端了。）



### 3. XSS防御

#### 3.1 给cookie设置httpOnly属性

给cookie设置了httpOnly，会阻止客户端脚本访问Cookie。只有当与服务端交互时，http请求头中才会带上该cookie的信息，从而减少 Cookie 被盗取的可能性。

![httpOnly](O:\notes\img/QQ截图20180417125325.png)

设置httpOnly进行的XSS防御成本极低，但也存在问题：

* 不仅恶意脚本无法读取该Cookie，**网站本身的页面也不能获取该cookie**（因此只会在部分重要的Cookie上设置httpOnly属性）
* **非根本性解决XSS**（并没有防御XSS，只是在发生了XSS后防止脚本去获取cookie信息）


####3.2 输入检查 

输入检查前端可绕过，需要服务端结合前端一起实现输入检查。

用户的输入有两种情况：

- 只允许用户提交纯文本，即把用户的输入当做普通的文本字符串。比如用户注册时填写姓名、地址等信息。

  对应的输入检查：**采用白名单形式，htmlEncode进行转义特殊字符**

- 允许用户提交一些自定义HTML代码（称之为富文本）。比如用户在论坛里发帖，帖子的内容有图片、链接、表格等（需要通过HTML代码来实现）。

  对应的输入检查：**用黑名单来过滤危险的标签和属性**





#####3.2.1 白名单

判断输入格式

```javascript
/**
 * 转义 HTML 特殊字符
 * @param {String} str
 */
function htmlEncode(str) {
  return String(str)
    .replace(/&/g, '&amp;')
    .replace(/"/g, '&quot;')
    .replace(/'/g, '&#39;')
    .replace(/</g, '&lt;')
    .replace(/>/g, '&gt;');
}
```



#####3.2.2 黑名单

- **过滤危险字符**

  过滤黑名单标签：去除`<script>`、`javascript`、`onclick`等的整个标签；

  过滤标签中的事件黑名单属性：去除`onerror`、`onclick`属性等

- **转义特殊字符**

  `<` ->`&lt;`、`>`->`&gt;`、`&`->`&amp;`、`"`->`&quot`、`'`->`&#39;`

```javascript
/**
 * 过滤函数
 * @param {String} str
 */
function filter(str) {
	// ig ： 不区分大小写，并全局搜索
    // \1 ： 表示引用第一次匹配到的分组
    // regTag 整个标签去除<script>、<style>、iframe
    //regAttr 去除onerror、onclick属性等
    var regTag = /<(script|style|iframe)[^<>]*?>.*?<\/\1>/ig;
    var regAttr = /(onerror|onclick)=([\"\']?)([^\"\'>]*?)\2/ig;
  return String(str)
    .replace(/<script>.*<\/script>/g,'')
    .replace(/onclick=".*"/g,'')
}
```



#### 3.3 输出检查

输出检查是XSS防御的最后一道防线，根据不同场景对数据进行处理。通常恶意脚本（XSS payload）会输出到：

- HTML标签中 - `<a href="#">$var</a>`

  用户输入将在HTML解析环境进行

  对特殊字符的处理规则：HtmlEncode()

  ```javascript
  //HtmlEncode()也就是输入检查的HtmlEncode()
  ```

  ​

- HTML属性中 - `<div name="$var"></div>`

  用户输入将在HTML解析环境进行

  对特殊字符的处理规则：HtmlEncode()

- script标签中 - `<script>var x = "$var";</script>`

  用户输入将在 JavaScript 解析环境进行

  对特殊字符的处理规则：JavaScriptEncode()

  ```javascript
  function JavaScriptEncode(str) {
      function encodeCharx(original) {
          var thecharchar = original.charAt(0);
          switch(thecharchar) {
              case '\n': return "\\n";break;
              case '\r':return "\\r";break;
              case '\'':return "\\'";break;
              case '\"':return "\\x22";break;
              case '\&':return "\\&"; break;
              case '\\':return "\\\\";break;
              case '\t':return "\\t";break;
              case '/':return "\\x2F";break;
              case '<':return "\\x3C";break;
              case '>':return "\\x3E";break;
              default:return thecharchar;
          }
      }
  }
  ```

- 事件属性中 - `<a href="" onclick="func($var)">test</a>`

  用户输入将在 JavaScript 解析环境进行

  处理方式：JavaScriptEncode()

- URL中 -  `<a href="www.test.com?test=$var">test</a>`

  用户输入将在 URL 解析环境进行

  处理方式：URLEncode()

  浏览器有现成的`encodeURI()`



### 4. Web相关编码和转义

提到 Web 前端安全，第一个想到的就是 XSS。
而提到 XSS，不得不说的就是编码了。事实上，编码涉及的知识十分的复杂，很难短时间梳理清楚和弄明白。在这里只要求大家对编码有一个概念就好，日后再进行深究。



本文从 XSS 中常用的编码讲起，分为

- **URL 编码**
- **HTML 编码**
- **JS 编码**



####4.1 URL编码

一般来说，URL只能使用英文字母（`a-zA-Z`）、数字（`0-9`）、`-_.~`4个特殊字符以及所有（`;,/?:@&=+$#`）保留字符。

意味着如果使用了一些其他文字和特殊字符，则需要通过编码的方式来进行表示，如：

```
// 使用了汉字
var url1 = 'http://www.帅.com';
```

另外我们知道在 URL 中传参是通过**键值对**形式进行的，格式上是以`？`、`&` 和 `=` 为特征标识进行解析，如果在键或者值的内容中包含一些特殊符号，就会造成解析错误，如下所示：

```
// 键为汉字
var url2 = 'http://www.a.com?我=1';
// 值的内容为特殊符号
var url3 = 'http://a.com?key=?&';
```

对于上面的情况如果我们要正常解析，则需要进行编码，需要用不会造成歧义的符号代替有歧义的符号。

我们可以通过使用系统原生实现的 API 来对字符进行 **URL 编码**如:

- [encodeURI()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/encodeURI)
- [encodeURIComponent()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/encodeURIComponent)
- [escape 逐渐废弃可以不理会](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/escape)



#####4.1.1  encodeURI

**encodeURI** 是用来编码 URI 的，最常见的就是编码一个 URL。**encodeURI** 会将需要编码的字符转换为 `UTF-8` 的格式。对于保留字符（`;,/?:@&=+$#`），以及非转义字符（字母数字以及 `-_.!~*'()`）不会进行转义。

例如之前 URL 中包含中文，我们可以使用 encodeURI：

```
encodeURI('http://www.帅.com'); // http://www.%E5%B8%85.com
encodeURI('http://www.a.com?我=1');// "http://www.a.com?%E6%88%91=1"
```

在这里，`%E5%B8%85` 就是 `帅` 的 URL 编码，`%E6%88%91` 即为 `我` 的 URL 编码。然后由于 **encodeURI** 不转义`&`、`?`和`=`。所以对于 URL 参数的值是无法转义的，如下面的例子：

```javascript
// 值的内容为特殊符号
encodeURI('http://a.com?key=?&'); // "http://a.com?key=?&"
```

此时我们就需要使用 **encodeURIComponent** 来解决。



#####4.1.2  encodeURIComponent

顾名思义，**encodeURIComponent** 是用来编码 URI 参数的。它会跳过非转义字符（字母数字以及`-_.!~*'()`）。但会转义 URL的 保留字符（`;,/?:@&=+$#`）。通常来说我们会 **encodeURI** 结合 **encodeURIComponent** 来使用，如下所示：

```javascript
// "http://a.com?a=%3F%26"
encodeURI('http://a.com') + '?a=' + encodeURIComponent('?&');  
```

其中 `%3F` 和 `%26` 分别为 `?` 和 `&` 的 URL 编码。需注意的是由于 **encodeURIComponent** 会编码所有的 URL 保留字，所以不适合编码 URL，例如：

```
// "http%3A%2F%2Fa.com%3Fkey%3D%3F%26"
encodeURIComponent('http://a.com?key=?&');
```

上面的 `http%3A%2F%2Fa.com%3Fkey%3D%3F%26` 在地址栏会被解析为一个普通的字符串而不是 URL。



#####4.1.3 URL解码

有了 URL 编码，相应的会有解码机制。比如上面对应的 3个 encode 的API对应的解码 API 如下：

- [decodeURI()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/decodeURI)
- [decodeURIComponent()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/decodeURIComponent)



####4.2 HTML编码

在 HTML 中，某些字符是预留的，比如不能使用小于号（`<`）和大于号（`>`），这是因为浏览器会误认为它们是标签。如果希望正确地显示预留字符，我们必须在 HTML 源代码中使用字符实体（character entities）。当然还另一个重要原因，有些字符在 ASCII 字符集中没有定义，因此需要使用字符实体来表示，比如中文。

HTML 编码分为：

- HTML 十六进制编码 `&#xH;`
- HTML 十进制编码 `&#D;`
- HTML 实体编码 `&lt;` 等

在 HTML 进制编码中其中的数字则是对应字符的 unicode 字符编码。
比如单引号的 unicode 字符编码是27，则单引号可以被编码为`&#x27;`



**HTML实体编码**

通常来说，在业务中我们用到更多的是 HTML 的实体编码。常用的 HTML 实体编码函数如下所示：

```
/**
 * 转义 HTML 特殊字符
 * @param {String} str
 */
function htmlEncode(str) {
  return String(str)
    .replace(/&/g, '&amp;')
    .replace(/"/g, '&quot;')
    .replace(/'/g, '&#39;')
    .replace(/</g, '&lt;')
    .replace(/>/g, '&gt;');
}
```

更完整的转义列表见[这里](http://w3school.com.cn/html/html_entities.asp)。



####4.3 Javascript转义

JavaScript 中有些字符有特殊用途，如果字符串中想使用这些字符原来的含义，需要使用反斜杠对这些特殊符号进行[转义](http://www.w3school.com.cn/js/js_special_characters.asp)。我们称之为 **Javascript编码**。一般有以下几类：

- 三个八进制数字，如果不够个数，前面补0，例如 “e” 编码为“\145”
- 两个十六进制数字，如果不够个数，前面补0，例如 “e” 编码为“\x65”
- 四个十六进制数字，如果不够个数，前面补0，例如 “e” 编码为“\u0065”
- 对于一些控制字符，使用特殊的C类型的转义风格（例如\n和\r）

如下面所示，双引号用于标注字符串，然而在字符串中带了双引号，就会发生歧义：

```
var str = "Hello"";
```

于是我们需要对字符串内的双引号进行转义，也就是加上反斜杠，告诉脚本引擎要区分对待：

```
var str = "Hello\"";
```



####4.4 更多阅读

对 Web 相关编码和转义十分感兴趣的同学可以阅读下面的文章和内容来加深理解。

- [前端的各种转义](https://github.com/FrankFang/githublog/blob/master/%E6%8A%80%E6%9C%AF/%E5%89%8D%E7%AB%AF%E7%9A%84%E5%90%84%E7%A7%8D%E8%BD%AC%E4%B9%89.md)
- [从零开始学web安全（3）](http://imweb.io/topic/57024e4606f2400432c1396d)





## 二、CSRF（跨站请求伪造）

### 1. CSRF简介

**CSRF：跨站请求伪造（ Cross Site Request Forgery ）**

![CSRF](O:\notes\img/QQ截图20180417130130.png)

相比正常请求，伪造的请求有三个区别：用户不知情；跨站请求；参数伪造。

### 2. CSRF防御

#### 2.1 token验证（常用）

Anti CSRF token验证是用来防御伪造请求的参数伪造特性。

![token防御](O:\notes\img/QQ截图20180417132133.png)

token防御需要前后台的协作。前端页面在提交的时候把cookie里的一个值使用复杂的算法生成一个token传送给后台。后台收到传送来的token后，拿同样的cookie的一个值用相同的算法生成一个值，两者进行比较，一致则执行请求。跨站请求伪造是读不了我们域的cookie的，所以攻击者无法伪造。



#### 2.2 referer验证

referer验证是用来防御伪造请求的跨站请求特性。

前端页面发送给后端的请求会挂一个referer属性，后台抓到这个请求头的时候会验证request.header.referer是不是本站点，正确则请求成功。事实上referer验证不常用，因为它不可靠，可能被篡改。



![referer验证](O:\notes\img/QQ截图20180417131930.png)



#### 2.3 验证码

验证码是用来防御伪造请求的用户不知情特性。但由于极影响用户体验，一般CSRF不会使用验证码来防御。验证码一般是用来防御DDOS的。



## 三、XSS与CSRF的结合

![XSS+CSRF](O:\notes\img/QQ截图20180417135312.png)

当一个站点同时存在XSS和CSRF漏洞，攻击者就不需要诱导用户访问钓鱼网站，而是直接在原站点注入一个XSS脚本来发起伪造请求。

这意味着CSRF防御的referer验证（因为发起请求的XSS脚本就在同一站点）和Anti CSRF token验证（XSS脚本有可能会直接获取或间接算出服务器返回给你的token，token验证也可能会失效）有可能都会失效。



### 1. XSS蠕虫（XSS+CSRF的极端情况）

![XSS蠕虫](O:\notes\img/QQ截图20180417140602.png)

攻击者在自己的社交平台主页发了一篇微博（内容包含XSS恶意脚本），当用户A、B、C...来访问这个页面，XSS恶意脚本开始起作用，它会调用CSRF漏洞来伪造请求，这个请求也就是自动发送微博（这篇微博复制的攻击者的微博，也包含XSS恶意脚本）。这些用户又各自有自己的社交网络，这个蠕虫会疯狂地复制自己传播开来。

XSS蠕虫本质上就是不断传播的XSS+CSRF攻击，它必须要有地方（用户交互页面）来承载XSS恶意脚本，传播范围非常广。



## 四、 DDOS（分布式拒绝服务）

DDOS：分布式拒绝服务（Distributed Denial Of Service）。核心是 **攻击者利用大量肉鸡对目标发起大量非正常的请求来耗尽服务器的资源，使得网站服务瘫痪导致无法正常运作。**

![DDOS](O:\notes\img/QQ截图20180417142352.png)

DDOS防范可以通过以下一些方式：

- **验证码**

  随着验证码方式的丰富化，导致黑客们没有一个比较有效的方式来获取和识别验证码。但由于验证码会影响到用户体验，因此验证码只有在比较重要的入口才会使用，如转账和登陆的场景。

- **限制请求频率（ratelimit）**

  通过一些标识（如IP与cookie）为每个客户端设置请求频率的限制。

  许多框架都会有相应地限制请求频率的工具包，如 `koa` 框架便可以使用下面这些：

  - [koa-limit](https://github.com/cnpm/koa-limit)
  - [koa-ratelimit](https://github.com/koajs/ratelimit)

- **扩容加带宽**

  增加机器增加服务带宽，只要超过了攻击流量便可以避免服务瘫痪。

  在**双十一活动**或者**12306 抢车票**等场景时，网站服务都会对自己的机器进行扩容和提高带宽。为了避免由于使用过多机器导致成本太高且浪费的情况。通常来说都是根据网站活动和请求情况来实施相应扩容操作。

- **其它方法**

  - 设置自己的业务为分布式服务，防止单点失效
  - 使用主流云系统和 CDN（云和 CDN 其自身有 DDOS 的防范作用）
  - 优化资源使用提高 web server 的负载能力



## 五、运营商HTTP劫持

![HTTP劫持](O:\notes\img/QQ截图20180417143810.png)



HTTP劫持就是**利用了HTTP传输明文（不加密）的特性，篡改了服务端发给用户的网页内容**的一种攻击方式。通常被窃听者（运营商）用来窥视或篡改（在网页中插入广告）。

为了解决 HTTP 协议的这一缺陷，需要使用另一种协议：HTTPS。



### 1. HTTPS（HTTP+SSL）

![SSL](O:\notes\img/QQ截图20180417144530.png)

为了数据传输的安全，HTTPS在HTTP的基础上加入了**SSL（安全套接层）**协议，SSL依靠证书来验证服务器的身份，并为浏览器和服务器之间的通信加密。

SSL标准化改名为**TSL（Transport Layer Security，传输层安全协议）**，SSL/TSL在传输层对网络连接进行加密。



SSL/TSL加密解密有两种方式：

- **对称加密**

  加密解密用的同一条秘钥

- **非对称加密**

  秘钥**成对出现**，通常其中一条不公开，叫**私钥**，另一条是公开的，称为**公钥**。用**其中一条加密**的内容只能用**另一条解开**。

  ​

  下面是HTTPS交换秘钥的过程（利用**非对称加密算法**在客户端与服务器之间协商得到**对称秘钥**的密钥协商过程）：

  ![HTTPS交换秘钥的过程](O:\notes\img/QQ截图20180417150004.png)

这种方式并不是绝对安全，因为服务端是直接返回公钥给客户端的。如果窃听者窃听到公钥，窃听者会假冒服务端与客户端协商秘钥，假冒客户端与服务端协商秘钥，从中窃听网页的内容。

![窃听者窃听到公钥](O:\notes\img/QQ截图20180417151912.png)

为了保证服务器公钥的合法性（服务器传送公钥不会被窃取），服务端不再直接返回公钥给客户端，而是使用**数字证书认证机构（CA）**。服务端返回一个证书（包含公钥）给客户端，客户端收到证书后会去询问CA这个证书是否合法。

![使用CA传输公钥](O:\notes\img/QQ截图20180417152310.png)



## 六、更多安全内容

- **点击劫持**

  点击劫持实际上就是在正常界面上覆盖一层透明的内容，引导用户在不知情的情况下进行了某些操作，这些操作可能是广告点击，或者是开启摄像头等操作，甚至可能会导致一些数据别窃取，也有可能结合 XSS 和 CSRF 进行更负责的攻击。

  [web安全之--点击劫持攻击与防御技术简介](http://www.jianshu.com/p/251704d8ff18)

- **服务端安全**

  除了前端注意安全，后端也需要对安全进行防范，常见比如防止 SQL 注入，登录会话管理等。



## 七、安全实战

- 使用工具库[js-xss](https://github.com/leizongmin/js-xss)设置合法的标签和属性

  ​

