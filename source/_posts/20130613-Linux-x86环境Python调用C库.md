---
title: Linux-x86环境Python调用C库
date: 2013-06-13 10:38:54
tags:
        - Linux
        - Python
        - C
        - x86
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
py2c工程，Makefile。
> 将python安装目录下的include，添加到Makefile中
> 将python安装目录下的libs，添加到Makefile中

- py2c.c
> ```
/* Example of embedding Python in another program */

#include "Python.h"

void initpy2c(void); /* Forward */

static PyObject *py2cError;

int
main(int argc, char **argv)
{
    /* Pass argv[0] to the Python interpreter */
    Py_SetProgramName(argv[0]);
    
    /* Initialize the Python interpreter.  Required. */
    Py_Initialize();
    
    /* Add a static module */
    initpy2c();
    
    /* Define sys.argv.  It is up to the application if you
       want this; you can also let it undefined (since the Python 
       code is generally not a main program it has no business
       touching sys.argv...) */
    PySys_SetArgv(argc, argv);
    
    /* Do some application specific code */
    printf("Hello, brave new world\n\n");
    
    /* Execute some Python statements (in module __main__) */
    PyRun_SimpleString("import sys\n");
    PyRun_SimpleString("print sys.builtin_module_names\n");
    PyRun_SimpleString("print sys.modules.keys()\n");
    PyRun_SimpleString("print sys.executable\n");
    PyRun_SimpleString("print sys.argv\n");
    
    /* Note that you can call any public function of the Python
       interpreter here, e.g. call_object(). */
    
    /* Some more application specific code */
    printf("\nGoodbye, cruel world\n");
    
    /* Exit, cleaning up the interpreter */
    Py_Exit(0);
    
    return 0;
    /*NOTREACHED*/
}

/* A static module */

/* 'self' is not used */
static PyObject *
py2c_test(PyObject *self, PyObject* args)
{
    const char *command;
    int sts;

    if (!PyArg_ParseTuple(args, "s", &command))
        return NULL;
    
    sts = printf(command);
    
    if (sts < 0) {
        PyErr_SetString(py2cError, "py2c test function failed!\n");
        return NULL;
    }
    
    printf("\n");
    
    return Py_BuildValue("i", sts);
    //return PyInt_FromLong(sts);
}

static PyMethodDef py2c_methods[] = {
    {"test",		py2c_test,	METH_VARARGS, //METH_NOARGS,
     "print string."},
    {NULL,		NULL}		/* sentinel */
};

void
initpy2c(void)
{
    PyObject *m;
    
    PyImport_AddModule("py2c");
    
    m = Py_InitModule("py2c", py2c_methods);
    if(m==NULL)
        return;
    
    py2cError = PyErr_NewException("py2c.error", NULL, NULL);
    Py_INCREF(py2cError);
    PyModule_AddObject(m, "error", py2cError);
    
    return;
}

```

- Makefile
> ```
# Makefile for embedded Python use py2c.
# edit lines marked with XXX.)

# XXX The compiler you are using
CC=	 	gcc

# XXX Top of the build tree and source tree
#srcdir=		../..
#blddir=		../..
srcdir= /usr/local/include
libdir= /usr/local/lib

# Python version
VERSION=	2.6

# Compiler flags
OPT=		-g
INCLUDES=	-I$(srcdir)/python$(VERSION) -I$(srcdir)
CFLAGS=		$(OPT)
CPPFLAGS=	$(INCLUDES)

# The Python library
LIBPYTHON=	$(libdir)/libpython$(VERSION).a
LIBEXTERN=  
#LIBEXTERN=  ./*.a

# XXX edit LIBS (in particular) to match $(blddir)/Modules/Makefile
#LIBS=		-lnsl -ldl -lreadline -ltermcap -lieee -lpthread -lutil
LIBS=		-lnsl -ldl -lieee -lpthread -lutil
LDFLAGS=	-Xlinker -export-dynamic -static
SYSLIBS=	-lm
MODLIBS=	
ALLLIBS=	$(LIBPYTHON) $(LIBEXTERN) $(MODLIBS) $(LIBS) $(SYSLIBS)

# Build the py2c applications
all:		clean py2c
py2c:		py2c.o
		$(CC) $(LDFLAGS) py2c.o $(ALLLIBS) -o py2c 

# Administrative targets

test: py2c
		./py2c

COMMAND="print 'hello world'"

clean:
		-rm -f *.o *.so py2c

clobber:	clean
		-rm -f *~ @* '#'* py2c

realclean:	clobber

```

- setup.py
> ```
from distutils.core import setup, Extension

module1 = Extension('py2c',
                    define_macros = [('MAJOR_VERSION', '1'),
                                     ('MINOR_VERSION', '0')],
                    #include_dirs = ['/usr/local/include'],
                    #libraries = ['***'],
                    #library_dirs = ['./'],
                    sources = ['py2c.c'])

setup (name = 'py2c',
       version = '1.0',
       description = 'This is a py2c package',
       author = '',
       author_email = '',
       url = '',
       long_description = 'This is really just a py2c package.',
       ext_modules = [module1])
```

- 编译
> ```
python setup.py build
python setup.py install
```
----------
## **2. 使用py2c** ##
> 将编译生成的py2c.dll文件改名为py2c.pyd，然后放到python安装目录下的DLLS目录中
> 运行python
> ```
import py2c
```

----------
本文中py2c_test函数还可以调用其他已有的C库，相当于通过py2c调用现有的C库。