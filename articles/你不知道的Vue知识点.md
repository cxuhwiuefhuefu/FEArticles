#### 结合场景来解答问题




# Vue 基础




## 1. Vue生命周期
   钩子函数 | 作用 | 应用场景 | 
   |:-|:-|:-|
   beforeCreate（创建前） | 获取不到props       或者data中的数据，因为这些数据初始化都在initState中。| 可以在这加一些loading效果，在created时进行移除 |
   created（创建后） | 可以访问到data中的数据，但是这个时候组件还没被挂载，所以看不到。| 异步请求数据丶页面初始化的工作 |     
   beforeMount（载入前） | 开始创建VDOM，相关的render函数首次被调用，把data里面的数据和模板生成html
   mounted（载入后） | 并将VDOM渲染成真实的DOM并且渲染数据，组件中如果有子组件的话，会递归挂载子组件，只有当子组件全部挂载完毕，才会执行根组件的挂载钩子 | 获取挂载dom元素节点 |
   beforeUpdate（更新前） | 数据更新前，发生在虚拟DOM重新渲染和打补丁之前，可以在此区间进一步更改状态，不会触发附加的渲染过程
   update（更新后） | 数据更新后 | 当数据更新需要做一些业务处理的时候使用 |
   beforeDestory（销毁前） | 适合移除事件丶定时器等等，否则会引起内存泄漏的问题 |
   destoryed（销毁后） | 进行一系列的销毁操作，如果有子组件的话，也会递归销毁子组件，当所有子组件都销毁完毕后才会执行根组件的destroy钩子函数。
   activated | 用keep-alive包裹的组件在切换时不会进行销毁，而是缓存到内存中执行这个钩子函数
   deactivated | 命中缓存渲染后会执行这个函数



## 2. Vue父子组件生命周期执行顺序
>Vue父子组件生命周期钩子的执行顺序遵循：从外到内丶然后从内到外。

组件加载渲染过程

`父组件beforeCreate` -> `父组件created` -> `父组件beforeMount` -> `子组件beforeCreated` -> `子组件created` -> `子组件beforeMount` -> `子组件mounted` -> `父组件mounted`

子组件更新过程

`父组件beforeUpdate` -> `子组件beforeUpdate` -> `子组件updated` -> `父组件updated`

父组件更新过程

`父组件beforeUpdate` -> `父组件updated`

销毁过程

`父组件beforeDestory` -> `子组件beforeDestory` -> `子destoryed` -> `父组件destoryed`




## 3. 组件通信
> 父组件与子组件传值
- 父组件传值给子组件：子组件通过props方法接受数据
- 子组件传值给父组件：$emit方法传递参数
```js
   <!DOCTYPE html>
    <html lang="en">
    <head>
        <meta charset="UTF-8">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
        <title>Document</title>
    </head>
    <body>

        <div id="app">
            <h3>{{title}}</h3>
            <users :users="users" v-on:titlechange="updatetitle"></users>
        </div>  
        
        <script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>

        <script>

            Vue.component('users' ,{
                template: `<ul>
                            <h1 @click=changeTitle>向父组件传值</h1>
                            <li v-for="user in users">{{user}}</li>
                        </ul>`,
                props: {
                    users: {
                        type: Array
                    }
                },
                methods: {
                    changeTitle() {
                        this.$emit("titlechange", '子组件向父组件传值');
                    }
                },
            })

            new Vue({
                el: "#app",
                data() {
                    return {
                        users: ['haha', 'haha1', 'haha2'],
                        title: ""
                    }
                },
                methods: {
                    updatetitle(e) {
                        this.title = e;
                    }
                },
            })

        </script>

    </body>
    </html>
```
> 非父子组件间的数据传递，兄弟组件传值
- eventBus，就是创建一个事件中心，相当于中转站，可以用它来传递事件和接受事件。项目小的时候  用这个比较合适
- vuex: 主要有两种数据会使用vuex进行管理 
  - 组件之间全局共享数据 
  - 通过后端异步请求的数据




## 4. extend能做什么
作用是拓展组件生成一个构造器，通常会与`$mount`一起使用。





## 5. mixin和mixins区别
- mixin
  - 用于全局混入，会影响到每个组件实例，通常插件都是这样做初始化的。
  - 虽然文档不建议我们在应用中直接使用mixin，但是如果不滥用的话也是很有帮助的，比如可以全局混入封装好的ajax或者一些工具函数等等。
