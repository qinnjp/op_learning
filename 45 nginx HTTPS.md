## nginx HTTPS



### 1. rewrite

​	***1. rewrite中的flag***

​		协议间的跳转,不做任何修改:  return
​		地址做改写  rewrite

	跳转
		redirect	302		临时跳转	 旧网站无影响，新网站无排名
		permanent	301		永久跳转     新跳转网站有排名，旧网站排名清空
		
		http  ---> https   302		浏览器不会记住新域名
		http  ---> https   301		浏览器会记录新域名
	server {
		listen 80;
		server_name url.oldxu.com;
		return ^/$ https://$http_host$request_uri;	
	}

- last		#本条规则匹配完成后，继续向下匹配新的location URI规则
  break	  	#本条规则匹配完成即终止，不再匹配后面的任何规则

> 当rewrite规则遇到break后，本location{}与其他location{}的所有rewrite/return规则都不再执行。
> 当rewrite规则遇到last后，本location{}里后续rewrite/return规则不执行，但重写后的url再次从头开始执行所有规则，哪个匹配执行哪个。

~~~nginx
server {
    listen 80;
    server_name url.oldxu.com;
    root /code;

    location / {
        rewrite /1.html /2.html 
	break;
        rewrite /2.html /3.html;
    }

    location /2.html {
        rewrite /2.html /a.html;
    }

    location /3.html {
        rewrite /3.html /b.html;
    }
} 
~~~







***1.什么是Https***

​	HTTPS全称超文本传输安全协议，是以安全为目标的http通道，简单来说就是http的安全版。

![img](https://img2018.cnblogs.com/blog/1488697/201906/1488697-20190604151034291-1134728216.png)



***2.为什么要使用Https***

​	因为HTTP不安全。当我们使用http网站时，会遭到劫持和篡改，如果采用https协议，那么数据在传输过程中是加密的，所以黑客无法窃取或者篡改数据报文信息，同时也避免网站传输时信息泄露。

​	那么我们在实现https时，需要了解ssl协议，但我们现在使用的更多的是TLS加密协议。
​	那么TLS是怎么保证明文消息被加密的呢？在OSI七层模型中，应用层是http协议，那么在应用层协议之下，我们的表示层，是ssl协议所发挥作用的一层，它通过（握手、交换秘钥、告警、加密）等方式，使应用层http协议没有感知的情况下做到了数据的安全加密

***3.模拟不使用Https的劫持和篡改?***

~~~nginx
#1.web端
server {
	listen 80;
	server_name s.oldxu.com;
	root /code;
	location / {
		index index.html;
	
	}

}
#2.代理端
server {
	listen 80;
	server_name s.oldxu.com;
	location / {
		proxy_pass http://10.0.0.7;
		sub_filter '<h1>' 'img src="**图片地址**" >';
		include proxy_params;
	}
}

~~~



***4.Https通讯是如何确定双方的身份？***

​	那么在数据进行加密与解密过程中，如何确定双方的身份，此时就需要有一个权威机构来验证双方省份。那么这个权威机构则是CA机构。那CA机构又是如何颁发证书。

- 证书申请流程

![img](https://img2018.cnblogs.com/blog/1488697/201906/1488697-20190604151048630-893395638.png)

我们首先需要申请证书，需要进行登记，登记我是谁，我是什么组织，我想做什么，到了登记机构在通过CSR发给CA，CA中心通过后，CA中心会生成一对公钥和私钥，那么公钥会在CA证书链中保存，公钥和私钥证书订阅人拿到后，会将其部署在WEB服务器上

- 1.当浏览器访问我们的https站点时，它会去请求我们的证书
- 2.Nginx这样的web服务器会将我们的公钥证书发给浏览器
- 3.浏览器会去验证我们的证书是否是合法和有效的。
- 4.CA机构会将过期的证书放置在CRL服务器，那么CRL服务的验证效率是非常差的，所以CA又推出了OCSP响应程序，OCSP响应程序可以查询指定的一个证书是否过期，所以浏览器可以直接查询OCSP响应程序，但OCSP响应程序性能还不是很高。
- 5.Nginx会有一个OCSP的开关，当我们开启后，Nginx会主动上OCSP上查询，这样大量的客户端直接从Nginx获取，证书是否有效。



***5.Https证书类型、购买指南、注意事项?***

![img](https://img2018.cnblogs.com/blog/1488697/201906/1488697-20190604151134378-1343404269.png)

- .HTTPS证书购买指南  oldxu.com
  - 保护1个域名 www								docs.oldxu.com
    
  - 保护5个域名 www images cdn test m			docs.oldxu.com  www.oldxu.com iamges.oldxu.com
    
  - 通配符域名 *.oldboy.com
    			1套证书
          			保护所有的域名

- HTTPS注意事项
    Https不支持续费,证书到期需重新申请新并进行替换。
      Https不支持三级域名解析，如 test.m.oldboy.com。
      Https显示绿色，说明整个网站的url都是https的，并且都是安全的。
      Https显示黄色，说明网站代码中有部分URL地址是http不安全协议的。
      Https显示红色，要么证书是假的，要么证书已经过期。		

***6.如何实现单台Https、又如何实现集群Https?***









***7.如何将Https集成集群架构实现全站Https?***
***8.有一个url地址不希望走https,其他正常走https?***