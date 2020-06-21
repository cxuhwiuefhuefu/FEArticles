> 文章首发于我的[GitHub博客](https://github.com/cxuhwiuefhuefu/FEArticles)，如果觉得不错，欢迎给个start哈




## 重排重绘
- 重排：
  - 当DOM的变化影响了元素的几何属性（宽和高），浏览器需要重新计算元素的几何属性，同样其他元素的几何属性和位置也会因此受到影响。
  - 浏览器会使渲染树中受到影响的部分失效，并重新构造渲染树，这个过程成为重排。
- 重绘：完成重排后，浏览器会重新绘制受影响的部分到屏幕，该过程称重绘。
- 重排重绘的危害：它们会导致Web应用程序的UI反应迟钝，所以应当尽可能减少这类过程的发生。
- 触发重排的情况
  - 添加或删除可见的DOM元素
  - 元素位置的改变
  - 元素尺寸的改变
  - 元素内容的改变（例如：一个文本被另一个不同尺寸的图片代替）
  - 页面渲染初始化（这个无法避免）
  - 浏览器窗口尺寸改变 
- 触发重绘的情况
  - 字体颜色
  - 背景颜色
- 优化
  - 读写分离操作
    ```objc
    div.style.top = '10px';
    div.style.bottom = '10px';
    console.log(div.offsetWidth);
    console.log(div.offsetHeight);
    ``` 
  - 样式集中改变
    - 通过class或cssText
    ```objc
    var left = 10;
    var top = 10;
    el.style.marginLeft = left + 'px';
    el.style.marginTop = top + 'px';

    // class
    .className{
      margin-left: 10px;
      margin-top: 10px;
    }
    el.className += ' className';
    // cssText
    var left = 10;
    var top = 10;
    div.style.cssText += 'margin-left: ' + Left + 'px; margin-top: ' + Top + 'px;'
    ``` 
  - 缓存布局信息
    ```objc
    // 触发两次重排
    div.style.left = div.offsetLeft + 1 + 'px';
    div.style.top = div.offsetTop + 1 + 'px';

    // 缓存布局信息 相当于读写分离
    var curLeft = div.offsetLeft;
    var curTop = div.offsetTop;
    div.style.left = curLeft + 1 + 'px';
    div.style.top = curTop + 1 + 'px';
    ``` 
  - 离线改变dom
    - 一旦我们给元素设置display: none;时，元素不会存在渲染树中，相当于将其从页面"拿掉"，我们之后的操作将不会触发重排和重绘，这叫做离线化。
      ```objc
      dom.display = 'none'
      // 修改dom样式
      dom.display = 'block';
      ``` 
    - 通过使用DomcumentFragment创建一个dom碎片，在它上面批量操作dom，操作完成之后，再添加到文档中，这样只会触发一次重排
    - 复制节点，在副本上工作，如何替换成它。
  - position属性为absolute或fixed
    - position属性为absolute或fixed的元素，重排开销比较小，不用考虑它对其他元素的影响。
  - 优化动画
    - 启动GPU加速，将2D的transform换成3D就可以强制开启GPU加速，提高动画性能。
      ```objc
      div {
        transform: translate3d(10px, 10px, 0);
      }
      ```         




## 虚拟DOM
- 什么是虚拟DOM？
  - 可以看做是一个使用JavaScript模拟了DOM结构的树形结构，这个树形结构包括整个DOM结构。 
- VDom特点（虚拟DOM和真实DOM的区别）
  - 虚拟DOM不会进行重排和重绘操作。
  - 虚拟DOM进行频繁操作，然后一次性修改真实DOM需要改的部分，最后并在真实的DOM中进行重排与重绘，减少过多DOM节点重排和重绘消耗。
  - 真实DOM频繁重排和重绘的效率是相当低的。
  - 虚拟DOM有效的降低大面积（真实DOM节点）的重排和排版，因为最终与真实DOM比较差异，可以只渲染局部。
  - 使用虚拟DOM的损耗计算
    ```objc
    总消耗 = 虚拟DOM增删改 + （与Diff算法效率有关）真实DOM差异增删改 + （较少的节点）重排与重绘
    ```  
  - 使用真实DOM的损耗计算
    ```objc
    总损耗 = 真实DOM完全增删改 + （可能较多的节点）重排与重绘
    ```
- 为什么使用虚拟DOM？
  - 之前使用原生JS或者jQuery写页面的时候会发现操作DOM是一件非常麻烦的事情，往往是DOM标签和JS逻辑同时写在JS文件里，数据交互是不是还要写很多的input隐藏域。如果没有好的代码规范的话会显得代码非常冗余混乱，耦合性高并且难以维护。
  - 另一个方面在浏览器一遍又一遍的渲染DOM是非常非常消耗性能的，常常会出现页面卡死的情况。所以尽量减少对DOM的操作成为了优化前端性能的必要手段，vdom就是将DOM的对比放在了JS层，通过对比不同之处来选择新渲染DOM节点，从而提高渲染效率。
- 使用虚拟DOM的损耗计算
  ```objc
  总消耗 = 虚拟DOM增删改 + （与Diff算法效率有关）真实DOM差异增删改 + （较少的节点）重排与重绘
  ```  
- 使用真实DOM的损耗计算
  ```objc
  总损耗 = 真实DOM完全增删改 + （可能较多的节点）重排与重绘
  ``` 
- Diff算法（Vue实现虚拟DOM对真实的DOM的更新）
  ```objc
  function updateChildren(vnode, newVnode) { // 创建对比函数
    var children = vnode.children || [];
    var newChildren = newVnode.children || [];

    children.forEach(function(childrenVnode, index) {
      var newChilrenVnode = newChildren[index]; // 首先拿到对应新的节点
      if(childrenVnode.tag = newChildrenVnode.tag) { // 判断节点是否相同
        updateChildren(childrenVnode, newChildVnode); // 如果相同就执行递归，深度对比节点
      }else {
        replaceNode(childrenVnode, newChildVnode); // 如果不同则将旧的节点替换成新的节点
      }
    })
  }
  function replaceNode(vnode, newVnode) { // 节点替换函数
    var elem = vnode.elem;
    var newEle = createElement(newVnode);
  }
  ```   
- 总之，一切为了减少频繁的大面积重绘引发的性能问题，不同框架不一定需要虚拟DOM，关键看框架是否会引发大面积的DOM操作。  




## 性能优化
- 减少HTTP请求(最重要最有效)
> 一个完整的请求都需要DNS寻址丶与服务器建立链接丶发送数据丶等待服务器响应丶接受数据这样一个漫长而复杂的过程。由于浏览器进行并发请求的请求数都是有上限的，因此请求数多了以后，浏览器需要进行分批请求，因此会增加用户的等待时间，给用户造成网站速度慢这种现象，即使可能用户看到的第一屏的资源都已经请求完了，但是浏览器的进度条一直在加载。
- 请求优化（减少HTTP请求数的途径只要有以下几个）
  - 从设计层面实现优化页面
    - 如果你的页面和百度的页面一眼简单，那么也就不需要什么优化操作了，因此保持页面简洁丶减少资源的使用是最直接的。
  - 合理设置HTTP缓存
    - 缓存的力量是强大的，恰当的缓存设置可以大大减少HTTP请求。
    - 怎么样才是合理的缓存，原则很简单：能缓存越多越好。
    - 例如：很少变化的图片资源就可以直接通过HTTP Header中的`Expires`设置一个很长的过期头，变化不频繁而又可能会变的资源可以使用`Last-Modified`来做请求校验，尽可能的让资源能够在缓存中待的更久。
  - 资源合并与压缩
    - 如果可以的话，尽可能的将外部脚本和样式进行合并，尽可能的合并为一个，另外CSS丶JS丶Image都可以使用相应的工具压缩，压缩往往能节省不少空间，或者使用webpack等前端工厂化工具进行代码的压缩和去重。       
  - CSSprites 雪碧图
    - 雪碧图又叫做精灵图，我们可以把网站中需要用到的一些icon全部放在一个图片资源中，如何通过改变位置来获取需要的图片，这样合并CSS图片，就可以大幅度减少HTTP请求数。
  - 网页内联图片 html inline image
    - 使用data: URL scheme的方式将图片嵌入到页面或CSS中，如果不考虑资源管理上的问题的话，不失为一个好办法，如果是嵌入页面的话换来的是增大了页面的体积，而无法利用好浏览器缓存，使用CSS中的图片的更加理想一些。 
    ```js
    <imgsrc="data:image/png;base64,iVAGRw0KGDCFGNSUhEUgACBBQAVGADCAIATYJ7ljmRGGAAGElEVQQIW2P4DwcMDAxAfBvMAhEQMYgcACEHG8ELxtbPACCCTElFTEVBQmGA" />
    ``` 
  - 懒加载(Lazy Load Image)
    - 这条策略实际上并不一定减少HTTP请求数，但是却能在某些条件下或者页面刚加载时减少HTTP请求。对于图片而言，在页面加载的时候只加载第一屏，当用户继续往后滚屏的时候才加载后续的图片，这样一来，假如用户只对第一屏的内容感兴趣的时，那剩余的图片请求都节省了。有的首页曾经的做法是在加载的时候把第一屏的之后的图片地址缓存在Textarea中，待用户往下滚屏的时候才"惰性"加载。     
    ```js
    <textarea rows="20" cols="120" id="textarea1" >
        <img src="https://dss1.bdstatic.com/70cFuXSh_Q1YnxGkpoWK1HF6hhy/it/u=3892521478,1695688217&fm=26&gp=0.jpg" alt="">  
    </TEXTAREA>
    ``` 
  - 瀑布流
    - 其实懒加载并不能减少HTTP请求数，它只是减少页面刚加载的时候的HTTP请求数，总数是不变的。对于图片而言，在页面加载的时候可能只加载第一屏的图片，随着用户的滚动才会加载后面的图片资源，这种瀑布流的加载方式就可以有效提高性能。
  - 减少不必要的HTTP跳转
    - 对于以目录形式访问的HTTP链接，很多人都会忽略最后是否带了"/"加入，服务器对此区别对待的话，那么你也要注意了，这其中可能隐藏了301跳转，增加了多余请求。
  - 避免重复的资源请求
    - 这种情况主要是由于在模块化开发的时候，我们不同的模块之间可能会有相同的部分，导致资源的重复请求。     
- 渲染优化
  - 将外部脚本放在底部
    - 前文有谈到，浏览器是可以并发请求的。这一特点使得其能够更快的加载资源，然而外部脚本在加载的时却会阻塞其他资源。例如在脚本加载完成之前，它后面的图片丶样式以及其他的脚本却处于阻塞状态，直至脚本加载完成后才会加载。如果将脚本放在靠前的位置，则会影响整个页面的加载速度从而影响用户体验，减少这一问题的方法有很多，而最简单可依赖的方法就是将脚本尽可能的往后挪，减少对并发下载的影响。
  - 将CSS放在Head中
    - 如果将CSS放在其他的地方比如body中，则浏览器有可能未下载和解析到CSS就开始渲染页面了，这就导致由无CSS状态跳转到CSS状态，用户体验比较糟糕。除此之外，有些浏览器会在CSS下载完成才开始渲染页面，如果CSS放在靠下的位置则会导致浏览器将渲染事件推迟。
  - 慎用with
    - with会改变作用域链，有可能导致我们的作用域链变长，导致查询性能下降。
  - 避免使用eval和Function
    - 每次eval或Function构造函数作用于字符串表示的源代码的时，脚本引擎都需要将源代码转换成可执行的代码，这是很耗费资源的操作。
    - 通常比简单的函数调用慢100倍以上        
  - 减少作用域链查找
    - 如果在循环中需要访问非作用域下的变量的时候，请遍历之前用局部变量缓存变量，并在遍历结束之后重写缓存变量。
  - 减少数据访问
    - js中对直接量和局部变量的访问时最快的，对对象属性以及数组的访问需要更大的开销，当出现下面的情况的时候，建议将数据放在局部变量。
      - 对任何对象属性的访问超过一次
      - 对任何数组成员的访问次数超过1次
      - 另外，要尽可能的减少对对象以及数组的深度查找。
  - 减少字符串的拼接
    - 字符串拼接尽可能少的使用"+"，这种方式的效率是十分低下的，因为每次运行都会开辟新的内存并生成新的字符串变量，然后将拼接的结果赋值给新变量。
    - 建议使用的是先转化为数组，如果通过数组的join方法来连接成字符串，不过由于数组也有一定的开销，因此需要权衡一下，当拼接的字符串比较少的时候，可以考虑用"+"的方式，比较多的时候就需要考虑用数组的json方法。
  - 减少DOM操作（DOM操作应该是脚本中最耗费性能的一类操作）
    - HTMLCollection(HTML收集器 返回的是一个数组的内容信息)
    - 因为是这个集合并不是一个静态的集合，它表示的仅仅是一个特定的查询，每次访问该集合时都会重新执行这个，查询从而更新查询结果，所谓的"访问集合"包括读取集合的length属性丶访问集合中的元素。    
  - 避免重排查绘
    - 页面有三个树：DOMTree丶CSSTree丶renderTree（实际上多余三个），renderTree上有两个规则，重排（reflow）和重绘（repaint）。
    - 重绘是元素自身的位置和宽高不变，只改变颜色之类的属性而不会导致后面的元素位置的变化的位置。
    - renderTree发生的动作，重排是元素自身的位置或宽高改变了从而导致的整个页面的大范围移动的时候，renderTree发生的动作，所以我们在DOM操作的时候，要尽量避免重排。  
- CSS优化
  - 慎用选择高消耗的样式，因为高消耗属性在绘制前需要浏览器进行大量计算。
    - box-shadow
    - border-radius
    - transform
  - 避免过分重排，因为发生重排的时候，浏览器需要重新计算布局的位置与大小
    - 常见的重排元素
      - width
      - height
      - padding
      - margin
      - display
      - border-width
      - position
      - float
      - ...
  - 正确使用display属性，因为display属性会影响页面渲染。
    - display: inline;后不应该使用width/height/margin/padding/float
    - display: inline-bloack;后不应该使用float
    - display: block;后不应该使用display: vertical-align;
  - 不滥用float，因为float是渲染时计算量比较大，尽量减少使用。
  - 动画性能优化
    - 动画实现的原理是利用了人眼的"视觉暂停"现象，在短时间内连续播放  数幅静止的画面，使肉眼因视觉残象产生错觉，而误以为画面在动。   
    - 动画的基本概念
      - 帧：在动画过程中，每一幅静止的画面即为一"帧"
      - 帧频：即每秒中播放的静止画面的停留时间，单位为fps（frame per second）
      - 帧时长：即每一幅静止画面停留时间，单位一般是ms（毫秒）
      - 跳帧（掉帧/丢帧）：在帧频固定的动画中，某一帧的时长远高于平均帧时长，导致其后续数帧被挤压丢失的现象。      
      - 一般浏览器的渲染的渲染帧频是60fps，所以在网页当中，帧频如果达到50-60fps将会相当流畅，让人感到舒畅。
    - 如果使用基于JavaScript的动画，尽量使用requestAnimationFrame避免使用setTimout/setInterval
    - 避免通过类似jQuery animation style改变帧频的样式，使用CSS声明，动画会得到更好的浏览器优化。
    - 使用translate取代absolute定位就会得到更好的fps，动画会更加流畅。
  - 多使用硬件能力，如通过3D变形加速开启GPU加速。
  - 提高CSS选择器性能
    - 因为CSS选择器对性能的影响源于浏览器匹配选择器和文档元素时所消耗的时间，所以优化选择器的原则是应尽量避免使用消耗更多的匹配时间的选择器，CSS选择器是从右往左进行规则匹配的。 
    - 减少搜索个数
      - 多使用类选择器少使用标签选择器
    - 减少层数
      - 使用BEM的命方式：块和元素之间用-连接，元素和修饰符之间用_连接
      - 使用面向对象的命名方式   
- 图片优化 





#### 如果你看完觉得这篇文章不错，帮我两件小事
- 关注公众号<text style="color: blue; font-weight: bold;">【前端应届生】</text>，持续为你推送精彩文章。
  
  <img style="width: 200px" src="https://imgkr.cn-bj.ufileos.com/51a2dad3-ae5b-4f66-9717-f0a36e4c68a7.png">
- 加我备注<text style="color: blue" style="color: blue;">【加群】</text>拉你进超级前端交流群，和各大厂的同学一起交流学习，共同进步。
  
  <img style="width: 200px" src="https://imgkr.cn-bj.ufileos.com/36e0b949-54df-4698-9ca2-d7e39ed0e2c0.jpg">