- mixins
  - 应该是我们最常使用的拓展组件的方式了。如果多个组件中有相同的业务逻辑，就可以将这些逻辑剥离出来，通过mixins混入代码，比如上拉下拉加载数据这种逻辑等等。
  - 另外需要注意的是mixins混入的钩子函数会先于组件内的钩子函数执行，并且在遇到同名选项的时候也会有选择性的进行合并。





## 6. computed和watch区别
- computed
  - 是计算属性，依赖其他属性计算值，并且computed的值有缓存，只有当计算机值变化才会返回内容。
  - 一般需要依赖别的属性来动态获得值的时候可以使用computed
  - 应用场景：购物车结算商品的时候
- watch
  - 监听到值的变化就会执行回调，在回调中可以执行一些逻辑操作。
  - 对于监听到值的变化需要做一些复杂逻辑的情况下可以使用watch
  - 使用场景：当一条数据影响到多条数据的时候就需要用到watch，例如搜索数据
- 共同点：都支持对象的写法




## 6. keep-alive组件有什么用
- 如果你需要在组件切换的时候，保存一些组件的状态防止多次渲染，就可以使用keep-alive组件包裹需要保存的组件。
- 对于keep-alive组件来说，它拥有两个独立的生命周期钩子函数，分别为activated和deactivated。用keep-alive包裹的组件在切换时不会进行销毁，而是缓存到内存中并执行deactivated钩子函数，命中缓存渲染后会执行actived钩子函数。
- 使用场景：有时候需要将整个路由页面一切缓存，也就是将<router-view>进行缓存
  - 商品列表页点击商品跳转商品详情，返回后任显示原有信息
  - 订单列表跳转到的订单详情，然后返回
#### keep-alive的生命周期
- 初次进入 
 
  1.`created` > `mouted` > `actived`
  
  2.退出后触发`deactived`     
- 再次进入
  
  1.只会触发`activated` 
- 事件挂载的方法等，只执行一次的放在`mouted`，组件每次进去执行的方法放在`activated`  




## 7. v-show与v-if区别    
共同点：都能控制元素的显示和隐藏  
不同点：
- 实现方式
  - v-if是直接从Dom树上删除和重建元素节点
  - v-show只是修改css样式，也就是display的属性值，元素始终在DOM树上
- 编译过程
  - v-if切换有一个局部编译/卸载的过程，切换过程中合适地销毁和重建内部的事件监听和子组件
  - v-show只是简单的基于CSS切换
- 编译条件
  - v-if是惰性的，如果初始条件为假，则什么都不做，只有在条件第一次为真的时候才开始编译
  - v-show是在任何条件都被编译，然后被缓存，而且DOM元素始终被保留
- 性能消耗
  - v-if有更高的切换消耗，不适合做频繁的切换
  - v-show有更高的初始渲染消耗，适合做频繁的切换    
- 使用场景
  - 如果需要频繁的切换，则使用v-show比较好
  - 如果在运行时条件很少改变，则使用v-if较好 



## 8. 组件中data什么时候可以使用对象
- 组件复用时所有组件实例都会共享data，如果data是对象的话，就会造成一个组件修改data以后会影响到其他所有组件，所以需要将data写成函数，每次用到就调用一次函数获得新的数据。
- 当我们使用new Vue()的方式的时候，无论我们将data设置为对象还是函数都是可以的，因为new Vue()的方式是生成一个根组件，该组件不会复用，也就不会共享data的情况了。





## 9. v-for与v-if的优先级
* 当Vue处理指令时，v-for比v-if具更高的优先级
* 永远不要把v-if和v-for同时用在同一个元素上，如果实在需要，则在外套层template，在这一层进行v-if判断，然后再内部进行v-for循环，避免每次只有v-if只渲染很少一部分元素，也要遍历同级的所有元素
```js
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <script src="https://cdn.jsdelivr.net/npm/vue"></script>
    <script src="https://cdn.bootcss.com/jquery/3.4.1/jquery.js"></script>
</head>
<body>
<div id="app">
    <ul >
        <li v-for="item in user" v-if="item.isActive">{{item.name}}</li>
    </ul>
</div>
</body>
<script>
    new Vue({
        el: '#app',
        data: {
            user: [
                {name: 'hehe1', isActive: true},
                {name: 'hehe2', isActive: false},
                {name: 'hehe3', isActive: true},
                {name: 'hehe4', isActive: true},
            ]
        },
    });


    // 将会进行如下运算
    // this.users.map(function(user) {
    //    if(user.isActive) {
    //       return user.name; 
    //    }    
    // })
</script>
</html>
```




