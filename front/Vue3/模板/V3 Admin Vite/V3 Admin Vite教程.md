# V3 Admin Vite

中文官方文档：https://juejin.cn/post/7089377403717287972

以下是新手教程：

## 下载、运行

```sh
# 配置
1. 一键安装 .vscode 目录中推荐的插件
2. node 版本 18.x 或 20+
3. pnpm 版本 8.x 或最新版

# 克隆项目
git clone https://github.com/un-pany/v3-admin-vite.git

# 进入项目目录
cd v3-admin-vite

# 安装依赖
pnpm i

# 启动服务
pnpm dev
```

## 接口、跨域、打包

如何使用该项目对接你自己的后端接口、如何处理接口跨域问题、如何正确的打包部署前端静态文件。

### 设置后端接口

前端所有的请求最终都是通过 Axios 来发送的，我们可以找到封装 Axios 的文件，看见后端接口的 `baseURL` 是 `import.meta.env.VITE_BASE_API`

![baseURL](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f8f85842a20f43c2940d7bbb6b6cba4d~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp?)

然后我们可以在 `.env` 配置文件中找到定义 `VITE_BASE_API` 的地方：

![VITE_BASE_API](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3871161ed651459ca040fbf25e058034~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp?)

具体到某一个API，如该接口的 url 是 `users/login`，也就意味着，在调用该接口时，最终请求的路径将会是：baseURL + 该接口的 url = `http://localhost:3333/api/v1/users/login`

### 跨域

#### 反向代理

![proxy](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a64fe4d5f0cd43f2b1060cf076767a68~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp?)

匹配 `/api/v1` 这个路径，假如发送的请求包含了这个路径，那么就将进行反向代理，将请求代理到 `target` 字段配置的路径。

***直接采用这种反向代理的好处就是前端调用后端接口时不会产生跨域问题，但要记得这只是开发环境配置好了反向代理，以后部署前端到线上环境的时候，需要采用 Nginx 或其他工具来实现线上环境的反向代理。***

#### CORS

就是将你的 `VITE_BASE_API` 配置填写为完整的绝对路径：

```sh
## 后端接口公共路径（如果解决跨域问题采用 CORS 就需要写全路径）
VITE_BASE_API = 'https://mock.mengxuegu.com/mock/63218b5fb4c53348ed2bc212/api/v1'
```

### 打包

#### 接口公共路径

打包的话比较简单，我们以正式环境配置 `.env.production` 为例。由于我们打包后部署的服务器上没有 `Nginx` 等工具帮助我们实现反向代理，所以我们就必须采用 CORS 的方式解决跨域问题，就需要将 `VITE_BASE_API` 写完整，也就是：

```sh
## 后端接口公共路径（如果解决跨域问题采用 CORS 就需要写全路径）
VITE_BASE_API = 'https://mock.mengxuegu.com/mock/63218b5fb4c53348ed2bc212/api/v1'
```

#### 路由模式

然后选择一种路由方式（`hash` 或 `html5`），模板本身默认是 `hash 模式`，如果你想切换为 html5 模式的话，更改 `VITE_ROUTER_HISTORY` 配置即可：

```sh
## 路由模式 hash 或 html5
VITE_ROUTER_HISTORY = 'hash'
```

#### 打包路径

最后再设置一下打包路径 `VITE_PUBLIC_PATH` 即可。模板项目本身是需要部署到这个域名下：`https://un-pany.github.io/v3-admin-vite/`，所以我们需要这么填写：

```sh
## 打包路径（就是网站前缀，例如部署到 https://un-pany.github.io/v3-admin-vite/ 域名下，就需要填写 /v3-admin-vite/）
VITE_PUBLIC_PATH = '/v3-admin-vite/'
```

依此类推，

假如是要部署到 `https://xxx.com/yyy/` 下，那么就需要填写 `VITE_PUBLIC_PATH = '/yyy/'`

假如是要部署到 `https://xxx.com/` 下，那么就需要填写 `VITE_PUBLIC_PATH = '/'`

#### 运行命令

打开 `package.json` 文件，可以看见所有该项目内置的命令：

![package.json](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1ceff4b22eac4435bd6a5cb0793c0e72~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp?)

和打包相关的命令是 `build`，我们以打包正式环境为例，就要运行下面的命令：

```sh
pnpm build:prod
```

