title: "OpenWrt下安装EasyCwmp (以及json-c版本问题)"
date: 2016-04-22 09:19:01
categories: "工作笔记"
tags:
- "openwrt"
- "TR-069"
- "CWMP"
- "easycwmp"
- "json-c"
---

##安装EasyCwmp到OpenWrt
1. **安装尚未安装的依赖**
  安装EasyCwmp所需要的依赖
  + libuci
  + libcurl
  + json-c：[可能会遇到版本问题，后附解决放案](#json-c版本问题).
  + libubox
  + libubus
  + libmicroxml：下载地址 http://easycwmp.org/download/libmicroxml.tar.gz
<!--more-->

  安装
		cd /openwrt目录/package/
		./scripts/feeds install libcurl
		./scripts/feeds install libmicroxml

  如果无法直接安装，下载并解压依赖安装信息包到OpenWrt。以libmicroxml为例：
		cd /openwrt目录/package/
		wget http://easycwmp.org/download/libmicroxml.tar.gz
		tar -xzvf libmicroxml.tar.gz
		cd ..

2. **下载并解压easycwmp安装信息包到OpenWrt**
		cd /openwrt目录/package/
		wget http://easycwmp.org/download/easycwmp-openwrt.tar.gz
		tar -xzvf easycwmp-openwrt.tar.gz
		cd ..

3. **添加到OpenWrt的配置并编译**
	- 编译为带有EasyCwmp的OpenWrt镜像
			make menuconfig
			[在Utilities里勾选easycwmp为<*>]
			make

	- 编译为单独的EasyCwmp模块
			make menuconfig
			[在Utilities里勾选easycwmp为<M>]
			make package/easycwmp/compile
	

##json-c版本问题
编译过程会遇到如下关于json-c的错误。

	......
	In file included from ../src/cwmp.c:17:0:
	../src/json.h:16:26: fatal error: json-c/json.h: No such file or directory
	compilation terminated.
	make[5]: *** [../src/easycwmpd-cwmp.o] Error 1
	......

因为尝试安装在OpenWrt的旧版本中，使用的json-c为0.9版本，鉴于其他依赖原因也不能更新。
而easycwmp已经更新为与新版json-c兼容。造成这个错误的原因仅仅是json-c的各版本名称不同。
json-c 0.9 的安装目录名为json，编译的库文件名称为libjson。
json-c 0.11/0.12 的安装目录名为json-c，编译的库文件名称为libjson-c。

为了和旧版json-c兼容。在EasyCwmp的原代码中将名称替换为旧版本使用的。
1. 在easycwmp原代码中，将external.c, json.c, json.h 中的宏
		#ifdef JSONC
		 #include <json-c/json.h>
		#else
		 #include <json/json.h>
		#endif
	更改为
		#include <json/json.h>

2. 在easycwmp原代码中，将configue.ac中的
		AC_ARG_ENABLE(jsonc, [AS_HELP_STRING([--enable-jsonc], [build with jsonc])], [
		 AC_DEFINE(JSONC)
		 LIBJSON_LIBS='-ljson-c'
		 AC_SUBST([LIBJSON_LIBS])
		], [
		 LIBJSON_LIBS='-ljson'
		 AC_SUBST([LIBJSON_LIBS])
		])
  改为
		AC_ARG_ENABLE(jsonc, [AS_HELP_STRING([--enable-jsonc], [build with jsonc])], [
		 LIBJSON_LIBS='-ljson'
		 AC_SUBST([LIBJSON_LIBS])
		])

3. 另外，需要更改 /openwrt目录/package/easycwmp/Makefile, 将
		define Package/easycwmp
			SECTION:=utils
			CATEGORY:=Utilities
			TITLE:=CWMP client (using libcurl)
			DEPENDS:=+libubus +libuci +libubox +libmicroxml +libjson-c +libcurl
		endef
  改为
		define Package/easycwmp
			SECTION:=utils
			CATEGORY:=Utilities
			TITLE:=CWMP client (using libcurl)
			DEPENDS:=+libubus +libuci +libubox +libmicroxml +libjson +libcurl
		endef

重新编译即可。

---
参考及相关
http://www.easycwmp.org/index.php/manual
http://www.shenjuanli.com/2015/05/16/安装-libubox-时遇到的-json-c-版本问题
http://support.easycwmp.org/view.php?id=9

