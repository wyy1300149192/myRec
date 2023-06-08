# Vue2
## vue/cli 脚手架工具

1. 安装脚手架和创建项目
   - 安装@vue/cli 脚手架工具：`npm i @vue/cli -g `
   - 创建 vue 项目：`vue create 名字`
## Vue 实例创建
```
var vm = new Vue({
  // 选项
})
```
## Vue 实例中的选项
`介绍 vue 实例中的配置`

### data
`vue数据对象`，vue 实例初始化时将 data 对象中的数据挂载在 vue 实例中，值发生改变时，模板也会匹配最新的值

### methods
`方法`methods 用于放置 js 方法

### computed
`计算属性`computed 中用于定义计算属性，本意是计算复杂的逻辑运算，与 medhods 相比，computed 对他的响应式依赖进行**缓存**，只有相关依赖发生变化才会重新调用

**完整写法**：考虑读取与修改值

```js
computed:{
    属性:{
    get(){return 属性值}
    set(value){修改执行}
    }
}
```
**简写**：只考虑读取
```js
 Compouted:{
    属性(){
        return 属性值
    }
}
```
### watch
`监视属性`**检测属性是否发生变化，发生变化则执行，也可用于获取属性变化的值**
- data: 监视的属性
- handler:监视属性发生改变时调用
- newValue:新的值
- oldValue:旧的值
  **完整写法**
  ```js
  watch:{
      data:{
  	     //配置
           handler(newValue,oldValue){
              //执行
          }
      }
  }
  ```
  **简写**：没有配置的情况下
  ```js
  watch:{
      data(newValue,oldValue){
              //执行
      }
  }
  ```
  **watch 中的常用配置**

| 配置项    | 默认值 | 值类型 | 说明                                                                                                                       |
| --------- | ------ | ------ | -------------------------------------------------------------------------------------------------------------------------- |
| handler   |        | 函数   | 监视属性发生改变时调用                                                                                                     |
| deep      | false  | 布尔   | 使 watch 深度监视：watch 默认只能检测第一级默认的数据变化，此属性将使 watch 可以检测多级数据的变化（深度监视无法获得旧值） |
| immediate | false  | 布尔   | 初始化时就让 handler 调用一次                                                                                              |
### filters

`过滤器` 用于处理简单的逻辑运算 `!vue3已弃用`

在模板中定义过滤器 `{{data | filtersName | filtersName2 }}`

- filtersNanme 是一个`函数`，需`返回`一个值

- filtersNanme 有两个`形参`

  1. data

  1. 函数形参

- `第二个过滤器`的第一个形参接收上一个过滤器的值，以此类推

**使用过滤器处理时间格式化**

```html
<body>
  <div class="box">
    <h2>现在的时间是：{{time | timeFormData }}</h2>
    <h2>现在的时间是：{{time | timeFormData("YYYY-MM-DD") }}</h2>
    <h2>现在的时间是：{{time | timeFormData | time2}}</h2>
  </div>
  <script>
    new Vue({
      el: ".box",
      data: {
        time: new Date(),
      },
      filters: {
        timeFormData(val, str = "YYYY-MM-DD HH:mm:ss") {
          return dayjs(val).format(str);
        },
        time2(val) {
          return val.slice(0, 4);
        },
      },
    });
  </script>
</body>
```

### directives

`自定义指令`用于开发者 自定义设定 操作 DOM 元素

**用法**
1. 模板标签定义`v-自定义指令`
2. directives 对象定义自定义指令函数，接收两个参数`el`:指令绑定的 dom `binding`：一个对象，可以获得 value

**什么时候会调用自定义指令函数？**

1. 初始化时
2. 指令所在的模板被重新解析时

**如果需要在特定的时间调用函数，将自定义指令写成`对象`，有以下几个钩子函数**

1. `bind() ` 指令与元素绑定时，一上来
2. `inserted() `指令所在元素插入到页面时
3. `updata() ` 指令所在模板重新解析时 4

## Vue 文件结构

vue 文件中有三个标签

1. `template` 对应 vue 模板
2. `script` 对应 js 语句
3. `style` 对应 css 样式