这个命令就会自动去读取我们前文配置好的 `.env.production` 文件，

而 `pnpm build:stage` 会自动读取 `.env.staging` 文件，代表的是预发布环境。

打包完成后，就可以在目录下，看见一个名为 `dist` 的静态资源文件夹，这整个文件夹就是需要丢到前端服务器上去的东西。

![dist](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3e7973a6e6824b028c9c7a94ebe2662e~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp?)

## 登录模块（涉及 API、Axios、Pinia、路由守卫、鉴权）

通过登录模块教会你配置 api 接口、在页面上调用接口发起请求、Pinia 保存用户信息、经过路由守卫的拦截，成功跳转到首页、Token 鉴权，判断是否退出登录。

### 配置登录接口

#### 建立目录结构

我们在 `@/api` 目录下找到 `login` 文件夹，没有的话就需要新建一个，这个文件夹即代表了登录模块（**注意是登录模块，不止是登录接口。如果该模块下还有子模块的话，你可以继续往下面再建立子模块的文件夹**）。然后再在 login 文件夹里面再建立一个 `types` 文件夹（这个文件夹专门放置和登录模块相关的 `TS 类型`）和 `index.ts`。

![@/api/login](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ddeb13d52fed498a992aa7d184af1dd3~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=293&h=176&s=4278&e=png&b=282829)

**举一反三**，如果复杂一点，假如我们有一个模块叫系统管理 system，里面有两个子模块，分别叫用户管理 user、角色管理 role，那么我们建立的目录大致就应该长这个样子：

![@/api/system](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a5cee0f1f9484d88a4cd0840f8f0694b~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=301&h=248&s=5598&e=png&b=272728)

#### 编写 TS 类型(login.ts)

*编写接口的 TS 类型，我们必须提前拿到接口的请求参数和响应数据的格式*，这里就需要你的后端同事提供接口文档了。

这个项目本身的登录接口的类型定义如下：

**请求数据类型 ILoginRequestData**

```ts
export interface LoginRequestData {
  /** admin 或 editor */
  username: "admin" | "editor"
  /** 密码 */
  password: string
  /** 验证码 */
  code: string
}
```

**响应数据类型 LoginResponseData**

```ts
export type LoginResponseData = ApiResponseData<{ token: string }>
```

这里的意思是，将类型 `{ token: string }` 作为泛型传递给类型 `ApiResponseData`，ApiResponseData 这个类型作为一个全局类型，被定义在 `@/types/api.d.ts` 文件里。

#### 编写接口(index.ts)

发送请求是通过封装好的 Axios，所以第一步就是导入相关的方法

```ts
import { request } from "@/utils/service"
```

还需要上文写好的登录接口的类型，将其导入进来

```ts
import type * as Login from "./types/login"
```

然后就可以开始写接口了：

```ts
/** 登录并返回 Token */
export function loginApi(data: Login.LoginRequestData) {
  return request<Login.LoginResponseData>({
    url: "users/login",
    method: "post",
    data
  })
}
```

这表示登录接口的函数名为 `loginApi`，它接受一个参数 data，类型为 `LoginRequestData`。

`request<Login.LoginResponseData>` 则表示的是待会接口响应成功的数据类型为 `LoginResponseData`。

`url` 代表接口地址，`method` 代表接口方法（get/post/put/delete），`data` 表示请求体数据（如果是 get 请求，则要换成 `params`）

