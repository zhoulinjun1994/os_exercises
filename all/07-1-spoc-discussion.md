# 同步互斥(lec 17) spoc 思考题


- 有"spoc"标记的题是要求拿清华学分的同学要在实体课上完成，并按时提交到学生对应的ucore_code和os_exercises的git repo上。

## 个人思考题

### 背景
 - 请给出程序正确性的定义或解释。
 - 在一个新运行环境中程序行为与原来的预期不一致，是错误吗？
 - 程序并发执行有什么好处和障碍？
 - 什么是原子操作？

### 现实生活中的同步问题

 - 家庭采购中的同步问题与操作系统中进程同步有什么区别？
 - 如何通过枚举和分类方法检查同步算法的正确性？
 - 尝试描述方案四的正确性。
 - 互斥、死锁和饥饿的定义是什么？

### 临界区和禁用硬件中断同步方法

 - 什么是临界区？
 - 临界区的访问规则是什么？
 - 禁用中断是如何实现对临界区的访问控制的？有什么优缺点？

### 基于软件的同步方法

 - 尝试通过枚举和分类方法检查Peterson算法的正确性。
 - 尝试准确描述Eisenberg同步算法，并通过枚举和分类方法检查其正确性。

### 高级抽象的同步方法

 - 如何证明TS指令和交换指令的等价性？
 - 为什么硬件原子操作指令能简化同步算法的实现？
 
## 小组思考题

