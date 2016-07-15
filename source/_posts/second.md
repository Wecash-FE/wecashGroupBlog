title: 分析vps主机商的网站技术
date: 2016-07-13 15:15:26
tags:
---
from 晓兰
## 一. [bandwagon](http://bandwagonhost.com/index.php)
### 首页请求
a. 首页55个request.<br>
b. 26.5kb大小.<br>
c. dom加载完时间: 1.57s <br>
d. 完整加载完页面的时间是5s.<br>
e. 用户行为分析统计: 谷歌分析<br>
f. 是否开启普通的js/css压缩:是<br>
g. 是否开启gzip压缩: 已开启.
### web技术栈
php<br>
bootstrap<br>
jquery<br>

总结: 中规中矩的网站技术.
html方面几乎没有用html5新标签.比如首页的主导航还是用
```
<div id="menu">
<ul>
<li><a href="/index.php">Home</a></li>
 
<li><a href="/vps-hosting.php">VPS Hosting</a></li>
<li><a href="/knowledgebase.php">Knowledgebase</a></li>
<li class="active"><a href="/affiliates.php">Affiliates</a></li>
</ul>
</div>
```
其实可以换成
```
<nav>
	<ul>
		<li><a href="/index.php">Home</a></li>
		<li><a href="/vps-hosting.php">VPS Hosting</a></li>
		<li><a href="/knowledgebase.php">Knowledgebase</a></li>
		<li class="active"><a href="/affiliates.php">Affiliates</a></li>
	</ul>
</nav>
```
改进的地方有两点.
1.把id话的menu改成用标签了. 增加语义化和权重.
2. 代码有了间隔. 而不是整起的左边都没有一点空格. 可读性不好.
## 2. linond