## 常用指令

| **指令**                   | **说明**                                                     | **语法**                        | **备注**                                                     | **书写位置**                     |
| -------------------------- | ------------------------------------------------------------ | ------------------------------- | ------------------------------------------------------------ | -------------------------------- |
| **v-bind**                 | 绑定参数，使用v-bind指令可响应式的更新html：                 | v-bind:属性:"变量"              | **简写：** :属性:"变量'                                      | 标签属性                         |
| **v-on**                   | 利用v-on绑定js监听事件                                       | v-on:事件名 = "methods里的方法" | **简写：**@事件名 = "methods里的方法"                        | 标签属性                         |
| **v-model**                | 参数的双向数据绑定，让value和变量互相绑定                    | v-model="变量"                  |                                                              | <input>、<textarea>  及 <select> |
| **v-html**      **v-text** | 相当于innerHtml和innerText，会覆盖插值表达式                 | v-html与v-text                  | **严重注意：**v-html在网站上动态渲染html是非常危险的，永远不要用在用户提交的内容上 | 标签属性                         |
| **v-show**                 | 是否隐藏元素，相当于display:none，true为显示，false为隐藏    | v-show = "变量"                 |                                                              | 标签属性                         |
| **v-if**                   | 条件为true则显示，否则隐藏,直接在页面中删除dom，可以配合v-else-if或v-else使用,中间不能被打断 | v-if="条件"                     |                                                              | 标签属性                         |
| **v-for**                  | 根据数组元素渲染一个列表                                     | v-for = "(元素,下标) in 源数组" | 遍历对象的话，两个形参为属性值和属性名    遍历时key默认是index | 标签属性                         |
| **:class**                 | 绑定class({}中均可使用变量)                                  | :class = "{类名:布尔值,…}"      |                                                              | 标签属性                         |
| **:style**                 | 绑定style({}中均可使用变量)                                  | :style= "{css属性:值,…}"        |                                                              | 标签属性                         |
| v-cloak                    | vue实例接管容器后，会删掉此属性，在实例接管之前，配合css的display：none解决html直接显示在页面的问题 | v-cloak                         |                                                              | 标签属性                         |
| v-once                     | 使所在标签的插值属性初次动态渲染后，变为静态，以后数据更新，不会影响v-once所在的标签的更新 | v-once                          |                                                              | 标签属性                         |
| v-pre                      | 跳过加了v-pre标签的vue编译过程，为不需要编译的标签跳过，节省性能 | v-pre                           |                                                              | 标签属性                         |
| ref                        | 给DOM元素起名字，方便vue操作dom元素                          | ref="name"                      | 如果放在组件标签上，得到的会是一个组件实例对象               | 标签属性                         |
| $refs                      | 获取标签属性有ref的dom节点                                   |                                 |                                                              | 组件内对象                       |

## 数据代理

