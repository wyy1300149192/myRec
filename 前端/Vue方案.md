# Vue2案例

## 解决跨域问题

### vue-cil 代理服务器

1. **修改配置文件.env.development**

```javascript
//.env.development 中的VUE_APP_BASE_API改成 /api

# just a flag
ENV = 'development'

# base api
VUE_APP_BASE_API = '/api'

```

2. **vue-config 配置*module*._exports_**

```javascript
devServer: {
    proxy: {
      // 这里的api 表示如果我们的请求地址有/api的时候,就出触发代理机制
      '/api': {
        target: '', // 我们要代理请求的地址
        changeOrigin: true //是否跨域
      }
    }
  },
```

## 登录模块设计流程

1. 设计**表单、登录按钮**等布局
2. 表单绑定 data 数据对象
3. 表单校验
4. 处理登录按钮事件，点击登录先进行兜底校验（可封装在 utils 中的 validate.js 中），校验通过再执行下面的：
5. 根据接口文档写好请求：写封装在 api 文件夹的 js 中
6. 随后发送请求登录（判断是否有数据需要存入 vuex，有则使用 vuex 请求）
7. 将 token 存入并持久化，持久化可使用 cookies
8. 跳转至登录后页面

## 退出登录流程

1. store 设置好 actions：
   - 清除用户信息，
   - 清除 token，
   - 跳转登录页，
   - 提示
2. 设置`退出登录`按钮事件，执行上面的 actions

## 登录未遂设置

**退出登录时**

1. 退出登录时跳转并携带当前路由

   ```js
   // 跳转并携带退出前路由
   // ?后面的redirect名字为自己定义的，=后面的参数会存到route.query中
   this.$router.replace(
     `/login?redirect=${encodeURIComponent(this.$route.fullPath)}`
   );
   ```

2. 在登录跳转判断 route.query.replace 有路径则跳转至这里，否则跳转至首页

   ```js
   // 跳转 ：判断有携带地址则跳转携带地址，否则跳转首页
   this.$router.push(this.$route.query.redirect || "/");
   ```

**token 过期退出登录时**

1. token 过期退出登录时携带当前路由

   ```js
   // 跳转并携带当前页面路径
   router.replace(
     `/login?redirect=${encodeURIComponent(router.currentRoute.fullPath)}`
   );
   ```

2. 在登录跳转判断 route.query.replace 有路径则跳转至这里，否则跳转至首页

   ```js
   // 跳转 ：判断有携带地址则跳转携带地址，否则跳转首页
   this.$router.push(this.$route.query.redirect || "/");
   ```

## 让 axios 少一层 data

```JavaScript
// 响应拦截器
instance.interceptors.response.use(function(response) {
  // 对响应数据做点什么
  const res = response.data
  return res
}, function(error) {
  // 对响应错误做点什么
  return Promise.reject(error)
})
```

## 让路由跳转有进度条

**使用 nprogress**：

**安装**

```
npm install --save nprogress
```

直接引入 js、css 或者通过 cdn 引入。

```html
<script src="nprogress.js"></script>
<link rel="stylesheet" href="nprogress.css" />
```

**使用**

直接调用 `start()`或者`done()`来控制进度条。

```javascript
NProgress.start();
NProgress.done();
```

可以通过调用 `.set(n)`来设置进度，n 是 0-1 的数字。

```javascript
NProgress.set(0.0); // Sorta same as .start()
NProgress.set(0.4);
NProgress.set(1.0); // Sorta same as .done()
```

## 页面 token 判断

**跳转路由时判断 token 是否存在**：

1. 将请求获得得 token 存入 state 中，并存入 Cookies 中，将 state 中的值设置为 cookies 存入的函数，否则刷新页面则会失效
2. 判断如果有 token：访问的是 login 页面则直接跳转到主页，不是登录也则放行

3. 如果没有 token：判断是白名单的直接放行，否则跳转至注册页

```javascript
// 路由白名单
const whiteList = ["/login"];
// 路由前置守卫
router.beforeEach(async (to, from, next) => {
  // 进度条开始
  NProgress.start();
  // 获取vuex中getters的token
  const token = store.getters.token;
  // 判断是否有token
  if (token) {
    // 有token访问的是login
    if (to.path === "/login") {
      // 跳转至主页
      next("/");
      NProgress.done();
    } else {
      // 放行
      next();
      NProgress.done();
    }
  } else {
    // 如果没有token
    // 判断访问的如果是白名单的地址
    if (whiteList.some((item) => item === to.path)) {
      // 放行
      next();
      NProgress.done();
    } else {
      // 跳转至登录页
      next("/login");
      NProgress.done();
    }
  }
});

// 路由后置守卫
router.afterEach(() => {
  // 进度条结束
  NProgress.done();
});
```

**跳转路由时判断 token 是否存在**：

1. 在响应拦截器中错误中判断错误 code 是否为 token 过期

2. 如果过期：

   - 删除 token
   - 删除用户信息
   - 跳转并携带当前页面路径

   ```js
   // 响应拦截器
   service.interceptors.response.use(
     (response) => {
       return response.data;
     },
     (error) => {
       // 如果过期：删除token 删除用户信息 跳转并携带当前页面路径
       if (
         error.response &&
         error.response.data &&
         error.response.data.code === 10002
       ) {
         // 提醒
         Message.error("登录过期，请重新登录");
         // 删除token
         store.commit("user/REMOVE_TOKEN");
         // 删除用户信息
         store.commit("user/RESET_STATE");
         // 跳转并携带当前页面路径
         router.replace(
           `/login?redirect=${encodeURIComponent(router.currentRoute.fullPath)}`
         );
       } else {
         Message.error({ message: error || "出现错误，请稍后再试" });
       }
       console.dir(error);
       return Promise.reject(error);
     }
   );
   ```

## 路由前置统一设置请求头 token

1. 将 vuex 中 state 的 token 放在 getters 中

2. 请求拦截器设置：

   ```
   // 请求拦截器
   service.interceptors.request.use(
     (config) => {
       const token = store.getters.token
       if (token) {
         config.headers.Authorization = `Bearer ${token}`
       }
       return config
     },
     (error) => {
       return Promise.reject(error)
     }
   )
   ```

## 导航栏 logo 控制

1. 是否显示设置在 src 下的 settings.js 中
2. 存入 vuex 中的 state 中
3. 在 logo 组件的 v-if 等于 vuex 中的 state

## 获取用户信息

1. 封装获取用户请求
2. 在前置路由守卫调用请求，存入 vuex 中

## 全局组件统一封装

如果希望在所有页面都要用, 那么就要`全局注册`组件，如果有很多个组件, 都要全局注册, 那么又会在 main.js 中写很多内容

1. Vue.use 可以接收一个对象, Vue.use(obj)
2. 对象中需要提供一个 install 函数
3. install 函数可以拿到参数 Vue, 且将来会在 Vue.use 时, 自动调用该 install 函数

index.js 统一引入组件

```js
import PageTools from "./PageTools";

export default {
  install(Vue) {
    Vue.component("PageTools", PageTools);
  },
};
```

入口文件注册插件（main.js）

```js
import Components from "./components";
Vue.use(Components);
```

## 封装过滤器

1. src 目录下创建`filters/index.js`文件

   ```js
   // 导入dayjs
   import dayjs from "dayjs";
   // 格式化组件过滤器
   export function formatData(value, str = "YYYY年MM月DD日") {
     return dayjs(value).format(str);
   }
   ```

2. 入口文件`main.js`全局注册过滤器

   ```js
   // 引入过滤器
   import * as filters from "@/filters";
   Object.keys(filters).forEach((key) => {
     // 注册过滤器
     Vue.filter(key, filters[key]);
   });
   ```

## Excel 导入

1. **封装 excel 导入组件(uploadExcel)**

   需要的依赖：`xlsx`

   ```vue
   <template>
     <div>
       <input
         ref="excel-upload-input"
         class="excel-upload-input"
         type="file"
         accept=".xlsx, .xls"
         @change="handleClick"
       />
       <div class="box">
         <div class="left">
           <el-button
             :loading="loading"
             style="margin-left: 16px"
             size="mini"
             type="primary"
             @click="handleUpload"
           >
             点击上传
           </el-button>
           <div class="text">(推荐下载模板文件，填写后上传)</div>
           <div class="text">点击查看文件上传要求</div>
         </div>
         <div
           class="rigth"
           @drop="handleDrop"
           @dragover="handleDragover"
           @dragenter="handleDragover"
           @click="handleUpload"
         >
           <div class="el-icon-upload" />
           <div>将文件拖入上传</div>
         </div>
       </div>
     </div>
   </template>

   <script>
   import * as XLSX from "xlsx";

   export default {
     props: {
       beforeUpload: Function, // eslint-disable-line
       onSuccess: Function, // eslint-disable-line
     },
     data() {
       return {
         loading: false,
         excelData: {
           header: null,
           results: null,
         },
       };
     },
     methods: {
       generateData({ header, results }) {
         this.excelData.header = header;
         this.excelData.results = results;
         this.onSuccess && this.onSuccess(this.excelData);
       },
       handleDrop(e) {
         e.stopPropagation();
         e.preventDefault();
         if (this.loading) return;
         const files = e.dataTransfer.files;
         if (files.length !== 1) {
           this.$message.error("Only support uploading one file!");
           return;
         }
         const rawFile = files[0]; // only use files[0]

         if (!this.isExcel(rawFile)) {
           this.$message.error(
             "Only supports upload .xlsx, .xls, .csv suffix files"
           );
           return false;
         }
         this.upload(rawFile);
         e.stopPropagation();
         e.preventDefault();
       },
       handleDragover(e) {
         e.stopPropagation();
         e.preventDefault();
         e.dataTransfer.dropEffect = "copy";
       },
       handleUpload() {
         this.$refs["excel-upload-input"].click();
       },
       handleClick(e) {
         const files = e.target.files;
         const rawFile = files[0]; // only use files[0]
         if (!rawFile) return;
         this.upload(rawFile);
       },
       upload(rawFile) {
         this.$refs["excel-upload-input"].value = null; // fix can't select the same excel

         if (!this.beforeUpload) {
           this.readerData(rawFile);
           return;
         }
         const before = this.beforeUpload(rawFile);
         if (before) {
           this.readerData(rawFile);
         }
       },
       readerData(rawFile) {
         this.loading = true;
         return new Promise((resolve, reject) => {
           const reader = new FileReader();
           reader.onload = (e) => {
             const data = e.target.result;
             const workbook = XLSX.read(data, { type: "array" });
             const firstSheetName = workbook.SheetNames[0];
             const worksheet = workbook.Sheets[firstSheetName];
             const header = this.getHeaderRow(worksheet);
             const results = XLSX.utils.sheet_to_json(worksheet);
             this.generateData({ header, results });
             this.loading = false;
             resolve();
           };
           reader.readAsArrayBuffer(rawFile);
         });
       },
       getHeaderRow(sheet) {
         const headers = [];
         const range = XLSX.utils.decode_range(sheet["!ref"]);
         let C;
         const R = range.s.r;
         /* start in the first row */
         for (C = range.s.c; C <= range.e.c; ++C) {
           /* walk every column in the range */
           const cell = sheet[XLSX.utils.encode_cell({ c: C, r: R })];
           /* find the cell in the first row */
           let hdr = "UNKNOWN " + C; // <-- replace with your desired default
           if (cell && cell.t) hdr = XLSX.utils.format_cell(cell);
           headers.push(hdr);
         }
         return headers;
       },
       isExcel(file) {
         return /\.(xlsx|xls|csv)$/.test(file.name);
       },
     },
   };
   </script>

   <style scoped>
   .box {
     text-align: center;
   }
   .excel-upload-input {
     display: none;
     z-index: -9999;
   }
   .rigth {
     border: 2px dashed #bbb;
     width: 400px;
     height: 180px;
     font-size: 24px;
     border-radius: 5px;
     text-align: center;
     color: #bbb;
     display: inline-block;
     border-left: 0;
   }

   .left {
     vertical-align: top;
     padding: 50px;
     border: 2px dashed #bbb;
     width: 400px;
     height: 180px;
     font-size: 24px;
     border-radius: 5px;
     text-align: center;
     color: #bbb;
     display: inline-block;
   }
   .el-icon-upload {
     font-size: 100px;
   }
   .left .text {
     font-size: 12px;
     color: #000;
     margin-top: 5px;
   }
   </style>
   ```

2. **上述组件接受两个函数**

   **beforeUpload** 导入前的回调 有一个实参: 文件对象

   - beforeUpload 需要**返回 true**才可继续导入成功的回调

   **onSuccess** 导入成功后的回调 有一个实参：一个对象，对象有两个属性`results`, `header`\*\*

   - results 代表 **表体数据**
   - header 代表 **表头数据**

   ```vue
   <template>
     <div class="app-container">
       <el-card>
         <div class="title">员工导入</div>
         <el-alert
           type="warning"
           :closable="false"
           show-icon
           title="每次导入仅可添加1000名员工，姓名、手机、入职时间、聘用形式为必填项"
         />
         <div class="import">
           <!-- 导入组件 -->
           <upload-excel-component
             :on-success="handleSuccess"
             :before-upload="beforeUpload"
           />
         </div>
       		</el-card>
     </div>
   </template>

   <script>
   import UploadExcelComponent from "@/components/UploadExcel/index.vue";

   export default {
     name: "UploadExcel",
     components: { UploadExcelComponent },
     data() {
       return {};
     },
     methods: {
       // 导入成功回调
       handleSuccess({ results, header }) {
         console.log(results);
         console.log(header);
       },
       // 导入前的回调，可用于判断文件大小、类型等
       beforeUpload(file) {
         console.log(file);
       },
     },
   };
   </script>

   <style lang="scss" scoped>
   .title {
     text-align: center;
     font-size: 20px;
     font-weight: 900;
     margin-bottom: 15px;
   }
   .import {
     margin-top: 100px;
   }
   .app-container {
     background-color: #f0f2f4;
   }
   </style>
   ```

