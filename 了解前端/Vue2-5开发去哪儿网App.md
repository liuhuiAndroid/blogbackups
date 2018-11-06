###第1章 课程介绍
[Windows环境搭建Vue开发环境](https://www.cnblogs.com/zhaomeizi/p/8483597.html)
学习前提：js、ES6、webpack、npm
- JavaScript由三部分组成：ECMAScript、DOM、BOM
- ECMAScript6是ECMAScript第六个版本，提供大量新特性
- webpack 是一个模块打包器
- npm 是 JavaScript 世界的包管理工具,并且是 Node.js 平台的默认包管理工具。通过 npm 可以安装、共享、分发代码,管理项目依赖关系。

###第2章 Vue 起步 
######2-1 课程学习方法
学习方法：具体知识点看官方文档
[Vue 官网](https://cn.vuejs.org/)

######2-2 hello world
根据官方文档完成：Vue.js实现hello world
```
<body>
  <div id="app">{{content}}</div>
  <script>
    var app = new Vue({
      el: '#app',
      data: {
        content: 'hello world'
      }
    })
    setTimeout(function() {
      app.$data.content = 'bye world'
    })
  </script>
<body>
```

######2-3 开发TodoList（v-model、v-for、v-on）
[ToDoList 官网](http://www.todolist.cn/)
v-model：数据的双向绑定
el：限制vue实例处理的dom的范围
data：定义一些数据 
```
<body>
  <div id="app">
    <input type="text" v-model="inputValue" />
    <button v-on:click="handleBtnClick">提交</button>
    <ul>
      <li v-for="item in list">{{item}}</li>
    </ul>
  </div>

  <script>
    var app = new Vue({
      el: '#app',
      data: {
        list: [],
        inputValue: ''
      }
    })
    methods: {
      handleBtnClick: function() {
        this.list.push(this.inputValue)
        this.inputValue = ''
      }
    }
  </script>
<body>
```

######2-4 MVVM模式
![image.png](https://upload-images.jianshu.io/upload_images/1956963-131f1dd49981a22b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

######2-5 前端组件化
######2-6 使用组件化思想改造TodoList
v-bind：向子组件传递绑定值
- 全局组件的使用：
```
<body>
  <div id="app">
    <input type="text" v-model="inputValue" />
    <button v-on:click="handleBtnClick">提交</button>
    <ul>
      <todo-item v-bind:content="item" 
                 v-for="item in list">
      </todo-item>
    </ul>
  </div>

  <script>
    Vue.component("TodoItem",{
      props: ['content'],
      template: "<li>{{content}}</li>"
    })

    var app = new Vue({
      el: '#app',
      data: {
        list: [],
        inputValue: ''
      }
    })
    methods: {
      handleBtnClick: function() {
        this.list.push(this.inputValue)
        this.inputValue = ''
      }
    }
  </script>
<body>
```
- 局部组件的使用：
需要注册到根vue的实例里
```
<body>
  <div id="app">
    <input type="text" v-model="inputValue" />
    <button v-on:click="handleBtnClick">提交</button>
    <ul>
      <todo-item v-bind:content="item" 
                 v-for="item in list">
      </todo-item>
    </ul>
  </div>

  <script>
    var TodoItem = {
      props: ['content'],
      template: "<li>{{content}}</li>"
    }

    var app = new Vue({
      el: '#app',
      components: {
        TodoItem: TodoItem
      },
      data: {
        list: [],
        inputValue: ''
      }
    })
    methods: {
      handleBtnClick: function() {
        this.list.push(this.inputValue)
        this.inputValue = ''
      }
    }
  </script>
<body>
```

######2-7 简单的组件间传值
子组件向父组件传值
v-on:click 简写 @click
v-on是监听事件
v-bind:content 简写 :content
主要就是：this.$emit("delete",this.index); 子组件向父组件发送事件。
```
<body>
  <div id="app">
    <input type="text" v-model="inputValue" />
    <button v-on:click="handleBtnClick">提交</button>
    <ul>
      <todo-item :content="item" 
                 :index="index" 
                 v-for="(item, index) in list"
                 @delete="handleItemDelete">
      </todo-item>
    </ul>
  </div>

  <script>
    var TodoItem = {
      props: ['content', 'index'],
      template: "<li @click='handleItemClick'>{{content}}</li>",
      methods: {
        handleItemClick: function() {
          this.$emit("delete",this.index);
        }
      }
    }

    var app = new Vue({
      el: '#app',
      components: {
        TodoItem: TodoItem
      },
      data: {
        list: [],
        inputValue: ''
      }
    })
    methods: {
      handleBtnClick: function() {
        this.list.push(this.inputValue)
        this.inputValue = ''
      },
      handleItemDelete: function() {
        this.list.splice(index,1)
      }
    }
  </script>
<body>
```

######2-8 章节小结
阅读官方文档基础部分介绍下的内容

###第3章 Vue 基础精讲
######3-1 Vue实例
- 每一个组件都是一个Vue的实例，Vue的实例上有很多属性和方法。
```
控制台输入：
vm 
vm.$data  
vm.$el 
vm.$destroy()  销毁vue实例
```
```
<body>
  <div id="root">
    <div @click="handleClick">
      {{message}}
    </div>
    <item></item>
  </div>

  <script>
    Vue.component('item',{
      template: '<div>hello item</div>'
    })

    var vm = new Vue({
      el: '#root',
      data: {
        message: 'hello world'
      },
      methods: {
        handleClick: function() {
          alert("hello")
        }
      }
    })
  </script>
<body>
```

######3-2 Vue实例生命周期
看官网Vue实例部分内容
其中生命周期图示，非常好用
```
<body>
  <div id="app"></div>

  <script>
    var vm = new Vue({
      el: '#app',
      template: "<div>hello world</div>"
      data: {
        test: 'hello world'
      },
      beforeCreate: function() {
        console.log("beforeCreate");
      },
      created: function() {
        console.log("created");
      },
      beforeMount: function() {
        console.log(this.$el);
        console.log("beforeMount");
      },
      mounted: function() {
        console.log(this.$el);
        console.log("mounted");
      },
      beforeDestroy: function() {
        console.log("beforeDestroy");
      },
      destroyed: function() {
        console.log("destroyed");
      },
      beforeUpdate: function() {
        console.log("beforeUpdate");
      },
      updated: function() {
        console.log("updated");
      }
    })
  </script>
<body>
```

######3-3 Vue的模版语法
差值表达式：{{}}
```
<body>
  <div id="app">
    <div>{{name + 'Lee'}}</div>
    <div v-text="name + ' Lee'"></div>
    <div v-html="name + ' Lee'"></div>
  </div>

  <script>
    var vm = new Vue({
      el: '#app',
      data: {
        name: 'hello world'
      }
    })
  </script>
<body>
```

######3-4 计算属性,方法与侦听器
计算属性：优先推荐，原因：既简洁又性能高
```
<body>
  <div id="app">
    {{fullName}}
    {{age}}
  </div>

  <script>
    var vm = new Vue({
      el: '#app',
      data: {
        firstName: 'hello',
        lastName: 'world',
        age: 28
      }, 
      // 计算属性
      computed: {
        fullName: function() {
          console.log("计算了一次");
          return this.firstName + " " + this.lastName;
        }
      }
    })
  </script>
<body>
```
方法
```
<body>
  <div id="app">
    {{fullName()}}
    {{age}}
  </div>

  <script>
    var vm = new Vue({
      el: '#app',
      data: {
        firstName: 'hello',
        lastName: 'world',
        age: 28
      }, 
      methods: {
        fullName: function() {
          console.log("计算了一次");
          return this.firstName + " " + this.lastName;
        }
      }
    })
  </script>
<body>
```
侦听器
```
<body>
  <div id="app">
    {{fullName}}
    {{age}}
  </div>

  <script>
    var vm = new Vue({
      el: '#app',
      data: {
        firstName: 'hello',
        lastName: 'world',
        fullName: 'hello world',
        age: 28
      },
      watch: {
        firstName: function() {
          console.log("计算了一次");
          this.fullName = this.firstName + " " + this.lastName;
        },
        lastName: function() {
          console.log("计算了一次");
          this.fullName = this.firstName + " " + this.lastName;
        }
      }
    })
  </script>
<body>
```

######3-5 计算属性的 getter 和 setter
```
<body>
  <div id="app">
    {{fullName}}
  </div>

  <script>
    var vm = new Vue({
      el: '#app',
      data: {
        firstName: 'hello',
        lastName: 'world',
      }, 
      // 计算属性
      computed: {
        fullName: {
          get: function() {
            console.log("计算了一次");
            return this.firstName + " " + this.lastName;
          }
          set: function(value) {
            var arr = value.split(" ");
            this.firstName = arr[0];
            this.lastName = arr[1];
          }
        }
      }
    })
  </script>
<body>
```

######3-6 Vue中的样式绑定
- class的对象绑定
```
<head>
  <style>
    .activated {
      color: red;
    }
  </style>
</head>
<body>
  <div id="app">
    <div @click="handleDivClick"
            :class="{activated: isActivated}"
      >
      Hello World
    </div>
  </div>

  <script>
    var vm = new Vue({
      el: '#app',
      data: {
        isActivated: false
      },
      methods: {
        handleDivClick: function() {
          this.isActivated = !this.isActivated;
        }
      }
    })
  </script>
<body>
```
- class的数组方式
```
<head>
  <style>
    .activated {
      color: red;
    }
  </style>
</head>
<body>
  <div id="app">
    <div @click="handleDivClick"
            :class="[activated]"
      >
      Hello World
    </div>
  </div>

  <script>
    var vm = new Vue({
      el: '#app',
      data: {
        activated: ""
      },
      methods: {
        handleDivClick: function() {
          this.activated= this.activated === "activated" ?
            "" : "activated";
        }
      }
    })
  </script>
<body>
```
- style的方式
```
<body>
  <div id="app">
    <div :style="styleObj" @click="handleDivClick">
      Hello World
    </div>
  </div>

  <script>
    var vm = new Vue({
      el: '#app',
      data: {
        styleObj: {
          color: black
        }
      },
      methods: {
        handleDivClick: function() {
          this.styleObj.color = this.styleObj.color === "black" ?
            "red" : "black";
        }
      }
    })
  </script>
<body>
```
- style的数组方式
```
  <div id="app">
    <div :style="[styleObj, {fontSize: '20px'}]" @click="handleDivClick">
      Hello World
    </div>
  </div>
// 别的不改变，和上面一样
```

######3-7 Vue中的条件渲染
v-show:false已经渲染，display:none。不会频繁删除添加。
v-if:false没有渲染
```
<body>
  <div id="app">
    <div v-if="show">{{message}}</div>
    <div v-else>Bye World</div>
    <div v-show="show">{{message}}</div>

    <div v-if="show2==='a'">This is A</div>
    <div v-else-if="show2==='b'">This is B</div>
    <div v-else>This is others</div>
  </div>

  <script>
    var vm = new Vue({
      el: '#app',
      data: {
        show: false,
        show2: "a",
        message: "Hello World"
      }
    })
  </script>
<body>
```
注意下面show值变化，vue复用input导致数据错乱，需要使用key来避免此情况
```
  <div id="app">
    <div v-if="show">
      用户名：<input key="username" />
    </div>
    <div v-else>
      邮箱名：<input key="password" />
    </div>
  </div>
```

######3-8 Vue中的列表渲染
item of index 比 item in index 更推荐使用
key值的使用：可以提高性能，最好不要用index，用服务端返回数据中的id
```
<body>
  <div id="app">
    <div v-for="(item,index) of list"
            :key="item.id">
      {{item.text}} --- {{index}}
    </div>
  </div>

  <script>
    var vm = new Vue({
      el: '#app',
      data: {
        list: [{
          id: “001”，
          text: "hello"
        },{
          id: “002”，
          text: "world"
        }] 
      }
    })
  </script>
<body>

vm.list.push({id:"004",text:"222"})
不能通过下标方式改变数组
牢记：只能通过vue提供的数组变异方法操作数组：pop push shift unshift splice sort reverse
vm.list.splice(1,1,{id:"333",text:"mh"})

还有一种方法是改变引用
vm.list =  [{
  id: "001",
  text: "hello1"
},{
  id: "002",
  text: "world2"
}] 
```
template模板占位符，可以帮助我们去包裹一些元素，但是在循环过程中并不会被真正的渲染到页面上。
```
<body>
  <div id="app">
    <template v-for="(item,index) of list"
            :key=“item.id”>
      <div>
        {{item.text}} --- {{index}}
      </div>
      <span>
        {{item.text}} --- {{index}}
      </span>
    </template>

  </div>

  <script>
    var vm = new Vue({
      el: '#app',
      data: {
        list: [{
          id: “001”，
          text: "hello"
        },{
          id: “002”，
          text: "world"
        }] 
      }
    })
  </script>
<body>
```
对象的循环
```
<body>
  <div id="app">
      <div v-for="(item,key,index) of userInfo">
        {{item}} --- {{key}} --- {{index}}
      </div>
  </div>

  <script>
    var vm = new Vue({
      el: '#app',
      data: {
        userInfo: {
          name: "D",
          age: 24,
          gender: "male",
          salary: "secret",
        }
      }
    })
  </script>
<body>

vm.userInfo.name = "lh"
vm.userInfo.address = "shanghai" # 页面不改变，无效
可以直接修改引用
vm.userInfo = {
  name: "lh",
  age: 18,
  gender: "male",
  salary: "secret",
}
```

######3-9 Vue中的set方法
```
对象上的使用方法：
Vue.set(vm.userInfo,"address","shang")
vm.userInfo
vm.$set(vm.userInfo,"address","shang")

数组上的使用方法：
Vue.set(vm.list,1,5)
```

###第4章 深入理解 Vue 组件
######4-1 使用组件的细节点
使用is解决h5标签的小bug
```
<body>
  <div id="app">
      <table>
        <tbody>
          <tr is="row"></tr>
          <tr is="row"></tr>
          <tr is="row"></tr>
        </tbody>
      </table>
  </div>

  <script>
    Vue.component('row',{
      template: '<tr><td>this is a row</td></tr>'
    })
    
    var vm = new Vue({
      el: '#app'
    })
  </script>
<body>
```
子组件定义data，data必须是一个函数，函数需要返回对象，包含你所需要的数据
```
<body>
  <div id="app">
      <table>
        <tbody>
          <tr is="row"></tr>
          <tr is="row"></tr>
          <tr is="row"></tr>
        </tbody>
      </table>
  </div>

  <script>
    Vue.component('row',{
      data: function() {
        return {
          content: 'this is a row'
        }
      }
      template: '<tr><td>{{content}}</td></tr>'
    })
    
    var vm = new Vue({
      el: '#app'
    })
  </script>
<body>
```
ref 的使用
```
<body>
  <div id="app">
      <counter ref="one" @change="handleChange"></counter>
      <counter ref="two" @change="handleChange"></counter>
      <div>{{total}}</div>
  </div>
    
  <script>
    Vue.component('counter',{
      template: '<div @click="handleClick">{{number}}</div>',
      data: function() {
        return {
          number: 0
        }
      },
      methods: {
        handleClick: function() {
          this.number ++
          this.$emit('change')
        }
      }
    )

    var vm = new Vue({
      el: '#app',
      data: {
        total: 0
      }
      methods: {
        handleChange: function() {
          this.total = this.$refs.one.number + this.$refs.two.number
        }
      }
    })
  </script>
<body>
```

######4-2 父子组件间的数据传递
父组件通过属性的方式向子组件传递数据
单向数据流：子组件最好不要修改父组件传递过来的参数。
```
<body>
    <div id="app">
        <counter :count="1" @inc="handleInc"></counter>
        <counter :count="2" @inc="handleInc"></counter>
        <div>{{total}}</div>
    </div>
    
    <script>
        var counter = {
            props: ['count'],
            data: function(){
                return {
                    number: this.count
                }
            },
            template: '<div @click="handleClick">{{number}}</div>',
            methods: {
                handleClick: function(){
                    this.number += 2;
                    this.$emit('inc',2);
                }
            }
        }
        var vm = new Vue({
            el: "#app",
            data:{
                total:3
            },
            components: {
                counter: counter
            },
            methods: {
                handleInc: function(step){
                    this.total += step;
                }
            }
        })
    </script>
</body>
```

######4-3 组件参数校验与非 props 特性 - 看到这里
父组件向子组件传递的必须是字符串需求
props 特性：属性传递不会再DOM标签上显示
非 props 特性：子组件没有接收父组件传递的内容，子组件无法使用。显示在DOM属性中
非 props 特性使用不是非常多。
```
<body>
    <div id="app">
        <child content="1"></child>
    </div>
    
    <script>
        Vue.component('child',{
            props: {
                content: {
                    type: String,
                    required: true,
                    default: 'default value',
                    validator: function(value){
                        return (value.length > 2)
                    }
                }
            },
            template: '<div>{{content}}</div>'
        })
        var vm = new Vue({
            el: "#app"
        })
    </script>
</body>
```

######4-4 给组件绑定原生事件
只需要在事件绑定的后面加native修饰符
```
<body>
    <div id="app">
        <child @click.native="handleClick"></child>
    </div>
    
    <script>
        Vue.component('child',{
            template:'<div>Child</div>'
            // template:'<div @click="handleChildClick">Child</div>',
            // methods:{
            //     handleChildClick: function(){
            //         alert("handleChildClick");
            //         this.$emit("click");
            //     }
            // }
        })

        var vm = new Vue({
            el: "#app",
            methods:{
                handleClick:function(){
                    alert("handleClick");
                }
            }
        })
    </script>
</body>
```

######4-5 非父子组件间的传值
```
<body>
    <div id="app">
        <child content="hello"></child>
        <child content="world"></child>
    </div>

    <script>
        Vue.prototype.bus = new Vue()

        Vue.component('child', {
            props: ['content'],
            data: function(){
                return {
                    selfContent: this.content
                }
            },
            template: '<div @click="handleClick">{{selfContent}}</div>',
            methods:{
                handleClick: function(){
                    this.bus.$emit('change',this.selfContent)
                }
            },
            mounted: function(){
                var this_ = this;
                this.bus.$on('change',function(msg){
                    this_.selfContent = msg;
                })
            }
        })

        var vm = new Vue({
            el: "#app"
        })
    </script>
</body>
```

######4-6 在Vue中使用插槽
插槽和具名插槽
```
<body>
    <div id="app">
        <child content="<p>123</p>">
            <h1>Dell</h1>
        </child>

        <div>------------------</div>

        <body-content>
            <div class='header' slot='header'>
                header
            </div>
           <div class='footer' slot='footer'>
               footer
           </div>
        </body-content>
    </div> 
    
    <script>
        Vue.component('child',{
            props:['content'],
            template: `<div>
                            <p>hello</p>
                            <div v-html="this.content"></div>
                            <slot>default content</slot>
                       </div>`
        })

        // vue中的具名插槽
        Vue.component('body-content',{
            // 多行字符串语法
            template: `<div>
                            <slot name='header'>
                                <div>default header</div>
                            </slot>
                            <div class='content'>content</div>
                            <slot name='footer'></slot>
                       </div>`
        })

        var vm = new Vue({
            el: "#app"
        })
    </script>
</body>
```

######4-7 作用域插槽
固定写法
```
<body>
    <div id="app">
        <child>
            <template slot-scope="props">
                <li>{{props.item}}</li>
            </template>
        </child>
    </div>
    
    <script>
        Vue.component('child',{
            data: function(){
                return {
                    list: [1,2,3,4]
                }
            },
            template:`<div>
                        <ul>
                            <slot v-for="item of list" :item=item>
                            </slot>
                        </ul>
                      </div>`
        })
        var vm = new Vue({
            el: "#app"
        })
    </script>
</body>
```

######4-8 动态组件与 v-once 指令
v-once可以把组件放到内存中，提高静态内容的展示效率
```
<body>
    <div id="app">
        <!-- 动态组件 -->
        <component :is='type'></component>

        <!-- v-once提高静态内容的展示效率 -->
        <!-- <child-one v-if="type === 'child-one'"></child-one>
        <child-two v-if="type === 'child-two'"></child-two> -->

        <button @click="handleBunClick">Change</button>
    </div>
    
    <script>
        Vue.component('child-one',{
            template:'<div v-once>child-one</div>'
        }) 

         Vue.component('child-two',{
            template:'<div v-once>child-two</div>'
        })

        var vm = new Vue({
            el: "#app",
            data:{
                type :'child-one'
            },
            methods: {
                handleBunClick: function(){
                    this.type = (this.type === 'child-one' ? 'child-two' : 'child-one');
                }
            }
        })
    </script>
</body>
```

###第5章 Vue 中的动画特效
######5-1 Vue动画 - Vue中CSS动画原理
![image.png](https://upload-images.jianshu.io/upload_images/1956963-9cd7560f0c75999c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```
<head>
    <script src="./vue.js"></script>
    <style>
        /* 
            v-enter：定义进入过渡的开始状态。
            v-leave-to: 2.1.8版及以上 定义离开过渡的结束状态。
         */
        .v-enter,
        .v-leave-to {
            /* 设置一个div元素的透明度级别 0:完全透明 1:完全不透明 */
            opacity: 0;
        }

        /* 
            v-enter-active：定义进入过渡生效时的状态。
            v-leave-active：定义离开过渡生效时的状态。
         */
        .v-enter-active,
        .v-leave-active {
           transition: opacity 2s;
        }
    </style>
</head>

<body>
    <div id="app">
        <transition>
            <div v-if="show">hello world</div>
        </transition>
        <button @click="handleClick">切换</button>
    </div>

    <script>
        var vm = new Vue({
            el: "#app",
            data: {
                show: true
            },
            methods: {
                handleClick: function () {
                    this.show = !this.show
                }
            }
        })
    </script>
</body>
```

######5-2 在Vue中使用 Animate.css 库
Animate.css是一个有趣的,跨浏览器的css3动画库。
**[animate.css github](https://github.com/daneden/animate.css)**
**[animate.css 官网](https://daneden.github.io/animate.css/)**
```
<head>
    <script src="./vue.js"></script>
    <link rel="stylesheet" type="text/css" href="./animate.css">
    <style>
        /* 使用keyframes这样的css3动画 */
        @keyframes bounce-in {
            0% {
                transform: scale(0);
            }
            50% {
                transform: scale(1.5);
            }
            100% {
                transform: scale(1);
            }
        }

        .v-enter-active{
            transform-origin:left center;
            animation: bounce-in 1s;
        }

        .v-leave-active {
            transform-origin:left center;
            animation: bounce-in 1s reverse;
        }

        .active {
            transform-origin: left center;
            animation: bounce-in 1s;
        }

        .leave {
            transform-origin: left center;
            animation: bounce-in 1s reverse;
        }
    </style>
</head>

<body>
    <div id="app">
        <transition>
            <div v-if="show">hello world</div>
        </transition>

        <!-- 自定义过渡类名 -->
        <!-- <transition enter-active-class="active" leave-active-class="leave">
            <div v-if="show">hello world</div>
        </transition> -->

        <!-- 自定义过渡类名 -->
        <!-- <transition enter-active-class="animated swing" leave-active-class="animated hinge">
            <div v-if="show">hello world</div>
        </transition> -->
        <button @click="handleClick">toggle</button>
    </div>

    <script>
        var vm = new Vue({
            el: "#app",
            data: {
                show: true
            },
            methods: {
                handleClick: function () {
                    this.show = !this.show
                }
            }
        })
    </script>
</body>
```

######5-3 在Vue中同时使用过渡和动画
appear 和 appear-active-class 实现页面的初次动画
keyframes动画和transition动画结合
type或者duration设置动画时长
```
<head>
    <link rel="stylesheet" type="text/css" href="./animate.css">
    <style>
        .fade-enter,
        .fade-leave-to{
            opacity: 0;
        }
        .fade-enter-active,
        .fade-leave-active {
            transition: opacity 1s;
        }
    </style>
</head>

<body>
    <div id="app">
        <transition 
            :duration={enter:5000,leave:10000}
            name="fade"
            enter-active-class="animated swing fade-enter-active" 
            leave-active-class="animated shake fade-leave-active"
            appear
            appear-active-class="animated swing">
            <div v-if="show">hello world</div>
        </transition>
        <button @click="handleClick">toggle</button>
    </div>

    <script>
        var vm = new Vue({
            el: "#app",
            data: {
                show: true
            },
            methods: {
                handleClick: function () {
                    this.show = !this.show
                }
            }
        })
    </script>
</body>
```

######5-4 Vue中的 Js 动画与 Velocity.js 的结合
Vue中如何使用Js钩子实现Vue中的Js动画效果
Velocity.js 是一个简单易用、高性能、功能丰富的轻量级JS动画库。
```
<head>
    <script src="./vue.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/jquery@3.1.0/dist/jquery.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/velocity-animate@1.5.0/velocity.min.js"></script>
</head>

<body>
    <div id="app">
        <transition
            @before-enter="handleBeforeEnter"
            @enter="handleEnter"
            @after-enter="handleAfterEnter">
            <div v-if="show">hello world</div>
        </transition>
        <button @click="handleClick">toggle</button>
    </div>

    <script>
        if (window.jQuery) {
            console.log("is jq");
        }else{
            console.log("is not jq");
        }
        var vm = new Vue({
            el: "#app",
            data: {
                show: true
            },
            methods: {
                handleClick: function () {
                    this.show = !this.show
                },
                handleBeforeEnter: function(el){
                    // el.style.color = 'red'
                    el.style.opacity = 0;
                },
                handleEnter: function(el ,done){
                    // setTimeout(()=>{
                    //     el.style.color = 'green'
                    // },2000)
                    // setTimeout(()=>{
                    //     done()
                    // },4000)
                    $.Velocity(el,{
                        opacity: 1
                    },{
                        duration: 1000,
                        complete: done
                    })
                },
                handleAfterEnter: function(el){
                    // el.style.color = '#000'
                    console.log("动画结束")
                }
            }
        })
    </script>
</body>
```

######5-5 Vue中多个元素或组件的过渡
三种
```
<head>
    <script src="./vue.js"></script>
    <style>
        .v-enter,
        .v-leave-to {
            opacity: 0;
        }

        .v-enter-active,
        .v-leave-active {
            transition: opacity 1s;
        }
    </style>
</head>

<body>
    <div id="app">
        <transition mode="out-in">
            <!-- <div v-if="show" key="hello">hello world</div>
            <div v-else key="bey">bye world</div>  -->

            <child v-if="show"></child>
            <child-one v-else></child-one>

            <component :is="type"></component>
        </transition>
        <button @click="handleClick">toggle</button>
    </div>

    <script>
        Vue.component('child',{
            template:'<div>child</div>'
        })

        Vue.component('child-one',{
            template:'<div>child-one</div>'
        })

        var vm = new Vue({
            el: "#app",
            data: {
                show: true,
                type: 'child'
            },
            methods: {
                handleClick: function () {
                    this.show = !this.show
                    this.type = this.type === 'child' ? 'child-one' : 'child'
                }
            }
        })
    </script>
</body>
```

######5-6 Vue中的列表过渡
```
<head>
    <style>
        .v-enter,
        .v-leave-to {
            opacity: 0;
        }

        .v-enter-active,
        .v-leave-active {
            transition: opacity 1s;
        }
    </style>
</head>

<body>
    <div id="app">
        <transition-group>
            <div v-for="item of list" :key="item.id">
                {{item.title}}
            </div>
        </transition-group>
        <button @click="handleBtnClick">add</button>
    </div>

    <script>
        var count = 0;

        var vm = new Vue({
            el: "#app",
            data: {
                list: []
            },
            methods: {
                handleBtnClick: function () {
                    this.list.push({
                        id: count++,
                        title: 'hello world'
                    })
                }
            }
        })
    </script>
</body>
```

######5-7 Vue中的动画封装
```
<body>
    <div id="app">
        <fade :show="show">
            <div>hello world</div>
        </fade>
        <fade :show="show">
            <h1>hello world</h1>
        </fade>
        <button @click="handleClick">toggle</button>
    </div>

    <script>
        Vue.component('fade', {
            props: ['show'],
            template: `
            <transition @before-enter="handleBeforeEnter" @enter="handleEnter">
                <slot v-if="show"></slot>
            </transition>
            `,
            methods: {
                handleBeforeEnter: function (el) {
                    el.style.color = 'red'
                },
                handleEnter: function (el, down) {
                    setTimeout(() => {
                        el.style.color = 'green'
                        down()
                    }, 1000)
                }
            }
        })
        var vm = new Vue({
            el: "#app",
            data: {
                show: true
            },
            methods: {
                handleClick: function () {
                    this.show = !this.show
                }
            }
        })
    </script>
</body>
```

######5-8 本章小节
动态过渡
状态过渡

###第6章 Vue 项目预热
######6-1 Vue项目预热 - 环境配置
```
安装node.js LTS长时间维护版本
node -v # 查看node是否安装成功
npm -v # 查看包管理工具npm是否安装成功
码云、github配置公钥
vue官网安装有命令行工具（CLI）构建工程travel，如下：

#全局安装vue-cli
npm install --global vue-cli
#创建一个基于webpack模板的新项目
vue init webpack travel
cd travel
npm install
npm run dev

git status
git add .
git commit -m 'project initialized'
git push
```

######6-2 Vue项目预热 - 项目代码介绍
package.json 依赖文件
package-lock.json package版本
index.html 首页模板
.eslintrc.js 代码规范
.eslintignore 代码规范忽略
.editorconfig 配置编辑器语法
.babelrc 语法解析器 
static目录 静态资源
src目录 源代码
- main.js 项目入口文件
- App.vue 原始根组件
- router目录 路由
- components目录 小组件
- assets目录 图片资源

node_modules目录 依赖包
config目录 配置文件
- index.js 基础配置信息
- prod.env.js 线上配置信息
- dev.env.js  开发配置信息

build目录 打包配置

######6-3 Vue项目预热 - 单文件组件与Vue中的路由
main.js 定义了一个Vue根实例 挂载到 index.html app的div元素
main.js 中components中App指的是App.vue局部组件，这是ES6语法
main.js 模板中直接渲染了这个App局部组件
main.js router 路由配置，就是根据网址的不同，返回不同的内容给用户

App.vue 以vue结尾的文件叫做单文件组件，App.vue是整个页面的根组件
<template></template> 中放组件的模板
<script></script> 中放组件的逻辑
<style></style> 中放组件的CSS样式
<router-view/> 显示的是当前路由地址所对应的内容

查看router中的index.js文件，当访问根路径，路由是HelloWorld。@代表src目录
查看HelloWorld.vue组件

######6-4 Vue项目预热 - 单页应用VS多页应用
![image.png](https://upload-images.jianshu.io/upload_images/1956963-493e7e4f51a0b288.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
```
npm run start和npm run dev 区别？？？npm scripts的内容，需要了解node...
<router-link to="/list">列表页</router-link> # 路由跳转
Vue是单页应用，靠JS感知路由跳转，前端路由
```
![image.png](https://upload-images.jianshu.io/upload_images/1956963-59a75992f2674656.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

######6-5 Vue项目预热 - 项目代码初始化
修改viewport这个meta标签，符合移动端
```
<meta name="viewport" content="width=device-width,initial-scale=1.0,
    minimum-scale=1.0,maximum-scale=1.0,user-scalable=no">
```
引入reset.css 重置页面样式表，使手机样式统一
引入border.css 解决1px边框问题
npm install fastclick --save 解决某些机型click事件300毫秒延迟问题
import fastClick from 'fastclick'
fastClick.attach(document.body)
npm run start 重启服务
iconfont 登录，选择图标管理，我的项目，创建项目
代码上传，结束。

###第7章 项目实战 - 旅游网站首页开发
######7-1 Vue项目首页 - header区域开发
stylus：富于表现力、动态的、健壮的CSS
```
安装stylus
npm install stylus --save
npm install stylus-loader --save

stylus的使用
1rem = html font-size = 50px
scoped使样式不对其他组件产生影响

这一节主要讲Header.vue
```

######7-2 Vue项目首页 - iconfont 的使用和代码优化
[iconfont-阿里巴巴矢量图标库](http://www.iconfont.cn/)
```
图标库、官方图标库、大麦网官方图标库，购物车内容添加至购物车，下载。
需要用到iconfont.eot、iconfont.svg、iconfont.ttf、iconfont.woff、iconfont.css,导入项目，需要修改iconfont.css
main.js import iconfont.css
<span class="iconfont">&#xe624;</>  其中串的代码需要在官网查看 
varibles.styl 通用颜色封装 $bgColor
@import varibles.styl 可以使用$bgColor
注意：CSS中引入其他的CSS @前需要加~
styles目录起一个别名：可以在build/webpack.base.conf.js中resolve起别名，需要重启服务npm run start。
提交代码
```

######7-3 Vue项目首页 - 首页轮播图
**[vue-awesome-swiper](https://github.com/surmon-china/vue-awesome-swiper)**

```
创建分支，写代码，合并分支
创建index-swiper分支
git pull
git checkout index-swiper
git status

第三方轮播插件：vue-awesome-swiper
npm install vue-awesome-swiper@2.6.7 --save
npm run start

# mount with global
import Vue from 'vue'
import VueAwesomeSwiper from 'vue-awesome-swiper'
import 'swiper/dist/css/swiper.css'
Vue.use(VueAwesomeSwiper, /* { default global options } */)

这一节主要讲Swiper.vue
data: function() {} 在ES6可以简化成 data () {}
width: 100% height: 31.25vw  #高度是宽度的百分之31.25

样式穿透：
.wrapper >>> .swiper-pagination-bullet-active {
    background: #fff !important;
}

git add .
git commit -m 'change'
git push
git checkout master
git merge origin/index-swiper
git push
```

######7-4 Vue项目首页 - 图标区域页面布局与实现
获取更多扩展程序,下载chrome插件Vue.js devtools
可以通过[Chrome访问助手](http://www.ggfwzs.com/)实现科学上网
```
创建index-icons分支
这一节主要讲Icons.vue
通过Vue.js devtools查看计算属性pages中的数据
mixins.styl 对CSS进行封装
提交代码，合并到主分支
```

######7-6 Vue项目首页 - 热销推荐组件开发
```
创建index-recommend分支
npm run start
这一节主要讲Recommend.vue
border-bottom引入一像素边框
```

######7-7 Vue项目首页 - 开发周末游组件
```
这一节主要讲Weekend.vue
```

######7-8 Vue项目首页 - 使用 axios 发送 ajax 请求
**[axios](https://github.com/axios/axios)**
```
创建index-ajax分支
npm install axios --save
这一节主要讲Home.vue
import axios from 'axios'
借助生命周期钩子发送ajax请求
http://localhost:8080/static/mock/index.json可以访问到数据
webpack-dev-server 工具提供 config/index.js proxyTable配置请求转发，方便开发环境模拟数据
```

######7-9 Vue项目首页 - 首页父子组组件间传值
```
v-if="list.length"防止轮播图第一次不是显示第一张
```

###第8章 项目实战 - 旅游网站城市列表页面开发
######8-1 Vue项目城市选择页 - 路由配置
```
创建city-router分支
配置路由router/index.js
这一节主要讲City.vue和Header.vue
<router-link to="/city"></router-link>会对其中的颜色改变，需要重写一下CSS样式
npm install better-scroll --save
```

######8-2 Vue项目城市选择页 - 搜索框布局
```
创建city-search分支
这一节主要讲Search.vue
```

######8-3 Vue项目城市选择页 - 列表布局
```
创建city-list分支
这一节主要讲List.vue
border-topbottom
```

######8-4 Vue项目城市选择页 - BetterScroll 的使用和字母表布局
**[better-scroll](https://github.com/ustbhuangyi/better-scroll)**
```
npm install better-scroll --save

import BScroll from 'better-scroll'

mounted () {
  this.scroll = new BScroll(this.$refs.wrapper)
}

Alphabet.vue 字母表
```

######8-5 Vue项目城市选择页 - 页面的动态数据渲染
```
创建city-ajax分支
City.vue通过axios获取数据，并显示
```

######8-6 Vue项目城市选择页 - 兄弟组件数据传递
```
创建city-components分支
Alphabet.vue点击事件
兄弟组件通过父组件传值

watch: {
  letter () {
    if(this.letter){
      const element = this.$refs[this.letter][0]
      this.scroll.scrollToElement(element)
    }
  }
}
Alphabet.vue touchstart touchmove touchend事件，监听滑动
完成字母表点击和滑动改变列表位置功能
```

######8-7 Vue项目城市选择页 - 列表性能优化
```
字母A距离顶部高度只计算一次
  updated () {
    this.startY = this.$refs['A'][0].offsetTop
  }

函数节流
  methods: {
    handleTouchStart () {
      this.touchStatus = true
    },
    handleTouchMove (e) {
      if (this.touchStatus) {
        // 函数截流
        if (this.timer) {
          clearTimeout(this.timer)
        }
        this.timer = setTimeout(() => {
          const touchY = e.touches[0].clientY - 79
          const index = Math.floor((touchY - this.startY) / 14)
          if (index >= 0 && index < this.letters.length) {
            this.$emit('change', this.letters[index])
          }
        })
      }
    },
    handleTouchEnd () {
      this.touchStatus = false
    }
  }
```

######8-8 Vue项目城市选择页 - 搜索逻辑实现
```
创建city-search-logic分支
Search.vue 编写搜索逻辑
```

######8-9 Vue项目城市选择页 - Vuex实现数据共享
**[vuex](https://vuex.vuejs.org/)**
```
创建city-vuex分支
查看官网vuex介绍
npm install vuex --save # 安装vuex
store/index.js
imoort store from './store'
this.$store.state.city # 获取数据
this.$store.dispatch('changeCity',city)
查看官网vuex中state actions mutations，理解数据传递
组件可以直接调用mutations，省略actions这个步骤：this.changeCity(city)
vue router编程式的导航：this.$router.push('/')
```

######8-10 Vue项目城市选择页 - Vuex的高级使用及localStorage
```
localStorage实现本地存储
localStorage.city = city
city = localStorage.city || '上海'
store下index.js拆分出state.js和 mutations.js几个部分

import { mapState } from 'vuex'
export default {
  computed: {
   ...mapState(['city']) # 把vuex中的属性映射到计算属性中
  }
}
这样this.$store.state.city可以写成this.city

import { mapMutations } from 'vuex'
methods: {
  ...mapMutations(['changeCity']) 
}
这样this.$store.commit('changeCity',city)可以改写成：this.changeCity(city)

介绍mapGetters的使用，就不贴代码了，类似计算属性的作用
介绍module的使用
```

######8-11 Vue项目城市选择页 - 使用keep-alive优化网页性能
```
创建city-keepalive分支
每次路由切换，mounted钩子都会被执行，ajax都会发送请求，性能很低
<keep-alive>
  <router-view/>
</keep-alive>
城市改变，需要调用ajax请求
使用activated钩子代替mounted钩子，结合lastCity == city解决此问题
```

###第9章 项目实战 - 旅游网站详情页面开发
######9-1 Vue项目详情页 - 动态路由和banner布局
```
创建detail-banner分支
<router-link tag="li"></router-link>
动态路由：'/detail/:id'，可以通过this.$route.params.id获取
iconfont替换图片需要替换iconfont目录下的4个字体文件，iconfont.css也需要修改base64的数据
渐变色：background-image: linear-gradient(top, rgba(0, 0, 0, 0), rgba(0, 0, 0, 0.8))
```

######9-2 Vue项目详情页 - 公用图片画廊拆分
```
公用画廊组件Gallary.vue
Banner.vue
使用vue-awesome-swiper完成画廊页面
vue-awesome-swiper底层实际上使用的是swiper3，查看swiper3 pagination使用方法

# 当swiper或父级元素变化自我刷新
observeParents: true
observer: true 
```

######9-3 Vue项目详情页 - 实现Header渐隐渐显效果
```
创建detail-header分支
主要讲Header.vue
window.addEventListener('scroll',this.handleScroll)
通过opacity实现渐隐渐显效果
```

######9-4 Vue项目详情页 - 对全局事件的解绑
```
activated () {
  window.addEventListener('scroll',this.handleScroll)
},
deactivated () {
  window.removeEventListener('scroll',this.handleScroll)
}
```

######9-5 Vue项目详情页 - 使用递归组件实现详情页列表
```
创建detail-list分支
主要编写List.vue

自己调用自己：
<div v-if="item.children">
  <detail-list :list="item.children"></detail-list>
</div>
```

######9-6 Vue项目详情页 - 动态获取详情页面数据
```
创建detail-ajax分支
动态路由获取路由参数：this.$route.params.id
axios.get('/api/detail.json',{
  params: {
    id: this.$route.params.id
  }
}).then(this.handleGetDataSucc)

keepalive缓存会导致详情页面请求不变，需要配合activated钩子
也可以keepalive 不给detail加缓存：
<keep-alive exclude="Detail">
  <router-view/>
</keep-alive>

组件的名字在递归组件、keepalive exclude 和vue插件查看页面时可以用到

vue-router滚动行为修改,在router/index中添加如下代码：
scrollBehavior (to, from, savedPosition) {
  return {x: 0, y: 0}
}
```

######9-7 Vue项目详情页 - 在项目中加入基础动画
```
创建detail-animation分支
common/fade/FadeAnimation.vue 制作渐隐渐显动画效果

```

###第10章 实战项目 - 项目的联调，测试与发布上线
######10-1 Vue项目的联调测试上线 - 项目前后端联调
```
修改config/index.js中proxyTable配置服务器ip地址
```
######10-2 Vue项目的联调测试上线 - 真机测试
```
真机会拒绝访问，8080端口无法被外界访问。
webpack dev server默认不支持通过ip形式访问
需要修改package.json中scripts dev 增加：--host 0.0.0.0
npm run start 重启服务器，ip地址可以访问

字母表bug修改：@touchstart.prevent 阻止默认行为，prevent 是事件修饰符

解决Android手机浏览器兼容问题,白屏：
npm install babel-polyfill --save
main.js 引入 import 'babel-polyfill' 
```
######10-3 Vue项目的联调测试上线 - 打包上线
```
npm run build
Build complete. 会生成dist目录，这就是最终上线代码。

如果不想上线在localhost而是想在localhost/project
需要修改config目录下的index.js中build部分的内容，修改assetsPublicPath: '/project'
```
######10-4 Vue项目的联调测试上线 - 异步组件实现按需加载
```
```
######10-5 Vue项目的联调测试上线 - 课程总结与后续学习指南
```
多看vue官方文档、vue-router文档、vuex文档、vue服务器端渲染、awesome-vue插件、vue源码
完结撒花。。。
```

### 源码
######[基础 vue-demo](https://github.com/liuhuiAndroid/vue-demo)
######[项目 travel](https://github.com/liuhuiAndroid/travel)
