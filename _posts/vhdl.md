---
layout:     post
title:      "vhdl"
subtitle:   "vhdl"
date:       2018-11-23 12:00:00
author:     "fa1con"
header-img: ""
header-mask: 0.3
catalog:    true
tags:
    - re
    - bin
---

一般情况下，一个完整的vhdl程序至少包含程序包，实体，结构体。
# 实体
```
ENTITY  实体名称 IS
GENERIC （
常数名称1 ：类型 [:= 缺省值] ；
…
常数名称N ：类型 [:= 缺省值] ；
）；
PORT （
端口信号名称1 ：输入/ 输出状态数据类型；
…
端口信号名称N ：输入/ 输出状态数据类型
）；
END  实体名称；
```
看一本书上的例子来学习一下
```
ENTITY cntm16 IS
PORT(
EN : IN STD_LOGIC;
Rd : IN STD_LOGIC;
CLK : IN STD_LOGIC;
Co : OUT STD_LOGIC;
Q : BUFFER STD_LOGIC_VECTOR(3 DOWNTO 0)--最后一个端口结尾不需要分号
);
END cntm16;
```
IN和OUT不多说  
INOUT----表示信号是双向的，既可以进入电路单元也可以从电路单元输出。  
BUFFER---信号从电路单元输出，同时在电路单元内部可以使用该输出信号。
OUT与BUFFER区别：信号是否内部有反馈。
# 结构体
结构体是以标识符ARCHITECTURE开头，以END结尾。
>描述方式:行为（BEAVHER）描述方式、数据流（DATAFLOW）描述方式和结构(STRUCTURE)描述方式

看一下模版  
```
ARCHITECTURE  结构体名 OF  实体名称 IS
说明语句
BEGIN
电路描述语句
END  结构体名；
```
>结构体中定义的参数（信号，变量等）名称不能与其所属实体的端口名重名。

接着上一个的实例，看一下它的结构体
```
ARCHITECTURE counstr OF cntm16 IS
BEGIN
    Co <= "1"WHEN (Q ="1111"AND EN ="1") ELSE „0‟; --条件赋值语句
        PROCESS (CLK, Rd) --PROCESS语句
        BEGIN
        IF (Rd=„0‟) THEN --IF语句
        Q<= ”0000”;
        ELSIF (CLK‟ EVENTAND CLK=„1‟) THEN --CLK上升沿计数
            IF(EN=„1‟) then
            Q <= Q+1;
            ENDIF;
        END IF;
    END PROCESS;
END counstr;
```