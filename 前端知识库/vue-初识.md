###一、什么是vue
#### [*vue*.js官网](https://cn.vuejs.org/)
>Vue是一套用于构建用户界面的渐进式框架

###二、vue的基础知识
- vue单文件
单文件就是以\*.vue结尾的文件。最终通过webpack也会编译成*.js在浏览器运行。
.vue的内容：<template></template> + <script></script> + <style></style>
    - 1:template中只能有一个根节点 2.x
    - 2:script中  按照 export default {配置} 来写
    - 3:style中 可以设置scoped属性，让其只在template中生效

- 以单文件的方式启动vue
webpack找人来理解你的单文件代码
    - vue-loader,vue-template-compiler,代码中依赖vue
启动命令:
`..\\node_modules\\.bin\\webpack-dev-server --inline --hot --open --port 8888`

- HelloWorld
```
1、在index.html中写上div id="app"
<div id="app"></div>
2、在main.js中引入Vue
import Vue from 'vue';
import App from './app.vue';
3、在main.js中创建一个Vue的实例，一般一个项目，大多就是一个vue实例来完成显示
new Vue({
    // el:'#app', //目的地
    // render:'dom结构'//渲染的内容
    el:'#app',
    render:function(creater){
        return creater(App); //App包含 template/script/style,最终生成DOM结构
    }
})
4、创建app.vue，写好模板代码
<template></template> + <script></script> + <style></style>
5、需要配置package.json、webpack.config.js、下载好node_modules
npm init -y 生成package.json
npm i  下载依赖
* package文件的作用：
package 可以对开发的命令进行简化  npm run dev
6、运行
npm run dev
``` 

- 双向数据流
一向：js内存属性发生改变，影响页面的变化
一向：页面的改变影响js内存属性的改变
```
app.vue
<template>
    <div>
           第一个vue程序<br/>
           输入：<input type="text" v-model="text"><br/>
           变化：{{text}}<br/>
            <ul>
                <li v-for="person in list">
                    {{person.name}}
                </li>
            </ul>
    </div>
</template>
<script>
    export default {
        data(){
            return {  //存放数据
                text:'大家好',
                list:[{name:'jack'},{name:'rose'}]
            }
        }
    }
</script>
<style></style>
```

- 启动的目录结构
project
    - src:源代码
    - dist:存放真实上线的代码，机器才能看懂
    - package.json:包信息描述
      创建文件夹后执行npm init -y可以生成package.json
      npm i 安装所有devDependencies 和 dependencies里面的模块
      npm i --production 安装package.json中dependencies部分的模块
      package.json中scripts下的dev：对开发命令的简化
      package.json中scripts下的build：打包代码到生产环境目录
    - webpack.config.js: 打包生成dist下的代码
    - node_modules: 项目中只有一个

webpack命令会读取webpack.config.js文件，生成到dist目录下
webpack-dev-server命令会运行src下的代码，虚拟出build.js来测试

- vue的常用指令
```
app.vue
<template>
    <div>   
        <pre>
                * v-text 是元素的innerText只能在双标签中使用
                * v-html 是元素的innerHTML，不能包含<!-- {{xxx}} -->
                * v-if 元素是否移除或者插入
                * v-show 元素是否显示或隐藏
                * v-model 
        </pre>

        v-text :
        <span v-text="text"></span>
        <input type="text" name="" v-text="text"><!-- input是单标签所以无效 -->
        <hr/>
        v-html :
        <span v-html="html"></span>
        <hr/>
        v-if :
        <div v-if="isShow" style="height:100px;background-color:red;"></div>
        <hr/>
        v-show:
        <div v-show="isShow" style="height:100px;background-color:green;"></div>
        <hr/>
        v-model:
        <input type="text" name="" v-model="mTest">{{mTest}}<br/>
        给下面的input的value赋值用v-bind:value="mTest"
        <input type="text" name="" v-bind:value="mTest">
    </div>
</template>
<script>
    export default {
        data(){
            return {
                text:'我是v-text',
                html:`
                    <ul>
                        <li>哈哈</li>
                        <li>呵呵</li>
                    </ul>
                `,
                isShow:false,
                mTest:'哈哈'
            }
        }
    }
</script>
<style>
    
</style>
```

