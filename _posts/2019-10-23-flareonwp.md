---
layout:     post
title:      "flare on6-challenge"
subtitle:   "flare on6-challenge"
date:       2019-10-23 12:00:00
author:     "fa1con"
header-img: ""
header-mask: 0.3
catalog:    true
tags:
    - re
    - bin
---
# 1：MemeCatBattlestation
C#逆向，搜字符，发现对应函数，可以查看调用函数
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
windows逆向，只弹出部分字符，将长度0x1C patch 成0x4C，运行得flag
![image](/picture/4.png)

# 3 - Flarebear
android逆向，核心判断函数如下：
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
linux逆向，猜测功能在.so里面，看到`getNextMove`函数，看到了拼接`.game-of-thrones.flare-on.com`,并调用`gethostbyname`，获得ip地址后判断ip地址是否符合，然后ip地址的内容与某个数组异或得到一组数，猜测是flag
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

# 6 - bmphide
C#逆向，题目提醒了隐写
跳过，以后填坑

终于回来填坑了，更新日期：2019.11.14
dnspy简单看了一下，程序运行需要三个参数，分别是图片 数据 图片
先来看一下对数据进行处理的h函数
## h函数
### f函数
有点emm，先跳
### e函数
```cs
		public static byte e(byte b, byte k)
		{
			for (int i = 0; i < 8; i++)
			{
				bool flag = (b >> i & 1) == (k >> i & 1);
				if (flag)
				{
					b = (byte)((int)b & ~(1 << i) & 255);
				}
				else
				{
					b = (byte)((int)b | (1 << i & 255));
				}
			}
			return b;
		}
```
仔细一看就是xor而已
### a函数
```cs
		public static byte a(byte b, int r)
		{
			return (byte)(((int)b + r ^ r) & 255);
		}
```
啥都没干，屁用没有
### c函数
```cs
		public static byte c(byte b, int r)
		{
			byte b2 = 1;
			for (int i = 0; i < 8; i++)
			{
				bool flag = (b & 1) == 1;
				if (flag)
				{
					b2 = (b2 * 2 + 1 & byte.MaxValue);
				}
				else
				{
					b2 = (b2 - 1 & byte.MaxValue);
				}
			}
			return b2;
		}
```
没看懂在干啥，第二个参数也没用，第一个参数也只用了最后一位

到这里一切都很迷惑，想跑起来调试一下，不过在init的时候崩溃了，应该是加了反调试，尝试attach上去，但是因为程序运行过快失败了，将测试数据改大，改到100M以上，然后断点下在h函数，再次尝试，成功附加上去，结果发现根本没调用a，c，f函数之类的，这时候想给出题人寄刀片
![image](/picture/12.png)
![image](/picture/13.png)
![image](/picture/14.png)
然后继续看把
### b函数
```cs
		public static byte b(byte b, int r)
		{
			for (int i = 0; i < r; i++)
			{
				byte b2 = (b & 128) / 128;
				b = (b * 2 & byte.MaxValue) + b2;
			}
			return b;
		}
```
对应的py
```python
def b(b,r):
    for i in range(r):
        b2 = (b & 128) // 128
        b = (b * 2 & 255) + b2
    return b
```
r的值固定是7，那么应该是
```python
def b(b):
    for i in range(7):
        b2 = (b & 128) // 128
        b = (b * 2 & 255) + b2
    return b
```
这函数咋逆？
魔改一下函数得到`b = (b << 1 & 0xff) + (b >>7)`
这不就是循环左移吗，正向b移7位，那么反向只要移1位就可以
so
```python
def b_re(b):
    b2 = (b & 128) // 128
    b = (b * 2 & 255) + b2
    return b
```
### d函数
```python
def d(b, r):
    for i in range(r):
        b2 = (b & 1) * 128
        b = ((b // 2) & 255) + b2
    return b
```
r固定是3
可以看出是循环右移，花里胡哨
逆的时候移五次就好了
```python

```
### g函数
```cs
		public static byte g(int idx)
		{
			byte b = (byte)((long)(idx + 1) * (long)((ulong)-306674912));
			byte k = (byte)((idx + 2) * 1669101435);
			return Program.e(b, k);
		}
```
看汇编，对应的py
```python
def g(idx):
    b = ((idx+1) * 0x126B6FC5) & 0xff
    k = ((idx+2) * 0xC82C97D) & 0xff
    return b^k
```

### i函数
写文件的函数，也是lsb函数
化简一下函数
```cs
		public static void i(Bitmap bm, byte[] data)
		{
			int num = 0;
			for (int i = 0; i < bm.Width; i++)
			{
				for (int j = 0; j < bm.Height; j++)
				{
					bool flag = num > data.Length - 1;
					if (flag)
					{
						break;
					}
					Color pixel = bm.GetPixel(i, j);
					int red = ((int)pixel.R & 0xf8 | ((int)data[num] & 7);
					int green = ((int)pixel.G & 0xf8 | (data[num] >> 3 & 7);
					int blue = ((int)pixel.B & 0xfc) | (data[num] >> 6 & 3);
					Color color = Color.FromArgb(0, red, green, blue);
					bm.SetPixel(i, j, color);
					num += 1;
				}
			}
		}
```
每个字节的最后三位写入red低三位，3-5位写入green低三位，6-7写入blue低两位
写提取数据的脚本
附上整个脚本
```python
from PIL import Image
def b(b,r):
    for i in range(r):
        b2 = (b & 128) // 128
        b = (b * 2 & 255) + b2
        print b
    return b

def b_re(b):
    b2 = (b & 128) // 128
    b = (b * 2 & 255) + b2
    return b

def c(b,r):
    return b^r

def c_re(b,r):
    return b^r

def d(b, r):
    for i in range(r):
        b2 = (b & 1) * 128
        b = ((b // 2) & 255) + b2
        print b
    return b

def d_re(b):
    for i in range(5):
        b2 = (b & 1) * 128
        b = ((b // 2) & 255) + b2
        #print b
    return b
def g(idx):
    b = ((idx+1) * 0x126B6FC5) & 0xff
    k = ((idx+2) * 0xC82C97D) & 0xff
    return b^k

def h(byte,i):
    num3 = byte
    num3 = d_re(num3)
    num4 = g(i*2 + 1)
    num3 = c_re(num3,num4)
    num3 = b_re(num3)
    num2 = g(i*2)
    num3 = c_re(num2,num3)
    return num3
#def h():
#b(123,8)
#d(123,8)
cipher = []
img = Image.open('bmp.bmp')
width, height = img.size
data = []
fp = open('bmp2.bmp', 'wb')
for x in range(width):
    for y in range(height):
        red, green, blue = img.getpixel((x,y))
        color = (red & 7) | ((green & 7) << 3) | ((blue & 3) << 6)
        num = y + x * height
        #data.append(h(color, num))
        fp.write(chr(h(color, num)))
fp.close()
```
会发现处理一次过后的数据是一个图片，不过比原图大小小了好多
![image](/picture/15.png)
再处理一次打开可以看到flag
![image](/picture/16.png)