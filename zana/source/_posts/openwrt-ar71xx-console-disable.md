title: "Disable console in OpenWRT ar71xx platform"
date: 2015-10-20 11:11:49
categories: "工作笔记"
tags: ["openwrt"]
---

Normally, to diable the console for OpenWRT, just follow: 
http://wiki.openwrt.org/doc/recipes/terminate.console.on.serial.
However, it does not work well on ar71xx platform. We have to do it manually.
1. Edit target/linux/ar71xx/image/Makefile.
2. Replace 
{% codeblock lang:c %}
$(if $(1),board=$(1) )$(if $(2),console=$(2)$(COMMA)$(3)) 
{% endcodeblock %}
with 
{% codeblock lang:c %}
$(if $(1),board=$(1) )$(if $(2),console=null)
{% endcodeblock %}
3. Recompile.

References:
https://dev.openwrt.org/ticket/11243
