# 前言
第二次出大比赛的题目，今年因为个人原因出了一道比较无聊的题目，代码本来是想参加密码学技术竞赛的，结果因为一些原因咕咕咕了,然后稍微改了一下就拿过来用了
# 题解
avx2网上找有很多资料，放点比较重要的
```
vmovdqu ymm2/m256, ymm1:Move unaligned packed integer values from ymm1 to ymm2/m256.    --->_mm256_setr_epi32 (or similar)
VPCMPEQB ymm1, ymm2, ymm3 /m256:Compare packed bytes in ymm3/m256 and ymm2 for equality.  
vpgatherdd       --->    _mm256_i32gather_epi32
```
```
//vpgatherdd example
MASK[MAXVL-1:256] ← 0;
FOR j←0 to 7
    i←j * 32;
    IF MASK[31+i] THEN
        MASK[i +31:i]←FFFFFFFFH; // extend from most significant bit
    ELSE
        MASK[i +31:i]←0;
    FI;
ENDFOR
FOR j←0 to 7
    i←j * 32;
    DATA_ADDR←BASE_ADDR + (SignExtend(VINDEX1[i+31:i])*SCALE + DISP;
    IF MASK[31+i] THEN
        DEST[i +31:i]←FETCH_32BITS(DATA_ADDR); // a fault exits the instruction
    FI;
    MASK[i +31:i]←0;
ENDFOR
DEST[MAXVL-1:256] ← 0;
```
`VPSHUFB ymm1, ymm2, ymm3/m256:Shuffle bytes in ymm2 according to contents of ymm3/m256.  --->_mm256_shuffle_epi8`


flag分布:
```
    256register1 :   flag[0:3]    flag[16:19]   flag[32:35]    flag[48:51]    repeat   repeat   repeat    repeat
    256register2 :   flag[4:7]    flag[20:23]   flag[36:39]    flag[52:55]    repeat   repeat   repeat    repeat
    256register3 :   flag[8:11]   flag[24:27]   flag[40:43]    flag[56:59]    repeat   repeat   repeat    repeat
    256register4 :   flag[12:15]  flag[28:31]   flag[44:47]    flag[60:63]    repeat   repeat   repeat    repeat
```
[具体的论文链接](http://html.rhhz.net/ZGKXYDXXB/20180205.htm)
由avx2指令想到分组密码，因为这种并行指令近年来一直应用于密码加速中。逆向工程中运用的密码一般是公开的加密算法，所以会有密钥和一些加密常量，利用这点去搜索常量识别对应的加密算法,在程序的主函数中，并没有发现密钥，(因为我指定了这部分在main函数前执行)，可以先搜寻指令，静态看一下程序逻辑，或者在x32dbg中可以看到ymm寄存器，动态调试分析指令做了些什么事，可以发现vpgatherdd是本次挑战中加载数据的主要方式，那么轮密钥也一定是用这个指令加载的，`vpgatherdd ymm2, dword ptr [edx+ymm0*4], ymm1`应当引起我们重视，跳转到edx所在地址可以发现32个int常量，这就是我们所要找的轮密钥，在这个地址下内存断点，可以跟踪到产生轮密钥的函数`sub_412AF0`，密钥是`unk_54E230`,这里用了AES的置换表来异或解得sms4的置换表(具有一点点迷惑性，没啥卵用)，搜索常量发现这是sms4算法，然后注意一下计算后密文的分布情况，提取出密文进行解密即可
PS:AAA大哥直接看出这是sm4算法，如果是这样，只需要提取出轮密钥进行解密即可
cipher分布：
```
256register4 : cipher[0:3]      cipher[16:19]       cipher[32:35]       cipher[48:51]   repeat   repeat   repeat    repeat
256register3 : cipher[4:7]      cipher[20:23]       cipher[36:39]       cipher[52:55]   repeat   repeat   repeat    repeat
256register2 : cipher[8:11]     cipher[24:27]       cipher[40:43]       cipher[56:59]   repeat   repeat   repeat    repeat
256register1 : cipher[12:15]    cipher[28:31]       cipher[44:47]       cipher[60:63]   repeat   repeat   repeat    repeat
```
程序中对比的密文常量分布是这样的：`cipher[0:3],cipher[16:19],cipher[32:35],cipher[48:51],cipher[4:7],cipher[20:23],cipher[36:39],cipher[52:55],cipher[8:11],cipher[24:27],cipher[40:43],cipher[56:59],cipher[12:15],cipher[28:31],cipher[44:47],cipher[60:63]`
整理顺序解密即可

[源码及keygen地址](https://github.com/fa1conn/D3CTF-2019-Rev-SIMD-Source-Code)