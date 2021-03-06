---
layout:     post
title:      "*ctf yy waiteup"
subtitle:   "*ctf yy waiteup"
date:       2019-5-8 12:00:00
author:     "fa1con"
header-img: ""
header-mask: 0.3
catalog:    true
tags:
    - re
    - bin
    - CTF
---

根据语法分析器，可以搜关于那个分析器的源码，虽然没卵用
调试可以得到如下类似虚拟机的对应关系表
```
0F -- a
10 -- b
11 -- c
12 -- d
13 -- e
14 -- f
15 -- g
16 -- h
17 -- i
18 -- j
19 -- k
1A -- l
1B -- m
1C -- n
1D -- o
1E -- p
1F -- q
20 -- r
21 -- s
22 -- t
23 -- u
24 -- v
25 -- w
26 --x
27 --y
28 --z
29 -- 0
2A -- 1
2B -- 2
2C -- 3
2D -- 4
2E -- 5
2F -- 6
30 -- 7
31 -- 8
32 -- 9
```

然后逆向可以知道需要多次加密，每次16字节，总共9组，每组的输入覆盖buffer的前某几位（根据每组的输入而定），每组要用某个符号进行分割，于是调试了一些常见的分割符号，终于发现是`_`
然后逆一下算法，每组16字节，与上一组的result异或，然后进入一个循环加密代码块，最后相互调换字节位存储起来，再与一个固定的`xmmword_56470E732160`异或，然后存起来，下一次加密时会用到这轮的，使用z3循环逆算法，z3有点小坑，`>>`不能正常实现，卡了半天，应该采用`&128/128`
由于是先逆的算法，所以到后面写循环有点麻烦，所以写了9个脚本，贴上其中一个，每个脚本仅需要更改12行中的`cmpcode[i+16*n]`和倒数第八行的`^cmp_code[i+16*(n-1)]`(这个脚本是第一个，为0，所以没加),n是第n个脚本(从0开始)
```python
'''
a = 0xAE4614F82A40CF5031D3FE048C061212
b = 0xD014F9A8C9EE2589E13F0CC8B6630CA6
print hex(a^b)
'''
#key1 = [0x7e,0x52,0xed,0x50,0xe3,0xae,0xea,0xd9,0xd0,0xec,0xf2,0xcc,0x3a,0x65,0x1e,0xb4] #Int
#buf 0,5,10,15,4,9,14,3,8,13,2,7,12,1,6,11
from z3 import *
cmp_code = [ 0xae, 0x46, 0x14, 0xf8, 0x2a, 0x40, 0xcf, 0x50, 0x31, 0xd3, 0xfe, 0x4, 0x8c, 0x6, 0x12, 0x12, 0x23, 0xfa, 0xc7, 0x26, 0xe8, 0x61, 0xd9, 0xc3, 0xa9, 0x3c, 0x45, 0x70, 0x1a, 0xc7, 0xf0, 0x3d, 0xdf, 0xbe, 0xbc, 0x16, 0xab, 0x6e, 0x37, 0xac, 0x14, 0x8b, 0x9c, 0x94, 0xf7, 0x5d, 0x62, 0x78, 0xfc, 0x16, 0x98, 0x1d, 0xb2, 0x31, 0xd3, 0x5a, 0xdc, 0x3a, 0x60, 0x86, 0x9a, 0xca, 0x7b, 0xa3, 0xb5, 0xd5, 0xf1, 0xb2, 0xd9, 0xff, 0xd2, 0x9, 0xd4, 0x77, 0xd7, 0x3d, 0xc0, 0x56, 0x19, 0x2, 0xb6, 0x9b, 0x42, 0x6c, 0xe8, 0xa2, 0x77, 0xe3, 0x99, 0xac, 0x32, 0x40, 0x91, 0xa9, 0x2a, 0x86, 0xf3, 0xfa, 0x47, 0x3c, 0xc3, 0x5c, 0x41, 0x9b, 0xe8, 0x5, 0x7, 0xd0, 0xd4, 0x30, 0x5a, 0x9e, 0x8d, 0x52, 0x9b, 0xa3, 0xfb, 0xad, 0xb6, 0x44, 0x3f, 0x72, 0x83, 0x9c, 0x22, 0x77, 0xfe, 0x48, 0xfe, 0x86, 0x84, 0x12, 0x0, 0x4e, 0xed, 0xff, 0xac, 0x44, 0x19, 0x23, 0x84, 0x1f, 0x12, 0xca, 0x2b, 0x7e, 0x15, 0x16, 0x28, 0xae, 0xd2, 0xa6, 0xab, 0xf7, 0x15, 0x88, 0x9, 0xcf, 0x4f, 0x3c ]
xmmword_55BFEA477160 = [ 0xd0, 0x14, 0xf9, 0xa8, 0xc9, 0xee, 0x25, 0x89, 0xe1, 0x3f, 0xc, 0xc8, 0xb6, 0x63, 0xc, 0xa6 ]
key1 = [0 for i in range(16)]
for i in range(len(xmmword_55BFEA477160)):
    key1[i] = cmp_code[i]^xmmword_55BFEA477160[i]
#print key1
turn = [0,5,10,15,4,9,14,3,8,13,2,7,12,1,6,11]
#key1 = [0xD7,0xDF,0xAD,0x6A,0x15,0x1B,0xFC,0x5B,0x78,0x9A,0xB6,0xF1,0x3A,0x12,0xE4,0x44]  #test:_mm_load_si128(&v60)
buf = [0 for i in range(len(key1))]
table = [99, 124, 119, 123, 242, 107, 111, 197, 48, 1, 103, 43, 254, 215, 171, 118, 202, 130, 201, 125, 250, 89, 71, 240, 173, 212, 162, 175, 156, 164, 114, 192, 183, 253, 147, 38, 54, 63, 247, 204, 52, 165, 229, 241, 113, 216, 49, 21, 4, 199, 35, 195, 24, 150, 5, 154, 7, 18, 128, 226, 235, 39, 178, 117, 9, 131, 44, 26, 27, 110, 90, 160, 82, 59, 214, 179, 41, 227, 47, 132, 83, 209, 0, 237, 32, 252, 177, 91, 106, 203, 190, 57, 74, 76, 88, 207, 208, 239, 170, 251, 67, 77, 51, 133, 69, 249, 2, 127, 80, 60, 159, 168, 81, 163, 64, 143, 146, 157, 56, 245, 188, 182, 218, 33, 16, 255, 243, 210, 205, 12, 19, 236, 95, 151, 68, 23, 196, 167, 126, 61, 100, 93, 25, 115, 96, 129, 79, 220, 34, 42, 144, 136, 70, 238, 184, 20, 222, 94, 11, 219, 224, 50, 58, 10, 73, 6, 36, 92, 194, 211, 172, 98, 145, 149, 228, 121, 231, 200, 55, 109, 141, 213, 78, 169, 108, 86, 244, 234, 101, 122, 174, 8, 186, 120, 37, 46, 28, 166, 180, 198, 232, 221, 116, 31, 75, 189, 139, 138, 112, 62, 181, 102, 72, 3, 246, 14, 97, 53, 87, 185, 134, 193, 29, 158, 225, 248, 152, 17, 105, 217, 142, 148, 155, 30, 135, 233, 206, 85, 40, 223, 140, 161, 137, 13, 191, 230, 66, 104, 65, 153, 45, 15, 176, 84, 187, 22, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0]
tmp1 = []
for i in range(len(key1)):
    for j in range(len(table)):
        if key1[i] == table[j]:
            tmp1.append(j)
print tmp1
for i in range(len(turn)):
    buf[turn[i]] = tmp1[i]
print buf
#xmmword_55BFEA477160 = 0x558A0D64D160
unk_55BFEA4770D0 = 0x558A0D64D0D0
unk_56339FBE40D0 = [ 0xa0, 0xfa, 0xfe, 0x17, 0x88, 0x54, 0x2c, 0xb1, 0x23, 0xa3, 0x39, 0x39, 0x2a, 0x6c, 0x76, 0x5, 0xf2, 0xc2, 0x95, 0xf2, 0x7a, 0x96, 0xb9, 0x43, 0x59, 0x35, 0x80, 0x7a, 0x73, 0x59, 0xf6, 0x7f, 0x3d, 0x80, 0x47, 0x7d, 0x47, 0x16, 0xfe, 0x3e, 0x1e, 0x23, 0x7e, 0x44, 0x6d, 0x7a, 0x88, 0x3b, 0xef, 0x44, 0xa5, 0x41, 0xa8, 0x52, 0x5b, 0x7f, 0xb6, 0x71, 0x25, 0x3b, 0xdb, 0xb, 0xad, 0x0, 0xd4, 0xd1, 0xc6, 0xf8, 0x7c, 0x83, 0x9d, 0x87, 0xca, 0xf2, 0xb8, 0xbc, 0x11, 0xf9, 0x15, 0xbc, 0x6d, 0x88, 0xa3, 0x7a, 0x11, 0xb, 0x3e, 0xfd, 0xdb, 0xf9, 0x86, 0x41, 0xca, 0x0, 0x93, 0xfd, 0x4e, 0x54, 0xf7, 0xe, 0x5f, 0x5f, 0xc9, 0xf3, 0x84, 0xa6, 0x4f, 0xb2, 0x4e, 0xa6, 0xdc, 0x4f, 0xea, 0xd2, 0x73, 0x21, 0xb5, 0x8d, 0xba, 0xd2, 0x31, 0x2b, 0xf5, 0x60, 0x7f, 0x8d, 0x29, 0x2f, 0xac, 0x77, 0x66, 0xf3, 0x19, 0xfa, 0xdc, 0x21, 0x28, 0xd1, 0x29, 0x41, 0x57, 0x5c, 0x0, 0x6e ]
for j in range(len(table)):
    if table[j] == 147:
        print j
for i in range(0x558A0D64D160,0x558A0D64D0D0,-0x10):
    buf0 = BitVec('buf0',8)
    buf1 = BitVec('buf1',8)
    buf2 = BitVec('buf2',8)
    buf3 = BitVec('buf3',8)
    buf4 = BitVec('buf4',8)
    buf5 = BitVec('buf5',8)
    buf6 = BitVec('buf6',8)
    buf7 = BitVec('buf7',8)
    buf8 = BitVec('buf8',8)
    buf9 = BitVec('buf9',8)
    buf10 = BitVec('buf10',8)
    buf11 = BitVec('buf11',8)
    buf12 = BitVec('buf12',8)
    buf13 = BitVec('buf13',8)
    buf14 = BitVec('buf14',8)
    buf15 = BitVec('buf15',8)
    buf128 = BitVec('buf128', 8)
    for j in range(16):
        buf[j] ^= unk_56339FBE40D0[i-0x558A0D64D0D0-16+j]
    print buf
    print buf[0]
    s = Solver()
#    s.add(390^194^27 == 2 * (buf5^buf0)^(buf15^buf10^buf5)^(27*((buf5^buf0)/128)))
#    s.add(95 == (2 * (buf5^buf0)^(buf15^buf10^buf5)^(27*((buf5^buf0)>>7)))%256)
    s.add(buf[0] == (2 * (buf5^buf0)^(buf15^buf10^buf5)^(27*(((buf5^buf0)&128)/128)))%256)
    s.add(buf[1] == (2 * (buf10^buf5)^(buf15^buf10^buf0)^(27*(((buf5^buf10)&128)/128)))%256)
    s.add(buf[2] == (2 * (buf10^buf15)^(buf15^buf5^buf0)^(27*(((buf15^buf10)&128)/128)))%256)
    s.add(buf[3] == (2 * (buf0^buf15)^(buf10^buf5^buf0)^(27*(((buf15^buf0)&128)/128)))%256)
    s.add(buf[4] == (2 * (buf9^buf4)^(buf14^buf9^buf3)^(27*(((buf9^buf4)&128)/128)))%256)
    s.add(buf[5] == (2 * (buf9^buf14)^(buf14^buf4^buf3)^(27*(((buf9^buf14)&128)/128)))%256)
    s.add(buf[6] == (2 * (buf3^buf14)^(buf3^buf4^buf9)^(27*(((buf3^buf14)&128)/128)))%256)
    s.add(buf[7] == (2 * (buf3^buf4)^(buf14^buf4^buf9)^(27*(((buf3^buf4)&128)/128)))%256)
    s.add(buf[8] == (2 * (buf8^buf13)^(buf13^buf7^buf2)^(27*(((buf13^buf8)&128)/128)))%256)
    s.add(buf[9] == (2 * (buf13^buf2)^(buf2^buf7^buf8)^(27*(((buf13^buf2)&128)/128)))%256)
    s.add(buf[10] == (2 * (buf2^buf7)^(buf13^buf7^buf8)^(27*(((buf2^buf7)&128)/128)))%256)
    s.add(buf[11] == (2 * (buf7^buf8)^(buf13^buf8^buf2)^(27*(((buf7^buf8)&128)/128)))%256)
    s.add(buf[12] == (2 * (buf1^buf12)^(buf11^buf6^buf1)^(27*(((buf1^buf12)&128)/128)))%256)
    s.add(buf[13] == (2 * (buf1^buf6)^(buf6^buf12^buf11)^(27*(((buf1^buf6)&128)/128)))%256)
    s.add(buf[14] == (2 * (buf6^buf11)^(buf12^buf11^buf1)^(27*(((buf6^buf11)&128)/128)))%256)
    s.add(buf[15] == (2 * (buf11^buf12)^(buf6^buf12^buf1)^(27*(((buf11^buf12)&128)/128)))%256)
    print s.check()
    m = s.model()
    tmp = [0 for i in range(16)]
    for i in s.model():
        if str(i) == "buf0":
            tmp[0] = s.model()[i]
        elif str(i) == "buf1":
            tmp[1] = s.model()[i]
        elif str(i) == "buf2":
            tmp[2] = s.model()[i]
        elif str(i) == "buf3":
            tmp[3] = s.model()[i]
        elif str(i) == "buf4":
            tmp[4] = s.model()[i]
        elif str(i) == "buf5":
            tmp[5] = s.model()[i]
        elif str(i) == "buf6":
            tmp[6] = s.model()[i]
        elif str(i) == "buf7":
            tmp[7] = s.model()[i]
        elif str(i) == "buf8":
            tmp[8] = s.model()[i]
        elif str(i) == "buf9":
            tmp[9] = s.model()[i]
        elif str(i) == "buf10":
            tmp[10] = s.model()[i]
        elif str(i) == "buf11":
            tmp[11] = s.model()[i]
        elif str(i) == "buf12":
            tmp[12] = s.model()[i]
        elif str(i) == "buf13":
            tmp[13] = s.model()[i]
        elif str(i) == "buf14":
            tmp[14] = s.model()[i]
        elif str(i) == "buf15":
            tmp[15] = s.model()[i]
    for k in range(16):
        for j in range(len(table)):
            if table[j] == tmp[k]:
                buf[k] = j
                break
#    print m
#    print tmp
    print buf

encode_key1 = [ 0x2b, 0x7e, 0x15, 0x16, 0x28, 0xae, 0xd2, 0xa6, 0xab, 0xf7, 0x15, 0x88, 0x9, 0xcf, 0x4f, 0x3c ]
for i in range(16):
    buf[i] ^= encode_key1[i]#^cmp_code[i+16*(n-1)]
    print hex(buf[i])[2:],
box = [ 0x82, 0x5, 0x86, 0x8a, 0xb, 0x11, 0x96, 0x1d, 0x27, 0xa9, 0x2b, 0xb1, 0xf3, 0x5e, 0x37, 0x38, 0xc2, 0x47, 0x4e, 0x4f, 0xd6, 0x58, 0xde, 0xe2, 0xe5, 0xe6, 0x67, 0x6b, 0xec, 0xed, 0x6f, 0xf2, 0x73, 0xf5, 0x77, 0x7f]
print buf
for i in range(16):
    for j in range(len(box)):
        if buf[i] == box[j]:
            print str(i)+" "+hex(j+0xF)
```
根据得到的连续的前某几个字节，对照上面的映射码，一个个推导出来，可得flag：`*CTF{yy_funct10n_1s_h4rd_and_n0_n33d_to_r3v3rs3}`
果然如flag所说，yy不用看，时间全被坑在z3和逆算法里了，后来官方说那是AES的部分算法