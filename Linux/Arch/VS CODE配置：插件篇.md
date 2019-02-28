# 插件列表

## 代码规范Linter

- ESLint: JS代码语法检测，代码规范
- HTMLHint: HTML 代码语法检测
- Vetur: Vue 语法高亮显示, 语法错误检查, 代码自动补全，配合ESLint插件使用



## 代码美化Prettier

- Prettier - Code formatter：代码格式化插件
- Bracket Pair Colorizer: 让配对的括号显示不同的颜色



## 框架相关

- Vue 2 Snippets: VUE2语法提示



## IDE增强

- Markdown All in One: 对MD文档的支持

- Auto Rename Tag: 自动重命名配对的HTML/XML标签

- Debugger for Chrome: 配合谷歌浏览器前端调试

- vscode-icons: 资源树目录加上图标

- Path Intellisense: 自动路径补全

- GitLens: git增强

- koroFileHeader: 顶部注释模板，可定义作者、时间等信息，并会自动更新最后修改时间

    

# 插件的配置

> 以下是我自己的常用配置  
>
> 若需要额外的资料，请查看对应插件的说明文档



## 代码规范

### ESLint

>   ESLint的介绍请看我的另外一篇文章

首先通过npm全局安装eslint模块：  

```
cnpm i -g eslint

D:\cloudsen\soft\node_global\eslint -> D:\cloudsen\soft\node_global\node_modules\eslint\bin\eslint.js
+ eslint@5.14.1
added 117 packages from 70 contributors in 25.965s
```

然后使用指令 `eslint --init` 在当前路径生成eslint配置文件，选择第三项，检查语法+查找错误+强制规范：  

```
PS D:\cloudsen\code\front-end-project-template> eslint --init

? How would you like to use ESLint? To check syntax, find problems, and enforce code style
? What type of modules does your project use? JavaScript modules (import/export)
? Which framework does your project use? Vue.js
? Where does your code run? (Press <space> to select, <a> to toggle all, <i> to invert selection)Browser
? How would you like to define a style for your project? Answer questions about your style
? What format do you want your config file to be in? YAML
? What style of indentation do you use? Spaces
? What quotes do you use for strings? Single
? What line endings do you use? Windows
? Do you require semicolons? Yes
? What format do you want your config file to be in? YAML
Checking peerDependencies of eslint-config-eslint:recommended,plugin:vue/essential@latest
Local ESLint installation not found.
The config that you've selected requires the following dependencies:

eslint-plugin-vue@latest eslint-config-eslint:recommended,plugin:vue/essential@latest error@[object Object] eslint@latest
Successfully created .eslintrc.yml file in D:\cloudsen\code\front-end-project-template
ESLint was installed locally. We recommend using this local copy instead of your globally-installed copy.
```

注意由于在生成eslint配置文件时，我选择的vue项目，因此它最后有提示我还差哪些依赖，这些依赖是必装的，因为在配置文件中已经自动配置好了。  

接着在本地项目安装以下模块：  

```
cnpm i --save-dev eslint-config-recommended
```

该模块，在 `.eslintrc.*` 配置文件的 `extend` 选项中使用：

```json
extends: [
    'eslint:recommended', // <-- 追加eslint推荐的js规范
],
```

### 对于vue项目

