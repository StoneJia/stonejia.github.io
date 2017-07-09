title: "GenieACS的鉴权配置"
date: 2016-04-21 10:21:06
categories: "工作笔记"
tags:
- "TR-069"
- "GenieACS"
---

———— 原文载于 [GenieACS Auth Config](https://github.com/zaidka/genieacs/wiki/GenieACS-Auth-Config)。

GenieACS的鉴权配置文件位于 /path_to_genieacs/config/auth.js. 目录中还有一个备份 auth-sample.js.
鉴权可以从两个方向实现，从CPE到ACS，或者从ACS到CPE。*InternetGatewayDevice.ManagementServer*对象中定义了两对鉴权参数。

##CPE to ACS:

*InternetGatewayDevice.ManagementServer.Username*
*InternetGatewayDevice.ManagementServer.Password*

目前并没有实现从CPE到ACS的鉴权。GenieACS对所有的HTTP/HTTPS的请求都回应答。这个功能将会在不久后实现。
一个解决办法是使用nginx来实现CPE到ACS一侧的鉴权。
<!--more-->

##CPE to ACS with nginx and ssl:

这个解决方案只能使用用户名/密码鉴权，不会检查deviceid！GenieACS服务会绑定在本地接口"127.0.0.1".
如果使用https下载文件，"FS_SSL"必须被设为true，以用https url向CPE发送下载请求。

编辑GenieACS配置

	{
	  "MONGODB_CONNECTION_URL" : "mongodb://127.0.0.1/genieacs",
	  "REDIS_PORT" : "6379",
	  "REDIS_HOST" : "127.0.0.1",
	  "CWMP_INTERFACE" : "127.0.0.1",
	  "CWMP_PORT" : 7547,
	  "NBI_INTERFACE" : "127.0.0.1",
	  "NBI_PORT" : 7557,
	  "FS_INTERFACE" : "127.0.0.1",
	  "FS_PORT" : 7567,
	  "FS_IP" : "tr069.tdt.de",
	  "FS_SSL" : true,
	  "LOG_INFORMS" : true,
	  "DEBUG" : false
	}

绑定genieacs-gui到接口和端口
	./genieacs-gui-trunk/bin/rails s -p 8080 -b 127.0.0.1

###在同一个服务器上安装nginx (Debian)

- 安装ngnix：sudo apt-get install nginx
- 增加新的nginx配置：touch /etc/nginx/sites-available/tr069.tdt.de
- 激活配置：ln -s /etc/nginx/sites-available/tr069.tdt.de /etc/nginx/sites-enabled/tr069.tdt.de

####重定向所有的http gui请求到https gui：
	server {
	    listen         80;
	    server_name    example.de;
	    return         301 https://$server_name$request_uri;
	}

####重定向所有的gui请求到本地nbi服务：
	server {
	    listen 10.1.4.17:7557;
	    server_name example.de;
	    ssl on;
	    ssl_certificate_key /home/tr069/genieacs/genieacs-trunk/config/acs_key.pem;
	    ssl_certificate /home/tr069/genieacs/genieacs-trunk/config/acs_cert.pem;

	    access_log /var/log/nginx/example.de.nbi.log combined;
	    error_log /var/log/nginx/example.de.nbi.log;

	    location / {
		proxy_pass http://127.0.0.1:7557;
		#proxy_http_version 1.1;
		#proxy_set_header Upgrade $http_upgrade;
		#proxy_set_header Connection 'upgrade';
		#proxy_set_header Host $host;
		#proxy_cache_bypass $http_upgrade;
		proxy_set_header Authorization "";
		auth_basic "Restricted";
		auth_basic_user_file /etc/nginx/ms-htpasswd;
	    }
	}

####重定向所有的cwmp请求到本地cwmp服务：
	server {
	    listen 10.1.4.17:7547;
	    server_name example.de;
	    ssl on;
	    ssl_certificate_key /home/tr069/genieacs/genieacs-trunk/config/acs_key.pem;
	    ssl_certificate /home/tr069/genieacs/genieacs-trunk/config/acs_cert.pem;

	    access_log /var/log/nginx/example.de.cwmp.log combined;
	    error_log /var/log/nginx/example.de.cwmp.log;

	    location / {
		proxy_pass http://127.0.0.1:7547;
		#proxy_http_version 1.1;
		#proxy_set_header Upgrade $http_upgrade;
		#proxy_set_header Connection 'upgrade';
		#proxy_set_header Host $host;
		#proxy_cache_bypass $http_upgrade;
		proxy_set_header Authorization "";
		auth_basic "Restricted";
		auth_basic_user_file /etc/nginx/ms-htpasswd;
	    }
	}

####重定向所有的fs请求到本地fs服务：
	server {
	    listen 10.1.4.17:7567;
	    server_name example.de;
	    ssl on;
	    ssl_certificate_key /home/tr069/genieacs/genieacs-trunk/config/acs_key.pem;
	    ssl_certificate /home/tr069/genieacs/genieacs-trunk/config/acs_cert.pem;

	    access_log /var/log/nginx/example.de.fs.log combined;
	    error_log /var/log/nginx/example.de.fs.log;

	    location / {
		proxy_pass https://127.0.0.1:7567;
		#proxy_http_version 1.1;
		#proxy_set_header Upgrade $http_upgrade;
		#proxy_set_header Connection 'upgrade';
		#proxy_set_header Host $host;
		#proxy_cache_bypass $http_upgrade;
		proxy_set_header Authorization "";
		auth_basic "Restricted";
		auth_basic_user_file /etc/nginx/ms-htpasswd;
	    }
	}

给cert和key文件建立链接：
	cd genieacs-trunk/config/
	ln -s acs_key.pem fs.key
	ln -s acs_cert.pem fs.crt

创建 */etc/nginx/ms-htpasswd*, 遵循[格式](http://nginx.org/en/docs/http/ngx_http_auth_basic_module.html)。

##ACS to CPE:

*InternetGatewayDevice.ManagementServer.ConnectionRequestUsername*
*InternetGatewayDevice.ManagementServer.ConnectionRequestPassword*

配置文件auth.js用于ACS到CPE的，链接请求和鉴权。默认情况下没有 username/password。
	exports.connectionRequest = function(deviceId) {
	  // return username/password pair for a given device
	  return ["", ""];
	}

可以自定义
	exports.connectionRequest = function(deviceId) {
	  // return username/password pair for a given device
	  return ["Username", "Userpass"];
	}

这个函数仅使用deviceId作为参数，并返回自定义的Username/Password。
目前使用的deviceId是字符串，将来会改，也会分别提供Manufacturer，Serial number，OUI
目前只能用一对用户名密码凭证，但是这个js文件，所以你可以按需求更改逻辑。

