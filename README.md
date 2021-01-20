# HTML自动分页插件。用于生成PDF,可选前端WEB打印生成PDF或后端wkhtmltopdf、chrome headless生成

- 主要解决，web打印，分页不可控的问题
- 结合wkhtmltopdf或chrome headless生成精美（渲染完毕后，在Console上会输出wkhtmlpdf的PDF配套生成命令）



web打印自动分页、预览。内容满了，自动换页， wkhtmltopdf生成PDF。bookjs-eazy

- 预览案例
	
 - <a href="https://bookjs.zhouwuxue.com/eazy-2.html" target="_blank" rel="noopener noreferrer">eazy-2.html</a>

 - <a href="https://bookjs.zhouwuxue.com/simple-4.html" target="_blank" rel="noopener noreferrer">另一个小说案例</a>


- 除了自动分页，打印页码。还可以生成目录
![](https://img-blog.csdnimg.cn/img_convert/cdd97e4bd4478de30a1680cd01c8b152.png)
![](https://img-blog.csdnimg.cn/img_convert/86c1dba85391ee791343650504491a06.png)
- 使用方式：
	1. 配置页面参数：
		在全局定义一个bookConfig

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
		pageSize : 'ISO_A0', // 定义纸张大小
		// orientation : 'portrait',// landscape
		orientation :  'landscape', // 定义是横屏还是竖屏放置
		padding : "10mm 10mm 10mm 10mm", // 边距
		pageNumberStart : 0, // 从第几页开始编号，0为第一页开始，或false值，没有页码,也可以为一个css选择器如：".first_cover"，从包含选择器接点的页面开始编号
		catalog : true, // boolean,会以h1,h2,h3,h4,h5,h6,等当成章节是否生成目录。

		// 当这个值为true时，页面才开始渲染。如果你的页面是动态的，
		// 就先将默认值设为false,当内容准备好后，在将其设为true
		start : true,
}
</script>
```
2. 定义一个id为content-box节点内放入要插入到文档里的内容；

```html
<div id="content-box" style="display: none">
	<h1>第1章</h1><!-- 块 -->
	<table class="nop-block-box simple-table" border="1"><!-- 块盒子 -->
		<thead>
			<tr><th>ID</th><th>姓名</th><th>年龄</th></tr>
		</thead>
		<tbody class="nop-fill-box"><!-- 子块列表，程序会自动差分 -->
			<tr><td>1</td><td>张三</td><td>12</td></tr>
			...
		</tbody>
	</table>
	<div class='nop-new-page'></div><!-- 新页面标记 -->
	<h1>第1章</h1><!-- 块 -->
	<p class="nop-text-box"><!-- 文本盒子 -->
		<span class="nop-fill-box">1234566....(很长的文字)</span><!-- 大文本，程序会自动差分 -->
	</p>
</div>
```
	内容分为3种插入方式：
		a. 块：如果当前页空间充足则整体插入，空间不足，则会整体插入到下一页（默认,content-box节点，下的节点如果不包含nop-new-page、nop-block-box和nop-text-box的都会默认为块）
		b. 块盒子：块盒子内部包含的多个块，盒子内的多个块被分割到多个页面时，都会复制包裹块的盒子的部分。
			以上图中的表格为例：table节点定义为块盒子使用class：nop-block-box标记。
			tbody节点定义为容纳块的节点，使用class: nop-fill-box标记
			这样在填充行tr时，当前页空间不足时，换页并复制外部table的节点。这样表头就得到复用
		c. 文本盒子：与块盒子类似，文本内容跨多个页面时，会复制外部包裹文本的盒子的部分。
			文本盒子节点 class: nop-text-box 填充节点用class : nop-fill-box标记
		d. 插入新页，class: nop-new-page 标记在此位置换页
		
3. 生成PDF,点击打印按钮，前端打印为PDF。或使用chrome headless,wkhtmltopdf后端生成PDF。



- 直接上示例代码：
```html
<!DOCTYPE html>
<html lang="zh-cmn-Hans">
<head>
    <meta charset="UTF-8">
    <title>NOP BOOK TEST</title>
    <meta http-equiv="X-UA-Compatible" content="IE=edge, chrome=1">
    <meta name="renderer" content="webkit">
    <meta name="apple-mobile-web-app-capable" content="yes">
    <meta name="format-detection" content="telephone=no">
    <meta name="author" content="nop">
    <meta name="generator" content="wkhtmltopdf">
    <link rel="shortcut icon" href="./js/nop.jpg" type="image/x-icon">
    <meta name="description" content="">
    <meta name="viewport" content="width=device-width, initial-scale=1, minimum-scale=1, maximum-scale=1, user-scalable=0">
    <script src="https://cdn.bootcdn.net/ajax/libs/js-polyfills/0.1.43/polyfill.min.js"></script>
    <script src="https://cdn.bootcdn.net/ajax/libs/jquery/2.1.4/jquery.min.js"></script>
    <script src="https://cdn.bootcdn.net/ajax/libs/lodash.js/4.17.20/lodash.min.js"></script>
    <script src="https://bookjs.zhouwuxue.com/js/bookjs/1.0.1/bookjs-eazy.min.js"></script>
</head>
<body>
<script>
    bookConfig = {
        pageSize : 'A4',
        // orientation : 'portrait',// landscape
        orientation :  'landscape',
        padding : "10mm 10mm 10mm 10mm",
        pageNumberStart : 0,
        catalog : true,
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
</style>
<div id="content-box" style="display: none">
    <h1>第1章</h1>
    <table class="nop-block-box simple-table" border="1">
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
    <div class='nop-new-page'></div>
    <h1>第2章</h1>
    <p class="nop-text-box">01234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890</p>
</div>
</body>
</html>

```
