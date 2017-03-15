# WX-ShareConfig
微信web配置及分享    
微信分享api：http://mp.weixin.qq.com/wiki/4/9ac2e7b1f1d22e9e57260f6553822520.html  
第一步：用户同意授权，获取code   

在确保微信公众账号拥有授权作用域（scope参数）的权限的前提下（服务号获得高级接口后，默认拥有scope参数中的snsapi_base和snsapi_userinfo），引导关注者打开如下页面：
https://open.weixin.qq.com/connect/oauth2/authorize?appid=APPID&redirect_uri=REDIRECT_URI&response_type=code&scope=SCOPE&state=STATE#wechat_redirect
若提示“该链接无法访问”，请检查参数是否填写错误，是否拥有scope参数对应的授权作用域权限。
尤其注意：由于授权操作安全等级较高，所以在发起授权请求时，微信会对授权链接做正则强匹配校验，如果链接的参数顺序不对，授权页面将无法正常访问

参考链接(请在微信客户端中打开此链接体验)
Scope为snsapi_base
https://open.weixin.qq.com/connect/oauth2/authorize?appid=wx520c15f417810387&redirect_uri=https%3A%2F%2Fchong.qq.com%2Fphp%2Findex.php%3Fd%3D%26c%3DwxAdapter%26m%3DmobileDeal%26showwxpaytitle%3D1%26vb2ctag%3D4_2030_5_1194_60&response_type=code&scope=snsapi_base&state=123#wechat_redirect
Scope为snsapi_userinfo
https://open.weixin.qq.com/connect/oauth2/authorize?appid=wxf0e81c3bee622d60&redirect_uri=http%3A%2F%2Fnba.bluewebgame.com%2Foauth_response.php&response_type=code&scope=snsapi_userinfo&state=STATE#wechat_redirect
尤其注意：跳转回调redirect_uri，应当使用https链接来确保授权code的安全性。

参数说明


参数		是否必须		说明
appid		    是		    公众号的唯一标识
redirect_uri	    是		    授权后重定向的回调链接地址，请使用urlencode对链接进行处理
response_type	    是		    返回类型，请填写code
scope		    是		    应用授权作用域，snsapi_base （不弹出授权页面，直接跳转，只能获取用户openid），snsapi_userinfo （弹出授权页				       面，可通过openid拿到昵称、性别、所在地。并且，即使在未关注的情况下，只要用户授权，也能获取其信息）
state		    否		    重定向后会带上state参数，开发者可以填写a-zA-Z0-9的参数值，最多128字节
#wechat_redirect    是	 	    无论直接打开还是做页面302重定向时候，必须带此参数

用户同意授权后

如果用户同意授权，页面将跳转至 redirect_uri/?code=CODE&state=STATE。若用户禁止授权，则重定向后不会带上code参数，仅会带上state参数redirect_uri?state=STATE

code说明 ：
code作为换取access_token的票据，每次用户授权带上的code将不一样，code只能使用一次，5分钟未被使用自动过期。
第二步：通过code换取网页授权access_token

首先请注意，这里通过code换取的是一个特殊的网页授权access_token,与基础支持中的access_token（该access_token用于调用其他接口）不同。公众号可通过下述接口来获取网页授权access_token。如果网页授权的作用域为snsapi_base，则本步骤中获取到网页授权access_token的同时，也获取到了openid，snsapi_base式的网页授权流程即到此为止。

尤其注意：由于公众号的secret和获取到的access_token安全级别都非常高，必须只保存在服务器，不允许传给客户端。后续刷新access_token、通过access_token获取用户信息等步骤，也必须从服务器发起。

请求方法

获取code后，请求以下链接获取access_token： 
https://api.weixin.qq.com/sns/oauth2/access_token?appid=APPID&secret=SECRET&code=CODE&grant_type=authorization_code
参数说明

参数	是否必须		说明
appid	    是		    公众号的唯一标识
secret	    是		    公众号的appsecret
code	    是	  	    填写第一步获取的code参数
grant_type  是		    填写为authorization_code
返回说明

正确时返回的JSON数据包如下：

{
   "access_token":"ACCESS_TOKEN",
   "expires_in":7200,
   "refresh_token":"REFRESH_TOKEN",
   "openid":"OPENID",
   "scope":"SCOPE"
}
参数		描述
access_token	 网页授权接口调用凭证,注意：此access_token与基础支持的access_token不同
expires_in	 access_token接口调用凭证超时时间，单位（秒）
refresh_token	 用户刷新access_token
openid		 用户唯一标识，请注意，在未关注公众号时，用户访问公众号的网页，也会产生一个用户和公众号唯一的OpenID
scope		 用户授权的作用域，使用逗号（,）分隔

