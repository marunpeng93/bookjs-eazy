# WEB打印，HTML转PDF工具。bookjs-eazy
- 仓库地址： [GITEE](https://gitee.com/wuxue107/bookjs-eazy) | [GITHUB](https://github.com/wuxue107/bookjs-eazy)
- 主要解决，HTML生成PDF，分页可控的问题
- 优势：1. 只需专注用H5构件你的PDF内容,而无需关心分页及内容截断问题，内容满了，自动换页,支持预览、WEB打印、页码/目录、自定义页眉页脚。2.前后端皆可生成PDF,后端配套使用chrome headless和wkhtmltopdf命令行PDF生成工具。



# 预览案例
- bug: 苹果手机预览会出线分页和字体大小问题，在于插件依赖js动态计算高度，似乎是因为苹果为了优化，渲染不同步所带来的
- <a href="https://bookjs.zhouwuxue.com/eazy-2.html" target="_blank" rel="noopener noreferrer">eazy-2.html</a>

![alt ](https://bookjs.zhouwuxue.com/js/bookjs/eazy-2-qrcode.png)

- <a href="https://bookjs.zhouwuxue.com/simple-4.html" target="_blank" rel="noopener noreferrer">另一个小说案例</a>

![alt ](https://bookjs.zhouwuxue.com/js/bookjs/simple-4-qrcode.png)

- [小票打印](https://gitee.com/wuxue107/nop-printer)

# 使用方式：
    渲染机制：
    程序会检查全局变量window.bookConfig.start 的值。
    直到此值为true时，才开始渲染将 #content-box 节点的内容渲染为PDF样式。
    重要：如果你的页面是动态的，就先将默认值设为false,当内容准备好后，在将其设为true，
## 配置页面参数：

- 定义一个全局配置变量 bookConfig

```html
<script>
bookConfig = {
    
    /**  全部纸张类型，未全量测试，常用ISO_A4
    ISO_A0、ISO_A1、ISO_A2、ISO_A3、ISO_A4、ISO_A5
    ISO_B0、ISO_B1、ISO_B2、ISO_B3、ISO_B4、ISO_B5、ISO_B6、ISO_B7、ISO_B8、ISO_B9、ISO_B10
    ISO_C0、ISO_C1、ISO_C2、ISO_C3、ISO_C4、ISO_C5、ISO_C6、ISO_C7、ISO_DL、ISO_C7_6
    JIS_B0、JIS_B1、JIS_B2、JIS_B3、JIS_B4、JIS_B5、JIS_B6、JIS_B7、JIS_B8、JIS_B9
    NA_LEGAL、NA_LETTER、NA_LEDGER、NA_EXECUTIVE、NA_INVOICE、
    BIG_K32
    **/
    // 定义纸张大小,两种方式,可选，默认：ISO_A4
    pageSize : 'ISO_A4', 
    orientation :  'landscape', // portrait/landscape 定义纸张是竖屏/横屏放置
    /** pageSizeConfig 和 pageSize/orientation组合 ，只选一即可 **/
    pageSizeOption : {
        width : '15cm', // 自定义宽高
        height : '20cm',
    }

    // 可选，边距，所列选项为默认值
    padding : "31.8mm 25.4mm 31.8mm 25.4mm", 
    
    // 简易页码插件，可选（默认未开启），所列选项为开启是的默认值
    simplePageNum : {
        // 从第几页开始编号，默认0为第一页开始，，没有页码,也可以为一个css选择器如：".first_page"，从包含选择器接点的页面开始编号
        pageBegin : 0, 
        // 从第几页结束编号，默认-1为最后一页结束，，没有页码,也可以为一个css选择器如：".end_page"，到包含选择器接点的页面结束编号
        pageEnd : -1,
        // 页面部件， 可选
        pendant : '<div class="page-num-simple"><span style="">${PAGE} / ${TOTAL_PAGE}</span></div>',
    }, 

    // 目录插件，可选（默认未开启），所列选项为开启是的默认值
    simpleCatalog : {
        titlesSelector : 'h1,h2,h3,h4,h5,h6', // 作为目录标题的选择器，按目录级别依次
        positionSelector : '.nop-page-item-pagenum-1', //可选，目录位置，默认为第一个编号页前
    },

    // 工具栏插件，可选（默认开启），所列选项为开启时的默认值
    toolBar : {
        webPrint : true,  // 可选，默认true,Web打印按钮功能
        saveHtml : false, // 可选，默认false,禁用保存HTML功能
        
        // 服务端打印并下载, 可选，bool|object，默认false,true:使用官网的api接口,object:使用自定义的服务端打印
        // true等效的object的配置：serverPrint : { serverUrl : './' }, 官网可用serverUrl : '//bookjs.zhouwuxue.com/'
        // 要使用serverPrint,必须server能访问到你的网页。网页不要使用登录状态授权，建议通过URL参数传递临时授权
        // 如果使用官方的server进行打印，则需公网上可正确访问你用bookjs-eazy构造的网页
        serverPrint : true,
        
        buttons : [
            // 这里可以自定义工具栏按钮
            // {
            //    id : 'cloudPrint',
            //    icon : 'https://xxxx.../aa.png'
            //    onClick : function(){ console.log("...do some thing"); }
            // }
        ]
    }
    
    // 重要
    // 当这个值为true时，页面才开始渲染。如果你的页面是动态的，
    // 就先将默认值设为false,当下节所述中的#content-box节点内容准备好后，在将其设为true，
    // bookConfig.start = true;
    start : true,
}
</script>
```

## PDF内容设计
- 定义一个id为content-box节点内放入要插入到文档里的内容；
- content-box下的每个节点都需定义属性 data-op-type,表示其在文档中的插入方式 其值含义如下：
```
block : 块：（默认）如果当前页空间充足则整体插入，空间不足，则会整体插入到下一页

block-box : 块盒子：块盒子内部nop-fill-box标记的节点包含的多个块，盒子内的多个块被分割到多个页面时，都会复制包裹块的外部节点。
    以下一个示例中的表格为例：
    table节点定义为块盒子
    tbody节点定义为容纳块的容器节点（使用class: nop-fill-box标记）
    这样在填充行tr时，当前页空间不足时，换页并复制外部table（除去nop-fill-box标记的部分）继续填充。这样表头就得到复用
    
text-box : 文本盒子：与块盒子类似，大文本内容跨多个页面时，会复制外部包裹文本的盒子的部分。
     文本盒子节点， 大文本的容器节点需用 class : nop-fill-box标记

mix-box : 混合盒子：与块盒子类似超出页面自动分页，（容器使用class: nop-fill-box标记），并复制容器外层，盒子内部放置的所有节点必须标记data-op-type属性，属性值： text或block 
     text:允许跨页截断
     block:（默认）不可跨页截断

new-page : 标记从新页，开始插入

pendants : 页面部件列表（页眉/页脚/页标签/水印背景等，相对页面纸张固定的元素），在其定义后的每个页面都会显示，直到下一个pendants出现。
```

- 使用样例

```html
<div id="content-box" style="display: none">
    <div data-op-type="pendants"><!-- 定义页面部件（页眉/页脚/页标签/水印背景等） -->
        <div class='pendant-title'>第一章：块盒子</div>
    </div>
    <h1 data-op-type='block'>第1章 块盒子</h1><!-- 块 -->
    <table data-op-type="block-box" class="simple-table" border="1"><!-- 块盒子 -->
        <thead>
            <tr><th>ID</th><th>姓名</th><th>年龄</th></tr>
        </thead>
        <tbody class="nop-fill-box"><!-- 子块列表，程序会自动差分 -->
            <tr><td>1</td><td>张三</td><td>12</td></tr>
            ...
        </tbody>
        <tfoot>
            <tr><td colspan="3">表格尾部</td></tr>
        </tfoot>
    </table>
    <div data-op-type="new-page"></div><!-- 新页面标记，强制从新页开始 -->
    <div data-op-type="pendants"><!-- 定义页面部件（页眉/页脚/书签/水印背景等） -->
        <div class='pendant-title'>第二章：文本盒子</div>
    </div>
    <h1  data-op-type='block'>第2章 文本盒子</h1><!-- 块 -->
    <p data-op-type="text-box"><!-- 文本盒子 -->
        <span class="nop-fill-box">1234566....(很长的文字)</span><!-- 大文本，程序会自动差分 -->
    </p>
    <div data-op-type="new-page"></div><!-- 新页面标记，强制从新页开始 -->
    <div data-op-type="pendants"><!-- 定义页面部件（页眉/页脚/书签/水印背景等） -->
        <div class='pendant-title'>第三章：混合盒子</div>
    </div>
    <h1  data-op-type='block'>第3章 混合盒子</h1><!-- 块 -->
    <div data-op-type="mix-box"><!-- 混合盒子 -->
        <div class="nop-fill-box" style='font-size: 14px;line-height: 1.5;color: white'><!-- 文本或块列表，程序会自动差分 -->
            <div data-op-type='block' style='background-color: red;height: 300px;'>red</div>
            <div data-op-type='block' style='background-color: green;height: 300px;'>green</div>
            <div data-op-type='block' style='background-color: blue;height: 300px;'>blue</div>
            <span data-op-type='text' style='color: red'>ABCDEFGHIJKLMNOPQRSTUVWXYZ...</span>
        </div>
    </div>
</div>
```



## 直接上示例代码：（效果见上：预览eazy-2）
```html
<!DOCTYPE html>
<html lang="zh-cmn-Hans">
<head>
    <meta charset="UTF-8">
    <title>NOP BOOK TEST</title>
    <meta http-equiv="X-UA-Compatible" content="IE=edge, chrome=1">
    <meta name="renderer" content="webkit">
    <meta name="apple-mobile-web-app-capable" content="yes">
    <!--<meta name="screen-orientation" content="portrait">-->
    <!--<meta name="x5-orientation" content="portrait">-->
    <meta name="format-detection" content="telephone=no">
    <meta name="author" content="nop">
    <meta name="generator" content="wkhtmltopdf">
    <link rel="shortcut icon" href="https://bookjs.zhouwuxue.com/js/nop.jpg" type="image/x-icon">
    <meta name="description" content="">
    <meta name="viewport" content="width=device-width, initial-scale=1, minimum-scale=1, maximum-scale=1, user-scalable=0">
    <script src="https://cdn.bootcdn.net/ajax/libs/js-polyfills/0.1.43/polyfill.min.js"></script>
    <script src="https://cdn.bootcdn.net/ajax/libs/jquery/2.1.4/jquery.min.js"></script>
    <script src="https://cdn.bootcdn.net/ajax/libs/lodash.js/4.17.20/lodash.min.js"></script>
    <script src="https://bookjs.zhouwuxue.com/js/bookjs/latest/bookjs-eazy.min.js"></script>

</head>
<body>
<script>
    bookConfig = {
        pageSize : 'A4',
        // orientation : 'portrait',// landscape
        orientation :  'landscape',
        padding : "20mm 10mm 20mm 10mm",
        simpleCatalog : {},
        simplePageNum : {
            pendant : '<div class="page-num-simple"><span style="">第${PAGE}页 / 共${TOTAL_PAGE}页</span></div>',
        },
        toolBar : {
            serverPrint : true,
            webPrint : true,
            saveHtml : false,
        },
        start : true,
    }
</script>
<style>
    body {
        color: #000;
        font-family: "Microsoft YaHei UI", -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica, Arial, sans-serif, "Apple Color Emoji", "Segoe UI Emoji", "Segoe UI Symbol";
        font-weight: 400;
    }

    .simple-table {
        width: 100%;
        border-spacing: 0;
        border-collapse: unset;
        border-top: #000 1px solid;
        border-right: #000 1px solid;
    }

    .simple-table th,
    .simple-table td {
        border-left: #000 1px solid;
        border-bottom: #000 1px solid;
    }

    h1,h2,h3,h4{
        font-weight: 600;
        padding-bottom: 1.5em;
        padding-top: 2.5em;
    }
    h1 {
        font-size: 1.6rem;
        text-align: center;
    }
    h2 {
        font-size: 1.5rem;
        text-align: center;
    }
    h3 {
        font-size: 1.4rem;
    }
    h4 {
        font-size: 1.3rem;
    }

    .pendant-title{
        position: absolute;
        color: #666666;
        top: 1cm;
        left: 1cm;
    }
</style>
<div id="content-box" style="display: none">
    <div data-op-type='new-page'></div>
    <div data-op-type="pendants">
        <div class='pendant-title'>第一章：块盒子示例</div>
    </div>
    <h1 data-op-type='block'>第1章 块盒子示例</h1>
    <table data-op-type="block-box" class="simple-table" border="1">
        <thead>
        <tr><th>ID</th><th>姓名</th><th>年龄</th></tr>
        </thead>
        <tbody class="nop-fill-box">
        <tr><td>1</td><td>张三</td><td>12</td></tr>
        <tr><td>2</td><td>张三</td><td>12</td></tr>
        <tr><td>3</td><td>张三</td><td>12</td></tr>
        <tr><td>4</td><td>张三</td><td>12</td></tr>
        <tr><td>5</td><td>张三</td><td>12</td></tr>
        <tr><td>6</td><td>张三</td><td>12</td></tr>
        <tr><td>7</td><td>张三</td><td>12</td></tr>
        <tr><td>8</td><td>张三</td><td>12</td></tr>
        <tr><td>9</td><td>张三</td><td>12</td></tr>
        <tr><td>10</td><td>张三</td><td>12</td></tr>
        <tr><td>1</td><td>张三</td><td>12</td></tr>
        <tr><td>2</td><td>张三</td><td>12</td></tr>
        <tr><td>3</td><td>张三</td><td>12</td></tr>
        <tr><td>4</td><td>张三</td><td>12</td></tr>
        <tr><td>5</td><td>张三</td><td>12</td></tr>
        <tr><td>6</td><td>张三</td><td>12</td></tr>
        <tr><td>7</td><td>张三</td><td>12</td></tr>
        <tr><td>8</td><td>张三</td><td>12</td></tr>
        <tr><td>9</td><td>张三</td><td>12</td></tr>
        <tr><td>10</td><td>张三</td><td>12</td></tr>
        <tr><td>1</td><td>张三</td><td>12</td></tr>
        <tr><td>2</td><td>张三</td><td>12</td></tr>
        <tr><td>3</td><td>张三</td><td>12</td></tr>
        <tr><td>4</td><td>张三</td><td>12</td></tr>
        <tr><td>5</td><td>张三</td><td>12</td></tr>
        <tr><td>6</td><td>张三</td><td>12</td></tr>
        <tr><td>7</td><td>张三</td><td>12</td></tr>
        <tr><td>8</td><td>张三</td><td>12</td></tr>
        <tr><td>9</td><td>张三</td><td>12</td></tr>
        <tr><td>10</td><td>张三</td><td>12</td></tr>
        <tr><td>1</td><td>张三</td><td>12</td></tr>
        <tr><td>2</td><td>张三</td><td>12</td></tr>
        <tr><td>3</td><td>张三</td><td>12</td></tr>
        <tr><td>4</td><td>张三</td><td>12</td></tr>
        <tr><td>5</td><td>张三</td><td>12</td></tr>
        <tr><td>6</td><td>张三</td><td>12</td></tr>
        <tr><td>7</td><td>张三</td><td>12</td></tr>
        <tr><td>8</td><td>张三</td><td>12</td></tr>
        <tr><td>9</td><td>张三</td><td>12</td></tr>
        <tr><td>10</td><td>张三</td><td>12</td></tr>
        <tr><td>1</td><td>张三</td><td>12</td></tr>
        <tr><td>2</td><td>张三</td><td>12</td></tr>
        <tr><td>3</td><td>张三</td><td>12</td></tr>
        <tr><td>4</td><td>张三</td><td>12</td></tr>
        <tr><td>5</td><td>张三</td><td>12</td></tr>
        <tr><td>6</td><td>张三</td><td>12</td></tr>
        <tr><td>7</td><td>张三</td><td>12</td></tr>
        <tr><td>8</td><td>张三</td><td>12</td></tr>
        <tr><td>9</td><td>张三</td><td>12</td></tr>
        <tr><td>10</td><td>张三</td><td>12</td></tr>
        <tr><td>1</td><td>张三</td><td>12</td></tr>
        <tr><td>2</td><td>张三</td><td>12</td></tr>
        <tr><td>3</td><td>张三</td><td>12</td></tr>
        <tr><td>4</td><td>张三</td><td>12</td></tr>
        <tr><td>5</td><td>张三</td><td>12</td></tr>
        <tr><td>6</td><td>张三</td><td>12</td></tr>
        <tr><td>7</td><td>张三</td><td>12</td></tr>
        <tr><td>8</td><td>张三</td><td>12</td></tr>
        <tr><td>9</td><td>张三</td><td>12</td></tr>
        <tr><td>10</td><td>张三</td><td>12</td></tr>
        <tr><td>1</td><td>张三</td><td>12</td></tr>
        <tr><td>2</td><td>张三</td><td>12</td></tr>
        <tr><td>3</td><td>张三</td><td>12</td></tr>
        <tr><td>4</td><td>张三</td><td>12</td></tr>
        <tr><td>5</td><td>张三</td><td>12</td></tr>
        <tr><td>6</td><td>张三</td><td>12</td></tr>
        <tr><td>7</td><td>张三</td><td>12</td></tr>
        <tr><td>8</td><td>张三</td><td>12</td></tr>
        <tr><td>9</td><td>张三</td><td>12</td></tr>
        <tr><td>10</td><td>张三</td><td>12</td></tr>
        <tr><td>1</td><td>张三</td><td>12</td></tr>
        <tr><td>2</td><td>张三</td><td>12</td></tr>
        <tr><td>3</td><td>张三</td><td>12</td></tr>
        <tr><td>4</td><td>张三</td><td>12</td></tr>
        <tr><td>5</td><td>张三</td><td>12</td></tr>
        <tr><td>6</td><td>张三</td><td>12</td></tr>
        <tr><td>7</td><td>张三</td><td>12</td></tr>
        <tr><td>8</td><td>张三</td><td>12</td></tr>
        <tr><td>9</td><td>张三</td><td>12</td></tr>
        <tr><td>10</td><td>张三</td><td>12</td></tr>
        <tr><td>1</td><td>张三</td><td>12</td></tr>
        <tr><td>2</td><td>张三</td><td>12</td></tr>
        <tr><td>3</td><td>张三</td><td>12</td></tr>
        <tr><td>4</td><td>张三</td><td>12</td></tr>
        <tr><td>5</td><td>张三</td><td>12</td></tr>
        <tr><td>6</td><td>张三</td><td>12</td></tr>
        <tr><td>7</td><td>张三</td><td>12</td></tr>
        <tr><td>8</td><td>张三</td><td>12</td></tr>
        <tr><td>9</td><td>张三</td><td>12</td></tr>
        <tr><td>10</td><td>张三</td><td>12</td></tr>
        <tr><td>1</td><td>张三</td><td>12</td></tr>
        <tr><td>2</td><td>张三</td><td>12</td></tr>
        <tr><td>3</td><td>张三</td><td>12</td></tr>
        <tr><td>4</td><td>张三</td><td>12</td></tr>
        <tr><td>5</td><td>张三</td><td>12</td></tr>
        <tr><td>6</td><td>张三</td><td>12</td></tr>
        <tr><td>7</td><td>张三</td><td>12</td></tr>
        <tr><td>8</td><td>张三</td><td>12</td></tr>
        <tr><td>9</td><td>张三</td><td>12</td></tr>
        <tr><td>10</td><td>张三</td><td>12</td></tr>
        <tr><td>1</td><td>张三</td><td>12</td></tr>
        <tr><td>2</td><td>张三</td><td>12</td></tr>
        <tr><td>3</td><td>张三</td><td>12</td></tr>
        <tr><td>4</td><td>张三</td><td>12</td></tr>
        <tr><td>5</td><td>张三</td><td>12</td></tr>
        <tr><td>6</td><td>张三</td><td>12</td></tr>
        <tr><td>7</td><td>张三</td><td>12</td></tr>
        <tr><td>8</td><td>张三</td><td>12</td></tr>
        <tr><td>9</td><td>张三</td><td>12</td></tr>
        <tr><td>10</td><td>张三</td><td>12</td></tr>
        <tr><td>1</td><td>张三</td><td>12</td></tr>
        <tr><td>2</td><td>张三</td><td>12</td></tr>
        <tr><td>3</td><td>张三</td><td>12</td></tr>
        <tr><td>4</td><td>张三</td><td>12</td></tr>
        <tr><td>5</td><td>张三</td><td>12</td></tr>
        <tr><td>6</td><td>张三</td><td>12</td></tr>
        <tr><td>7</td><td>张三</td><td>12</td></tr>
        <tr><td>8</td><td>张三</td><td>12</td></tr>
        <tr><td>9</td><td>张三</td><td>12</td></tr>
        <tr><td>10</td><td>张三</td><td>12</td></tr>
        <tr><td>1</td><td>张三</td><td>12</td></tr>
        <tr><td>2</td><td>张三</td><td>12</td></tr>
        <tr><td>3</td><td>张三</td><td>12</td></tr>
        <tr><td>4</td><td>张三</td><td>12</td></tr>
        <tr><td>5</td><td>张三</td><td>12</td></tr>
        <tr><td>6</td><td>张三</td><td>12</td></tr>
        <tr><td>7</td><td>张三</td><td>12</td></tr>
        <tr><td>8</td><td>张三</td><td>12</td></tr>
        <tr><td>9</td><td>张三</td><td>12</td></tr>
        <tr><td>10</td><td>张三</td><td>12</td></tr>
        <tr><td>1</td><td>张三</td><td>12</td></tr>
        <tr><td>2</td><td>张三</td><td>12</td></tr>
        <tr><td>3</td><td>张三</td><td>12</td></tr>
        <tr><td>4</td><td>张三</td><td>12</td></tr>
        <tr><td>5</td><td>张三</td><td>12</td></tr>
        <tr><td>6</td><td>张三</td><td>12</td></tr>
        <tr><td>7</td><td>张三</td><td>12</td></tr>
        <tr><td>8</td><td>张三</td><td>12</td></tr>
        <tr><td>9</td><td>张三</td><td>12</td></tr>
        <tr><td>10</td><td>张三</td><td>12</td></tr>
        <tr><td>1</td><td>张三</td><td>12</td></tr>
        <tr><td>2</td><td>张三</td><td>12</td></tr>
        <tr><td>3</td><td>张三</td><td>12</td></tr>
        <tr><td>4</td><td>张三</td><td>12</td></tr>
        <tr><td>5</td><td>张三</td><td>12</td></tr>
        <tr><td>6</td><td>张三</td><td>12</td></tr>
        <tr><td>7</td><td>张三</td><td>12</td></tr>
        <tr><td>8</td><td>张三</td><td>12</td></tr>
        <tr><td>9</td><td>张三</td><td>12</td></tr>
        <tr><td>10</td><td>张三</td><td>12</td></tr>
        <tr><td>1</td><td>张三</td><td>12</td></tr>
        <tr><td>2</td><td>张三</td><td>12</td></tr>
        <tr><td>3</td><td>张三</td><td>12</td></tr>
        <tr><td>4</td><td>张三</td><td>12</td></tr>
        <tr><td>5</td><td>张三</td><td>12</td></tr>
        <tr><td>6</td><td>张三</td><td>12</td></tr>
        <tr><td>7</td><td>张三</td><td>12</td></tr>
        <tr><td>8</td><td>张三</td><td>12</td></tr>
        <tr><td>9</td><td>张三</td><td>12</td></tr>
        <tr><td>10</td><td>张三</td><td>12</td></tr>
        </tbody>
        <tfoot>
        <tr><td colspan="3">表格尾部</td></tr>
        </tfoot>
    </table>
    <div data-op-type="new-page"></div>
    <div data-op-type="pendants">
        <div class='pendant-title' style='color: #666666'>第二章：文本盒子示例</div>
    </div>
    <h1  data-op-type='block'>第2章 文本盒子示例</h1>
    <p data-op-type="text-box">012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789</p>
    <div data-op-type="new-page"></div><!-- 新页面标记，强制从新页开始 -->
    <div data-op-type="pendants"><!-- 定义页面部件（页眉/页脚/书签/水印背景等） -->
        <div class='pendant-title'>第三章：混合盒子</div>
    </div>
    <h1  data-op-type='block'>第3章 混合盒子</h1><!-- 块 -->
    <div data-op-type="mix-box"><!-- 混合盒子 -->
        <div class="nop-fill-box" style='font-size: 14px;line-height: 1.5;color: white'><!-- 文本或块列表，程序会自动差分 -->
            <div data-op-type='block' style='background-color: red;height: 300px;'>red</div>
            <div data-op-type='block' style='background-color: green;height: 300px;'>green</div>
            <div data-op-type='block' style='background-color: blue;height: 300px;'>blue</div>
            <span data-op-type='text' style='color: red'>ABCDEFGHIJKLMNOPQRSTUVWXYZABCDEFGHIJKLMNOPQRSTUVWXYZABCDEFGHIJKLMNOPQRSTUVWXYZABCDEFGHIJKLMNOPQRSTUVWXYZABCDEFGHIJKLMNOPQRSTUVWXYZABCDEFGHIJKLMNOPQRSTUVWXYZABCDEFGHIJKLMNOPQRSTUVWXYZABCDEFGHIJKLMNOPQRSTUVWXYZABCDEFGHIJKLMNOPQRSTUVWXYZABCDEFGHIJKLMNOPQRSTUVWXYZABCDEFGHIJKLMNOPQRSTUVWXYZABCDEFGHIJKLMNOPQRSTUVWXYZABCDEFGHIJKLMNOPQRSTUVWXYZABCDEFGHIJKLMNOPQRSTUVWXYZABCDEFGHIJKLMNOPQRSTUVWXYZABCDEFGHIJKLMNOPQRSTUVWXYZABCDEFGHIJKLMNOPQRSTUVWXYZABCDEFGHIJKLMNOPQRSTUVWXYZABCDEFGHIJKLMNOPQRSTUVWXYZABCDEFGHIJKLMNOPQRSTUVWXYZABCDEFGHIJKLMNOPQRSTUVWXYZABCDEFGHIJKLMNOPQRSTUVWXYZABCDEFGHIJKLMNOPQRSTUVWXYZABCDEFGHIJKLMNOPQRSTUVWXYZABCDEFGHIJKLMNOPQRSTUVWXYZABCDEFGHIJKLMNOPQRSTUVWXYZABCDEFGHIJKLMNOPQRSTUVWXYZABCDEFGHIJKLMNOPQRSTUVWXYZABCDEFGHIJKLMNOPQRSTUVWXYZABCDEFGHIJKLMNOPQRSTUVWXYZABCDEFGHIJKLMNOPQRSTUVWXYZABCDEFGHIJKLMNOPQRSTUVWXYZABCDEFGHIJKLMNOPQRSTUVWXYZABCDEFGHIJKLMNOPQRSTUVWXYZABCDEFGHIJKLMNOPQRSTUVWXYZABCDEFGHIJKLMNOPQRSTUVWXYZABCDEFGHIJKLMNOPQRSTUVWXYZABCDEFGHIJKLMNOPQRSTUVWXYZABCDEFGHIJKLMNOPQRSTUVWXYZABCDEFGHIJKLMNOPQRSTUVWXYZABCDEFGHIJKLMNOPQRSTUVWXYZABCDEFGHIJKLMNOPQRSTUVWXYZABCDEFGHIJKLMNOPQRSTUVWXYZABCDEFGHIJKLMNOPQRSTUVWXYZABCDEFGHIJKLMNOPQRSTUVWXYZABCDEFGHIJKLMNOPQRSTUVWXYZABCDEFGHIJKLMNOPQRSTUVWXYZABCDEFGHIJKLMNOPQRSTUVWXYZABCDEFGHIJKLMNOPQRSTUVWXYZABCDEFGHIJKLMNOPQRSTUVWXYZABCDEFGHIJKLMNOPQRSTUVWXYZABCDEFGHIJKLMNOPQRSTUVWXYZABCDEFGHIJKLMNOPQRSTUVWXYZABCDEFGHIJKLMNOPQRSTUVWXYZABCDEFGHIJKLMNOPQRSTUVWXYZABCDEFGHIJKLMNOPQRSTUVWXYZABCDEFGHIJKLMNOPQRSTUVWXYZABCDEFGHIJKLMNOPQRSTUVWXYZABCDEFGHIJKLMNOPQRSTUVWXYZABCDEFGHIJKLMNOPQRSTUVWXYZABCDEFGHIJKLMNOPQRSTUVWXYZABCDEFGHIJKLMNOPQRSTUVWXYZABCDEFGHIJKLMNOPQRSTUVWXYZABCDEFGHIJKLMNOPQRSTUVWXYZABCDEFGHIJKLMNOPQRSTUVWXYZABCDEFGHIJKLMNOPQRSTUVWXYZABCDEFGHIJKLMNOPQRSTUVWXYZABCDEFGHIJKLMNOPQRSTUVWXYZABCDEFGHIJKLMNOPQRSTUVWXYZABCDEFGHIJKLMNOPQRSTUVWXYZABCDEFGHIJKLMNOPQRSTUVWXYZABCDEFGHIJKLMNOPQRSTUVWXYZABCDEFGHIJKLMNOPQRSTUVWXYZABCDEFGHIJKLMNOPQRSTUVWXYZABCDEFGHIJKLMNOPQRSTUVWXYZABCDEFGHIJKLMNOPQRSTUVWXYZABCDEFGHIJKLMNOPQRSTUVWXYZABCDEFGHIJKLMNOPQRSTUVWXYZABCDEFGHIJKLMNOPQRSTUVWXYZABCDEFGHIJKLMNOPQRSTUVWXYZABCDEFGHIJKLMNOPQRSTUVWXYZABCDEFGHIJKLMNOPQRSTUVWXYZABCDEFGHIJKLMNOPQRSTUVWXYZABCDEFGHIJKLMNOPQRSTUVWXYZABCDEFGHIJKLMNOPQRSTUVWXYZABCDEFGHIJKLMNOPQRSTUVWXYZABCDEFGHIJKLMNOPQRSTUVWXYZABCDEFGHIJKLMNOPQRSTUVWXYZABCDEFGHIJKLMNOPQRSTUVWXYZABCDEFGHIJKLMNOPQRSTUVWXYZABCDEFGHIJKLMNOPQRSTUVWXYZABCDEFGHIJKLMNOPQRSTUVWXYZABCDEFGHIJKLMNOPQRSTUVWXYZABCDEFGHIJKLMNOPQRSTUVWXYZABCDEFGHIJKLMNOPQRSTUVWXYZABCDEFGHIJKLMNOPQRSTUVWXYZABCDEFGHIJKLMNOPQRSTUVWXYZABCDEFGHIJKLMNOPQRSTUVWXYZABCDEFGHIJKLMNOPQRSTUVWXYZABCDEFGHIJKLMNOPQRSTUVWXYZABCDEFGHIJKLMNOPQRSTUVWXYZABCDEFGHIJKLMNOPQRSTUVWXYZABCDEFGHIJKLMNOPQRSTUVWXYZABCDEFGHIJKLMNOPQRSTUVWXYZABCDEFGHIJKLMNOPQRSTUVWXYZABCDEFGHIJKLMNOPQRSTUVWXYZABCDEFGHIJKLMNOPQRSTUVWXYZABCDEFGHIJKLMNOPQRSTUVWXYZABCDEFGHIJKLMNOPQRSTUVWXYZABCDEFGHIJKLMNOPQRSTUVWXYZABCDEFGHIJKLMNOPQRSTUVWXYZABCDEFGHIJKLMNOPQRSTUVWXYZABCDEFGHIJKLMNOPQRSTUVWXYZABCDEFGHIJKLMNOPQRSTUVWXYZABCDEFGHIJKLMNOPQRSTUVWXYZABCDEFGHIJKLMNOPQRSTUVWXYZABCDEFGHIJKLMNOPQRSTUVWXYZABCDEFGHIJKLMNOPQRSTUVWXYZABCDEFGHIJKLMNOPQRSTUVWXYZABCDEFGHIJKLMNOPQRSTUVWXYZABCDEFGHIJKLMNOPQRSTUVWXYZABCDEFGHIJKLMNOPQRSTUVWXYZABCDEFGHIJKLMNOPQRSTUVWXYZABCDEFGHIJKLMNOPQRSTUVWXYZABCDEFGHIJKLMNOPQRSTUVWXYZABCDEFGHIJKLMNOPQRSTUVWXYZABCDEFGHIJKLMNOPQRSTUVWXYZABCDEFGHIJKLMNOPQRSTUVWXYZABCDEFGHIJKLMNOPQRSTUVWXYZABCDEFGHIJKLMNOPQRSTUVWXYZABCDEFGHIJKLMNOPQRSTUVWXYZABCDEFGHIJKLMNOPQRSTUVWXYZABCDEFGHIJKLMNOPQRSTUVWXYZABCDEFGHIJKLMNOPQRSTUVWXYZABCDEFGHIJKLMNOPQRSTUVWXYZABCDEFGHIJKLMNOPQRSTUVWXYZABCDEFGHIJKLMNOPQRSTUVWXYZABCDEFGHIJKLMNOPQRSTUVWXYZABCDEFGHIJKLMNOPQRSTUVWXYZABCDEFGHIJKLMNOPQRSTUVWXYZ</span>
        </div>
    </div>
</div>
</body>
</html>

```


# 生成PDF，配套PDF生成命令行工具的使用
- 可以通过浏览器点击打印按钮，打印另存为PDF
- (推荐) 也可以直接使用打开toolBar.serverPrint = true,直接使用官网api,进行生成下载。
    前提是。你的页面外网可访问。也可以使用自定义自己的server。
    如果使用官网api，建议使用短期授权码机制，携带在url上。只有在有授权码在一定时间段内才有访问bookjs-eazy创建的页面
- 此插件适配了wkhtmltopdf和chrome headless。可使用本项目中配套封装的命令行工具，从后端生成精美PDF


## 使用chrome headless方式渲染

```bash
    # 首次使用时,安装bin/html2pdf的依赖包
    yarn install
```

```bash
    # 安装过后，执行命令
    # 示例：
    bin/html2pdf print --output eazy-2-1.pdf "https://bookjs.zhouwuxue.com/eazy-2.html"
    
    #
    # 命令行说明：
    #   Usage: html2pdf print [options] <url>
    #   
    #   Options:
    #     -o --output [outputfile]     保存PDF文件的路径 (default: "output.pdf")
    #     -t --timeout [type]          超时时间 (default: 60000)
    #     -a --agent [agent]           指定转换引擎chrome-headless|puppeteer，默认：puppeteer (default: "puppeteer")
    #     -d --printDelay [delay]      打印前等待延迟（毫秒） (default: 1000)
    #     -c --checkJs [jsExpression]  检查是否渲染完成的js表达式 (default: "window.status === 'PDFComplete'")
    #                                  "window.document.readyState === 'complete'" 这个表达式可以用作非bookjs-eazy构建的网页
    #     -h, --help                   display help for command
    #
    #
```

## 使用wkhtmltopdf渲染(会更据h1-h6生成PDF书签),需自己去下载命令行，放入PATH的环境变量所在目录下
    
```bash

    bin/pdf-a4-landscape "https://bookjs.zhouwuxue.com/eazy-2.html" eazy-2-2.pdf
    #
    # 在bin目录下，有数个同类脚本文件。
    # 
    # bin/pdf-[纸张]-[纸张方向]  [预览的链接] [输出文件]
    #
    # 注意：如果使用wkhtmltopdf方式的自定义尺寸，不用担心，浏览器渲染完毕后，在Console上会输出wkhtmltopdf的PDF配套生成命令
```
## 生成常见问题
- 生成的PDF里全是框框,原因在于。在linux服务器环境下，通常没有安装所需字体。
- 执行bin/pdf-xx-xx 相关命令，找不到wkhtmltopdf，需自己去下载wkhtmltopdf放置PATH目录下
- bin/html2pdf --agent=chrome-headless参数启动时，报错。该agent使用的是本地已存在的chrome headless remote-debug服务
```bash
:: 启动一个本地chrome headless
"chrome.exe" --headless --disable-gpu --remote-debugging-port=9222 --disable-extensions --mute-audio
:: 然后再使用 --agent=chrome-headless 则会成功。
:: 默认的 --agent=puppeteer 则不需,以上操作，会启动自带的浏览器。
```