## 10. Vue子组件调用父组件方法总结
- `this.$emit('fn')`
  
  ```js
    <!DOCTYPE html>
    <html lang="en">
    <head>
        <meta charset="UTF-8">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
        <title>Document</title>
        <script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
    </head>
    <body>
        
        <div id="app">
            <div>
                <child @fatherMethod="fatherMethod"></child>
            </div>
        </div>

        <script>

            Vue.component('child', {
                template: `<div @click="activeBtn">点击我</div>`,
                methods: {
                    activeBtn() {
                        console.log(11);
                        this.$emit('fatherMethod');
                    }
                }          
            })
            var app = new Vue({
                el: "#app",
                methods: {
                    fatherMethod() {
                        console.log('父组件');
                    }
                }
            })

        </script>
        
    </body>
    </html> 
    ```
- `this.$parent.fn()`
  
  ```js
  <!DOCTYPE html>
    <html lang="en">
    <head>
        <meta charset="UTF-8">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
        <title>Document</title>
        <script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
    </head>
    <body>
        
        <div id="app">
            <div>
                <child></child>
            </div>
        </div>

        <script>

            Vue.component('child', {
                template: `<div @click="activeBtn">点击我</div>`,
                methods: {
                    activeBtn() {
                        this.$parent.fatherMethod();
                    }
                }          
            })
            var app = new Vue({
                el: "#app",
                methods: {
                    fatherMethod() {
                        console.log('父组件');
                    }
                }
            })

        </script>
        
    </body>
    </html>
  ``` 
- 父组件把方法通过props传入子组件中，在子组件里面调用这个方法
  
  ```js
  <!DOCTYPE html>
    <html lang="en">
    <head>
        <meta charset="UTF-8">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
        <title>Document</title>
        <script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
    </head>
    <body>
        
        <div id="app">
            <div>
                <child :acson="fatherMethod"></child>
            </div>
        </div>

        <script>

            Vue.component('child', {
                template: `<div @click="activeBtn">点击我</div>`,
                props: {
                    acson: {
                        type: Function,
                        default: null
                    }  
                },
                methods: {
                    activeBtn() {
                        this.acson();
                    }
                }          
            })
            var app = new Vue({
                el: "#app",
                methods: {
                    fatherMethod() {
                        console.log('父组件');
                    }
                }
            })

        </script>
        
    </body>
    </html>
  ```   









## 响应式原理
Vue内部使用了




## vue中的key
- key是为了Vue中的vNode标记的唯一标识，通过这个标识，我们的diff操作可以更准确丶更快速。 
- 准确：如果不加key，那么vue会选择复用节点（Vue的就地更新策略），导致之前节点的状态被保留下来，会产生一系列的bug
- 快速：key的唯一性可以被Map数据结构充分利用，相比于遍历查找的时间复杂度O(n)，Map的时间复杂度仅仅为O(1)。
```objc
function createKeyOldIdx(children, beginIndex, endIndex) {
    let i, key;
    const map = {};
    for(i = beginIndex, i <= endIndex, ++i) {
        key = children[i].key;
        if(isDef(key)) {
            map[key] = i；  
        }
    }
    return map;
}
```




## MVVM
### 什么是MVVM
- MVVM是Model-View-ViewModel的缩写，MVVM是一种设计思想。Model层代表数据模型，也可以在Model中定义数据修改和操作的业务逻辑；View代表UI组件，它负责将数据模型转化成UI展示出来，ViewModel是一个同步View和Model的对象。
- 在MVVM架构下，View和Model之间并没有直接的联系，而是通过ViewModel进行交互，Model和ViewModel之间的交互是双向的，因此View数据的变化会同步到Model中，而Model数据的变化也会立即反应到View上。
- ViewModel通过双向数据绑定把View层和Model层连接了起来，而View和Model之间的同步工作完全是自动的，无需人为干涉，因此开发者只需要关注业务逻辑，不需要手动操作DOM，不需要关注数据状态的同步问题，复杂的数据状态维护完全有MVVM来统一管理。


