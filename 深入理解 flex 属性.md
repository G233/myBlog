# 引言
最近想更改一下博客首页的布局样式，添加几个侧栏，就去看了看常见的布局方式，什么「双飞翼布局」「圣杯布局」也就两侧固定中间自适应的三栏布局方案。不过我之前一直都是用的 Flex 布局，Flex 布局是不是也可以做到呢，网上搜索一番，代码如下：
```html
<html lang="en">
  <body>
    <div style="display: flex;">
      <div class="a"></div>
      <div class="b"></div>
      <div class="c"></div>
    </div>
  </body>

  <style>
    .a {
      background-color: thistle;
      width: 20%;
    }
    .b {
      background-color: tomato;
      flex: 1;
    }
    .c {
      background-color: violet;
      width: 100px;
    }
    .a,
    .b,
    .c {
      height: 100px;
    }
  </style>
</html>
```
这样一看是不是非常简单，比前文介绍的两种布局优雅多了，但是让我困惑的是第 17 行的 `flex: 1`，在阮一峰老师的博客上是这样描述的
> flex属性是flex-grow, flex-shrink 和 flex-basis的简写，默认值为0 1 auto。

Google 搜索出来的结果也都是说 flex 属性就是简写，那为什么 `flex：1`可以实现中间部分自适应，这黑箱一般的原理一直困扰着我，flex 到底是怎么起作用的呢，下面我们来详细研究研究。（关于这几个属性的意义可以看阮一峰老师的[博文](http://www.ruanyifeng.com/blog/2015/07/flex-grammar.html?utm_source=tuicool))）
<a name="S9RHc"></a>
# flex 属性的语法
首当其冲的困惑就是 `flex`属性是`flex-grow`，`flex-shrink`和`flex-basis`的缩写。那么为什么会有`flex:1`这种后面接一个数字的呢？这就不得不提到 flex 的语法，不知道同学们有没有看 CSS 语法的习惯，查询 MDN 可知其正式语法为
> none | [ <'flex-grow'> <'flex-shrink'>? || <'flex-basis'> ]

感觉看起来眼都花了，这怎么理解嘛，别急我们一个一个来分析，`|`这个符号表示排他，可以为后者或者前者，所以这就有两种语法了
```css
flex: none;
flex: [ <'flex-grow'> <'flex-shrink'>? || <'flex-basis'> ]
```
接下来就是方括号`[]`，表示范围，也就是说 flex 属性后面跟随的以三个数就是表示这三个属性的，比如：
```css
{
  flex:1 1 100px
}
/* 等于 */
{
  flex-grow:1
  flex-shrink:1
  flex-basis:100px
}
```
再看符号`||` 表示的意思是「或者」，可以为 `flex: [  <'flex-basis'> ]`或者为`flex: [ <'flex-grow'> <'flex-shrink'>]` ，也可以共存。而符号`？`表示 0 或者 1，也就是说`flex-shrink`可有可无，这样一来一共是有以下几种组合方式。
```css
flex: [ <'flex-basis'> ]
flex: [ <'flex-grow'>]
flex: [ <'flex-grow'> <'flex-shrink'>]
flex: [ <'flex-grow'> <'flex-basis'> ]
flex: [ <'flex-grow'> <'flex-shrink'><'flex-basis'> ]
```
可以发现 flex 属性后面跟一个值的情况就是 1 和 2 行，而 `flex-basis` 的单位是长度单位，所以`flex:1` 就相当于 `flex-grow : 1`，原来如此，这样设置就使元素填满剩余空间。从完成「双飞翼布局」角度来说这个疑问好像已经得到了解答，但是 flex 属性的这三个值却远远没有这么简单。<br />

<a name="Q1Nl6"></a>
# 深入 flex 属性值
简单介绍一下这三个属性

- **flex-grow**
  - 指定了容器剩余空间多余时候的分配规则，默认值是`0`，多余空间不分配。
- **flex-shrink**
  - 指定了容器剩余空间不足时候的分配规则，默认值是`1`，空间不足要分配。
- **flex-basis**
  - 指定了固定的分配数量，默认值是`auto`。如会忽略设置的同时设置`width`或者`height`属性。
<a name="XBvDZ"></a>
## flex-grow 计算规则
<a name="UlAZm"></a>
### 对于单个元素来说

- 0 = flex-grow
  - 若有剩余空间不进行分配，也就是长度不会变化，width 设置多宽就是多宽，这也是默认状态
- 1 >= flex-grow
  - 有剩余空间就分配，在只有一个元素的情况下，效果就是元素占满
- 0 < flex-grow < 1
  - 这种情况比较少见，元素分配空间为可分配空间的 flex-grow / 1，看一下实例比较直观

演示代码如下：
```html
<html>
  <body>
    <div class="flex">
      <div class="a"></div>
    </div>
  </body>
  <style>
    .flex {
      display: flex;
      background-color: violet;
    }
    .a {
      background-color: thistle;
      width: 100px;
      flex-grow: 0.1;
      height: 100px;
    }
  </style>
</html>

```
元素 a 原本的宽度值为 100px ，然后手动将 flex 宽度设置为 500px，这个时候再查看元素 a 的实际宽度，为 140px
![enter description here](https://img.liuxiaogu.com/blog-img/2020-5-5-1588665599028.png)
![enter description here](https://img.liuxiaogu.com/blog-img/2020-5-5-1588665642254.png)
分配空间（140-100 = 40）=可分配空间（500-100=400px）* flex-grow （0.1）<br />所以元素实际宽度 = 自身长度（100）+分配空间（40）= 140px
<a name="K75l2"></a>
### 多个元素
也是有三种情况，为 0 很容易理解就不介绍了
<a name="rtoMp"></a>
#### 都大于 1

  - 这个时候剩余空间按照 flex-grow 的值按比例分配，上代码
```html
<html lang="en">
  <body>
    <div class="flex">
      <div class="a"></div>
      <div class="b"></div>
      <div class="c"></div>
    </div>
  </body>

  <style>
    .flex {
      display: flex;
      background-color: violet;
    }
    .a {
      background-color: thistle;
      width: 100px;
      flex-grow: 1;
    }
    .b {
      background-color: tomato;
      width: 100px;
      flex: 2;
    }
    .c {
      background-color: violet;
      width: 100px;
      flex: 3;
    }
    .a,
    .b,
    .c {
      height: 100px;
    }
  </style>
</html>
```
一样手动将 flex 元素宽度设置为 500px，我们来看一下三个元素的实际宽度
![enter description here](https://img.liuxiaogu.com/blog-img/2020-5-5-1588665712187.png)
![enter description here](https://img.liuxiaogu.com/blog-img/2020-5-5-1588665722479.png)
![enter description here](https://img.liuxiaogu.com/blog-img/2020-5-5-1588665728136.png)
我们来看看实际宽度是怎么计算出来的<br />a 实际宽度 = 可分配空间（500-300=200）* （1 / 1+2+3）+设置宽度（100）= 33.33 + 100 =133.33<br />其他几个也是这样计算的，应该还是挺容易理解的吧
#### 当所有元素的 flex-grow 值相加小于 1

  - 这个时候不会将所有可分配空间分配出去，实际分配空间 = 可分配空间 * flex-grow 之和，上代码瞧瞧
```html
<html lang="en">
  <body>
    <div class="flex">
      <div class="a">a</div>
      <div class="b">b</div>
      <div class="c">c</div>
    </div>
  </body>

  <style>
    .flex {
      display: flex;
      background-color: violet;
    }
    .a {
      background-color: thistle;
      width: 100px;
      flex-grow: 0.1;
    }
    .b {
      background-color: tomato;
      width: 100px;
      flex-grow: 0.2;
    }
    .c {
      background-color: aqua;
      width: 100px;
      flex-grow: 0.3;
    }
    .a,
    .b,
    .c {
      height: 100px;
    }
  </style>
</html>

```
![enter description here](https://img.liuxiaogu.com/blog-img/2020-5-5-1588665774235.png)
可以看到元素们并没有占满空间， 看看实际宽度

![enter description here](https://img.liuxiaogu.com/blog-img/2020-5-5-1588665785240.png)
<br />a 实际宽度 = 实际分配空间 * （0.1 / 0.1+0.2+0.3）+设置宽度（100）= 20  + 100 =120<br />和大于 0 情况相比较的话仅仅是「可分配空间」->「实际分配空间」，而上文提到过 
> 实际分配空间 = 可分配空间 * flex-grow 之和

也就是本实例中的实际分配空间为 200 * （0.1+0.2+0.3）=120，其实就是把可分配空间与 flex-grow 之和相乘，然后按比例分配空间。
<a name="mE1tH"></a>
#### 元素 flex-grow 值都小于 0 但是相加大于 1
这种情况其实想一想就能得出结论，和大于 1，就是占满可分配空间，然后按照比例分配
<a name="nwwQf"></a>
#### 元素 flex-grow 有大于 1 有小于 1
其实也是一样结论，占满可分配空间之后，然后按照比例分配
<a name="bM4OB"></a>
#### 结论
到这我们就可以得出结论了，其实这种对单个 flex-grow 值进行分类的方式是错误的，而应该按照所有 flex-grow 相加是否大于 1，因为大于 1 就将所有剩余空间按比例分配，小于 1 则需要先计算实际分配空间再按比例分配，不会占满所有剩余空间。
<a name="iBSiZ"></a>
## flex-shrink 计算规则
flex-shrink 属性是当父元素长度不够的时候各元素的缩小规则，默认为 1 ，也就是大家都按照相同比例缩小，至于具体怎么缩小，是不是也像 flex-grow 那样按照数值等比例缩小呢，并没有那么简单，我们先来看个实例，元素宽度相等的情况下：
```html
<html lang="en">
  <body>
    <div class="flex">
      <div class="a">a</div>
      <div class="b">b</div>
      <div class="c">c</div>
    </div>
  </body>

  <style>
    .flex {
      display: flex;
      background-color: violet;
    }
    .a {
      background-color: thistle;
      width: 200px;
      flex-shrink: 1;
    }
    .b {
      background-color: tomato;
      width: 200px;
      flex-shrink: 2;
    }
    .c {
      background-color: aqua;
      width: 200px;
      flex-shrink: 3;
    }
    .a,
    .b,
    .c {
      height: 100px;
    }
  </style>
</html>

```
手动将 flex 元素宽度设置为 400px ，这个时候看三个子元素的实际宽度
![enter description here](https://img.liuxiaogu.com/blog-img/2020-5-5-1588665829554.png)
![enter description here](https://img.liuxiaogu.com/blog-img/2020-5-5-1588665834954.png)
![enter description here](https://img.liuxiaogu.com/blog-img/2020-5-5-1588665840895.png)
很容易可以通过计算得出
> 元素 c 收缩长度 = 实际长度（200） - 超出长度（600-400）*（3/1+2+3） = 100

此时计算方式确实和 flex-grow 一模一样，但是当各元素长度不一样的时候呢？
```html
<html lang="en">
  <body>
    <div class="flex">
      <div class="a">a</div>
      <div class="b">b</div>
      <div class="c">c</div>
    </div>
  </body>

  <style>
    .flex {
      display: flex;
      background-color: violet;
    }
    .a {
      background-color: thistle;
      width: 100px;
      flex-shrink: 1;
    }
    .b {
      background-color: tomato;
      width: 200px;
      flex-shrink: 2;
    }
    .c {
      background-color: aqua;
      width: 300px;
      flex-shrink: 3;
    }
    .a,
    .b,
    .c {
      height: 100px;
    }
  </style>
</html>
```
![enter description here](https://img.liuxiaogu.com/blog-img/2020-5-5-1588665937619.png)

![enter description here](https://img.liuxiaogu.com/blog-img/2020-5-5-1588665942108.png)

![enter description here](https://img.liuxiaogu.com/blog-img/2020-5-5-1588665946548.png)
a 收缩长度 = 14.28<br />b 收缩长度 = 57.14<br />c 收缩长度 = 128.58<br />可以看到收缩长度并不和元素各自的 flex-shrink 值成正比，查阅 W3 文档和 MDN 也没有发现详细的计算方式，Google 了一大圈终于破解了这个秘密，原来与 flex-grow 延伸长度只与 flex-grow 值相关不一样，flex-shrink 元素的收缩长度不仅和 flex-shrink 相关，还与元素自身大小相关，具体计算方式如下：
> 收缩长度 = (元素宽度 * flex-shrink) / (100 * 1+200 * 2+300 * 3) * 需要收缩宽度 

代入本例计算一下：<br />a 元素收缩长度 = 100 * 1/ (100 * 1+200 * 2+300 * 3) * ( 600-400 ) = 14.28 =100 - 85.72<br />与实际情况相复合，flex-shrink 了解完毕。 等着！ 忘了 flex-grow 里面当 flex-grow 相加小于 1 这种情形了吗，那么在这是怎么表现的呢，看看实例：
```html
<html lang="en">
  <body>
    <div class="flex">
      <div class="a">a</div>
      <div class="b">b</div>
      <div class="c">c</div>
    </div>
  </body>

  <style>
    .flex {
      display: flex;
      background-color: violet;
    }
    .a {
      background-color: thistle;
      width: 100px;
      flex-shrink: 0.1;
    }
    .b {
      background-color: tomato;
      width: 200px;
      flex-shrink: 0.2;
    }
    .c {
      background-color: aqua;
      width: 300px;
      flex-shrink: 0.3;
    }
    .a,
    .b,
    .c {
      height: 100px;
    }
  </style>
</html>

```
![enter description here](https://img.liuxiaogu.com/blog-img/2020-5-5-1588665995046.png)
可以很直观的看到，三元素长度相加是大于 flex 元素长度的，也就是说并没有「完全」收缩，各个元素长度为
![enter description here](https://img.liuxiaogu.com/blog-img/2020-5-5-1588666015968.png)

![enter description here](https://img.liuxiaogu.com/blog-img/2020-5-5-1588666022026.png)

![enter description here](https://img.liuxiaogu.com/blog-img/2020-5-5-1588666025913.png)
通过类比 flex-grow 可以发现：
> 实际总收缩长度 = 需要收缩长度 * flex-shrink 之和

在本例中，实际收缩长度 = 200 * 0.6 = 120 =600 - 480（收缩后总长度），与实际符合，得出实际收缩长度之后各元素收缩长度就和上文提到的方法是一样的了。
<a name="2jUD0"></a>
## 计算方式总结
虽然上面的计算公式看起来比较复杂，其实理解了之后还是比较简单的，借用知乎[@谢然](https://zhuanlan.zhihu.com/p/24372279)的总结：
> 如果所有元素的 flex-grow/shrink 之和大于等于 1，则所有子元素的尺寸一定会被调整到适应父元素的尺寸（在不考虑 max/min-width/height 的前提下），而如果 flex-grow/shrink 之和小于 1，则只会 grow 或 shrink 所有元素 flex-grow/shrink 之和相对于 1 的比例。grow 时的每个元素的权重即为元素的 flex-grow 的值；shrink 时每个元素的权重则为元素 flex-shrink 乘以 width 后的值。

<a name="Vw9OG"></a>
# 总结
说到 flex 布局，谁还不能说上几句，仿佛全世界都在用 flex 布局了，但是你真的了解吗，我其实最早开始接触 CSS 的时候就是使用的 flex 布局，垂直居中什么的玩得特别 6 ，但是在这几天才对 flex 属性有了真正的认识。这也让我认识到不求甚解在技术界是一种陋习，直接照着写上一句 `flex:1`确实很容易实现三栏布局，但是当需求发生变化就懵逼了。只有真正了解了底层原理才能随心所欲的实现心中所想。<br />还有一点就是你的信息来源渠道很大程度上决定了你信息的准确性与真实性，官网文档永远是最正确最完善的，尽量去信息源头寻找答案。强如阮一峰老师的flex教程中对于 flex 属性的描述也不够清楚，Google 永远都比百度好使。<br />到这我们的深入挖掘就告一段落了，可能你发现不是还有 flex-basis 没有讲吗，这是因为 flex-basis 也是一个有非常多细节的属性，但是其细节又与这两个需要计算的不一样，值得单独写一篇文章来进行介绍，在下篇文章中我们再来细聊。
<a name="dWXO8"></a>
# 参考文章

- [MDN flex](https://developer.mozilla.org/zh-CN/docs/Web/CSS/flex)
- [CSS Flexible Box Layout Module Level 1](https://link.zhihu.com/?target=https%3A//www.w3.org/TR/css-flexbox-1/%23valdef-flex-flex-shrink)
- [`flex-grow` is weird. Or is it?](https://css-tricks.com/flex-grow-is-weird/)
- [详解 flex-grow 与 flex-shrink](https://zhuanlan.zhihu.com/p/24372279)
- [CSS flex属性深入理解链接](https://www.zhangxinxu.com/wordpress/2019/12/css-flex-deep/)