3. 最后 header 获取的是表头的数组

   ```js
   ["入职日期", "手机号", "姓名", "转正日期", "工号"];
   ```

4. results 获取的表体 每一行一个对象的数组

   ```js
   [{…}, {…}]
   ```

5. Excel 日期格式化

   ```javascript
    formatDate(numb, format) {
         const time = new Date((numb - 25567) * 24 * 3600000 - 5 * 60 * 1000 - 43 * 1000 - 24 * 3600000 - 8 * 3600000)
         const year = time.getFullYear() + ''
         const month = time.getMonth() + 1 + ''
         const date = time.getDate() + ''
         if (format && format.length === 1) {
           return year + format + (month < 10 ? '0' + month : month) + format + (date < 10 ? '0' + date : date)
         }
         return year + (month < 10 ? '0' + month : month) + (date < 10 ? '0' + date : date)
       }
   ```

# Vue3案例

## 目录解析

![01](https://yang-cloud-img.oss-cn-beijing.aliyuncs.com/img/01.png)

## Vuex持久化

利用`vuex-persistedstate`插件实现vuex数据持久化，将数据存入localStorage中

**实现步骤**

1. **安装**

   ```cmd
   npm i vuex-persistedstate
   ```

2. **vuex的plugins配置中使用createPersistedstate函数**

   * key： localStorage键名

   * paths： 需要存入state中的那些数据

   ```js
   import { createStore } from 'vuex'
   import createPersistedstate from 'vuex-persistedstate'
   
   export default createStore({
     plugins: [
       createPersistedstate({
         // key localStorage键名
         key: 'erabbit-client-pc-store',
         // paths 需要存入state中的那些数据
         paths: ['user', 'cart']
       })
     ]
   })
   ```

## Logger Plugin(vue3的vuex调试)

在vue3中vue2中的调试工具已不支持当前查看，需要我们使用`Logger Plugin`调试

Logger Plugin为vuex内置的一个模块，只需要引入并且注册为插件即可

```js
import { createStore, createLogger } from 'vuex'
export default createStore({
  plugins: [
    createLogger()
  ]
})
```

安装好这个插件后，每次触发action函数和mutation函数，都自动在控制台打印出提交记录与详细信息，包括`名称` `参数` `修改前后的state数据`

![image-20220801175511077](https://yang-cloud-img.oss-cn-beijing.aliyuncs.com/img/image-20220801175511077.png)

## style的自动化导入

> 使用vue-cli的style-resoures-loader 插件完成自动注入到每个vue组件的style标签，避免在每个样式文件中手动的@import导入

1） 执行命令`vue add style-resources-loader`

2）安装完成后`vue.config.js`中自动添加配置

```js
module.exports = {
  pluginOptions: {
    'style-resources-loader': {
      preProcessor: 'less',
      patterns: []
    }
  }
}
```

2）在patterns数组中配置需要导入的style

```js
patterns: [
        // 配置哪些文件需要自动导入
path.join(__dirname,'./src/styles/variables.less')
      ]
```



## 全局组件封装

components/index.js 统一注册组件

```js
// 引入组件
import Skeleton from './Skeleton'

// 导出install方法 注册组件
export default {
  install (app) {
    app.component(Skeleton.name, Skeleton)
  }
}
```

main.js 入口文件`vue实例`应用组件

```js
import componentPlugin from '@/components'
// use应用组件
createApp(App).use(store).use(router).use(componentPlugin).mount('#app')
```

## 全局自定义指令封装

`@/directives/index.js`

```js
export default {
  install (app) {
    app.directive('指令名', {
      // el为绑定该指令的dom
      // binding可以获得该指令的值
      mounted (el, binding) {
		// 执行
      }
    })
  }
}

```

vue注册

```js
import directivePlugin from '@/directives'

createApp(App).use(directivePlugin)
```



## vueuse/core工具库

> VueUse是基于[Composion API](https://v3.vuejs.org/guide/composition-api-introduction.html)的实用程序函数的集合。

**安装**

```cmd
npm i @vueuse/core
```

### 监听滚动获得滚动距离

利用`useWindowScroll`方法监听滚动距离

```html
<script>
import HeaderNav from './header-nav'
import { useWindowScroll } from '@vueuse/core'
export default {
  name: 'AppHeaderSticky',
  components: { HeaderNav },
  setup () {
    // y表示具体顶部的滚动距离 会动态更新
    const { y } = useWindowScroll()
    return { y }
  }
}
</script>
```

### ⭐组件数据懒加载

> 电商项目的核心优化手段：组件数据懒加载
>
> 说明：电商项目大多数据很多，默认就加载一个页面的所有数据，必然会发送很多不需要的请求
>
> 解决方案：`等所在模块进入 可视区 再获取数据`

**方案**：

我们可以使用 `@vueuse/core` 中的 `useIntersectionObserver`方法 来实现监听组件进入可视区域行为

**实现**：

封装公共方法

判断处于可视区域时才执行请求函数

利用stop函数避免重复执行请求函数

```js
import { ref } from 'vue'
import { useIntersectionObserver } from '@vueuse/core'

export function useObserver (apiFunc) {
  // target为绑定的dom节点
  const target = ref(null)
  const { stop } = useIntersectionObserver(
    target,
    // isIntersecting为是否处于可视区域 返回一个布尔值
    ([{ isIntersecting }], observerElement) => {
      // 判断如果处于可视区域执行api函数
      if (isIntersecting) {
        apiFunc()
        // 停止监听
        stop()
      }
    }
  )
  // 返回一个target
  return target
}

```

### ⭐图片懒加载

> 电商类项目图片非常多，可以做一下图片的懒加载

**方案**：利用自定义指令与vueuse/core工具库 监测在可视距离后再给img的src赋值

```js
import { useIntersectionObserver } from '@vueuse/core'

export default {
  install (app) {
    //定义自定义指令名 imgLazy
    app.directive('imgLazy', {
      mounted (el, binding) {
        const { stop } = useIntersectionObserver(
          el,
          ([{ isIntersecting }], observerElement) => {
            if (isIntersecting) {
              // 监测在可视区域后再将img标签src赋值
              el.src = binding.value
              stop()
            }
          }
        )
      }
    })
  }
}
```

**使用**

```html
<img v-imgLazy='item.picture' alt="">
```



## 骨架组件业务使用

> 当页面数据还未获取到时，利用骨架组件进行占位，提升用户体验

1）定义公共骨架组件

组件接收三个值： 宽 、高 、背景色

```vue
<template>
  <!-- 决定组件的宽高 -->
  <div
    class="xtx-skeleton shan"
    :style="{ width: width + 'px', height: height + 'px' }"
  >
    <!-- 决定组件的背景色 -->
    <div class="block" :style="{ backgroundColor: bg }"></div>
  </div>
</template>
<script>
export default {
  name: 'XtxSkeleton',
  props: {
    // 宽度定制
    width: {
      type: Number,
      default: 100
    },
    // 高度定制
    height: {
      type: Number,
      default: 60
    },
    // 背景颜色定制
    bg: {
      type: String,
      default: '#ccc'
    }
  }
}
</script>
<style scoped lang="less">
.xtx-skeleton {
  display: inline-block;
  position: relative;
  overflow: hidden;
  vertical-align: middle;
  .block {
    width: 100%;
    height: 100%;
    border-radius: 2px;
  }
}
.shan {
  &::after {
    content: "";
    position: absolute;
    animation: shan 1.5s ease 0s infinite;
    top: 0;
    width: 50%;
    height: 100%;
    background: linear-gradient(
      to left,
      rgba(255, 255, 255, 0) 0,
      rgba(255, 255, 255, 0.3) 50%,
      rgba(255, 255, 255, 0) 100%
    );
    transform: skewX(-45deg);
  }
}
@keyframes shan {
  0% {
    left: -100%;
  }
  100% {
    left: 120%;
  }
}
</style>

```

2）在数据处使用v-if进行判断数据未请求到时展示骨架

```vue
<template v-if="list.length > 0">
      <li v-for="item in list" :key="item.id">
        <RouterLink :to="'/category/' + item.id">{{ item.name }}</RouterLink>
      </li>
</template>
    
<template v-else>
      <li v-for="i in 9" :key="i">
        <Skeleton
          :width="60"
          :height="32"
          style="margin-right: 5px"
          bg="rgba(0,0,0,0.5)"
        />
      </li>
```

## 面包屑组件封装

**封装两个组件**：面包屑组件与面包屑项目组件

`面包屑组件`

- 接受一个值：面包屑的分隔符
- 有一个默认插槽
- 将分隔符传出，可供项目组件使用

```vue
<template>
  <div class="xtx-bread">
    <slot />
  </div>
</template>

<script>
// 分隔符数据是位于Bread组件中 而对于分隔符数据的使用是在底层的组件中使用
// provide/inject
import { provide } from 'vue'
export default {
  name: 'XtxBread',
  props: {
    separator: {
      type: String,
      default: ''
    }
  },
  setup (props) {
    // 为底层组件提供数据
    provide('separator', props.separator)
  }
}
</script>

<style scoped lang="less">
.xtx-bread {
  display: flex;
  padding: 25px 10px;
  &-item {
    a {
      color: #666;
      transition: all 0.4s;
      &:hover {
        color: @xtxColor;
      }
    }
  }
  i {
    font-size: 12px;
    margin-left: 5px;
    margin-right: 5px;
    line-height: 22px;
  }
}
</style>
```

`面包屑项目组件`

- 接收一个值：to 用作路由跳转

- 有一个默认插槽放文字
- 判断如果to存在渲染路由链接标签，否则渲染一个span标签
- 接收面包屑组件的分隔符，如没有用默认的

```vue
<template>
  <div class="xtx-bread-item">
    <!--
      如果to存在 有值 我们就渲染一个router-link标签
      如果to不存在  那就渲染一个span标签
     -->
    <router-link v-if="to" :to="to"><slot /></router-link>
    <span v-else><slot /></span>
    <!-- 分隔符 -->
    <i v-if="separator">{{ separator }}</i>
    <i v-else class="iconfont icon-angle-right"></i>
  </div>
</template>

<script>
import { inject } from 'vue'
export default {
  name: 'XtxBreadItem',
  props: {
    to: {
      type: String
    }
  },
  setup () {
    const separator = inject('separator')
    return {
      separator
    }
  }
}
</script>
<style lang="less" scoped>
.xtx-bread-item {
  i {
    margin: 0 6px;
    font-size: 10px;
  }
  // 最后一个i隐藏
  &:nth-last-of-type(1) {
    i {
      display: none;
    }
  }
}
</style>

```

**应用**：

```vue
<XtxBread>
  <XtxBreadItem to="/">首页</XtxBreadItem>
  <XtxBreadItem>美食</XtxBreadItem>
</XtxBread>
```

## ⭐电商`sku`组件

> - SPU（Standard Product Unit）：标准化产品单元。（IphoneX）   Iphone12
>
>   是商品信息聚合的最小单位，是一组可复用、易检索的标准化信息的  **`集合`**，该集合描述了一个产品的特性。
>
>   通俗点讲，属性值、特性相同的商品就可以称为一个SPU。(广义，宽泛)
>
> - SKU（Stock Keeping Unit）**`库存 量 单位`**。（IphoneX  蓝色红色粉色黑色，64g 128g 256g  =>  4 * 3  =>  12种SKU）
>
>   即库存进出计量的单位， 可以是以件、盒、托盘等为单位。
>
>   **SKU是物理上不可分割的最小存货单元。**在使用时要根据不同业态，不同管理模式来处理。

### 结构

**1) 封装goods-sku.vue**

```vue
<template>
  <div class="goods-sku">
    <!-- 使用无序列表布局商品属性 -->
    <dl>
      <dt>颜色</dt>
      <dd>
        <img
          class="selected"
          src="https://yanxuan-item.nosdn.127.net/d77c1f9347d06565a05e606bd4f949e0.png"
          alt=""
        />
        <span class="disabled">10英寸</span>
      </dd>
    </dl>
  </div>
</template>
<script>
export default {
  name: 'GoodsSku'
}
</script>
<style scoped lang="less">
.sku-state-mixin () {
  border: 1px solid #e4e4e4;
  margin-right: 10px;
  cursor: pointer;
  &.selected {
    border-color: @xtxColor;
  }
  &.disabled {
    opacity: 0.6;
    border-style: dashed;
    cursor: not-allowed;
  }
}
.goods-sku {
  padding-left: 10px;
  padding-top: 20px;
  dl {
    display: flex;
    padding-bottom: 20px;
    align-items: center;
    dt {
      width: 50px;
      color: #999;
    }
    dd {
      flex: 1;
      color: #666;
      > img {
        width: 50px;
        height: 50px;
        .sku-state-mixin ();
      }
      > span {
        display: inline-block;
        height: 30px;
        line-height: 28px;
        padding: 0 20px;
        .sku-state-mixin ();
      }
    }
  }
}
</style>
```

**2）父组件引入`sku`组件并将商品信息传给`sku`组件**

```vue
<GoodsSku :goods="goods"></GoodsSku>
```

**3）`sku`组件接收goods对象**

```js
export default {
  name: 'GoodsSku',
  // 接收goods对象
  props: {
    goods: {
      type: Object,
      default: () => {}
    }
  }
}
```

goods对象主要用到其中两个数组`skus` `specs`

`skus`说明

- `skus`对应商品各个选项组合对应的id 包含数量、价格等信息
- `结构 `：skus数组>id对象>商品规格数组

<img src="https://yang-cloud-img.oss-cn-beijing.aliyuncs.com/img/Snipaste_2022-08-03_20-24-15.jpg" alt="Snipaste_2022-08-03_20-24-15" style="zoom: 50%;" />

`specs`说明

- `specs`对应各个商品选项类别和其选项 包含：选项类别名、选项名、选项图片
- `结构`：specs数组>选项类别对象>选项数组

<img src="https://yang-cloud-img.oss-cn-beijing.aliyuncs.com/img/image-20220803203101943.png" alt="image-20220803203101943" style="zoom: 50%;" />

**4）利用`skus`和`specs`渲染商品选项**

```vue
<template>
  <div class="goods-sku">
    <!-- 使用无序列表布局商品属性 -->
    <!-- 遍历选项类别：有几个类别创建几个 -->
    <dl v-for="spec in goods.specs" :key="spec.id">
      <!-- 选项类别名 -->
      <dt>{{ spec.name }}</dt>
      <dd>
        <!-- 遍历选项：有几个选项创建几个 -->
        <template v-for="option in spec.values" :key="option.name">
          <!-- 判断有图片路径显示img，否则显示sapn -->
          <img
            v-if="option.picture"
            class="selected"
            :src="option.picture"
            :alt="option.name"
            :title="option.name"
          />
          <span v-else>{{ option.name }}</span>
        </template>
      </dd>
    </dl>
  </div>
</template>
```

**5）商品选项点击事件**

1. 模板`img`与`span`选项标签设定点击事件`@click="optionClick(spec.values,option)"` 传入`选项类别对象`和`当前选项数组`

2. 设置点击事件：在当前数组选项中加个selected属性，并在模板`img`与`span`选项标签使用`:class="{ selected: option.selected }"`判断是否选中

   ```js
   const optionClick = (values, option) => {
     // 遍历所有选项类别，排除选中
     values.forEach(item => {
       item.selected = false
     })
     //  当前点击取反
     if (option.selected) {
       option.selected = false
     } else {
       option.selected = true
     }
   }
   ```

### 处理商品选项`禁用状态`

   **1）根据后台获取的`skus`数据生成`路径字典`**

   ​				skus数据：

   <img src="https://yang-cloud-img.oss-cn-beijing.aliyuncs.com/img/image-20220804184505352.png" alt="image-20220804184505352" style="zoom:50%;" />

​	

2）首先需要有一个工具求集合的所有子集:`幂集算法封装`  `接收`一个数组 `返回`处理好的数组，包裹着各个子集数组[[],[],[]]

```js
/**
 * Find power-set of a set using BITWISE approach.
 *
 * @param {*[]} originalSet
 * @return {*[][]}
 */
export default function bwPowerSet(originalSet) {
  const subSets = [];

  // We will have 2^n possible combinations (where n is a length of original set).
  // It is because for every element of original set we will decide whether to include
  // it or not (2 options for each set element).
  const numberOfCombinations = 2 ** originalSet.length;

  // Each number in binary representation in a range from 0 to 2^n does exactly what we need:
  // it shows by its bits (0 or 1) whether to include related element from the set or not.
  // For example, for the set {1, 2, 3} the binary number of 0b010 would mean that we need to
  // include only "2" to the current set.
  for (
    let combinationIndex = 0;
    combinationIndex < numberOfCombinations;
    combinationIndex += 1
  ) {
    const subSet = [];

    for (
      let setElementIndex = 0;
      setElementIndex < originalSet.length;
      setElementIndex += 1
    ) {
      // Decide whether we need to include current element into the subset or not.
      if (combinationIndex & (1 << setElementIndex)) {
        subSet.push(originalSet[setElementIndex]);
      }
    }

    // Add current subset to the list of all subsets.
    subSets.push(subSet);
  }

  return subSets;
}
```

**3）声明生成路径字典的函数 `接收`一个skus `返回`处理好的字典对象**

```js
// 根据skus建立路径字典 选项：[id]
function handleSku (skus) {
  // 创建一个数组用于储存
  const obj = {}
  // 遍历skus
  skus.forEach(sku => {
    // 判断库存是否为0
    if (sku.inventory) {
      // 获取当前id的所有选项保存为一个数组
      const options = sku.specs.map(option => option.valueName)
      // 使用幂集算法求所有子集
      const skuArr = getPowerSet(options)
      // 遍历所有子集
      skuArr.forEach(item => {
        // 将数组使用*为分隔符转成字符串
        const key = item.join('*')
        // 判断onj中是否有当前子集
        if (obj[key]) {
          // 有则value中增加当前skuid
          obj[key].push(sku.id)
        } else {
          // 没有则新增 key：子集 value:[id]
          obj[key] = [sku.id]
        }
      })
    }
  })
  return obj
}
```

**4）让页面初始化使就判断好选项`禁用状态`**

在setup中定义函数：

```js
// 页面初始化判断商品选项禁用
const isDisabled = (specs, PathMap) => {
  // 遍历商品类别
  specs.forEach((spec) => {
    // 遍历当前商品类别的选项
    spec.values.forEach((option) => {
      // 判断路径字典有没有当前选项 给disabled赋值
      if (PathMap[option.name]) {
        option.disabled = false
      } else {
        option.disabled = true
      }
    })
  })
}
isDisabled(props.goods.specs, PathMap)
```

选项的`点击事件函数`中开头增加一个判断，如果当前选项`disabled`为true停止往下判断

```js
if (option.disabled) return
```

**5）点击选项后判断其他选项是否禁用**

先定义一个获取当前选择的选项的数组的函数

```js
const getCurrentSelected = (specs) => {
  // 建立空数组
  const arr = []
  // 遍历选项类别
  specs.forEach((spec, index) => {
    // 获取当前选项类别的选中的对象
    const option = spec.values.find(item => item.selected)
    // 如果获取到了给arr赋值，设置下标和路径字典一致
    if (option) {
      arr[index] = option.name
    } else {
      arr[index] = undefined
    }
  })
  return arr
}
```

而后根据当前选择的选项数组 判断当前和其他组合是否禁用

```js
// 更新选项禁用状态
const updateDisabled = (specs, PathMap) => {
  // 获取当前选中的数组
  const currentSelected = JSON.stringify(getCurrentSelected(specs))
  // 遍历选项类别
  specs.forEach((spec, index) => {
    // 遍历当前类别选项
    spec.values.forEach(option => {
      // 拷贝当前选项列表
      const selectValues = JSON.parse(currentSelected)
      // 当前选项 + 遍历的当前选项 组合
      selectValues[index] = option.name
      // 格式化key为何路径字典相同
      const key = selectValues.filter(v => v).join('*')
      // 判断当前选项 + 遍历的当前选项 组合是否存在于路径字典
      if (PathMap[key]) {
        option.disabled = false
      } else {
        option.disabled = true
      }
    })
  })
}
```

**6）将`全部选中`得数据传出**

在`选项点击事件`中做`判断`

```js
// 判断选中所有项将数据传出
// 将当前选择的选项数组获取到
const selectedArr = getCurrentSelected(props.goods.specs).filter(
  (v) => v
)
// 判断当前选项数组和选项类别数量是否一致，一致既是全部选中了
if (selectedArr.length === props.goods.specs.length) {
  // 从字典获取id
  const skuId = PathMap[selectedArr.join('*')][0]
  // 根据id获取到sku
  const sku = props.goods.skus.find((item) => item.id === skuId)
  // 传出数据
  context.emit('change', {
    skuId: sku.id,
    price: sku.price,
    oldPrice: sku.oldPrice,
    inventory: sku.inventory,
    specsText: sku.specs
      .reduce((str, item) => `${str} ${item.name}：${item.valueName}`, '')
      .trim('')
  })
} else {
  // 否则传一个空的
  context.emit('change', {})
}
```

`接收`数据

```vue
<GoodsSku :goods="goods" skuId="1369155873162661889" @change="handleSelected"></GoodsSku>
```

# 

# 人力资源项目

## 技术栈：

**`vue-admin-template`  二次开发** 

`ES6`, `vue`, `vuex`, `vue-router`, `vue-cli`, `axios`, `element-ui`

## 项目前置

### 1.**二次封装axios**

`@/utils/request`

```js
// 封装了axios
// axios 发送请求
import axios from 'axios'

const service = axios.create({
  // 根据环境不同baseURL会改变
  baseURL: process.env.VUE_APP_BASE_API, // url = base url + request url
})

// request 请求拦截器
service.interceptors.request.use(
  config => {
    return config
  },
  error => {
    return Promise.reject(error)
  }
)

// 响应拦截器
service.interceptors.response.use(

  response => {
    return response
  },
  // 错误的处理
  error => {
    return error
  }
)

export default service
```

`@/api/xxx.js`

```js
// 导入request
import request from '@/utils/request'

/**
 * 登录接口
 * @param {Object} data
 * @param {String} data_Mobile
 * @param {String} data_Password
 * @returns {Promise} Promise
 */
export function login(data) {
  return request({
    url: '/vue-admin-template/user/login',
    method: 'post',
    data
  })
}

```

`@/api/index.js`

```js
// 统一导出所有接口
export * from '/xxx'
```

### 2.利用vue-cli跨域解决

跨域是因为浏览器的同源策略所导致，同源策略（Same origin policy）是一种约定，它是浏览器最核心也最基本的安全功能，同源是指：域名、协议、端口相同

![image-20220907104811142](https://yang-cloud-img.oss-cn-beijing.aliyuncs.com/img/image-20220907104811142.png)

**利用vue-cli内置的代理服务器解决跨域**

`.env.development`默认地址改为`/api`

```js
# base api
VUE_APP_BASE_API = '/api'
```

`vue.config.js`中**devServer**增加**proxy**

```js
 devServer: {
    // 项目启动的端口号
    proxy: {
      // 这里的 api 表示如果我们的请求地址以 /api 开头的时候，就出触发代理机制
      '/api': {
        target: 'http://ihrm.itheima.net', // 需要代理的地址
        changeOrigin: true // 是否跨域，需要设置此值为 true 才可以让本地服务代理我们发出请求
      }
    },
    port: port,
    overlay: {
      warnings: false,
      errors: true
    }
  }
```

### 3.axios响应data剥壳处理

`@/utils/request`

```js
// 响应拦截器
service.interceptors.response.use(

  response => {
    // response.data 因为axios在处理返回数据的时候会给response外层包裹一个data
    return response.data
  },
  // 错误的处理
  error => {
    return error
  }
)
```

### 4.设置路由

`@/router/index.js`

一级路由：login,404,layout

```js
export const constantRoutes = [
  {
    path: '/login',
    component: () => import('@/views/login/index'),
    // 如果设置为true，项目将不会显示在侧边栏中（默认值为false）
    hidden: true
  },

  {
    path: '/404',
    component: () => import('@/views/404'),
    hidden: true
  },

  {
    path: '/',
    component: Layout,
    redirect: '/dashboard',
    // 二级陆游 =》看看layout.vue文件中的<app-main />部分的代码 对应看
    children: [{
      path: 'dashboard',
      name: 'Dashboard',
      component: () => import('@/views/dashboard/index'),
      // title控制的是路由的名字还有面包屑的名子
      // icon是路由前面的小icon
      meta: { title: '小憨包', icon: 'dashboard' }
    }]
  },
```



## 登录模块

实现思路：

<img src="https://yang-cloud-img.oss-cn-beijing.aliyuncs.com/img/image-20220907173006928.png" alt="image-20220907173006928" style="zoom: 67%;" />

### 1.布局页面

### 2.绑定表单data、设置表单校验

### 3.登录按钮事件

**表单兜底校验：**

**校验通过：通过`store`的`actions`发送请求**

- 设置state
  - token

`@/store/modules/user.js`

```js
import { getProfile, Login } from '@/api'
import { getToken, setToken } from '@/utils/auth'
import { Message } from 'element-ui'

// 设定默认state值
const defaultstate = () => {
  return {
    token: getToken(),
  }
}

const state = defaultstate()

const mutations = {
  // 设置token
  SET_TOKEN(state, value) {
    setToken(value)
    state.token = value
  }
}

const actions = {
  // 登录
  async login({ commit }, form) {
    // 登录接口
    const data = await Login(form)
    const token = data.data
    // 设置token
    commit('SET_TOKEN', token)
    Message.success(data.message)
    // 返回登录接口的结果
    return data
  }
}

export default {
  // 命名空间开启
  namespaced: true,
  state,
  mutations,
  actions
}
```

获取用户信息的接口需要在请求头增加token

`@/utile/request.js`:判断有token在请求头加上token

```js
// request 请求拦截器
service.interceptors.request.use(
  config => {
    // 获得store的token
    const token = store.getters.token
    // 如果有token在请求头上加上token
    if (token) {
      config.headers.Authorization = `Bearer ${token}`
    }
    return config
  },
  error => {
    return Promise.reject(error)
  }
)
```

登录按钮接口

`@/view/login/index.vue`

```js
handleLogin() {
  // 兜底校验
  this.$refs.loginRef.validate(async vali => {
    if (vali) {
      // 等待返回的登录结果
      const data = await this.$store.dispatch('user/login', this.form)
      // 判断登录成功即跳转
      if (data.success) this.$router.replace('/')
    }
  })
}
```

## 路由守卫验证token

>  路由跳转前后，需要进行一些处理

<img src="https://yang-cloud-img.oss-cn-beijing.aliyuncs.com/img/image-20220907192030136.png" alt="image-20220907192030136" style="zoom:67%;" />

- 进度条
  - 利用`nprogress`插件前置守卫进度条开始，后置守卫进度条结束
- 每次跳转路由需要`验证token`是否存在
  - 验证`token`存在：验证目标是否login，是则回到主页，否则通过
  - `token`不存在验证是否是白`名单`中的地址，如果是则`通过`；不是则跳转至`登录`页

`@/permission.js`:控制页面权限

```js
import router from '@/router'
import { Message } from 'element-ui'
// 进度条设置
import NProgress from 'nprogress'
import 'nprogress/nprogress.css'
import store from './store'

const WhiteList = ['/login']
// 全局路由前置守卫
router.beforeEach((to, from, next) => {
  // 进度条开始
  NProgress.start()
  // 获取store的token
  const token = store.getters.token

  // 判断是否有token
  if (token) {
    // 如果有token请求进入login页面则回到主页
    if (to.path === '/login') {
      next('/')
      // 进度条结束
      NProgress.done()
    } else {
      next()
    }
  } else {
    // 如果没有token，判断是否在白名单中
    if (WhiteList.includes(to.path)) {
      next()
    } else {
      // 登录已过期
      Message.success('登录已过期')
      next('/login')
    }
  }
})

// 全局路由后置守卫
router.afterEach((to, from, failure) => {
  // 进度条结束
  NProgress.done()
})

```

## 获取用户信息

> 因为store保存的用户信息一刷新就没了，所以需要在每次进入页面时都获取用户信息，登录页除外

`@/store/modules/user.js`store设置保存用户信息

```js
import { getProfile, Login } from '@/api'
import { getToken, setToken } from '@/utils/auth'
import { Message } from 'element-ui'

// 设定默认state值
const defaultstate = () => {
  return {
    token: getToken(),
    userInfo: null
  }
}

const state = defaultstate()

const mutations = {
  // 设置用户信息
  SET_USERINFO(state, value) {
    state.userInfo = value
  }
}

const actions = {
  // 获取用户信息
  async getUserProfile({ commit }) {
    const res = await getProfile()
    commit('SET_USERINFO', res.data)
    console.log(res)
  }
}

export default {
  namespaced: true,
  state,
  mutations,
  actions
}

```

`@/permission.js`有token和不是login页面触发

```js
// 判断是否有token
if (token) {
  // 如果有token请求进入login页面则回到主页
  if (to.path === '/login') {
    next('/')
    // 进度条结束
    NProgress.done()
  } else {
    // 有token并且不是登录页获取用户信息
    // 判断 是否已经有用户数据
    if (!store.getters.avatar) {
      await store.dispatch('user/getUserProfile')
    }
    next()
  }
```

## 处理token过期

`@/utils/request.js`在响应拦截器的错误拦截中处理

```js
// 响应拦截器
service.interceptors.response.use(

  response => {
    // response.data 因为axios在处理返回数据的时候会给response外层包裹一个data
    return response.data
  },
  // 错误的处理
  error => {
    // 如果错误编码是10002 代表token过期
    if (error?.response?.data?.code === 10002) {
      // 提示
      Message.error(error.response.data.message + '请重新登录')
      // 重置用户信息
      store.commit('user/RESET_STATE')
      // 重置token
      store.commit('user/REMOVE_TOKEN')
      // 跳转至登录页
      router.replace('/login')
    } else {
      Message.error(error.response.data.message)
    }
    return error
  }
)
```

## 主页头部样式

`@/layout/Navbar.vue`

```js
<template>
  <div class="navbar">
    <hamburger :is-active="sidebar.opened" class="hamburger-container" @toggleClick="toggleSideBar" />
    <!-- 公司名 -->
    <span class="title">江苏传智播客教育科技股份有限公司</span>
    <!-- 右侧菜单 -->
    <div class="right-menu">
      <el-dropdown class="avatar-container" trigger="click">
        <div class="avatar-wrapper">
          <!-- 头像 -->
          <img :src="avatar || 'http://ihrm.itheima.net/static/img/head.b6c3427d.jpg'" class="user-avatar">
          <!-- 用户名显示 -->
          <span class="username">{{ username }}</span>
          <i class="el-icon-caret-bottom" />
        </div>
        <!-- 下拉菜单 -->
        <el-dropdown-menu slot="dropdown" class="user-dropdown">
          <router-link to="/">
            <el-dropdown-item>
              首页
            </el-dropdown-item>
          </router-link>
          <a target="_blank" href="https://github.com/PanJiaChen/vue-admin-template/">
            <el-dropdown-item>项目地址</el-dropdown-item>
          </a>
          <el-dropdown-item divided @click.native="logout">
            <span style="display:block;">退出登录</span>
          </el-dropdown-item>
        </el-dropdown-menu>
      </el-dropdown>
    </div>
  </div>
</template>

<script>
import { mapGetters } from 'vuex'
import Hamburger from '@/components/Hamburger'
export default {
  components: {
    Hamburger
  },
  // 获取用户名和头像
  computed: {
    ...mapGetters([
      'sidebar',
      'avatar',
      'username'
    ])
  },
  methods: {
    toggleSideBar() {
      this.$store.dispatch('app/toggleSideBar')
    }

  }
}
</script>

<style lang="scss" scoped>
.navbar {
  height: 50px;
  overflow: hidden;
  position: relative;
  background: #4979fb;
  box-shadow: 0 1px 4px rgba(0,21,41,.08);

  .hamburger-container {
    line-height: 46px;
    height: 100%;
    float: left;
    cursor: pointer;
    transition: background .3s;
    -webkit-tap-highlight-color:transparent;

    &:hover {
      background: rgba(0, 0, 0, .025)
    }
  }

  .title {
    color: #fff;
    font-size: 17px;
    line-height: 50px;
  }

  .right-menu {
    float: right;
    height: 100%;
    line-height: 50px;

    &:focus {
      outline: none;
    }

    .right-menu-item {
      display: inline-block;
      padding: 0 8px;
      height: 100%;
      font-size: 18px;
      color: #5a5e66;
      vertical-align: text-bottom;

      &.hover-effect {
        cursor: pointer;
        transition: background .3s;

        &:hover {
          background: rgba(0, 0, 0, .025)
        }
      }
    }

    .avatar-container {
      margin-right: 30px;

      .avatar-wrapper {
        height: 50px;
        margin-top: 5px;
        position: relative;
        .username {
          cursor: pointer;
          line-height: 50px;
          color: #fff;
          margin-left: 8px;
          font-size: 15px;
          vertical-align: top;
        }

        .user-avatar {
          margin-top: 8px;
          cursor: pointer;
          width: 30px;
          height: 30px;
          border-radius: 50%;
        }

        .el-icon-caret-bottom {
          color: #fff;
          cursor: pointer;
          position: absolute;
          right: -20px;
          top: 25px;
          font-size: 12px;
        }
      }
    }
  }
}
</style>
```

## 退出登录和登录未遂设置

**主页退出登录事件**

登录未遂实现：

- 跳转登录页时携带当前地址，使用`route.fullPath`
- 使用`encodeURIComponent`对`url`加密

`@/layout/Navbar.vue`

```js
<template>
  // 退出按钮
  <el-dropdown-item divided @click.native="logout">
     <span style="display:block;">退出登录</span>
  </el-dropdown-item>
</template>

<script>
export default {
  methods: {
    logout() {
      this.$store.commit('user/REMOVE_TOKEN')
      // 删除用户信息
      this.$store.commit('user/RESET_STATE')
      // 跳转至登录页 携带当前 路由地址
      this.$router.replace(`/login?redirect=${encodeURIComponent(this.$route.fullPath)}`)
      // 提示
      this.$message.success('退成成功，跳转至登录页!')
    }
  }
}
</script>
```

token失效也需要携带当前url：

`@/utils/request.js`

```js
// 响应拦截器
service.interceptors.response.use(

  response => {
    // response.data 因为axios在处理返回数据的时候会给response外层包裹一个data
    return response.data
  },
  // 错误的处理
  error => {
    // 如果错误编码是10002 代表token过期
    if (error?.response?.data?.code === 10002) {
      // 提示
      Message.error(error.response.data.message + '请重新登录')
      // 重置用户信息
      store.commit('user/RESET_STATE')
      // 重置token
      store.commit('user/REMOVE_TOKEN')
      // 跳转至登录页 携带当前地址
+++++ router.push(`/login?redirect=${encodeURIComponent(router.currentRoute.fullPath)}`)
    } else {
      Message.error(error.response.data.message)
    }
    return error
  }
)
```

## 修改左侧导航栏样式

### 1）左侧导航栏样式

`@/style/variables.scss`:根据情况修改相应变量

```scss
// sidebar
// 导航栏选项文字
$menuText:#fff;
// 导航栏活动选项文字
$menuActiveText:#fff;
$subMenuActiveText:#f4f4f5; //https://github.com/ElemeFE/element/issues/12951

// 导航栏背景色
$menuBg:#5a8bff;
// 导航栏选项鼠标悬浮背景色
$menuHover:#fff;

$subMenuBg:#1f2d3d;
$subMenuHover:#001528;

$sideBarWidth: 210px;

// the :export directive is the magic sauce for webpack
// https://www.bluematador.com/blog/how-to-share-variables-between-js-and-sass
:export {
  menuText: $menuText;
  menuActiveText: $menuActiveText;
  subMenuActiveText: $subMenuActiveText;
  menuBg: $menuBg;
  menuHover: $menuHover;
  subMenuBg: $subMenuBg;
  subMenuHover: $subMenuHover;
  sideBarWidth: $sideBarWidth;
}
```

### 2）设置是否显示logo

`@/settings.js`

```js
module.exports = {
  // 浏览器标签上面的title
  title: '劳务分发系统',

  /**
   * @type {boolean} true | false
   * @description Whether fix the header
   */
  // 顶部的条是否固定
  fixedHeader: true,

  /**
   * 是否显示logo
   * @type {boolean} true | false
   * @description Whether show the logo in sidebar
   */
  sidebarLogo: true
}
```

### **3）logo图片设置**

`@/layout/components/Sidebar/Logo.vue`

```js
<template>
  <div class="sidebar-logo-container" :class="{ collapse: collapse }">
    <!-- log链接 -->
    <router-link key="collapse" class="sidebar-logo-link" to="/">
      <!-- log图片 -->
      <img src="@/assets/images/logo.png" class="sidebar-logo">
    </router-link>
  </div>
</template>

<script>
export default {
  name: 'SidebarLogo',
  props: {
    // 是否点击了收缩按钮
    collapse: {
      type: Boolean,
      required: true
    }
  }
}
</script>

<style lang="scss" scoped>
.sidebar-logo-container {
  position: relative;
  width: 100%;
  height: 50px;
  line-height: 50px;
  background: #2b2f3a;
  text-align: center;
  overflow: hidden;

  & .sidebar-logo-link {
    height: 100%;
    width: 100%;

    & .sidebar-logo {
      width: 140px;
      height: 55 px;
      vertical-align: middle;
      margin-right: 12px;
    }

    & .sidebar-title {
      display: inline-block;
      margin: 0;
      color: #fff;
      font-weight: 600;
      line-height: 50px;
      font-size: 14px;
      font-family: Avenir, Helvetica Neue, Arial, Helvetica, sans-serif;
      vertical-align: middle;
    }
  }

  &.collapse {
    .sidebar-logo {
      width: 40px;
      margin-right: 0px;
    }
  }
}
</style>
```

### 4）设置导航栏项目字体和图标样式

设置字体：

`@/layout/components/Sidebar/Item.vue`

```css
<style scoped>
  span {
    font-size: 14px;
  }
<style scoped>
```

设置图标：

`@/components/SvgIcon/index.vue`

```css
<style scoped>
.svg-icon {
  // 宽
  width: 2em;
  // 高
  height: 1em;
  fill: currentColor;
  overflow: hidden;
}
</style>
```

`@/style/sidebar.scss`

```js
.hideSidebar {
  .sidebar-container {
    width: 54px !important;
  }
  .main-container {
    margin-left: 54px;
  }
  .submenu-title-noDropdown {
    padding: 0 !important;
    position: relative;
    .el-tooltip {
      padding: 0 !important;
      // 这里设置 侧边栏收缩状态 的 图标左边距
      .svg-icon {
        margin-left: 12px;
      }
      .sub-el-icon {
        margin-left: 19px;
      }
    }
  }
```

## 动态路由设置

1）建立vue页面，例如：`@/view/approvals/index.js`

- approvals
- attendances
- departments
- employees
- permission
- salarys
- social
- setting

2）建立上述界面路由模块

`@/router/modules/approvals.js`

```js
import Layout from '@/layout'

export default {
  path: '/approvals',
  component: Layout,
  children: [
    {
      path: '',
      name: 'approvals',
      component: () => import('@/views/approvals'),
      meta: { title: '审批', icon: 'tree-table' }
    }
  ]
}
```

3）创建动态路由表，并暂时合并路由

`@router/index.js`

```js
// 导入动态路由
import approvals from '@/router/modules/approvals.js'
import attendances from '@/router/modules/attendances.js'
import departments from '@/router/modules/departments.js'
import employees from '@/router/modules/employees.js'
import permission from '@/router/modules/permission.js'
import salarys from '@/router/modules/salarys.js'
import social from '@/router/modules/social.js'
import setting from './modules/setting'

// 动态路由
const dynamicRoutes = [
  approvals,
  attendances,
  departments,
  employees,
  permission,
  salarys,
  setting,
  social
]

// 函数声明
const createRouter = () => new Router({
  // mode: 'history', // require service support
  // scrollBehavior 配置路由切换时的滚动条位置
  scrollBehavior: () => ({ y: 0 }),
  // 最后加·{ path: '*', redirect: '/404', hidden: true }·是保证404在最后面
  routes: [...constantRoutes, ...dynamicRoutes, { path: '*', redirect: '/404', hidden: true }]
})
```

## 面包屑组件封装

1）组件封装

`@/components/Breadcrumb.index.vue`

```js
<template>
  <div class="main">
    <el-card
      class="box-card"
      :body-style="{ height: '35px', padding: '0px', width: '100%' }"
    >
      <!-- 首页固定标签 -->
      <span
        class="item"
        :class="{ active_tab: $route.fullPath === '/dashboard' }"
      >
        <span class="active_icon">●</span>
        <router-link to="/">首页</router-link>
        <span class="el-icon-close" />
      </span>
      <!-- 路由标签 根据路由列表渲染-->
      <span
        v-for="item in list"
        :key="item.title"
        class="item"
        :class="{ active_tab: item.path === $route.fullPath }"
      >
        <span class="active_icon">●</span>
        <router-link :to="item.path">{{ item.title }}</router-link>
        <!-- 删除按钮 -->
        <span class="el-icon-close" @click="delBtn(item.path)" />
      </span>
    </el-card>
  </div>
</template>

<script>
export default {
  data() {
    return {
      // 路由列表
      list: []
    }
  },
  computed: {
    // 获取当前路由地址和标题
    currentPath() {
      const path = this.$route.fullPath
      const title = this.$route.meta.title
      return { path, title }
    }
  },
  watch: {
    // 监视当前路由 发生改变执行
    currentPath: {
      immediate: true,
      handler(newValue, oldValue) {
        if (newValue.title !== '首页') {
          this.updataList(newValue)
        }
      }
    }
  },
  methods: {
    // 更新路由列表
    updataList(newPath) {
      const index = this.list.findIndex((item) => item.path === newPath.path)
      if (index < 0) this.list.push(newPath)
    },
    // 删除按钮
    delBtn(path) {
      // 获得下标
      const index = this.list.findIndex((item) => item.path === path)
      // 如果下标不为-1
      if (index >= 0) {
        // 删除数组数据
        this.list.splice(index, 1)
      }
      // 如果当前页面为删除的页面
      if (this.$route.fullPath === path) {
        // 判断是否是最后一个页签
        if (index === this.list.length) {
          if (index === 0) {
            // 如果是仅剩页签就去首页
            this.$router.push('/')
          } else {
            // 不是仅剩页签就跳转数组的上一个页签
            this.$router.push(this.list[index - 1].path)
          }
          // 如果不是最后一个页签
        } else {
          // 就跳转到下一个页签
          this.$router.push(this.list[index].path)
        }
      }
    }
  }
}
</script>

<style lang="scss" scoped>
.main {
  width: 100%;
  height: 35px;

  .item {
    padding: 0 8px;
    display: inline-block;
    margin-top: 3px;
    margin-left: 4px;
    line-height: 25px;
    text-align: center;
    font-size: 12px;
    background-color: #fff;
    border: 1px solid #d9dce5;
    .active_icon {
      display: none;
      font-size: 18px;
      color: #fff;
      margin: 0 2px;
    }
    .el-icon-close {
      text-align: center;
      line-height: 25px;
      width: 22px;
      height: 25px;
      transform: scale(0.6);
      &:hover {
        background: #b6bccb;
        background-size: 30px;
        border-radius: 50%;
      }
    }
  }
  .active_tab {
    color: #fff;
    background-color: #409eff !important;
    .active_icon {
      display: inline;
    }
  }
}
</style>
>
```

2）固定在头部导航栏上

`@/layout/components/Navbar.vue`

```vue
<template>
  <div class="headNavbar">
    <div class="navbar">
    </div>
 +++<Breadcrumb />
  </div>
</template>
```



3）调整路由页面的上边距，因为加了一个面包屑

`@/layout/components/AppMain.vue`

```scss
<style scoped>
.app-main {
  /*50 = navbar  */
  min-height: calc(100vh - 50px);
  width: 100%;
  position: relative;
  overflow: hidden;
}
.fixed-header+.app-main {
++padding-top: 85px;
}
</style>
```

## 组织架构模块

### 页面布局

1）利用elment的el-row组件进行比例布局

2）使用elment的tree组件展示部门

- tree使用的是`[lable:name,children:[lable:name]]`形式的层级数据

- 请求到的数据是id与pid形式的，所以需要封装一个`递归函数`处理

  `@/utils/index.js`

  ```js
  /**
   * 递归处理组织架构数据
   * @param {Array} id-pid数组
   * @param {String} 根id
   */
  export function recursiveData(data, pid) {
    // 筛选出pid的根数据
    const arr = data.filter(item => item.pid === pid)
    // 根数据遍历
    arr.forEach(item => {
      // 子数据获取
      const children = recursiveData(data, item.id)
      // 如果子数据有的话 赋值children
      if (children.length) {
        item.children = children
      }
    })
    // 返回数组
    return arr
  }
  ```

- 将处理后的数据渲染到页面上

  `@/view/departments/index.vue`

  ```vue
  <el-tree
    v-loading="TreeLoading"
    :data="list"
    :props="props"
    :expand-on-click-node="false"
    style="margin-top: 10px"
  >
    <div slot-scope="{ node, data }" style="width: 100%; font-size: 14px">
      <el-row style="width: 100%">
        <!-- 节点名 -->
        <el-col :span="18"><span>{{ node.label }}</span> </el-col>
        <!-- 负责人 -->
        <el-col :span="2"><span> {{ data.manager }}</span></el-col>
        <!-- 操作下拉栏 -->
        <el-col :span="2">
          <el-dropdown @command="(command) => handlBtn(command, data)">
            <span>操作<i class="el-icon-arrow-down el-icon--right" /></span>
            <el-dropdown-menu slot="dropdown">
              <el-dropdown-item command="add">
                添加子部门
              </el-dropdown-item>
              <el-dropdown-item command="update">
                编辑部门
              </el-dropdown-item>
              <el-dropdown-item command="remove">
                删除部门
              </el-dropdown-item>
            </el-dropdown-menu>
          </el-dropdown>
        </el-col>
      </el-row>
    </div>
  </el-tree>
  ```

  

3）Dialog封装

- 用于增加、编辑部门信息的弹出框

  `@/components/Dialog/index.vue`

  ```vue
  <template>
    <el-dialog
      :title="title"
      :visible="dialogVisible"
      width="800px"
      @close="close"
    >
      <!-- 表单 -->
      <el-form ref="formRef" :model="form" :rules="rules">
        <el-form-item prop="name" label="部门名称" label-width="100px">
          <el-input
            v-model="form.name"
            style="width: 600px"
            type="text"
            placeholder="1-50个字符"
          /></el-form-item>
        <el-form-item prop="code" label="部门编码" label-width="100px">
          <el-input
            v-model="form.code"
            style="width: 600px"
            type="text"
            placeholder="1-50个字符"
          /></el-form-item>
        <el-form-item prop="manager" label="部门负责人" label-width="100px">
          <el-select
            v-model="form.manager"
            :loading="selectLoaing"
            style="width: 600px"
            placeholder="请选择负责人"
            @visible-change="selectLoading"
          >
            <el-option
              v-for="item in managerList"
              :key="item.id"
              :label="item.username"
              :value="item.username"
            /> </el-select></el-form-item>
        <el-form-item prop="introduce" label="部门介绍" label-width="100px">
          <el-input
            v-model="form.introduce"
            style="width: 600px"
            type="textarea"
            :rows="5"
            placeholder="1-300个字符"
          /></el-form-item>
      </el-form>
      <!-- footer -->
      <template slot="footer">
        <el-button @click="close">取消</el-button>
        <el-button
          type="primary"
          :loading="submitLoading"
          @click="submit"
        >确定</el-button>
      </template>
    </el-dialog>
  </template>
  
  <script>
  import { addDepartment, reqGetEmployeeSimple, updateDepartment } from '@/api'
  export default {
    props: {
      // 显示Dialog
      dialogVisible: {
        type: Boolean,
        default: false
      },
      // 默认部门列表
      defaultList: {
        type: Array,
        default: () => []
      },
      // 当前节点部门id
      id: {
        type: String,
        default: ''
      }
    },
    data() {
      // 部门名称自定义校验
      const nameValidate = (rule, value, callback) => {
        // 当前id的下属部门是否有名称相同的
        const falg = this.defaultList.some((item) => {
          if (item.pid === this.id) {
            return item.name === value
          }
        })
        console.log(falg)
        if (falg) {
          callback(new Error('同级部门有相同部门名称'))
        } else {
          callback()
        }
      }
      // 部门编码校验
      const codeValidate = (rule, value, callback) => {
        // 判断所有部门中是否有相同编码
        const falg = this.defaultList.some((item) => {
          return item.code === value
        })
        if (falg) {
          callback(new Error('已存在此部门编码'))
        } else {
          callback()
        }
      }
      return {
        // 表单数据
        form: {
          name: '',
          code: '',
          manager: '',
          introduce: ''
        },
        // 提交进度条
        submitLoading: false,
        // 表单校验
        rules: {
          name: [
            { required: true, message: '请输入部门名称', trigger: 'blur' },
            { min: 1, max: 50, message: '请输入1-50个字符', trigger: 'blur' },
            { validator: nameValidate, trigger: 'blur' }
          ],
          code: [
            { required: true, message: '请输入部门编码', trigger: 'blur' },
            { min: 1, max: 50, message: '请输入1-50个字符', trigger: 'blur' },
            { validator: codeValidate, trigger: 'blur' }
          ],
          manager: [{ required: true, message: '请选择负责人', trigger: 'blur' }],
          introduce: [
            { required: true, message: '请输入部门介绍', trigger: 'blur' },
            { min: 1, max: 50, message: '请输入1-300个字符', trigger: 'blur' }
          ]
        },
        // 负责人数据
        managerList: [],
        // 负责人选择框加载
        selectLoaing: false
      }
    },
    computed: {
      // Dialog标题
      title() {
        return this.form.id ? '编辑部门' : '新增部门'
      }
    },
    methods: {
      // 关闭回调
      close() {
        this.form = {
          name: '',
          code: '',
          manager: '',
          introduce: ''
        }
        this.$emit('close')
      },
      // Dialog提交
      submit() {
        // 兜底校验
        this.$refs.formRef.validate(async(vali) => {
          if (vali) {
            // 增加部门
            if (this.title === '新增部门') {
              // 提交按钮进度条开始
              this.submitLoading = true
              // 增加部门请求
              await addDepartment({ ...this.form, pid: this.id })
              // 提交按钮进度条结束
              this.submitLoading = false
              // 提示
              this.$message.success('增加部门成功')
              // 关闭dialog
              this.close()
              // 更新部门列表
              this.$emit('update')
            } else {
              // 提交按钮进度条开始
              this.submitLoading = true
              // 修改部门请求
              await updateDepartment(this.form)
              // 提示
              this.$message.success('修改部门成功')
              // 提交按钮进度条结束
              this.submitLoading = false
              // 关闭dialog
              this.close()
              // 更新部门列表
              this.$emit('update')
            }
          }
        })
      },
      // 负责人选择栏加载
      async selectLoading(flag) {
        // 如果是打开
        if (flag) {
          // 进度条开始
          this.selectLoaing = true
          // 负责人请求
          const res = await reqGetEmployeeSimple()
          // 负责人列表赋值
          this.managerList = res.data
          // 进度条结束
          this.selectLoaing = false
        }
      }
    }
  }
  </script>
  <style lang="scss" scoped></style>
  
  ```

4）操作下拉栏事件：添加、修改、删除

`@/view/departments/index.vue`

```vue
<template>
  <div class="main">
    <el-card body-style="padding:50px 0 50px 150px">
      <!-- 组织架构标题 -->
      <el-row class="row">
        <el-col :span="18">传智教育</el-col>
        <el-col :span="2" style="margin-left: 4px">负责人</el-col>
        <el-col
          :span="2"
        ><el-dropdown @command="mainAdd">
          <span>操作<i class="el-icon-arrow-down el-icon--right" /></span>
          <el-dropdown-menu slot="dropdown">
            <el-dropdown-item command="add"> 添加子部门 </el-dropdown-item>
          </el-dropdown-menu>
        </el-dropdown></el-col>
      </el-row>
      <!-- 组织架构tree -->
      <el-tree
        v-loading="TreeLoading"
        :data="list"
        :props="props"
        :expand-on-click-node="false"
        style="margin-top: 10px"
      >
        <div slot-scope="{ node, data }" style="width: 100%; font-size: 14px">
          <el-row style="width: 100%">
            <el-col :span="18">
              <span>{{ node.label }}</span>
            </el-col>
            <el-col
              :span="2"
            ><span>
              {{ data.manager }}
            </span></el-col>

            <el-col :span="2">
              <!-- 操作下拉栏 -->
              <el-dropdown @command="(command) => handlBtn(command, data)">
                <span>操作<i class="el-icon-arrow-down el-icon--right" /></span>
                <el-dropdown-menu slot="dropdown">
                  <el-dropdown-item command="add">
                    添加子部门
                  </el-dropdown-item>
                  <el-dropdown-item command="update">
                    编辑部门
                  </el-dropdown-item>
                  <el-dropdown-item command="remove">
                    删除部门
                  </el-dropdown-item>
                </el-dropdown-menu>
              </el-dropdown>
            </el-col>
          </el-row>
        </div>
      </el-tree>
    </el-card>
    <Dialog
      :id="id"
      ref="dialogRef"
      :default-list="defaultList"
      :dialog-visible="dialogVisible"
      @close="close"
      @update="getDepartments"
    />
  </div>
</template>

<script>
import { getDepartmentsList, removeDepartment } from '@/api'
import { recursiveData } from '@/utils'
import Dialog from './components/Dialog.vue'
export default {
  components: { Dialog },
  data() {
    return {
      component: {
        Dialog
      },
      // 默认组织架构列表
      defaultList: [],
      // 处理后组织架构列表
      list: [],
      // 组织架构加载进度条
      TreeLoading: false,
      // 当前组织架构节点id
      id: '',
      // 是否显示Dialog
      dialogVisible: false,
      // 组织架构Tree对应关系
      props: {
        children: 'children',
        label: 'name'
      }
    }
  },
  created() {
    // 发请求 获取部门列表
    this.getDepartments()
  },
  methods: {
    // 获取部门列表
    async getDepartments() {
      // 组织架构tree进度条 开始
      this.TreeLoading = true
      // 请求 组织架构列表
      const data = await getDepartmentsList()
      // 默认列表 赋值
      this.defaultList = data.data.depts
      // 处理 层级数据
      const arr = recursiveData(data.data.depts, '')
      // 层级数据赋值
      this.list = arr
      // 组织架构tree进度条 结束
      this.TreeLoading = false
    },
    // 标题增加部门
    mainAdd(command) {
      if (command === 'add') {
      // 赋值当前id
        this.id = ''
        this.dialogVisible = true
      }
    },
    // 操作下拉栏事件
    handlBtn(command, node) {
      // 赋值当前id
      this.id = node.id
      // 添加子部门事件
      if (command === 'add') {
        this.dialogVisible = true
      }
      // 修改部门事件
      if (command === 'update') {
        // dialog表单赋值当前选择的部门信息
        this.$refs.dialogRef.form = Object.assign({}, node)
        this.dialogVisible = true
      }
      // 删除部门事件
      if (command === 'remove') {
        // 删除提示
        this.$confirm('此操作将永久删除该部门, 是否继续?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        })
          .then(async() => {
            // 删除请求
            await removeDepartment(this.id)
            // 更新部门列表
            this.getDepartments()
            this.$message({
              type: 'success',
              message: '删除成功!'
            })
          })
          .catch(() => {
            this.$message({
              type: 'info',
              message: '已取消删除'
            })
          })
      }
    },
    // 关闭dialog回调
    close() {
      this.dialogVisible = false
    }

  }
}
</script>

<style lang="scss" scoped>
.main {
  padding: 20px;
}
</style>

```

## 员工模块

### 工具栏

封装全局工具栏组件

`@/components/Toolbar/index.vue`

```js
<template>
  <div class="main">
    <el-card :body-style="{ height: '70px' }">
      <div class="left">
        <slot name="left" />
      </div>
      <div class="right">
        <slot name="right" />
      </div>
    </el-card>
  </div>
</template>

<script>
export default {}
</script>

<style lang="scss" scoped>
.main {
  padding: 0 10px;
  .left {
    float: left;
  }
  .right {
    float: right;
  }
}
</style>
```

Vue全局注册

`@/components/index.js`

```js
import Toolbar from './Toolbar'
export default {
  install(Vue) {
    Vue.component('Toolbar', Toolbar)
  }
}
```

`@/main.js`

```js
import components from '@/components'
Vue.use(components)
```

放置工具栏

`@/view/employees/index.vue`

```js
<template>
  <div class="main">
    <!-- 工具栏 -->
    <Toolbar>
      <!-- 工具栏左侧 -->
      <div slot="left">
        <div class="tip">
          <div class="centent"><i class="el-icon-info" /> 共26条记录</div>
        </div>
      </div>
      <!-- 工具栏右侧 -->
      <div slot="right">
        <el-button type="danger" size="mini">普通excel导出</el-button>
        <el-button type="info" size="mini">复杂表头excel导出</el-button>
        <el-button type="success" size="mini">excel导入</el-button>
        <el-button type="primary" size="mini">新增员工</el-button>
      </div>
    </Toolbar>
  </div>
</template>

<script>
export default {}
</script>

<style lang="scss" scoped>
.main {
  padding: 20px 10px 10px 10px;
  .tip {
    width: 120px;
    height: 30px;
    text-align: center;
    border: 1px solid #a4d3fd;
    border-radius: 3px;
    background-color: #eaf6fe;
    .centent {
      font-size: 14px;
      line-height: 30px;
      .el-icon-info {
        color: #629cfb;
      }
    }
  }
}
</style>
```

### 员工列表表格

> 利用elment的table组件，利用请求获取的数据渲染表格

1）请求数据

`@/view/employees/index.vue`

```js
data() {
    return {
      // 表格进度条
      tableLoading: false,
      // 员工列表
      employeeList: [],
      // 当前页
      page: 1,
      // 页大小
      size: 10,
      // 总数
      total: 0
}
    
// 员工列表请求
async getEmployeeList() {
  // 表格进度条开始
  this.tableLoading = true
  // 发请求获取列表
  const params = { page: this.page, size: this.size }
  const res = await reqGetEmployeeList(params)
  // 员工列表赋值
  this.employeeList = res.data.rows
  // 总数赋值
  this.total = res.data.total
  // 表格进度条结束
  this.tableLoading = false
},
```

2）表格组件与分页组件设置 数据对应

`@/view/employees/index.vue`

```vue
<el-table v-loading="tableLoading" :data="employeeList" border style="width: 100%">
  <el-table-column label="序号" width="180" />
  <el-table-column prop="username" label="姓名" width="180" />
  <el-table-column label="头像" />
  <el-table-column prop="mobile" label="手机号" />
  <el-table-column prop="workNumber" label="工号" />
  <el-table-column prop="formOfEmployment" label="聘用形式" />
  <el-table-column prop="departmentName" label="部门" />
  <el-table-column prop="timeOfEntry" label="入职时间" />
  <el-table-column label="状态" />
  <el-table-column label="操作" />
</el-table>
<el-pagination
  layout="total,prev, pager, next"
  :page-size="size"
  :current-page="page"
  :total="total"
  @current-change="pageChange"
/>
```

3）序号格式化 根据页数和页大小计算序号

`@/view/employees/index.vue`

```vue
<el-table-column label="序号" width="180" type="index" :index="format" />

<script>
 // 序号格式化
 format(index) {
   return (this.page - 1) * this.size + index + 1
 }
</script>
```

4）显示头像和生成二维码

> 利用table插槽获取头像链接

图片错误显示默认图片：利用`@error`事件

点击图片弹出二维码显示dialog：利用`qrcode`插件生成二维码

- `qrcode`用法：`qrcode.toCanvas(Canvas, content)`

`@/view/employees/index.vue`

```vue
<el-table-column label="头像">
  <template v-slot=" { row }">
    <img class="staffPhoto" :src="row.staffPhoto" @error="imgError" @click="imgClick(row.staffPhoto)">
  </template>
</el-table-column>
<script>
  import qrcode from 'qrcode'
  export default {
	methods:{
      // 图片加载失败回调
      imgError(e) {
       e.target.src = 'http://ihrm.itheima.net/static/img/head.b6c3427d.jpg'
      },
      // 点击图片事件
       imgClick(url) {
         if (url.trim()) {
           this.dialogVisible = true
           this.$nextTick(() => {
             qrcode.toCanvas(this.$refs.myCanvas, url)
           })
         } else {
           this.$message.info('无头像！')
         }
       }
    }
  }
</script>
```

5）聘用形式 格式化

> 由于 聘用形式 从后台后去的是id形式的 需要通过枚举显示中文的聘用形式

`@/view/employees/index.vue`:使用formatter属性格式化表格内容

- 有四个回调参数row, column, cellValue, index

```vue
// 表格行
<el-table-column prop="formOfEmployment" label="聘用形式" :formatter="hireFormatter" />

// 聘用形式格式化
<script>
  export default {
	methods:{
      hireFormatter(row, column) {
  		console.log(EmployeeEnum)
  		const obj = EmployeeEnum.hireType.find(item => +item.id === +row.formOfEmployment)
  		return obj ? obj.value : '未知'
	},
    }
  }
</script>

```

### 删除员工

1）设置删除按钮事件

`@/view/employees/index.vue`

```vue
<el-table-column label="操作" width="210">
  <template v-slot=" { row }">
    <el-link type="primary" :underline="false">查看</el-link>
    <el-link type="primary" :underline="false">转正</el-link>
    <el-link type="primary" :underline="false">调岗</el-link>
    <el-link type="primary" :underline="false">离职</el-link>
    <el-link type="primary" :underline="false">角色</el-link>
    <el-link type="primary" :underline="false" @click="handleDel(row.id)">删除</el-link>
  </template>
</el-table-column>

<script>
import { reqDelEmployee, reqGetEmployeeList } from '@/api'
export default {
    // 删除按钮回调
    handleDel(id) {
      console.log(id)
      // 确定框
      this.$confirm('此操作将永久删除该员工, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(async() => {
        // 进度条开始
        this.cardLoading = true
        // 删除请求
        const { message, success } = await reqDelEmployee(id)
        // 进度条结束
        this.cardLoading = false
        // 提示
        this.$message({
          type: success ? 'success' : 'warning',
          message: message
        })
        // 重新渲染表格
        this.getEmployeeList()
      }).catch(() => {
        this.$message.info('已取消删除')
      })
    },
  }
}
</script>
```

### 新增员工

1）先进行`dialog`封装组件

`@/view/employees/components/AddDialog.vue`：布局表单

```js
<template>
  <el-dialog
    title="新增员工"
    :visible="addDialogVisible"
    @close="addDialogClose"
  >
    <el-form :model="form">
      <el-form-item label="姓名" label-width="100px">
        <el-input
          v-model="form.username"
          style="width: 500px"
          placeholder="请输入姓名"
        />
      </el-form-item>
      <el-form-item label="手机号" label-width="100px">
        <el-input
          v-model="form.mobile"
          style="width: 500px"
          placeholder="请输入手机号"
        />
      </el-form-item>
      <el-form-item label="入职时间" label-width="100px">
        <el-date-picker
          v-model="form.timeOfEntry"
          align="right"
          type="date"
          placeholder="选择日期"
        />
      </el-form-item>
      <el-form-item label="聘用形式" label-width="100px">
        <el-select v-model.number="form.formOfEmployment" style="width: 500px">
          <el-option :value="1" label="正式" />
          <el-option :value="2" label="非正式" />
        </el-select>
      </el-form-item>
      <el-form-item label="工号" label-width="100px" prop="workNumber">
        <el-input v-model="form.workNumber" style="width: 500px" placeholder="请选择工号" />
      </el-form-item>
      <!-- 部门 -->
      <el-form-item label="部门" label-width="100px">
        <el-input
          v-model="form.departmentName"
          style="width: 500px"
        />
      </el-form-item>
      <el-form-item label="转正时间" label-width="100px">
        <el-date-picker
          v-model="form.correctionTime"
          align="right"
          type="date"
          placeholder="选择日期"
        />
      </el-form-item>
    </el-form>
    <!-- 底部 -->
    <div slot="footer">
      <el-button type="primary" @click="submit">确定</el-button>
      <el-button type="">取消</el-button>
    </div>
  </el-dialog>
</template>

<script>
import { getDepartmentsList } from '@/api'
import { recursiveData } from '@/utils'
export default {
  props: {
    addDialogVisible: {
      type: Boolean,
      default: false
    }
  },
  data() {
    return {
      // 表单
      form: {
        username: '',
        mobile: '',
        formOfEmployment: '',
        workNumber: '',
        departmentName: '',
        timeOfEntry: '',
        correctionTime: ''
      },
    }
  },
    
}
</script>
```

点击`新增员工`按钮显示`增加dialog`

`@/view/employees/index.vue`

```vue
<template>
  <el-button type="primary" size="mini" @click="handleAdd">新增员工</el-button>
  <!-- 新增员工弹框 -->
  <AddDialog :add-dialog-visible="addDialogVisible" />
</template>

<script>
import AddDialog from './components/AddDialog.vue'
export default {
  components: { AddDialog },
  data() {
    return {
      // 新增员工dialog是否显示
      addDialogVisible: false,
    }
  },

  methods: {
    // 新增员工按钮回调
    handleAdd() {
      this.addDialogVisible = true
    }

  }
}
</script>
```

2）处理`部门选择器`

首先在部门`input`下面放置一个`tree`组件，和`组织架构`模块的一样

`@/view/employees/components/AddDialog.vue`

```vue
<!-- 部门 -->
 <!-- 这里使用 :value 只能读取 不能输入 -->
  <el-input
    :value="form.departmentName" // 
    style="width: 500px"
    placeholder="请选择部门"
    @focus="departmentFocus"
    @blur="departmentBlur"
/>
<el-tree
  v-show="showTree"
  v-loading="TreeLoading"
  :data="departmentsList"
  :props="props"
  @node-click="departmentClick"
>
  <div slot-scope="{ node }" style="width: 100%; font-size: 14px">
    <!-- 节点名 -->
    <span>{{ node.label }} </span>
  </div>
</el-tree>
```

tree依赖数据准备：

`@/view/employees/components/AddDialog.vue`

```js
<script>
import { getDepartmentsList } from '@/api'
import { recursiveData } from '@/utils'
export default {
  data() {
    return {
      // 部门tree进度条
      TreeLoading: false,
      // 是否显示部门tree
      showTree: false,
      // 部门列表
      departmentsList: [],
      // tree对应关系
      props: {
        label: 'name',
        children: 'children'
      }
    }
  },
</script>
```

逻辑：

1. 部门input聚焦时 获取`部门列表`并显示`tree`
2. 选择部门后 隐藏`tree` 赋值部门`input`为选择的`部门名称`

- input聚焦回调：判断`部门列表`为空则请求，不为空则不请求
- dialog关闭回调：清空`部门列表`

`@/view/employees/components/AddDialog.vue`

```js
  methods: {
    // 获取部门列表
    async getDepartments() {
      // tree进度条开始
      this.TreeLoading = true
      //   如果部门列表为空 发请求
      if (!this.departmentsList.length) {
        const res = await getDepartmentsList()
        this.departmentsList = recursiveData(res.data.depts, '')
      }
      //   tree进度条结束
      this.TreeLoading = false
      console.log(this.departmentsList)
    },
    // 部门表单focus
    departmentFocus() {
      // 显示tree
      this.showTree = true
      //   获取部门列表
      this.getDepartments()
    },
    // 部门表单blur
    departmentBlur() {
      // 隐藏tree
    },
    // dialog关闭回调
    addDialogClose() {
      // 表单复位
      this.form = {
        username: '',
        mobile: '',
        formOfEmployment: '',
        workNumber: '',
        departmentName: '',
        timeOfEntry: '',
        correctionTime: ''
      }
      //  清除部门列表
      this.departmentsList = []
      //   隐藏dialog
      this.$parent.addDialogClose()
    },
    // 部门选择回调
    departmentClick(data) {
      this.form.departmentName = data.name
      this.showTree = false
      console.log(data)
    }
  }
}
```

3）设置表单校验规则：

`@/view/employees/components/AddDialog.vue`

```js
 data() {
   // 手机号校验
   const mobileValidator = (rule, value, callback) => {
     if (valiMobile(value)) {
       callback()
     } else {
       callback(new Error('手机号格式不正确'))
     }
   }
   return {
     // 表单校验
     formRules: {
       name: [
         { required: true, message: '请输入姓名！', trigger: 'blur' }
       ],
       mobile: [
         { required: true, message: '请输入手机号！', trigger: 'blur' },
         { validator: mobileValidator, trigger: 'blur' }
       ],
       formOfEmployment: [
         { required: true, message: '聘用形式不能为空', trigger: ['blur', 'change'] }
       ],
       workNumber: [
         { required: true, message: '工号不能为空', trigger: ['blur', 'change'] }
       ],
       departmentName: [
         { required: true, message: '部门不能为空', trigger: ['blur', 'change'] }
       ],
       timeOfEntry: [
         { required: true, message: '请选择入职时间', trigger: ['blur', 'change'] }
       ]
     }
   }
 },
```

4）提交按钮事件

`@/view/employees/components/AddDialog.vue`

```vue
<el-button type="primary" :loading="submitLoading" @click="submit">确定</el-button>

<script>
import {reqAddEmployee } from '@/api'
export default {
  data() {
    return {
      // 确定按钮进度条
      submitLoading: false,

    }
  },
  methods: {
    // 提交按钮
    submit() {
      // 兜底校验
      this.$refs.formRef.validate(async(vali) => {
        // 校验通过
        if (vali) {
          // 按钮进度条开始
          this.submitLoading = true
          // 请求
          const res = await reqAddEmployee(this.form)
          if (res.success) {
            // 重新渲染列表
            this.$parent.getEmployeeList()
            // 提示
            this.$message.success('新增员工成功')
            // 关闭dialog
            this.addDialogClose()
          }
          // 按钮进度条结束
          this.submitLoading = false
        }
      })
    }
  }
}
</script>
```

### 批量导入员工

1. **封装 excel 导入组件(uploadExcel)**

   需要的依赖：`xlsx`

   `@/components/UploadExcel.vue`

   ```vue
   <template>
     <div>
       <input
         ref="excel-upload-input"
         class="excel-upload-input"
         type="file"
         accept=".xlsx, .xls"
         @change="handleClick"
       />
       <div class="box">
         <div class="left">
           <el-button
             :loading="loading"
             style="margin-left: 16px"
             size="mini"
             type="primary"
             @click="handleUpload"
           >
             点击上传
           </el-button>
           <div class="text">(推荐下载模板文件，填写后上传)</div>
           <div class="text">点击查看文件上传要求</div>
         </div>
         <div
           class="rigth"
           @drop="handleDrop"
           @dragover="handleDragover"
           @dragenter="handleDragover"
           @click="handleUpload"
         >
           <div class="el-icon-upload" />
           <div>将文件拖入上传</div>
         </div>
       </div>
     </div>
   </template>
   
   <script>
   import * as XLSX from "xlsx";
   
   export default {
     props: {
       beforeUpload: Function, // eslint-disable-line
       onSuccess: Function, // eslint-disable-line
     },
     data() {
       return {
         loading: false,
         excelData: {
           header: null,
           results: null,
         },
       };
     },
     methods: {
       generateData({ header, results }) {
         this.excelData.header = header;
         this.excelData.results = results;
         this.onSuccess && this.onSuccess(this.excelData);
       },
       handleDrop(e) {
         e.stopPropagation();
         e.preventDefault();
         if (this.loading) return;
         const files = e.dataTransfer.files;
         if (files.length !== 1) {
           this.$message.error("Only support uploading one file!");
           return;
         }
         const rawFile = files[0]; // only use files[0]
   
         if (!this.isExcel(rawFile)) {
           this.$message.error(
             "Only supports upload .xlsx, .xls, .csv suffix files"
           );
           return false;
         }
         this.upload(rawFile);
         e.stopPropagation();
         e.preventDefault();
       },
       handleDragover(e) {
         e.stopPropagation();
         e.preventDefault();
         e.dataTransfer.dropEffect = "copy";
       },
       handleUpload() {
         this.$refs["excel-upload-input"].click();
       },
       handleClick(e) {
         const files = e.target.files;
         const rawFile = files[0]; // only use files[0]
         if (!rawFile) return;
         this.upload(rawFile);
       },
       upload(rawFile) {
         this.$refs["excel-upload-input"].value = null; // fix can't select the same excel
   
         if (!this.beforeUpload) {
           this.readerData(rawFile);
           return;
         }
         const before = this.beforeUpload(rawFile);
         if (before) {
           this.readerData(rawFile);
         }
       },
       readerData(rawFile) {
         this.loading = true;
         return new Promise((resolve, reject) => {
           const reader = new FileReader();
           reader.onload = (e) => {
             const data = e.target.result;
             const workbook = XLSX.read(data, { type: "array" });
             const firstSheetName = workbook.SheetNames[0];
             const worksheet = workbook.Sheets[firstSheetName];
             const header = this.getHeaderRow(worksheet);
             const results = XLSX.utils.sheet_to_json(worksheet);
             this.generateData({ header, results });
             this.loading = false;
             resolve();
           };
           reader.readAsArrayBuffer(rawFile);
         });
       },
       getHeaderRow(sheet) {
         const headers = [];
         const range = XLSX.utils.decode_range(sheet["!ref"]);
         let C;
         const R = range.s.r;
         /* start in the first row */
         for (C = range.s.c; C <= range.e.c; ++C) {
           /* walk every column in the range */
           const cell = sheet[XLSX.utils.encode_cell({ c: C, r: R })];
           /* find the cell in the first row */
           let hdr = "UNKNOWN " + C; // <-- replace with your desired default
           if (cell && cell.t) hdr = XLSX.utils.format_cell(cell);
           headers.push(hdr);
         }
         return headers;
       },
       isExcel(file) {
         return /\.(xlsx|xls|csv)$/.test(file.name);
       },
     },
   };
   </script>
   
   <style scoped>
   .box {
     text-align: center;
   }
   .excel-upload-input {
     display: none;
     z-index: -9999;
   }
   .rigth {
     border: 2px dashed #bbb;
     width: 400px;
     height: 180px;
     font-size: 24px;
     border-radius: 5px;
     text-align: center;
     color: #bbb;
     display: inline-block;
     border-left: 0;
   }
   
   .left {
     vertical-align: top;
     padding: 50px;
     border: 2px dashed #bbb;
     width: 400px;
     height: 180px;
     font-size: 24px;
     border-radius: 5px;
     text-align: center;
     color: #bbb;
     display: inline-block;
   }
   .el-icon-upload {
     font-size: 100px;
   }
   .left .text {
     font-size: 12px;
     color: #000;
     margin-top: 5px;
   }
   </style>
   ```

   **上述组件接受两个函数**

   **beforeUpload** 导入前的回调 有一个实参: 文件对象

   - beforeUpload 需要**返回 true**才可继续导入成功的回调

   **onSuccess** 导入成功后的回调 有一个实参：一个对象，对象有两个属性`results`, `header`\*\*

   - results 代表 **表体数据**
   - header 代表 **表头数据**

2. 创建导入页面

   `@/views/uploadExcel/index.vue`：导入组件 

   - 导入组件
   - 写两个回调函数

   ```vue
   <template>
     <div class="uploadExcelMain">
       <!-- 导入组件 -->
       <UploadExcel :before-upload="beforeUpload" :on-success="onSuccess" />
     </div>
   </template>
   
   <script>
   import { reqUploadEmployee } from '@/api'
   export default {
   
     methods: {
       // 导入前回调
       beforeUpload(File) {
         return true
       },
       // 导入后回调
       async onSuccess(excelData) {
         // 枚举创建
         const userRelations = {
           '入职日期': 'timeOfEntry',
           '手机号': 'mobile',
           '姓名': 'username',
           '转正日期': 'correctionTime',
           '工号': 'workNumber',
           '聘用形式': 'formOfEmployment',
           '部门': 'departmentName'
         }
         // 创建接口需要的数组
         const arr = []
         // 遍历表体数据
         excelData.results.forEach(item => {
           // 中转对象
           const obj = {}
           // 遍历枚举
           for (const key in userRelations) {
             // 如果是日期
             if (['timeOfEntry', 'correctionTime'].includes(userRelations[key])) {
               // 枚举的属性值作为属性名，数值名为标题属性名为当前枚举属性名的值
               // formatDate为格式化excel数字日期转成yyyy-mm-dd
               obj[userRelations[key]] = this.formatDate(item[key])
             } else {
               obj[userRelations[key]] = item[key]
             }
           }
           // 数组增加当前对象
           arr.push(obj)
         })
         // 发请求
         const res = await reqUploadEmployee(arr)
         console.log(res)
         if (res.success) {
           this.$message.success('导入成功')
           this.$router.back()
         } else {
           this.$message.error(res.message)
         }
       },
       // 日期格式化
       formatDate(numb, format) {
         const time = new Date((numb - 25567) * 24 * 3600000 - 5 * 60 * 1000 - 43 * 1000 - 24 * 3600000 - 8 * 3600000)
         const year = time.getFullYear() + ''
         const month = time.getMonth() + 1 + ''
         const date = time.getDate() + ''
         if (format && format.length === 1) {
           return year + format + (month < 10 ? '0' + month : month) + format + (date < 10 ? '0' + date : date)
         }
         return year + '-' + (month < 10 ? '0' + month : month) + '-' + (date < 10 ? '0' + date : date)
       }
     }
   }
   </script>
   
   <style lang="scss" scoped>
   .uploadExcelMain {
     padding-top: 150px;
   }
   </style>
   ```

3. 设置路由

   `@/router/index.js`

   ```js
   // 静态路由表
   export const constantRoutes = [
     {
       path: '/uploadExcel',
       component: Layout,
       children: [{
         path: '',
         name: 'uploadExcel',
         component: () => import('@/views/uploadExcel'),
         hidden: true
       }]
   
     }
   ]
   ```

   

### 导出员工

安装依赖：`xlsx` ，`file-saver` ，`script-loader`

导出功能模块

`@/vendor/Export2Excel.js`

```js
/* eslint-disable */
import { saveAs } from 'file-saver'
import * as XLSX from 'xlsx'

function generateArray(table) {
  var out = [];
  var rows = table.querySelectorAll('tr');
  var ranges = [];
  for (var R = 0; R < rows.length; ++R) {
    var outRow = [];
    var row = rows[R];
    var columns = row.querySelectorAll('td');
    for (var C = 0; C < columns.length; ++C) {
      var cell = columns[C];
      var colspan = cell.getAttribute('colspan');
      var rowspan = cell.getAttribute('rowspan');
      var cellValue = cell.innerText;
      if (cellValue !== "" && cellValue == +cellValue) cellValue = +cellValue;

      //Skip ranges
      ranges.forEach(function (range) {
        if (R >= range.s.r && R <= range.e.r && outRow.length >= range.s.c && outRow.length <= range.e.c) {
          for (var i = 0; i <= range.e.c - range.s.c; ++i) outRow.push(null);
        }
      });

      //Handle Row Span
      if (rowspan || colspan) {
        rowspan = rowspan || 1;
        colspan = colspan || 1;
        ranges.push({
          s: {
            r: R,
            c: outRow.length
          },
          e: {
            r: R + rowspan - 1,
            c: outRow.length + colspan - 1
          }
        });
      };

      //Handle Value
      outRow.push(cellValue !== "" ? cellValue : null);

      //Handle Colspan
      if (colspan)
        for (var k = 0; k < colspan - 1; ++k) outRow.push(null);
    }
    out.push(outRow);
  }
  return [out, ranges];
};

function datenum(v, date1904) {
  if (date1904) v += 1462;
  var epoch = Date.parse(v);
  return (epoch - new Date(Date.UTC(1899, 11, 30))) / (24 * 60 * 60 * 1000);
}

function sheet_from_array_of_arrays(data, opts) {
  var ws = {};
  var range = {
    s: {
      c: 10000000,
      r: 10000000
    },
    e: {
      c: 0,
      r: 0
    }
  };
  for (var R = 0; R != data.length; ++R) {
    for (var C = 0; C != data[R].length; ++C) {
      if (range.s.r > R) range.s.r = R;
      if (range.s.c > C) range.s.c = C;
      if (range.e.r < R) range.e.r = R;
      if (range.e.c < C) range.e.c = C;
      var cell = {
        v: data[R][C]
      };
      if (cell.v == null) continue;
      var cell_ref = XLSX.utils.encode_cell({
        c: C,
        r: R
      });

      if (typeof cell.v === 'number') cell.t = 'n';
      else if (typeof cell.v === 'boolean') cell.t = 'b';
      else if (cell.v instanceof Date) {
        cell.t = 'n';
        cell.z = XLSX.SSF._table[14];
        cell.v = datenum(cell.v);
      } else cell.t = 's';

      ws[cell_ref] = cell;
    }
  }
  if (range.s.c < 10000000) ws['!ref'] = XLSX.utils.encode_range(range);
  return ws;
}

function Workbook() {
  if (!(this instanceof Workbook)) return new Workbook();
  this.SheetNames = [];
  this.Sheets = {};
}

function s2ab(s) {
  var buf = new ArrayBuffer(s.length);
  var view = new Uint8Array(buf);
  for (var i = 0; i != s.length; ++i) view[i] = s.charCodeAt(i) & 0xFF;
  return buf;
}

export function export_table_to_excel(id) {
  var theTable = document.getElementById(id);
  var oo = generateArray(theTable);
  var ranges = oo[1];

  /* original data */
  var data = oo[0];
  var ws_name = "SheetJS";

  var wb = new Workbook(),
    ws = sheet_from_array_of_arrays(data);

  /* add ranges to worksheet */
  // ws['!cols'] = ['apple', 'banan'];
  ws['!merges'] = ranges;

  /* add worksheet to workbook */
  wb.SheetNames.push(ws_name);
  wb.Sheets[ws_name] = ws;

  var wbout = XLSX.write(wb, {
    bookType: 'xlsx',
    bookSST: false,
    type: 'binary'
  });

  saveAs(new Blob([s2ab(wbout)], {
    type: "application/octet-stream"
  }), "test.xlsx")
}

export function export_json_to_excel({
  multiHeader = [],
  header,
  data,
  filename,
  merges = [],
  autoWidth = true,
  bookType = 'xlsx'
} = {}) {
  /* original data */
  filename = filename || 'excel-list'
  data = [...data]
  data.unshift(header);

  for (let i = multiHeader.length - 1; i > -1; i--) {
    data.unshift(multiHeader[i])
  }

  var ws_name = "SheetJS";
  var wb = new Workbook(),
    ws = sheet_from_array_of_arrays(data);

  if (merges.length > 0) {
    if (!ws['!merges']) ws['!merges'] = [];
    merges.forEach(item => {
      ws['!merges'].push(XLSX.utils.decode_range(item))
    })
  }

  if (autoWidth) {
    /*设置worksheet每列的最大宽度*/
    const colWidth = data.map(row => row.map(val => {
      /*先判断是否为null/undefined*/
      if (val == null) {
        return {
          'wch': 10
        };
      }
      /*再判断是否为中文*/
      else if (val.toString().charCodeAt(0) > 255) {
        return {
          'wch': val.toString().length * 2
        };
      } else {
        return {
          'wch': val.toString().length
        };
      }
    }))
    /*以第一行为初始值*/
    let result = colWidth[0];
    for (let i = 1; i < colWidth.length; i++) {
      for (let j = 0; j < colWidth[i].length; j++) {
        if (result[j]['wch'] < colWidth[i][j]['wch']) {
          result[j]['wch'] = colWidth[i][j]['wch'];
        }
      }
    }
    ws['!cols'] = result;
  }

  /* add worksheet to workbook */
  wb.SheetNames.push(ws_name);
  wb.Sheets[ws_name] = ws;

  var wbout = XLSX.write(wb, {
    bookType: bookType,
    bookSST: false,
    type: 'binary'
  });
  saveAs(new Blob([s2ab(wbout)], {
    type: "application/octet-stream"
  }), `${filename}.${bookType}`);
}

```

`@/vendor/Export2Zip.js`

```js
/* eslint-disable */
import { saveAs } from 'file-saver'
import JSZip from 'jszip'

export function export_txt_to_zip(th, jsonData, txtName, zipName) {
  const zip = new JSZip()
  const txt_name = txtName || 'file'
  const zip_name = zipName || 'file'
  const data = jsonData
  let txtData = `${th}\r\n`
  data.forEach((row) => {
    let tempStr = ''
    tempStr = row.toString()
    txtData += `${tempStr}\r\n`
  })
  zip.file(`${txt_name}.txt`, txtData)
  zip.generateAsync({
    type: "blob"
  }).then((blob) => {
    saveAs(blob, `${zip_name}.zip`)
  }, (err) => {
    alert('导出失败')
  })
}
```

设置导出按钮事件

`@/views/employees/index.vue`

```vue
<template>
	<el-button
 		type="danger"
        size="mini"
        @click="handleExport"
    >普通excel导出</el-button>
</<template>	

<script>
export default {
  methods: {
    // 导出员工按钮回调
    async handleExport() {
      // 获取所有员工数据
      const {
        data: { rows }
      } = await reqGetEmployeeList(1, this.total)
      // 表头
      const headersArr = [
        '姓名',
        '手机号',
        '入职日期',
        '聘用形式',
        '转正日期',
        '工号',
        '部门'
      ]
      // 枚举：用于对应后台的数据
      const headersRelations = {
        姓名: 'username',
        手机号: 'mobile',
        入职日期: 'timeOfEntry',
        聘用形式: 'formOfEmployment',
        转正日期: 'correctionTime',
        工号: 'workNumber',
        部门: 'departmentName'
      }
      // 处理后台获取的数据
      const dataArr = this.formatJson(rows, headersRelations)

      // 懒加载 ：导出操作
      import('@/vendor/Export2Excel').then((excel) => {
        excel.export_json_to_excel({
          header: headersArr, // 表头 必填
          data: dataArr, // 具体数据 必填
          filename: '人员列表', // 非必填
          autoWidth: true, // 非必填
          bookType: 'xlsx' // 非必填
        })
      })
    },
    // 处理导出数据的
    formatJson(rows, headersRelations) {
      // 空数组
      const arr = []
      // 遍历服务器数据
      rows.forEach((item) => {
        // 中转数组
        const rowArr = []
        // 遍历枚举
        for (const key in headersRelations) {
          // 中转数组push：按照枚举顺序推送
          rowArr.push(item[headersRelations[key]])
        }
        // 推送当前员工数组
        arr.push(rowArr)
      })
      return arr
    }
  }
}
</script>

```

### 员工详情

> 利用`el-tabs`制作分页，显示不同页面的内容

1）建立详情页面

`@/views/employees/detail/index.vue`：设置页面

```vue
<template>
  <div class="detail_main">
    <el-card>
      <el-tabs v-model="activeName">
        <el-tab-pane label="登陆账户设置" name="accountSetting"></el-tab-pane>
        <el-tab-pane label="个人详情" name="accountDetail">登录</el-tab-pane>
        <el-tab-pane label="岗位信息" name="position">登录</el-tab-pane>
      </el-tabs>
    </el-card>
  </div>
</template>

<script>
import { getDetailProfile, reqSaveUserDetailById } from '@/api'
export default {
  data() {
    return {
      // 活动页签
      activeName: 'accountSetting',
    }
  },
}
</script>

<style lang="scss" scoped>
.detail_main {
  padding: 25px;
  #pane-accountSetting {
    margin: 30px 0 0 140px;
  }
}
</style>

```

2）设置路由

`@/router/modules/employees.js`

```js
import Layout from '@/layout'

export default {
  path: '/employees',
  component: Layout,
  children: [
    {
      path: '',
      name: 'employees',
      component: () => import('@/views/employees'),
      meta: { title: '员工', icon: 'people' }

    },
    {
      path: '/employees/detail',
      component: () => import('@/views/employees/detail'),
      hidden: true
    }
  ]
}
```

3）员工列表`查看`按钮跳转携带id，以备详情页使用

`@/view/employees/index.vue`

```vue
<el-link type="primary" :underline="false" @click="$router.push(`/employees/detail?id=${row.id}`)">查看</el-link>
```

3）`登录账户设置`

- 表单铺设 ：表单校验、表单绑定
- `created`钩子函数：获取当前id用户数据，表单赋值
- 更新按钮事件

```vue
<el-tab-pane label="登陆账户设置" name="accountSetting">
  <el-form
    ref="accountSettingRef"
    :model="accountSettingForm"
    :rules="rules"
  >
    <el-form-item label="用户名" prop="username" label-width="100px">
      <el-input
        v-model="accountSettingForm.username"
        style="width: 300px"
      />
    </el-form-item>
    <el-form-item label="密码" prop="password" label-width="100px">
      <el-input
        v-model="accountSettingForm.password"
        type="password"
        style="width: 300px"
      />
    </el-form-item>
    <el-form-item label-width="100px">
      <el-button type="primary" @click="handleUpdate">更新</el-button>
    </el-form-item>
  </el-form>
</el-tab-pane>

<script>
import { getDetailProfile, reqSaveUserDetailById } from '@/api'
export default {
  data() {
    return {
      // 活动页签
      activeName: 'accountSetting',
      // 账户设置form
      accountSettingForm: {},
      // 表单校验
      rules: {
        username: [
          { required: true, message: '姓名不能为空', trigger: 'blur' }
        ],
        password: [
          { required: true, message: '密码不能为空', trigger: 'blur' },
          { max: 20, min: 6, message: '最小6位，最大20位', trigger: 'blur' }
        ]
      }
    }
  },
  created() {
    this.getDetailProfile()
  },
  methods: {
    // 更新按钮
    handleUpdate() {
      this.$refs.accountSettingRef.validate(async(vali) => {
        if (vali) {
          const res = await reqSaveUserDetailById(this.accountSettingForm)
          console.log(res)
          if (res.success) {
            this.$message.success('更新成功')
          } else {
            this.$message.error(res.message)
          }
        }
      })
    },
    // 获取用户基本信息
    async getDetailProfile() {
      const { data } = await getDetailProfile(this.$route.query.id)
      this.accountSettingForm = data
    }
  }
}
</script>
```

## 项目中遇到的问题：

1. 在验证token的时候，判断没有token跳转login时，导致无限循环白屏，应该没有token设置如果是login时直接next() 不要跳转login
2. 面包屑组件封装时，监听route的url变化时加入面包屑数组，在后期出现携带id的相同页面时，除了问题，导致数组重复。最后是将这些重复的id的页面不加入面包屑数组中
3. 在导入excel数据时，有时间的格式的时候，因为excel默认是数值型的日期格式，导致导入的时间显示有问题，最后是写了一个方法，在导入前格式化了日期数据