- class结合v-bind的使用 
- methods和on
```
<template>
    <div>
        <!-- class取一个,返回一个字符串 -->
        <div v-bind:class="isRed?'red':'green'">单个class</div>
        <!-- class取多个,返回一个对象 -->
        <div :class="{'red':true,'big':true}">多个class</div>
        复杂情况，通过遍历，根据当前对象的成绩，匹配成绩和样式的清单对象，用成绩做key，取对象的value，最终返回字符串做样式名
        <ul>
            <li v-for="stu in stus" :class="{'A':'red','B':'green'}[stu.score]">
                {{stu.name}}
            </li>
        </ul>
        <button v-on:click="isRed = !isRed">点我看变化</button>
        <button @click="change">点我看变化</button>
    </div>
</template>
<script>
    export default {
        data(){
            return {
                isRed:false,
                stus:[{name:'jack',score:'A'},{name:'rose',score:'B'}]
            }
        },
        //声明函数，属于组件对象的
        methods:{
           //包含多个函数名做key，函数体做value
           change(){
               //在script中能使用的对象，基本template中也能使用，但是
               //在script中加this,template中不需要this
               this.isRed = !this.isRed;
               this.stus.push({
                name:'mick',score:'A'
               })
           }
        }
    }
</script>
<style>
.red{
    background-color: red;
}
.green{
    background-color: green;
}
.big{
    font-size: 30px;
}
</style>
```

- v-for
```
<template>
    <div>
        <ul>
          <li v-for="(stu,index) in stus" v-bind:key="index">
            index:{{index}},stu:{{stu}}
          </li>
          使用对象的方式 <hr/>
          <li v-for="(value,key,index) in person" :key="index">
              value:{{value}}
              key:{{key}}
              index:{{index}}
          </li>
        </ul>
    </div>
</template>
<script>
    export default {
        data(){
            return {
                stus:[{name:'jack',score:'A'},{name:'rose',score:'B'}],
                person:{name:'mick',alise:'迈克'}
            }
        }
    }
</script>
<style>
.red{
    background-color: red;
}
.green{
    background-color: green;
}
.big{
    font-size: 30px;
}
</style>
```

- 漂亮的列表
```
<template>
    <div>
        <ul>
            <li v-for="(hero,index) in heros" :key="index" 
              :class="{'A':'red','B':'blue','C':'green','D':'pink'}[hero.score]">
              {{hero.name}},{{hero.score}}
              <button @click="del(index)">删除</button>
            </li>
        </ul>
        英雄姓名:<input type="text" name="" v-model="name"> <br/>
        英雄成绩:<input type="text" name="" v-model="score">
        <button @click="addHero">添加英雄</button>
    </div>
</template>
<script>
    export default {
        data(){
            return {
                name:'',score:'',
                heros:[{
                  id:1,
                  name:'奥利安娜',
                  score:'A'
                },{
                  id:2,
                  name:'崔斯特',
                  score:'B'
                },{
                  id:3,
                  name:'妲己',
                  score:'C'
                },{
                  id:4,
                  name:'阿拉斯加',
                  score:'D'
                }]
                
            }
        },
        methods:{
            addHero(){
                  this.heros.push({
                    name: this.name,
                    score:this.score
                  });
                  this.name = '';
                  this.score = '';
            },
            del(index){
              this.heros.splice(index,1);
            }
        }
    }
</script>
<style>
.red{
    background-color: red;
}
.green{
    background-color: yellowgreen;
}
.blue{
    background-color: skyblue;
}
.pink{
    background-color: hotpink;
}
</style>
```

- 父子组件的使用
```
父
<template>
    <div>
        <header-vue></header-vue>
        <body-vue></body-vue>
        <footer-vue></footer-vue>
    </div>
</template>
<script>
    //引入子组件对象
    import headerVue from './components/header.vue';
    import bodyVue from './components/body.vue';
    import footerVue from './components/footer.vue';
    export default {
        data(){
            return {   
            }
        },
        methods:{},
        //必须声明
        components:{
          //组件名（在模板中使用） :组件对象
          headerVue:headerVue,
          bodyVue, //简写
          footerVue
        }
    }
</script>
<style>
</style>
```
```
子
<template>
    <div>
        我是头部
    </div>
</template>
<script>
    export default {
        data(){
            return {
            }
        }
    }
</script>
<style scoped>
div{
    height: 100px;
    background-color: yellowgreen;
}
</style>
```