**更多关于 Axios 的参数，你就得去 [官网](https://link.juejin.cn/?target=https%3A%2F%2Faxios-http.com%2F) 学习了**，比如工作中我们偶尔会遇到后端需要一些奇奇怪怪格式的数据，普通的 JSON 满足不了的情况下，可能就需要添加 `transformRequest` 属性来 `qs 格式化` 请求数据...

### 调用登录接口

#### 点击登录按钮

来到登录页面上，首先是找到登录按钮将调用的函数是 handleLogin：

```ts
const handleLogin = () => {
  loginFormRef.value?.validate((valid: boolean, fields) => {
    if (valid) {
      loading.value = true
      useUserStore()
        .login(loginFormData)
        .then(() => {
          router.push({ path: "/" })
        })
        .catch(() => {
          createCode()
          loginFormData.password = ""
        })
        .finally(() => {
          loading.value = false
        })
    } else {
      console.error("表单校验不通过", fields)
    }
  })
}
```

这里的 `loginFormRef.value?.validate` 是校验登录表单，`useUserStore()` 是写好的 `状态管理器 Pinia` 的 `Store` 待会我们下文就会讲到。

调用该 Store 的 `login action`，并传入 loginFormData 参数（用户名、密码、验证码）即可。

`login action` 返回值是一个 Promise，所以我们后面链式跟一个 `.then` 、 `.catch` 和 `.finally`，接口调用成功则会执行 `.then` （跳转到首页），如果途中发生错误，则会执行 `.catch`，而无论什么情况都会执行 `.finally`

#### 状态管理

由于点击登录按钮触发了 `useUserStore` 的 `login action`，我们现在顺着逻辑找到它

![store/modules/user](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3d4630a01f9e49f38625ff31d1dd57ab~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=839&h=1199&s=117418&e=png&b=1e1e1e)

可以看到，在 `useUserStore` 里，我们引入了上文写好的登录接口 `loginApi`：

```ts
import { loginApi, getUserInfoApi } from "@/api/login"
```

然后在 `login action` 中调用这个 loginApi 并传入对应参数（**如果这里参数传递错误，那么 TS 就会报错提醒我们，因为我们在上文中定义接口的时候已经约束了类型**）

调用登录接口成功时，我们将接口返回的响应数据 `res` 中的 `token` 分别保存到 `cookie`（对应语句 setToken(res.data.token)）和 `当前 Store`（对应语句 token.value = res.data.token） 中，如果接口失败，则直接 `reject`（[ES6 Promise 知识](https://link.juejin.cn?target=https%3A%2F%2Fes6.ruanyifeng.com%2F%23docs%2Fpromise)） 即可

**如果这里执行了 .then 那么登录页面也将执行 .then，也就会开始跳转路由到首页，那么就会触发路由守卫**

#### 路由守卫

路径：`@/router/permission.ts`

```ts
import router from "@/router"
import { useUserStoreHook } from "@/store/modules/user"
import { usePermissionStoreHook } from "@/store/modules/permission"
import { ElMessage } from "element-plus"
import { setRouteChange } from "@/hooks/useRouteListener"
import { useTitle } from "@/hooks/useTitle"
import { getToken } from "@/utils/cache/cookies"
import routeSettings from "@/config/route"
import isWhiteList from "@/config/white-list"
import NProgress from "nprogress"
import "nprogress/nprogress.css"

const { setTitle } = useTitle()
NProgress.configure({ showSpinner: false })

router.beforeEach(async (to, _from, next) => {
  NProgress.start()
  const userStore = useUserStoreHook()
  const permissionStore = usePermissionStoreHook()
  const token = getToken()

  // 判断该用户是否已经登录
  if (!token) {
    // 如果在免登录的白名单中，则直接进入
    if (isWhiteList(to)) return next()
    // 其他没有访问权限的页面将被重定向到登录页面
    return next("/login")
  }

  // 如果已经登录，并准备进入 Login 页面，则重定向到主页
  if (to.path === "/login") {
    return next({ path: "/" })
  }

  // 如果用户已经获得其权限角色
  if (userStore.roles.length !== 0) return next()

  // 否则要重新获取权限角色
  try {
    await userStore.getInfo()
    // 注意：角色必须是一个数组！ 例如: ["admin"] 或 ["developer", "editor"]
    const roles = userStore.roles
    // 生成可访问的 Routes
    routeSettings.dynamic ? permissionStore.setRoutes(roles) : permissionStore.setAllRoutes()
    // 将 "有访问权限的动态路由" 添加到 Router 中
    permissionStore.addRoutes.forEach((route) => router.addRoute(route))
    // 确保添加路由已完成
    // 设置 replace: true, 因此导航将不会留下历史记录
    next({ ...to, replace: true })
  } catch (err: any) {
    // 过程中发生任何错误，都直接重置 Token，并重定向到登录页面
    userStore.resetToken()
    ElMessage.error(err.message || "路由守卫过程发生错误")
    next("/login")
  }
})

router.afterEach((to) => {
  setRouteChange(to)
  setTitle(to.meta.title)
  NProgress.done()
})
```

路由守卫做了什么事：

1. 判断用户是否登录，没登录则只能进入白名单页面，比如登录页
2. 如果已经登录，那么将不允许进入登录页
3. 如果已经登录，那么还要检查是否拿到用户角色，如果没有，则要调用用户详情接口
4. 如果开启了动态路由功能，就根据角色去过滤动态路由；如果没有开启动态路由功能，则生成所有路由
5. 不管什么情况，一旦发生错误，就重置 Token，并重定向到登录页

**一般情况下，不建议大家更改路由守卫逻辑，如果必须要更改，请大家一定熟练掌握原本的逻辑以及 vue-router 路由守卫本身的知识点**

这里如果通过路由守卫的检查后，我们就能正常跳转到首页了。

### 鉴权

后续所有的操作，我都将携带保存在前端的 `token` 去调用接口，`token` 将是后端服务判断当前请求合不合法的依据。

如何携带，项目本身已经写在 Axios 的封装里面了

![axios](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d3682dbd5f7744eda74b7b3dc9c12872~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=909&h=766&s=58876&e=png&b=1e1e1e)

假如 `token` 已经过期后，理论上接口会给我们抛出一个 `http code 401` 的错误，我们只需要在响应拦截器里重定向到登录页即可

![axios-error](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9aca133d3dfc4d09949e12478f2fe4b0~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=875&h=1027&s=77198&e=png&b=1f1f1f)

## 平台配置（涉及布局、路由菜单、全局样式配置）

如何使用该平台内置的一些配置，比如和布局相关的隐藏标签栏、和路由相关的路由缓存、和样式相关的修改全局颜色。

### 路由菜单

路由的定义以及配置在 `@/router/index.ts` 文件里，比如登录路由配置：

```ts
{
  path: "/login",
  component: () => import("@/views/login/index.vue"),
  meta: {
    hidden: true
  }
}
```

首先我们要了解一个基本的知识点：我们平台自定义的配置项都在 `meta` 属性下，而其他的比如 `path`、`component`、`redirect`、`children`、`name` 这些属性，是 `vue-router` 自带的，如果你还不了解它们各自代表什么意思，那你应该去 [官网](https://link.juejin.cn?target=https%3A%2F%2Frouter.vuejs.org%2Fzh%2F) 查阅一下。

平台在 `meta` 属性下已经内置了这么多配置项了：

```ts
// 设置 noRedirect 的时候该路由在面包屑导航中不可被点击
redirect: 'noRedirect'

// 动态路由：必须设定路由的名字，一定要填写不然重置路由可能会出问题
// 如果要在 tags-view 中展示，也必须填 name
name: 'router-name'

meta: {
  // 设置该路由在侧边栏和面包屑中展示的名字
  title: 'title'
  
  // 设置该路由的图标，记得将 svg 导入 @/icons/svg
  svgIcon: 'svg name'
  
  // 设置该路由的图标，直接使用 Element Plus 的 Icon（与 svgIcon 同时设置时，svgIcon 将优先生效）
  elIcon: 'element-plus icon name'
  
  // 默认 false，设置 true 的时候该路由不会在侧边栏出现
  hidden: true
  
  // 设置该路由进入的权限，支持多个权限叠加（动态路由才需要设置）
  roles: ['admin', 'editor']
  
  // 默认 true，如果设置为 false，则不会在面包屑中显示
  breadcrumb: false
  
  // 默认 false，如果设置为 true，它则会固定在 tags-view 中
  affix: true
  
  // 当一个路由下面的 children 声明的路由大于1个时，自动会变成嵌套的模式
  // 只有一个时，会将那个子路由当做根路由显示在侧边栏
  // 若想不管路由下面的 children 声明的个数都显示你的根路由
  // 可以设置 alwaysShow: true，这样就会忽略之前定义的规则，一直显示根路由
  alwaysShow: true

  // 示例: activeMenu: "/xxx/xxx"
  // 当设置了该属性进入路由时，则会高亮 activeMenu 属性对应的侧边栏
  // 该属性适合使用在有 hidden: true 属性的路由上
  activeMenu: '/dashboard'
  

  // 是否缓存该路由页面
  // 默认为 false，为 true 时代表需要缓存，此时该路由和该页面都需要设置一致的 Name
  keepAlive: false
}
```

为了让编辑器对这些配置项有类型提示，平台还对它们进行了 TS 定义，放在了 `@/types/vue-router.d.ts` 文件下，如果你需要改造或者新增配置项，那你也应该同步修改这个文件！

它们的 TS 类型定义如下：

```ts
/**
     * 设置该路由在侧边栏和面包屑中展示的名字
     */
    title?: string
    /**
     * 设置该路由的图标，记得将 svg 导入 @/icons/svg
     */
    svgIcon?: string
    /**
     * 设置该路由的图标，直接使用 Element Plus 的 Icon（与 svgIcon 同时设置时，svgIcon 将优先生效）
     */
    elIcon?: string
    /**
     * 默认 false，设置 true 的时候该路由不会在侧边栏出现
     */
    hidden?: boolean
    /**
     * 设置能进入该路由的角色，支持多个角色叠加
     */
    roles?: string[]
    /**
     * 默认 true，如果设置为 false，则不会在面包屑中显示
     */
    breadcrumb?: boolean
    /**
     * 默认 false，如果设置为 true，它则会固定在 tags-view 中
     */
    affix?: boolean
    /**
     * 当一个路由下面的 children 声明的路由大于 1 个时，自动会变成嵌套的模式，
     * 只有一个时，会将那个子路由当做根路由显示在侧边栏，
     * 若想不管路由下面的 children 声明的个数都显示你的根路由，
     * 可以设置 alwaysShow: true，这样就会忽略之前定义的规则，一直显示根路由
     */
    alwaysShow?: boolean
    /**
     * 示例: activeMenu: "/xxx/xxx"，
     * 当设置了该属性进入路由时，则会高亮 activeMenu 属性对应的侧边栏。
     * 该属性适合使用在有 hidden: true 属性的路由上
     */
    activeMenu?: string
    /**
     * 是否缓存该路由页面
     * 默认为 false，为 true 时代表需要缓存，此时该路由和该页面都需要设置一致的 Name
     */
    keepAlive?: boolean
```

#### 设置缓存

设置路由缓存必须同时满足这四个条件：

1. 路由 `keepAlive` 为 true
2. 路由有 `Name`
3. 页面有 `Name`
4. 路由和页面 Name `保持一致`

以表格路由为例：

```ts
{
path: "/table",
component: Layout,
redirect: "/table/element-plus",
name: "Table",
meta: {
  title: "表格",
  elIcon: "Grid"
},
children: [
  {
    path: "element-plus",
    component: () => import("@/views/table/element-plus/index.vue"),
    name: "ElementPlus",
    meta: {
      title: "Element Plus",
      keepAlive: true
    }
  },
  {
    path: "vxe-table",
    component: () => import("@/views/table/vxe-table/index.vue"),
    name: "VxeTable",
    meta: {
      title: "Vxe Table",
      keepAlive: true
    }
  }
]
}
```

可以看见两个路由的 Name 分别是 `ElementPlus` 和 `VxeTable`，我们还需要去对应的页面上配置相同的 Name：

![table/element-plus](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/197c6dc309834e548d037511b0cdf509~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp?)



### 全局样式

全局样式相关的的文件，全都在 `@/styles` 目录下：

`vxe-table.scss`：这里可以写样式来覆盖 vxe-table 原本的样式

`element-plus.scss`：这里可以写样式来覆盖 element-plus 原本的样式

`transition.scss`： 这里可以写动画相关的样式

`mixins.scss`：这里可以写和 scss mixin 相关的样式

`variables.css`：**这里是本项目内置的一些比较重要的和布局、颜色相关的全局样式**

`index.scss`：这里是所有样式的入口，也可以写样式来覆盖原生 html 的样式

`theme`： **这里是多主题模式相关的样式文件，目前内置了黑暗模式、深蓝色模式**

## 前端权限（涉及角色、动态路由、权限函数、权限指令）

如何通过用户的 “角色” 字段来进行页面级别的权限控制（动态路由的挂载）；以及通过角色字段进行内容级别的权限控制（权限函数、权限指令）

### 页面权限

#### 动态路由

`@/router/index.ts` 就是用来存放常驻路由和动态路由的文件，如图所示：

![router/index.ts](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/708d49a7f7294ee1babb74bdd640cfa5~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=598&h=461&s=18726&e=png&b=1e1e1e)

不管什么情况下，都需要挂载的路由，我们就存放在 `constantRoutes` 数组下，比如登录页、首页；需要用户登录并根据角色字段来判断是否有权限的路由，我们就放在 `asyncRoutes` 数组下，并且要为该路由配置好 `roles` 和 `name` 属性

下面源码就是项目中写好的一个动态路由示例，注意看它是有 `roles` 和 `name` 属性的：

```ts
{
  path: "/permission",
  component: Layout,
  redirect: "/permission/page",
  name: "Permission", // 不要忘了写
  meta: {
    title: "权限管理",
    svgIcon: "lock",
    roles: ["admin", "editor"], // 可以在根路由中设置角色
    alwaysShow: true // 将始终显示根菜单
  },
  children: [
    {
      path: "page",
      component: () => import("@/views/permission/page.vue"),
      name: "PagePermission", // 不要忘了写
      meta: {
        title: "页面权限",
        roles: ["admin"] // 或者在子导航中设置角色
      }
    },
    {
      path: "directive",
      component: () => import("@/views/permission/directive.vue"),
      name: "DirectivePermission", // 不要忘了写
      meta: {
        title: "指令权限" // 如果未设置角色，则表示：该页面不需要权限，但会继承根路由的角色
      }
    }
  ]
}
```

#### 开启动态路由功能

*项目默认是开启状态*

写好了动态路由后，我们在 `@/config/route.ts` 文件中可以找到是否开启动态路由的开关，源码如下，只需要将下面代码中的 `routeSettings.dynamic` 设置为 `true` 就可以开启动态路由功能：

```ts
/** 路由配置 */
interface RouteSettings {
  /**
   * 是否开启动态路由功能？
   * 1. 开启后需要后端配合，在查询用户详情接口返回当前用户可以用来判断并加载动态路由的字段（该项目用的是角色 roles 字段）
   * 2. 假如项目不需要根据不同的用户来显示不同的页面，则应该将 dynamic: false
   */
  dynamic: boolean
  /** 当动态路由功能关闭时：
   * 1. 应该将所有路由都写到常驻路由里面（表明所有登录的用户能访问的页面都是一样的）
   * 2. 系统自动给当前登录用户赋值一个没有任何作用的默认角色
   */
  defaultRoles: Array<string>
  /**
   * 是否开启三级及其以上路由缓存功能？
   * 1. 开启后会进行路由降级（把三级及其以上的路由转化为二级路由）
   * 2. 由于都会转成二级路由，所以二级及其以上路由有内嵌子路由将会失效
   */
  thirdLevelRouteCache: boolean
}

const routeSettings: RouteSettings = {
  dynamic: true,
  defaultRoles: ["DEFAULT_ROLE"],
  thirdLevelRouteCache: false
}

export default routeSettings
```

开启以后，主要是作用于路由守卫 `@/router/permission.ts` 中的这样一段代码：

```ts
await userStore.getInfo()
// 注意：角色必须是一个数组！ 例如: ["admin"] 或 ["developer", "editor"]
const roles = userStore.roles
// 生成可访问的 Routes
routeSettings.dynamic ? permissionStore.setRoutes(roles) : permissionStore.setAllRoutes()
// 将 "有访问权限的动态路由" 添加到 Router 中
permissionStore.addRoutes.forEach((route) => router.addRoute(route))
```

简单来说就是：**如果开启该功能，那么通过用户详情接口拿到用户角色数组后，调用 `setRoutes` 根据角色去过滤动态路由，否则调用 `setAllRoutes` 拿到所有路由，然后再通过 `router.addRoute()` 挂载过滤之后的动态路由**

#### 关闭动态路由功能

假如，你选择**关闭动态路由**功能，那么你需要记得将**所有路由**都写在常驻路由数组里面（虽然写在动态路由数组里也行，因为程序兼容了这种偷懒）

这样的话，所有登陆的用户能访问的页面都是一模一样的了

### 内容权限

#### 权限函数

`@/utils/permission.ts` 文件里，有一个 `checkPermission` 权限判断函数：

```ts
import { useUserStoreHook } from "@/store/modules/user"

/** 权限判断函数 */
export const checkPermission = (permissionRoles: string[]): boolean => {
  if (Array.isArray(permissionRoles) && permissionRoles.length > 0) {
    const { roles } = useUserStoreHook()
    return roles.some((role) => permissionRoles.includes(role))
  } else {
    console.error("need roles! Like checkPermission(['admin','editor'])")
    return false
  }
}
```

向该函数传递一个权限数组，然后它会去对比当前登录用户的角色数组，如果能匹配上，就返回 `true`

使用方法非常简单，`checkPermission` 函数配合 `v-if` 即可：

```vue
// 引入
import { checkPermission } from "@/utils/permission"

// 使用
<el-button v-if="checkPermission(['admin'])">按钮</el-button>
```

更多详细的使用案例，可见 `@/views/permission/directive.vue` 页面

#### 权限指令

`@/directives/permission/index.ts` 文件里，写好了权限判断指令 `v-permission`：

```ts
import { type Directive } from "vue"
import { useUserStoreHook } from "@/store/modules/user"

/** 权限指令 */
export const permission: Directive = {
  mounted(el, binding) {
    const { value: permissionRoles } = binding
    const { roles } = useUserStoreHook()
    if (Array.isArray(permissionRoles) && permissionRoles.length > 0) {
      const hasPermission = roles.some((role) => permissionRoles.includes(role))
      // hasPermission || (el.style.display = "none") // 隐藏
      hasPermission || el.parentNode?.removeChild(el) // 销毁
    } else {
      throw new Error(`need roles! Like v-permission="['admin','editor']"`)
    }
  }
}
```

向该指令传递一个权限数组，然后它会去对比当前登录用户的角色数组，如果不能匹配上，就通过 CSS `style.display = "none"` 将其隐藏或直接销毁

`v-permission` 已经通过 `app.directive()` 挂载完成，可以直接在 `template` 中直接使用：

```vue
<el-button v-permission="['admin']">按钮</el-button>
```

更多详细的使用案例，可见 `@/views/permission/directive.vue` 页面

### 后端返回动态路由？

## 前端项目规范

该项目是基于哪些主流的社区规范来开发的，并做了哪些和代码规范相关的配置。

https://juejin.cn/post/7231771821832618043

### 命名规范

因为 [Vue 风格指南](https://link.juejin.cn?target=https%3A%2F%2Fv2.cn.vuejs.org%2Fv2%2Fstyle-guide%2F%23%E5%8D%95%E6%96%87%E4%BB%B6%E7%BB%84%E4%BB%B6%E6%96%87%E4%BB%B6%E5%90%8D%E7%9A%84%E5%A4%A7%E5%B0%8F%E5%86%99%E5%BC%BA%E7%83%88%E6%8E%A8%E8%8D%90) 中有提到：**单文件组件文件名应该要么始终是单词大写开头 (PascalCase)，要么始终是横线连接 (kebab-case)**

本 [V3 Admin Vite](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fun-pany%2Fv3-admin-vite) 项目的前身，也就是 [V3 Admin](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fun-pany%2Fv3-admin) 项目，在早期的时候，就是全采用的 **短横线连接** 的命名方式，但是现在已经将 Component 组件命名改为 **大驼峰** 命名方式。

#### Components

所有的 Component，也就是组件，都采用单词大写开头的 **大驼峰** 命名方式 (PascalCase)。

比如在 `@/components` 目录下的公共组件：

- `@/components/Screenfull/index.vue`
- `@/components/SvgIcon/index.vue`
- `@/components/ThemeSwitch/index.vue`

看上面的例子，也就是说，如果是 index.vue，那么 `index.vue 是不用大驼峰命名` 的。

如果不是 index.vue，比如下面这个 NotifyList.vue 组件，那么就需要 **大驼峰** 命名：

- `@/components/Notify/index.vue`
- `@/components/Notify/NotifyList.vue`

即便是在 views 目录下只属于某个页面的组件，也同样需要遵守**大驼峰**命名：

- `@/views/error-page/components/ErrorPageLayout.vue`
- `@/views/permission/components/SwitchRoles.vue`

#### Views

放在 `@/views` 目录下代表着页面和路由的 .vue 文件或目录，它们一律采用 **短横线连接 (kebab-case)** 的命名方式，比如：

- `@/views/table/element-plus/index.vue`
- `@/views/table/vxe-table/index.vue`
- `@/views/error-page/404.vue`
- `@/views/error-page/403.vue`
- `@/views/hook-demo/use-fetch-select.vue`
- `@/views/hook-demo/use-fullscreen-loading.vue`

那，为什么 Component 采用大驼峰，View 采用短横线？

主要因为这两点：

- 能更好的区分 Component 和 View
- 导入 Component 时，采用的就是大驼峰方式
- 符合社区主流命名习惯

#### Hooks / Composables

采用 **小驼峰 (camelCase)**，比如：

- `@/hooks/useTheme.ts`
- `@/layout/hooks/useResize.ts`

#### Props

在声明 prop 的时候，其命名应该始终采用 **小驼峰 (camelCase)**，而在模板和 JSX 中应该始终使用 **短横线连接 (kebab-case)**，比如：

以 `@/components/Screenfull/index.ts` 组件为例：

声明定义 Props

```ts
const props = defineProps({
  /** 全屏的元素，默认是 html */
  element: {
    type: String,
    default: "html"
  },
  /** 打开全屏提示语 */
  openTips: {
    type: String,
    default: "全屏"
  },
  /** 关闭全屏提示语 */
  exitTips: {
    type: String,
    default: "退出全屏"
  }
})
```

在 `@/layout/components/NavigationBar/index.ts` 模板中使用该组件并向 Props 传递数据

```html
<Screenfull v-if="showScreenfull" element=".app-main" open-tips="内容区全屏" class="screenfull" />
```

### 代码规范

基本上学习一下 [Vue 风格指南](https://link.juejin.cn?target=https%3A%2F%2Fv2.cn.vuejs.org%2Fv2%2Fstyle-guide) 就足够了，其中我认为比较重要的几点：

- prop 的定义应该尽量详细，至少需要指定其类型
- 在组件上总是必须用 key 配合 v-for，以便维护内部组件及其子树的状态
- 避免 v-if 和 v-for 用在一起
- 命名规范相关的所有内容

### Git 提交规范

现在社区一般都大致遵循这个规范：

- **feat:** 增加新的业务功能
- **fix:** 修复业务问题/BUG
- **perf:** 优化性能
- **style:** 更改代码风格, 不影响运行结果
- **refactor:** 重构代码
- **revert:** 撤销更改
- **test:** 测试相关, 不涉及业务代码的更改
- **docs:** 文档和注释相关
- **chore:** 更新依赖/修改脚手架配置等琐事
- **workflow:** 工作流改进
- **ci:** 持续集成相关
- **types:** 类型定义文件更改
- **wip:** 开发中

当然，上面这里的话只是规范你提交的描述信息。至于何时提交一个 commit 的话，我们简单采用一个原则：**完成一件事情，就提交一次 commit**。而不是等到你写完一整天的代码后，才在下班前只提交一次。

### 注释规范

由于项目采用 TS 5.x 进行开发，为了获得更好的 TS 提示，项目采用了大量的 `/** xxx */` 注释，它的优点就是鼠标放上去会有注释显示出来：

![注释规范](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/036339327dd44c489f9547eb39355ddb~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1055&h=857&s=55201&e=png&b=1e1e1e)

## 后端配套接口

基于本项目**编写一个配套后端**或想**了解项目现有接口**的小伙伴。

### RESTful API

项目调用的接口是采用 [Easy Mock](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Feasy-mock%2Feasy-mock) 和 [Fast Mock](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2FMarvenGong%2Ffastmock) 来模拟的真实接口（非常感谢这两个平台），模拟接口遵循的是 RESTful 风格，如果不了解 RESTful 风格的小伙伴可以看看这篇简文：[✒️ 带你了解前后端进化史 与 RESTful API 的出现](https://juejin.cn/post/7225111825066360887)。

### 后端接口返回的数据格式

无论是 HTTP 请求还是响应都要将 `Content-Type` 要设置为 `application/json`，也就是 JSON。

具体的响应 JSON 格式如下：

```ts
{
  code: number
  data: T
  message: string
}
```

其中 code 就是业务状态码，它是团队内部自行设定的，本项目采用了 **code 为 0 时代表一切正常Token 鉴权

### Token 鉴权

本项目采用将 Token 字符串绑定到 HTTP 请求 headers 的 Authorization 属性中发送给后端进行鉴权这种方式。

```ts
// src/utils/service.ts 中对应的前端代码
Authorization: token ? `Bearer ${token}` : undefined,
```

### 401 状态码

后端检测到 Token 过期或无效时，应当直接返回 HTTP 状态码为 401 给前端，这时前端将强制返回登录页面。

现有接口

`@/src/api/...`