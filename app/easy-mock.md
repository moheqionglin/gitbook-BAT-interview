| Mock        | Mock 对象           | 
| ------------- |:-------------:|
| _req.url      |获得请求 url 地址 |
| _req.method      | 获取请求方法      |
| _req.params| 获取 url 参数对象     |
| _req.querystring| 获取查询参数字符串(url中?后面的部分)，不包含 ?     |
| _req.query| 将查询参数字符串进行解析并以对象的形式返回，如果没有查询参数字字符串则返回一个空对象     |
| _req.body      |当 post 请求以 x-www-form-urlencoded 方式提交时，我们可以拿到请求的参数对象 |
| _req.path	      |获取请求路径名 |
| _req.header     |获取请求头对象 |
| _req.originalUrl      |获取请求原始地址 |
| _req.search     |获取查询参数字符串，包含 ? |
| _req.host      |获取 host (hostname:port) |
| _req.hostname	      |获取 hostname |
| _req.type      |获取请求 Content-Type，不包含像 “charset” 这样的参数 |
| _req.protocol      |返回请求协议 |
| _req.ip      |请求远程地址 |
| _req.get(field)      |获取请求 header 中对应 field 的值
 |
| _req.get(field)      |_req.cookies(field)	获取请求 cookies 中对应 field 的值 |
	
	
	
	
	docker run -it -v /Users/wanli.zhou/Workspace/docker/easymock/data/db:/data/db mongo:3.4.1  /bin/bash

docker run -it -p 7300:7300  -v /Users/wanli.zhou/Workspace/docker/easymock/data/logs:/home/easy-mock/easy-mock/logs -v /Users/wanli.zhou/Workspace/docker/easymock/production.json:/home/easy-mock/easy-mock/config/production.json -v  /Users/wanli.zhou/Workspace/docker/easymock/upload:/home/easy-mock/easy-mock/public/upload  easymock/easymock:1.6.0 /bin/bash 


	运行(-d 后台运行) > docker-compose up -d
停止 > docker-compose stop
移除容器 > docker-compose rm
停止+移除容器 > docker-compose down
构建 > docker-compose build
查看错误日志 > docker-compose logs -f
