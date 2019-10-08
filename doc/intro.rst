NumExpr 是如何工作的？
============

把字符串代入到 :code:`evaluate` 里编译成一个对象来表示表达式，
然后阵列的类型通过 :code:`numexpr` 函数来使用。

使用 Python 的 :code:`compile` 时先编译表达式（这意味着
表达式要是合法的 Python 表达式）。这样才能得到变量名。然后
使用一个特殊对象的实例完成表达式评估，这个特殊对象保存了对表达式
所做的追踪信息，并且对表达式建立了一棵语法分析树。

然后把这棵语法分析树编译成一个字节编码的程序，该程序描述如何执行
智能元素操作。虚拟机使用“向量注册器”：
每个注册器有许多元素宽（默认是 4096 个元素）。
对于 NumExpr 速度的关键是一次性处理这些元素的皮卡形式。

评估一个表达式的智能元素时，有2个极端。
你可以把每个操作当成阵列来执行，返回临时阵列。
这就是你使用 NumPy 时所做的：
:code:`2*a+3*b` 使用了三个临时阵列同规模操作在 :code:`a` 或 :code:`b` 上。
这种策略导致内存浪费（如果你的阵列太大就有问题），
并且用不好缓存：对大型阵列来说，
:code:`2*a` 和 :code:`3*b` 不会在缓存中做加法。

另一个极端就是要迭代每一个元素，如同使用 `for` 循环语句一样::

    for i in xrange(len(a)):
        c[i] = 2*a[i] + 3*b[i]

这不会消耗额外的内存，并且用好了缓存，但是
如果表达式没有编译成机器代码，你会有一个大问题
（或者一堆问题），那就是在循环内部对每个元素
增加了大量过量操作，并且会伤害到用在 CPU 上的分支预测语句。

:code:`numexpr` 使用了平衡术方法。阵列都处理成皮卡
（4096 个元素一组）一次处理一组，使用了一个注册器机制。
使用 Python 时代码看起来像下面一样::

    for i in xrange(0, len(a), 256):
       r0 = a[i:i+128]
       r1 = b[i:i+128]
       multiply(r0, 2, r2)
       multiply(r1, 3, r3)
       add(r2, r3, r2)
       c[i:i+128] = r2

（记住，三段式把结果存储在第三个参数中，而不是分配一个新阵列）。
这种实现在缓存和分支预测之间得到了良好地平衡。
并且虚拟机是完全用 C 语言写的，这让上面的 Python 代码运行的更快了。
进一步来说，虚拟机也是多线程的，所以让 NumPy 的平行计算有了效率。

更多信息和历史记录在如下地址中：

http://www.bitsofbits.com/2014/09/21/numpy-micro-optimization-and-numexpr/

期望的性能
====================

NumExpr 的加速范围尊重 NumPy 库，可以介于 0.95x 和 20x 之间，
典型情况是 2x, 3x 或 4x ，与表达式的多层化程度和操作符使用的内部优化有关。
在阵列出现步幅和不整齐的情况中，也已经优化完成，所以如果表达式含有这类阵列，
加速可以有足够的提升。当然，你会需要与大型阵列一起操作（典型来说，一个阵列
大于你的 CPU 缓存大小）才能明白性能提升的意义。

这里又许多实时例子，连续阵列情况::

    In [1]: import numpy as np
    In [2]: import numexpr as ne
    In [3]: a = np.random.rand(1e6)
    In [4]: b = np.random.rand(1e6)
    In [5]: timeit 2*a + 3*b
    10 loops, best of 3: 18.9 ms per loop
    In [6]: timeit ne.evaluate("2*a + 3*b")
    100 loops, best of 3: 5.83 ms per loop   # 3.2x: medium speed-up (simple expr)
    In [7]: timeit 2*a + b**10
    10 loops, best of 3: 158 ms per loop
    In [8]: timeit ne.evaluate("2*a + b**10")
    100 loops, best of 3: 7.59 ms per loop   # 20x: large speed-up due to optimised pow()

不整齐阵列情况，提速甚至更大::

    In [9]: a = np.empty(1e6, dtype="b1,f8")['f1']
    In [10]: b = np.empty(1e6, dtype="b1,f8")['f1']
    In [11]: a.flags.aligned, b.flags.aligned
    Out[11]: (False, False)
    In [12]: a[:] = np.random.rand(len(a))
    In [13]: b[:] = np.random.rand(len(b))
    In [14]: timeit 2*a + 3*b
    10 loops, best of 3: 29.5 ms per loop
    In [15]: timeit ne.evaluate("2*a + 3*b")
    100 loops, best of 3: 7.46 ms per loop   # ~ 4x speed-up
