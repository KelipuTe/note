---
draft: true
date: 2022-07-04 08:00:00 +0800
lastmod: 2022-07-04 08:00:00 +0800
title: "readelf"
summary: "readelf"
toc: true

categories:
- linux

tags:
- computer-science(计算机科学)
- operating-system(操作系统)
- linux
- command
---

> 操作系统架构：x86_64 AMD Ryzen <br/>
> 操作系统版本：Ubuntu 2204

> 样例代码 <br/>
> demo-c/demo-in-linux/memory-layout/code_and_data.c

### `readelf -h {xxx}`

输出文件头

```
> readelf -h code_and_data

ELF Header:
  Magic:   7f 45 4c 46 02 01 01 00 00 00 00 00 00 00 00 00
  Class:                             ELF64
  Data:                              2's complement, little endian
  Version:                           1 (current)
  OS/ABI:                            UNIX - System V
  ABI Version:                       0
  Type:                              DYN (Position-Independent Executable file)
  Machine:                           Advanced Micro Devices X86-64
  Version:                           0x1
  Entry point address:               0x1080
  Start of program headers:          64 (bytes into file)
  Start of section headers:          15288 (bytes into file)
  Flags:                             0x0
  Size of this header:               64 (bytes)
  Size of program headers:           56 (bytes)
  Number of program headers:         13
  Size of section headers:           64 (bytes)
  Number of section headers:         37
  Section header string table index: 36
```

### `readelf -s {xxx}`

输出符号表

```
> readelf -s code_and_data

Symbol table '.dynsym' contains 8 entries:
   Num:    Value          Size Type    Bind   Vis      Ndx Name
     0: 0000000000000000     0 NOTYPE  LOCAL  DEFAULT  UND
     1: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND _[...]@GLIBC_2.34 (2)
     2: 0000000000000000     0 NOTYPE  WEAK   DEFAULT  UND _ITM_deregisterT[...]
     3: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND __[...]@GLIBC_2.4 (3)
     4: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND [...]@GLIBC_2.2.5 (4)
     5: 0000000000000000     0 NOTYPE  WEAK   DEFAULT  UND __gmon_start__
     6: 0000000000000000     0 NOTYPE  WEAK   DEFAULT  UND _ITM_registerTMC[...]
     7: 0000000000000000     0 FUNC    WEAK   DEFAULT  UND [...]@GLIBC_2.2.5 (4)

Symbol table '.symtab' contains 41 entries:
   Num:    Value          Size Type    Bind   Vis      Ndx Name
     0: 0000000000000000     0 NOTYPE  LOCAL  DEFAULT  UND
     1: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS Scrt1.o
     2: 000000000000038c    32 OBJECT  LOCAL  DEFAULT    4 __abi_tag
     3: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS crtstuff.c
     4: 00000000000010b0     0 FUNC    LOCAL  DEFAULT   16 deregister_tm_clones
     5: 00000000000010e0     0 FUNC    LOCAL  DEFAULT   16 register_tm_clones
     6: 0000000000001120     0 FUNC    LOCAL  DEFAULT   16 __do_global_dtors_aux
     7: 0000000000004018     1 OBJECT  LOCAL  DEFAULT   26 completed.0
     8: 0000000000003db8     0 OBJECT  LOCAL  DEFAULT   22 __do_global_dtor[...]
     9: 0000000000001160     0 FUNC    LOCAL  DEFAULT   16 frame_dummy
    10: 0000000000003db0     0 OBJECT  LOCAL  DEFAULT   21 __frame_dummy_in[...]
    11: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS code_and_data.c
    12: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS crtstuff.c
    13: 00000000000021b8     0 OBJECT  LOCAL  DEFAULT   20 __FRAME_END__
    14: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS
    15: 0000000000003dc0     0 OBJECT  LOCAL  DEFAULT   23 _DYNAMIC
    16: 0000000000002088     0 NOTYPE  LOCAL  DEFAULT   19 __GNU_EH_FRAME_HDR
    17: 0000000000003fb0     0 OBJECT  LOCAL  DEFAULT   24 _GLOBAL_OFFSET_TABLE_
    18: 0000000000001169    19 FUNC    GLOBAL DEFAULT   16 func1
    19: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND __libc_start_mai[...]
    20: 0000000000000000     0 NOTYPE  WEAK   DEFAULT  UND _ITM_deregisterT[...]
    21: 0000000000004000     0 NOTYPE  WEAK   DEFAULT   25 data_start
    22: 0000000000004018     0 NOTYPE  GLOBAL DEFAULT   25 _edata
    23: 0000000000004010     4 OBJECT  GLOBAL DEFAULT   25 x
    24: 0000000000001298     0 FUNC    GLOBAL HIDDEN    17 _fini
    25: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND __stack_chk_fail[...]
    26: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND printf@GLIBC_2.2.5
    27: 0000000000004000     0 NOTYPE  GLOBAL DEFAULT   25 __data_start
    28: 0000000000000000     0 NOTYPE  WEAK   DEFAULT  UND __gmon_start__
    29: 0000000000004008     0 OBJECT  GLOBAL HIDDEN    25 __dso_handle
    30: 0000000000002000     4 OBJECT  GLOBAL DEFAULT   18 _IO_stdin_used
    31: 000000000000117c    21 FUNC    GLOBAL DEFAULT   16 func2
    32: 0000000000004020     0 NOTYPE  GLOBAL DEFAULT   26 _end
    33: 0000000000001080    38 FUNC    GLOBAL DEFAULT   16 _start
    34: 0000000000004014     4 OBJECT  GLOBAL DEFAULT   25 y
    35: 0000000000004018     0 NOTYPE  GLOBAL DEFAULT   26 __bss_start
    36: 0000000000001191   262 FUNC    GLOBAL DEFAULT   16 main
    37: 0000000000004018     0 OBJECT  GLOBAL HIDDEN    25 __TMC_END__
    38: 0000000000000000     0 NOTYPE  WEAK   DEFAULT  UND _ITM_registerTMC[...]
    39: 0000000000000000     0 FUNC    WEAK   DEFAULT  UND __cxa_finalize@G[...]
    40: 0000000000001000     0 FUNC    GLOBAL HIDDEN    12 _init
```
