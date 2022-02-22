---
title: matlab mex tutorial
comment: true
mathjax: true
date: 2021-06-16 21:42:16
updated: 2021-06-27 22:37
description: 在自己使用C语言写的二阶低通滤波器，需要查看自己写的C语言代码是否正确，是否能正确实现数字低通滤波，故需要验证一下。要实现把C语言产生的数据分享给导入matlab中，可以方便使用matlab来分析数据，或者方便画图。同时在设计滤波器参数，使用到了matlab 的 filterDesigner工具。当然C语言也有画图的库，或者在github上能找到，但是不想学习C的库函数，想用熟悉的matlab画图工具！
tags:
  - MATLAB
  - MEX
  - Filter

categories:
  - MATLAB

---



# 为什么要使用 matlab mex

我来聊聊怎么和 **matlab mex** 相遇的吧！哈哈哈！也可以说是我有求于 **matlab mex**,需要它的帮助！

*Matlab Mex brother! help me! help me!* 

自己在用C语言写低通滤波器，但是在实际的硬件调试电路中，发现滤波器输出的结果 **inf**(无穷大)，
到底是怎么一回事？我应该去哪里debug呢？


放下硬件调试电路，去周围溜达溜达，出去透透气，呼吸呼吸人间烟火。休息回来后，思考思考，滤波器为什
么不正常工作？ 难道我滤波器参数设置不对吗？还是C语言代码出了问题？还是其他原因？为了debug,还是把
这部分滤波器代码分离出来，看看是否正确？ 

在得到输入的信号数据和输出信号数据，需要对他们进行分析？由于有成千上万个数据点，不可能自己手输入到
matlab数组里面，那么有两种方案，解决数据分析问题，第一种还是用C语言分析数据和画图(C语言代码偏向底层，
不适合分析数据，python倒是可以，但是我的DSP中的程序是C语言，为了保持DSP中的代码和我测试代码一致性，
python不适合当前操作。去github上找C语言的图形库函数或数据分析包，这是一个不错的解决方案，但我主观上
不像学习其他库函数，学习成本比较大),第二种使用matlab和C语言的函数接口，即今天的主要内容 matlab mex, 
实现C语言数据和matlab数据之间的沟通。

---

# 操作环境

- OS：Linux
- matlab version: matlab 2017b
- gcc verison: 8.3.0 (5.3官方推荐)

---

# 低通滤波器离散设计

使用matlab的filterDesiner工具来设计低通滤波器，该工具主要用于滤波器的设计，可以设计多种滤波器。

## fiterDesiner quick Overview

在matlab command窗口输入 **filterDesigner** ,启动滤波器设计工具。如下图所示，为filterDesigner启动
后的界面。

**各个功能块做如下说明**：

1. 显示当前的设计滤波器的信息。其中 **Order**指的是滤波器的阶数
2. 设计滤波器的的类型，是低通滤波器还是高通滤波器还是带通滤波器或是其他的
3. 滤波器的设计方法，首先分为两大类(针对离散化滤波器设计)，IIR和FIR, 然后其中可以选择 butterworth或
chebyshev I或 chebyshev II滤波方法
4. 指定是滤波器的阶数，可以设置确定的阶数，也可以使用自动优化后的最小阶数，一般来说，最小阶数的方法
优化的滤波器结果更好，但是阶数一般都大于2阶以上，对于需要二阶或以下的滤波器，需要具体指定阶数。
5. 频率参数的设计，首先是**Unit**指定单位，然后 **Fs**为采样频率， 至于 **Fpass** **Fstop**可以查看中
的参数，可以明确知道， **Fpass**为截至频率, **Fstop**和 **Astop**配合， **Astop**指定具体的衰减分贝，
对应的频率为 **Fstop**.
6. 幅度参数设置，和频率参数的设计结合使用， **Apass**就是我们所说的通带衰减分贝和截至频率 **Fpass**相
对应，一般来说**Apass=3dB**, **Astop**和 **Fstop**对应，一般是每20dB对应衰减幅度$\frac{1}{10}$. 分贝
公式如下: $$ dB = -20 \lg A $$ A为增益倍数。

7. 参数设计的参考图

![filterDesigner overview](matlab-mex-tutorial/filterDesignerOverview.png)
针对**5,6**的设置多看看 **7**就明白了。


# filterDesigner Example

设计二阶低通滤波器，采用IIR butterworth 滤波器，截至频率$fc=600Hz$,采样频率$fs=10000Hz$,
具体操作，请看如下动图。

![filterDesigner Example](matlab-mex-tutorial/filterDesignerUseExample.gif)

设计后的具体参数如下：
```
--------------------------                
Section #1                                
--------------------------                
Numerator:                                
1                                         
2                                         
1                                         
Denominator:                              
 1                                        
-1.475480443592646073724949928873684257269
 0.586919508061190309256005548377288505435
Gain:                                     
0.027859766117136031127188289246987551451 
--------------------------                
Output Gain:                              
1        

```

