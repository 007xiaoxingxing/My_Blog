---
title: 深入理解java虚拟机之动手编译OpenJDK 7
categories:
  - Program
tags:
  - java
date: 2017-05-31 19:31:28
---

### 编译环境构建

1. 操作系统

   我选择的是Ubuntu Server 14.04 LTS,系统已经自带了openjdk 7，可以作为编译openjdk 7的bootstrap jdk,这里我就省略了安装bootstrap jdk的步骤了。

2. 安装依赖的软件包
   ```bash
   $sudo apt-get install build-essential gawk m4 libasound2-dev libcups2-dev libxrender-dev xorg-dev xutils-dev binutils libmotif-dev ant
   ```

3. 下载openjdk 7源码并解压

   ```bash
   $wget http://download.java.net/openjdk/jdk7u75/ri/openjdk-7u75-src-b13-18_dec_2014.zip
   ```
   <!-- more -->
4. 设置一些编译所需的环境变量

   ```bash
   export LANG=C
   #bootstrap jdk的路径，我放在了~/jdk1.6
   export ALT_BOOTDIR=~/jdk1.6
   #允许编译过程中自动下载
   export ALLOW_DOWNLOADS=true
   #设置并行编译的线程数，我这虚拟机给了一个核，写个1
   export HOTSPOT_BUILD_JOBS=1
   export ALT_PARALLEL_COMPILE_JOBS=1
   export SKIP_COMPARE_IMAGES=true
   #使用预编译头文件
   export USE_PRECOMPILED_HEADER=true
   #需要编译的内容
   export BUILD_LANGTOOLS=true
   export BUILD_HOTSPOT=true
   export BUILD_JDK=true
   export BUILD_DEPLOY=false
   #编译结果的存放路径
   export ALT_OUTPUTDIR=~/compiled_jdk
   unset JAVA_HOEM
   unset CLASSPATH
   ```

### 准备就绪，开搞

```bash
xing@ubuntu-compile:~/openjdk$ make sanity 
.
.
.
External File/Binary Locations:
  USRJDKINSTANCES_PATH = /opt/java
  BUILD_JDK_IMPORT_PATH = /NOT-SET/re/jdk/1.7.0/promoted/latest/binaries
    ALT_BUILD_JDK_IMPORT_PATH = 
  JDK_IMPORT_PATH = /NOT-SET/re/jdk/1.7.0/promoted/latest/binaries/linux-amd64
    ALT_JDK_IMPORT_PATH = 
  LANGTOOLS_DIST = 
    ALT_LANGTOOLS_DIST = /home/xing/compiled_jdk/langtools/dist
  CORBA_DIST = 
    ALT_CORBA_DIST = /home/xing/compiled_jdk/corba/dist
  JAXP_DIST = 
    ALT_JAXP_DIST = /home/xing/compiled_jdk/jaxp/dist
  JAXWS_DIST = 
    ALT_JAXWS_DIST = /home/xing/compiled_jdk/jaxws/dist
  HOTSPOT_DOCS_IMPORT_PATH = /NO_DOCS_DIR
    ALT_HOTSPOT_DOCS_IMPORT_PATH = 
  HOTSPOT_IMPORT_PATH = /home/xing/compiled_jdk/hotspot/import
    ALT_HOTSPOT_IMPORT_PATH = /home/xing/compiled_jdk/hotspot/import
  HOTSPOT_SERVER_PATH = /home/xing/compiled_jdk/hotspot/import/jre/lib/amd64/server
    ALT_HOTSPOT_SERVER_PATH = 
  CACERTS_FILE = ./../src/share/lib/security/cacerts
    ALT_CACERTS_FILE = 
  CUPS_HEADERS_PATH = /usr/include
    ALT_CUPS_HEADERS_PATH = 
 
OpenJDK-specific settings:
  FREETYPE_HEADERS_PATH = /usr/include
    ALT_FREETYPE_HEADERS_PATH = 
  FREETYPE_LIB_PATH = /usr/lib
    ALT_FREETYPE_LIB_PATH = 
 
Previous JDK Settings:
  PREVIOUS_RELEASE_PATH = USING-PREVIOUS_RELEASE_IMAGE
    ALT_PREVIOUS_RELEASE_PATH = 
  PREVIOUS_JDK_VERSION = 1.6.0
    ALT_PREVIOUS_JDK_VERSION = 
  PREVIOUS_JDK_FILE = 
    ALT_PREVIOUS_JDK_FILE = 
  PREVIOUS_JRE_FILE = 
    ALT_PREVIOUS_JRE_FILE = 
  PREVIOUS_RELEASE_IMAGE = /usr/lib/jvm/java-7-openjdk-amd64/
    ALT_PREVIOUS_RELEASE_IMAGE = 


Sanity check passed.

xing@ubuntu-compile:~/openjdk$ make
.
.
.
```