- 全局组件的使用
在main.js中声明全局组件
```
//引入子组件对象
import headerVue from './components/header.vue';
import bodyVue from './components/body.vue';
import footerVue from './components/footer.vue';

//声明全局组件
Vue.component('headerVue', headerVue); 
Vue.component('bodyVue', bodyVue);
Vue.component('footerVue', footerVue);
```

- 父组件传递值给子组件
父组件通过子组件的属性将值进行传递
子组件需要声明：
    - 根属性props:['prop1','prop2']
    - 在页面中直接使用{{prop1}}
    - 在js中应该通过this.prop1获取
```
父
<template>
    <div>
        <header-vue textone="大"></header-vue>
        <body-vue v-bind:texttwo="text2"></body-vue>
        <footer-vue></footer-vue>
    </div>
</template>
<script>
    export default {
        data(){
            return {
                text2:'哈哈呵呵'   
            }
        }
    }
</script>
<style>
</style>
```
```
header.vue
<template>
    <div>
        {{textone}}
        <button @click="show">从js中获取父组件值</button>
    </div>
</template>
<script>
    export default {
        data(){
            return {
            }
        },
        //接受父组件值参数
        props:['textone'],
        methods:{
            show(){
                alert(this.textone)
            }
        }
    }
</script>
<style scoped>
div{
    height: 100px;
    background-color: yellowgreen;
 }
</style>
```
```
body.vue
<template>
    <div>
        {{texttwo}}
    </div>
</template>
<script>
    export default {
        data(){
            return {
            }
        },
        props:['texttwo']
    }
</script>
<style scoped>
div{
    height: 100px;
    background-color: skyblue;
}
</style>
```

- vueBus通信使用
通过new Vue()这样的一个对象，通过$on('事件名',fn(prop1,pro2))来接收消息
另一个组件引入同一个对象,  通过$emit('事件名',prop1,pro2)来发送消息
```
在main.js中引入子组件对象
import sub from './components/sub.vue';
Vue.component('subVue', sub);
```
```
connector.js
import Vue from 'vue';
var connector = new Vue();
export default connector;
```
```
app.vue
<template>
    <div>
        <subVue></subVue>
        <button @click="listen">听电话</button>
    </div>
</template>
<script>
    import connect from './connector.js';
    export default {
        data(){
            return {
                text2:'哈哈呵呵'   
            }
        },
        methods:{
            listen(){
                connect.$on('phone',function(msg){
                    console.log(msg);
                })
            }
        },
    }
</script>
<style>
</style>
```
```
sub.vue
<template>
    <div>
        <button @click="callDaddy">打电话</button>
    </div>
</template>
<script>
    import connect from '../connector.js';
    export default {
        data(){
            return {
            }
        },
        methods:{
            callDaddy(){
                connect.$emit('phone','62分钟来');
            }
        }
    }
</script>
<style scoped>
div{
    height: 100px;
    background-color: hotpink;
}
</style>
```
----------------------------------------------------------
- 过滤器
```
<template>
    <div>
        <input type="text" name="" v-model="text"><br/>
        反转：{{text | myFilter}}
    </div>
</template>
<script>
    import SubVue from './sub.vue';
    export default {
        filters:{
            myFilter:function(value){
                return value.split('').reverse().join('');
            }
        },
        data(){
            return {
                text:''
            }
        }
    }
</script>
<style>
</style>
```
还可以在main.js中定义全局过滤器
```
Vue.filter('myFilter', function(value) {
    return '我是全局过滤器';
});
```
- 获取DOM元素
```
app.vue
<template>
    <div>
        <sub-vue ref="sub"></sub-vue>
        <hr/>
        <div ref="myDiv">
            {{text}}
        </div>
    </div>
</template>
<script>
    import SubVue from './sub.vue';
    export default {
        data(){
            return {
                text:'123'
            }
        },
        components:{
            subVue:SubVue
        },
        //组件创建后,数据已经完成初始化，但是DOM还未生成
        created(){  //事件的处理函数(created)
            console.log('created:',this.$refs.myDiv);//获取不到
            //打印：created: undefined
        },
        //数据装载DOM上后，各种数据已经就位,将数据渲染到DOM上，DOM已经生成
        mounted(){
            console.log('sub:',this.$refs.sub.$el);
            //获取组件对象，并获取到其的DOM对象
            this.$refs.sub.$el.innerHTML = '嘻嘻';
            console.log('mounted:',this.$refs.myDiv);
            // 涉及DOM类的操作
            this.$refs.myDiv.innerHTML = '哈哈呵呵';
            // 涉及到数据的操作
            this.text = '嘻嘻呵呵'
        }
    }
</script>
<style>
</style>
```
```
sub.vue
<template>
    <div>
        sub.vue
        {{'大家好，我是sub'}}
    </div>
</template>
<script>
    export default {
        data(){
            return {
            }
        }
    }
</script>
<style>
</style>
```

