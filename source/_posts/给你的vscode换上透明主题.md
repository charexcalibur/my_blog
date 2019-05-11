---
title: 给你的vscode换上透明主题
date: 2019-05-11 21:52:09
tags:
  - vscode
categories:
  - 技术
thumbnail: https://cdn.axis-studio.org/D160DDCA-37D3-488B-978A-BA829DF657F2.jpg
---
## 原文 issue 及 解决方案

[issue and comment on github](https://github.com/Microsoft/vscode/issues/32257#issuecomment-489474744)

## 环境

- vscode 1.33.1
- macOS Mojave 10.14.3

## 步骤

1. 在 vscode 中 install extension `Custom CSS and JS Loader`
2. 创建 css 以及 js 文件, `vi /to/your/path/custom.css vi /to/your/path/custom.js`

<details>
<summary>css 文件</summary>

```css
html {
    background: transparent !important;
}

.scroll-decoration {
    box-shadow: none !important;
}

.minimap {
    opacity: 0.5;
}

.editor-container {
    background: rgba(37, 37, 37, 0.4) !important;
}

.search-view .search-widget .input-box, .search-view .search-widget .input-box .monaco-inputbox,
.monaco-workbench>.part.editor>.content>.one-editor-silo>.container>.title .tabs-container>.tab,
.monaco-editor-background,
.monaco-editor .margin,
.monaco-workbench>.part>.content,
.monaco-workbench>.editor>.content>.one-editor-silo.editor-one,
.monaco-workbench>.part.editor>.content>.one-editor-silo>.container>.title,
.monaco-workbench>.part>.title,
.monaco-workbench,
.monaco-workbench>.part,
body {
    background: transparent !important;
}

.editor-group-container>.tabs {
    background-color: rgba(37, 37, 37,0.5) !important;
}

.editor-group-container>.tabs .tab {
    background-color: transparent !important;
}

.editor-group-container>.tabs .tab.active {
    background-color: rgba(37, 37, 37,0.4) !important;
}

.monaco-list.settings-toc-tree .monaco-list-row.focused {
    outline-color: rgb(37, 37, 37,0.6) !important;
}

.monaco-list.settings-toc-tree .monaco-list-row.selected,
.monaco-list.settings-toc-tree .monaco-list-row.focused,
.monaco-list .monaco-list-row.selected,
.monaco-list.settings-toc-tree:not(.drop-target) .monaco-list-row:hover:not(.selected):not(.focused) {
    background-color: rgb(37, 37, 37,0.6) !important;
}

.monaco-list.settings-editor-tree .monaco-list-row {
    background-color: transparent !important;
    outline-color: transparent !important;
}

.monaco-inputbox {
    background-color: rgba(41, 41, 41,0.2) !important;
}

.monaco-editor .selected-text {
    background-color: rgba(58, 61, 65, 0.6) !important;
}

.monaco-editor .focused .selected-text {
    background-color: rgba(38, 79, 120,0.8) !important;
}

.monaco-editor .view-overlays .current-line {
    border-color: rgba(41, 41, 41,0.2) !important;
}

.extension-editor,
.monaco-inputbox>.wrapper>.input,
.monaco-workbench>.part.editor>.content>.one-editor-silo>.container>.title .tabs-container>.tab.active,
.preferences-editor>.preferences-header,
.preferences-editor>.preferences-editors-container.side-by-side-preferences-editor .preferences-header-container,
.monaco-editor, .monaco-editor .inputarea.ime-input {
    background: transparent !important;
}

.editor-group-container>.tabs .tab {
    border: none !important;
}

```
</details>

<details>
<summary>js 文件</summary>

```js
nodeRequire('electron').remote.getCurrentWindow().setVibrancy('dark');
```
setVibrancy() 有几个 type 可以选择 `appearance-based`, `light`, `dark`, `titlebar`, `selection`, `menu`, `popover`, `sidebar`, `medium-light` 或者 `ultra-dark`

更多详情信息，请查阅 [win.setVibrancy(type)](https://electronjs.org/docs/api/browser-window#winsetvibrancytype-macos)

</details>

3. 在 vscode 的 setting.json 中写入配置

```json
"vscode_custom_css.imports": [
  "file:///to/your/path/custom.css",
  "file:///to/your/path/custom.js"
],
"vscode_custom_css.policy": true
```

4. 在 vscode 命令面板中输入 `Reload Custom CSS and JS`
5. restart vscode

效果：

![](https://cdn.axis-studio.org/vscode-transparent-min.png)


## windows

windows 用户可以参考这篇[文档](https://github.com/be5invis/vscode-custom-css#getting-started)

## 本文参考

- thumbnail: my axela
- EYHN 大佬在 issue 中的 [comment](https://github.com/Microsoft/vscode/issues/32257#issuecomment-489474744)
- [NSVisualEffectView](https://developer.apple.com/documentation/appkit/nsvisualeffectview?preferredLanguage=objc)
- electronjs api, [win.setVibrancy(type)](https://electronjs.org/docs/api/browser-window#winsetvibrancytype-macos)



