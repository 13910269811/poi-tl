# poi-tl(poi-template-language)

[![Build Status](https://travis-ci.org/Sayi/poi-tl.svg?branch=master)](https://travis-ci.org/Sayi/poi-tl) ![jdk1.6+](https://img.shields.io/badge/jdk-1.6%2B-orange.svg) ![jdk1.8](https://img.shields.io/badge/jdk-1.8-orange.svg) ![poi3.16%2B](https://img.shields.io/badge/apache--poi-3.16%2B-blue.svg) ![poi4.0.0](https://img.shields.io/badge/apache--poi-4.0.0-blue.svg) [![Gitter](https://badges.gitter.im/Sayi/poi-tl.svg)](https://gitter.im/Sayi/poi-tl?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge)

Word 模板引擎，基于Apache POI - the Java API for Microsoft Documents。

## Why poi-tl
常见的文本型模板引擎（如FreeMarker、Velocity）是通过**文本模板**和可变数据生成新的HTML页面，配置文件等，而poi-tl是Word模板引擎，通过**Microsoft Word模板**和可变数据生成新的Word文档。

poi-tl是一种"logic-less"模板引擎，没有复杂的控制结构和变量赋值，只有标签，一些标签可以被替换为文档元素，有些标签则会隐藏文档元素，而另一些则被替换为一系列文档元素。

与文本型模板不同的是Word模板文档拥有样式，poi-tl在生成的文档中Word模板原有的样式都会被保留，标签的样式也会被应用到替换后的文本上，因此你可以专注于模板设计。

poi-tl支持自定义渲染函数（插件），函数可以应用到Word模板的任何位置，poi-tl的设计目标是在文档的任何地方做任何事情(*Do Anything Anywhere*)。

## Maven

```xml
<dependency>
  <groupId>com.deepoove</groupId>
  <artifactId>poi-tl</artifactId>
  <version>1.7.0</version>
</dependency>
```

## 2分钟快速入门
从一个超级简单的例子开始：把{{title}}替换成"Poi-tl 模板引擎"。

1. 新建文档模板template.docx，包含标签{{title}}
2. TDO模式：Template + data-model = output

```java
//核心API采用了极简设计，只需要一行代码
XWPFTemplate.compile("template.docx").render(new HashMap<String, Object>(){{
        put("title", "Poi-tl 模板引擎");
}}).writeToFile("out_template.docx");
```

## 标签
标签由前后两个大括号环绕，`{{var}}`是标签，`{{?var}}`也是标签，`?`标识了标签类型，`var`是这个标签的键。

### 文本
Word模板中最基本的标签类型是文本，`{{var}}`会在数据中寻找key为var的值然后进行渲染，如果找不到var，默认会清空标签，可以配置是保留标签还是快速抛出异常。

模板中标签的样式会被保留到生成的文档中。

可变的数据:
```json
{
  "name": "Mama",
  "thing": "chocolates"
}
```

给定Word模板:

**{{name}}** always said life was like a box of {{thing}}.
~~{{name}}~~ always said life was like a box of {{thing}}.

将输出以下内容:

**Mama** always said life was like a box of chocolates.
~~Mama~~ always said life was like a box of chocolates.

### 图片
图片标签以`@`开始，如`{{@var}}`，它会被渲染成指定地址的图片。
可变的数据:
```json
{
  "watermelon": {
    "path": "watermelon.png"
  },
  "lemon": {
    "path": "http://xxx/lemon.png"
  },
  "banana": {
    "path": "sob.png",
    "width": 24,
    "height": 24
  }
}
```

给定Word模板:

```
Fruit Logo:
watermelon {{@watermelon}}
lemon {{@lemon}}
banana {{@banana}}
```

将输出以下内容:

```
Fruit Logo:
watermelon 🍉
lemon 🍋
banana 🍌
```

### 表格
表格标签以`#`开始，如`{{#var}}`，它会被渲染成word表格。
可变的数据:
```json
{
  "song": {
    "header": [
      {
        "cellDatas": [
          {"renderData": {"text": "Song name"}},
          {"renderData": {"text": "artist"}}
        ]
      }
    ],
    "rowDatas": [
      {
        "cellDatas": [
          {"renderData": {"text": "Memories"}},
          {"renderData": {"text": "Maroon 5"}}
        ],
        "rowStyle":{
          "backgroundColor":"f6f8fa"
        }
      }
    ]
  }
}
```

给定Word模板:

```
{{#song}}
```

将输出以下内容:

<table>
<tr><td>Song name</td><td>Artist</td></tr>
<tr><td>Memories</td><td>Maroon 5</td></tr>
</table>

### 列表
列表标签以`*`开始，如`{{*var}}`，它会被渲染成Word符号列表或者编号列表。
可变的数据:
```json
{
  "feature": {
    "numFmt": {
      "decimal": "%1)"
    },
    "numbers": [
      {
        "text": "Plug-in function, define your own function"
      },
      {
        "text": "Supports text, pictures, table, list, if, foreach..."
      },
      {
        "text": "Templates, not just templates, but also style templates"
      }
    ]
  }
}
```

给定Word模板:

```
{{*feature}}
```

将输出以下内容:

```
1) Plug-in function, define your own function
2) Supports text, pictures, table, list, if, foreach...
3) Templates, not just templates, but also style templates
```

### 区块标签对
区块标签对是由前后两个标签组成，开始标签以`?`标识，结束标签以`/`标识，如`{{?var}}`作为var区块的起始标签，`{{/var}}`为结束标签，var是这个区块标签对的键。

位于区块中的文档元素（文本、图片、表格等）可以被渲染零次，一次或N次，这取决于区块键的取值。

#### False值或空的集合
如果区块键的值是`null`、`false`或者空的集合，位于区块中的所有文档元素将不会显示。

可变的数据:
```json
{
  "announce": false
}
```

给定Word模板:

```
Made it,Ma!{{?announce}}Top of the world!{{/announce}}
Made it,Ma!
{{?announce}}
Top of the world!🎋
{{/announce}}
```

将输出以下内容:

```
Made it,Ma!
Made it,Ma!
```

#### 非False值，且不是集合
如果区块键的值不为`null`、`false`，且不是集合，位于区块中的所有文档元素只会被渲染一次。

可变的数据:
```json
{
  "person": { "name": "Sayi" }
}
```

给定Word模板:

```
{{?person}}
  Hi {{name}}!
{{/person}}
```

将输出以下内容:

```
  Hi Sayi!
```

#### 非空集合
如果区块键的值是一个集合，区块中的文档元素会被渲染一次或者N次，这取决于集合的大小，类似于foreach语法。

可变的数据:
```json
{
  "songs": [
    { "name": "Memories" },
    { "name": "Sugar" },
    { "name": "Last Dance(伍佰)" }
  ]
}
```

给定Word模板:

```
{{?songs}}
{{name}}
{{/songs}}
```

将输出以下内容:

```
Memories
Sugar
Last Dance(伍佰)
```

### 嵌套
嵌套标签是在模板中引入另一个模板，可以理解为import导入、include包含或者word文档合并，以`+`标识，如{{+var}}。

可变的数据:
```json
{
  "nested": {
    "file": "template/sub.docx",
    "dataModels": [
      {
        "addr": "Hangzhou,China"
      }
    ]
  }
}
```

给定两个Word模板:

```
main.docx:
Hello, World
{{+nested}}

sub.docx:
Address: {{addr}}
```

将输出以下内容:

```
Hello, World
Address: Hangzhou,China
```

## 详细文档与示例

[中文文档](http://deepoove.com/poi-tl) 

* [基础(图片、文本、表格、列表)示例：软件说明文档](http://deepoove.com/poi-tl/#_%E8%BD%AF%E4%BB%B6%E8%AF%B4%E6%98%8E%E6%96%87%E6%A1%A3)
* [表格示例：付款通知书](http://deepoove.com/poi-tl/#example-table)
* [循环模板示例：文章写作](http://deepoove.com/poi-tl/#example-article)
* [Example：个人简历](http://deepoove.com/poi-tl/#_%E4%B8%AA%E4%BA%BA%E7%AE%80%E5%8E%86)

更多的示例以及所有示例的源码参见JUnit单元测试。

![](http://deepoove.com/poi-tl/demo.png)
![](http://deepoove.com/poi-tl/demo_result.png)

## 建议和完善
参见[常见问题](http://deepoove.com/poi-tl/#_%E5%B8%B8%E8%A7%81%E9%97%AE%E9%A2%98)，欢迎在GitHub Issue中提问和交流。

社区交流讨论群：[Gitter频道](https://gitter.im/Sayi/poi-tl)

