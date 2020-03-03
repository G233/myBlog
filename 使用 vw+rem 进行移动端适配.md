# 引言
移动端适配方案千千万：弹性盒子布局，媒体查询，百分比长度，rem 单位，是否了解了很多手段却不知道如何在项目中实践，今天我来告诉大家我是如何组合使用这些方法进行网站移动端适配的
# 淘宝Flexible的REM适配
在一次阿里AMFE无线前端团队双十一技术分享的时候，从手机淘宝团队中流传出了一种更加灵活的布局方案——Flexible布局，其原理可以概括成如下几点：
* 将视觉稿分成 100 份， 每份被称为一个单位 a 
* 将根元素的 font-size 设置为 10 a
* 即 1rem 为 10 a  （rem单位长度始终等于根元素 font-size 大小）

这样就可以使rem单位长度随着视口宽度而变化，然后使用 rem 单位进行开发，这样元素大小等就会随着不同设备的视口大小而变化，真正的实现了弹性布局，适配性非常的不错。
> 关于淘宝Flexible的具体使用方法和更深层次的原理，请移步[使用Flexible实现手淘H5页面的终端适配](https://www.w3cplus.com/mobile/lib-flexible-for-html5-layout.html)

# VW + REM 适配方案
淘宝团队的方案虽然适配性非常不错，但是是通过 js 计算出来的视口宽度，在不同的浏览器中会出现一些问题，这也是手淘团队承认过的。在看到方案原理的时候，就觉得这个把视口宽度分成一百份的理念非常熟悉，这不就是 CSS 中的 vw 单位吗，是不是可以直接使用 vw 单位设置 font-size 的值达到同样的效果呢。可能有小伙伴不太熟 vw 单位，夏敏引用一段对 vw、vh、vm 的描述
> * vw是视口宽度的单位，视口宽度是100vw
> * vh是视口高度的单位，视口高度是100vh
> * vm取vw和vh较小的一个除以100作为单位

直接将根元素的 font-sizi 设置为 10vw，即可实现相同效果

## 浏览器兼容性
这样一来 font-size 的设置就变得非常简单了，不过这种开发者都比较少用的单位还是得看看浏览器兼容性，通过 [Can I use](https://caniuse.com/#search=vw%20vh)查询到的兼容性：

![enter description here](https://img.liuxiaogu.com/blog-img/2020-3-3-1583242856092.png) 
还能接受，现代浏览器全部都兼容，再加上我这只是个人网站并不需要考虑什么 IE6、8 的兼容。

# 具体实施
理论完备了就开始改造项目了
## flexible.css
在项目根目录下新建 `assets/flexible.css` 文件

``` css
/* 移动端适配 CSS  */
html {
  font-size: 10vw !important;
}
```
然后在 `nuxt.config.js` 文件中导入

``` javascript
css: [
    "assets/flexible.css"
  ],
```
## postcss-px2rem
rem 单位的动态长度做好了，但是我项目里面的单位全部都是用的 px 啊，一个一个手动计算改单位显然是不显示的，这里我们选择使用`postcss-px2rem` 自动进行单位的换算，首先安装一下

``` javascript
npm install postcss-px2rem –save
```
然后在 `nuxt.config.js`中引入

``` javascript

postcss: [
   require('postcss-px2rem')({
     remUnit: 75
   })
],
```
这个 remUnit 参数就是 1rem = 多少 px ，设置为 75 就是 1rem=75px，正好是 750px 设计稿的 1/10，这样一来开发的时候直接按照设计稿上标注的大小写就好了，很方便。`npm run dev` 看看效果怎么样。
打开浏览器控制台查看样式：
![截图](https://img.liuxiaogu.com/blog-img/2020-3-3-1583245012069.png)
可以看到距离单位已经全部变成了 rem ，打开已计算面板
![enter description here](https://img.liuxiaogu.com/blog-img/2020-3-3-1583245062102.png)
可以看到随着宽度的变化，这些边距都在变化。但是当宽度变得很大的时候，rem 不可控制的变得超级大，这显然不是我们想要的效果，通过查看淘宝团队方案的源码，发现当视口宽度大于 540px 的时候会将基础字体大小保持在 54px，于是对 `assets/flexible.css` 进行更改

``` javascript
/* 移动端适配 CSS  */
html {
  font-size: 10vw !important;
}
@media (min-width: 540px) {
  html {
    font-size: 54px !important; /*no*/
  }
}
```
这样一来在平板等设备上也能正常显示了。至此移动端适配方案就完成了。

# 总结
可以发现这套适配方案主要是针对不同大小的移动设备进行适配，PC 端和移动端的适配我还是通过使用了部分媒体查询来实现的。vw 单位兼容性问题也是值得注意的事情，但是后来找到了`lib-flexible.js`的 github 仓库发现有对于 vw 单位的说明
> 由于viewport单位得到众多浏览器的兼容，lib-flexible这个过渡方案已经可以放弃使用，不管是现在的版本还是以前的版本，都存有一定的问题。建议大家开始使用viewport来替代此方案。vw的兼容方案可以参阅[《如何在Vue项目中使用vw实现移动端适配》](https://www.w3cplus.com/mobile/vw-layout-in-vue.html)一文。

可知现在这套 js 的方案已经被弃用，vw 单位已经登堂入室了，放心大胆的用起来吧。

## 参考文章
- [再谈移动端适配](https://blog.lenconda.top/posts/2018/04/11/mobile_adjust_again/)
- [使用Flexible实现手淘H5页面的终端适配](https://www.w3cplus.com/mobile/lib-flexible-for-html5-layout.html)