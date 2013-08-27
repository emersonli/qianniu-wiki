#JS变更升级#
##2013年08月27日##

###请大家修改JS地址为：###

http:
>http://l.tbcdn.cn/apps/top/c/sdk-mobile.js?version=你应用的版本
	
https:
>https://s.tbcdn.cn/apps/top/c/sdk-mobile.js?version=你应用的版本

请大家在引入JS的时候，`每次系统升级时如果有调用新接口都变动version的值`，通过从服务端取保证引入的是最新的JS SDK版本，避免部分用户的JS被缓存在本地，导致找不到新接口。

`做了离线缓存的同学，version变动后，记得改manifest文件里SDK地址的version哦`

###旺旺和中文站国际站打通###
现在旺旺聊天接口可以支持和中文站以及国际站的用户聊天了。

调用示例

	TOP.mobile.ww.chat({
    	chatNick:'朱棣',
      	text:'你好,欢迎',
     	domain_code:'taobao'|'alichn'|'aliint'
      	iid:'124234234234',
      	tid:'42423423423423'
    });
新增参数说明：

domain_code为被聊天用户所属站点，可选值有：taobao(淘宝)|alichn(中文站)|aliint(国际站)

默认值为taobao

`新版本1.4+,当用户点击推荐商品的时候，通过itemlist跳到ISV插件的时候，会带上用户nick和domain_code`
	
	
###新增同意退款组件接口###
1.4+版本新增退款组件功能

调用示例：
	
	var ui = TOP.mobile.widget.refund(rid);
    ui(function(){
    	//用户退款结束回调
    });
    ui对象可以重复使用

	