---
title: Ant Design Pro 源码分析之 ProLayout 中的 menu request 实现
date: 2021-10-11 18:56:41
tags:
  - Ant Design Pro
  - React
  - ProLayout
thumbnail: https://cdn.axis-studio.org/bw2021/bw2021_16.jpg?imageMogr2/quality/50
---


# Ant Design Pro 源码分析之 ProLayout 中的 menu request 实现

带着问题来研究源码。


## 版本

- "@ant-design/pro-layout": "6.26.0"

## 起因

在使用 ProLayout 组件动态渲染菜单时，遇到了切换账号但没有触发 menu request 的问题。查了 [issue](https://github.com/ant-design/ant-design-pro/issues/8932)，发现需要手动传递 params 来出发更新。想着深入了解一下这块儿的工作机制，研究一下源码。

## 经过

首先需要找到 pro-layout 源码以及引入位置，方便打 log 调试。经过查找，最终定位项目中引入的 pro-layout 位于 `your_project/node_modules/@ant-design/pro-layout/es` 路径下。

需要注意，经过测试：
  - 每次修改 pro-layout 源码 ，需要重新 `run start`
  - 需要关闭 `mfsu`

### 阅读源码

下面这段代码位于：`your_project/node_modules/@ant-design/pro-layout/es/BasicLayout.js`

```javascript
  var swrKey = useMemo(function () {
    if (!(menu === null || menu === void 0 ? void 0 : menu.params)) return [defaultId];
    return [defaultId, menu === null || menu === void 0 ? void 0 : menu.params];
  }, [defaultId, stringify(menu === null || menu === void 0 ? void 0 : menu.params)]);
  var preData = useRef(undefined);

  console.log('swrKey: ', swrKey)

  var _useSWR = useSWR(swrKey, /*#__PURE__*/function () {
    console.log('in useSWR fetcher')
    var _ref4 = _asyncToGenerator( /*#__PURE__*/regeneratorRuntime.mark(function _callee(_, params) {
      var _menu$request;

      var msg;
      return regeneratorRuntime.wrap(function _callee$(_context) {
        while (1) {
          switch (_context.prev = _context.next) {
            case 0:
              setMenuLoading(true);
              _context.next = 3;
              return menu === null || menu === void 0 ? void 0 : (_menu$request = menu.request) === null || _menu$request === void 0 ? void 0 : _menu$request.call(menu, params || {}, (route === null || route === void 0 ? void 0 : route.routes) || []);

            case 3:
              msg = _context.sent;
              setMenuLoading(false);
              return _context.abrupt("return", msg);

            case 6:
            case "end":
              return _context.stop();
          }
        }
      }, _callee);
    }));

    return function (_x, _x2) {
      return _ref4.apply(this, arguments);
    };
  }(), {
    revalidateOnFocus: false,
    shouldRetryOnError: false,
    revalidateOnReconnect: false
  }),
      data = _useSWR.data;
  console.log('_useSWR.data: ', _useSWR.data)
  preData.current = data; // params 更新的时候重新请求
  console.log('preData.current: ', preData.current)
  useEffect(function () {
    if (!preData.current) {
      return;
    }
    mutate(swrKey); // eslint-disable-next-line react-hooks/exhaustive-deps
  }, [swrKey]);
```

首先当我们在 layout config 中不传入入 menu params。

```javascript
// in app.tsx
 return {
    menu: {
      // params: {
      //   currentUser: initialState?.currentUser?.result?.current_user,
      // },
      request: async () => {
        // https://pro.ant.design/zh-CN/docs/advanced-menu
        const menuRes = await queryMenusData();
        return mappingIcon(menuRes);
      },
    },
 }
```

重新 `run start`，登录账号 A，可以看到控制台打印出

```
swrKey:  ['pro-layout-1']
```

切换为账号 B，发现 swrKey 值没变。之后没有触发 useSWR，查阅 SWR [文档](https://swr.bootcss.com/docs/advanced/performance), SWR 会对相同的 swrKey 做性能优护，只会请求一次数据。所以这边应该是没有触发 fetcher。

下面来测试传入 params。将 params 注释恢复为代码，

```javascript
// in app.tsx
 return {
    menu: {
      // params: {
      //   currentUser: initialState?.currentUser?.result?.current_user,
      // },
      request: async () => {
        // https://pro.ant.design/zh-CN/docs/advanced-menu
        const menuRes = await queryMenusData();
        return mappingIcon(menuRes);
      },
    },
 }
```

重复之前操作，退出账号 B，登录账号 A，观察控制单输出

```javascript
swrKey:  
  (2) ['pro-layout-1', {…}]
  0: "pro-layout-1"
  1: {currentUser: 'myNameA'}
  lastIndex: （…）
  lastItem: （…）
  length: 2
  [[Prototype]]: Array(0)
```

发现 swrKey 多了一个对象，存放了 params 参数。此时再切换账号 B，发现菜单已经更新，观察控制台输出。

```javascript
swrKey:  
  (2) ['pro-layout-1', {…}]
  0: "pro-layout-1"
  1: {currentUser: 'myNameB'}
  lastIndex: （…）
  lastItem: （…）
  length: 2
  [[Prototype]]: Array(0)
```

所以如果想要切换账号，或者说切换路由时重新执行 `menu request` 需要给 `menu params` 传入新值，触发 `useMemo`, `useSWR` 更新。

## 本文参考

- [SWR](https://swr.bootcss.com/docs/advanced/performance)