# java.util.Collections 排序原理

> 版本：JDK7

直接看一下Collections排序中的一个具体实现。

这是sort(List<T>)的方法调用栈。

![](/imgs/java-base/jb-2-1.png)

```java
public static <T extends Comparable<? super T>> void sort(List<T> list) {
    // 将list转换成Object数组
    Object[] a = list.toArray();
    // 调用Arrays.sort 对数组a进行排序
    Arrays.sort(a);
    // 获取list的迭代器
    ListIterator<T> i = list.listIterator();
    // 遍历list，将排序之后的数组a里面的元素以此复制到list中
    for (int j=0; j<a.length; j++) {
        i.next();
        i.set((T)a[j]);
    }
}
```

从上面sort方法的声明里面可以看出，要想调用Collections.sort(List<T>)的List需要满足几个条件：

1. 容器必须是List的实现类。
1. 容器内的元素必须继承或者实现Comparable接口。

Collections中的 **sort** 方法实际上调用了 **Arrays.sort**。

下面来看一下Arrays.sort的具体代码实现。

```java
public static void sort(Object[] a) {
    // 默认是不启用遗留的归并排序
    if (LegacyMergeSort.userRequested)
        legacyMergeSort(a);
    else
        ComparableTimSort.sort(a);
}
```

**LegacyMergeSort** 是Arrays的一个静态内部类，只有一个属性 `userRequested`，默认是 **false**。

```java
static final class LegacyMergeSort {
    private static final boolean userRequested =
        java.security.AccessController.doPrivileged(
            new sun.security.action.GetBooleanAction(
                "java.util.Arrays.useLegacyMergeSort")).booleanValue();
}
```

Arrays.sort内部又调用了 **ComparableTimSort.sort** 这才是排序的真正实现。

```java
static void sort(Object[] a) {
        sort(a, 0, a.length);
}

/**
 * @param a, 要排序的数组
 * @param lo, 数组的起始下标
 * @param hi, 数组的结束下标
 */
static void sort(Object[] a, int lo, int hi) {

    // 检查数组：
    // 1. 起始下标位置是否大于结束下标位置，如果是，抛出异常。
    // 2. 起始下标位置是否小于0，如果是，抛出异常。
    // 3. 结束下标位置是否大于数组长度，如果是抛出异常。
    rangeCheck(a.length, lo, hi);

    // 计算剩余的排序个数
    int nRemaining  = hi - lo;

    // 如果剩余的排序个数小于2，说明数组里面最多只有1个元素，这个时候就不需要进行排序。
    if (nRemaining < 2)
        return;

    // 如果剩余的排序个数，小于32，则进行“mini-TimSort”
    if (nRemaining < MIN_MERGE) {

        // 获取指定数组中指定位置开始的运行的长度，听起来很绕，先看下面，后面再来看看这个方法具体做什么
        int initRunLen = countRunAndMakeAscending(a, lo, hi);
        // 进行二分插入排序
        binarySort(a, lo, hi, lo + initRunLen);
        return;
    }

    // 当剩余的排序个数大于等于32个的时候，进行TimSort
    ComparableTimSort ts = new ComparableTimSort(a);
    
    int minRun = minRunLength(nRemaining);
    do {
        int runLen = countRunAndMakeAscending(a, lo, hi);
        if (runLen < minRun) {
            int force = nRemaining <= minRun ? nRemaining : minRun;
            binarySort(a, lo, lo + force, lo + runLen);
            runLen = force;
        }

        ts.pushRun(lo, runLen);
        ts.mergeCollapse();

        lo += runLen;
        nRemaining -= runLen;
    } while (nRemaining != 0);

    assert lo == hi;
    ts.mergeForceCollapse();
    assert ts.stackSize == 1;
}
```

ComparableTimSort.sort 里面涉及了两种排序：

1. binarySort，其实是一种插入排序。
1. TimSort，是结合了合并排序（merge sort）和插入排序（insertion sort）而得出的排序算法。

##   参考

-   [wiki - Timsort](https://en.wikipedia.org/wiki/Timsort)

## [Back](../../summary.md)