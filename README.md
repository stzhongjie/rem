### 为什么要使用rem
 **** 
之前有些适配做法，是通过js动态计算viewport的缩放值（initial-scale）。

> 例如以屏幕320像素为基准，设置1，那屏幕375像素就是375/320=1.18以此类推。

但直接这样强制页面缩放过于粗暴，会导致页面图片文字失真模糊。

Px是相对固定单位，字号大小直接被定死，所以用户无法根据自己设置的浏览器字号而缩放，em和rem虽然都是相对单位，但em是相对于它的父元素的font-size，页面层级越深，em的换算就越复杂，而rem是直接相对于根元素，这就避开了很多层级关系。移动端新型浏览器对rem的兼容很好，可以放心使用。

### 通用换算和一些坑
 **** 

有时我们会看到有些使用rem的页面里会先给页面根元素一个样式：
```
html {font-size: 62.5%; /*10 ÷ 16 × 100% = 62.5%*/}
```

为什么是62.5%？

大多数浏览器的默认字号是16px，因此1rem=16px，这样不方便我们px和rem的换算，假设1rem=10px，那么100px=10rem，25px=0.25rem。这样就好换算很多，于是就有了上面的10/16。


如果是640的设计稿，需要除以2转化为和iphone5屏幕等宽的320。所以设计稿px单位/2/10转为rem。之后再媒体查询设置每个屏幕大小的font-size百分比，页面会根据上面设置的根font-size适配。

 _看到这里是不是觉得一切很完美？然而，这里面有两个深坑：_ 


1.我看了网上很多关于rem的资料，基本都说浏览器的默认字号就是16px，然后直接定义font-size:62.5%。但是，rem属于css3的属性，有些浏览器的早期版本和一些国内浏览器的默认字号并不是16px，那么上面的10/16换算就不成立，直接给html定义font-size: 62.5%不成立。


2.chrome强制字体最小值为12px，低于12px按12px处理，那上面的1rem=10px就变成1rem=12px，出现偏差（下面给demo）。 

**解决方案：** 将1rem=10px换为1rem=100px（或者其它容易换算的比例值）;不要在pc端使用rem。

那么上面的页面根元素样式要改为：
```
html {font-size: 625%; /*100 ÷ 16 × 100% = 625%*/}
```

再用本工厂总结得出的各分辨率媒体查询换算：
```
@media screen and (min-width:360px) and (max-width:374px) and (orientation:portrait) {
    html { font-size: 703%; }
}
@media screen and (min-width:375px) and (max-width:383px) and (orientation:portrait) {
    html { font-size: 732.4%; }
}
@media screen and (min-width:384px) and (max-width:399px) and (orientation:portrait) {
    html { font-size: 750%; }
}
@media screen and (min-width:400px) and (max-width:413px) and (orientation:portrait) {
    html { font-size: 781.25%; }
}
@media screen and (min-width:414px) and (max-width:431px) and (orientation:portrait){
    html { font-size: 808.6%; }
}
@media screen and (min-width:432px) and (max-width:479px) and (orientation:portrait){
    html { font-size: 843.75%; }
}

```

至此，坑填完。设计稿px换算/2/100即可得到rem值。

### 更精准健壮的换算 
**** 

然而，上面的625%大法除了有兼容性问题，也无法很好地根据不同设计稿精准适配，不是我们的最佳选择。网易和淘宝分别有自己的一套适配方法，适配性也很完美。

