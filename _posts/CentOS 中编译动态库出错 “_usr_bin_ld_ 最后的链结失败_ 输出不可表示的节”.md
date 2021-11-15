---
title: "CentOS 中编译动态库出错 “/usr/bin/ld: 最后的链结失败: 输出不可表示的节”"
date: 2021-10-23 14:17:57
updated: 2021-10-23 14:17:57
categories: [技术杂谈]
tags: [CentOS,GCC]
toc: true
---



问题场景：使用 GCC 编译动态库时报错。

中文报错信息：

```
/usr/bin/ld: 最后的链结失败: 输出不可表示的节
```

英文报错信息：

```
/usr/bin/ld: final link failed: Nonrepresentable section on output
```

<!--more-->

-

我这里的解决方案是在编译命令最后添加参数 `-no-pie`。

如果你的问题解决了，并且你关心为何导致此错误，那就接着看下面的内容。

-

详细的错误信息有好多条，这里截取了几条

```
...
/usr/bin/ld: ./utilities/transactions/lock/range/range_tree/lib/locktree/manager.o: relocation R_X86_64_32 against symbol `__pthread_key_create@@GLIBC_2.2.5' can not be used when making a shared object; recompile with -fPIC
/usr/bin/ld: ./utilities/transactions/lock/range/range_tree/lib/locktree/wfg.o: relocation R_X86_64_32 against symbol `__gxx_personality_v0@@CXXABI_1.3' can not be used when making a shared object; recompile with -fPIC
/usr/bin/ld: ./utilities/transactions/lock/range/range_tree/lib/standalone_port.o: relocation R_X86_64_32S against `.rodata.str1.1' can not be used when making a shared object; recompile with -fPIC
...
```

诸如此类错误信息中，最后都有一个 `recomplie with -fPIC`，但是当我加上 `-fPIC` 后重新编译，又报错如下：

中文报错信息：

```
make: PIC: 没有那个文件或目录
make: *** 没有规则可制作目标“PIC”。 停止。
```

英文报错信息：

```
make: PIC: No such file or directory
make: *** No rule to make target 'PIC'.  Stop.
```

说明我这里的问题应该不是由 `-fPIC` 引起的。

后来由查了一些资料，错误原因应当是我当前的 Linux 版本默认启用了 `PIE`，但是 makefile 里的这些库（指上面没有编译成功的那部分库）不支持 PIE，所以导致了上面的错误。至此问题已定位。

解决方法就是上面说过的，在编译时禁用 `PIE` 即可，添加参数 `-no-pie`。