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
![image](/picture/1.png)
两关，stage1From和stage2From
![image](/picture/3.png)
明文存储
![image](/picture/2.png)
异或
```python
eql = [0x3,ord(' '),ord('&'),ord('$'),ord('-'),0x1e,0x2,ord(' '),ord('/'),ord('/'),ord('.'),ord('/')]
key = ""
for i in range(len(eql)):
    key += chr(eql[i]^ord('A'))
print "RAINBOW"
print key
```
# 2 - Overlong
只弹出部分字符，将长度0x1C patch 成0x4C，运行得flag
![image](/picture/4.png)

# 3 - Flarebear
核心判断函数如下：
![image](/picture/5.png)
三个功能，z3解一下就好了
![image](/picture/6.png)
![image](/picture/7.png)
![image](/picture/8.png)
```python
from z3 import *
s = Solver()
feed = Int('feed')
play = Int('play')
clean = Int('clean')
s.add(feed * 10 - play * 2 == 72)
s.add(feed * 2 - clean * 1 + play * 4 == 30 )
s.add(feed * (-1) + clean * 6 - play * 1 == 0)
s.check()
m = s.model()
print m
```
[clean = 2, play = 4, feed = 8]
![image](/picture/9.png)

# 4 - Dnschess
猜测功能在.so里面，看到`getNextMove`函数，看到了拼接`.game-of-thrones.flare-on.com`,并调用`gethostbyname`，获得ip地址后判断ip地址是否符合，然后ip地址的内容与某个数组异或得到一组数，猜测是flag
分析UI文件，找到`dlsym`加载`getNextMove`函数,指针交叉引用到函数调用点，然后分析各个参数，第一个显然是计数，第二，三，四个是拼接的前半部分，最后一个应该是反馈
从pcap中提取所有的ip地址，然后依次循环找到符合条件的那个，生成flag就可以了
```python
from scapy.all import *
pcap = 'capture.pcap'
pkts = rdpcap(pcap)
list = []
for p in pkts:
    if p.haslayer(DNSRR):
        list.append(p[DNSRR].rdata)
#print list
ip = []
for i in range(len(list)):
    if i%2 == 1:
        ip.append(list[i])
print ip
flag = ""
txt = [ 0x79, 0x5a, 0xb8, 0xbc, 0xec, 0xd3, 0xdf, 0xdd, 0x99, 0xa5, 0xb6, 0xac, 0x15, 0x36, 0x85, 0x8d, 0x9, 0x8, 0x77, 0x52, 0x4d, 0x71, 0x54, 0x7d, 0xa7, 0xa7, 0x8, 0x16, 0xfd, 0xd7 ]
for i in range(15):
    for j in ip:
        tmp = j.split('.')
        #print tmp
        if int(tmp[0]) == 0x7f and int(tmp[3])&1 == 0 and int(tmp[2])&0xf == i:
            flag += chr(int(tmp[1]) ^ txt[i*2])
            flag += chr(int(tmp[1]) ^ txt[i*2 +1])
            break
print flag
# https://scapy.readthedocs.io/en/latest/
```

# 5 - demo
加壳了，可以在0x400015发现inc edi，edi初始值是0x420000，猜测解码的代码段是在这，循环十次过后，跳转到0x420000处看汇编，发现是壳中壳，在pushad处下断点，取消之前断点，断在了pushad处，脱掉壳中壳后dump程序，函数没有名字，在x32 debug或者od中可以看到部分call的函数名，在ida中手动更改函数名，发现调用了两次D3DXCreateMeshFVF，而sub_42038A处调用了一次D3DXMatriTranslation
```c
D3DXMATRIX * D3DXMatrixTranslation(
    D3DXMATRIX * pOut,
    FLOAT x,
    FLOAT y,
    FLOAT z
);
第一个参数是返回平移矩阵，第二个参数是X轴的平移量，第三个参数是Y轴的平移量，第四个参数是Z轴的平移量。
比如要沿着Z轴的正方向平移5个单位，就需要按下面来设置平移矩阵：
D3DXMATRIX mTrans;
D3DXMatrixTranslation(&mTrans,0,0,5);
```
那么把z修改小一点就可以了
![image](/picture/10.png)
可能需要稍微改大一点，这里就不搞了
![image](/picture/11.png)