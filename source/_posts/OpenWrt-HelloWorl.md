---
title: OpenWRT软件开发-----软件包HeeloWorld的构建
categories:
  - Linux
tags:
  - OpenWRT
date: 2017-09-09 15:24:56
---

OpenWrt提供了方便的机制来让用用户可以扩充和实现自己所需的功能。这篇博文就说明一下怎么构建一个能够安装在OpenWrt上ipk包的过程。

<!--more-->

1.文件目录


	helloworld/
	├── Makefile  #Makefile 必不可少
	└── src #源码目录
		├── Hello.c #程序源码
		└── Makefile #编译源码的Makefile

2.文件内容

src/Hello.c

```c
#include <stdio.h>
#include <unistd.h>
int main(void)
{
     printf("Hell! Dear OpenWrt!\n\n");
     return 0;
}
```

src/Makefile

```makefile
# build Hello executable when user executes "make"
Hello:Hello.o
	$(CC) $(LDFLAGS) Hello.o -o Hello
Hello.o:Hello.c
	$(CC) $(CFLAGS) -c Hello.c
#remove object files and executable when user executes "make clean"
clean:
	rm *.o Hello

```

构建OpenWrt软件包最主要的内容都包括在Makefile中了，首先看一下Makefile的内容。

Makefile

```makefile
##############################################
# OpenWrtMakefile for HelloWorld program
#
#
# Most ofthe variables used here are defined in
# theinclude directives below. We just need to
# specifya basic description of the package,
# whereto build our program, where to find
# thesource files, and where to install the
# compiled program on the router.
#
# Be verycareful of spacing in this file.
# Indentsshould be tabs, not spaces, and
# thereshould be no trailing whitespace in
# linesthat are not commented.
#
##############################################
include $(TOPDIR)/rules.mk 
# Nameand release number of this package
PKG_NAME:=Hello #软件包的名称，编译的时候可以看到，安装，卸载所用到的名称
PKG_RELEASE:=1 #软件包的版本
# Thisspecifies the directory where we're going to build the program.
# Theroot build directory, $(BUILD_DIR), is by default the build_mipsel
#directory in your OpenWrt SDK directory
PKG_BUILD_DIR:= $(BUILD_DIR)/$(PKG_NAME)  #源码所在的目录
include $(INCLUDE_DIR)/package.mk
# Specifypackage information for this program.
# Thevariables defined here should be self explanatory.
# If youare running Kamikaze, delete the DESCRIPTION
#variable below and uncomment the Kamikaze define
# directivefor the description below
define Package/$(PKG_NAME)  #这里是定义在make menuconfig的时候，该软件包出现的位置
	SECTION:=utils
	CATEGORY:=Utilities
	TITLE:=HelloOpenwrt-- Start to build software on your router!
endef
# Specifywhat needs to be done to prepare for building the package.
# In ourcase, we need to copy the source files to the build directory.
# This isNOT the default. The default uses thePKG_SOURCE_URL and the
#PKG_SOURCE which is not defined here to download the source from the web.
# Inorder to just build a simple program that we have just written, it is
# mucheasier to do it this way.
define Build/Prepare
	mkdir -p $(PKG_BUILD_DIR)
	$(CP) ./src/* $(PKG_BUILD_DIR)/
endef
# We donot need to define Build/Configure or Build/Compile directives
# Thedefaults are appropriate for compiling a simple program such as this one
# Specifywhere and how to install the program. Since we only have one file,
# thehelloworld executable, install it by copying it to the /bin directory on
# therouter. The $(1) variable represents the root directory on the router running
#OpenWrt. The $(INSTALL_DIR) variable contains a command to prepare the install
#directory if it does not already exist. Likewise $(INSTALL_BIN) contains the
# commandto copy the binary file from its current location (in our case the build
#directory) to the install directory.
define Package/$(PKG_NAME)/install  #定义软件包安装行为
	$(INSTALL_DIR) $(1)/bin #需要安装的目录，这里必须先指定，不然编译的会报找不到目录的错误
	$(INSTALL_BIN) $(PKG_BUILD_DIR)/Hello $(1)/bin/
endef
# Thisline executes the necessary commands to compile our program.
# Theabove define directives specify all the information needed, but this
# linecalls BuildPackage which in turn actually uses this information to
# build apackage.

$(eval $(call BuildPackage,$(PKG_NAME))) #正式开始执行编译

```

代码编写完成后，就可以把软件包放到OpenWrt源码的Package目录或者OpenWrt SDk的Package目录中去，执行make，就可以生成ipk软件包了。