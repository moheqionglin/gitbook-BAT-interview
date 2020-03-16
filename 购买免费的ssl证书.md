最近需要写一个小程序。部署的时候小程序需要后端服务器必须是https的才可以。下面就介绍配置的教程，最重要的是ssl证书免费。
apache 服务器绑定ssl证书配置

# 购买免费的ssl证书

找了一圈，发现腾讯云竟然有免费的SSL证书，果断注册腾讯云账号，购买了一个免费的SSL证书。 具体购买流程这里就不说了。

# 配置过程
买好证书以后，下载下来，解压以后会有如下目录

```

 
drwxr-xr-x 2 root root  4096 1月   6 14:44 Apache
drwxr-xr-x 2 root root  4096 1月   6 14:44 IIS
drwxr-xr-x 2 root root  4096 1月   6 14:44 Nginx
drwxr-xr-x 2 root root  4096 1月   6 14:44 Tomcat
-rw-r--r-- 1 root root  1000 1月   6 14:44 xxx.com.csr

```
因为我的服务器是centos的，用的是apache作为反向代理服务器。所以只要把Apache目录里面的东西拷贝出来，完成如下的apache配置就好。


```
yum install mod_ssl openssl

```



```

 <VirtualHost *:80>
     # ServerAdmin webmaster@dummy-host2.example.com
     DocumentRoot "/var/www/html/"
     ServerName secondDomain.xxx.com
     ServerAlias secondDomain.xxx.com
     ProxyRequests Off
     ProxyMaxForwards 100
     ProxyPreserveHost On
     ErrorLog "/var/log/secondDomain.xxx.com-error.log"
     CustomLog "/var/log/secondDomain.xxx.com-access.log" common


     #反向代理设置
     ProxyPass / http://127.0.0.1:9809/
     ProxyPassReverse / http://127.0.0.1:8909/
     <Proxy *>
        Order Deny,Allow
        Allow from all
     </Proxy>
 </VirtualHost>
 
 <VirtualHost *:443>
     # ServerAdmin webmaster@dummy-host2.example.com
     DocumentRoot "/var/www/html/"
     ServerName secondDomain.xxx.com
     ServerAlias secondDomain.xxx.com
     ProxyRequests Off
     ProxyMaxForwards 100
     ProxyPreserveHost On
     ErrorLog "/var/log/secondDomain.xxx.comssl-error.log"
     CustomLog "/var/log/secondDomain.xxx.comssl-access.log" common

	 SSLEngine on
  	 SSLCertificateFile /root/ssl/server.crt
	 SSLCertificateKeyFile /root/ssl/server.key
	 SSLCACertificateFile /root/ssl/ca.crt


     #反向代理设置
     ProxyPass / http://127.0.0.1:9809/
     ProxyPassReverse / http://127.0.0.1:9809/
     <Proxy *>
        Order Deny,Allow
        Allow from all
     </Proxy>
 </VirtualHost>
```

# 重新加载配置

```
service httpd reload
```