该参数是z变换下的常数，参数对应的公式如下：
$$H(z)=0.027856\frac{1+2z^{-1}+z^{-2}}{1-1.17548z^{-1}+0.58692z^{-2}}$$
转换成离散表达式：
$$Y(n)=0.027856[X(n)+2X(n-1)+X(n-2)]+1.47548Y(n-1)-0.58693Y(n-2)$$
输入为X,输出为Y。通过上述离散表达式书写C语言数字滤波器。

# 低通滤波器C语言代码

## 二阶数字滤波器宏定义

```C

// IIR Butterworth second order sections
// fs=10kHz,fc=600Hz
#define LPF2_k1      0.02786		//gain
#define LPF2_a11     -1.47548		//denominator
#define LPF2_a12     0.58692
#define LPF2_b10     1*LPF2_k1		//numerator
#define LPF2_b11     2*LPF2_k1
#define LPF2_b12     1*LPF2_k1

#define PI 3.14159

struct filterValues {
	float x[3];	/* input values: x(n):x[0], x(n-1):x[1], x(n-2)=x[2]  */
	float y[3];	/* output values: y(n):y[0], y(n-1):y[1], y(n-2)=y[2]  */
};

```

## 二阶滤波函数

```C
/*! \brief low pass filter
 *
 * second order low pass filter:
 * 	fs=10khz(sample frequency), fc=600hz(cutoff frequency)
 * 	z transfer function:
 * 			   		1+2z^(-1)+z^(-2)
 * 		h(z)=0.02786  ---------------------------------
 *				1-1.47548z^(-1)+0.58692z^(-2)
 *
 * \param inputvalue: current input value
 * \param values: pointer to filtervalues
 *
 * \return no return
 */
void lowpassfilter(float inputvalue, struct filtervalues *values)
{
	values->x[0]=inputvalue;
	values->y[0] =lpf2_b10*(values->x[0])+lpf2_b11*(values->x[1])+lpf2_b12*(values->x[2])-lpf2_a11*(values->y[1])-lpf2_a12*(values->y[2]);

	values->x[2]=values->x[1];                                                  //x存储值更新
    	values->x[1]=values->x[0];

    	values->y[2]=values->y[1];                                                //y存储值更新
    	values->y[1]=values->y[0];                                                //y[0]为最终滤波输出
}

```

## 需要导入的头文件

```C
#include <stdio.h>
#include <math.h>
#include "mex.h"
```

**math.h**文件主要用于计算sin的值，用于产生信号。 **mex.h**主要用于C语言和matlab数据交互，
需要使用mex工具。

---

# matlab mex

