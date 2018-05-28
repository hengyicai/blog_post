---
layout: post
title:  "C#调用Fortran"
date: 2015-04-03 08:42:42 +0800
comments: true
categories: 软件开发
description: 如何在C#中调用Fortran代码
---

在具体过程之前，有几点需要我们注意

- 需要将编译 Fortran 生成的 DLL 文件放置到 C# 的 Debug 目录中
- 对于数组我们要格外注意，因为 Fortran 中的数组存放是按列存放的(类似于 MATLAB ),而 C# 中的数组是和 C 的存储方式一致，按行存储。
- Fortran 中的类型和 C# 中类型的对应问题
- 标量值 在 Fortran 中是按引用传递，对应的， C# 中我们也应该对此有相同的处理，这里经常出现的异常是 memory access violations.


## 调用前

调用前我们应该把 Fortran 编译生成的 DLL 文件放到 C# 对应代码目录中的 Debug 文件夹中去，假设我们这里的开发环境是 Intel Visual Fotran（IVF） 和 Visual Studio (VS)，我们在 VS 中新建 Fortran   工程如下，并编写好代码，并进行编译，编译成功之后就会在项目的 Debug目录下生成对应的 DLL 文件

{% img center /images/新建Fortran项目.png %}

{% img center /images/编译Fortran.png %}

注意这里我们在Fortran代码中被调用子例程中写了如下的几行

```fortran

      !DEC$ ATTRIBUTES REFERENCE :: out_arr
      !DEC$ ATTRIBUTES DLLEXPORT, STDCALL, ALIAS:'SLAB' :: SLAB

      real , intent(in) , dimension(1:30) :: para
      real , intent(out), dimension(366,5) :: out_arr

```

注意前两行，这个是编译选项，第一行指定 out_arr 的传递类型为 REFERENCE , 第二行指定了**堆栈管理方式**由**被调用方**清除堆栈，这一点要和 C# 中的 管理方式一致，另外由于在Fortran编译器中默认的导出函数名全部是大写形式，而在C#中调用Fortran Dll时必须指定函数名一致，所以使用ALIAS(别名)属性指定导出函数名。

后面两行很清楚，我们指定传入参数和传出参数。

另外在 Fortran 和 C# 中的**参数类型**要保持一致，在Fortran中常用的数据参数类型有

- REAL:表示浮点数据类型，即小数，等价于C#的float,
- INTEGER:表示整数类型，相当于C#的int数据类型
- DOUBLE PRECISION:表示双精度数据类型，相当于C#的double数据类型。

## C#中对Fortran进行调用

调用的过程也很简单，我们只要导入 DLL 文件并进行声明即可

{% img center /images/1.png %}

注意声明部分，由于我们在 Fortran 中要是使用的例程的参数都是数组，我们在这里没有用 ref 关键字，若是传递值的话则需要。

接下来我们进行像普通方法一样进行调用即可

{% img center /images/2.png %}

因为这里我们接受的是数组，但是因为数组在 Fortran 中和 C# 中的存储方式不同，这里我们需要进行一下转换。

至此，我们完成了在 C# 中调用 Fortran 的粗浅讨论。

