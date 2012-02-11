---
layout: post
title: "Install Nginx PHP MySQL On Gentoo"
category: 
tags: ['Gentoo', 'Nginx', 'PHP', 'VPS']
---
{% include JB/setup %}

这篇文章是整理之前的顺便按照最新的方法写的，cgi 方式改成 fpm

`/etc/portage/package.use`
	dev-lang/php cli fpm ctype exif mysql mysqli pdo gd curl xml hash json -soap sockets \
	-snmp simplexml calendar mhash xmlreader xmlwriter
	www-servers/nginx fastcgi ssl status ipv6
	
`/etc/portage/packages.keywords`
	dev-php5/pecl-memcache ~x86
	dev-php/pear ~x86
	
然后安装
	emerge nginx php
	
这样就可以自动完成安装，安装完成后根据提示设置 MySQL 密码，然后配置 Nginx
`/etc/nginx/fastcgi_params`
	fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
	
`/etc/nginx/nginx.conf`
	server {
		listen 80;
		server_name localhost;
		root /var/www;
		index index.html index.php;
		
		location ~ ^(.+\.php)(.*)$ {
			fastcgi_pass 127.0.0.1:9000;
			fastcgi_index index.php;
			include fastcgi_params;
		}
	}
	
至此安装完成，启动
	/etc/init.d/nginx start
	/etc/init.d/php-fpm start
	rc-update add nginx default
	rc-update add php-fpm default