1. （spoc）阅读[简化x86计算机模拟器的使用说明](https://github.com/chyyuu/ucore_lab/blob/master/related_info/lab7/lab7-spoc-exercise.md)，理解基于简化x86计算机的汇编代码。

>

略。

2. （spoc)了解race condition. 进入[race-condition代码目录](https://github.com/chyyuu/ucore_lab/tree/master/related_info/lab7/race-condition)。

 - 执行 `./x86.py -p loop.s -t 1 -i 100 -R dx`， 请问`dx`的值是什么？
 - 执行 `./x86.py -p loop.s -t 2 -i 100 -a dx=3,dx=3 -R dx` ， 请问`dx`的值是什么？
 - 执行 `./x86.py -p loop.s -t 2 -i 3 -r -a dx=3,dx=3 -R dx`， 请问`dx`的值是什么？
 - 变量x的内存地址为2000, `./x86.py -p looping-race-nolock.s -t 1 -M 2000`, 请问变量x的值是什么？
 - 变量x的内存地址为2000, `./x86.py -p looping-race-nolock.s -t 2 -a bx=3 -M 2000`, 请问变量x的值是什么？为何每个线程要循环3次？
 - 变量x的内存地址为2000, `./x86.py -p looping-race-nolock.s -t 2 -M 2000 -i 4 -r -s 0`， 请问变量x的值是什么？
 - 变量x的内存地址为2000, `./x86.py -p looping-race-nolock.s -t 2 -M 2000 -i 4 -r -s 1`， 请问变量x的值是什么？
 - 变量x的内存地址为2000, `./x86.py -p looping-race-nolock.s -t 2 -M 2000 -i 4 -r -s 2`， 请问变量x的值是什么？ 
 - 变量x的内存地址为2000, `./x86.py -p looping-race-nolock.s -a bx=1 -t 2 -M 2000 -i 1`， 请问变量x的值是什么？ 

>

```
./x86.py -p loop.s -t 1 -i 100 -R dx：

dx          Thread 0
0
-1   1000 sub  $1,%dx
-1   1001 test $0,%dx
-1   1002 jgte .top
-1   1003 halt

./x86.py -p loop.s -t 2 -i 100 -a dx=3,dx=3 -R dx：

dx          Thread 0                Thread 1
3
2   1000 sub  $1,%dx
2   1001 test $0,%dx
2   1002 jgte .top
1   1000 sub  $1,%dx
1   1001 test $0,%dx
1   1002 jgte .top
0   1000 sub  $1,%dx
0   1001 test $0,%dx
0   1002 jgte .top
-1   1000 sub  $1,%dx
-1   1001 test $0,%dx
-1   1002 jgte .top
-1   1003 halt
3   ----- Halt;Switch -----  ----- Halt;Switch -----
2                            1000 sub  $1,%dx
2                            1001 test $0,%dx
2                            1002 jgte .top
1                            1000 sub  $1,%dx
1                            1001 test $0,%dx
1                            1002 jgte .top
0                            1000 sub  $1,%dx
0                            1001 test $0,%dx
0                            1002 jgte .top
-1                            1000 sub  $1,%dx
-1                            1001 test $0,%dx
-1                            1002 jgte .top
-1                            1003 halt

./x86.py -p loop.s -t 2 -i 3 -r -a dx=3,dx=3 -R dx。

dx          Thread 0                Thread 1
3
2   1000 sub  $1,%dx
2   1001 test $0,%dx
2   1002 jgte .top
3   ------ Interrupt ------  ------ Interrupt ------
2                            1000 sub  $1,%dx
2                            1001 test $0,%dx
2   ------ Interrupt ------  ------ Interrupt ------
1   1000 sub  $1,%dx
1   1001 test $0,%dx
1   1002 jgte .top
2   ------ Interrupt ------  ------ Interrupt ------
2                            1002 jgte .top
1                            1000 sub  $1,%dx
1                            1001 test $0,%dx
1   ------ Interrupt ------  ------ Interrupt ------
0   1000 sub  $1,%dx
0   1001 test $0,%dx
1   ------ Interrupt ------  ------ Interrupt ------
1                            1002 jgte .top
0                            1000 sub  $1,%dx
0                            1001 test $0,%dx
0   ------ Interrupt ------  ------ Interrupt ------
0   1002 jgte .top
-1   1000 sub  $1,%dx
0   ------ Interrupt ------  ------ Interrupt ------
0                            1002 jgte .top
-1   ------ Interrupt ------  ------ Interrupt ------
-1   1001 test $0,%dx
0   ------ Interrupt ------  ------ Interrupt ------
-1                            1000 sub  $1,%dx
-1                            1001 test $0,%dx
-1   ------ Interrupt ------  ------ Interrupt ------
-1   1002 jgte .top
-1   1003 halt
-1   ----- Halt;Switch -----  ----- Halt;Switch -----
-1                            1002 jgte .top
-1   ------ Interrupt ------  ------ Interrupt ------
-1                            1003 halt

./x86.py -c -p looping-race-nolock.s -t 1 -M 2000 。

2000          Thread 0
0
0   1000 mov 2000, %ax
0   1001 add $1, %ax
1   1002 mov %ax, 2000
1   1003 sub  $1, %bx
1   1004 test $0, %bx
1   1005 jgt .top
1   1006 halt

./x86.py -p looping-race-nolock.s -t 2 -a bx=3 -M 2000 ，循环3次的目的是便于和之前作对比：

2000          Thread 0                Thread 1
0
0   1000 mov 2000, %ax
0   1001 add $1, %ax
1   1002 mov %ax, 2000
1   1003 sub  $1, %bx
1   1004 test $0, %bx
1   1005 jgt .top
1   1000 mov 2000, %ax
1   1001 add $1, %ax
2   1002 mov %ax, 2000
2   1003 sub  $1, %bx
2   1004 test $0, %bx
2   1005 jgt .top
2   1000 mov 2000, %ax
2   1001 add $1, %ax
3   1002 mov %ax, 2000
3   1003 sub  $1, %bx
3   1004 test $0, %bx
3   1005 jgt .top
3   1006 halt
3   ----- Halt;Switch -----  ----- Halt;Switch -----
3                            1000 mov 2000, %ax
3                            1001 add $1, %ax
4                            1002 mov %ax, 2000
4                            1003 sub  $1, %bx
4                            1004 test $0, %bx
4                            1005 jgt .top
4                            1000 mov 2000, %ax
4                            1001 add $1, %ax
5                            1002 mov %ax, 2000
5                            1003 sub  $1, %bx
5                            1004 test $0, %bx
5                            1005 jgt .top
5                            1000 mov 2000, %ax
5                            1001 add $1, %ax
6                            1002 mov %ax, 2000
6                            1003 sub  $1, %bx
6                            1004 test $0, %bx
6                            1005 jgt .top
6                            1006 halt

./x86.py -p looping-race-nolock.s -t 2 -M 2000 -i 4 -r -s 0：

2000          Thread 0                Thread 1
0
0   1000 mov 2000, %ax
0   1001 add $1, %ax
1   1002 mov %ax, 2000
1   1003 sub  $1, %bx
1   ------ Interrupt ------  ------ Interrupt ------
1                            1000 mov 2000, %ax
1   ------ Interrupt ------  ------ Interrupt ------
1   1004 test $0, %bx
1   1005 jgt .top
1   1006 halt
1   ----- Halt;Switch -----  ----- Halt;Switch -----
1   ------ Interrupt ------  ------ Interrupt ------
1                            1001 add $1, %ax
1   ------ Interrupt ------  ------ Interrupt ------
2                            1002 mov %ax, 2000
2                            1003 sub  $1, %bx
2   ------ Interrupt ------  ------ Interrupt ------
2                            1004 test $0, %bx
2   ------ Interrupt ------  ------ Interrupt ------
2                            1005 jgt .top
2   ------ Interrupt ------  ------ Interrupt ------
2                            1006 halt

./x86.py -c -p looping-race-nolock.s -t 2 -M 2000 -i 4 -r -s 1
：

2000          Thread 0                Thread 1
0
0   1000 mov 2000, %ax
0   1001 add $1, %ax
1   1002 mov %ax, 2000
1   1003 sub  $1, %bx
1   ------ Interrupt ------  ------ Interrupt ------
1                            1000 mov 2000, %ax
1                            1001 add $1, %ax
2                            1002 mov %ax, 2000
2   ------ Interrupt ------  ------ Interrupt ------
2   1004 test $0, %bx
2   1005 jgt .top
2   ------ Interrupt ------  ------ Interrupt ------
2                            1003 sub  $1, %bx
2   ------ Interrupt ------  ------ Interrupt ------
2   1006 halt
2   ----- Halt;Switch -----  ----- Halt;Switch -----
2                            1004 test $0, %bx
2   ------ Interrupt ------  ------ Interrupt ------
2                            1005 jgt .top
2                            1006 halt

 ./x86.py -c -p looping-race-nolock.s -t 2 -M 2000 -i 4 -r -s 2 ：

2000          Thread 0                Thread 1
0
0   1000 mov 2000, %ax
0   ------ Interrupt ------  ------ Interrupt ------
0                            1000 mov 2000, %ax
0                            1001 add $1, %ax
0   ------ Interrupt ------  ------ Interrupt ------
0   1001 add $1, %ax
0   ------ Interrupt ------  ------ Interrupt ------
1                            1002 mov %ax, 2000
1   ------ Interrupt ------  ------ Interrupt ------
1   1002 mov %ax, 2000
1   ------ Interrupt ------  ------ Interrupt ------
1                            1003 sub  $1, %bx
1                            1004 test $0, %bx
1                            1005 jgt .top
1   ------ Interrupt ------  ------ Interrupt ------
1   1003 sub  $1, %bx
1   1004 test $0, %bx
1   1005 jgt .top
1   ------ Interrupt ------  ------ Interrupt ------
1                            1006 halt
1   ----- Halt;Switch -----  ----- Halt;Switch -----
1   1006 halt

./x86.py -c -p looping-race-nolock.s -a bx=1 -t 2 -M 2000 -i 1：

2000          Thread 0                Thread 1
0
0   1000 mov 2000, %ax
0   ------ Interrupt ------  ------ Interrupt ------
0                            1000 mov 2000, %ax
0   ------ Interrupt ------  ------ Interrupt ------
0   1001 add $1, %ax
0   ------ Interrupt ------  ------ Interrupt ------
0                            1001 add $1, %ax
0   ------ Interrupt ------  ------ Interrupt ------
1   1002 mov %ax, 2000
1   ------ Interrupt ------  ------ Interrupt ------
1                            1002 mov %ax, 2000
1   ------ Interrupt ------  ------ Interrupt ------
1   1003 sub  $1, %bx
1   ------ Interrupt ------  ------ Interrupt ------
1                            1003 sub  $1, %bx
1   ------ Interrupt ------  ------ Interrupt ------
1   1004 test $0, %bx
1   ------ Interrupt ------  ------ Interrupt ------
1                            1004 test $0, %bx
1   ------ Interrupt ------  ------ Interrupt ------
1   1005 jgt .top
1   ------ Interrupt ------  ------ Interrupt ------
1                            1005 jgt .top
1   ------ Interrupt ------  ------ Interrupt ------
1   1006 halt
1   ----- Halt;Switch -----  ----- Halt;Switch -----
1   ------ Interrupt ------  ------ Interrupt ------
1                            1006 halt

结论：进程切换的时候通用寄存器是要恢复为上一次的值，内存则不用，因此，如果值是保存在寄存器中，进程切换不会影响其值，但如果值保存在内存中，则会基于上次进程执行完毕后的数值继续运行。
```

3. （spoc） 了解software-based lock, hardware-based lock, [software-hardware-lock代码目录](https://github.com/chyyuu/ucore_lab/tree/master/related_info/lab7/software-hardware-locks)

  - 理解flag.s,peterson.s,test-and-set.s,ticket.s,test-and-test-and-set.s 请通过x86.py分析这些代码是否实现了锁机制？请给出你的实验过程和结论说明。能否设计新的硬件原子操作指令Compare-And-Swap,Fetch-And-Add？
```
Compare-And-Swap

int CompareAndSwap(int *ptr, int expected, int new) {
  int actual = *ptr;
  if (actual == expected)
    *ptr = new;
  return actual;
}
```

```
Fetch-And-Add

int FetchAndAdd(int *ptr) {
  int old = *ptr;
  *ptr = old + 1;
  return old;
}
```

>

flag.s不可以，注意到第11行有一个赋值操作，而判断操作在赋值操作前，那么，如果进程0刚好运行到赋值操作前被打断，进程1发现未上锁，于是它进入锁内执行，然而这时进程2由于已经执行过判断也进入了锁，不满足互斥要求，自旋锁设计失败。
