---
title: Ant Design Pro 防止从浏览器输入框进入未授权路由
date: 2021-01-10 13:21:56
tags:
 - React
 - Ant Design Pro
categories:
  - 技术
thumbnail: https://cdn.axis-studio.org/cp27/cp27_35.jpg?imageMogr2/quality/50
---

# Ant Design Pro 防止从浏览器输入框进入未授权路由

## 版本

 - antd ^4.8.6
 - umi ^3.2.14

## 动态渲染菜单

首先要把 `BasicLayout` 组件里的菜单渲染逻辑修改成动态渲染，同时还要把 icon 显示出来。

```js
// path src/layouts/BasicLayout.js
const iconsEnum = {
  table: <TableOutlined />
}

// in BasicLayout function
const mappingIcon = menuData => {
  const _authority = getAuthority()
  const mappingMenu = menuData.map(item => ({
    ...item,
    icon: iconsEnum[item.icon],
    authority: _authority,
    children: item.children ? mappingIcon(item.children) : [],
  }));
  return mappingMenu;
};

const _menuData = mappingIcon(menuData) // 将 icon jsx 绑定到 menuData


// in ProLayout
menuDataRender={() => _menuData
```

## 防止从浏览器进入未授权路由

比如我在地址栏输入 `/admin` 能进入 admin 路由，而该路由并不存在于 `menuData`，显然不合理。所以需要校验 `window.location.pathname` 是否存在于 `menuData`。因为 `menuData` 可能还有子路由嵌套，所以应该使用递归函数来处理这个问题。

```js
// path src/layouts/BasicLayout.js
const checkMenuAccess = (menuItem) => {
  if(menuItem.children) {
    menuItem.children.forEach((item) => {
      return checkMenuAccess(item)
    })
  } else {
    return menuItem.path === window.location.pathname
  }
}


if(!loading && loading !== undefined) {
  const pathAccess = menuData.find((item) => {
    if(window.location.pathname === '/404') {
      return true
    }
    return checkMenuAccess(item)
  })

  if(!pathAccess && pathAccess !== undefined) {
    history.push('/404')
  }
}
```

使用 dva loading 来避免重复渲染。

## 本文参考
- 封图摄于 cp27