### MVVM和MVC区别
- MVC和MVVM其实区别不大。都是一种设计思想。主要是MVC中的Controller演变成MVVM中的viewModel。
- MVVM主要是解决了MVC中大量的DOM操作使页面性能降低，加载速度变慢，影响用户体验。和当Model频繁发生变化，开发者需要主动更新到View。   








## vue优点
- 轻量级框架：只关注视图层，是一个构建数据的视图集合，大小只有几十KB
- 简单易学：国人开发，中文文档，不存在语言障碍，易于理解和学习
- 双向数据绑定：保留了angular的特点，在数据操作方面更加简单
- 组件化：保留了react的优点，实现了html的封装和重用，在构建单页面应用方面有着独特的优势
- 视图丶数据丶结构分离：使数据的更改更为简单，不需要进行逻辑代码的修改，只需要操作数据就能完成相关操作
- 虚拟DOM: dom操作是非常耗费性能的，不再使用原生的dom操作结点，极大解放dom操作，更具体操作还是dom，只不过换了一种方式
- 运行速度赶快：相比较与react而言，同样是操作虚拟DOM，就性能而言，vue存在很大的优势




## vue常用修饰符
.prevent: 提交事件不再重载页面
.stop: 阻止单击冒泡事件
.capture: 事件监听 事件发生的时候会调用
.self: 当事情发生在该元素本身而不是子元素的时候会触发
.number: 将用户的输入转为成数值类型
.once: 只会触发一次



## v-on可以监听多个方法吗？
可以
```js
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>
<body>
    <input type="text"   @input = "onInput" @focus ="onFucus"  @blur ="onBlur">
    <script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
    <script>
       new Vue({
           el: "input",
           methods: {
                onInput: function () {
                     console.log(1);
                },
                onBlur: function () {
                    console.log(2);
                },
                onFucus: function() {
                    console.log(3);
                }
           },
       }) 
    </script>

</body>
</html>
```



## vue事件中如何使用event对象？
- 如果是函数的形式的，直接给个形参
- 如果是函数执行的形式，需要传入一个参数`$event`






## $nextTick的使用和作用
- 在下一次DOM更新循环之后执行延迟回调，在修改数据之后立即使用这个方法，获取更新后的DOM
- 为什么要使用`$nextTick`:这是由于Vue是异步执行dom更新的，一旦观察到数据变化，Vue就会开启一个队列，然后把在同一个事件循环（event loop）当中观察到数据变化的watcher推送进这个队列。如果这个watcher被触发多次，只会被推送到队列一次。这种缓冲行为可以有效的去掉重复数据造成的不必要的计算和dom操作。而下一个事件循环开始时，Vue会进行必要的dom更新，并清空队列（$nextTick方法就相当于在dom更新和清空队列后额外插入的执行步骤）
- 应用场景：需要在视图更新之后，基于新的视图进行操作，在created和mouted阶段，如果操作渲染后的视图，也要善于`$nextTick`
  - 点击按钮显示原本以v-show = false隐藏起来的输入框，并获得焦点 
    ```js
    <!DOCTYPE html>
    <html lang="en">
    <head>
        <meta charset="UTF-8">
        <title>Title</title>
        <script src="https://cdn.jsdelivr.net/npm/vue"></script>
        <script src="https://cdn.bootcss.com/jquery/3.4.1/jquery.js"></script>
    </head>
    <body>
    <div id="app">
        <input type="text" v-show = "showit" id="keywords">

        <button @click="click">click</button>
    </div>
    </body>
    <script>
        new Vue({
            el: '#app',
            data: {
                showit: false
            },
            mounted: function () {
              
            },
            methods: {
                showsou() {
                    this.showit = true;
                    // document.getElementById('keywords').focus();

                    this.$nextTick(function() {
                        document.getElementById('keywords').focus();
                    })
                },
                click() {
                    this.showsou();
                }
            }
        });
    </script>
    </html>
    ``` 
  - 点击获取元素的宽度
    ```js
    <!DOCTYPE html>
    <html lang="en">
    <head>
        <meta charset="UTF-8">
        <title>Title</title>
        <script src="https://cdn.jsdelivr.net/npm/vue"></script>
        <script src="https://cdn.bootcss.com/jquery/3.4.1/jquery.js"></script>
    </head>
    <body>
    <div id="app">
        <p ref="myWidth" v-if="showMe">{{message}}</p>
        <button @click="getMyWidth">click</button>
    </div>
    </body>
    <script>
        new Vue({
            el: '#app',
            data: {
                showMe: false,
                message: ""
            },
            methods: {
                getMyWidth() {
                    this.showMe = true;
                    // this.message = this.$refs.myWidth.offsetWidth;

                    this.$nextTick(() => {
                        this.message = this.$refs.myWidth.offsetWidth;
                    })
                }
            }
        });
    </script>
    </html>
    ``` 
  - 使用swiper插件通过ajax请求图片后的滑动问题






