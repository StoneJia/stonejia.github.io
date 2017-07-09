title: "MikroTik RB750上配置PPPoE服务器"
date: 2016-05-30 09:30:10
tags:
- "MikroTik"
- "PPPoE"
- "PPPoE Server"
- "Network"
---

本文记录用WinBox在MikroTik路由器上配置PPPoE服务器的步骤。
设备: MikroTik RouterBoard 750 (RB750)
工具: RouterOS WinBox

##Step 1
	启动WinBox, 登陆RB750。
##Step 2 (可选)
	选择左侧菜单中的Interfaces, 查看要配置的接口。
##Step 3 (可选)
	选择 IP -> Addresses, 查看RB750的IP设置。
##Step 3 设置DNS服务器
	选择 IP -> DNS。
	在Servers中设置DNS服务器。
	点击Apply确定。
##Step 3
	选择IP -> Routes, 设置网关。

---
##Step 6
	设置新的IP池，IP -> Pool。点击加号。
	Name设置为IP池的名称 (e.g. pppoe)。
	Addresses设置PPPoE用户的IP范围 (e.g. 192.168.168.3-192.168.168.254)。
	点击Apply确定。
	
##Step 7
	点击PPP。
	在Interface选项卡下，添加PPPoE Server Binding。
	点击Apply确定。

	在PPPoE选项卡下，点击加号, 添加PPPoE服务。
	Service Name: 设置为 pppoe。
	Interface: 选取要配置PPPoE服务器的接口。
	Default Profile: 选取default。
	Authentication: 只勾选pap，不勾选mschap2，mschap1和chap。




参考

