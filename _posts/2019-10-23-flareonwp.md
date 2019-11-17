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
#### f函数
有点emm，先跳
#### e函数
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
#### a函数
```cs
		public static byte a(byte b, int r)
		{
			return (byte)(((int)b + r ^ r) & 255);
		}
```
啥都没干，屁用没有
#### c函数
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
#### b函数
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
#### d函数
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
#### g函数
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

#### i函数
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

# 7 - wopr
![image](/picture/17.png)
怀疑是python打包的程序，ida打开字符串搜一下就确定了
pyinstxtractor解包
题目提醒找入口代码
![image](/picture/18.png)
这两个文件缺少头部，16字节
python-uncompyle6反编译

pyiboot01_bootstrap.py
```python
import sys, pyimod03_importers                                                                     
pyimod03_importers.install()                                                                       
import os                                                                                          
if not hasattr(sys, 'frozen'):                                                                     
    sys.frozen = True                                                                              
sys.prefix = sys._MEIPASS                                                                          
sys.exec_prefix = sys.prefix                                                                       
sys.base_prefix = sys.prefix                                                                       
sys.base_exec_prefix = sys.exec_prefix                                                             
VIRTENV = 'VIRTUAL_ENV'                                                                            
if VIRTENV in os.environ:                                                                          
    os.environ[VIRTENV] = ''                                                                       
    del os.environ[VIRTENV]                                                                        
python_path = []                                                                                   
for pth in sys.path:                                                                               
    if not os.path.isabs(pth):                                                                     
        pth = os.path.abspath(pth)                                                                 
    python_path.append(pth)                                                                        
    sys.path = python_path                                                                         
                                                                                                   
class NullWriter:                                                                                  
    softspace = 0                                                                                  
    encoding = 'UTF-8'                                                                             
                                                                                                   
    def write(*args):                                                                              
        pass                                                                                       
                                                                                                   
    def flush(*args):                                                                              
        pass                                                                                       
                                                                                                   
    def isatty(self):                                                                              
        return False                                                                               
                                                                                                   
                                                                                                   
if sys.stdout is None or sys.stdout.fileno() < 0:                                                  
    sys.stdout = NullWriter()                                                                      
if sys.stderr is None or sys.stderr.fileno() < 0:                                                  
    sys.stderr = NullWriter()                                                                      
try:                                                                                               
    import encodings                                                                               
except ImportError:                                                                                
    pass                                                                                           
                                                                                                   
if sys.warnoptions:                                                                                
    import warnings                                                                                
try:                                                                                               
    import ctypes, os                                                                              
    from ctypes import LibraryLoader, DEFAULT_MODE                                                 
                                                                                                   
    def _frozen_name(name):                                                                        
        if name:                                                                                   
            frozen_name = os.path.join(sys._MEIPASS, os.path.basename(name))                       
            if os.path.exists(frozen_name):                                                        
                name = frozen_name                                                                 
            return name                                                                            
                                                                                                   
                                                                                                   
    class PyInstallerImportError(OSError):                                                         
                                                                                                   
        def __init__(self, name):                                                                  
            self.msg = 'Failed to load dynlib/dll %r. Most probably this dynlib/dll was not found w
hen the application was frozen.' % name                                                            
            self.args = (self.msg,)                                                                
                                                                                                   
                                                                                                   
    class PyInstallerCDLL(ctypes.CDLL):                                                            
                                                                                                   
        def __init__(self, name, *args, **kwargs):                                                 
            name = _frozen_name(name)                                                              
            try:                                                                                   
                (super(PyInstallerCDLL, self).__init__)(name, *args, **kwargs)                     
            except Exception as base_error:                                                        
                try:                                                                               
                    raise PyInstallerImportError(name)                                             
                finally:                                                                           
                    base_error = None                                                              
                    del base_error                                                                 
                                                                                                   
                                                                                                   
    ctypes.CDLL = PyInstallerCDLL                                                                  
    ctypes.cdll = LibraryLoader(PyInstallerCDLL)                                                   
                                                                                                   
    class PyInstallerPyDLL(ctypes.PyDLL):                                                          
                                                                                                   
        def __init__(self, name, *args, **kwargs):                                                 
            name = _frozen_name(name)                                                              
            try:                                                                                   
                (super(PyInstallerPyDLL, self).__init__)(name, *args, **kwargs)                    
            except Exception as base_error:                                                        
                try:                                                                               
                    raise PyInstallerImportError(name)                                             
                finally:                                                                           
                    base_error = None                                                              
                    del base_error                                                                 
                                                                                                   
                                                                                                   
    ctypes.PyDLL = PyInstallerPyDLL                                                                
    ctypes.pydll = LibraryLoader(PyInstallerPyDLL)                                                 
    if sys.platform.startswith('win'):                                                             
                                                                                                   
        class PyInstallerWinDLL(ctypes.WinDLL):                                                    
                                                                                                   
            def __init__(self, name, *args, **kwargs):                                             
                name = _frozen_name(name)                                                          
                try:                                                                               
                    (super(PyInstallerWinDLL, self).__init__)(name, *args, **kwargs)               
                except Exception as base_error:                                                    
                    try:                                                                           
                        raise PyInstallerImportError(name)                                         
                    finally:                                                                       
                        base_error = None                                                          
                        del base_error                                                             
                                                                                                   
                                                                                                   
        ctypes.WinDLL = PyInstallerWinDLL                                                          
        ctypes.windll = LibraryLoader(PyInstallerWinDLL)                                           
                                                                                                   
        class PyInstallerOleDLL(ctypes.OleDLL):                           
            def __init__(self, name, *args, **kwargs):                    
                name = _frozen_name(name)                                 
                try:                                                      
                    (super(PyInstallerOleDLL, self).__init__)(name, *args, **kwargs)               
                except Exception as base_error:                           
                    try:                                                  
                        raise PyInstallerImportError(name)                
                    finally:                                              
                        base_error = None                                 
                        del base_error                                                              
        ctypes.OleDLL = PyInstallerOleDLL                                 
        ctypes.oledll = LibraryLoader(PyInstallerOleDLL)                  
except ImportError:                                                       
    pass       

if sys.platform.startswith('darwin'):                                     
    try:                                                                  
        from ctypes.macholib import dyld                                  
        dyld.DEFAULT_LIBRARY_FALLBACK.insert(0, sys._MEIPASS)             
    except ImportError:                                                                      
        pass                                                              
                                                                
    d = 'eggs'                                                           
    d = os.path.join(sys._MEIPASS, d)                                     
    if os.path.isdir(d):                                                 
        for fn in os.listdir(d):                                         
            sys.path.append(os.path.join(d, fn))                                                   
```