> 详细介绍请看这里 ：[eslint-plugin-vue](https://eslint.vuejs.org/)

还需要安装以下模块：  

```
cnpm i --save-dev eslint-plugin-vue@latest
```

eslint-plugin-vue是用来检测 `.vue` 文件中的 `<template>` 和 `<script>` 标签。该模块在 `.eslintrc.*` 配置文件的 `extend` 选项中使用：  

```json
extends: [
    'plugin:vue/recommended' // <-- 追加vue规则,最强等级的校验，强一致性
],
```

```yaml
extends:
	- 'plugin:vue/recommended'
```

### 最终配置文件预览

```yaml
env:
  browser: true
  es6: true
extends:
  - "eslint:recommended"
  - "plugin:vue/essential"
globals:
  Atomics: readonly
  SharedArrayBuffer: readonly
parserOptions:
  ecmaVersion: 2018
  sourceType: module
plugins:
  - vue
rules:
  indent:
    - error
    - 4
  linebreak-style:
    - error
    - windows
  quotes:
    - error
    - single
  semi:
    - error
    - always
```



## 代码美化



## 框架相关



## IDE增强

### Debugger for Chrome

右键Chrome快捷方式，点击属性，在目标输入框中追加 `--remote-debugging-port=9222` 。  



### Markdown All in One

#### 快捷键

- Ctrl + B：加粗

- Ctrl + I：斜体

- Alt + S：删除线
- Ctrl + Shift + ]：快速增加head级数
- Ctrl + Shift + [：快速减小head级数
- Ctrl + M：进入数学编辑
- Alt + C：快速选中/取消选中todo list
- Ctrl + Shift + V：打开预览

#### 配置

```json
/*=================markdown start===============*/
"markdown.extension.toc.githubCompatibility": true,
// 打开md文件时，自动打开预览
"markdown.extension.preview.autoShowPreviewToSide": true
/*=================markdown end===============*/
```



### Path Intellisense自动路径补全

如果是windows开发环境，会触发一个[bug](<https://github.com/ChristianKohler/NpmIntellisense/issues/12>)，解决方式是在按键映射中加入以下配置：  

```json
// Place your key bindings in this file to override the defaults
[
  // for path intellisense plugin bug
  {
    "key": ".",
    "command": ""
  }
]
```

打开用户配置文件加入以下设置：

```json
/*=================fileheader start=================*/
// 显示files.exclude隐藏的文件
"path-intellisense.showHiddenFiles": false,
// 当导航到目录时，自动添加斜杠
"path-intellisense.autoSlashAfterDirectory": true,
// 解析绝对路径的方式，true是当前工作区的根目录，false是当前磁盘根目录
"path-intellisense.absolutePathToWorkspace": true
/*=================fileheader end=================*/
```



###  koroFileHeader配置

打开用户配置文件加入以下设置：

```json
/*=================fileheader start=================*/
// 文件顶部注释配置
"fileheader.customMade": {
    "Author": "CloudSen",
    "Date": "Do not edit",
    "LastEditors": "CloudSen",
    "LastEditTime": "Do not edit",
    "Description": ""
},
// 函数注释配置
"fileheader.cursorMode": {
    "description": "",
    "param": "",
    "return": ""
},
// fileheader插件配置
"fileheader.configObj": {
    // true此文件的创建时间，false注释生成时的时间
    "createFileTime": true,
    // true只显示日期，false显示时分秒
    "timeNoDetail": false,
    // 对不同的语言配置不同的注释符号, key最好用文件后缀
    "language": {
        "java": {
            "head": "/**",
            "middle": " * @",
            "end": " */"
        },
        "html": {
            "head": "<!--",
            "middle": "  -- ",
            "end": " -->"
        },
        "js": {
            "head": "/**",
            "middle": " * @",
            "end": " */"
        }
    },
    // 是否自动添加顶部注释
    "autoAdd": false,
    // 只让支持的语言，自动添加头部注释，上面设置true才有用
    "autoAlready": true,
    // 默认注释形式
    "annotationStr": {
        "head": "/*",
        "middle": " * @",
        "end": " */",
        "use": false
    },
    // 在第几行插入顶部注释, 默认第一行
    "headInsertLine": {
        "php": 2,
        "html": 2 // html注释须在DOCTYPE下放定义
    },
    // 头部注释前面插入的内容，key是文件后缀
    "beforeAnnotation": {
        "py": "#!/usr/bin/env python\n# coding=UTF-8"
    },
    "specialOptions": {}
}
/*=================fileheader end=================*/
```



