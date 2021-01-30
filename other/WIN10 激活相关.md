---
date: 2021-01-18 15:58:28
categories: 
 - win10激活
tags: 
 - win10激活
---

# WIN10 激活相关

```shell
# WIN10 pro
slmgr /ipk W269N-WFGWX-YVC9B-4J6C9-T83GX
slmgr /skms kms.03k.org # 公司可换成网关ip
slmgr /ato
```

<!-- more -->

```shell
# office
cd "C:\Program Files\Microsoft Office\Office16"
cscript ospp.vbs /sethst:kms.03k.org
cscript ospp.vbs /act
```

```shell
# 删除C盘临时文件
del /s /q %temp%
```

