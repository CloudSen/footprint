# 如何搭建VUE前端开发环境

## 环境的搭建

### 安装Node.js

...  

### 安装VUE-CLI3脚手架
> [官方文档](https://cli.vuejs.org/zh/guide/#%E8%AF%A5%E7%B3%BB%E7%BB%9F%E7%9A%84%E7%BB%84%E4%BB%B6)

执行以下指令，全局安装脚手架：  

```bash
npm install -g @vue/cli
```

### 通过VUE脚手架新建项目

1. 以终端的方式：  

    ```bash
    vue create <你的项目名字>
    ```

2. 以UI界面的方式【推荐】：  

    ```bash
    vue ui
    ```

### 安装依赖

在终端中打开新建项目文件夹。

#### 运行依赖

> 运行依赖存放在 `package.json` 文件的 `dependencies` 中。

***安装Vue-Router***  

> [官方文档])(https://router.vuejs.org/zh/installation.html)

通过脚手架直接安装：  
```bash
vue add router
```

***安装Vuex***  

> [官方文档])(https://vuex.vuejs.org/zh/)

```bash
vue add vuex
```

***【可选】安装UI框架——Element UI***  

> [官方文档](https://element.eleme.cn/#/zh-CN/component/installation)

```bash
npm i element-ui -S
```

安装完毕后，在 `main.js` 中追加以下内容，全局引入UI：  

```javascript
import Element from 'element-ui'
import 'element-ui/lib/theme-chalk/index.css'

Vue.use(Element)
```

***安装UI框架——Vuetify UI***  

> [官方文档]([https://vuetifyjs.com](https://vuetifyjs.com/))

直接使用vuetify插件，帮我们自动导入框架：  

```bash
vue add vuetify
```

插件中包含了 `vuetify-loader`， 可以让webpack仅读取使用的组件，无需额外配置。

***安装markdown-it-vue***  

用于渲染MD文章。  

```bash
npm i markdown-it-vue -S
```

***安装vue-fragment***  

在vue文件中，经常有很多无意义的 `div` 标签，通过这个插件的 `<fragment></fragment>` 标签来省略很多无意义div。  

```bash
npm i vue-fragment -S
```

***安装particles.js***  

用于动态粒子效果背景：  

```bash
npm i particles.js -S
```

***安装highlight.js***  

用于代码高亮：  

```bash
npm i highlight.js -S
```

***安装vue-parallaxy***  

用于开发视差效果：  

```bash
npm i vue-parallaxy -S
```

***安装markdown-it***  

用于渲染文章：  

```bash
npm i markdown-it -S
```

#### 开发依赖

> 开发依赖存放在 `package.json` 文件的 `devDependencies` 中。

***ESLINT***  

> [Airbnb规范要求](https://github.com/airbnb/javascript)

`vue-cli3`  在图形界面配置时，已经帮我们安装了 `eslint` 相关的依赖：  

- `@vue/cli-plugin-eslint`
- `@vue/cli-plugin-babel`
- `@vue/eslint-config-airbnb`
- `eslint`
- `babel-eslint`
- `eslint-plugin-vue` 

这还不够，我们需要 `eslint-plugin-import` 来对import语法进行检测：  

```bash
npm i eslint-plugin-import -D
```

eslint配置文件如下：  

```json
module.exports = {
  root: true,
  env: {
    node: true,
  },
  extends: [
    'plugin:vue/essential',
    '@vue/airbnb',
  ],
  rules: {
    'no-console': process.env.NODE_ENV === 'production' ? 'error' : 'off',
    'no-debugger': process.env.NODE_ENV === 'production' ? 'error' : 'off',
    // 必须使用单引号
    'quotes': ['error', 'single'],
    // 仅在必要的时候使用分号，以 [, (, /, +, - 开头的行
    'semi': ['error', 'never'],
    // 不能使用alert
    'no-alert': 'error',
    // 块之前的空格
    'space-before-blocks'        : ['error', { 'functions': 'always', 'keywords': 'always', 'classes': 'always' }],
    // 方法定义相关的空格
    'space-before-function-paren': ['error', { 'anonymous': 'always', 'named': 'always', 'asyncArrow': 'always' }],
    // 解决Missing file extension "vue" for "xxxx" (import/extensions)
    'import/extensions': ['error', 'always', {
      js: 'never',
      mjs: 'never',
      jsx: 'never',
      ts: 'never',
      tsx: 'never',
      vue: 'never'
    }]
  },
  parserOptions: {
    parser: 'babel-eslint',
  },
};
```



## 配置VSCODE

### 插件安装

[见VSCODE插件推荐]()

### 插件配置

#### Vetur配置

先对vscode进行配置，在vscode配置文件中加入：  

```json
//  启用保存时自动修复,默认只支持.js文件
"eslint.autoFixOnSave": true,
"eslint.validate": [
  "javascript",
  // 检测vue文件
  {
    "language": "vue",
    "autoFix": true
  },
  {
    "language": "html",
    "autoFix": true
  },
],
// Vetur 格式化配置
"vetur.format.defaultFormatterOptions": {
    "prettyhtml": {
        // 单行超过100个长度的时候开始换行
        "printWidth": 100,
        "tabWidth": 2,
        "useTabs": false,
        "singleQuote": false,
        "wrapAttributes": true,
        "sortAttributes": true,
    },
},
// 禁用vetur的JS格式化，交给eslint处理
"vetur.format.defaultFormatter.js": "none",
// 保存和粘贴时自动格式化
"editor.defaultFormatter": "octref.vetur",
"editor.formatOnPaste": true,
"editor.formatOnSave": true,
```
然后在项目根目录下面新建 `jsconfig.json` 文件，这个文件会包含所有的vue文件：  

```json
{
    "include": [
        "./src/**/*"
    ],
    "compilerOptions": {
        "baseUrl": ".",
        "paths": {
            "@/components/*": [
                "src/components/*"
            ]
        }
    }
}
```

#### Debugger for Chrome配置

> [官方说明](https://cn.vuejs.org/v2/cookbook/debugging-in-vscode.html)

点击 `Debug` 侧栏 => 点击小齿轮 => 选择 `Chrome`。  

然后你会发现自动在根目录创建了 `.vscode` 文件夹，里面生成了 `lauch.json` 配置文件，编辑此文件，写入以下内容：  

```json
{
    // Use IntelliSense to learn about possible attributes.
    // Hover to view descriptions of existing attributes.
    // For more information, visit: https://go.microsoft.com/fwlink/?linkid=830387
    "version": "0.2.0",
    "configurations": [
        {
            "type": "chrome",
            "request": "launch",
            "name": "vuejs: chrome",
            "url": "http://localhost:8081",
            "webRoot": "${workspaceFolder}/src",
            "breakOnLoad": true,
            "sourceMapPathOverrides": {
                "webpack:///./src/*": "${webRoot}/*",
                "webpack:///src/*": "${webRoot}/*",
                "webpack:///*": "*",
                "webpack:///./~/*": "${webRoot}/node_modules/*"
              }
        }
    ]
}
```

然后在项目根目录，创建 `vue.config.js` ，写入以下内容：  

```javascript
// VUE 配置文件
module.exports = {
  configureWebpack: {
    devtool: 'source-map',
  },
}

```



