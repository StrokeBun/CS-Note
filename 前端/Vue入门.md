## Vue 入门

### 1. 介绍

Vue 针对 HTML + CSS + JS： 视图层

配合其他模块使用

网络通信：axios

页面跳转：vue-router

状态管理：vuex



Vue 基于 **MVVM** 模式，使用 **观察者** 模式实现

![avatar](img/mvvm.jpg)

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

### 3. 绑定事件

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

### 4. 双向绑定

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