- element-ui
#### [element 网页端](https://github.com/ElemeFE/element)
#### [mint-ui 移动端](https://github.com/ElemeFE/mint-ui)
`下载：npm i mint-ui -S`
`需要在Html中加入头来适配移动端 <meta name="viewport" content="width=device-width, user-scalable=no, initial-scale=1.0, maximum-scale=1.0, minimum-scale=1.0">`  `meta:vp`
在main.js中引入并安装
```
import MintUi from 'mint-ui';
import 'mint-ui/lib/style.css';
Vue.use(MintUi);
```
app.vue
```
<template>
    <div>  
        <mt-header title="标题">
            <mt-button icon="back">返回</mt-button>
            <mt-button>关闭</mt-button>
            <mt-button icon="more" slot="right"></mt-button>
        </mt-header>
        <button @click="handleClose">显示弹出框</button>
        <mt-switch v-model="value"></mt-switch>
        <mt-swipe :auto="4000">
            <mt-swipe-item>1</mt-swipe-item>
            <mt-swipe-item>2</mt-swipe-item>
            <mt-swipe-item>3</mt-swipe-item>
        </mt-swipe>
    </div>
</template>
<script>
    //在js部分所有变量都是模块作用域,如果需要使用就必须引入
    import { Toast } from 'mint-ui';
    export default {
        data(){
            return {
                text:'123',
                value:false
            }
        },
        methods:{
            handleClose(){
                Toast('提示信息');
            }
        }
    }
</script>
<style>
</style>
```

