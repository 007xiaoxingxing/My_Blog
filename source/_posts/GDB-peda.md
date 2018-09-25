---
title: 在Ubuntu16.04上配置peda
date: 2016-11-20 19:25:18
categories: Linux
tags:
- GDB
- Linux
- Debugger
- peda
---
# 在Ubuntu16.04上配置peda


#### 0x1.peda是什么？
从github上摘抄一段关于peda的介绍如下:
``` 
peda
PEDA - Python Exploit Development Assistance for GDB
Key Features:

    Enhance the display of gdb: colorize and display disassembly codes, registers, memory information during debugging.
    Add commands to support debugging and exploit development (for a full list of commands use peda help):
        aslr -- Show/set ASLR setting of GDB
        checksec -- Check for various security options of binary
        dumpargs -- Display arguments passed to a function when stopped at a call instruction
        dumprop -- Dump all ROP gadgets in specific memory range
        elfheader -- Get headers information from debugged ELF file
        elfsymbol -- Get non-debugging symbol information from an ELF file
        lookup -- Search for all addresses/references to addresses which belong to a memory range
        patch -- Patch memory start at an address with string/hexstring/int
        pattern -- Generate, search, or write a cyclic pattern to memory
        procinfo -- Display various info from /proc/pid/
        pshow -- Show various PEDA options and other settings
        pset -- Set various PEDA options and other settings
        readelf -- Get headers information from an ELF file
        ropgadget -- Get common ROP gadgets of binary or library
        ropsearch -- Search for ROP gadgets in memory
        searchmem|find -- Search for a pattern in memory; support regex search
        shellcode -- Generate or download common shellcodes.
        skeleton -- Generate python exploit code template
        vmmap -- Get virtual mapping address ranges of section(s) in debugged process
        xormem -- XOR a memory region with a key
```
<!-- more -->
#### 0x2. 安装方法
```
    git clone https://github.com/longld/peda.git ~/peda
echo "source ~/peda/peda.py" >> ~/.gdbinit
echo "DONE! debug your program with gdb and enjoy"
```

#### 0x3. 尝试安装peda
```
star-chen@starchen-vb:~$ cat peda_install.sh 
git clone https://github.com/longld/peda.git ~/peda
echo "source ~/peda/peda.py" >> ~/.gdbinit
echo "DONE! debug your program with gdb and enjoy"

star-chen@starchen-vb:~$ sudo sh peda_install.sh 
正克隆到 '/home/star-chen/peda'...
remote: Counting objects: 304, done.
remote: Total 304 (delta 0), reused 0 (delta 0), pack-reused 304
接收对象中: 100% (304/304), 197.98 KiB | 45.00 KiB/s, 完成.
处理 delta 中: 100% (192/192), 完成.
检查连接... 完成。
DONE! debug your program with gdb and enjoy
star-chen@starchen-vb:~$ gdb
GNU gdb (Ubuntu 7.11.1-0ubuntu1~16.04) 7.11.1
Copyright (C) 2016 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.  Type "show copying"
and "show warranty" for details.
This GDB was configured as "x86_64-linux-gnu".
Type "show configuration" for configuration details.
For bug reporting instructions, please see:
<http://www.gnu.org/software/gdb/bugs/>.
Find the GDB manual and other documentation resources online at:
<http://www.gnu.org/software/gdb/documentation/>.
For help, type "help".
Type "apropos word" to search for commands related to "word".
gdb-peda$ 

```
安装完成。
   
   