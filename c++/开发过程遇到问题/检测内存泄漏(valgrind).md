使用工具valgrind

## c程序

首先看一段 C 程序示例，比如：

```c++
#include <stdlib.h>
int main()
{
    int *array = malloc(sizeof(int));
    return 0;
}
```

编译程序：`gcc -g -o main main.c`，比哪一需要加上 `-g` 选项打开调试，使用 IDE 的可以用 Debug 模式编译。

使用 Valgrind 检测内存使用情况：

```
valgrind --tool=memcheck --leak-check=full ./main
```

结果：

```c++
==31416== Memcheck, a memory error detector
==31416== Copyright (C) 2002-2017, and GNU GPL'd, by Julian Seward et al.
==31416== Using Valgrind-3.13.0 and LibVEX; rerun with -h for copyright info
==31416== Command: ./main_c
==31416==
==31416==
==31416== HEAP SUMMARY:
==31416==     in use at exit: 4 bytes in 1 blocks
==31416==   total heap usage: 1 allocs, 0 frees, 4 bytes allocated
==31416==
==31416== 4 bytes in 1 blocks are definitely lost in loss record 1 of 1
==31416==    at 0x4C2DBF6: malloc (vg_replace_malloc.c:299)
==31416==    by 0x400537: main (main.c:5)
==31416==
==31416== LEAK SUMMARY:
==31416==    definitely lost: 4 bytes in 1 blocks
==31416==    indirectly lost: 0 bytes in 0 blocks
==31416==      possibly lost: 0 bytes in 0 blocks
==31416==    still reachable: 0 bytes in 0 blocks
==31416==         suppressed: 0 bytes in 0 blocks
==31416==
==31416== For counts of detected and suppressed errors, rerun with: -v
==31416== ERROR SUMMARY: 1 errors from 1 contexts (suppressed: 0 from 0)
```

先看看输出信息中的 `HEAP SUMMARY`，它表示程序在堆上分配内存的情况，其中的 `1 allocs` 表示程序分配了 1 次内存，`0 frees` 表示程序释放了 0 次内存，`4 bytes allocated` 表示分配了 4 个字节的内存。

另外，Valgrind 也会报告程序是在哪个位置发生内存泄漏。例如，从下面的信息可以看到，程序发生了一次内存泄漏，位置是 `main.c` 文件的第 5 行：

```
==31416== 4 bytes in 1 blocks are definitely lost in loss record 1 of 1
==31416==    at 0x4C2DBF6: malloc (vg_replace_malloc.c:299)
==31416==    by 0x400537: main (main.c:5)
```

## c++程序

对于 C++ 程序一样如此：

```
#include <string>
int main()
{
    auto ptr = new std::string("Hello, World!");
    delete ptr;
    return 0;
}
valgrind --tool=memcheck --leak-check=full --show-leak-kinds=all ./main
```

结果：

```c++
==31438== Memcheck, a memory error detector
==31438== Copyright (C) 2002-2017, and GNU GPL'd, by Julian Seward et al.
==31438== Using Valgrind-3.13.0 and LibVEX; rerun with -h for copyright info
==31438== Command: ./main_cpp
==31438==
==31438==
==31438== HEAP SUMMARY:
==31438==     in use at exit: 72,704 bytes in 1 blocks
==31438==   total heap usage: 2 allocs, 1 frees, 72,736 bytes allocated
==31438==
==31438== 72,704 bytes in 1 blocks are still reachable in loss record 1 of 1
==31438==    at 0x4C2DBF6: malloc (vg_replace_malloc.c:299)
==31438==    by 0x4EC3EFF: ??? (in /usr/lib/x86_64-linux-gnu/libstdc++.so.6.0.21)
==31438==    by 0x40104E9: call_init.part.0 (dl-init.c:72)
==31438==    by 0x40105FA: call_init (dl-init.c:30)
==31438==    by 0x40105FA: _dl_init (dl-init.c:120)
==31438==    by 0x4000CF9: ??? (in /lib/x86_64-linux-gnu/ld-2.23.so)
==31438==
==31438== LEAK SUMMARY:
==31438==    definitely lost: 0 bytes in 0 blocks
==31438==    indirectly lost: 0 bytes in 0 blocks
==31438==      possibly lost: 0 bytes in 0 blocks
==31438==    still reachable: 72,704 bytes in 1 blocks
==31438==         suppressed: 0 bytes in 0 blocks
==31438==
==31438== For counts of detected and suppressed errors, rerun with: -v
==31438== ERROR SUMMARY: 0 errors from 0 contexts (suppressed: 0 from 0)
```

使用 Valgrind 分析 C++ 程序时，有一些问题需要留意。例如，这个程序并没有发生内存泄漏，但是从`HEAP SUMMARY` 可以看到，程序分配了 2 次内存，但却只释放了 1 次内存，为什么会这样呢？实际上这是由于 C++ 在分配内存时，为了提高效率，使用了它自己的内存池。当程序终止时，内存池的内存才会被操作系统回收，所以 Valgrind 会将这部分内存报告为 `reachable` 的，需要注意，**`reachable` 的内存不代表内存泄漏**，例如，从上面的输出中可以看到，有 72704 个字节是 `reachable` 的，但没有报告内存泄漏。

不仅如此，Valgrind 还能进行越界访问检测，内存未初始化检测等等。