- Wappalyzer
Wappalyzer是一款功能强大且非常实用网站技术分析插件
[Wappalyzer 官网](https://www.wappalyzer.com/)

- vue-router
在main.js中安装路由插件，创建路由对象并配置路由规则
```
import Vue from 'vue';
import VueRouter from 'vue-router';
Vue.use(VueRouter);

import App from './components/app.vue';
import Home from './components/home.vue'

//创建路由对象并配置路由规则
let router = new VueRouter({
    routes: [
        { path: '/home', component: Home }
    ]
});

new Vue({
    el: '#app',
    //让vue知道我们的路由规则
    router: router,
    render: c => c(App),
})
```
app.vue
```
<template>
    <div>
        <router-view></router-view>
    </div>
</template>
<script>
    export default {
        data(){
            return {
            }
        }
    }
</script>
<style>
</style>
```

- 命名方式使用router-link
```
let router = new VueRouter({
    routes: [
        { name: 'list', path: '/list', component: List },
        { name: 'detail', path: '/detail', component: Detail },
        { name: 'detail', path: '/detail/:id', component: Detail }
    ]
});
```
- SPA（单页应用）
main.js
```
import Vue from 'vue';
import VueRouter from 'vue-router';
Vue.use(VueRouter);

import App from './components/app.vue';
import Music from './components/music.vue'
import Movie from './components/movie.vue'

let router = new VueRouter({
    routes: [
        { name: 'music', path: '/music', component: Music },
        { path: '/movie', component: Movie }
    ]
});

new Vue({
    el: '#app',
    router,
    render: c => c(App),
})
```
```
<template>
    <div>
        <div class="h">头部
            <router-link :to="{name:'music'}">进入音乐1</router-link>
            <router-link :to="{name:'music'}">进入音乐2</router-link>
            <router-link :to="{name:'music'}">进入音乐3</router-link>
            <!-- 也可以跟路径使用 -->
            <router-link to="/movie">进入电影</router-link>
        </div>
        <router-view class="b"></router-view>
        <div class="f">底部</div>
    </div>
</template>
<script>
    export default {
        data(){
            return {
            }
        }
    }
</script>
<style scoped>
    .h{
        height: 100px;
        background-color: yellowgreen;
    }
    .b{
        height: 100px;
        background-color: skyblue;
    }
    .f{
        height: 100px;
        background-color: hotpink;
    }
</style>
```

- SPA（单页应用）  router-link对象方式传递参数
main.js
```
import Vue from 'vue';
import VueRouter from 'vue-router';
Vue.use(VueRouter);

import App from './components/app.vue';
import List from './components/list.vue'
import Detail from './components/detail.vue'

//创建路由对象并配置路由规则
let router = new VueRouter({
    routes: [
        { name: 'list', path: '/list', component: List },
        //以下规则匹配  /detail?xxx=x
        //查询字符串path不用改
        { name: 'detail', path: '/detail', component: Detail },
        //{name:'detail',params:{id:index}  } -> /detail/12
        { name: 'detail', path: '/detail/:id', component: Detail }
    ]
});

new Vue({
    el: '#app',
    router,
    render: c => c(App),
})
```
app.vue
```
<template>
    <div>
        <div class="h">头部</div>
        <router-view class="b"></router-view>
        <div class="f">底部</div>
    </div>
</template>
<script>
    export default {
        data(){
            return {
            }
        }
    }
</script>
<style scoped>
    .h{
        height: 100px;
        background-color: yellowgreen;
    }
    .b{
        height: 300px;
        background-color: skyblue;
    }
    .f{
        height: 100px;
        background-color: hotpink;
    }
</style>
```
list.vue
```
<template>
    <div>
        查询字符串列表:
        <ul>
            <li v-for="(hero,index) in heros" :key="index">
                    {{hero.name}}
                    <router-link :to="{name:'detail',query:{id:index}}">查看</router-link>
            </li>
        </ul>
        path列表:
        <ul>
            <li v-for="(hero,index) in heros" :key="index">
                    {{hero.name}}  
                    <router-link :to="{name:'detail',params:{id:index}}">查看</router-link>
            </li>
        </ul>
    </div>
</template>
<script>
    export default {
        data(){
            return {
                heros:[{
                    name:'李白'
                },{
                    name:'王昭君'
                }]
            }
        }
    }
</script>
<style>
</style>
```
detail.vue
```
<template>
    <div>
        详情
    </div>
</template>
<script>
    export default {
        data(){
            return {
            }
        },
        created(){
            //获取路由参数
            //vue-router中挂载两个对象的属性
            //$route（信息数据）  $router (功能函数)
            console.log(this.$route.query);
            console.log(this.$route.params);
        },
        mounted(){
        }
    }
</script>
<style>
</style>
```

- 编程导航
```
// 跳转到音乐
this.$router.push('/music');
// 跳转带参数
this.$router.push({
                    name:'music',query:{ id:1,name:2  }
                });
// 后退
this.$router.go(-1); 
// 前进
this.$router.go(1);
```

- 路由
路由 = 锚点监视+模板替换
路由原理：
```
<html>
    <head>
        <meta charset="UTF-8">
        <title>Document</title>
    </head>
    <body>
        <div class="h">我是头部</div>
        <div id="content" class="b"></div>
        <div class="f">我是底部</div>
        <script type="text/javascript">
            //监视锚点值的改变
            window.addEventListener('hashchange', function() {
                var text = '';
                switch (location.hash) {
                    case '#/music':
                        text = '各种音乐的数据';
                        break;
                    case '#/movie':
                        text = '各种电影的数据';
                        break;
                }
                document.getElementById('content').innerHTML = text;
            })
        </script>
    </body>
</html>
```

- 重定向和404
```
let router = new VueRouter();
router.addRoutes([
    { path: '/', redirect: { name: 'home' } },
    { name: 'home', path: '/home', component: Home },
    { path: '*', component: NotFound }
]);
```

- 多视图
main.js
```
let router = new VueRouter({
    routes: [{
            path: '/',
            components: {
                header: header,
                default: header,
                footer: footer
            }
        }
    ]
});
```
app.vue
```
<template>
    <div>
        <div>头部</div>
        <hr/>
            <router-view name="header"></router-view>
            <router-view ></router-view>
            <router-view name="footer"></router-view>
        <hr/>
        <div>底部</div>
    </div>
</template>
<script>
    export default {
        data(){
            return {
            }
        }
    }
</script>
<style scoped>
</style>
```

- 嵌套路由
节约不变数据的重复渲染
多页应用开发
main.js
```
import Vue from 'vue';
import VueRouter from 'vue-router';

import App from './components/app.vue';
import header from './components/header.vue'
import footer from './components/footer.vue'
import Music from './components/music.vue'
import Oumei from './components/oumei.vue'
import Guochan from './components/guochan.vue'
Vue.component('headerVue', header);
Vue.component('footerVue', footer);
Vue.use(VueRouter);

let router = new VueRouter({
    routes: [{path: '/',redirect: { name: 'music' }},
        {
            name: 'music',
            path: '/music',
            component: Music,
            children: [
                { name: 'music_oumei', path: 'oumei', component: Oumei },
                { name: 'music_guochan', path: 'guochan', component: Guochan }
            ]
        }
    ]
});

new Vue({
    el: '#app',
    router,
    render: c => c(App),
})
```
app.vue
```
<template>
    <div>
        <header-vue></header-vue>
        <router-view></router-view>
        <footer-vue></footer-vue>
    </div>
</template>
<script>
    export default {
        data(){
            return {
            }
        }
    }
</script>
<style scoped>
</style>
```
music.vue
```
<template>
    <div>
        音乐世界
        <router-link :to="{name:'music_oumei'}">欧美音乐</router-link>
        <router-link :to="{name:'music_guochan'}">国产音乐</router-link>
        <hr/>
        <!-- 变化的音乐数据 -->
        <router-view></router-view>
    </div>
</template>
<script>
    export default {
        data(){
            return {
            }
        }
    }
</script>
<style>
</style>
```

- vue-resource 
1、下载插件：npm i vue-resource axios -S
2、在main.js中
引入：import VueResource from 'vue-resource'
安装：Vue.use(VueResource);  //未来通过this.$http使用
3、可以在created()中
console.log(this.$http) 看看有没有加载成功
4、测试get和post
```
            this.$http.get('http://localhost:8899/api/getPic')
            .then(res=>{
                this.data = res.body.message;
            },err=>{
                console.log(err)
            })
```
```
            this.$http.post('http://localhost:8899/api/postComment/1',{
                content:'额'
            },{
                emulateJSON:true
            })
            .then(res=>{
                this.data = res.body.message;
            },err=>{
                console.log(err);
            })
```
5、解决options预检请求
`emulateJSON:true`

- axios
main.js
```
import Axios from 'axios';
Axios.defaults.baseURL = 'http://localhost:8899/api/';
Vue.prototype.$axios = Axios;
```
```
           this.$axios.get('getPic',{
                params:{
                    id:'123'
                },
                baseURL:'http://www.baidu.com'
           })
           .then(res=>{
            this.data = res.data.message;
           })
           .catch(err=>{
            console.log(err);
           })
           .finally(()=>{
            console.log('最终');
           })
```
```
           this.$axios.post('postComment/1','content=1')
           .then(res=>{
                this.data = res.data.message;
           })
           .catch(err=>{
            console.log(err);
           })
```
- ###vue-element-admin
####[vue-element-admin 中文文档](https://panjiachen.github.io/vue-element-admin-site/#/)
 
