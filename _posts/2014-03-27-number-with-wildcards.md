---
layout: post
title : 带通配符的数
category : algorithm
---
{% include JB/setup %}

csdn上有一个题目：

	给定一个带通配符问号的数W，问号可以代表任意一个一位数字。
	再给定一个整数X，和W具有同样的长度。
	问有多少个整数符合W的形式并且比X大？

	输入格式
	多组数据，每组数据两行，第一行是W，第二行是X，它们长度相同。在[1..10]之间.
	输出格式
	每行一个整数表示结果。

	输入样例
	36?1?8
	236428
	8?3
	910
	?
	5
	输出样例
	100
	0
	4

看了题目后，我的思路是按照W中?的个数N将题目分成N个部分来处理，如p[1]、p[2]、p[3]...

比如

				   p[1]  p[2]  p[3]
	12?45?7? --->  12?   45?   7?
				   w[1]  w[2]  w[3]
	12344589 --->  123   445   89
				   x[1]  x[2]  x[3]

我将上面的例子根据?的个数3，而分成3个部分来处理，分别为p[1]、p[2]、p[3]，每个部分对应的W和X的值(不包括?所在位)分别为w[1]、w[2]、w[3]和x[1]、x[2]、x[3]。

在每一个部分中，可以总结出以下规律：

	1）若w[i]>x[i]，则i以后的部分可以不用再处理
	2）若w[i]<x[i]，则i以后的部分可以不用再处理
	3）若w[i]=x[i]，则i以后的部分需要继续处理

根据上述3个规律，可以得出下面3个式子(其中result为处理部分i之前得到的结果，n为W中尚未处理的?的个数即N-i+1，x[p[i]]为W中第i个?所在位对应X中的值)：

	1) 若w[i]>x[i]，返回result = result*pow(10,n) + pow(10,n)
	2) 若w[i]<x[i]，返回result = result*pow(10,n)
	3) 若w[i]=x[i]，result = result*10 + 9 - x[p[i]]，继续处理i＋1部分

然后我根据上述思路编写了代码，但是测试没有通过，突然发现忘记考虑最后一个?之后有可能还有数字，我将其标记为w[N+1]和x[N+1]，根据上述规律，可以得出以下操作：

	若w[N+1]>x[N+1]，返回result ＝ result＋1，否则result无变化

对代码进行了修改之后，发现测试依然没有通过，思考了一下，发现对W[N+1]与X[N+1]的操作应该只有在w[i]与x[i]全相等的情况下才能进行。

最后得出了以下代码：

```c
#include <stdio.h>
#include <string.h>
#include <math.h>

int main(){
	char w[20],x[20];
	char *ptr,c;
	int result,cmp,count,index;
	while((gets(w)!=NULL) && (gets(x)!=NULL)){
		if(strcmp(w,x) > 0){
			c = '?';
			result = 0;
			count = 0;
			index = 0;
			while(1){
				ptr = strchr(w,c);
				if (ptr){
					count++;
					w[ptr-w] = '@';
				}else{
					break;
				}
			}
			c = '@';
			if(count == 0){
				double tmp1,tmp2;
				sscanf(w,"%lf",&tmp1);
				sscanf(x,"%lf",&tmp2);
				result = tmp1 - tmp2;
			}else{
				while(count){
					ptr = strchr(w,c);
					cmp = strncmp(w,x,ptr-w);
					if(cmp > 0){
					    	result = result*pow(10,count) + pow(10,count);
					    	break;
					}else if(cmp == 0){
							result = result*10 + '9' - x[ptr-w]; 
							count--;
							cmp = ptr-w+1;
							for(;index<cmp;index++){
								w[index] = '0';
								x[index] = '0';
							}
							index = cmp;
					}else{
						result = result*pow(10,count);
						break;
					}
				}
				if((count == 0) && (strcmp(w,x) > 0)){
					result += 1;
				}
			}
			
		}else{
			result = 0;
		}
		printf("%d\n", result);
		memset(w,0,sizeof(w));
		memset(x,0,sizeof(x));
	}
	return 0;
}
```

悲剧的是，上述代码依然没有通过测试。而且不知道是学校网络的问题还是csdn oj服务器的问题，我每次提交测试，一直卡在“编译和测试中，请稍候。。。”，每次都得把文件传给July大神，让他帮我提交测试，麻烦了他好几次，我也就不好意思再麻烦他了。最后一次的出错提示是这样的：

<center><img alt=“number-with-wildcards” src="{{ ASSET_PATH }}hooligan/img/post/number-with-wildcards.jpg"/></center>

如果你看到了这篇文章，并且知道上述思路错在哪里，欢迎留言告诉我。