-  **网易手机端：** 
![图1](https://git.oschina.net/uploads/images/2017/0518/093212_fe78cae9_788632.png "在这里输入图片标题")
![图2](https://git.oschina.net/uploads/images/2017/0518/093311_918331a0_788632.png "在这里输入图片标题")
![图3](https://git.oschina.net/uploads/images/2017/0518/093321_60c4366f_788632.png "在这里输入图片标题") 
_基准值：_ 可以看到，无论页面以哪种手机比例缩放，body的width都是7.5rem。很明显，目前网易的手机端设计稿是基于iPhone6，750（设计师给的设计稿是物理分辨率，会是我们写样式的逻辑分辨率的两倍，如果给的设计稿是640，那么是基于iPhone5，320），且基准值是100px（750/7.5=100）。这个基准值很关键，后面的css换算，都和这个基准值有关。 
_动态font-size：_ 我们看到图1、图2、图3的font-size都有根据屏幕大小而动态改变，可以推算出公式：

    > 屏幕宽度/设计稿rem宽度=页面动态font-size值（如：375/7.5=50）

    获取到这个值，再赋给html元素的style：
```
document.documentElement.style.fontSize = document.documentElement.clientWidth / 7.5 + 'px';
```

    这样就设置好了每个页面的根fonts-size，因为rem单位是基于根font-size，因此只要确定一种设计稿对应手机的换算，其余屏幕尺寸均可自动适配。

    上面我们得出设计稿换算rem的基准值是100，因此只需要把设计稿上的px值除以100即为我们要的rem值。

        > Px/100=rem，所以100px=1rem,25px=0.25rem

-  **淘宝手机端：**  :fa-long-arrow-right: 大名鼎鼎的Flexible

     **资料引用** 

     > [大漠：使用Flexible实现手淘H5页面的终端适配](http://www.w3cplus.com/mobile/lib-flexible-for-html5-layout.html)

     > [齐神：flexible解读及应用](https://git.oschina.net/janking/Infinite-f2e/issues/79)

     > [齐神：（续）adaptive解读及应用](https://git.oschina.net/janking/Infinite-f2e/issues/82)

    很多大神包括我们公司同事都有对此适配方案做了解析，所以我这边简单综述：

     **引入：** 直接引用阿里的CDN文件（或下载到本地引入）
```
<script src="http://g.tbcdn.cn/mtb/lib-flexible/0.3.4/??flexible_css.js,flexible.js"></script>
```
 **设定：** 页面不要设定 _<meta name="viewport" content="initial-scale=1, maximum-scale=1, user-scalable=no">_ 。Flexible会自动设定每个屏幕宽度的根font-size、动态viewport、针对Retina屏做的dpr。
 
     **换算：** 假设拿到的设计稿和上述网易的一样都是750，Flexible会把设计稿分为10份，可以理解为页面width=10rem，即1rem=75px，所以根font-size（基准值）=75px。

  之后的css换算rem公式为：

   > px/75=rem,所以100px=100/75=1.33rem,50px=50/75=0.66rem



### 换算工具 
**** 
显然，可以看出px与rem的换算因为基准值的不同而有些复杂，甚至需要借助计算器的辅助。在这里推荐一个换算神器：[cssrem](https://github.com/flashlizi/cssrem)

安装好之后，做一些设置



> px_to_rem - px转rem的单位比例，假设拿到设计稿750，基准值是75，此处就设75

> max_rem_fraction_length - px转rem的小数部分的最大长度。默认为6。

> available_file_types - 启用此插件的文件类型。[".css", ".less", ".sass", ".scss"]。
![插件设置详细](https://git.oschina.net/uploads/images/2017/0518/094956_942117dd_788632.png "在这里输入图片标题")

### 上述三种换算方案的步骤和优劣 
**** 

- **通用方案** 

1、设置根font-size：625%（或其它自定的值，但换算规则1rem不能小于12px）

2、通过媒体查询分别设置每个屏幕的根font-size

3、css直接除以2再除以100即可换算为rem。

优：有一定适用性，换算也较为简单。

劣：有兼容性的坑，对不同手机适配不是非常精准；需要设置多个媒体查询来适应不同手机，单某款手机尺寸不在设置范围之内，会导致无法适配。



- **网易方案** 

1、拿到设计稿除以100，得到宽度rem值

2、通过给html的style设置font-size，把1里面得到的宽度rem值代入x :fa-chevron-right: 
document.documentElement.style.fontSize = document.documentElement.clientWidth / x + 'px';

3、设计稿px/100即可换算为rem

优：通过动态根font-size来做适配，基本无兼容性问题，适配较为精准，换算简便。

劣：无viewport缩放，且针对iPhone的Retina屏没有做适配，导致对一些手机的适配不是很到位。


- **手淘方案** 

1、拿到设计稿除以10，得到font-size基准值

2、引入flexible

3、不要设置meta的viewport缩放值

4、设计稿px/ font-size基准值，即可换算为rem

优：通过动态根font-size、viewpor、dpr来做适配，无兼容性问题，适配精准。 

劣：需要根据设计稿进行基准值换算，在不使用sublime text编辑器插件开发时，单位计算复杂。

### Demo 
**** 

下面看看demo

设计稿：基于iPhone5,宽度640。

那么在开发模式，iphone5是320，所有数值均是设计稿一半大小。

期望效果：在iPhone5中，box1宽高50px，box2宽高125px，字体15px。其他屏幕终端自动适配。

![demo-psd](https://git.oschina.net/uploads/images/2017/0518/095703_8ca232e8_788632.jpeg "在这里输入图片标题")

 **1、[62.5%方案](http://images.vrm.cn/2017/05/18/Rem1.html)** 

可以看出，基于chrome iPhone5的调试，box1宽高是60，box2宽高是150。出现了误差，就是上文提到字号最小值强制12px的原因。

 **2、[625%方案](http://images.vrm.cn/2017/05/18/Rem2.html)** 

比例正常。

 **3、[网易方案](http://images.vrm.cn/2017/05/18/Rem3.html)** 

比例正常。

 **4、[手淘方案](http://images.vrm.cn/2017/05/18/Rem4.html)** 

比例正常（Retina屏做了缩放）。

### 到底用哪种换算方案呢？
 **** 
每个人评判的标准不同。但个人更倾向flexible，动态计算viewport和针对iphone手机的dpr缩放使得页面适配更加精确，而且手淘页面用户访问量比网易页面大很多。

### 移动端有用px的时候吗？ 
**** 

有。当你的页面图片或者某一元素比例要固定，不想进行任何缩放时，rem就不适合了，这时候用px单位，能保证该元素不会因缩放而失真模糊。

### 总结 
**** 

纸上得来终觉浅，绝知此事要躬行。

各种终端适配一直是前端头疼的点，自己之前做的适配大多只是了解个大概就直接使用，没有去研究透。这次刚好接了公司的前端公会任务，【了解rem的换算方法】。原本以为是个比较简单的任务，但却着实花了不少时间，经常遇到某个点没想透进而去查资料写demo。一路下来，文章基本完成，自己之前疑惑的点也明了不少。抛砖引玉，文章有什么错漏或者新观点希望前端小伙伴们能提出，共同学习。


