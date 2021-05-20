# 前端部分（熟悉首页需求，使用Vue实现首页功能）

##  1目录结构

前端代码在client部分，重点：

1. router文件路由（vue-router）
2. store下存放的页面状态管理文件（vuex）
3. vuetify框架
4. material design 图标库

<img src="C:\Users\dell\AppData\Roaming\Typora\typora-user-images\1621328950248.png" alt="1621328950248" style="zoom: 50%;" />

## 2 Vue-Router 路由管理器

1. npm前端安装各种包使用的，Node.js的包管理工具，用来安装各种Node.js扩展

`npm install vue-router`

2. 要通过 Vue.use() 明确地安装路由功能

```javescript
import Vue from 'vue'
import VueRouter from 'vue-router'
Vue.use(VueRouter)
```

3. 跳转
   - router-link
   - this.$router.push() query:get;params:post新增一个记录
   - this.$router.replace()替换
   - this.$router.go(n)

## 3 vuex状态管理

1. npm
2. `const store = new Vuex.Store`
   - state中count值，记录状态
   - mutations定义方法，变更状态，`state.count++`
   - actions执行，调用mutation中方法来改变。调用方式：`commit('increment')`

3. 页面中改变store变量值：`this.$store.dispatch('increment')`

## 4 vuetify

1. npm

2. ```javascript
   Vue.use(Vuetify)
   const opts ={}
   export default new Vuetify(opts)
   ```

## 5 material design

看图标：`<v-icon> 图标名 </v-icon>`

## 6 首页导航栏功能

## 7 资料

1. https://aka.ms/ApisMuseumCheckin-13083
2. API直播：http://live.bilibili.com/21704593
3. Vue文档：https://developer.mozilla.org/zh-CN/docs/Learn/Tools_and_testing/Client-side_JavaScript_frameworks/Vue_getting_started（官网：学习+示例）

