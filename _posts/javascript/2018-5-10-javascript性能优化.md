---
layout: blog
javascript: true
background-image: https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1514897300420&di=98b95608be22799574c540dcb1d23ea2&imgtype=0&src=http%3A%2F%2Fpic.92to.com%2F201612%2F14%2F2016128110105715.jpg
category: javascript
title: JS性能优化方法
tags:
- js性能
---

## 加载和执行

### JS影响页面渲染的原因：
> 当浏览器遇到script标签的时候，HTMl页面无法获取到javascript是否会改变页面中标签的内容，浏览器就会停止处理页面，先执行js代码，然后再解析和渲染页面；同样适用src的外链的js文件，会下载解析执行完js在渲染页面

### 脚本位置
> HTML4规范指出script标签可以放在HTML的head和body标签中，并允许出现多次；JS放在页面的顶部，会出现页面加载会空白； 所以推荐放在尾部


### 组织脚本
> 每个script去下载js都会阻塞页面的渲染，所以减少script标签的数量也可以有效的提升页面加载速度

### 无阻塞脚本

> 向页面逐步加载js文件；意味着window对象的load事件触发后再下载脚本
#### 延时脚本
1. defer属性：该属性指明改JS脚本不会修改DOM代码可以安全的延时执行；
2. async属性：和上边的是一样的
> defer和async的区别： 都是并行下载；async是加载完后自动执行；defer是加载完后等待页面完成在执行；
- [x] defer加载的JS脚本会在window.onload之前调用；

#### 动态脚本元素
> 动态创建script标签来引入js文件；这样创建的JS文件下载不会影响页面的渲染，可以放在任何的位置，但是会出现一个BUG，浏览器对这个的处理是不相同的；有的下载完自动执行，有的等到其他的动态节点下载完会才执行；如果加载JS文件是页面中js的方法，会出先错误；需要先确保本js执行加载再进行操作利用load事件；IE会有一个readystatechange事件为script元素添加一个readyState属性；

```javascript
var script = document.creatElement('script');
script.type = 'text/javascript';
if (script.readyState) {
    script.onreadystatechange = function() {
        if (script.readyState === 'complete' || script.readyState === 'loaded') {
            // 移除script的动态标签事件
            script.onreadystatechange = null;
            // 执行接下来的操作
        }
    }
    
} else {
    script.onload = function() {
        // 加载完毕执行的操作
    }
}

script.src = './js/file.js';
document.getElementsByTagName('head')[0].appendChild(script);
```
#### XMLHttpRequest脚本注入
> 利用XMLHttpRrquest（XHR）对象获取脚本注入到页面中；先传建一个xmr对象，下载js文件，最后通过创建动态的script元素将代码注入到页面中；

```javascript
const XHR = new XMLHttpRequest();
XHR.open('get', 'file.js', true);
XHR.onreadystatechange = function() {
    if (XHR.readyState === 4) {
        if (XHR.status >= 200 && XHR.status < 300 || XHR.status == 304) {
            var script = document.createElement('script');
            script.type = 'text/javascript';
            script.text = XHR.responseText;
            document.body.appendChild(script);
        }
    }
}
 
```
### 数据存储
>  字面量的存储读取速度和局部变量是相同都是快于数组项和对象属性的读取速度的；
- [x] 建议尽量使用局部变量来缓存都次操作的数组项或者对象属性

#### 作用域链和标识符
> 函数被调用时会创建活动对象；活动对象就是执行环境+全局对象；标识符的解析过程就是查找变量的过程；正是这个过程消耗性能的；
- [x] 标识符的位置越深解析越慢，消耗的性能就越多；最快的就是函数的局部变量；尽量使用局部变量；如果某一个跨作用域的值在函数中被引用2次以上就将他缓存到局部变量；

#### 对象
- [x] 嵌套成员的会影响性能window.location.href性能低于location.href；
- 函数分为自身属性和继承属性；查找的属性在原型链的位置越深，越消耗性能；最好的办法利用变量缓存对象成员的值

