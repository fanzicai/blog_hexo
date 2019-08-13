---
title: Windows环境Python调用C库
date: 2013-06-13 10:24:51
tags:
        - Windows
        - Python
        - C
        - 2013
categories:
        - Program
comments: false
lang:
        - zh-CN

---
Python调用C库，通常情况下采用[ctypes](http://starship.python.net/crew/theller/ctypes/)模块。本文则采用另外一种方式。

<!-- more -->
## **1. py2c工程** ##
采用VC创建“py2c”的dll工程。
> 将python安装目录下的include，添加到编译环境中
> 将python安装目录下的libs，添加到编译环境中

- py2c.c
> ```
// py2c.cpp : Defines the entry point for the DLL application.
//

#include "stdafx.h"

#include <Python.h>

#ifdef __cplusplus
extern "C"{
#endif
	static PyObject *py2c_showMsg(PyObject *self, PyObject *args);
	
	__declspec(dllexport) void initpy2c()
	{
		static PyMethodDef py2cMethods[] = {
			{
				"showMsg", py2c_showMsg, METH_VARARGS
			},
			{
				NULL, NULL, NULL
			}
		};
		PyObject *py2c = Py_InitModule("py2c", py2cMethods);
	}

	static PyObject *py2c_showMsg(PyObject *self, PyObject *args)
	{
		const char *msg = NULL;
		
		if(!PyArg_ParseTuple(args,"s",&msg))
			return NULL;

		int r = ::MessageBox(NULL, msg, "Caption:Form C module", MB_ICONINFORMATION | MB_OK);

		return Py_BuildValue("i", r);
	}


#ifdef __cplusplus
}
#endif



#pragma comment(lib,"python27.lib")

BOOL APIENTRY DllMain( HANDLE hModule, 
                       DWORD  ul_reason_for_call, 
                       LPVOID lpReserved
					 )
{
    return TRUE;
}

```

- 编译
> 编译工程，生成py2c.dll



----------
## **2. 使用py2c** ##
> 将编译生成的py2c.dll文件改名为py2c.pyd，然后放到python安装目录下的DLLS目录中
> 运行python
> ```
import py2c
py2c.showMsg(“test msg”)
```
> 运行结果，会弹出窗口显示“test msg”信息


----------
本文为在***其他环境***下进行“Python调用C库”打下基础。