![4234234234234](https://yang-cloud-img.oss-cn-beijing.aliyuncs.com/img/4234234234234.png)

## 组件实例原理

![1231231242435234](https://yang-cloud-img.oss-cn-beijing.aliyuncs.com/img/1231231242435234.png)

## 插槽

- **基本使用：**子组件**slot**标签**可以接收父组件中**组件标签中的内容
- **具名插槽：**如子组件需要有多个slot标签传输不同的内容:
  1. **slot**标签加**name属性**命名
  2. 在父组件的组件标签中使用**`<template v-solt:name>`**设置各个slot标签内容
- **作用域插槽：**主组件的组件标签需要用到子组件的属性
  1. 子组件slot标签：`<slot :名称="值">`
  2. 子组件根据定义名称和值**生成对象**
  3. 主组件template标签中`v-slot="自定名称"`(此会收到子组件传来的对象{名称:'值'})

## 生命周期

![](https://yang-cloud-img.oss-cn-beijing.aliyuncs.com/img/%E6%9C%AA%E5%91%BD%E5%90%8D%E5%9B%BE%E7%89%87.png)

# Vue3

## Vue3 介绍

**vue3.0 正式版 于 2020年 9月18日 发布**

1. **性能提升**

   - 渲染速度快 55%，更新渲染快 133%

   * 打包大小减少 41%

   * 内存减少 54%

2. **源码提升**

   - **使用 Proxy 实现响应式**

   * 重写虚拟 DOM

3. **更好支持 TS**

4. **新特性**
   
   - **组合式API Composition API ()**

## 新生命周期图示

## 组合式API

> 当项目变大时 在处理vue2的单个逻辑时，需要跳转好几个地方找相关代码，非常麻烦
>
> 所以，组合式API可将单个逻辑收集在一起便于维护

### **`setup`组件选项**

新的 `setup` 选项在**组件被创建之前**执行，是一个接收`props` 和 `context`的函数，`setup`可以返回一个对象，将对象中的数据暴露出来，可供模板使用

**注意**：在 `setup` 中你应该避免使用 `this`，因为它不会找到组件实例

### `ref`响应式变量

在 Vue3.0 中，我们可以通过一个`ref函数`设置响应式变量

```js
import { ref } from 'vue'

const num = ref(1)
```

ref接受参数，并将包裹在一个带有value的对象返回，然后可以通过此对象访问或更改值

```js
import { ref } from 'vue'

const num = ref(1)

console.log(num.value) // 1
```

### 生命周期钩子

setup中的生命周期钩子名称在选项式的名称加上前缀为`on`，如`onMounted`

钩子函数接受一个回调，当钩子函数被组件调用时，该回调将被执行

```js
import { onMounted } from 'vue'

setup() {
    onMounted(() => {
        console.log('onMounted') // 在mounted时打印'onMounted'
    })
}
```

### watch

**setup中的watch接受三个参数**

- 监视的属性
- 回调函数
- 可选配置选项

**范例**

```js
import { watch } from 'vue'

const num = ref(0)
watch(num, (newValue,oldValue) => {
    console.log('新值为' + oldValue)
})
```

### computed

和钩子函数类似，接受一个回调函数在依赖属性发生变化执行回调

```js
import { computed } from 'vue'
const num = ref(0)

const sum = computed(() => num + 100)

num.value++
console.log(sum) // 101
```



## Provide / Inject

> 对于深度嵌套的传递数据，使用props需要逐级传递，会很麻烦
>
> 这种情况，使用一对provide和inject，无论层级有多深，父组件可以给所有下级组件传递数据

父组件使用provide提供数据

子组件使用inject使用这些数据

# Router

## 基本使用

### JavaScript

```js
// 1. 定义路由组件.
// 也可以从其他文件导入
const Home = { template: '<div>Home</div>' }
const About = { template: '<div>About</div>' }

// 2. 定义一些路由
// 每个路由都需要映射到一个组件。
// 我们后面再讨论嵌套路由。
const routes = [
  { path: '/', component: Home },
  { path: '/about', component: About },
]

// 3. 创建路由实例并传递 `routes` 配置
// 你可以在这里输入更多的配置，但我们在这里
// 暂时保持简单
const router = VueRouter.createRouter({
  // 4. 内部提供了 history 模式的实现。为了简单起见，我们在这里使用 hash 模式。
  history: VueRouter.createWebHashHistory(),
  routes, // `routes: routes` 的缩写
})

// 5. 创建并挂载根实例
const app = Vue.createApp({})
//确保 _use_ 路由实例使
//整个应用支持路由。
app.use(router)

app.mount('#app')

// 现在，应用已经启动了！
```

### HTML

```vue
<script src="https://unpkg.com/vue@3"></script>
<script src="https://unpkg.com/vue-router@4"></script>

<div id="app">
  <h1>Hello App!</h1>
  <p>
    <!--使用 router-link 组件进行导航 -->
    <!--通过传递 `to` 来指定链接 -->
    <!--`<router-link>` 将呈现一个带有正确 `href` 属性的 `<a>` 标签-->
    <router-link to="/">Go to Home</router-link>
    <router-link to="/about">Go to About</router-link>
  </p>
  <!-- 路由出口 -->
  <!-- 路由匹配到的组件将渲染在这里 -->
  <router-view></router-view>
</div>
```

## 重定向

重定向也是通过 `routes` 配置来完成，下面例子是从 `/home` 重定向到 `/`：

```js
const routes = [{ path: '/home', redirect: '/' }]
```

重定向的目标也可以是一个命名的路由：

```js
const routes = [{ path: '/home', redirect: { name: 'homepage' } }]
```

## 别名

**将 `/` 别名为 `/home`，意味着当用户访问 `/home` 时，URL 仍然是 `/home`，但会被匹配为用户正在访问 `/`**

```js
const routes = [{ path: '/', component: Homepage, alias: '/home' }]
```

## 导航守卫

# uni-app

## 介绍

可以在三端同时兼容：

- web
- 手机端
- 小程序

## 运行

第一次运行需要设置微信开发者工具：设置-安全设置-服务端口：打开

否则会报错：

![image-20220920110132740](https://yang-cloud-img.oss-cn-beijing.aliyuncs.com/img/image-20220920110132740.png)

## 页面配置

### 通过globalStyle进行全局配置

`@/pages.json的globalStyle对象配置`

| 属性                         | 类型     | 默认值  | 描述                                                         | 平台差异说明                                     |
| :--------------------------- | :------- | :------ | :----------------------------------------------------------- | :----------------------------------------------- |
| navigationBarBackgroundColor | HexColor | #F7F7F7 | 导航栏背景颜色（同状态栏背景色）                             | APP与H5为#F7F7F7，小程序平台请参考相应小程序文档 |
| navigationBarTextStyle       | String   | white   | 导航栏标题颜色及状态栏前景颜色，仅支持 black/white           |                                                  |
| navigationBarTitleText       | String   |         | 导航栏标题文字内容                                           |                                                  |
| navigationStyle              | String   | default | 导航栏样式，仅支持 default/custom。custom即取消默认的原生导航栏，需看[使用注意](https://uniapp.dcloud.net.cn/collocation/pages#customnav) | 微信小程序 7.0+、百度小程序、H5、App（2.0.3+）   |
| backgroundColor              | HexColor | #ffffff | 下拉显示出来的窗口的背景色                                   | 微信小程序                                       |
| backgroundTextStyle          | String   | dark    | 下拉 loading 的样式，仅支持 dark / light                     | 微信小程序                                       |
| enablePullDownRefresh        | Boolean  | false   | 是否开启下拉刷新，详见[页面生命周期](https://uniapp.dcloud.net.cn/tutorial/page.html#lifecycle)。 |                                                  |
| onReachBottomDistance        | Number   | 50      | 页面上拉触底事件触发时距页面底部距离，单位只支持px，详见[页面生命周期](https://uniapp.dcloud.net.cn/tutorial/page.html#lifecycle) |                                                  |
| backgroundColorTop           | HexColor | #ffffff | 顶部窗口的背景色（bounce回弹区域）                           | 仅 iOS 平台                                      |
| backgroundColorBottom        | HexColor | #ffffff | 底部窗口的背景色（bounce回弹区域）                           | 仅 iOS 平台                                      |

### pages注册页面

需要在`pages.json`的pages数组添加新建的页面

- `pages`数组中第一项表示应用启动页

创建了message页面并设置为启动页：

```json
"pages": [ //pages数组中第一项表示应用启动页
	{
		"path": "pages/index/message",
		"style": {
			"navigationBarTitleText": "信息"
		}
	},
	{
		"path": "pages/index/index",
		"style": {
			"navigationBarTitleText": "我的app"
		}
	}
],
```

### tabBar

- 当设置 position 为 top 时，将不会显示 icon
- tabBar 中的 list 是一个数组，只能配置最少2个、最多5个 tab，tab 按数组的顺序排序。
- tabbar 切换第一次加载时可能渲染不及时，可以在每个tabbar页面的onLoad生命周期里先弹出一个等待雪花（hello uni-app使用了此方式）
- tabbar 的页面展现过一次后就保留在内存中，再次切换 tabbar 页面，只会触发每个页面的onShow，不会再触发onLoad。
- 顶部的 tabbar 目前仅微信小程序上支持。需要用到顶部选项卡的话，建议不使用 tabbar 的顶部设置，而是自己做顶部选项卡，可参考 hello uni-app->模板->顶部选项卡。

**属性说明：**

| 属性            | 类型     | 必填 | 默认值 | 描述                                                         | 平台差异说明                                         |
| :-------------- | :------- | :--- | :----- | :----------------------------------------------------------- | :--------------------------------------------------- |
| color           | HexColor | 是   |        | tab 上的文字默认颜色                                         |                                                      |
| selectedColor   | HexColor | 是   |        | tab 上的文字选中时的颜色                                     |                                                      |
| backgroundColor | HexColor | 是   |        | tab 的背景色                                                 |                                                      |
| borderStyle     | String   | 否   | black  | tabbar 上边框的颜色，可选值 black/white，也支持其他颜色值    | App 2.3.4+ 、H5 3.0.0+                               |
| blurEffect      | String   | 否   | none   | iOS 高斯模糊效果，可选值 dark/extralight/light/none（参考:[使用说明 (opens new window)](https://ask.dcloud.net.cn/article/36617)） | App 2.4.0+ 支持、H5 3.0.0+（只有最新版浏览器才支持） |
| list            | Array    | 是   |        | tab 的列表，详见 list 属性说明，最少2个、最多5个 tab         |                                                      |
| position        | String   | 否   | bottom | 可选值 bottom、top                                           | top 值仅微信小程序支持                               |
| fontSize        | String   | 否   | 10px   | 文字默认大小                                                 | App 2.3.4+、H5 3.0.0+                                |
| iconWidth       | String   | 否   | 24px   | 图标默认宽度（高度等比例缩放）                               | App 2.3.4+、H5 3.0.0+                                |
| spacing         | String   | 否   | 3px    | 图标和文字的间距                                             | App 2.3.4+、H5 3.0.0+                                |
| height          | String   | 否   | 50px   | tabBar 默认高度                                              | App 2.3.4+、H5 3.0.0+                                |
| midButton       | Object   | 否   |        | 中间按钮 仅在 list 项为偶数时有效                            | App 2.3.4+、H5 3.0.0+                                |

其中 list 接收一个数组，数组中的每个项都是一个对象，其属性值如下：

| 属性             | 类型    | 必填 | 说明                                                         | 平台差异                    |
| :--------------- | :------ | :--- | :----------------------------------------------------------- | :-------------------------- |
| pagePath         | String  | 是   | 页面路径，必须在 pages 中先定义                              |                             |
| text             | String  | 是   | tab 上按钮文字，在 App 和 H5 平台为非必填。例如中间可放一个没有文字的+号图标 |                             |
| iconPath         | String  | 否   | 图片路径，icon 大小限制为40kb，建议尺寸为 81px * 81px，当 position 为 top 时，此参数无效，不支持网络图片，不支持字体图标 |                             |
| selectedIconPath | String  | 否   | 选中时的图片路径，icon 大小限制为40kb，建议尺寸为 81px * 81px ，当 position 为 top 时，此参数无效 |                             |
| visible          | Boolean | 否   | 该项是否显示，默认显示                                       | App (3.2.10+)、H5 (3.2.10)+ |
| iconfont         | Object  | 否   | 字体图标，优先级高于 iconPath                                | App（3.4.4+）               |

**midButton 属性说明**

| 属性            | 类型   | 必填 | 默认值 | 描述                                                         |
| :-------------- | :----- | :--- | :----- | :----------------------------------------------------------- |
| width           | String | 否   | 80px   | 中间按钮的宽度，tabBar 其它项为减去此宽度后平分，默认值为与其它项平分宽度 |
| height          | String | 否   | 50px   | 中间按钮的高度，可以大于 tabBar 高度，达到中间凸起的效果     |
| text            | String | 否   |        | 中间按钮的文字                                               |
| iconPath        | String | 否   |        | 中间按钮的图片路径                                           |
| iconWidth       | String | 否   | 24px   | 图片宽度（高度等比例缩放）                                   |
| backgroundImage | String | 否   |        | 中间按钮的背景图片路径                                       |
| iconfont        | Object | 否   |        | 字体图标，优先级高于 iconPath                                |

	
