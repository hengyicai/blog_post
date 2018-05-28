---
layout: post
title:  "利用F2PY实现在Python中调用Fortran"
date: 2015-03-31 19:35:46 +0800
comments: true
categories: 软件开发
description: 利用F2PY实现在Python中调用Fortran
---

## 在linux下实现Python调用Fortran

首先，Python是解释型语言，Fortran是编译型语言，在你的机器上首先应该装好了对应的解释器和编译器，具体的安装过程这里不再赘述，**本文的Python解释器是Python2.7，Fortran编译器是gfortran**。

假设你的需求是这样的，现在已经有了一个完整的 Fortran 代码，你可能要在 Python 中调用其中的某些模块，**F2PY(Fortran to Python interface generator)**项目的目的是要在Python 和 Fortran 语言之间提供一个连接。F2PY 是一个 Python 包(包含一个命令行工具 f2py 和一个模块 f2py2e),用来建立 **Python C/API 扩展模块**,从而能够

-  调用 Fortran 77/90/95 外部子程序、Fortran 90/95 模块子程序以及C 函数;
-  访问 Fortran 77 COMMON blocks 和 Fortran 90/95 module 数据,包括 allocatable arrays。

下图可以给个大致的过程

{% img center /images/F2PY.bmp %}

下面给出一个简单的示例，假设 Fortran 代码如下

```fortran

  SUBROUTINE main_routine( para, out_arr)

      real , intent(in) , dimension(1:30) :: para
      real , intent(out), dimension(366,5) :: out_arr

      common sr3,third,f227,dbt,utst,wse,bs,bse,nssm
      common bb0,b0,r0,utm3,utc3,sft,htp,htp0,ncalc
      common h,bb,b,beta,r,g,vg,v,w,cm,u,t,rho,cv,xn,x
      common wma,cpa,grav,rhoa,rhos,pa,rr,qs,ws,hrf,urf
      common cf0,cf1,cri,tutrd,f17,f19,fu,ft,fv,sfu
      common cth,keval,uab,uab0,u0,cws,wss,ala0,zt,cu1,cu2

c =================
c == more code ====
c =================

      grav = 9.80665
      pa = 101325.

      wmw = 0.01802
      rhowl = 1000.
      cpwv = 1846.
      cpwl = 4178.
      dhw = 2441000.
      spaw = 15.08
      spbw = 5514.

      rpwa = .01*rh*exp(spaw-spbw/ta)
      cmwa = wmw*rpwa/(wma+(wmw-wma)*rpwa)
      cmdaa = 1.-cmwa
      cpaa = cmdaa*cpa + cmwa*cpwv
      wmae = wma*wmw/(wmw+(wma-wmw)*cmwa)

c  number of integration steps

      nssm = ncalc + ncalc + ncalc
      msfm = 11
      mnfm = 50
      mffm = 61  

c ================
c ================


      call editcc
        

      do l = 1,6
          do m = 1,61
              index_i = (l-1)*61 + m
              out_arr(index_i,1) = xp(m) 
              out_arr(index_i,2) = (-0.5 + l*0.5)*bbcp(m)
              out_arr(index_i,3) = zp(1)
              out_arr(index_i,4) = tav
              out_arr(index_i,5) = cvpt(l,1,m)
          end do 
      end do 

      END SUBROUTINE main_routine
```

上述代码可能是你要调用的Fortran代码的某一个例程，名叫main_routine，这个例程中可能有很多**common**修饰的变量，或者大量调用其他例程，但是我们看到，它接收两个数组，第一个作为输入，第二个作为输出，这就够了，接下来我们用 **F2PY** 来编译它

{% img center /images/F2PY.png %}

编译之后我们看到有如下输出，且生成的**slab.so**是动态链接库文件

{% img center /images/F2PY_1.png %}

之后我们在python控制台将其引入，不加后缀名**.so**，进行调用，传入**main_routine**例程所需参数数组(30个元素的数组)，我们便可以得到返回的输出数组，如下

{% img center /images/F2PY_2.png %}

可能有人疑问，在 Fortran 中**main_routine**是接收两个参数，为什么这里只接收一个，并且有返回值，原因在于前两行，你可以在上面的 Fortran 例程中看到，由 **intent(in) 来修饰 第一个参数， 由 intent(out) 来修饰第二个参数**，所以F2PY可以识别出**接收参数**和**返回参数**，对于 F2PY 还有很多的智能化选项，使得更方便地在 Python 中调用Fortran，具体可参见官方文档。 

## Windows 实现 Python 调用 Fortran

Windows 下的过程和 linux 下的是大致相同的，不过在 F2PY 编译 Fortran 源代码的时候要注意指定**编译参数**，在此之前确保你正确安装了对应 Python 版本的 **Numpy模块**，Fortran 编译器 **G95-MinGW-41** 和 **mingGW** (本文最后给出了这三个程序在  win64，python2.7 环境下的下载地址),并设置好了相关的环境变量， G95 在安装的时候会提示你让其设置环境变量，确认即可， mingGW 需要手动将其bin路径添加到**系统的PATH变量**中，这些做好之后，在需要调用的 Fortran 代码所在的文件夹下打开 cmd 执行如下命令


```plain

C:\Python27\python.exe   C:\Python27\Scripts\f2py.py   -c   --fcompiler=gnu95   --compiler=mingw32   -m   指定模块名   指定Fortran文件名

```
你需要将第一个和第二个参数换成你自己本机的路径，并且指定模块名(如: hello )，指定 Fortran 文件名(如 hello.f )。

{% img center /images/compile_fo.png %}

执行之后，生成一个 **hello.pyd 文件**，将这个文件放到你的 python 库文件中，便可以在脚本中使用 **import hello** 导入该模块，之后就像使用其他模块一样使用它即可，注意，无论是 linux 环境下生成的 so 文件还是 windows 下生成的 pyd 文件，都是不可移植的，也就是说你不能简单的把它拷贝到另外一台机器上然后使用。

## 下载链接

[numpy 下载地址](http://iweb.dl.sourceforge.net/project/numpy/NumPy/1.8.1/numpy-1.8.1-win32-superpack-python2.7.exe)

[g95-MinGW 下载地址](http://tcc.customer.sentex.ca/g95/4.1/g95-MinGW-41.exe)

[mingw-w64 下载地址](http://iweb.dl.sourceforge.net/project/mingw-w64/Toolchains%20targetting%20Win32/Personal%20Builds/mingw-builds/installer/mingw-w64-install.exe)











