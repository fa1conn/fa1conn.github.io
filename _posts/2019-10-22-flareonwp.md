---
layout:     post
title:      "flare on6-challenge"
subtitle:   "flare on6-challenge"
date:       2019-10-22 12:00:00
author:     "fa1con"
header-img: ""
header-mask: 0.3
catalog:    true
tags:
    - re
    - bin
---
# 1：MemeCatBattlestation
搜字符，发现对应函数，可以查看调用函数
![image](https://github.com/fa1conn/fa1conn.github.io/tree/master/_posts/picture/1.png)
两关，stage1From和stage2From
![image](https://github.com/fa1conn/fa1conn.github.io/tree/master/_posts/picture/3.png)
明文存储
![image](https://github.com/fa1conn/fa1conn.github.io/tree/master/_posts/picture/2.png)
异或
```python
eql = [0x3,ord(' '),ord('&'),ord('$'),ord('-'),0x1e,0x2,ord(' '),ord('/'),ord('/'),ord('.'),ord('/')]
key = ""
for i in range(len(eql)):
    key += chr(eql[i]^ord('A'))
print "RAINBOW"
print key
```