### DOM优化
> 为啥操作DOM耗性能：文档对象模型DOM是一个独立于语言的，用于操作XML和HTML文档的程序接口（API）；浏览器通常会把DOM和js单独实现，当你用js操作DOM时调用DOM的API，相互独立的功能只要通过接口批链接就会产生性能消耗；
#### DOM的访问和修改
- 最坏的情况就是通过循环，不断利用js调用DOM的接口；
- [x] 减少DOM的次数，把运算尽量留在ECMAScript端完成；
```javascript
// 耗能写法
for (var i = 0; i < 10; i++) {
    document.getElementById('show').innerHTML += 'a';
}
// 高效写法
var html = '';
for (var i = 10; i--;) {
    html = html + 'a';
}
document.getElementById('show').innerHTML = html;
// DOM的操作次数越多代码消耗的性能越多
```
- 选用性能更好的API来操作DOM；
- [x] 使用el.cloneNode()(el表示已有的节点)来代替document.createElement()来创建节点
- HTML合集；包含了DOM节点引用的类数组对象；
    1. document.getElementsByTagName();等方法返回的类数组的DOM集合
    > HTML集合以一种‘假定实时态’实时存在，这意味着底部文档更新时，他也会自动更新；下边的例子就是一个死循环
    
    ```javascript
    var div = document.getElementsByTagName('div');
    for (var i = 0; i < div.length; i++) {
        document.body.appendChild(document.createElement('div'));
    }
    // 会进入一个死循环，因为每次获取的div的数量都是最新的div数量；
    // 正确做法：先将长度存贮
    var div = document.getElementsByTagName('div')，
    len = div.length;
    
    for (var i = 0; i < len; i++) {
        document.body.appendChild(document.createElement('div'));
    }
    ```

    2. 多次访问一个集合的元素的多个属性的时候建议用变量缓存
    
    ```javascript
    var divs = document.getElementsByTagName('div');
    len = divs.length,
    el = null;
    
    for (var i = 0; i < len; i++) {
    // 利用变量缓存
        el = divs[i].style;
        w = el.width;
        h = el.height;
    }
    
    ```
- 遍历DOM；
> 遍历查找DOM元素的方法有很，我们需要利用那些高性能的API来进行遍历的操作

1. 只返回DOM元素的API；children；firstElementChild等
2. 选择器API；原生的获取DOM的方法；还有新家的利用css选择器获取元素的API，document，querySelectorAll（选择器）

### 重绘重排参照单独的文章

### 事件委托；

### 算法与流程控制

#### 循环
- [x] 提高循环性能的两个方面；
1. 减少每次迭代的代码工作量


```javascript
// 减少对象或者数组的属性查询次数，这里缓存了数组的length属性
for (let i = 0, len = arr.lengthl; i++) {
    //操作
}
// 数组项的书序与要执行的任务无太大的关联，建立使用倒序循环的方法操作
// 将减法放在条件判断中；
for (let i = arr.length; i--; ) {
    // 操作
}

```

2. 减少循环次数

```javascript
// 限制循环迭代次数模式：‘达夫设备’，一般都是千次以上的循环
let startAn = len % 8,
count = Math.floor(len / 8);
while(startAn --) {
    // 操作函数
}
while(count --) {
    // 8次操作函数
}
```
3. 基于函数的迭代；forEach(function(value, index, arr) {});但是这个还是比基础的循环慢一点；

#### 条件判断语句
- [x] 根据判断的数量来确定用if-else还是用switch；数量大的时候switch比if快
1. 简化判断的流程；
> 如果存在多重的判断可以选取中间值判断来

```javascript
// 常规写法，如果是0；判断次数过于多
if ( i === 1) {
    return 1;
} else if ( i === 2) {
    return 2
} else if ( i === 3) {
    return 3
} else if ( i === 4) {
    return 4
} else if ( i === 5) {
    return 5
} else if ( i === 6) {
    return 6
} else if ( i === 7) {
    return 7
} else if ( i === 8) {
    return 8
} else if ( i === 9) {
    return 9
} else {
    return 0
}
// 高效写法

if (i >= 5) {
    if (i >= 7) {
        if (i === 7) {
            return 7;
        } else if (i === 8) {
            return 8
        } else {
            return 9;
        }
    } else {
        if (i === 6) {
            return 6;
        } else {
            return 5;
        }
    }
    
} else {
    if (i >= 3) {
        if ( i === 3) {
            return 3
        } else if (i === 4) {
            return 4
        }
    } else {
        if ( i === 2) {
            return 2;
        } else if ( i === 1) {
            return 1;
        } else {
            return 0;
        }
    }
}

```

