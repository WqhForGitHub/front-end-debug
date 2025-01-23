## vs code debug

首先，调试配置文件不用自己创建，可以直接点击 Debug 窗口的 **`create a launch.json file`** 快速创建

**`launch/attach`**

创建 Chrome Debug 配置有两种方式：launch 和 attach，它们只是 request 的配置不同：

![](https://github.com/WqhForGitHub/juejin-book/blob/main/%E5%89%8D%E7%AB%AF%E8%B0%83%E8%AF%95%E9%80%9A%E5%85%B3%E7%A7%98%E7%B1%8D/static/4/1.jpg?raw=true)

```json
{
    "version": "0.2.0",
    "configurations": [
    {
        "name": "Launch Chrome",
        "request": "launch",
        "type": "chrome",
        "url": "http://localhost:8080",
        "webRoot": "${workspaceFolder}"
    },
        {
            "type": "chrome",
            "request": "launch",
            "name": "针对 localhost 启动 Chrome",
            "url": "http://localhost:8080",
            "webRoot": "${workspaceFolder}"
        }
    ]
}
```

**`launch`** 的意思是把 url 对应的网页跑起来，指定调试端口，然后 frontend 自动 attach 到这个端口。

但如果你已经有一个在调试模式跑的浏览器了，那直接连接上就行了，这时候就直接  **`attach`** 。

launch 的配置项里也有 **`userDataDir`** 的配置：

![](https://github.com/WqhForGitHub/juejin-book/blob/main/%E5%89%8D%E7%AB%AF%E8%B0%83%E8%AF%95%E9%80%9A%E5%85%B3%E7%A7%98%E7%B1%8D/static/4/2.jpg?raw=true)

默认是 **`true`**，代表创建一个临时目录来保存用户数据。

你也可以设置为 **`false`**，使用默认 user data dir 启动 chrome。

你也可以指定要给自定义的路径，这样用户数据就会保存在那个目录下：

![](https://github.com/WqhForGitHub/juejin-book/blob/main/%E5%89%8D%E7%AB%AF%E8%B0%83%E8%AF%95%E9%80%9A%E5%85%B3%E7%A7%98%E7%B1%8D/static/4/3.jpg?raw=true)



**`runtimeExecutable`** 

常用的还是 Chrome 和 Canary。

  

## chrome devTools debug

Chrome DevTools 的 **`Performance`** 工具是性能分析和优化的利器，因为它可以记录每一段代码的耗时，进而分析出性能瓶颈，然后做针对性的优化。

### 1. 如何用 Performance 工具分析并优化性能

**`性能分析`** 

首先，我们准备这样一段代码：

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>worker performance optimization</title>
</head>
<body>
    <script>
        function a() {
           b();
        }
        function b() {
            let total = 0;
            for(let i = 0; i< 10*10000*10000; i++) {
                total += i;
            }
            console.log('b:', total);
        }

        a();
    </script>
    <script>
        function c() {
            d();
        }
        function d() {
            let total = 0;
            for(let i = 0; i< 1*10000*10000; i++) {
                total += i;
            }
            console.log('c:', total);
        }
        c();
    </script>
</body>
</html>

```

很明显，两个 script 标签是两个宏任务，第一个宏任务的调用栈是 a、b，第二个宏任务的调用栈是 c、d。

![](https://github.com/WqhForGitHub/juejin-book/blob/main/%E5%89%8D%E7%AB%AF%E8%B0%83%E8%AF%95%E9%80%9A%E5%85%B3%E7%A7%98%E7%B1%8D/static/28/1.jpg?raw=true)

![](https://github.com/WqhForGitHub/juejin-book/blob/main/%E5%89%8D%E7%AB%AF%E8%B0%83%E8%AF%95%E9%80%9A%E5%85%B3%E7%A7%98%E7%B1%8D/static/28/2.jpg?raw=true)

![](https://github.com/WqhForGitHub/juejin-book/blob/main/%E5%89%8D%E7%AB%AF%E8%B0%83%E8%AF%95%E9%80%9A%E5%85%B3%E7%A7%98%E7%B1%8D/static/28/3.jpg?raw=true)

图中标出的 **`Main`** 就是主线程。其余的 **`Frames、Network`** 等是浏览器的其他线程。

主线程是不断执行 Event Loop 的，可以看到有两个 Task（宏任务），调用栈分别是 a、b 和 c、d，和我们分析的对上了。

**`Performance 工具最重要的是分析主线程的 Event Loop，分析每个 Task 的耗时、调用栈等信息`**。

当你点击某个宏任务的时候，在下面的面板会显示调用栈的详情（选择 **`bottom-up`** 是列表展示， **`call tree`** 是树形展示）

![](https://github.com/WqhForGitHub/juejin-book/blob/main/%E5%89%8D%E7%AB%AF%E8%B0%83%E8%AF%95%E9%80%9A%E5%85%B3%E7%A7%98%E7%B1%8D/static/28/4.jpg?raw=true)

![](https://github.com/WqhForGitHub/juejin-book/blob/main/%E5%89%8D%E7%AB%AF%E8%B0%83%E8%AF%95%E9%80%9A%E5%85%B3%E7%A7%98%E7%B1%8D/static/28/5.jpg?raw=true)

每个函数的耗时也都显示在左侧，右侧有源码地址，点击就可以跳到 **`Sources`** 对应的代码。

![](https://github.com/WqhForGitHub/juejin-book/blob/main/%E5%89%8D%E7%AB%AF%E8%B0%83%E8%AF%95%E9%80%9A%E5%85%B3%E7%A7%98%E7%B1%8D/static/28/6.jpg?raw=true)

直接展示了每行代码的耗时，太方便了。

工具介绍完了，我们来分析下代码哪里有性能问题。

很明显， b 和 d 两个函数的循环累加耗时太高了。

在 **`Performance`** 中也可以看到 **`Task`** 被标红了，下面的 **`summary`** 面板也显示了 **`long task`** 的警告。

![](https://github.com/WqhForGitHub/juejin-book/blob/main/%E5%89%8D%E7%AB%AF%E8%B0%83%E8%AF%95%E9%80%9A%E5%85%B3%E7%A7%98%E7%B1%8D/static/28/7.jpg?raw=true)

有同学可能会问：为什么要优化 **`long task`** 呢？

**`因为渲染和 JS 执行都在主线程，在一个 Event Loop 中，会相互堵塞，如果 JS 有长时间执行的 Task ，就会阻塞渲染，导致页面卡顿。所以，性能分析主要的目的是找到 long task，之后消除它。`**

找到了要优化的代码，也知道了优化的目的（消除 **`long task`**），那么就开始优化吧。


<br/>
<br/>

### 2. 会用 Performance 工具，就能深入理解 Event Loop

网页加载后，浏览器会解析 html、执行 js 、渲染 css，这些工作都是在 Event Loop 里完成的，理解了 Event Loop 就能理解网页的运行流程。

首先我们需要一个网页，我这里用的 react 测试 fiber 用的网页：

**`https://claudiopro.github.io/react-fiber-vs-stack-demo/fiber.html`**

点击 **`Performance`** 面板的 **`reload`**，录制 3 s 的数据：

![](https://github.com/WqhForGitHub/juejin-book/blob/main/%E5%89%8D%E7%AB%AF%E8%B0%83%E8%AF%95%E9%80%9A%E5%85%B3%E7%A7%98%E7%B1%8D/static/29/1.jpg?raw=true)

其中 **`Main`** 这部分就是网页的主线程，也就是执行 Event Loop 的部分：

![](https://github.com/WqhForGitHub/juejin-book/blob/main/%E5%89%8D%E7%AB%AF%E8%B0%83%E8%AF%95%E9%80%9A%E5%85%B3%E7%A7%98%E7%B1%8D/static/29/2.jpg?raw=true)

这块区域包含了所有 **`task`** 执行的流程，每个 **`task`** 的调用栈，因为像燃烧的火焰，所以也叫做 **`火焰图`** 。

鼠标划到想看的部分，向下拖动，就可以放大那个区域：

![](https://github.com/WqhForGitHub/juejin-book/blob/main/%E5%89%8D%E7%AB%AF%E8%B0%83%E8%AF%95%E9%80%9A%E5%85%B3%E7%A7%98%E7%B1%8D/static/29/3.jpg?raw=true)

左右上下拖动可以调整看的位置：

![](https://github.com/WqhForGitHub/juejin-book/blob/main/%E5%89%8D%E7%AB%AF%E8%B0%83%E8%AF%95%E9%80%9A%E5%85%B3%E7%A7%98%E7%B1%8D/static/29/4.jpg?raw=true)

展示的信息中很多种颜色，这些颜色代表着不同的含义：

灰色就代表 **`宏任务 task`** :

![](https://github.com/WqhForGitHub/juejin-book/blob/main/%E5%89%8D%E7%AB%AF%E8%B0%83%E8%AF%95%E9%80%9A%E5%85%B3%E7%A7%98%E7%B1%8D/static/29/5.jpg?raw=true)

蓝色的是 html 的 parse，橙色的是浏览器内部的 JS：

![](https://github.com/WqhForGitHub/juejin-book/blob/main/%E5%89%8D%E7%AB%AF%E8%B0%83%E8%AF%95%E9%80%9A%E5%85%B3%E7%A7%98%E7%B1%8D/static/29/6.jpg?raw=true)

紫色是样式的 reflow、repaint，绿色的部分就是渲染：

![](https://github.com/WqhForGitHub/juejin-book/blob/main/%E5%89%8D%E7%AB%AF%E8%B0%83%E8%AF%95%E9%80%9A%E5%85%B3%E7%A7%98%E7%B1%8D/static/29/7.jpg?raw=true)

其余的颜色都是用户 JS 的执行了，那些可以不同区分。

怎么从 **`Performance`** 中看出 Event Loop 执行的流程呢？

我们一起来看一下：

你会发现每隔一段时间就会有一个这种任务：

![](https://github.com/WqhForGitHub/juejin-book/blob/main/%E5%89%8D%E7%AB%AF%E8%B0%83%E8%AF%95%E9%80%9A%E5%85%B3%E7%A7%98%E7%B1%8D/static/29/8.jpg?raw=true)

放大一下是这样的：

![](https://github.com/WqhForGitHub/juejin-book/blob/main/%E5%89%8D%E7%AB%AF%E8%B0%83%E8%AF%95%E9%80%9A%E5%85%B3%E7%A7%98%E7%B1%8D/static/29/9.jpg?raw=true)

执行了 **`Animation Frame`** 的回调，然后执行了回流重绘，最后执行渲染。

这种任务每隔 **`16.7 ms`** 就会执行一次（因为我电脑是 60HZ 的刷新率，一秒 60 次，也就是 1 s / 60 = 16.7 ms 刷新一次）：

![](https://github.com/WqhForGitHub/juejin-book/blob/main/%E5%89%8D%E7%AB%AF%E8%B0%83%E8%AF%95%E9%80%9A%E5%85%B3%E7%A7%98%E7%B1%8D/static/29/10.jpg?raw=true)

这就是网页里这么执行渲染的。

所以说 **`requestAnimationFrame`** 的回调是在渲染前执行的，drAF 和渲染构成了一个宏任务。

为什么有的时候会掉帧、卡顿，就是因为阻塞的渲染的宏任务的执行：

![](https://github.com/WqhForGitHub/juejin-book/blob/main/%E5%89%8D%E7%AB%AF%E8%B0%83%E8%AF%95%E9%80%9A%E5%85%B3%E7%A7%98%E7%B1%8D/static/29/11.jpg?raw=true)

在 **`Performance`** 中宽度代表时间，超过 **`50 ms `** 就被认为是 **`Long Task`**，会被标红。因为如果 16.7 ms 渲染一帧，那 50 ms 就跨了 3、4 帧了。

我们做性能分析，就是要找到这些 **`Long Task`**，然后优化掉它。



那微任务是怎么执行的呢？

![](https://github.com/WqhForGitHub/juejin-book/blob/main/%E5%89%8D%E7%AB%AF%E8%B0%83%E8%AF%95%E9%80%9A%E5%85%B3%E7%A7%98%E7%B1%8D/static/29/12.jpg?raw=true)

有的 task 中包含 Run Microtasks，也就是说 **`micro task`** 只是 task 的一部分。

这就是这个网页的 Event Loop 执行过程。

当你对这些熟悉了之后，看到下面的火焰图，你就能分析出一些东西来了：

![](https://github.com/WqhForGitHub/juejin-book/blob/main/%E5%89%8D%E7%AB%AF%E8%B0%83%E8%AF%95%E9%80%9A%E5%85%B3%E7%A7%98%E7%B1%8D/static/29/13.jpg?raw=true)

中间比较宽的标红的就是 **`Long Task`**，是性能优化的主要目标。

一些比较窄的周期性的 Task 就是 requestAnimationFrame 回调以及 reflow、repaint 和渲染。

比较长的那个调用栈一般是递归，而且递归层数特别多。

当你展开看的时候，它也能展示完整的代码运行流程：

![](https://github.com/WqhForGitHub/juejin-book/blob/main/%E5%89%8D%E7%AB%AF%E8%B0%83%E8%AF%95%E9%80%9A%E5%85%B3%E7%A7%98%E7%B1%8D/static/29/14.jpg?raw=true)



**`总结`** 

**`Performance`** 工具能够看到网页的 Event Loop 是怎么运行的，不同的颜色代表不同的含义：

* **`灰色：task`**
* **`橙色：浏览器内部的 JS`**
* **`蓝色：html parse`**
* **`紫色：reflow、repaint`**
* **`绿色：渲染`**

其余的颜色都是用户自己的 JS。



## vue-devTools
