---
layout:     post
title:      "现代密码学第一次实验"
subtitle:   "The first experiment of modern cryptography"
date:       2019-11-14 12:00:00
author:     "fa1con"
header-img: ""
header-mask: 0.3
catalog:    true
tags:
    - crypto
    - homework
---
# 现代密码学第一次实验
## 前言
老师让写在博客，代码有些是copy的（实验实在太多，什么？居然不是全抄的？别怀疑，有些题还是我自己写的），如有侵权，请联系本文作者删除，时间匆忙，难免有错（你们懂的XD，也留给后来的同学抄一抄
## 第一题：Many Time Pad
该题目提供了十条使用同一密钥(有意义的英文语句)加密后的密文结果,要求根据所给的密文解密最终的target ciphertext（目标密文）.并且根据提示,本题目需要用到空格与字母之间的特殊异或关系
空格与大写字母异或会变成小写，与小写字母异或会变成大写，并且任意两个字母异或的结果不在字母范围内。由于对于使用同一密钥进行异或的两对密文，进行异或的结果等于对应明文进行异或的结果即
c1^c2^c3^c2 = c1^c2
所以对两个明文两两异或，如果对应位是可见字符，那么猜测对应的位是空格和相反的字母，最后拼凑起来得到明文，并还原密钥
```python
import binascii

def strxor(a, b):     
    if len(a) > len(b):
       return "".join([chr(ord(x) ^ ord(y)) for (x, y) in zip(a[:len(b)], b)])
    else:
       return "".join([chr(ord(x) ^ ord(y)) for (x, y) in zip(a, b[:len(a)])])
def decrypt(a,b):
    c = strxor(a,b)
    return c

txt = "When using a stream cipher, never use the key more than once"
dic = "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ "
fopen = open("ciphertext.txt","r")
cipher = []
for i in fopen:
    cipher.append(binascii.a2b_hex(i.strip()))
result = [[] for i in range(len(cipher[10]))]
for i in range(len(cipher)-1):
    b = decrypt(cipher[10],cipher[i])
    print i,
    print ":",
    for i in range(len(b)):
        if ord(b[i]) in range(40,123):
            if b[i] not in result[i]:
                result[i].append(b[i])
            print b[i],
        else:
            print "?",
    print "\n"
for i in range(len(result)):
    print result[i]

#guess the target message:The secret message is: When using a stream cipher, never use the key more than once

txt = "The secret message is: When using a stream cipher, never use the key more than once"
fopen = open("ciphertext.txt","r")
cipher = []
for i in fopen:
    cipher.append(binascii.a2b_hex(i.strip()))
#result = [[] for i in range(len(cipher[10]))]
key = decrypt(txt,cipher[10])
print key.encode('hex')
for i in range(len(cipher)-1):
    print decrypt(key,cipher[i])
```

## 第二题：Vigenere-Like Cipher
该题目的本质依然为密钥重复使用导致的不安全问题,只是多用了维吉尼亚密码.题目随机选取1-14范围内的数字为密钥长度,并随机设置每一位加密密钥去加密密文,希望读者通过所给的密文及特定的重用加密方式还原明文及密钥

