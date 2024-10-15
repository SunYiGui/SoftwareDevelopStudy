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