title: "Remove internal WIFI interface from kenel for wpj344"
date: 2015-10-20 13:29:15
categories: "工作笔记"
tags: ["openwrt"]
---
To remove the internal radio of wpj344 from the kernel.

##Version Gen3:
1. Edit target/linux/ar71xx/files/arch/mips/ath79/mach-wpj344.c
2. Change Code:
{% codeblock lang:c %}
ath79_register_awl_wmac("Atheros AR934x", ATH79_IP2_IRQ(1),
                                                        KSEG1ADDR(AR934X_WMAC_BASE), AR934X_WMAC_SIZE, art_buf);
{% endcodeblock %}
to
<!--more-->
{% codeblock lang:c %}
ath79_register_awl_wmac(NULL, ATH79_IP2_IRQ(1),
                                                        KSEG1ADDR(AR934X_WMAC_BASE), AR934X_WMAC_SIZE, art_buf);
{% endcodeblock %}
3. Remove the code:
{% codeblock lang:c %}
ath79_register_wmac(art+WPJ344_WMAC_CALDATA_OFFSET,NULL);
{% endcodeblock %}
4. Recompile.


##Old Version, Gen1&2:
1. Edit target/linux/ar71xx/files/arch/mips/ath79/mach-wpj344.c
2. Remove the code: 
{% codeblock lang:c %}
ath79_register_awl_wmac("Atheros AR934x", ATH79_IP2_IRQ(1),
							KSEG1ADDR(AR934X_WMAC_BASE), AR934X_WMAC_SIZE, art_buf);
{% endcodeblock %}
3. Recompile.
