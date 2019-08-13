---
title: Linux-ARM环境Python编译安装
date: 2013-06-13 10:42:52
tags:
        - Linux
        - Python
        - ARM
        - 2013
categories:
        - Program
comments: false
lang:
        - zh-CN

---
[Python](https://www.python.org/)（英国发音：/ˈpaɪθən/ 美国发音：/ˈpaɪθɑːn/）, 是一种面向对象、解释型计算机程序设计语言，由Guido van Rossum于1989年发明，第一个公开发行版发行于1991年。
Python是纯粹的自由软件， 源代码和解释器CPython遵循 GPL(GNU General Public License)协议。

<!-- more -->
## **1. Python2.6** ##
- 1.1. 准备工作
> 源码包有：
> ```
Python-2.6.5.tar.bz2
sqlite-autoconf-3071700.tar.gz
```
> arm运行环境下的网络文件系统根目录
> ```
/root/rootfs
```
> 交叉编译环境
> ```
arm-***.tar.bz2
```
> 解压后目录为arm-\*\*\*/
> 将arm-\*\*\*/bin目录添加到环境变量
> 方法一：export PATH=$PATH:/解压目录/arm-\*\*\*/bin
> 方法二：/etc/profile文件末尾增加"PATH=$PATH:/解压目录/arm-\*\*\*/bin"

- 1.2. zlib
> ```
cd Python2.6.5/Modules/zlib
./configure --prefix=/root/rootfs
```
> 修改Makefile
> 将CC=gcc改为CC=arm-\*\*\*-gcc
> ```
make
make install
```

- 1.3. sqlite
> ```
tar xvf sqlite-autoconf-3071700.tar.gz
cd sqlite-autoconf-3071700
./configure --host=arm-none-linux-gnueabi --prefix=/root/rootfs --enable-shared --disable-readline --disable-dynamic-extensions
```
> 修改Makefile
> a. 将CFLAG的-g去掉
> b. 将CXXFLAG的-g去掉
> ```
make install
```

- 1.4. pgen
> ```
tar xvf Python-2.6.5.tar.bz2
cd Python-2.6.5
mkdir build.pc
cd build.pc
../configure
make
```

- 1.5. configure修改
> ```
cd Python-2.6.5
```
> 删除从“起始于”到”结束于“的内容
> 起始于
> ```
{ echo "$as_me:$LINENO: checking for %zd printf() format support" >&5
echo $ECHO_N "checking for %zd printf() format support... $ECHO_C" >&6; }
```
> 结束于
> ```
fi
rm -f core *.core core.conftest.* gmon.out bb.out conftest$ac_exeext conftest.$ac_objext conftest.$ac_ext
fi
```

- 1.6. Makefile生成
> ```
cd Python-2.6.5
mkdir build.arm
cd build.arm
../configure --host=arm-\*\*\* --prefix=/root/rootfs --enable-shared –disable-ipv6
```

- 1.7. Makefile修改
> ```
cd Python-2.6.5/build.arm
```
> - 行OPT=		-DNDEBUG -g -fwrapv -O3 -Wall -Wstrict-prototypes
> 改为OPT=		-DNDEBUG -fwrapv –O2 -Wall -Wstrict-prototypes
> - 行PGEN=		Parser/pgen$(EXE)下添加
> GEN_HOST = ../build.pc/Parser/pgen$(EXE)
> PYTHON_HOST= ../build.pc/python$(EXE)
> - 在使用PGEN的地方改为PGEN_HOST，亦即
> ```
$(GRAMMAR_H) $(GRAMMAR_C): $(PGEN) $(GRAMMAR_INPUT)
		-@$(INSTALL) -d Include
		-$(PGEN) $(GRAMMAR_INPUT) $(GRAMMAR_H) $(GRAMMAR_C)
```
> 改为
> ```
$(GRAMMAR_H) $(GRAMMAR_C): $(PGEN) $(GRAMMAR_INPUT)
		-@$(INSTALL) -d Include
		-$(PGEN_HOST) $(GRAMMAR_INPUT) $(GRAMMAR_H) $(GRAMMAR_C)
```
> - 所有的./$(BUILDPYTHON)都改为./$(PYTHON_HOST)

- 1.8. setup.py修改
> - 行build_ext.build_extension(self, ext)后面加return
> - 删除行
> ```
add_dir_to_list(self.compiler.library_dirs, '/usr/local/lib')
add_dir_to_list(self.compiler.include_dirs, '/usr/local/include')
```
> - 行lib_dirs = self.compiler.library_dirs + ['/lib64', '/usr/lib64',            '/lib','/usr/lib',]
> 改为lib_dirs = self.compiler.library_dirs + []
> - 行inc_dirs = self.compiler.include_dirs + ['/usr/include']
> 改为inc_dirs = self.compiler.include_dirs + []
> - 行        for d in inc_dirs + sqlite_inc_paths:
> 改为        for d in [‘/root/rootfs/include’]:
> - Main函数中
> ```
          scripts = ['Tools/scripts/pydoc', 'Tools/scripts/idle',
                     'Tools/scripts/2to3',
                     'Lib/smtpd.py']
```
> 改为
> ```
          scripts = []
```
> - 行disabled_module_list = []
> 改为disabled_module_list = [‘_ctypes’]
> - 行if (self.compiler.find_library_file(lib_dirs, 'z')):
> 改为if (self.compiler.find_library_file(['/root/rootfs/lib'], 'z')):
> - 行zlib_extra_link_args = ()
> 改为zlib_extra_link_args = ('-L/root/rootfs/lib',)

- 1.9. 解决Binascii和zlib问题
> 当/usr/local/lib/目录下有libz.a，libz.so两个文件时，将该两文件删除或者分别改名为libz.a.bak，libz.so.bak

- 1.10. 编译安装
> ```
cd Python-2.6.5/build.arm
make
make install
```

- 1.11. 精简Python
> cleanpy-2.6.sh
> ```
#!/bin/sh

DELFILE="
	bin/idle 
	bin/pydoc
	lib/python2.6/pydoc.pyo
	lib/python2.6/decimal.pyo
	lib/python2.6/doctest.pyo
	lib/python2.6/mailbox.pyo
	lib/python2.6/tarfile.pyo
	lib/python2.6/difflib.pyo
	lib/python2.6/imaplib.pyo
	lib/python2.6/gopherlib.pyo
	lib/python2.6/mhlib.pyo
	lib/python2.6/nntplib.pyo
	lib/python2.6/audiodev.pyo
	lib/python2.6/hashlib.pyo
	lib/python2.6/lib-dynload/audioop.so
	lib/python2.6/lib-dynload/_ctypes_test.so
	lib/python2.6/lib-dynload/_curses.so
	lib/python2.6/lib-dynload/_curses_panel.so
	lib/python2.6/lib-dynload/imageop.so
	lib/python2.6/lib-dynload/linuxaudiodev.so
	lib/python2.6/lib-dynload/ossaudiodev.so
	lib/python2.6/lib-dynload/rgbimg.so
	lib/python2.6/lib-dynload/_sha512.so
	lib/python2.6/lib-dynload/_sha256.so
	lib/python2.6/lib-dynload/pyexpat.so
	lib/python2.6/lib-dynload/Python-2.5.1.egg-info
	lib/python2.6/lib-dynload/_codecs_*.so
	lib/python2.6/lib-dynload/*_failed.so
	"
DELDIR="
	man 
	lib/python2.6/lib-tk		 
	lib/python2.6/ctypes		 
	lib/python2.6/email
	lib/python2.6/lib2to3
	lib/python2.6/xml
	lib/python2.6/json
	lib/python2.6/lib-old
	lib/python2.6/plat-linux2
	lib/python2.6/curses		 
	lib/python2.6/idlelib/		 
	lib/python2.6/test		 
	lib/python2.6/sqlite3/test	 
	lib/python2.6/bsddb/		 
	lib/python2.6/distutils
	lib/python2.6/email/test/	 
	lib/python2.6/ctypes/test/	 
	"

for i in $DELFILE ;
do
	[ -f $i ] && echo "exist file:" $i && rm -f $i
done

echo
echo

for i in $DELDIR ;
do
	[ -d $i ] && echo "exist dir:" $i && rm -rf $i
done

find lib/python2.6 -name '*.py' -exec rm -f \{\} \;
find lib/python2.6 -name '*.pyc' -exec rm -f \{\} \;

```
> 将cleanpy-2.6.sh拷贝到/root/rootfs/下，并运行即可

- 1.12. 运行Python
> 在ARM环境下，运行python之前
> 执行export PYTHONHOME=/
> 在执行python时，增加关键字-OO
> ```
python -OO xxx.py
```

----------
## **2. Python2.7** ##
- 1.1. 准备工作
> 源码包有：
> ```
Python-2.7.5.tgz
Python-2.7.5-xcompile.patch
```
> arm运行环境下的网络文件系统根目录
> ```
/root/rootfs
```
> 交叉编译环境
> ```
arm-***.tar.bz2
```
> 解压后目录为arm-\*\*\*/
> 将arm-\*\*\*/bin目录添加到环境变量
> 方法一：export PATH=$PATH:/解压目录/arm-\*\*\*/bin
> 方法二：/etc/profile文件末尾增加"PATH=$PATH:/解压目录/arm-\*\*\*/bin"

- 1.2. 解压
> ```
export RFS=/root/rootfs
tar xvf Python-2.7.5.tgz
cp XXX/Python-2.7.5-xcompile.patch Python-2.7.5/
```

- 1.3. 编译x86工具
> ```
cd Python-2.7.5/
./configure
make python Parser/pgen
mv python python_for_build
mv Parser/pgen Parser/pgen_for_build
make distclean
```

- 1.4. patch补丁
> ```
patch –p 1 < Python-2.7.5-xcompile.patch
```
> 运行后，根据提示输入要patch的文件

- 1.5. configure
> ```
./configure --host=arm-none-linux-gnueabi --build=i686-linux-gnu --prefix=/ ac_cv_file__dev_ptmx=no ac_cv_file__dev_ptc=no ac_cv_have_long_long_format=yes
```

- 1.6. make
> ```
make –jobs=8 CFLAGS="-g0 -Os -s -I${RFS}/usr/include -fdata-sections -ffunction-sections" LDFLAGS='-L${RFS}/usr/lib -L${RFS}/lib'
make install DESTDIR=${RFS} PATH=”${PATH}”
```

- 1.7. 精简Python
> cleanpy-2.7.sh
> ```
#!/bin/sh

DELFILE="
	bin/idle 
	bin/pydoc
	lib/python2.7/pydoc.pyo
	lib/python2.7/decimal.pyo
	lib/python2.7/doctest.pyo
	lib/python2.7/mailbox.pyo
	lib/python2.7/tarfile.pyo
	lib/python2.7/difflib.pyo
	lib/python2.7/imaplib.pyo
	lib/python2.7/gopherlib.pyo
	lib/python2.7/mhlib.pyo
	lib/python2.7/nntplib.pyo
	lib/python2.7/audiodev.pyo
	lib/python2.7/hashlib.pyo
	lib/python2.7/lib-dynload/audioop.so
	lib/python2.7/lib-dynload/_ctypes_test.so
	lib/python2.7/lib-dynload/_curses.so
	lib/python2.7/lib-dynload/_curses_panel.so
	lib/python2.7/lib-dynload/imageop.so
	lib/python2.7/lib-dynload/linuxaudiodev.so
	lib/python2.7/lib-dynload/ossaudiodev.so
	lib/python2.7/lib-dynload/rgbimg.so
	lib/python2.7/lib-dynload/_sha512.so
	lib/python2.7/lib-dynload/_sha256.so
	lib/python2.7/lib-dynload/pyexpat.so
	lib/python2.7/lib-dynload/Python-2.5.1.egg-info
	lib/python2.7/lib-dynload/_codecs_*.so
	lib/python2.7/lib-dynload/*_failed.so
	"
DELDIR="
	man 
	lib/python2.7/lib-tk		 
	lib/python2.7/ctypes		 
	lib/python2.7/email
	lib/python2.7/lib2to3
	lib/python2.7/xml
	lib/python2.7/json
	lib/python2.7/lib-old
	lib/python2.7/plat-linux2
	lib/python2.7/curses		 
	lib/python2.7/idlelib/		 
	lib/python2.7/test		 
	lib/python2.7/sqlite3/test	 
	lib/python2.7/bsddb/		 
	lib/python2.7/distutils
	lib/python2.7/email/test/	 
	lib/python2.7/ctypes/test/	 
	"

for i in $DELFILE ;
do
	[ -f $i ] && echo "exist file:" $i && rm -f $i
done

echo
echo

for i in $DELDIR ;
do
	[ -d $i ] && echo "exist dir:" $i && rm -rf $i
done

find lib/python2.7 -name '*.py' -exec rm -f \{\} \;
find lib/python2.7 -name '*.pyc' -exec rm -f \{\} \;

```
> 将cleanpy-2.6.sh拷贝到/root/rootfs/下，并运行即可

- 1.8. 运行Python
> 在ARM环境下，运行python之前
> 执行export PYTHONHOME=/
> 在执行python时，增加关键字-OO
> ```
python -OO xxx.py
```

