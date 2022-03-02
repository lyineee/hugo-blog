---
title: "C++/C 学习"
date: 2022-02-21
---

## 编译器

GCC 编译器分为了前端和后端

前端只要做词法分析等工作，这些工作应该只与语言的类型有关，比如 C，C++，而后端的工作与系统底层更为相关，主要的工作是代码的优化等。前端与后端需要有中间语言来进行通信，在 GCC 中是 obj 文件。

---

GCC的编译步骤有四个：

1. 预处理生成 .i 参数：-E
2. 编译生成汇编文件 .s 参数：-S
3. 汇编生成 .o **目标文件** 参数：-c （这个命令也可以直接编译 cpp 文件为 .o 文件）
4. 联接生成可执行文件 .exe

参考：[https://blog.csdn.net/edisonlg/article/details/7081357](https://blog.csdn.net/edisonlg/article/details/7081357)

## 概念

编译单元（Translation Unit）：一个 .cpp 文件和它所 include 的所有 .h 文件。在编译时，.h 文件中的内容都会扩展到 .cpp 文件中后再进行编译。

未解决符号表（unresolved symbol table）：目标文件中用于告诉编译器有哪些符号地址是未知的表（extern 关键字）

导出符号表（export symbol table）：目标文件中用于告诉编译器自己可以提供哪些符号地址的表

[C++编译链接过程_edisonlg的专栏-CSDN博客](https://blog.csdn.net/edisonlg/article/details/7081357)

[名字修饰](https://zh.wikipedia.org/wiki/%E5%90%8D%E5%AD%97%E4%BF%AE%E9%A5%B0)

名字修饰（name mangling）：编译后在目标文件中对符号名进行修改，在联接时可以提供额外的信息。不如 C++ 中重载的实现就是把参数类型加入到符号名中。

---

[Misc](C++%20C%20%E5%AD%A6%E4%B9%A0%208b647/Misc%20242bd.md)

[目标](C++%20C%20%E5%AD%A6%E4%B9%A0%208b647/%E7%9B%AE%E6%A0%87%207b8f6.md)

[clang 与 llvm](C++%20C%20%E5%AD%A6%E4%B9%A0%208b647/clang%20%E4%B8%8E%20ll%20caf3b.md)

[llvm 编译](C++%20C%20%E5%AD%A6%E4%B9%A0%208b647/llvm%20%E7%BC%96%E8%AF%91%20f7830.md)