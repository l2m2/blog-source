---
title: 基于Vue.js的后台管理系统的权限实现
toc: true
date: 2021-04-13 09:15:00
description: ...
tags:
- Vue.js
- 权限设计
---

## 权限访问控制模型

采用RBAC，即Role-based access control，基于角色的访问控制。

每个用户关联一个或多个角色，每个角色关联一个或多个权限，从而可以实现了非常灵活的权限管理。如下图。

![](/images/cabin-2.png)

更多的权限访问控制模型参考[这里](https://l2m2.top/2020/07/19/2020-07-19-casbin/)。

## 权限设计：初步方案

采用Ant Design Pro中的[默认方案](https://pro.antdv.com/docs/authority-management), 即前端固定路由表和权限配置，通过后端接口获取到用户拥有的权限代码，来识别是否拥有路由权限或操作权限。

大概的流程如下：

![](/images/vue-admin-permission-1.png)

## 实现：权限配置

需要考虑的点：

1. 权限配置需要支持国际化
2. 大多数模块的权限都有增删改查

最终实现：

1. 用文件目录结构体现权限层次

   层次固定为3级

   第一级为大分类，例如系统设置、功能配置等

   第二级为模块名称，例如用户管理、角色管理等

   第三级为具体的 Action，例如新增、修改、删除、读取等

2. 配置

   固定在permissions目录下编写

   ```
   |--permissions/
   |  |--admin/
   |  |  |--user.json
   |  |  |--role.json
   ```

   权限的第一级是permissions下的子目录(例如admin)。

   权限的第二级是配置文件的文件名。

   权限的第三级在JSON中书写，例如`["read", "create", "edit", "delete"]`,由于通常增删查改是模块的基础权限，支持`"CRUD"`表示增删查改来简化配置, 也支持用CRUD的子集来表示部分权限，比如`"R"`表示仅有读取的权限。

3. 翻译

   在`locales/permissions.i18n.js`中进行翻译配置。

   CRUD四种基础权限不用翻译。

## 权限配置上传工具

我们需要写一个权限配置上传工具来将上传到后端数据库。

步骤：

1. 获取所有权限code
2. 写入数据库

我用python写了一个，大概长这样：

```python
def _get_all_permissions_code():
  files = [y for x in os.walk(PERMISSION_CONF_DIR) for y in glob.glob(os.path.join(x[0], '*.json'))]
  total = []
  for file in files:
    match = re.search(re.compile(PERMISSION_CONF_DIR + "/(.*?).json"), file)
    if match:
      prefix = match.group(1)
      items = []
      with open(file, 'r') as f:
        for item in json.load(f):
          crud_match = re.search(r'^[CRUD]+$', item)
          if crud_match:
            for crud_item in crud_match.group(0):
              if crud_item == "C":
                items.append("create")
              elif crud_item == "R":
                items.append("read")
              elif crud_item == "U":
                items.append("update")
              elif crud_item == "D":
                items.append("delete")
      total = total + ['.'.join(f"{prefix}/{x}".split('/')) for x in items]
  return total
```

上传后我们数据库的权限表大概长这样：

![](/images/vue-admin-permission-2.png)

## 实现：路由表配置

除了动态生成的路由，我们有一些基础的不需要动态的路由，例如登录：

```js
export const basicRouters = [
  {
    path: "/user",
    component: UserLayout,
    redirect: "/user/login",
    children: [
      {
        path: "login",
        name: "login",
        component: () => import("@/views/login/Login")
      }
    ]
  },
  {
    path: "/404",
    component: () => import("@/views/exception/404")
  }
];
```

在动态路由的配置中，我们可以在meta中进行权限配置。

```js
export const dynamicRouters = [
  {
    path: "/",
    name: "index",
    component: BasicLayout,
    meta: { title: "menu.home" },
    redirect: "/dashboard",
    children: [
      {
        path: "/dashboard",
        name: "dashboard",
        component: () => import("@/views/dash/Dashboard"),
        meta: { title: "menu.dashboard", icon: "dashboard" }
      },
      {
        path: "/admin",
        component: RouteView,
        meta: { title: "menu.admin.default", icon: "setting" },
        children: [
          {
            path: "/admin/user",
            name: "user",
            component: () => import("@/views/admin/user/User"),
            meta: { title: "menu.admin.user", permission: ["admin.user.read"] }
          },
          {
            path: "/admin/role",
            name: "role",
            component: () => import("@/views/admin/role/Role"),
            meta: { title: "menu.admin.role", permission: ["admin.role.read"] }
          }
        ]
      }
    ]
  },
  {
    path: "*",
    redirect: "/404"
  }
];
```

## 实现：路由钩子

我们将动态生成路由的逻辑放在路由钩子中。

大概长这样：

`router/router-hook.js`

```js
const allowList = ["login"];
const loginRoutePath = "/user/login";
const defaultRoutePath = "/dashboard";
router.beforeEach((to, from, next) => {
  if (store.getters.token) {
    if (to.path === loginRoutePath) {
      next({ path: defaultRoutePath });
    } else {
      if (store.getters.additionalRouters.length === 0) {
        store
          .dispatch("user/getUserInfo")
          .then(res => {
            const permissions = res.permissions;
            store.dispatch("permission/generateRouters", { permissions }).then(() => {
              store.getters.additionalRouters.forEach(item => {
                router.addRoute(item);
              });
              const redirect = decodeURIComponent(from.query.redirect || to.path);
              if (to.path === redirect) {
                next({ ...to, replace: true });
              } else {
                next({ path: redirect });
              }
            });
          })
          .catch(() => {
            store.dispatch("user/logout").then(() => {
              next({ path: loginRoutePath, query: { redirect: to.fullPath } });
            });
          });
      } else {
        next();
      }
    }
  } else {
    if (allowList.includes(to.name)) {
      next();
    } else {
      next({ path: loginRoutePath, query: { redirect: to.fullPath } });
    }
  }
});
```

## 实现：Store

在Store中实现动态路由的逻辑并存储动态的路由表

`store/modules/permission.js`

```js
function hasPermission(router, permissions) {
  if (router.meta && router.meta.permission) {
    return !router.meta.permission.some(val => permissions.indexOf(val) === -1);
  }
  return true;
}

function filterRouters(routers, permissions) {
  const accessedRouters = routers.filter(router => {
    if (hasPermission(router, permissions)) {
      if (router.children && router.children.length) {
        router.children = filterRouters(router.children, permissions);
      }
      return true;
    }
    return false;
  });
  return accessedRouters;
}

const state = {
  routers: basicRouters,
  additionalRouters: []
};

const mutations = {
  SET_ROUTERS: (state, routers) => {
    state.additionalRouters = routers;
    state.routers = basicRouters.concat(routers);
  }
};

const actions = {
  generateRouters({ commit }, data) {
    return new Promise(resolve => {
      const { permissions } = data;
      const accessedRouters = store.getters.username === "admin" ? dynamicRouters : filterRouters(_cloneDeep(dynamicRouters), permissions);
      commit("SET_ROUTERS", accessedRouters);
      resolve();
    });
  }
};
```

## 实现：操作权限控制

我们可以给Vue注册一个全局方法，用来验证是否有权限。

写一个插件。

`plugin\auth.js`

```js
const plugin = {
  install: (Vue, { store }) => {
    if (!store) {
      throw new Error("Please provide vuex store.");
    }
    Vue.prototype.$auth = function(permissions) {
      if (store.getters.username === "admin") {
        return true;
      }
      return !permissions.some(val => store.getters.permissions.indexOf(val) === -1);
    };
  }
};
export default plugin;
```

这样，我们可以在模版中对组件进行权限控制。

```vue
<button v-if="$auth(['code1'])">
  Test
</button>
```

为了避免与组件本身的v-if逻辑混在一起写，我们推荐使用指令的方式来实现。

大同小异，我们定义一个auth指令。

`directives/auth.js`

```js
import Vue from "vue";
import store from "@/store";
const auth = Vue.directive("auth", {
  inserted: function(el, binding) {
    if (store.getters.username === "admin") {
      return;
    }
    let permissions = binding.value;
    if (typeof permissions === "string") {
      permissions = [permissions];
    }
    if (permissions.some(val => store.getters.permissions.indexOf(val) === -1)) {
      if (el.parentNode) {
        el.parentNode.removeChild(el);
      } else {
        el.style.display = "none";
      }
    }
  }
});
export default auth;
```

在模版中进行权限控制：

```vue
<button v-auth="'admin.user.create'">
  Test
</button>
```

## 实现：动态导航菜单

我们可以使用上面过滤后的动态路由来生成菜单：

```js
computed: {
  ...mapState({
    mainMenu: state => state.permission.additionalRouters
  })
},
created() {
    const routes = this.mainMenu.find(item => item.path === "/");
    this.menus = (routes && routes.children) || [];
}
```

## 回顾一下

回顾一下，主要包括以下几部分。

- 权限配置：更简化的配置，使用自动化工具上传权限。
- 动态路由：在路由钩子中实现。
- 操作权限控制：使用指令。
- 动态导航菜单：通过动态路由生成。

完整的实现在[这里](https://github.com/l2m2/fastapi-vue-admin)。

## Reference

- https://pro.antdv.com/docs/authority-management