### 遇到的问题​

``` 
cd linux_amd64_compiler2/product &amp;amp;amp;amp;amp;amp;amp;&amp;amp;amp;amp;amp;amp;amp; ./test_gamma
   Using java runtime at: /usr/lib/jvm/java-7-openjdk-amd64/jre
   Error occurred during initialization of VM
   java.lang.NullPointerException

   	at java.util.Hashtable.put(Hashtable.java:514)
   	at java.lang.System.initProperties(Native Method)
   	at java.lang.System.initializeSystemClass(System.java:1119)

   make[4]: *** [product] Error 1
   make[4]: Leaving directory `/home/xing/compiled_jdk/hotspot/outputdir'
   make[3]: *** [generic_build2] Error 2
   make[3]: Leaving directory `/home/xing/openjdk/hotspot/make'
   make[2]: *** [product] Error 2
   make[2]: Leaving directory `/home/xing/openjdk/hotspot/make'
   make[1]: *** [hotspot-build] Error 2
   make[1]: Leaving directory `/home/xing/openjdk'
   make: *** [build_product_image] Error 2
```

从网上找到的解决办法是删除makefile中的 ./test_gamma,或者安装oracle jdk1.6来作为bootstrap jdk应该也可以。
``` 
/usr/lib/jvm/java-7-openjdk-amd64//bin/java -XX:-PrintVMOptions -XX:+UnlockDiagnosticVMOptions -XX:-LogVMOutput -Xmx512m -Xms512m -XX:PermSize=32m -XX:MaxPermSize=160m -jar /home/xing/compiled_jdk/btjars/generatecurrencydata.jar -o /home/xing/compiled_jdk/lib/currency.data.temp \&amp;amp;amp;amp;amp;amp;amp;lt; ../../../src/share/classes/java/util/CurrencyData.properties
      Error: time is more than 10 years from present: 1136059200000
      java.lang.RuntimeException: time is more than 10 years from present: 1136059200000
      	at build.tools.generatecurrencydata.GenerateCurrencyData.makeSpecialCaseEntry(GenerateCurrencyData.java:285)
      	at build.tools.generatecurrencydata.GenerateCurrencyData.buildMainAndSpecialCaseTables(GenerateCurrencyData.java:225)
      	at build.tools.generatecurrencydata.GenerateCurrencyData.main(GenerateCurrencyData.java:154)
      make[4]: *** [/home/xing/compiled_jdk/lib/currency.data] Error 1
      make[4]: Leaving directory `/home/xing/openjdk/jdk/make/java/java'
      make[3]: *** [all] Error 1
      make[3]: Leaving directory `/home/xing/openjdk/jdk/make/java'
      make[2]: *** [all] Error 1
      make[2]: Leaving directory `/home/xing/openjdk/jdk/make'
      make[1]: *** [jdk-build] Error 2
      make[1]: Leaving directory `/home/xing/openjdk'
      make: *** [build_product_image] Error 2
```

错误提示是说时间超过了10年，解决办法是更改源文件，将10年前的时间改到十年之内。

### 编译结果

```
>>>Finished making images @ Fri Jun  2 01:43:41 CST 2017 ...
make[2]: Leaving directory `/home/xing/openjdk/jdk/make'
########################################################################
##### Leaving jdk for target(s) sanity all docs images             #####
########################################################################
##### Build time 00:06:34 jdk for target(s) sanity all docs images #####
########################################################################

#-- Build times ----------
Target all_product_build
Start 2017-06-02 01:36:50
End   2017-06-02 01:43:41
00:00:07 corba
00:00:04 hotspot
00:00:02 jaxp
00:00:02 jaxws
00:06:34 jdk
00:00:01 langtools
00:06:51 TOTAL
-------------------------
make[1]: Leaving directory `/home/xing/openjdk'

```

整个编译过程还是比较顺利的，遇到的也是些小问题~

### 运行一下

```bash
xing@ubuntu-compile:~/compiled_jdk$ ~/compiled_jdk/bin/java -version
openjdk version "1.7.0-internal"
OpenJDK Runtime Environment (build 1.7.0-internal-xing_2017_06_02_01_12-b00)
OpenJDK 64-Bit Server VM (build 24.75-b04, mixed mode)

```

可以看到已经能够成功运行，并且带上了编译的时间和用户。



参考书籍：《深入理解Java虚拟机 JVM高级特性与最佳实践》