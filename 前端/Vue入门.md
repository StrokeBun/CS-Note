[toc]



## Vue 入门

### 1. 介绍

Vue 针对 HTML + CSS + JS： 视图层

配合其他模块使用

网络通信：axios

页面跳转：vue-router

状态管理：vuex



Vue 基于 **MVVM** 模式，使用 **观察者** 模式实现

![](img/mvvm.jpg)

``` javascript
// 第一个程序
<div id="app">
    {{message}}
</div>

<script src="../js/vue.js"></script>
<script>
    var app = new Vue({
        el: "#app",
        data: {
            message: "Hello Vue!"
        }
    });
</script>
```

### 2. 基本元素

#### 2.1 判断与循环

带有 v- 前缀的指令，会在渲染的 DOM 应用特殊的响应式行为

``` javascript
<div id="app">
    // 条件判断
    <h1 v-if="type==='A'">A</h1>
	<h1 v-else-if="type==='B'">B</h1>
    <h1 v-else>C</h1>
	// 循环
    <li v-for="item in items">
        {{item.message}}
    </li>
</div>

<script src="../js/vue.js"></script>
<script>
    var app = new Vue({
        el: "#app",
        data: {
            type: 'A',
            items: [
                {message:"bzzb"},
                {message:"xuan"}
            ]
        }
    });
</script>
```

#### 2.2 绑定事件

``` javascript
<div id="app">
    <!-- 通过 v-on 进行绑定-->
    <button style="width: 200px; height: 200px" v-on:click="sayHi">点我</button>
</div>

<script src="../js/vue.js"></script>
<script>
    var app = new Vue({
        el: "#app",
        data: {
            message: "Hello Vue!"
        },
        // 方法必须定义在methods中
        methods: {
            sayHi: function() {
                alert(this.message);
            }
        }
    });
</script>
```

#### 2.3 双向绑定

将 视图与数据进行绑定，一个改变将导致另一个的改变

``` javascript
<div id="app">
    输入的文本:<input type="text" v-model="message" /> {{message}}
</div>

<script src="../js/vue.js"></script>
<script>
    var app = new Vue({
        el: "#app", //  是 EL 而不是 E1
        data: {
            message: "Hello Vue!"
        }
    });
</script>
```

#### 2.4 组件

``` javascript
<div id="app">
    <!-- 通过v-for 进行遍历，通过 v-bing 进行绑定 -->
    <test v-for="item in items" v-bind:qin="item"></test>
</div>

<script src="../js/vue.js"></script>
<script>
    // 必须先注册再new Vue
    Vue.component("test", {
    	// 通过 props 访问组件的数据
        props: ['qin'],
        template: '<li>{{ qin }}</li>'
    });

    var app = new Vue({
        el: "#app",
        data: {
            items: ["java","js"]
        }
    });
</script>
```

### 3. Axios

异步通信框架，vue 推荐使用

``` javascript
<div id="vue" v-clock>
    <div>{{info.name}}</div>
    <div>{{info.version}}</div>
    <a v-bind:href="info.url">包老师的首页</a>
</div>

<script src="../js/vue.js"></script>
<script src="https://unpkg.com/axios/dist/axios.min.js"></script>
<script>
    var app = new Vue({
        el: "#vue",
        data(){
            return{
                // 请求的返回参数与 json 字符串相同
                info: {
                    name:null,
                    version: null,
                    url: null
                }
            }
        },
        // 钩子函数，链式编程
        mounted() {
            // GET 请求
            axios.get('data.json').then(response=>(this.info=response.data))
        }
    });
</script>
```

### 4. 计算属性

计算属性相比于函数会进行**缓存**

``` javascript
<div id="app">
    <p>currentTime1: {{currentTime1()}}</p>
    <p>currentTime2: {{currentTime2}}</p>
</div>

<script src="../js/vue.js"></script>
<script>
    var app = new Vue({
        el: "#app",
        data: {
            message: "Hello Vue!"
        },
        methods: {
            currentTime1: function () {
                return Date.now();
            }
        },
        // computed 中如果与 methods 里的重名，将会优先调用 methods
        computed:{
            currentTime2: function () {
                this.message; // 当 message 改动时，缓存更新
                return Date.now();
            }
        }
    });
</script>
```

### 5. Slot

