---
title: 使用多线程加速快速排序
date: 2016-03-11 21:22:53
categories:
- programming
tags:
- c
- apue
- Unix
- multi-thread
---

一般来说，使用qsort能得到很快的速度，这也是它称之为快速排序的原因。但是在一些特定的情况下，可以继续特高快排的速度，今天这种情况就是：

- 需要排序的数量特别大，如对百万级别的数据排序
- 硬件能支持多线程（多核）

如果使用的是多核的机器，可能利用起多核计算的优势来加速排序，思路比较简单，就是将一个大的数组切分成和与机器逻辑线程数量相等的小数组再进行排序（小数组的个数也可以略多，但是过多的切分会导致频繁的线程调度）。下面的代码主要来自APUE 第11章线程屏障部分的示例代码，我只简单修改了一部分并添加上注释便于理解。
<!-- more -->

```c
#include <stdio.h>
#include <stdlib.h>
#include <pthread.h>
#include <limits.h>
#include <sys/time.h>

// 排序线程数 8
#define NTHR 8

// 数组大小为8000000（8百万）
#define NUMNUM 8000000L

// 每个小数组的大小（小数组其实仍是基于大数组分配的内存），注意宏定义的括号
#define TNUM (NUMNUM / NTHR)

// 多线程共享的数据，用于排序的数组
long nums[NUMNUM];
// 用于排序的数组的备份，用于qsort排序，比较两种排序的性能差异
long qnums[NUMNUM];

// 用于保存多线程排序的结果
long sortNums[NUMNUM];

// 同步多个排序线程的屏障（barrier）
pthread_barrier_t bt;

// 用于qsort的比较函数
int complong(const void *arg1, const void *arg2) {
  long l1 = *(long *)arg1;	// 先将void *指针类型转换为 long * 类型指针，再解引用
  long l2 = *(long *)arg2;
  if (l1 > l2) return 1;
  else if (l1 < l2) return -1;
  else return 0;
}

// 排序线程执行的函数，线程函数签名为： void *fn(void *arg)
// 通过 arg传递此线程要排序的起始位置
void *thread_sort(void *arg) {
  long numIdx = (long)arg;
  qsort(&nums[numIdx], TNUM, sizeof(long), complong);

  // 本线程执行完了，在屏障处等待其它线程执行结束 
  pthread_barrier_wait(&bt);

  return (void *)0;
}

// 合并排序好的子数组，这部分和归并排序类似，要好好理解
void merge(void) {
   long idxs[NTHR];
   int i;
   for (i=0; i<NTHR; i++) {
      idxs[i] = i * TNUM;
   }

   int  sIdx;
   for (sIdx=0; sIdx<NUMNUM; sIdx++) {
      long min = LONG_MAX;
      int curMinIdx = 0;
      for (i=0; i<NTHR; i++) {
        if ((idxs[i] < (i+1) * TNUM) && (nums[idxs[i]] < min)) {
        curMinIdx = i;
        min = nums[idxs[i]];
        }
      }
      sortNums[sIdx] = nums[idxs[curMinIdx]];
      idxs[curMinIdx]++;
   }
}

int main(void) {
   unsigned long i;
  // 保存时间，通过 gettimeofday 可以获得一个精度很高的时间戳（微秒级）
  struct timeval start, end;
  long long startusec, endusec;
  int err;
  double duration;

  pthread_t tid;

  // 生产随机数
  srandom((unsigned)time(NULL));
  for (i=0; i<NUMNUM; i++) {
    nums[i] = rand();
    qnums[i] = nums[i];
  }

  // 初始化屏障，注意屏障等待的数量为NTHR+1，这是主线程也要等待
  pthread_barrier_init(&bt, NULL, NTHR+1);
  gettimeofday(&start, NULL);

  // 分成小数组排序
  for (i=0; i<NTHR; i++) {
    err = pthread_create(&tid, NULL, thread_sort, (void *)(i * TNUM));
    if (err != 0) {
      printf("create thread error\n");
      return -1;
    }
  }
  // 主线程等待
  pthread_barrier_wait(&bt);

  // 所有排序线程执行完，开始合并结果
  merge();
  gettimeofday(&end, NULL);

  // 计算使用的时间， 属性前缀 tv 是 time value的缩写，sec是second的缩写，usec是微秒的缩写
  startusec = start.tv_sec * 1000000 + start.tv_usec;
  endusec = end.tv_sec * 1000000 + end.tv_usec;
  duration = (double)(endusec - startusec) / 1000000.0;

  printf("time used for sort %d random array: %f\n", NUMNUM, duration);
  printf("start with: %ld, end with: %ld\n", sortNums[0], sortNums[NUMNUM-1]);

  // 计算同样的数组，直接使用qsort的效率，注意传入qsort的数组不能是nums了，因为nums已经部分有序了
  // 传入nums的备份，qnums
  gettimeofday(&start, NULL);
  qsort(qnums, NUMNUM, sizeof (long), complong);
  gettimeofday(&end, NULL);
  startusec = start.tv_sec * 1000000 + start.tv_usec;
  endusec = end.tv_sec * 1000000 + end.tv_usec;
  printf("time cost using qsort: %f\n", (double)(endusec - startusec)/1000000.0);
}
```

通过测试，在一台配置了双核的虚拟机上执行时，使用多线程排序比直接使用qsort快了一倍（gcc 编译时可能需要手动链接一下pthread）。

其结果如下：

```
$ gcc -o multi_thread_sort -lpthread multi_thread_sort.c
$ ./multi_thread_sort
time used for sort 8000000 random array: 2.546938
start with: 1, end with: 2147483605
time cost using qsort: 5.367226
```

参考：

- Unix环境高级编程（APUE）
- [http://www.tutorialspoint.com/c_standard_library/c_function_rand.htm](http://www.tutorialspoint.com/c_standard_library/c_function_rand.htm)