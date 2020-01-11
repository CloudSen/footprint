[TOC]

# VSCODE 配置说明

## 插件安装

### 【必备】Eslint

强制统一js、html、css代码规范。  

vue-cli生成webpack项目时，强烈推荐选择eslint的Airbnb规范。  

主要的配置文件是 `.eslintignore` 和 `.eslintrc.js` 。

### 【必备】EditorConfig for VS Code

在你保存代码时，自动根据配置格式化代码。  

配合Eslint使用，使代码更加优雅。配置文件是 `.editorconfig`。  

### 【必备】Vetur

VUE开发必备插件，能配合Eslint插件对代码规范进行检查。 

- Syntax-highlighting 语法高亮
- 支持Snippet代码片段
- Emmet格式化
- Linting / Error Checking 分析代码以查找潜在错误
- Formatting 自带各种格式化
- Auto Completion 自动补全
- 支持 Debugging 调试

安装Vetur后，需要配合以下插件获得最佳体验：  

- `Sass` sass语法高亮
- `language-stylus` stylus语法高亮
- `ESLint` 用于检测vue文件，和JS文件

### 【可选】Auto Complete Tag

自动闭合HTML标签，自动重命名配对的闭合标签。

### 【可选】vscode-icons

美化左边资源管理器里项目文件的图标，每一种文件后缀都对应一个图标。  

### 【可选】vscode-background

设置编辑器背景。  

### 【可选】Path Autocomplete

路径自动提示，代码引用其他资源（比如图片）写相对路径时，会有提示。 

### 【可选】Rainbow Brackets

配对的尖括号、括号、方括号等，添加颜色。

### 【可选】GitLens

Git必备插件。  

### 【可选】Complete Statement

类似与IDEA中的 complete statement `ctrl` + `shift` + `enter`。  

安装好之后，使用快捷键 `ctrl` + `;` 即可。

### 【可选】Element UI Snippets

Element UI 的代码模板插件。  

更多见，[模板缩写](https://marketplace.visualstudio.com/items?itemName=SS.element-ui-snippets) 。

### 【可选】vuetify-vscode

vuetify ui框架插件，自带提示功能和代码模板。  





## 插件配置

### Eslint插件配置【若VUE-CLI3自动生成，可忽略初始化】

首先通过 `npm` 安装 `eslint` ，已经安装过的直接下一步：  

```bash
npm install eslint --save-dev
```

初始化 `eslint` ，生成配置文件：  

```bash
cd ./node_modules/.bin/
./eslint --init
```
> How would you like to use eslint? => To check syntax, find problems, and enforce code style  
What type of modules does your project use? => JavaScript modules (import/export)  
Which framework does your project use? => Vue.js  
Where does your code run? => (\*) Browser (\*) Node  
How would you like to define a style for your project? => Answer questions about your sytle  
What format do you want your config file to be in? => YAML  
What style of indentation do you use? => Spaces  
What quotes do you use for strings? => Single  
What line ending do you use? => Windows  
Do you require semicolons? => Y  
What format do you want your config file to be in? => YAML  

执行完毕后，会在 `./bin`目录下， 得到 `.eslintrc.yml` 配置文件，将它复制到项目根目录下。  

初始的 `.eslintrc.yml` 文件配置如下，之后可以下里面加入更多的自定义配置：  

```yaml
env:
  browser: true
  es6: true
  node: true
extends:
  - 'eslint:recommended'
  - 'plugin:vue/essential'
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
### Vetur插件配置

先对vscode进行配置，在vscode配置文件中加入：  

```json
//  启用保存时自动修复,默认只支持.js文件
"eslint.autoFixOnSave": true,
/* vetur 相关配置 */
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
// 禁用vetur的JS格式化，交给eslint处理
"vetur.format.defaultFormatter.js": "none",
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

### vuetify-vscode插件配置

vuetify-vscode默认配置无法在 `" "` 之间进行提示，需要对vscode进行额外配置。  

在vscode用户设置文件中，加入以下内容：  

```json
/* vuetify 插件配置 */
    "editor.quickSuggestions": {
      "strings": true,
    }
```

## es配置文件
```
{
    "workbench.iconTheme": "vscode-icons",
    "workbench.colorTheme": "Solarized Dark",
    "editor.fontFamily": "Hack, Consolas, 'Courier New', monospace",
    "editor.formatOnSave": true,
    "editor.formatOnPaste": true,
    "breadcrumbs.enabled": false,
    "editor.renderWhitespace": "all",
    "vetur.format.defaultFormatter.js": "none"
}
```
