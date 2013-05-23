#HTML5离线应用#
间断性网络一直是目前手机web应用的老大难问题，手机网络的低速不稳定以及低流量导致web应用和native应用在响应和消耗上都差了一个数量级。

HTML5离线应用就是为了解决在间断性网络下对服务端的强依赖问题。
#HTML5离线介绍#
一个完美的HTML5应用首先需要能够做到展示层（HTML）和数据层逻辑层（javascript）分离。一般的做法是：HTML静态，通过javascript请求服务端数据，回填页面。

一个HTML5离线应用需要由开发者书写离线声明，离线声明里包括：哪些文件每次都读取缓存，哪些文件需要每次请求网络，哪些文件可以折中（折中的意思就是：有网络的时候用网络，无网络的时候用本地缓存）。

有了这份离线声明，浏览器（当然包括手机端的webview）就会按照你的声明来请求数据，一个完美的HTML5离线应用能最大限度的节约流量。
#HTML5离线应用示例#
http://jdy.tmall.com/offline/toolbar.html

http://jindoucloud.aliapp.com/item/item.html

上面两个示例，您可以在读完本文后，再对照着看看。您可以在联网和断开网络的情况下访问看看。
#详解离线声明#
##如何书写##
离线声明一般我们称之为manifest文件，下面是一个manifest文件的示例：

	CACHE MANIFEST

	#注释用#号
	#这两个文件缓存
	http://l.tbcdn.cn/apps/top/c/util/zepto.min.js
	http://l.tbcdn.cn/apps/top/c/sdk-mobile.js
	
	NETWORK:
	#这些路径每次都网络请求
	http://img01.taobaocdn.com/top/i1/
	http://img01.daily.taobaocdn.net/sns_album/
	http://log.mmstat.com/
	
	FALLBACK:
	#这些折中
	/ajax/  ajax.html

##如何使用##
接下来我们只需要在HTML页面里加上这份离线声明。

	<html manifest='application.manifest'>
	</html>
不要以为到这里就大功告成了，有个关键的点还没有说到，当你的html标签里加入manifest标记，浏览器就会自动请求application.manifest路径获取离线声明文件，你需要把离线声明文件放入配置的路径上，同时在http响应头里声明Content-Type的类型为text/cache-manifest。

这也意味着，其实浏览器辨认离线声明文件是通过类型是cache-manifest，而不是通过后缀名。所以,你完全可以给你的离线声明文件命名为zhudi.offline。
	
	Content-Type	text/cache-manifest manifest;charset=UTF-8
	
这样就大功告成了。

别急，往下看，否则会遇到很多莫名其妙的问题

##如何升级##
需要升级离线的文件很简单，只需要修改一下application.manifest文件即可。用户下次打开HTML页面，浏览器判断application.manifest文件的MD5值不一致，浏览器会在后台默默请求服务端的最新的缓存文件。

你完全可以在文件里加一行注释，这样MD5值就不一样了，浏览器也会重新更新应用。

这也就是说：

1. 用户每次打开你的HTML页面，至少会请求application.manifest文件一次（这次请求其实还能节约，这个后面再说）
2. 当application.manifest升级，用户打开你的HTML页面时，浏览器会在后台请求最新的数据，但是本次展示的还是老的HTML资源。用户需要下次进入的时候才能看到最新的页面（这个也能通过JS避免，不过其实没这个必要）
3. 一旦application.manifest升级，浏览器会下载所有的mainfest文件里声明的**所有**离线资源，即使你只更新了一个文件。（这个也有节省的办法，后面会说到）

##需要注意##

###缓存区不支持通配符###
这是HTML5使用起来最不方便的地方，在你声明哪些文件需要缓存的时候，不能够使用通配符和上层路径，必须具体到某个资源。NETWORK区域和FALLBACK区域可以。这也是和HTML5离线应用的处理机制有关，他的缓存不是等到页面里使用的时候再去判断，而是在获取到离线声明文件后，就开始后台下载所有声明的离线文件。

###离线声明文件不容错###
需要注意manifest文件是不容错的，也就是说你的离线声明文件一旦配置错误，将导致离线声明失效，应用将退化到普通HTML应用，而不会有任何提醒。

所以你每次修改离线文件，最好校验一次，校验办法为

	访问一次你的网页（请用高级浏览器，IE慎用），看是否正常显示，然后拉掉网线，继续访问一次。
	
哪些情况下，会被认为离线文件配置错误呢？

1.	文件不符合规定，比如写错了NETWORK
2.	Content-Type 不正确
3.	**缓存声明区包含不存在的资源文件**。这个需要尤其注意

###不支持IP###
HTML地址必须位于域名下，通过IP访问不支持。

###默认处理###
如果一个请求不在缓存声明中，也不符合折中和网络规则，那么这次网络请求会被取消。所以一般的做法有两种：

1. 在NETWORK里配置一个*（上线推荐）
2. 配置一个折中方案，把*引导到一个JS上，该JS弹出alert。（开发阶段推荐，这样有利于及时发现遗漏的未配置的可缓存资源）

##高级进阶##
HTML5离线更像一个应用层面的东西，所以我们原来应用在每次http请求上的缓存策略可以继续使用。

这不得不和大家回顾下经常使用的两种http的缓存策略。

1. max-age 在请求的返回头里加上max-age头，浏览器会按照max-age里给的时间进行本地缓存。在用户不强刷等其他外力影响下，理论上，在max-age时间内，不会发起任何请求。这也就意味着缓存时间内，无法更新。
2.  last-modified 在请求的返回头里加上last-modified:Fri, 12 May 2006 18:53:33 GMT头，那么浏览器在下次请求的时候会带上If-Modified-Since:Fri, 12 May 2006 18:53:33 GMT的头，服务端判断无更新，则返回HTTP 304 （Not Changed.）状态码。

max-age节约请求。last-modified节约流量。

再回到HTML5离线应用，一次请求根据离线声明的判断需要发起网络请求，那么接下来会做HTTP级别的缓存策略的判断，如果在max-age范围内，会直接用本地版本返回，如果不在max-age范围内，接下来就走last-modified的逻辑。

那么如何节约application.manifest的请求次数或者流量，或者如何节约application.manifest升级时资源文件下载消耗的流量是不是很清楚了？

##顺便说一下##
**从千牛1.1.2开始**,支持从JS发起top的请求了。这样大家的显示层（HTML）逻辑层（JS）都被离线缓存，数据层直接通过JS发起到TOP。大家的服务端是不是变的很轻量啊。赶快TKS放翁朱棣君彦……
demo实例：商品管理 http://jindoucloud.aliapp.com/item/item.html






































	

