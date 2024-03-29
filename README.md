# windows2012 下编译32位mod_wsgi并配置django项目

## 需要软件

1.apache2.4 32位<br>
https://www.apachelounge.com/download/VS16/binaries/httpd-2.4.39-win32-VS16.zip<br>
下载解压放在C:\Apache24<br>

2.python 3.7.1 32位<br>
正常安装<br>

3.Build Tools for Visual Studio 2019<br>
https://visualstudio.microsoft.com/zh-hans/downloads/<br>
正常安装<br>

4.mod_wsgi-4.6.5<br>
https://codeload.github.com/GrahamDumpleton/mod_wsgi/zip/4.6.5<br>
下载解压放在C:\mod_wsgi-4.6.5<br>

## 编译准备

1.拷贝common-VC10.mk 成 common-VC14.mk，并修改如下

```
CPPFLAGS = \
 /machine:x86 \
 /DNDEBUG \
 /I"$(APACHE_ROOTDIR)\include" \
 /I"$(PYTHON_ROOTDIR)\include"

CFLAGS = \
 /MD \
 /GF \
 /Gy \
 /O2 \
 /Wall \
 /Zc:wchar_t \
 /Zc:forScope

LDFLAGS = \
 /link \
 /LIBPATH:$(APACHE_ROOTDIR)\lib \
 /LIBPATH:$(PYTHON_ROOTDIR)\libs \
 /OPT:REF \
 /OPT:ICF=2 \
 /RELEASE \
 /SUBSYSTEM:WINDOWS

LDLIBS = \
 python$(PYTHON_VERSION).lib \
 libhttpd.lib \
 libapr-1.lib \
 libaprutil-1.lib

SRCFILES = ..\src\server\*.c

mod_wsgi.so : $(SRCFILES)
	cl $(CPPFLAGS) $(CFLAGS) $(SRCFILES) /LD $(LDFLAGS) $(LDLIBS) /OUT:$@

VARIANT = py$(PYTHON_VERSION)-VC14

install : mod_wsgi.so
	copy $? $(APACHE_ROOTDIR)\modules\mod_wsgi-$(VARIANT).so
	:
	:
	:
	:
	: You now need to edit $(APACHE_ROOTDIR)\conf\httpd.conf and add:
	:
	:   LoadModule wsgi_module modules/mod_wsgi-$(VARIANT).so
	:
	:
	:
	:

clean :
	del *.obj *.so *.so.manifest *.lib *.exp
``` 
  
2.拷贝ap24py34-win32-VC10.mk 成 ap24py37-win32-VC14.mk，并修改如下

```
APACHE_ROOTDIR = c:\Apache24
PYTHON_ROOTDIR = C:\Users\aicontractAgent\AppData\Local\Programs\Python\Python37-32
PYTHON_VERSION = 37
include common-VC14.mk
``` 

## 编译

打开developer Command Prompt for VS 2019执行下面的命令
``` 
cd C:\mod_wsgi-4.6.5\win32
nmake -f ap24py37-win32-VC14.mk
``` 

## 结果

mod_wsgi.so文件生成在mod_wsgi-4.6.5\win32下

## 后续

在httpd.conf最后里添加
```
WSGIPythonPath "C:/xxx/Python/Python37-32;C:/Users/xxx/Python/Python37-32/Lib/site-packages"
<VirtualHost *:80>
    WSGIPassAuthorization On
    WSGIScriptAlias / "C:/xxx/wsgi.py"
    <Directory C:/xxx>
        <Files wsgi.py>
            Require all granted
        </Files>
    </Directory>
</VirtualHost>
```

wsgi.py文件里添加，否则会出现找不到module
```
import sys
sys.path.append(os.path.dirname(os.path.abspath(__file__)))
sys.path.append(os.path.dirname(os.path.abspath(__file__)) + '/..')
```

启动apache，终于出来了
