---
title: Ant Design pro 路由权限管理源码分析
date: 2020-10-31 12:06:13
tags:
  - react
  - Ant Design pro
thumbnail: https://cdn.axis-studio.org/cp/cp26day243.jpg
---
# Ant Design pro 路由权限管理源码分析

## 前言

最近在使用 Ant Design pro 搭建一套管理系统 [moe_front](https://github.com/charexcalibur/moe_front)，后端使用 [moe_back](https://github.com/charexcalibur/moe_back) 实现 Restful Api 以及 RBAC 权限管理。

由于是第一次使用 Ant Design pro，对这套解决方案的实现很感兴趣。所以写下这篇分析，记录下看源码的思路。

## 版本

关键依赖版本

- "react": "^16.14.0"
- "dva": "^2.6.0-beta.21"
- "antd": "^4.7.3"
- "@ant-design/pro-layout": "^5.0.19"
- "@ant-design/pro-table": "^2.9.12"
- "umi": "^2.13.15"

## 数据流

首先，我们以整个路由权限管理的末端（BasicLayout.tsx）组件作为起点开始分析。

```javascript
// path src/layouts/BasicLayout.tsx
import Authorized from '@/utils/Authorized';

const menuDataRender = (menuList: MenuDataItem[]): MenuDataItem[] =>
  menuList.map(item => {
    console.log('menuListItem: ', item)
    const localItem = { ...item, children: item.children ? menuDataRender(item.children) : [] };
    console.log('localItem: ', localItem)
    return Authorized.check(item.authority, localItem, null) as MenuDataItem;
  });
```

可以看到在 `BasicLayout.tsx` 组件下引入了一个 `Authorized` 模块，并调用了该模块的 `check` 方法, 对 `menuList` 做了权限校验。校验完成后返回该权限下的路由。`menuDataRender` 默认加载的路由数据为 `config.tx` 中的 `routes`。

## 获取 authority

根据 `config.tx` 中 `routes` 的数据结构，我们可以看出 Ant Design pro 是根据路由中的 `authority` 字段进行权限管理的。

```javascript
import RenderAuthorize from '@/components/Authorized';
import { getAuthority } from './authority';

let Authorized = RenderAuthorize(getAuthority());

// Reload the rights component
const reloadAuthorized = (): void => {
  Authorized = RenderAuthorize(getAuthority());
};

/**
 * hard code
 * block need it。
 */
window.reloadAuthorized = reloadAuthorized;

export { reloadAuthorized };
export default Authorized;
```

```javascript
let CURRENT: string | string[] = 'NULL';

type CurrentAuthorityType = string | string[] | (() => typeof CURRENT);
/**
 * use  authority or getAuthority
 * @param {string|()=>String} currentAuthority
 */
const renderAuthorize = <T>(Authorized: T): ((currentAuthority: CurrentAuthorityType) => T) => (
  currentAuthority: CurrentAuthorityType,
): T => {
  if (currentAuthority) {
    if (typeof currentAuthority === 'function') {
      CURRENT = currentAuthority();
    }
    if (
      Object.prototype.toString.call(currentAuthority) === '[object String]' ||
      Array.isArray(currentAuthority)
    ) {
      CURRENT = currentAuthority as string[];
    }
  } else {
    CURRENT = 'NULL';
  }
  return Authorized;
};

export { CURRENT };
export default <T>(Authorized: T) => renderAuthorize<T>(Authorized);
```

可以看出 `Authorized` 会将 `getAuthority()` 作为参数, 从 `localStorage` 中获取 `authority` 的值。然后执行 `RenderAuthorize` 对 `currentAuthority` 进行一系列类型判断。

```javascript
function check<T, K>(authority: IAuthorityType, target: T, Exception: K): T | K | React.ReactNode {
  return checkPermissions<T, K>(authority, CURRENT, target, Exception);
}
```

```javascript
import React from 'react';
import { CURRENT } from './renderAuthorize';
// eslint-disable-next-line import/no-cycle
import PromiseRender from './PromiseRender';

export type IAuthorityType =
  | undefined
  | string
  | string[]
  | Promise<boolean>
  | ((currentAuthority: string | string[]) => IAuthorityType);

/**
 * 通用权限检查方法
 * Common check permissions method
 * @param { 权限判定 | Permission judgment } authority
 * @param { 你的权限 | Your permission description } currentAuthority
 * @param { 通过的组件 | Passing components } target
 * @param { 未通过的组件 | no pass components } Exception
 */
const checkPermissions = <T, K>(
  authority: IAuthorityType,
  currentAuthority: string | string[],
  target: T,
  Exception: K,
): T | K | React.ReactNode => {
  // 没有判定权限.默认查看所有
  // Retirement authority, return target;
  if (!authority) {
    console.log('no authority')
    console.log('target: ', target)
    return target;
  }
  // 数组处理
  if (Array.isArray(authority)) {
    if (Array.isArray(currentAuthority)) {
      if (currentAuthority.some(item => authority.includes(item))) {
        return target;
      }
    } else if (authority.includes(currentAuthority)) {
      return target;
    }
    return Exception;
  }
  // string 处理
  if (typeof authority === 'string') {
    if (Array.isArray(currentAuthority)) {
      if (currentAuthority.some(item => authority === item)) {
        return target;
      }
    } else if (authority === currentAuthority) {
      return target;
    }
    return Exception;
  }
  // Promise 处理
  if (authority instanceof Promise) {
    return <PromiseRender<T, K> ok={target} error={Exception} promise={authority} />;
  }
  // Function 处理
  if (typeof authority === 'function') {
    try {
      const bool = authority(currentAuthority);
      // 函数执行后返回值是 Promise
      if (bool instanceof Promise) {
        return <PromiseRender<T, K> ok={target} error={Exception} promise={bool} />;
      }
      if (bool) {
        return target;
      }
      return Exception;
    } catch (error) {
      throw error;
    }
  }
  throw new Error('unsupported parameters');
};

export { checkPermissions };

function check<T, K>(authority: IAuthorityType, target: T, Exception: K): T | K | React.ReactNode {
  return checkPermissions<T, K>(authority, CURRENT, target, Exception);
}

export default check;

```

## 校验 authority

我们来看一下 `check` 函数的处理逻辑，接受三个参数，分别为：

- authority 路由权限字段
- target 通过路由
- Exception 未通过路由

在 `check` 函数中返回 `checkPermissions` 函数并将参数带入，同时新增一个 `CURRENT` 参数作为当前晕乎权限 `currentAuthority`。

然后进入 `checkPermissions`, 我们可以看到，其实整个函数就是在根据 `authority` `currentAuthority`, 做组件过滤，自上而下分别做了：

- 没有判定权限.默认查看所有
- 数组处理
- string 处理
- Promise 处理
- Function 处理

每个判断中都会过滤出相应条件的组件。然后把逻辑返回到开头的 `ProLayout` 组件中，就可以实现权限路由了。

```javascript
// path src/layouts/BasicLayout.tsx
import Authorized from '@/utils/Authorized';

const menuDataRender = (menuList: MenuDataItem[]): MenuDataItem[] =>
  menuList.map(item => {
    console.log('menuListItem: ', item)
    const localItem = { ...item, children: item.children ? menuDataRender(item.children) : [] };
    console.log('localItem: ', localItem)
    return Authorized.check(item.authority, localItem, null) as MenuDataItem;
  });

  // 这里将 menuDataRender 作为 props 传给 menuDataRender，就可以根据 authority 实现权限路由了
  <ProLayout
    menuDataRender={menuDataRender}
  >
  </ProLayout>
```


## 本文参考

- 封图 摄于 cp26
- [ant-design-pro
](https://github.com/ant-design/ant-design-pro/)
- [Router and Nav](https://pro.ant.design/docs/router-and-nav-cn)