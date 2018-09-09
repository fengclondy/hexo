---
layout: post
title: hexoBlog
date: 2018-08-07 07:00:01
tags:
---

# 一级标题
## 二级标题
### 三级标题
#### 四级标题
##### 五级标题
###### 六级标题 


hexo常用命令
````
hexo server //本地启动
hexo new page "My New Post"   //创建新文章
hexo new "My New Paper"   //创建新页面
hexo generate    //生成静态文件
hexo deploy    //部署到服务器
hexo clean   //清除缓存
hexo d -g   //生成文件并部署到远程服务器
````

```
在hexo项目根目录下的source文件夹中新建名为 CNAME 没后缀名的文件，
其中内容编辑为你绑定的域名，例如 blog.exrick.cn；
或者在Github设置页面的Github Pages找到Custom domain 填入你的域名
```

<!-- more -->

- 文本1
- 文本2
- 文本3

1. 文本1
2. 文本2
3. 文本3

[简书](http://www.jianshu.com)--插入链接

下面是插入图片语法：![](图片链接地址)
![](http://upload-images.jianshu.io/upload_images/259-0ad0d0bfc1c608b6.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

下面是引用的基本格式：>
> 一盏灯， 一片昏黄； 一简书， 一杯淡茶。 守着那一份淡定， 品读属于自己的寂寞。 保持淡定， 才能欣赏到最美丽的风景！ 保持淡定， 人生从此不再寂寞。

粗体和斜体的表示：用两个 * 表粗体，一个表斜体

*一盏灯*， 一片昏黄；**一简书**， 一杯淡茶。 守着那一份淡定， 品读属于自己的寂寞。 保持淡定， 才能欣赏到最美丽的风景！ 保持淡定， 人生从此不再寂寞。

`代码单行引用` [简](http://www.jianshu.com)

```
多行引用
多行引用
多行引用
```

> 使用微信公众号编辑器有一个十分头疼的问题——粘贴出来的代码，格式错乱，而且特别丑。这块编辑器能够解决这个问题。

**在这里粘贴您的Markdown文档，点击“预览”按钮转换为HTML格式。** 


这里查看：<http://gqsu.top>
表格：

| Tables        | Are           | Cool  |
| ------------- |:-------------:| -----:|
| col 3 is      | right-aligned | $1600 |
| col 2 is      | centered      |   $12 |
| zebra stripes | are neat      |    $1 |


| 品类 | 个数 | 备注 |
|-----|-----|------|
| 苹果 | 1   | nice |
| 橘子 | 2   | job |

> 最外层引用  
      > 2嵌套一层引用  

> 213 >31 2 > 可以嵌套很多层

如果另起一行，只需在当前行结尾加 2 个空格

行内 HTML 元素
目前只支持部分段内 HTML 元素效果，包括` <kdb> <b> <i> <em> <sup> <sub> <br>`  

键位显示：使用 <kbd>Ctrl</kbd>+<kbd>Alt</kbd>+<kbd>Del</kbd> 重启电脑
代码块：使用 <pre></pre> 元素同样可以形成代码块
粗斜体：<b> Markdown 在此处同样适用，如 *加粗* </b>

符号转义：使用 \_ \# \* 进行避免。

代码块支持的所有语言：
支持的语言：
```
1c, abnf, accesslog, actionscript, ada, apache, applescript, 
arduino, armasm, asciidoc, aspectj, autohotkey, autoit, avrasm, awk, 
axapta, bash, basic, bnf, brainfuck, cal, capnproto, ceylon, clean, clojure, 
clojure-repl, cmake, coffeescript, coq, cos, cpp, crmsh, crystal, cs, csp, css, 
d, dart, delphi, diff, django, dns, dockerfile, dos, dsconfig, dts, dust, ebnf, elixir, elm, 
erb, erlang, erlang-repl, excel, fix, flix, fortran, fsharp, gams, gauss, gcode, gherkin, glsl, 
go, golo, gradle, groovy, haml, handlebars, haskell, haxe, hsp, htmlbars, http, hy, inform7, ini, 
irpf90, java, javascript, json, julia, kotlin, lasso, ldif, leaf, less, lisp, livecodeserver, livescript,
llvm, lsl, lua, makefile, markdown, mathematica, matlab, maxima, mel, mercury, mipsasm, mizar, 
mojolicious, monkey, moonscript, n1ql, nginx, nimrod, nix, nsis, objectivec, ocaml, openscad, 
oxygene, parser3, perl, pf, php, pony, powershell, processing, profile, prolog, protobuf, puppet, 
purebasic, python, q, qml, r, rib, roboconf, rsl, ruby, ruleslanguage, rust, scala, scheme, scilab, 
scss, smali, smalltalk, sml, sqf, sql, stan, stata, step21, stylus, subunit, swift, taggerscript, 
tap, tcl, tex, thrift, tp, twig, typescript, vala, vbnet, vbscript, vbscript-html, verilog,
vhdl, vim, x86asm, xl, xml, xquery, yaml, zephir\
```

```xml
<user>username</user>
```


扩展
支持 jsfiddle、gist、runjs、优酷视频，直接填写 url，在其之后会自动添加预览点击会展开相关内容。

http://{url_of_the_fiddle}/embedded/[{tabs}/[{style}]]/
https://gist.github.com/{gist_id}
http://runjs.cn/detail/{id}
http://v.youku.com/v_show/id_{video_id}.html


参考文献：https://segmentfault.com/markdown

<!-- <iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=330 height=86> -->




文字段：（一般不推荐使用）

{% blockquote [author[, source]] [link] [source_link_title] %}
content
{% endblockquote %}
---

代码段：

{% codeblock [title] [lang:language] [url] [link text] %}
code snippet
{% endcodeblock %}


#### 点击图片可放大

<p align="center">
 <img src="https://img.shields.io/circleci/project/vuejs/vue/dev.svg" alt="Build Status">
  <img src="https://img.shields.io/badge/Spring%20Cloud-EdgwareSR4-orange.svg" alt="Coverage Status">
  <img src="https://img.shields.io/badge/Spring%20Boot-1.5.13-blue.svg" alt="Downloads">
  <img src="https://img.shields.io/npm/v/npm.svg" alt="Version">
  <img src="https://img.shields.io/npm/l/vue.svg" alt="License">
</p>

<table>
    <tr>
        <td><img src="https://oss.pig4cloud.com/pic/201806/login.png"/></td>
        <td><img src="https://oss.pig4cloud.com/pic/201806/1.png"/></td>
    </tr>
    <tr>
        <td><img src="https://oss.pig4cloud.com/pic/201806/2.png"/></td>
        <td><img src="https://oss.pig4cloud.com/pic/201806/3.png"/></td>
    </tr>
    <tr>
</table>






<p align="center">
 <img src="https://img.shields.io/circleci/project/vuejs/vue/dev.svg" alt="Build Status">
  <img src="https://img.shields.io/badge/Spring%20Cloud-EdgwareSR4-orange.svg" alt="Coverage Status">
  <img src="https://img.shields.io/badge/Spring%20Boot-1.5.13-blue.svg" alt="Downloads">
  <img src="https://img.shields.io/npm/v/npm.svg" alt="Version">
  <img src="https://img.shields.io/npm/l/vue.svg" alt="License">
</p>

<table>
    <tr>
        <td><img src="https://oss.pig4cloud.com/pic/201806/login.png"/></td>
        <td><img src="https://oss.pig4cloud.com/pic/201806/1.png"/></td>
    </tr>
    <tr>
        <td><img src="https://oss.pig4cloud.com/pic/201806/2.png"/></td>
        <td><img src="https://oss.pig4cloud.com/pic/201806/3.png"/></td>
    </tr>
    <tr>
</table>