#### vue中如何编写可复用的组件？
- 高内聚，低耦合
- 单一职责
- 组件分类
  - 通用组件（可复用组件）    
  - 业务组件 （一次性组件）
- 可复用组件尽量减少对外部条件的依赖，所有与vuex相关的操作都不应该在可复用组件中出现 
- 组件应当避免对父组件的依赖，不要通过this.$parent来操作父组件的示例，父组件也不要通过this.$children来引用子组件的示例，而是通过子组件的接口与之交互

？？
https://www.jianshu.com/p/79a37137e45d




#### Vue插槽
- 使用场景：
  - 父组件向子组件传递DOM节点
  - “固定部分+动态部分”的组件的使用场景
- 单个插槽 | 默认插槽 | 匿名插槽
    
  - 特点：不用设置name属性
  - 源码实现
    ```js
    <!DOCTYPE html>
    <html lang="en">
    <head>
        <meta charset="UTF-8">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
        <title>Document</title>
        <script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
    </head>
    <body>

        <div id="app">
            <h1>我是父组件的标题</h1>
            <my-home>
                <p>这是一些初始内容</p>
                <p>这是更多的初始内容</p>
            </my-home>

        </div>

        <script>

            Vue.component('my-home', {
                data: function() {
                    return {
                        count: 0
                    }
                },
                template: `<div><h2>我是子组件的标题</h2><slot>只有在没有要分发的内容时才会显示。</slot></div>`
            })
            new Vue({
                el: "#app"
            })

        </script>
        
    </body>
    </html>
    ```
- 具名插槽
  - 特点：有name属性，具名属性可以在一个组件
  中出现多次，出现在不同的位置
  - 源码实现 
  ```js
  <!DOCTYPE html>
    <html lang="en">
    <head>
        <meta charset="UTF-8">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
        <title>Document</title>
        <script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
    </head>
    <body>

        <div id="app">
            <app-layout>
                <h1 slot="header">这里可能是一个页面标题</h1>
                <p>主要内容的一个段落</p>
                <p>另一个主要段落</p>
                <p slot="footer">这里有一些联系信息</p>
            </app-layout>
        </div>
        
        <script>

            // 具名插槽
            Vue.component('app-layout', {
                template: `<div class="container">
                            <header>
                                <slot name="header"></slot>
                            </header>

                            <main>
                                <slot></slot>
                            </main>

                            <footer>
                                <slot name="footer"></slot>
                            </footer>
                        </div>`
            })

            new Vue({
                el: "#app"
            })

        </script>

    </body>
    </html>
  ```
- 作用域插槽
  - 特点：在slot上绑定数据，可从子组件获取数据的可复用的插槽
  - 应用场景：封装一个列表组件（或者类似列表的组件）因为在真正使用场景下，子组件的数据都是来自父组件的，作为组件内部应该保持纯净，就像element-ui里的table组件，肯定不会定义一些数据在组件内部，然后传递给你，table组件的数据都是来自调用者，然后table会把每一行的row，在开发者需要时，传递出去
  - 源码实现：
  ```js
  <!DOCTYPE html>
    <html lang="en">
    <head>
        <meta charset="UTF-8">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
        <title>Document</title>
        <script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
    </head>
    <body>

        <div id="app">
            <child>
                <template slot-scope="user">
                    <div class="tmp">
                        <span v-for="item in user.data">{{ item }}</span>
                    </div>
                </template>
            </child>

        </div>

        <script>

            Vue.component('child', {
                data: function() {
                    return {
                        data: ['haha', 'hihi', 'hehe']
                    }
                },
                template: `<div class="child">
                            <slot :data="data"></slot>
                        </div>`
            })


            new Vue({
                el: "#app"
            })

        </script>

        
    </body>
    </html>
  ```
  ```js
  <!DOCTYPE html>
    <html lang="en">
    <head>
        <meta charset="UTF-8">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
        <title>Document</title>
        <script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
    </head>
    <body>

        <div id="app">
            <todo-list :todos="todos">
                <template slot-scope="slotProps">
                    <span v-if="slotProps.todo.isComplete">✓</span>
                    <span>{{ slotProps.todo.text }}</span>
                </template>
            </todo-list>
        </div>

        <script>

            Vue.component('todoList', {
                props: {
                    todos: {
                        type: Array 
                    }
                },
                template: `<ul>
                            <li v-for="todo in todos" :key="todo.id">
                                <slot :todo="todo"></slot>
                            </li>
                        </ul>`
            })

            new Vue({
                el: "#app",
                data() {
                    return {
                        todos: [
                            {
                                id: 0,
                                text: 'haha0',
                                isComplete: false
                            },
                            {
                                text: 'haha1',
                                id: 1,
                                isComplete: true
                            },
                            {
                                text: 'haha2',
                                id: 2,
                                isComplete: false
                            },
                            {
                                text: 'haha3',
                                id: 3,
                                isComplete: false
                            }
                        ]
                    }
                }
            })

        </script>

        
    </body>
    </html>
  ```
