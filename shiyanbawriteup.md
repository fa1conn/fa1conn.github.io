1.CFG to C
题目连接http://ctf5.shiyanbar.com/reverse/cfg-to-c/index.html  
1）%ebp的值与0相比，大于等于则跳转至func_80484F8并减1回到该判断处，可以得出是B）  
2）%ebp与%eax比较，结果根据不同调至不同分支，并且返回不同的值，为C）  
3）这个比较难，因为我学的汇编语法与其并不是完全相同，只能看个大概，图中第二个框是判断过程，即比较i < b,因此12（%ebp）是i，%eax是b，而func_8048516中，%edi为i，%eax为c，顺便推出-4（%ebp）为c，由mov %eax，8（%ebp）得出8（%ebp）为a，-0x4(%ebp)则为i，答案为D  
4）很简单，为A

2.babyCrack  
链接： http://ctf5.shiyanbar.com/re/babyCrack.exe
直接开IDA不知为何找不到任何的字符串程序也无法进行F5，但是看了一会直接找到flag![image](http://ww3.sinaimg.cn/large/0060lm7Tly1flfd28yfuhj311y0lcmzp.jpg)

3.证明自己吧  
链接：http://ctf5.shiyanbar.com/crackme/  
1.IDA查找字符串![image](http://ww3.sinaimg.cn/large/0060lm7Tly1flfel39m5wj311y0lc421.jpg)
2.F5这时候看到了GoodKey这个if函数很可能就是核心算法。进入sub_401060 ![image](http://ww3.sinaimg.cn/large/0060lm7Tly1flfizsezfzj311y0lcgo2.jpg) 
3.
```
signed int __cdecl sub_401060(const char *a1)
{
  unsigned int v1; // edx@2
  unsigned int v2; // edx@4
  unsigned int v3; // edx@6
  int v5; // [sp+Ch] [bp-10h]@1
  int v6; // [sp+10h] [bp-Ch]@1
  int v7; // [sp+14h] [bp-8h]@1
  __int16 v8; // [sp+18h] [bp-4h]@1
  char v9; // [sp+1Ah] [bp-2h]@1

  v5 = dword_40708C;
  v6 = dword_407090;
  v8 = word_407098;
  v9 = byte_40709A;
  v7 = dword_407094;
  if ( strlen(a1) == strlen(&v5) )
  {
    v1 = 0;
    if ( strlen(a1) != 0 )
    {
      do
        a1[v1++] ^= 0x20u;
      while ( v1 < strlen(a1) );
    }//a1与0x20u进行异或
    v2 = 0;
    if ( strlen(&v5) != 0 )
    {
      do
        *(&v5 + v2++) -= 5;
      while ( v2 < strlen(&v5) );
    }//v5的值都减5
    v3 = 0;
    if ( strlen(&v5) == 0 )
      return 1;
    while (  *(&v5 + v3 + a1 - &v5) == *(&v5 + v3) )
    {
      ++v3;
      if ( v3 >= strlen(&v5) )
        return 1;
    }
  }//注意，将 *(&v5 + v3 + a1 - &v5)化简后得 *(v3 + a1)，也就是比较a1与v5各个值，相等即可
  return 0;
}
```
由此可以写个c程序
```
#include<stdio.h>
int main()
{
	int v5[14],a=0,b=0;
	int i,flag;
	for(;a<14;a++){
		scanf("%x",&v5[a]);
	}
	for(a=0;a<14;a++){
		v5[a]=v5[a]-5;
	}
	for(;b<14;b++){
		for(flag=0;flag<127;flag++){
			if((flag^32)==v5[b]){
				printf("%c",flag);
			}
		}
	}
	
}
```
![image](http://ww2.sinaimg.cn/large/0060lm7Tly1flfiupvui2j311y0lcadq.jpg)
现在说说解决时候遇到的问题：前面还好，遇到v5到底是啥的时候出了问题，由v5 = dword_40708C定位v5的位置，65Eh忘记补全了，应补成065EH,而且我一直没意识到应该是从68开始而不是48，另外写c也出了好几次bug，其中有一个是异或时没有加括号，而==运算优先级是高于^的
