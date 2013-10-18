#推荐HTML5_offline更新策略#

>发现大家在更新离线缓存的时候经常有问题，这里公开下千牛官方的做法

#版本号#
##做法一：每个页面一个版本##
系统每个页面都维护一个版本号的概念，版本号保存在一个配置文件里，配置文件支持热加载。

示例配置文件如下

	page1=1.0.0
	page2=1.0.0
	page3=1.0.3

##做法二：全局版本##
整个应用就一个版本号

	version=1.0.0
	
	
#Manifest文件#

在渲染manifest文件时，程序代码找到本页面对应的版本号，配置成变量version（如果才用全局版本，直接把全局版本变成变量version即可）

示例manifest文件如下：
	
	CACHE MANIFEST

	#version : $version

	http://l.tbcdn.cn/apps/top/c/sdk-mobile.js?v=$version

	NETWORK:
	*

#页面文件#
同时在页面文件里同样引入和manifest文件对应的version变量

	<html>
		<head>
	        <script src="http://l.tbcdn.cn/apps/top/c/util/zepto.min.js?v=$version" type="text/javascript"></script>
        <script src="http://l.tbcdn.cn/apps/top/c/sdk-mobile.js?v=$version" type="text/javascript"></script>
        </head>
        <body>
        ............
        </body>
    </html>


#做法解析#

我们都知道，在JS和CSS文件的地址后加上不同的参数，对于浏览器而言是不同的文件，所以当version变动之后，JS,CSS会全部重新请求，这样保证CSS和JS会更新到最新的服务端版本。

当Manifest文件变动后，所有的资源都会重新请求，这样可以做到同步更新，只需要改动一个version参数，客户端就都更新了。

#需要提醒的是#
如果您的JS不能做到向下兼容，建议新建一个JS，在新页面上引入新JS，而不是在老JS上动手脚，一般千牛的做法是在JS路径上维护一个version，如：
	
	<script src="http://l.tbcdn.cn/apps/top/c/refund/1.0/refund.js?v=$version" type="text/javascript">
	<script src="http://l.tbcdn.cn/apps/top/c/refund/2.0/refund.js?v=$version" type="text/javascript">
	<script src="http://l.tbcdn.cn/apps/top/c/refund/3.0/refund.js?v=$version" type="text/javascript">
	
看出区别了么？各位。