## matlab mex install
matlab mex的安装比较简单，参考[MinGW-w64 C/C++ Compiler](https://ww2.mathworks.cn/matlabcentral/fileexchange/52848-matlab-support-for-mingw-w64-c-c-compiler?s_tid=mwa_osa_a)
，登录matlab帐号，下载安装文件即可。用于Window平台，GCC编译器的安装，基本上是点点点就行了。
但是需要外网，网络不好，只有手动安装GCC编译器。安装matlab版本相对应的GCC版本，对于 R2017b 
and R2018a GCC 5.3。最好安装和matlab版本一致的版本。 对于linux系统，就无需使用该方法安装
GCC编译器。linux一般情况下是自带gcc编译器，如果没有，安装也比windows方便。当前我的gcc版本
和推荐的版本不一致，比matlab匹配的GCC版本高, 在使用mex会有警告，提示版本不一致，可以忽略。
在我当前使用中是没有问题的。

![mex complie C ](matlab-mex-tutorial/mexFirstUse.png)

## matlab mexFunction
C语言的滤波函数和matlab mex已经配置完成。剩下的便是完成C语言和matlab之间的编程接口。
顺便提一下，matlab的mex支持C语言和Fortran语言。在本低通滤波操作使用的是C语言。通过
mex编译C语言的源文件，可以得到matlab可运行的mex文件。而在实现C和matlab之间的数据操
作，需要在C源代码文件中添加 **mexFunction** 函数，可以理解在C语言中的主函数，用于
调用自己写的其他C函数，实现具体的功能。

```C
#include "mex.h"				//引入mex头文件
void mexFunction(int nlhs, mxArray *plhs[], 	//输出结果
int nrhs, const mxArray *prhs[])		//输入参数
```

**mexFunction参数**: 

1. **nlhs**: Number of expected output mxArrays  输出结果的数量
2. **plhs**: Array of pointers to the expected output mxArrays  输出结果的指针数组
3. **nrhs**: Number of input mxArrays	输入参数的数量
4. **prhs**: Array of pointers to the input mxArrays. Do not modify any prhs values in your MEX file. 
Changing the data in these read-only mxArrays can produce undesired side effects. 输入参数的指针数组。

一句话总结一下mexFunction，输出参数的和放回结果的数量和指针数组(存储输入和输出的具体结果)。

## mexFunction中常用函数
对mexFunction中需要使用的一些常用函数做简要说明。

`mwPointer mxCreateDoubleMatrix(m, n, ComplexFlag)`
该函数用于创建一个矩阵，m为行，n为列， ComplexFlag用于指定矩阵中元素的数据类型，对于实数：mxREAL,
返回的是该矩阵的指针(地址)。一般是把返回值赋给数组plhs[]中的一个指针。实际使用该函数，主要用于在
内存中为矩阵分配空间。

`double *mxGetPr(const mxArray *pm)`
该函数用于获取数组的首地址，即第一个元素的地址指针。

`int mexPrintf(const char *message, ...)` 和C语言中的`printf()`函数的用法一样。


## mexFunction 具体C代码
mexFunction二阶低通滤波器的C源代码。在正弦基波分量的基础上，添加了高频率的毛刺
(也是正弦分量，只是频率比C基本的高很多)。如下为二阶低通滤波和matlab数据交换的具体代码：


```C
/*
 * nlhs 输出结果 数量
 * plhs 输出结果 指针
 *
 * nrhs 输入参数 数量
 * prhs 输入参数 指针
 *
 */

void mexFunction(int nlhs, mxArray *plhs[],
        int nrhs, const mxArray *prhs[])
{
	int N=400;
	int n=10;	//显示周期数
	int f=3000;	//相对频率
	int i,m;
	float delta=2*PI/N/f;
	float point=0;
    	double *input,*result;
	struct filterValues fvalues={{0,0,0},{0,0,0}};

	plhs[0]= mxCreateDoubleMatrix(1, N*n*f, mxREAL);
	plhs[1] = mxCreateDoubleMatrix(1, n*N, mxREAL);

	input  = mxGetPr(plhs[0]);
    	result = mxGetPr(plhs[1]);
	m=0;
	for (i = 0; i < N*n*f; ++i) {
		input[i]=sin(point)+0.2*sin(point*f);
		point+=delta;
		if (i%(f)==0) {
		LowPassFilter(input[i], &fvalues);
       		result[m]=fvalues.y[0];
	       	m++;	//remember increment m!
		}
		//mexPrintf("y[%d]=%f\n",i,fvalues.y[0]);
	}

}

```

## compile C source file by mex

在matlab命令行中输出如下内容，其中LowPassFilter.c为c语言源代码文件。
```C
mex LowPassFilter.c
```

编译成功后，可在matlab中直接执行LowPassFilter命令，由于上述有mexFunction
有两个返回值，故在matlab Commander窗口输入：

```
[a,b]=LowPassFilter;

% 作图
plot(linspace(0,10*pi,12000000),a)
hold on
plot(linspace(0,10*pi,4000),b)
```


## lowpass filer result

滤波结果，请见下图，可以发现滤波该滤波参数能较好的进行滤波。至于最初为什么
实际的硬件电路无法完成滤波，还需进一步查找原因，至少能排除参数和滤波器C语言
源代码没有问题。

![low pass filter result](matlab-mex-tutorial/lowpassResult.svg)

---

# 滤波器输出inf问题

在实际的DSP硬件电路中，数字滤波器输出无穷大**inf**问题，即`filtervalues.y[0]=inf`
需要解决该问题，正如前面排除滤波器C语言代码的问题，回想以下。用实际的示波器测量一
下滤波器未滤波的波形，发现每个周期有多个突变点，问题应该出在输入波形具有突变，输入
值的 **inf**,自然滤波得到的结果为无穷大。故在滤波的前，**输入信号增加一个限幅环节**。
即在调用`LowPassFilter(float inputvalue, struct filterDesigner *values)`之前，添加一句
限幅代码。限幅代码添加后，硬件滤波波形完好了。在实际硬件应用中，添加一个限幅环节，完美
解决问题。实际的软件仿真模拟中，输入信号比较理想，不存在突变的情况，故可以不加限幅环节。

**限幅！限幅！限幅！**

```C
for (i = 0; i < N*n*f; ++i) {
	input[i]=sin(point)+0.2*sin(point*f);
	point+=delta;
	/************添加的限幅代码 start*********************/
	//limit input values 
	if (input[i]>10) {
		input[i]=10;
	} else if (input[i]<-10) {
		input[i]=-10;
	}
	/*************添加的限幅代码 end********************/
	if (i%(f)==0) {
	LowPassFilter(input[i], &fvalues);
	result[m]=fvalues.y[0];
	m++;	//remember increment m!
	}
	//mexPrintf("y[%d]=%f\n",i,fvalues.y[0]);
}

```

# Conclusion
最终，终于实现C代码和matlab之间的互通，并且验证自己的数字低通滤波器参数和C语言
代码都没问题。做一天的硬件调试，发现问题出在未加限幅环节。因为实际的信号存在各种
不确定性，添加限幅环节，可以防止突变。

![实验低通滤波波形](matlab-mex-tutorial/experimentFilterWave.BMP)
请看上图的实验波形，棕色波形是带有高频信号噪声的正弦波，紫色波形是经过数字滤波后的
的波形。十分赏心悦目,终于实现硬件实验下的数字滤波。


# Reference

[Matlab 帮助文档](https://ww2.mathworks.cn/help/)




# 备注

I would greatly appreciate hearing about any error in this article, even minor ones.
I welcome your suggestions for improvements, even tiny one. You can give advice on
the following comment area and email to me!. Have fun!

 