pyiboot02_cleanup.py
```python
# uncompyle6 version 3.5.0
# Python bytecode 3.7 (3394)
# Decompiled from: Python 2.7.15 (v2.7.15:ca079a3ea3, Apr 30 2018, 16:30:26) [MSC v.1500 64 bit (AMD64)]
# Embedded file name: pyiboot02_cleanup.py
# Size of source mod 2**32: 3062 bytes
"""
Once upon a midnight dreary, while I pondered, weak and weary,            
Over many a quaint and curious volume of forgotten lore-                  
While I nodded, nearly napping, suddenly there came a tapping,            
As of some one gently rapping, rapping at my chamber door-                
"'Tis some visitor," I muttered, "tapping at my chamber door-             
               Only this and nothing more."                               
Ah, distinctly I remember it was in the bleak December;                   
And each separate dying ember wrought its ghost upon the floor.           
Eagerly I wished the morrow;-vainly I had sought to borrow                
From my books surcease of sorrow-sorrow for the lost Lenore-              
For the rare and radiant maiden whom the angels name Lenore-              
               Nameless here for evermore.                                
And the silken, sad, uncertain rustling of each purple curtain            
Thrilled me-filled me with fantastic terrors never felt before;           
So that now, to still the beating of my heart, I stood repeating,         
"'Tis some visitor entreating entrance at my chamber door-                
Some late visitor entreating entrance at my chamber door;-                
               This it is and nothing more."                              
Presently my soul grew stronger; hesitating then no longer,               
"Sir," said I, "or Madam, truly your forgiveness I implore;               
But the fact is I was napping, and so gently you came rapping,            
And so faintly you came tapping, tapping at my chamber door,              
That I scarce was sure I heard you"-here I opened wide the door;-         
               Darkness there and nothing more.                           
Deep into that darkness peering, long I stood there wondering, fearing,   
Doubting, dreaming dreams no mortal ever dared to dream before;           
But the silence was unbroken, and the stillness gave no token,            
And the only word there spoken was the whispered word, "Lenore?"          
This I whispered, and an echo murmured back the word, "Lenore!"-          
               Merely this and nothing more.                              
Back into the chamber turning, all my soul within me burning,             
Soon again I heard a tapping somewhat louder than before.                 
"Surely," said I, "surely that is something at my window lattice;         
Let me see, then, what thereat is, and this mystery explore-              
Let my heart be still a moment and this mystery explore;-                 
               'Tis the wind and nothing more!"                           
Open here I flung the shutter, when, with many a flirt and flutter,       
In there stepped a stately Raven of the saintly days of yore;             
Not the least obeisance made he; not a minute stopped or stayed he;       
But, with mien of lord or lady, perched above my chamber door-            
Perched upon a bust of Pallas just above my chamber door-                 
                Perched, and sat, and nothing more.                       
Then this ebony bird beguiling my sad fancy into smiling,                 
By the grave and stern decorum of the countenance it wore,                
"Though thy crest be shorn and shaven, thou," I said, "art sure no craven,
Ghastly grim and ancient Raven wandering from the Nightly shore-          
Tell me what thy lordly name is on the Night's Plutonian shore!"          
               Quoth the Raven "Nevermore."                               
Much I marvelled this ungainly fowl to hear discourse so plainly,         
Though its answer little meaning-little relevancy bore;                   
For we cannot help agreeing that no living human being                    
Ever yet was blest with seeing bird above his chamber door-                    
Bird or beast upon the sculptured bust above his chamber door,            
               With such name as "Nevermore."                                                                        
But the Raven, sitting lonely on the placid bust, spoke only              
That one word, as if his soul in that one word he did outpour.            
Nothing further then he uttered-not a feather then he fluttered-          
Till I scarcely more than muttered "Other friends have flown before-      
On the morrow he will leave me, as my hopes have flown before."                                                        
               Then the bird said "Nevermore."                                                             
Startled at the stillness broken by reply so aptly spoken,                                    
"Doubtless," said I, "what it utters is its only stock and store                                                                                                                     
Caught from some unhappy master whom unmerciful Disaster                                                                                
Followed fast and followed faster till his songs one burden bore-                                                              
Till the dirges of his Hope that melancholy burden bore                   
               Of 'Never-nevermore.'"                                                                                                                                   
But the Raven still beguiling my sad fancy into smiling,                                    
Straight I wheeled a cushioned seat in front of bird, and bust and door;  
Then, upon the velvet sinking, I betook myself to linking                 
Fancy unto fancy, thinking what this ominous bird of yore-                
What this grim, ungainly, ghastly, gaunt and ominous bird of yore                                                                                                                                                                                                  
               Meant in croaking "Nevermore."                                                     
This I sat engaged in guessing, but no syllable expressing                                                                                                                                                 
To the fowl whose fiery eyes now burned into my bosom's core;                                                                                                                            
This and more I sat divining, with my head at ease reclining                                                                                                                                                                                                
On the cushion's velvet lining that the lamp-light gloated o'er,                                                                         
But whose velvet violet lining with the lamp-light gloating o'er,         
               She shall press, ah, nevermore!                            
Then, methought, the air grew denser, perfumed from an unseen censer                           
Swung by Seraphim whose foot-falls tinkled on the tufted floor.           
"Wretch," I cried, "thy God hath lent thee-by these angels he hath sent thee                                                                                                                                                         
Respite-respite and nepenthe, from thy memories of Lenore;                                                                                                                                                               
Quaff, oh quaff this kind nepenthe and forget this lost Lenore!"          
               Quoth the Raven "Nevermore."                                                              
"Prophet!" said I, "thing of evil!-prophet still, if bird or devil!-                                                                                                                                                                                                
Whether Tempter sent, or whether tempest tossed thee here ashore,                            
Desolate yet all undaunted, on this desert land enchanted-                                                    
On this home by Horror haunted-tell me truly, I implore-                          
Is there-is there balm in Gilead?-tell me-tell me, I implore!"                                                   
               Quoth the Raven "Nevermore."                               
                   
"Prophet!" said I, "thing of evil-prophet still, if bird or devil!                                                                                                                                    
By that Heaven that bends above us-by that God we both adore-                                                                                                                                                                                                                                                            
Tell this soul with sorrow laden if, within the distant Aidenn,                                                                                                                                                    
It shall clasp a sainted maiden whom the angels name Lenore-                                                          
Clasp a rare and radiant maiden whom the angels name Lenore."                                             
                Quoth the Raven "Nevermore."                              
"Be that word our sign in parting, bird or fiend!" I shrieked, upstarting-                                                               
"Get thee back into the tempest and the Night's Plutonian shore!          
Leave no black plume as a token of that lie thy soul hath spoken!                                                        
Leave my loneliness unbroken!-quit the bust above my door!                
Take thy beak from out my heart, and take thy form from off my door!"     
               Quoth the Raven "Nevermore."                                                           
And the Raven, never flitting, still is sitting, still is sitting         
On the pallid bust of Pallas just above my chamber door;                  
And his eyes have all the seeming of a demon's that is dreaming,          
And the lamp-light o'er him streaming throws his shadow on the floor;     
And my soul from out that shadow that lies floating on the floor          
               Shall be lifted-nevermore!                               """
import hashlib, io, lzma, pkgutil, random, struct, sys, time
from ctypes import *
print('LOADING...')
BOUNCE = pkgutil.get_data('this', 'key')

def ho(h, g={}):
    k = bytes.fromhex(format(h, 'x')).decode()
    return g.get(k, k)


a = 1702389091
b = 482955849332L
g = ho(29516388843672123817340395359L, globals()) #__builtins__
aa = getattr(g, ho(a))           #exec
bb = getattr(g, ho(b))           #print
a ^= b
b ^= a
a ^= b#交换a和b
setattr(g, ho(a), aa)
setattr(g, ho(b), bb) #应该是交换函数地址，即print变exec，exec变print

def eye(face):
    leg = io.BytesIO()
    for arm in face.splitlines():
        arm = arm[len(arm.rstrip(' \t')):]
        leg.write(arm)

    face = leg.getvalue()
    bell = io.BytesIO()
    x, y = (0, 0)
    for chuck in face:
        taxi = {9:0, 
         32:1}.get(chuck)
        if taxi is None:
            continue
        x, y = x | taxi << y, y + 1
        if y > 7:
            bell.write(bytes([x]))
            x, y = (0, 0)

    return bell.getvalue()


def fire(wood, bounce):
    meaning = bytearray(wood)
    bounce = bytearray(bounce)
    regard = len(bounce)
    manage = list(range(256))

    def prospect(*financial):
        return sum(financial) % 256

    def blade(feel, cassette):
        cassette = prospect(cassette, manage[feel])
        manage[feel], manage[cassette] = manage[cassette], manage[feel]
        return cassette

    cassette = 0
    for feel in range(256):
        cassette = prospect(cassette, bounce[(feel % regard)])
        cassette = blade(feel, cassette)

    cassette = 0
    for pigeon, _ in enumerate(meaning):
        feel = prospect(pigeon, 1)
        cassette = blade(feel, cassette)
        meaning[pigeon] ^= manage[prospect(manage[feel], manage[cassette])]

    return bytes(meaning)


for i in range(256):
    try:
        print(lzma.decompress(fire(eye(__doc__.encode()), bytes([i]) + BOUNCE)))#exec执行代码
    except Exception:
        pass
# okay decompiling pyiboot02_cleanup.pyc

```
交换print和exec，然后取__doc__数据和key进行一些变换（盲猜是rc4）
doc数据应该是上面那一堆
折腾了半天也没把数据弄出来，一直报类型错误，查看别人的wp解法差不多，心累
于是先直接拿别人反编译好的看吧
```python
GREETINGS = ["HI", "HELLO", "'SUP", "AHOY", "ALOHA", "HOWDY", "GREETINGS", "ZDRAVSTVUYTE"]
STRATEGIES = ['U.S. FIRST STRIKE', 'USSR FIRST STRIKE', 'NATO / WARSAW PACT', 'FAR EAST STRATEGY', 'US USSR ESCALATION', 'MIDDLE EAST WAR', 'USSR CHINA ATTACK', 'INDIA PAKISTAN WAR', 'MEDITERRANEAN WAR', 'HONGKONG VARIANT', 'SEATO DECAPITATING', 'CUBAN PROVOCATION', 'ATLANTIC HEAVY', 'CUBAN PARAMILITARY', 'NICARAGUAN PREEMPTIVE', 'PACIFIC TERRITORIAL', 'BURMESE THEATERWIDE', 'TURKISH DECOY', 'ARGENTINA ESCALATION', 'ICELAND MAXIMUM', 'ARABIAN THEATERWIDE', 'U.S. SUBVERSION', 'AUSTRALIAN MANEUVER', 'SUDAN SURPRISE', 'NATO TERRITORIAL', 'ZAIRE ALLIANCE', 'ICELAND INCIDENT', 'ENGLISH ESCALATION', 'MIDDLE EAST HEAVY', 'MEXICAN TAKEOVER', 'CHAD ALERT', 'SAUDI MANEUVER', 'AFRICAN TERRITORIAL', 'ETHIOPIAN ESCALATION', 'TURKISH HEAVY', 'NATO INCURSION', 'U.S. DEFENSE', 'CAMBODIAN HEAVY', 'PACT MEDIUM', 'ARCTIC MINIMAL', 'MEXICAN DOMESTIC', 'TAIWAN THEATERWIDE', 'PACIFIC MANEUVER', 'PORTUGAL REVOLUTION', 'ALBANIAN DECOY', 'PALESTINIAN LOCAL', 'MOROCCAN MINIMAL', 'BAVARIAN DIVERSITY', 'CZECH OPTION', 'FRENCH ALLIANCE', 'ARABIAN CLANDESTINE', 'GABON REBELLION', 'NORTHERN MAXIMUM', 'DANISH PARAMILITARY', 'SEATO TAKEOVER', 'HAWAIIAN ESCALATION', 'IRANIAN MANEUVER', 'NATO CONTAINMENT', 'SWISS INCIDENT', 'CUBAN MINIMAL', 'CHAD ALERT', 'ICELAND ESCALATION', 'VIETNAMESE RETALIATION', 'SYRIAN PROVOCATION', 'LIBYAN LOCAL', 'GABON TAKEOVER', 'ROMANIAN WAR', 'MIDDLE EAST OFFENSIVE', 'DENMARK MASSIVE', 'CHILE CONFRONTATION', 'S.AFRICAN SUBVERSION', 'USSR ALERT', 'NICARAGUAN THRUST', 'GREENLAND DOMESTIC', 'ICELAND HEAVY', 'KENYA OPTION', 'PACIFIC DEFENSE', 'UGANDA MAXIMUM', 'THAI SUBVERSION', 'ROMANIAN STRIKE', 'PAKISTAN SOVEREIGNTY', 'AFGHAN MISDIRECTION', 'ETHIOPIAN LOCAL', 'ITALIAN TAKEOVER', 'VIETNAMESE INCIDENT', 'ENGLISH PREEMPTIVE', 'DENMARK ALTERNATE', 'THAI CONFRONTATION', 'TAIWAN SURPRISE', 'BRAZILIAN STRIKE', 'VENEZUELA SUDDEN', 'MALAYSIAN ALERT', 'ISREAL DISCRETIONARY', 'LIBYAN ACTION', 'PALESTINIAN TACTICAL', 'NATO ALTERNATE', 'CYPRESS MANEUVER', 'EGYPT MISDIRECTION', 'BANGLADESH THRUST', 'KENYA DEFENSE', 'BANGLADESH CONTAINMENT', 'VIETNAMESE STRIKE', 'ALBANIAN CONTAINMENT', 'GABON SURPRISE', 'IRAQ SOVEREIGNTY', 'VIETNAMESE SUDDEN', 'LEBANON INTERDICTION', 'TAIWAN DOMESTIC', 'ALGERIAN SOVEREIGNTY', 'ARABIAN STRIKE', 'ATLANTIC SUDDEN', 'MONGOLIAN THRUST', 'POLISH DECOY', 'ALASKAN DISCRETIONARY', 'CANADIAN THRUST', 'ARABIAN LIGHT', 'S.AFRICAN DOMESTIC', 'TUNISIAN INCIDENT', 'MALAYSIAN MANEUVER', 'JAMAICA DECOY', 'MALAYSIAN MINIMAL', 'RUSSIAN SOVEREIGNTY', 'CHAD OPTION', 'BANGLADESH WAR', 'BURMESE CONTAINMENT', 'ASIAN THEATERWIDE', 'BULGARIAN CLANDESTINE', 'GREENLAND INCURSION', 'EGYPT SURGICAL', 'CZECH HEAVY', 'TAIWAN CONFRONTATION', 'GREENLAND MAXIMUM', 'UGANDA OFFENSIVE', 'CASPIAN DEFENSE', 'CRIMEAN GAMBIT', 'BRITISH ANTICS', 'HUNGARIAN EXPULSION', 'VENEZUELAN COLLAPSE']

def wrong():
    trust = windll.kernel32.GetModuleHandleW(None)

    computer = string_at(trust, 1024)
    dirty, = struct.unpack_from('=I', computer, 60)

    _, _, organize, _, _, _, variety, _ =  struct.unpack_from('=IHHIIIHH', computer, dirty)
    assert variety >= 144

    participate, = struct.unpack_from('=I', computer, dirty + 40)
    for insurance in range(organize):
        name, tropical, inhabitant, reader, chalk, _, _, _, _, _ = struct.unpack_from('=8sIIIIIIHHI', computer, 40 * insurance + dirty + variety + 24)
        if inhabitant <= participate < inhabitant + tropical:
            break

    spare = bytearray(string_at(trust + inhabitant, tropical))
    
    issue, digital = struct.unpack_from('=II', computer, dirty + 0xa0)
    truth = string_at(trust + issue, digital)

    expertise = 0
    while expertise <= len(truth) - 8:
        nuance, seem = struct.unpack_from('=II', truth, expertise)

        if nuance == 0 and seem == 0:
            break

        slot = truth[expertise + 8:expertise + seem]

        for i in range(len(slot) >> 1):
            diet, = struct.unpack_from('=H', slot, 2 * i)
            fabricate = diet >> 12
            if fabricate != 3: continue
            diet = diet & 4095
            ready = nuance + diet - inhabitant
            if 0 <= ready < len(spare): 
                struct.pack_into('=I', spare, ready, struct.unpack_from('=I', spare, ready)[0] - trust)

        expertise += seem

    return hashlib.md5(spare).digest()

class Terminal(object):
        
    DELAY = 0.02

    def write(self, text):
        for line in text.splitlines(True):
            sys.stdout.write(line)
            sys.stdout.flush()
            time.sleep(self.DELAY)     

    def typewrite(self, text):
        for char in text:
            if char == '\n':
                sys.stdout.write(char)
                sys.stdout.flush()
                time.sleep(self.DELAY)
            else:
                sys.stdout.write(char.lower())
                sys.stdout.flush()
                time.sleep(self.DELAY)
                sys.stdout.write('\b' + char)
                sys.stdout.flush()
        
    def typewriteln(self, text):
        self.typewrite(text + '\n')
        
    def read(self):
        return ' '.join(''.join(_ for _ in input().upper() if _ in ' 0123456789ABCDEFGHIJKLMNOPQRSTUVWXZY?').split())
t = Terminal()

xor = [212, 162, 242, 218, 101, 109, 50, 31, 125, 112, 249, 83, 55, 187, 131, 206]
h = list(wrong())
h = [h[i] ^ xor[i] for i in range(16)]

t.write('''
      _/\/\______/\/\____/\/\/\/\____/\/\/\/\/\____/\/\/\/\/\___
     _/\/\__/\__/\/\__/\/\____/\/\__/\/\____/\/\__/\/\____/\/\_
    _/\/\/\/\/\/\/\__/\/\____/\/\__/\/\/\/\/\____/\/\/\/\/\___
   _/\/\/\__/\/\/\__/\/\____/\/\__/\/\__________/\/\__/\/\___
  _/\/\______/\/\____/\/\/\/\____/\/\__________/\/\____/\/\_
 __________________________________________________________

''')

t.typewrite('GREETINGS PROFESSOR FALKEN.\n')

while True:
    t.typewrite('\n> ')
    cmd = t.read()
    if cmd.rstrip('!?') in GREETINGS:
        t.typewriteln(random.choice(GREETINGS))
    elif cmd == 'HELP GAMES':
        t.typewriteln("'GAMES' REFERS TO MODELS, SIMULATIONS AND GAMES\nWHICH HAVE TACTICAL AND STRATEGIC APPLICATIONS.")
    elif cmd == 'LIST GAMES':
        t.typewriteln('FALKEN\'S MAZE\nTIC-TAC-TOE\nGLOBAL THERMONUCLEAR WAR')
    elif cmd in ('HELP', '?'):
        t.typewriteln('AVAILABLE COMMANDS:\nHELP\nHELP GAMES\nLIST GAMES\nPLAY <game>')
    elif cmd.startswith('HELP '):
        t.typewriteln('HELP NOT AVAILABLE')
    elif cmd == 'PLAY':
        t.typewriteln('WHICH GAME?')
    elif cmd.startswith('PLAY F') or cmd == 'PLAY 1':
        t.typewriteln('GAME IS TEMPORARILY UNAVAILABLE DUE TO MAINTENANCE')
    elif cmd.startswith('PLAY T') or cmd == 'PLAY 2':
        t.typewriteln('GAME IS TEMPORARILY UNAVAILABLE DUE TO MAINTENANCE')
    elif cmd.startswith('PLAY G') or cmd in ('PLAY ARMAGEDDON', 'PLAY 3'):
        t.typewriteln('*** GAME ROUTINE RUNNING ***')
        break
    elif cmd.startswith('PLAY '):
        t.typewriteln('THAT GAME IS NOT AVAILABLE')
    else:
        t.typewriteln('COMMAND NOT RECOGNIZED')


t.write('''
r"""""""""""""""""""7ooooo"""oooooo"""""""""""""""""""""""""""""""""""""""""""7
|           .__Looooooo ""7oooooooo`     'ooo"   ""._,    .JooL_,    .___     |
o  __L______oLoooooooo7o_, |oooor""       ._____,,Jo__JoooooooooooooJoooL_____J
r7._ooooooooooooooo"JoJoo|  oor   'o`   .Jooo7oooooooooooooooooooooooooooo"oo"7
| '`"'`   ooooooooooL,Jooo_,         _oL.ooLoooooooooooooooooooooooooo_  |r`  |
|         ''ooooooooooooooJo         ""oooooooooor"ooooooooooooooooooo7       |
|           7oooooooooo"             |or`oo'ooJoJo |ooooooooooooooro ./       |
|            "oooooo7o`              JooooJ_JLJoooLJoooooooooooooo, or        |
|             '"oo|  oo            .oooooooooooroooJo"7oooooooooo7,           |
|   ""          ""`oorL|L,         |ooooooooooooooo"`  7or` 7ooo |,           |
|                   '\_oL__         7ooooooooooooLr     7|   L"` |o|          |
|                    .Jooooo|            7ooooooo"       `  'o|_oL"L          |
|                    |oooooooooL         'oooooo|            'o_J7Lrooo_J_,   |
|                     7oooooooo`          Jooooo|._            "`"7LJ/7r  "`, |
|                       oooooor           7oooo|.o|             __oooooo_  _| 7
|                      |ooooo             'ooor  "              7oooooooo|    |
|                      Jooor               |o|                  '""  "oor    .J
|                      oo|                                            "o|   _oo
|                     'or _                           |                     " |
|                      ""`                                                    |
|                       .,'                       ___.   .______________      |
|        ______________ooo`        ._JLooooooooooooooooooooooooooooooooooooo" |
|  |L7ooooooooooooooooL___,.Jo_Jooooooooooooooooooooooooooooooooooooooooooor` |
ooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooo

AWAITING FIRST STRIKE COMMAND
-----------------------------
                
PLEASE SPECIFY PRIMARY TARGET
BY CITY AND/OR COUNTRY NAME:

''')
target = input()

t.typewriteln("\nPREPARING NUCLEAR STRIKE FOR " + target.upper())
t.typewrite("ENTER LAUNCH CODE: ")
launch_code = input().encode()

# encoding map coordinates
x = list(launch_code.ljust(16, b'\0'))
b = 16 * [None]

# calculate missile trajectory
b[0] = x[2] ^ x[3] ^ x[4] ^ x[8] ^ x[11] ^ x[14]
b[1] = x[0] ^ x[1] ^ x[8] ^ x[11] ^ x[13] ^ x[14]
b[2] = x[0] ^ x[1] ^ x[2] ^ x[4] ^ x[5] ^ x[8] ^ x[9] ^ x[10] ^ x[13] ^ x[14] ^ x[15]
b[3] = x[5] ^ x[6] ^ x[8] ^ x[9] ^ x[10] ^ x[12] ^ x[15]
b[4] = x[1] ^ x[6] ^ x[7] ^ x[8] ^ x[12] ^ x[13] ^ x[14] ^ x[15]
b[5] = x[0] ^ x[4] ^ x[7] ^ x[8] ^ x[9] ^ x[10] ^ x[12] ^ x[13] ^ x[14] ^ x[15]
b[6] = x[1] ^ x[3] ^ x[7] ^ x[9] ^ x[10] ^ x[11] ^ x[12] ^ x[13] ^ x[15]
b[7] = x[0] ^ x[1] ^ x[2] ^ x[3] ^ x[4] ^ x[8] ^ x[10] ^ x[11] ^ x[14]
b[8] = x[1] ^ x[2] ^ x[3] ^ x[5] ^ x[9] ^ x[10] ^ x[11] ^ x[12]
b[9] = x[6] ^ x[7] ^ x[8] ^ x[10] ^ x[11] ^ x[12] ^ x[15]
b[10] = x[0] ^ x[3] ^ x[4] ^ x[7] ^ x[8] ^ x[10] ^ x[11] ^ x[12] ^ x[13] ^ x[14] ^ x[15]
b[11] = x[0] ^ x[2] ^ x[4] ^ x[6] ^ x[13]
b[12] = x[0] ^ x[3] ^ x[6] ^ x[7] ^ x[10] ^ x[12] ^ x[15]
b[13] = x[2] ^ x[3] ^ x[4] ^ x[5] ^ x[6] ^ x[7] ^ x[11] ^ x[12] ^ x[13] ^ x[14]
b[14] = x[1] ^ x[2] ^ x[3] ^ x[5] ^ x[7] ^ x[11] ^ x[13] ^ x[14] ^ x[15]
b[15] = x[1] ^ x[3] ^ x[5] ^ x[9] ^ x[10] ^ x[11] ^ x[13] ^ x[15]

if b == h:
    t.typewriteln("LAUNCH CODE ACCEPTED.\n\n*** RUNNING SIMULATION ***\n")
    random.shuffle(STRATEGIES)
    for i in range(0, len(STRATEGIES), 6):
        t.write('\n'.join('{:24} {:8}'.format(k, v) for k, v in ([('STRATEGY:', 'WINNER:'), ('-' * 24, '-' * 8)] + [(_, 'NONE') for _ in STRATEGIES[i:i+6]])) + '\n\n')
        time.sleep(0.5)
    t.typewriteln("*** SIMULATION COMPLETED ***\n")
    t.typewriteln('\nA STRANGE GAME.\nTHE ONLY WINNING MOVE IS\nNOT TO PLAY.\n')
    eye = [219, 232, 81, 150, 126, 54, 116, 129, 3, 61, 204, 119, 252, 122, 3, 209, 196, 15, 148, 173, 206, 246, 242, 200, 201, 167, 2, 102, 59, 122, 81, 6, 24, 23]
    flag = fire(eye, launch_code).decode()
    t.typewrite(f"CONGRATULATIONS! YOU FOUND THE FLAG:\n\n{flag}\n")
else:
    t.typewrite("\nIDENTIFICATION NOT RECOGNIZED BY SYSTEM\n--CONNECTION TERMINATED--\n")
```
注意到b == h
wrong函数里第一行`windll.kernel32.GetModuleHandleW(None)`，参数是None的时候返回进程本身的句柄，所以应该先加载这个程序，然后脚本去读取句柄
wrong()函数第一行改成这样,然后挂载程序，必须注意的是程序是32位的，所以python3对应的也必须是
`trust = pydll.LoadLibrary(r'C:\Users\fa1con\learn\ctf\Flare-On6_Challenges\7 - wopr\7 - wopr\wopr.exe')._handle`
然后z3解
```
from z3 import *
b = [115, 29, 32, 68, 106, 108, 89, 76, 21, 71, 78, 51, 75, 1, 55, 102]
x=[BitVec('x[%d]' % i,8) for i in range(len(b))]
s = Solver()
s.add(b[0] == x[2] ^ x[3] ^ x[4] ^ x[8] ^ x[11] ^ x[14])
s.add(b[1] == x[0] ^ x[1] ^ x[8] ^ x[11] ^ x[13] ^ x[14])
s.add(b[2] == x[0] ^ x[1] ^ x[2] ^ x[4] ^ x[5] ^ x[8] ^ x[9] ^ x[10] ^ x[13] ^ x[14] ^ x[15])
s.add(b[3] == x[5] ^ x[6] ^ x[8] ^ x[9] ^ x[10] ^ x[12] ^ x[15])
s.add(b[4] == x[1] ^ x[6] ^ x[7] ^ x[8] ^ x[12] ^ x[13] ^ x[14] ^ x[15])
s.add(b[5] == x[0] ^ x[4] ^ x[7] ^ x[8] ^ x[9] ^ x[10] ^ x[12] ^ x[13] ^ x[14] ^ x[15])
s.add(b[6] == x[1] ^ x[3] ^ x[7] ^ x[9] ^ x[10] ^ x[11] ^ x[12] ^ x[13] ^ x[15])
s.add(b[7] == x[0] ^ x[1] ^ x[2] ^ x[3] ^ x[4] ^ x[8] ^ x[10] ^ x[11] ^ x[14])
s.add(b[8] == x[1] ^ x[2] ^ x[3] ^ x[5] ^ x[9] ^ x[10] ^ x[11] ^ x[12])
s.add(b[9] == x[6] ^ x[7] ^ x[8] ^ x[10] ^ x[11] ^ x[12] ^ x[15])
s.add(b[10] == x[0] ^ x[3] ^ x[4] ^ x[7] ^ x[8] ^ x[10] ^ x[11] ^ x[12] ^ x[13] ^ x[14] ^ x[15])
s.add(b[11] == x[0] ^ x[2] ^ x[4] ^ x[6] ^ x[13])
s.add(b[12] == x[0] ^ x[3] ^ x[6] ^ x[7] ^ x[10] ^ x[12] ^ x[15])
s.add(b[13] == x[2] ^ x[3] ^ x[4] ^ x[5] ^ x[6] ^ x[7] ^ x[11] ^ x[12] ^ x[13] ^ x[14])
s.add(b[14] == x[1] ^ x[2] ^ x[3] ^ x[5] ^ x[7] ^ x[11] ^ x[13] ^ x[14] ^ x[15])
s.add(b[15] == x[1] ^ x[3] ^ x[5] ^ x[9] ^ x[10] ^ x[11] ^ x[13] ^ x[15])

print s.check()
m = s.model()
print m
```
![image](/picture/19.png)

之前没接触过，感觉有点难，[参考](https://bestwing.me/flare-on-6th-writeup.html#7-wopr)