- 动态插槽名
  ```js
  <!DOCTYPE html>
    <html lang="en">
    <head>
        <meta charset="UTF-8">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
        <title>Document</title>
        <script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
    </head>
    <body>

        <div id="app">
            <base-layout>
                <template v-slot:[dyname]>
                    This is Me.
                </template>
            </base-layout>
        </div>

        <script>

            Vue.component('base-layout', {
                template: `<div>
                            <header style="font-size: 20rpx">
                                <slot name="header"></slot>
                            </header>
                        </div>`
            })

            new Vue({
                el: "#app",
                data: {
                    dyname: 'header'
                }
            })

        </script>

        
    </body>
    </html>
  ``` 







-----
# vue路由




## 说说你对SPA单页面的理解，它的优缺点是什么？
> SPA（single-page application）单页面仅在WEB页面初始化时加载相应的HTML丶JavaScript和CSS。一旦页面加载完成，单页面不会因为用户的操作而进行页面的重新加载或跳转；取而代之的是利用路由机制实现HTML内容的变换，UI和用户的交互，避免页面的重新加载。
#### 优点：
- 用户体验好丶快，内容的改变不需要加载整个页面，避免了不必要的跳转和重新渲染
- 基于上面一点，SPA相对于服务器的压力较小
- 前后端分离，架构清晰，前后端进行交互逻辑，后端负责数据处理
#### 缺点：
- 初始加载耗时多：为实现单页面WEB应用功能及显示效果。需要在加载页面的时候将JavaScript丶CSS统一加，部分页面按需加载
- 前进后退路由管理：由于单页面应用在一个页面中显示所有的内容，所以不能使用浏览器的前进后退功能，所有的页面切换需要自己建立堆栈管理
- SEO难度较大：由于所有的内容都在一个页面中动态替换显示，所以在SEO上其有天然的弱势





#### 1. mvvm框架是什么？
vue是实现了双向数据绑定的mvvm框架，当视图层改变更新模型层，当模型层改变更新视图层。在vue中，使用了双向数据技术，就是视图层的变化实时让让模型层的数据发生变化，而模型层数据的变化也能实时更新到视图层


## vue-router是什么？它有哪些组件？
vue用来写路由的一个插件。router-link丶router-view


## active-class是哪些组件的属性？
vue-router模块的router-link组件   children数组定义子路由


## 怎么定义vue-router的动态路由？怎么获取传过来的值？
在router目录下的index.js文件 对path属性加上/:id

使用router对象的params.id


## vue-router有哪些导航钩子？
三种

第一种：是全局导航钩子：router.beforeEach(to, from, next) 作用：跳转之前进行判断拦截

第二种：组件内的钩子

第三种：单独路由独享组件



## `$route`和`$router`的区别
`$router`是VueRouter的实例，在script标签中想要导航到不同的URL，使用$router.push方法。返回上一个历史history用$router.to(-1)

$route为当前router跳转对象 里面可以获取当前路由的name丶path丶query丶params等



## vue-router的两种模式
- hash模式：即地址栏URL的#符号
- history模式：window.history对象打印出来可以看到里面提供的方法和记录长度。利用了HTML5 History Interface中新增的pushState()和replaceState()方法。（需要特定浏览器支持）



## vue-router实现路由懒加载（动态加载路由）
三种方式

