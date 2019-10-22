---
layout:     post
title:      "看雪ctf2019 Q3"
subtitle:   "kanxue CTF 2019 Q3"
date:       2019-10-22 12:00:00
author:     "fa1con"
header-img: ""
header-mask: 0.3
catalog:    true
tags:
    - CTF
    - re
    - bin
---
前段时间忙(摸)，没打比赛，emmm，抽空补一下
# 签到题：乱世鬼雄
用户名的十六进制编码后与序列号异或，然后看到md5的常量，猜测是md5，然后试了一下发现确实是md5，KCTF补0与前面异或的结果再异或回去就是答案了
```python
import hashlib
print "5D78C3FDF21998AC".encode('hex')

test = 0xF3A0FD8D8DE1FEB889808A8FF2D7FDA2 ^ 0x35443738433346444632313939384143
print hex(test)[2:-1]
print hex(test)[2:-1].decode('hex')
print hashlib.md5(hex(test)[2:-1].decode('hex')).hexdigest()
print hashlib.md5(hex(test)).hexdigest()

print ("KCTF"+'\x00'*12).encode('hex')

serile = 0x4b435446000000000000000000000000 ^ test
print "user:KCTF"
print "serile:" + hex(serile)[2:-1].upper()
```

# 第五题：魅影舞姬
`std::mersenne_twister_engine`梅森旋转算法
xxtea算法
key:0x67452301,0xEFCDAB89,0x98BADCFE,0x10325476

然后发现flag在checkfun验证过后在下面又验证了一次，直接看下面的部分
substr 24h，猜测flag长度为36
4个一次循环，所以O0oo0oO0O()是base64decode，解密完长度应该是27
flag取前24字节切割成三段，每段8字节
然后函数名被混淆了，emmm，PEID有个加密常量检测，检测到有DES table
看了一下，flag的三段是当作key使用了，取base64decode完的后9位，用三个key进行3DES，然后覆盖base64decode后9位
base64decode后的27bit划分成6，6，6，8
迷宫
第一个
* * * * * * * * * * * * *                         
* @ * * * * * * * * * * *
* - * * * * * * * * * * *
* - - * * - - - - * * * *                         
* - * * * - * * - * * * *
* - * * * # * * - * * * *                         
* - - * * * * * - * * * *                        
* * - * * * * * - * * * *                        
* * - - - - - - - * * * *                        
* * - * - - - - - * * * *                       
* * - - - * * - * * * * *                        
* * - * * * * - - * * * *                        
* * * * * * * * * * * * *
ssss sdss dddd ddww wwwa aass

第二个
* * * * * * * * * * * * *                      
* @ * * * * * * * * * * *                    
* - * * * * * * * * * * *                      
* - * * * - * * - * * * *                      
* - - * * - - - - * * * *                       
* - * * * # * * - * * * *                       
* - - * * * * * - * * * *                       
* * - * * * * * - * * * *                       
* * - * - - - - - * * * *                        
* * - - - * * - * * * * *                       
* * - - - - - - - * * * *                        
* * - * * * * - - * * * *                        
* * * * * * * * * * * * *
ssss sdss sddw dddd wwww aaas

第三个
* * * * * * * * * * * * *                 
* @ * * * * * * * * * * *                 
* - * * * * * * * * * * *                    
* - * * * - * * - * * * *                     
* - - * * * * * - * * * *                      
* - * * * # * * - * * * *                        
* - - * * - - - - * * * *                        
* * - * * * * * - * * * *                         
* * - * * * * - - * * * *                         
* * - - - * * - * * * * *                         
* * - * - - - - - * * * *                         
* * - - - - - - - * * * *                         
* * * * * * * * * * * * *
ssss sdss sdds dddw wdww aaaw

第四个
* * * * * * * * * * * * *              
* * - - - - - - - * * * *               
* - - * * * * * - * * * *                 
* - * * * - * * - * * * *                   
* - - * * * * * - * * * *                   
* - * * * # * * - * * * *                    
* - * * * - * * - * * * *                    
* - * * * - * * - * * * *                     
* - - * * - - - - * * * *                       
* * - - - * * - * * * * *                         
* - - * * * * * - * * * *                         
* @ * * * * * * * * * * *                        
* * * * * * * * * * * * *
wdww awww wwwd wddd ddds ssss ssaa awww
解出来就是
```
0x5575fff002a5L
0x55757cff00a9L
0x55757dfc30a8L
0x3080033ffd555a80L
```
前面三个是flag base64decode过的，所以应该base64encode,解出来是`VXX/8AKlVXV8/wCpVXV9/DCo`,然后解最后一段，再base64encode，下面有md5验证，不管了
```python
def tobin(txt):
    t = ''
    for i in range(len(txt)):
        if txt[i] == 'w':
            t += '00'
        if txt[i] == 's':
            t += '01'
        if txt[i] == 'a':
            t += '10'
        if txt[i] == 'd':
            t += '11'
    #print t
    print hex(int(t,2))[2:-1]
    return hex(int(t,2))[2:-1].decode('hex').encode('base64')
t1 = 'sssssdssddddddwwwwwaaass'
t2 = 'sssssdsssddwddddwwwwaaas'
t3 = 'sssssdsssddsdddwwdwwaaaw'
t4 = 'wdwwawwwwwwdwddddddsssssssaaawww'
t1 = tobin(t1)
print t1
t2 = tobin(t2)
print t2
t3 = tobin(t3)
print t3
tobin(t4)
t4 = '3080033ffd555a80'.decode('hex')
from Crypto.Cipher import DES3
key = t1.strip() + t2.strip() + t3.strip()
print key
c = DES3.new(key,DES3.MODE_ECB)
cipher = (c.encrypt(t4)).encode('base64')
sn = key + cipher
print sn
```
