# Thymeleaf  简介

## 什么是Thymeleaf

Thymeleaf是一个基于java服务端的模板引擎，可同时用于web应用和原生应用。  

Thymeleaf的宗旨是提供一个优雅并且易于维护的方式来创建模板代码。  

Thymeleaf能够处理的模板如下：

- HTML
- XML
- TEXT
- JAVASCRIPT
- CSS
- RAW

## Thymeleaf 模式

Thymeleaf的模式有两种：

- 两个标记语言模板模式：`HTML`, `XML`
  - `HTML模板` 允许使用多种HTML格式，如 `HTML5`, `HTML4`, `XHTML` 。没有任何的代码验证或规范验证会被执行，模板将会最大可能地原样输出；
  - `XML模板`  与html模板不太相同，xml模板需要遵守严谨的格式规范（如不能有关闭的标签或没有关闭的引号等等），若是检测到格式问题，会抛出异常。注意 `DTD` 或者 `XML Schema` 这样的验证不会被执行；
- 三个文本模板模式：`TEXT`, `JAVASCRIPT`, `CSS`
  - `TEXT模板` 允许使用特殊的一些语法，比如一些文本邮箱或是一些模板文档。注意HTML和XML也能被这个模式解析为TEXT，在这种状况下，它就不是标记语言了，所有的tag标签、DOCTYPE等都会被视为普通文本；
  - `JAVASCRIPT模板` 允许在Thymeleaf应用中处理JS语言。js被视为文本来处理，因此拥有和 `TEXT模板` 相同的特定语法；
  - `CSS模板` 允许在Thymeleaf应用中处理CSS语言。它和JS一样被视为文本处理，因此拥有和 `TEXT模板` 相同的特定语法；
- 一个无操作模式：`RAW`
  - `RAW模板` 完全不会处理模板。它用来向正在处理的模板插入一些无法触及的资源（如文件、URL相应等等）；

## 标准方言

Thymeleaf核心库所提供的方言叫标准方言，它可以满足绝大多数用户的需求。  

当然用户可以创建自己的方言，也可以继承标准方言进行扩展。  

> 比如与Spring集成的 `thymeleaf-springX` jar包所带有一个名为 `SpringStandard Dialect` 的方言。它与thymeleaf标准方言几乎相同，但是提供了额外的，能更好适应spring框架的方言（比如使用Spring Expression或者SpringEL或者OGNL表达式）。

`处理器` ：一个给标记产物（如，tag标签、一些文本、一些占位符等）赋予逻辑的对象，叫做处理器。

绝大多数的处理器都是 `属性处理器` ，就算HTML模板文件没有被处理，它也允许浏览器正确地显示HTML页面，因为浏览器可以略过标签中它不认识的所有属性。  

例如：  

使用标记语言的JSP页面中，可能包含浏览器无法直接显示的代码片段：  

``` html  
<form:inputText name="userName" value="${user.name}" />
```

使用Thymeleaf我们可以这样写:  

``` html
<input type="text" name="userName" value="CloudSen" th:value="${user.name}" />
```

使用Thymeleaf的好处是：浏览器不仅正确显示了INPUT标签，并且允许我们给某个属性设置一个默认值（CloudSen），在页面静态地打开时，能够显示这个默认值；在页面被模板引擎处理时，`${user.name}` 可以被动态的替换。  

属性处理器一般使用命名空间表示法 `th:*`，为了兼容HTML5的语法检测，可以使用 `data-th-*` 表示法，但是***这种表示法仅仅能在HTML5文件中使用***。  

## 自然模板

从以上设计可以看出，Thymeleaf能让设计人员和开发人员处理几乎相同的模板文件，减少了将静态原型转换为工作模板文件所需的工作量。这样做法称为 `自然模板`。