- 第一种：vue异步组件技术 === 异步加载，vue-router配置路由，使用vue的异步组件技术，可以实现按需加载，但是这种情况下一个组件生成一个JS文件

- 路由懒加载（使用import）

- webpack提供的require(), vue-router配置路由，使用webpack的require.ensure技术，也可以按需加载。这种情况下，多个路由指定相同的chunkName，会合并打包成一个JS文件





## 路由守卫
- 分类
  - 全局守卫：是指路由实例上直接操作的钩子函数，特点是所有路由配置的组件都会触发，直白点就是触发路由就会触发这些钩子函数
     - router.beforeEach(to, from, next) 
     - router.beforeResolve(to, from, next)
     - router.afterEach(to, from)
     
  - 路由独享守卫：是指在单个路由配置的时候也可以设置的钩子函数
    - beforeEnter(to, from, next) 
  - 组件内守卫: 是指在组件内执行的钩子函数，类似于组件内的生命周期，相当于为配置路由的组件添加的生命周期钩子函数
    - beforeRouterEnter(to, from, next)
    - beforeRouterUpfate(to, from, next)
    - beforeRouterLeave(to, from, next)
- 应用场景
  - 只有当用户已经登录并拥有某些权限时才能进入某些路由
  - 一个由多个表单组成的向导，例如注册流程，用户只有在当前路由的组件中填写了满足的信息才可以导航到下一个路由  
  - 当用户未执行保存操作而试图离开导航时提醒用户
  - 应用场景：在用户离开当前界面时询问
  - 应用场景：验证用户登录过期






## vue路由传参的形式
- 页面刷新数据不会丢失
  ```js
  <div class="exmaine" @click="insurance(2)">查看详情</div>

  methods: {
      insurance(id): {
          this.$router.push({
              path: "/particulars/${id}"
          })
      }
  }
  
  // 对应的路由配置
  {
      path: "/particulars/:id",
      name: "particulars",
      component: particulars
  }

  // 另外页面获取参数如下
  this.$route.params.id
  ```
- 页面刷新数据会丢失（可以在路由的path里加参数，加上参数以后刷新页面数据就不会丢了）
  ```js
  // 通过路由属性中的name来确定匹配的路由 通过params来传递参数
  methods: {
      insurance(id) {
          this.$router.push({
              name: "particulars",
              params: {
                  id: id
              }
          })
      }
  }

  // 对应路由配置：注意这里不能使用:/id来传递参数了 因为组件中已经使用params来携带参数了
  {
      path: "/particulars",
      name: "particulars",
      component: particulars
  }

  // 子组件中主要获取参数
  this.$route.params.id
  ```   
- 使用path来匹配路由，然后通过query来传递参数
  ```js
  methods: {
      insurance(id) {
          this.$router.push({
              path: "/particulars",
              qury: {
                  id: id
              }
          })
      }
  }

  {
      path: "/particulars",
      name: "particulars",
      components: "particulars"
  }

  this.$route.query.id
  ```   
- 应用场景
  - 点击父组件   






----
# VUEX面试题





#### Vuex是什么？怎么使用？ 那种功能场景下使用它 ？
- vuex框架中状态管理。
- 在main.js引入store.新建一个一个目录store.js
- 场景：单页面应用中 组件之间的状态 音乐播放 登录状态 加入购物车




## Vuex有哪种属性
State丶Getter丶Mutation丶Action丶Module

state ==> 基本数据（数据存放的地方）

getters => 从基本数据派生出来的数据

mutation => 提交更新数据的方法 同步

action => 像一个装饰器 包裹着mutation 使之可以异步

modules => 模块化vuex





## vue.js中的ajax请求代码应该写在组件的methods中还是vuex的action中
如果请求来的数据不是被其他组件公用，仅仅在请求的组件内使用，就不要放入vuex的state里面

如果被其他地方复用，这个很大概率上是需要的，如果是需要的请将放入action里面，方便使用








https://zhuanlan.zhihu.com/p/41237859
https://www.jianshu.com/p/2d3f584be910
https://blog.csdn.net/lqlqlq007/article/details/93462955
https://segmentfault.com/a/1190000012861862?utm_source=tag-newest
https://www.cnblogs.com/jin-zhe/p/9523782.html
https://www.cnblogs.com/mengzekun/p/12575265.html
https://segmentfault.com/a/1190000015884505

