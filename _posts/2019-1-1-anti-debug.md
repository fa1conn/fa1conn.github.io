---
layout:     post
title:      "anti-debug"
subtitle:   "anti-debug"
date:       2019-1-1 12:00:00
author:     "fa1con"
header-img: ""
header-mask: 0.3
catalog:    true
tags:
    - re
    - bin
---

[toc]
# linux反调试
## ptrace
原理：同一时间内进程最多只能被一个调试器进行调试。通过调试进程自身，判断是否已经有调试器的存在。

```c
#include <stdio.h>
#include <sys/ptrace.h>
int main(int argc, char *argv[]) {
    if(ptrace(PTRACE_TRACEME, 0, 0, 0) == -1) {
       printf("Debugger detected");
       return 1;
   }  
   printf("All good");
   return 0;
}
```
`PTRACE_TRACEME`来指明进程将被调试,忽略其他参数，如果正在被调试返回-1
## 检查父进程名称
原理：程序由gdb启动，所以父进程是gdb
```c
#include <stdio.h>
#include <string.h>
int main(int argc, char *argv[]) {
   char buf0[32], buf1[128];
   FILE* p;
 
   snprintf(buf0, 24, "/proc/%d/cmdline", getppid());
   p = fopen(buf0, "r");
   fgets(buf1, 128, p);
   fclose(p);
 
   if(!strcmp(buf1, "gdb")) {
       printf("Debugger detected");
       return 1;
   }  
   printf("All good");
   return 0;
}
```
## 检查进程运行状态
若程序是attach到调试器上的，程序的父进程就不是调试器，应该检查/proc/self/status文件，TracerPid由0变为非0的数，即调试器的PID
```c

#include <stdio.h>
#include <string.h>
int main(int argc, char *argv[]) {
   int i;
   scanf("%d", &i);
   char buf1[512];
   FILE* fin;
   fin = fopen("/proc/self/status", "r");
   int tpid;
   const char *needle = "TracerPid:";
   size_t nl = strlen(needle);
   while(fgets(buf1, 512, fin)) {
       if(!strncmp(buf1, needle, nl)) {
           sscanf(buf1, "TracerPid: %d", &tpid);
           if(tpid != 0) {
                printf("Debuggerdetected");
                return 1;
           }
       }
    }
   fclose(fin);
   printf("All good");
   return 0;
}
```
## 检测运行时间
`alarm`设置定时，到达时中止程序
```c
#include <stdio.h>
#include <signal.h>
#include <stdlib.h>
void alarmHandler(int sig) {
   printf("Debugger detected");
   exit(1);
}
void__attribute__((constructor))setupSig(void) {
   signal(SIGALRM, alarmHandler);
   alarm(2);
}
int main(int argc, char *argv[]) {
   printf("All good");
   return 0;
}
```
>__attribute__((constructor))指定该函数运行于main函数之前  
## 检查进程打开的filedescriptor(有问题，未解决)
如果被调试的进程是通过gdb <TARGET>的方式启动，那么它便是由gdb进程fork得到的。而fork在调用时，父进程所拥有的fd(file descriptor)会被子进程继承。由于gdb在往往会打开多个fd，因此如果进程拥有的fd较多，则可能是继承自gdb的，即进程在被调试。
```c
#include <stdio.h>
#include <dirent.h>
int main(int argc, char *argv[]) {
   struct dirent *dir;
   DIR *d = opendir("/proc/self/fd");
   while(dir=readdir(d)) {
       if(!strcmp(dir->d_name, "5")) {
           printf("Debugger detected");
           return 1;
       }
    }
   closedir(d);
   printf("All good");
   return 0;
}
```
参考文章：https://blog.csdn.net/earbao/article/details/53933238