2. 查找表；就是让你判断的数据和数组或者对象对应起来，操作更加快速高效

3. 递归操作

```javascript
function quickSort(arr) {
    let len = arr.length;
    if (len <= 1) {
        return arr;
    }
    let index = Math.floor(len / 2),
    temp = arr[index],
    left = [],
    right = [];
    for (let i = len; len --; ) {
        if (arr[i] < temp) {
            left.push(arr[i]);
        } else {
            right.push(arr[i]);
        }
    }
    return quickSort(left).concat(temp, right);
}
```
4. 避免重复的工作；

```javascript
// 普通的阶乘递归
function leav(num) {
    if (num === 1) {
        return num;
    }
    return num * leav(num - 1);
}
// 递归阶乘的优化
// 这里存在这性能的问题，当你求的3的阶乘后在求5的阶乘，相当于将3的阶乘连续求值了2次；
function quickLeav(num) {
    if (!quickLeav.cache) {
        quickLeav.cache = { 1 : 1};
    }
    if (quickLeav.cache.hasOwnProperty(num)) {
        return quickLeav.cache[num];
    } else {
        quickLeav.cache[num] = num * quickLeav(num - 1);
    }
}
```
### 字符串
- [x] 下边拼接字符串发生了什么
> str += 'one' + 'two'
1. 创建临时字符串
2. 链接‘one’‘two’赋值给该临时字符串
3. 临时字符串与str字符串链接
4. 结果赋值给str；
- [x] 可以想办法控制不要产生临时字符串，这样可以减少1，2步，提高性能；
> str += 'oen'; str += 'two'; <=====> str = str + 'one' + 'two'; 将str当作临时字符串直接操作

> 字符串拼接的原理：表达式左侧被复制的字符串变量会被分配一个比较大的内存，基础字符串（就是表示排在最前边的字符串）；所以在赋值字符串的时候可以将左侧的字符串当作基础字符串进行赋值的操作，避免了重复拷贝一个不断变大的基础字符串；




### 对于js在构建，部署时的操作
1. 合并多个js文件；减少http请求
2. 预处理js文件
3. js文件压缩
> js的HTTP压缩；Gizp格式服务器；发送请求Accept-Encoding请求头告诉服务器，web端支持哪个编码；服务端利用Content-Encoding通知web端，服务器发送的文件格式；一般是gizp的压缩格式；
4. 缓存js文件同样适用于其他的图片等静态资源；
    1. 服务器端设置Expires：时间，来强制缓存，时间是过期时间；
    2. 利用本地存储来缓存文件，自己设置过期时间
5. CDN（将js文件发布在CDN上可以快速的下载）















1. 避免使用 eval 或 Function 构造器
> 每次进行 eval 或调用 Function 构造器，脚本引擎都会启动一些机制来将字符串形式的源代码转换为可执行的代码。代码是在调用 eval 的上下文档中解释，这就意味着编译器无法优化相关上下文，就会留给浏览器很多需要在运行时解释的内容。这就造成了额外的性能影响。
2. 不要使用 with
> with 会在引用变量时为脚本引擎构造一个额外的作用域。
3. 尽量不用全局变量
> 在函数中查找全局变量没有查找局部变量块，作用域链的问题；
4. 在要求性能的函数中避免使用 for-in
> for-in 循环需要脚本引擎为所有可枚举的属性创建一个列表，然后检查其中的重复项，之后才开始遍历。
5. 基本运算比调用函数更快

```javascript
// 慢
var min = Math.min(a,b);
A.push(v);
// 快
var min = a < b ? a : b;
A[A.length] = v;
```