由于不知道key的长度，所以先爆破key长度，假设key的长度是某个值，取每组中的第一个字节去异或0-256范围的值，如果在某个值内异或出的值都是可见字符，那么这个值就是第1位密钥，同时也得到了密钥的长度，然后再循环爆破出密钥剩下的位就好了
```python
import string
ctext = [0xF9,0x6D,0xE8,0xC2,0x27,0xA2,0x59,0xC8,0x7E,0xE1,0xDA,0x2A,0xED,0x57,0xC9,0x3F,0xE5,0xDA,0x36,0xED,0x4E,0xC8,0x7E,0xF2,0xC6,0x3A,0xAE,0x5B,0x9A,0x7E,0xFF,0xD6,0x73,0xBE,0x4A,0xCF,0x7B,0xE8,0x92,0x3C,0xAB,0x1E,0xCE,0x7A,0xF2,0xDA,0x3D,0xA4,0x4F,0xCF,0x7A,0xE2,0x92,0x35,0xA2,0x4C,0x96,0x3F,0xF0,0xDF,0x3C,0xA3,0x59,0x9A,0x70,0xE5,0xDA,0x36,0xBF,0x1E,0xCE,0x77,0xF8,0xDC,0x34,0xBE,0x12,0x9A,0x6C,0xF4,0xD1,0x26,0xBF,0x5B,0x9A,0x7C,0xFE,0xDF,0x3E,0xB8,0x50,0xD3,0x7C,0xF0,0xC6,0x3A,0xA2,0x50,0x9A,0x76,0xFF,0x92,0x27,0xA5,0x5B,0x9A,0x6F,0xE3,0xD7,0x20,0xA8,0x50,0xD9,0x7A,0xB1,0xDD,0x35,0xED,0x5F,0xCE,0x6B,0xF0,0xD1,0x38,0xA8,0x4C,0xC9,0x31,0xB1,0xF1,0x21,0xB4,0x4E,0xCE,0x70,0xF6,0xC0,0x32,0xBD,0x56,0xC3,0x3F,0xF9,0xD3,0x20,0xED,0x5C,0xDF,0x7A,0xFF,0x92,0x26,0xBE,0x5B,0xDE,0x3F,0xF7,0xDD,0x21,0xED,0x56,0xCF,0x71,0xF5,0xC0,0x36,0xA9,0x4D,0x96,0x3F,0xF8,0xD4,0x73,0xA3,0x51,0xCE,0x3F,0xE5,0xDA,0x3C,0xB8,0x4D,0xDB,0x71,0xF5,0xC1,0x7F,0xED,0x51,0xDC,0x3F,0xE8,0xD7,0x32,0xBF,0x4D,0x96,0x3F,0xF3,0xC7,0x27,0xED,0x4A,0xC8,0x7E,0xF5,0xDB,0x27,0xA4,0x51,0xD4,0x7E,0xFD,0x92,0x30,0xBF,0x47,0xCA,0x6B,0xFE,0xC1,0x2A,0xBE,0x4A,0xDF,0x72,0xE2,0x92,0x24,0xA8,0x4C,0xDF,0x3F,0xF5,0xD7,0x20,0xA4,0x59,0xD4,0x7A,0xF5,0x92,0x32,0xA3,0x5A,0x9A,0x7A,0xE7,0xD3,0x3F,0xB8,0x5F,0xCE,0x7A,0xF5,0x92,0x3A,0xA3,0x1E,0xDB,0x3F,0xF7,0xD3,0x3A,0xBF,0x52,0xC3,0x3F,0xF0,0xD6,0x73,0xA5,0x51,0xD9,0x3F,0xFC,0xD3,0x3D,0xA3,0x5B,0xC8,0x31,0xB1,0xF4,0x3C,0xBF,0x1E,0xDF,0x67,0xF0,0xDF,0x23,0xA1,0x5B,0x96,0x3F,0xE5,0xDA,0x36,0xED,0x68,0xD3,0x78,0xF4,0xDC,0x36,0xBF,0x5B,0x9A,0x7A,0xFF,0xD1,0x21,0xB4,0x4E,0xCE,0x76,0xFE,0xDC,0x73,0xBE,0x5D,0xD2,0x7A,0xFC,0xD7,0x73,0xBA,0x5F,0xC9,0x3F,0xE5,0xDA,0x3C,0xB8,0x59,0xD2,0x6B,0xB1,0xC6,0x3C,0xED,0x5C,0xDF,0x3F,0xE2,0xD7,0x30,0xB8,0x4C,0xDF,0x3F,0xF7,0xDD,0x21,0xED,0x5A,0xDF,0x7C,0xF0,0xD6,0x36,0xBE,0x1E,0xDB,0x79,0xE5,0xD7,0x21,0xED,0x57,0xCE,0x3F,0xE6,0xD3,0x20,0xED,0x57,0xD4,0x69,0xF4,0xDC,0x27,0xA8,0x5A,0x96,0x3F,0xF3,0xC7,0x27,0xED,0x49,0xDF,0x3F,0xFF,0xDD,0x24,0xED,0x55,0xD4,0x70,0xE6,0x9E,0x73,0xAC,0x50,0xDE,0x3F,0xE5,0xDA,0x3A,0xBE,0x1E,0xDF,0x67,0xF4,0xC0,0x30,0xA4,0x4D,0xDF,0x3F,0xF5,0xD7,0x3E,0xA2,0x50,0xC9,0x6B,0xE3,0xD3,0x27,0xA8,0x4D,0x96,0x3F,0xE5,0xDA,0x32,0xB9,0x1E,0xD3,0x6B,0xB1,0xD1,0x32,0xA3,0x1E,0xD8,0x7A,0xB1,0xD0,0x21,0xA2,0x55,0xDF,0x71,0xB1,0xC4,0x36,0xBF,0x47,0x9A,0x7A,0xF0,0xC1,0x3A,0xA1,0x47,0x94];
dic = string.ascii_letters + " ,.!?"

def find_key_size():
    for keysize in range(1,14):
        data = ctext[0::keysize]
        for key in range(256):
            
            flag = 1
            for elem in data:
                tmp = chr(elem ^ key)
                if tmp not in dic:
                    flag = 0
                    break
            if flag:
                print("KEYSIZE: {}\nkey: {}\n".format(keysize, key))

def find_key():
    for k in range(0,7):
        text1 = ctext[k::7]
        for i in range(0,256):
            ret = 0
            ptext = ""
            for j in text1:
                tmp = chr(i^j)
                if tmp not in dic:
                    ret = 1
                    break
                ptext += tmp
            if ret:
                continue
            print("Num: {} KEY: {}\n".format(k, i))

if __name__ == "__main__":
    find_key_size()
    find_key()
    key = [186, 31, 145, 178, 83, 205, 62]
    ptext = ""
    for i in range(len(ctext)):
        ptext += chr(ctext[i] ^ (key[i % 7]))
    print(ptext)
```
## 第三题：Cryptopals-Set1
题目：
1)实现一个函数,将十六进制字符串转化为base64编码的字符串.
2)实现一个函数,输入为两个等长的字符串,输出为这两个字符串的异或结果.
3)给定一个十六进制字符串,该字符串是明文与一个单字符key进行异或加密之后得到的结果.现要求在已知cipher_hex_string的情况下还原plaintext.
4)给出challenge4.txt文件,文件中的一个字符串是由单字符异或方法加密的十六进制编码串,找到并解密这个字符串.
5)实现一个函数,给定待加密的字符串和密钥,该函数通过使用重复密钥异或的方法加密一段英文文本,返回加密后的结果.
6)该挑战中提供challenge6.txt文件,文件使用重复密钥异或方法加密后,再经过bas64编码后得到的文本,我们需要找到密钥,对其进行解密.

