---
layout:     post
title:      "vhdl学习笔记之数据类型"
subtitle:   "vhdl learning"
date:       2018-11-21 12:00:00
author:     "fa1con"
header-img: ""
header-mask: 0.3
catalog:    true
tags:
    - re
    - bin
    - fpga
    - vhdl
---
#VHDL学习笔记之数据类型
##概述
相同数据类型的量才能互相传递和作用。  
## 分类
1.标量类型（SCALAR TYPE）  
2.复合类型（COMPOSITE TYPE）  
3.存取类型（ACCESS TYPE）  
4.文件类型（FILES TYPE）  
或者分为：预定义数据类型、用户自定义数据类型 。
## 定义
### 常量
constant 常数名:数据类型: = 表达式  
例如：  
constant Vcc: real:=5.0; --定义Vcc的数据类型  
constant bus_width: integer := 8 --定义总线宽度为常数8
### 变量
variable 变量名: 数据类型 :=初始值;  
variable count:integer 0 to 255:=20; --定义count整数变量，变化范围0-255，初始值为20。
#### 赋值
```
x:=10.0  
Y:=1.5+x --运算表达式赋值，注意表达式必须与目标变量的数据类型相同。
A(3 to 6):=("1101"); --位矢量赋值
```
### 信号
signal 信号名:数据类型:=初始值
```
signal clock： bit:='0'; --定义时钟信号类型  
signal count： BIT_VECTOR(3 DOWNTO 0); --定义count为4位位矢量
```
#### 信号赋值语句
目标信号名 <= 表达式；  
例如：  
```
x<=9;  
Z<=x after 5 ns; --在5ns后将x的值赋予z
```
## VHDL的预定义数据类型
### 1.布尔量（boolean）
false = 0  
ture = 1  
转换类型：boolean_var := (bit_var = '1');  
### 2.位(bit)
bit表示一位信号值。放在单引号中，如'0'或者'1'
### 3.位矢量 （bit_vector)
bit_vector 是用双引号括起来的一组位数据。  
如： "001100",X"00B10B"
### 4.字符(character)
variable character_var: character:='A'; 
### 5.整数(integer)
VHDL综合器要求对具体的整数作出范围限定,否则无法综合成硬件电路。
如：signal sig: interger 0 to 15:=2;  --必须指定范围