错误时微信会返回JSON数据包如下（示例为Code无效错误）:

{"errcode":40029,"errmsg":"invalid code"}
全局返回码说明

第三步：刷新access_token（如果需要）

由于access_token拥有较短的有效期，当access_token超时后，可以使用refresh_token进行刷新，refresh_token拥有较长的有效期（7天、30天、60天、90天），当refresh_token失效的后，需要用户重新授权。

请求方法

获取第二步的refresh_token后，请求以下链接获取access_token： 
https://api.weixin.qq.com/sns/oauth2/refresh_token?appid=APPID&grant_type=refresh_token&refresh_token=REFRESH_TOKEN
参数		是否必须		说明
appid		    是		    公众号的唯一标识
grant_type	    是		    填写为refresh_token
refresh_token	    是	    	    填写通过access_token获取到的refresh_token参数
返回说明

正确时返回的JSON数据包如下：

{
   "access_token":"ACCESS_TOKEN",
   "expires_in":7200,
   "refresh_token":"REFRESH_TOKEN",
   "openid":"OPENID",
   "scope":"SCOPE"
}
参数		描述
access_token	  网页授权接口调用凭证,注意：此access_token与基础支持的access_token不同
expires_in	  access_token接口调用凭证超时时间，单位（秒）
refresh_token	  用户刷新access_token
openid		  用户唯一标识
scope	 	  用户授权的作用域，使用逗号（,）分隔

错误时微信会返回JSON数据包如下（示例为Code无效错误）:

{"errcode":40029,"errmsg":"invalid code"}
全局返回码说明

第四步：拉取用户信息(需scope为 snsapi_userinfo)

如果网页授权作用域为snsapi_userinfo，则此时开发者可以通过access_token和openid拉取用户信息了。

请求方法

http：GET（请使用https协议）
https://api.weixin.qq.com/sns/userinfo?access_token=ACCESS_TOKEN&openid=OPENID&lang=zh_CN
参数说明

参数		描述
access_token	  网页授权接口调用凭证,注意：此access_token与基础支持的access_token不同
openid	  	  用户的唯一标识
lang		  返回国家地区语言版本，zh_CN 简体，zh_TW 繁体，en 英语
返回说明

正确时返回的JSON数据包如下：

{
   "openid":" OPENID",
   " nickname": NICKNAME,
   "sex":"1",
   "province":"PROVINCE"
   "city":"CITY",
   "country":"COUNTRY",
    "headimgurl":    "http://wx.qlogo.cn/mmopen/g3MonUZtNHkdmzicIlibx6iaFqAc56vxLSUfpb6n5WKSYVY0ChQKkiaJSgQ1dZuTOgvLLrhJbERQQ4eMsv84eavHiaiceqxibJxCfHe/46", 
	"privilege":[
	"PRIVILEGE1"
	"PRIVILEGE2"
    ],
    "unionid": "o6_bmasdasdsad6_2sgVt7hMZOPfL"
}
参数		描述
openid	 	  用户的唯一标识
nickname	  用户昵称
sex		  用户的性别，值为1时是男性，值为2时是女性，值为0时是未知
province	  用户个人资料填写的省份
city		  普通用户个人资料填写的城市
country		  国家，如中国为CN
headimgurl	  用户头像，最后一个数值代表正方形头像大小（有0、46、64、96、132数值可选，0代表640*640正方形头像），用户没有头像时该项为空。若用户		     更换头像，原有头像URL将失效。
privilege	  用户特权信息，json 数组，如微信沃卡用户为（chinaunicom）
unionid		 只有在用户将公众号绑定到微信开放平台帐号后，才会出现该字段。详见：获取用户个人信息（UnionID机制）

错误时微信会返回JSON数据包如下（示例为openid无效）:

{"errcode":40003,"errmsg":" invalid openid "}
全局返回码说明

附：检验授权凭证（access_token）是否有效

请求方法

http：GET（请使用https协议）
https://api.weixin.qq.com/sns/auth?access_token=ACCESS_TOKEN&openid=OPENID
参数说明

参数		描述
access_token	  网页授权接口调用凭证,注意：此access_token与基础支持的access_token不同
openid		  用户的唯一标识
返回说明

正确的Json返回结果：

{ "errcode":0,"errmsg":"ok"}
错误时的Json返回示例：

{ "errcode":40003,"errmsg":"invalid openid"}
