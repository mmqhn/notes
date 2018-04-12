### 1、Cookie是什么



![在chrome控制台查看cookie](C:\Users\ZHENG.Zheng-PC\Desktop\QQ截图20180411131441.png)



>**MDN上对cookie的介绍**
>
>HTTP Cookie（也叫Web Cookie或浏览器Cookie）是服务器发送到用户浏览器并保存在本地的一小块数据，它会在浏览器下次向同一服务器再发起请求时被携带并发送到服务器上。通常，它用于告知服务端两个请求是否来自同一浏览器，如保持用户的登录状态。Cookie使基于[无状态](https://developer.mozilla.org/en-US/docs/Web/HTTP/Overview#HTTP_is_stateless_but_not_sessionless)的HTTP协议记录稳定的状态信息成为了可能。
>Cookie主要用于以下三个方面：
>
>- 会话状态管理（如用户登录状态、购物车、游戏分数或其它需要记录的信息）
>- 个性化设置（如用户自定义设置、主题等）
>- 浏览器行为跟踪（如跟踪分析用户行为等）



>cookie 虽然可以储存本地信息，但是不适合储存大量的信息，因为 cookie 还有个特点是每次请求都会带上这个域相关的所有 cookie，就是说，cookie 会影响浏览器请求的大小，因此我们要尽量保持 cookie 体积小，还有尽量用于保存跟用户相关的信息。同时，浏览器本身对 cookie 总大小也有限制，目前是不允许超过 4k，超过的会丢失。
>
>由此而知，cookie 适用于数据量比较小的，跟用户信息相关的储存，而如果要储存相对量比较大的数据，而且这些这些数据不需要随着请求发到服务器的话，则可以使用 [`localStorage`](https://developer.mozilla.org/zh-CN/docs/Web/API/Window/localStorage)



根据过期时间可分为两种cookie：

- **session cookie**（临时cookie）
  关闭浏览器就会删除。控制台查看cookie时Expires的值表示为Session
- **permanent cookie**（持久cookie）
  到了过期时间才会删除。

### 2、Cookie的基本属性

- **Name** 名称

- **Value** 值

- **Domain** 域。默认为当前文档位置的路径的域名部分，只能设为上级域名

- **Expires** 过期时间

- **Max-Age** 距离过期的秒数。（IE6/7/8不支持Max-Age）

- **Path** 路径。默认为当前文档位置的路径。一般会设为 '/'

- **Secure** 布尔值。cookie只会被https传输

- **HttpOnly** 布尔值。如果设置了httpOnly，那么通过JS脚本（document.cookie）不能读取到cookie

###3、JS操作cookie

js-cookie库提供了比较方便的接口进行cookie的读、写、删除：

```javascript
Cookies.set('name','value',opts);
Cookies.get('name');
Cookies.remove('name');
```



以下是原生操作cookie的方式：

#### 3.1  如何获取cookie

```javascript
// 获取到的cookie格式如 name1=value1; name2=value2; name3=value3
// document.cookie只能获取到name和value，因为
document.cookie

// 法一：for循环方法获取某个具体的cookie的value的方法
function getCookie(cname) {
    // 解码
    var decodedCookie = decodeURIComponent(document.cookie);
    // 用';'将一个字符串decodedCookie分割成字符串数组ca
    var carr = decodedCookie.split('; ')
    for(var i = 0; i < carr.length; i++) {
        var singleCarr = carr[i].split('=')
        if(singleCarr[0] === cname) {
            return singleCarr[1]
        }
    }
    return ''
}
// 法二：正则获取某个具体的cookie的value的方法
```



#### 3.2  如何增加cookie

```javascript
document.cookie = 'username=cover'
// 当然设置的字符串要先进行URI编码
document.cookie = encodeURIComponent('username') + '=' +
    			 encodeURIComponent('cover');
// 再来添加cookie的其它一些属性
document.cookie = encodeURIComponent('username') + '=' +
    			 encodeURIComponent('cover') +
    			 ';domain=ke.qq.com' +
                  ';expires=' + new Date('2019-1-1');
```



#### 3.3 如何删除一条cookie

```javascript
//给过期时间expires设置任何一个过去的时间
document.cookie = encodeURIComponent('username') + '=' +
    			 encodeURIComponent('cover') +
    			 ';domain=ke.qq.com' +
                  ';expires=' + new Date(0);
```



#### 3.4 封装一个原生 JS 操作cookie的库

```javascript
/**
 * 实现简单 cookie 封装库
 */

 // Cookie 对象
 Cookie = {

    /**
     * TODOS: 实现 cookie 设置方法
     * @param {String} name 属性名
     * @param {String} value 属性值
     */
    set: function(name, value) {
        document.cookie = encodeURIComponent(name) + '=' +
                          encodeURIComponent(value);
    },

    /**
    * TODOS: 实现 cookie 获取方法
    * @param {String} name 属性名
    * @return {String} 返回对应的属性值
    */
    get: function(name) {
        // (?:^| ) 匹配字符串的开始或者空格
        // (.*?) 匹配除换行符外的任意字符，重复0或更多次，且是惰性匹配
        // (?:;|$) 匹配分号或者字符串的结尾
        var reg = new RegExp('(?:^| )' + name + '=(.*?)(?:;|$)')
        // exec() 匹配失败返回null
        var ret = reg.exec(document.cookie)
        // 逻辑与（&&）如果运算符左边能转换成false则返回左边的值，否则返回右边的值
        return ret && ret[1]
    },

    /**
    * TODOS: 实现 cookie 删除方法
    * @param {String} name 属性名
    */
    remove: function(name) {
        document.cookie = encodeURIComponent(name) + '=' +
                          '' + 
            			 ';expires=' + new Date(0);
    }

}

// 正确运行下面代码
Cookie.set('username', 'kevin'); // 增加 username=kevin cookie
Cookie.get('username'); // kevin
Cookie.remove('username'); // 删除 username=kevin cookie
```

### 4. 使用cookie保存用户信息

cookie保存登录信息的原理：

![服务器是如何保存用户的登录信息的](C:\Users\ZHENG.Zheng-PC\Desktop\QQ截图20180411174814.png)

注销登录的原理：

![注销登录的原理](C:\Users\ZHENG.Zheng-PC\Desktop\QQ截图20180411175132.png)



通常在web服务中，都是通过服务器在不同用户浏览器中设置特定cookie的方式来标识用户的。为了安全起见，这些标识的 cookie 会经过一些加密等操作，防止别人直接猜到用户的 cookie 的值。

另外，这些比较重要的 cookie 通常还添加了 'HttpOnly' 属性，让 js 就没法去操作这些 cookie，这也是出于安全考虑。