解法：
1.	首先将十六进制字符串使用decode方法解码为unicode编码,然后再将字符串编码为base64.
2.	判断哪个字符串比较长，然后循环的时候取短的那个就好了
3.	已知密文是明文与同一字符进行异或生成，遍历所有可能的字典，与密文进行异或，然后进行统计，如果每次都是可见字符，那么这个字符很可能就是明文
```python
import base64
import string
txt1 = "49276d206b696c6c696e6720796f757220627261696e206c696b65206120706f69736f6e6f7573206d757368726f6f6d"
print "Challenge1:"+ base64.b64encode(txt1.decode('hex'))

def xor(t1,t2):
    d1 = []
    d2 = []
    txt2 = ""
    for i in range(len(t1)):
        d1.append(ord(t1[i]))
        d2.append(ord(t2[i]))
    for i in range(len(d1)):
        txt2 += chr(d1[i]^d2[i])
    return txt2.encode('hex')
t1 = "1c0111001f010100061a024b53535009181c".decode('hex')
t2 = "686974207468652062756c6c277320657965".decode('hex')
print "Challenge2:" + xor(t1,t2)

t3 = "1b37373331363f78151b7f2b783431333d78397828372d363c78373e783a393b3736"
t4 = t3.decode('hex')
dic = string.ascii_letters+'0123456789'
print dic
def challenge3(k,t4): 
    for i in dic:
        d1 = list(t4)
        txt3 = ""
        a = chr(0)
        cnt = 0
        for j in range(len(d1)):
            a = chr(ord(d1[j]) ^ ord(i))
            txt3 += a
            if a >= ' ' and a <= 'z' and a != ']' and a != '%' and a != '&' and a != '#' and a != '[' :
                cnt +=1
        #print cnt
        if cnt == len(d1):
            print "Challenge" + str(k) + ':' + i + ':' + txt3
challenge3(3,t4)
```
4.跟上一题差不多，异或，然后统计分析频率
```python
import string
def xor_data(key, data):
    "xor key with data, repeating key as necessary"
    if len(key) == 1:
        #shortcut
        key = ord(key)
        return ''.join(chr(ord(x) ^ key) for x in data)

    stream = itertools.cycle(key)
    return ''.join(chr(ord(x) ^ ord(y)) for x,y in itertools.izip(data, stream))
ok = set(string.letters + ' ')
def score_ratio(s):
    "ratio of letters+space to total length"
    count = sum(1 for x in s if x in ok)
    #print count
    return count / float(len(s))
def score_decodings(keys, fscore, data):
    "return list of decodings, scored by fscore"
    scores = []
    for key in keys:
        plain = xor_data(key, data)
        score = fscore(plain)
        scores.append((score, key, plain))
    return sorted(scores, reverse=True)
def cc4():
    keys = [chr(x) for x in xrange(256)]
    best_scores = []
    with open('data.txt') as f:
        for line in f:
            ciphertext = line.strip().decode('hex')
            scores = score_decodings(keys, score_ratio, ciphertext)
            best_scores.append(scores[0])
    best = sorted(best_scores, reverse=True)[0]
    print "score: %.2f key: '%s' plain: %s" % best
cc4()
```
5.将待加密的字符串按照keysize的大小进行分块，将每块分别与密钥key进行逐字符异或加密，最后将每块的加密结果合并到一起
```python
txt1 = "Burning 'em, if you ain't quick and nimble\nI go crazy when I hear a cymbal" 
key = "ICE"

cnt = 0
def encrypt(txt):
    global cnt
    a = list(txt)
    b = ""
    print cnt
    for i in range(len(txt)):
        a[i] = chr(ord(a[i]) ^ ord(key[cnt % 3]))
        b += a[i]
        cnt += 1
    print b.encode('hex')
encrypt(txt1)
```
6.对密文攻击采用了流密码加密密钥重用对应的攻击方法。首先从2-40猜测密钥长度，对于每个KEYSIZE，求得解密块两两之间的汉明距离，对KEYSIZE求平均值，选取2-3个最小汉明距离对于的KEYSIZE作为最终的密钥长度。获得密钥长度后，按照密钥长度对密文进行分块，即将使用同一单字符密钥加密后的密文分到一组，使用破解单字符异或的方法来逐字符破解密钥。对不同KEYSIZE解密后的明文进行评估，选取得分最高的一组作为最终的明文
```python
#!/usr/bin/env python

# using PyCrypto
from Crypto.Cipher import AES
import random
import time


def hexToBase64(s):
    binary_string = hexToBinary(s)
    base64_str = binaryToBase64(binary_string)
    return base64_str

def hexToBinary(s):
    binary_string = ''
    ascii_list = hexToAscii(s)
    for i in ascii_list:
        binary_string += intToBinary(i)
    return binary_string

def hexToAscii(s):
    ascii_list = []    
    for i in splitString(s,2):
        ascii_list.append(str(int(i,16)))
    return ascii_list

def hexToStr(s):
    string = ''
    for i in splitString(s,2):
        string += chr(int(i,16))
    return string


def intToBinary(n):
    return '{0:08b}'.format(int(n))

def splitString(s, n):
    return [s[i:i+n] for i in range(0, len(s), n)]

def base64ToBinary(s):
    baseList = 'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/'
    binary_str = ''
    padding_bits = 0
    for c in s:
        if c == '=':
            padding_bits+= 2
            continue
        #binary_str += intTo6BitBinary(baseList.index(c))
        binary_str += '{0:06b}'.format(baseList.index(c))
    if(padding_bits):
        binary_str = binary_str[:-padding_bits]
    return binary_str

def binaryToStr(s):
    string = ''
    for c in splitString(s, 8):
        string += chr(binaryToAscii(c))
    return string   

def fixed_xor(a,b):
    return int(a != b)

def fixed_xor_str(a, b):
    c = ''

    for i in range(0,len(a)):
        c += str(fixed_xor(a[i], b[i]))

    return c

def binaryToHex(s):
    ascii_numbers = binaryStrToAscii(s)
    hex_string = ''
    for i in ascii_numbers:
        hex_string += "{0:0>2x}".format(int(i))

    return hex_string 

def binaryToAscii(s):
    return int(s,2)




def getRank(s):
    common = ['e','t','a','o','i','n']
    less_common = ['s','h','r','d','l','u']
    common_words = ['the', 'be', 'to', 'of', 'and', 'a', 'in', 'that', 'have', 'is', "I'm", 'on', 'at']
    rank = 0
    string = s.lower()
    for i in range (0, len(string)):
        if s[i] in common:
            rank += 2
        if s[i] in less_common:
            rank += 1

    for word in s.split(' '):
        if(word in common_words):
            rank += 1
    return rank



def findSingleCharacterKey(s):
    a_to_z = "1234567890abcdefghijklmnopqrstuvwxyzABCDEDFGHJKLMNOPQRSTUVWXYZ:'., "
    ranked = {}
    highest_ranked = 0
    for key in a_to_z:
        decoded = decodeSingleCharacterKey(s, intToBinary(ord(key)))
        # check the rank of each potential decoded message
        rank = getRank(decoded)
        if(rank > 0 and rank > highest_ranked):
            ranked = key
            highest_ranked = rank
    if ranked:
        return ranked

def strToBinary(s):
    binary_str = ''
    for c in s:
        binary_str += charToBinary(c)
    return binary_str

def charToBinary(c):
    return intToBinary(ord(c))

def decodeSingleCharacterKey(s, key):
    string = ''
    # XOR each 8-bit segment
    for byte in splitString(s, 8):
        string += fixed_xor_str(byte, key)
    # convert decoded message back to hexadecimal
    decoded = binaryToStr(string)
    return decoded

def encryptRepeatingKeyXOR(s, key):
    binary_s = strToBinary(s)
    binary_keys = splitString(strToBinary(key), 8)

    encoded_string = ''
    for i, binary in enumerate(splitString(binary_s, 8)):
        encoded_string += fixed_xor_str(binary, binary_keys[i%len(key)])

    return encoded_string


def hamming_distance(s1, s2):
    tuplets = zip(s1, s2)
    sum_of = 0
    for ch1, ch2 in tuplets:
        sum_of += int(ch1 != ch2)

    return sum_of


def keyWithLowestVal(d):
     return min(d, key=d.get)


def findRepeatingXORKeysize(s):
    norm_edit_distances = {}
    for KEYSIZE in range(2,40):
        keysize_message = splitString(s, (KEYSIZE*8))
        total_blocks = len(keysize_message)
        distance = 0

        for i,k in zip(keysize_message[0::2], keysize_message[1::2]):
            distance +=hamming_distance(i,k)

        norm_edit_distances[KEYSIZE] = (float(distance)/float(total_blocks)) / float(KEYSIZE)
    #print norm_edit_distances
    return keyWithLowestVal(norm_edit_distances)

def decryptRepeatingKeyXOR(s, key):
    binary_keys = splitString(strToBinary(key), 8)
    encoded_string = ''
    for i, byte in enumerate(splitString(s, 8)):
        encoded_string += fixed_xor_str(byte, binary_keys[i%len(key)])
    return binaryToStr(encoded_string)
def findRepeatingXORKey(binary_cipher):
    KEYSIZE = findRepeatingXORKeysize(binary_cipher)
    #print KEYSIZE
    keysize_blocks = splitString(binary_cipher, 8*KEYSIZE)

    transposed_blocks = []

    for i, block in enumerate(keysize_blocks[0:-1]):
        for j, byte in enumerate(splitString(block, 8)):
            if i == 0:
                transposed_blocks.append(byte)
            else:
                transposed_blocks[j] = transposed_blocks[j] + byte
    #print transposed_blocks
    KEY = ''
    for block in transposed_blocks:

        #solve each block as if it where a single key XOR
        KEY += findSingleCharacterKey(block)

    return KEY
string_a = strToBinary('this is a test')
string_b = strToBinary('wokka wokka!!!')
assert hamming_distance(string_a, string_b) == 37, "hamming_distance incorrect"

# read message from file into list of blocks
with open ("3_6.txt", "r") as myfile:
    blocks = myfile.readlines()

blocks = [x.strip('\n') for x in blocks]

b64_cipher =  "".join(blocks)
binary_cipher = base64ToBinary(b64_cipher)
key = findRepeatingXORKey(binary_cipher)

message = decryptRepeatingKeyXOR(binary_cipher, key)
print message

```

## 第四题：Keyboard-Attack
数字键盘只有2468，可能当作上下左右，(据说ps打开可以看到这部分其实是没有标识的，有点神奇)那么剩余有标识的按键组成字典，然后sha1碰撞即可
```python
import itertools
from hashlib import sha1

s=''
k=[('Q','q'),('W','w'),('%','5'),('(','8'),
          ('=','0'),('I','i'),('*','+'),('N','n')]
for item in itertools.product(k[0],k[1],k[2],k[3],k[4],k[5],k[6],k[7]):
    s=''.join(item)
    for i in itertools.permutations(s):
        key=''.join(i)
        h=sha1(key)
        if h.hexdigest()=='67ae1a64661ac8b4494666f58c4822408dd0a3e4':
            print(key)
            break
```