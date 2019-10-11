---
title: Qt编译Oracle驱动
toc: false
date: 2019-05-06 15:56:18
description: Windows下Qt连接Oracle数据库，需要重新编译qsqloci.dll。
categories:
- Qt Everywhere
tags:
- Qt
---
1. 安装Oracle

   我安装的版本是Oracle Database 11*g* Release 2

2. 用QtCreator打开`C:\Qt\Qt5.6.3\5.6.3\Src\qtbase\src\plugins\sqldrivers\oci\oci.pro`

   我编译使用的Qt版本是Qt5.6.3 MSVC2015

3. 修改`qsql_oci.pri`

   添加`INCLUDEPATH`和`LIBS`

   ```
   HEADERS += $$PWD/qsql_oci_p.h
   SOURCES += $$PWD/qsql_oci.cpp
   
   INCLUDEPATH += "F:\app\cdpc001\product\11.2.0\dbhome_1\OCI\include"
   LIBS += "F:\app\cdpc001\product\11.2.0\dbhome_1\OCI\lib\MSVC\oci.lib"
   
   unix {
       !contains(LIBS, .*clnts.*):LIBS += -lclntsh
   } else {
   #    LIBS *= -loci
   }
   mac:QMAKE_LFLAGS += -Wl,-flat_namespace,-U,_environ
   
   ```

4. 构建完成

5. 若需要你的客户端程序能正常运行，除了要拷贝步骤4中构建生产的`qsqloci.dll`到plugins/sqldrivers下，还需要`oci.dll`以及它依赖的其他dll。

   你可以从官网下载对应的Instant Client以获得它。我下载的版本是instantclient-basic-nt-11.2.0.4.0。

