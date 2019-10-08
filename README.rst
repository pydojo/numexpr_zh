======================================================
NumExpr: 为 NumPy 加速数学表达式评估器
======================================================

:Author: David M. Cooke, Francesc Alted and others
:Contact: faltet@gmail.com
:URL: https://github.com/pydata/numexpr
:Documentation: http://numexpr.readthedocs.io/en/latest/
:Travis CI: |travis|
:Appveyor: |appveyor|
:PyPi: |version|
:DOI: |doi|
:readthedocs: |docs|

.. |travis| image:: https://travis-ci.org/pydata/numexpr.png?branch=master
        :target: https://travis-ci.org/pydata/numexpr
.. |appveyor| image:: https://ci.appveyor.com/api/projects/status/we2ff01vqlmlb9ip
        :target: https://ci.appveyor.com/project/robbmcleod/numexpr
.. |docs| image:: https://readthedocs.org/projects/numexpr/badge/?version=latest
        :target: http://numexpr.readthedocs.io/en/latest
.. |doi| image:: https://zenodo.org/badge/doi/10.5281/zenodo.2483274.svg
        :target:  https://doi.org/10.5281/zenodo.2483274
.. |version| image:: https://img.shields.io/pypi/v/numexpr.png
        :target: https://pypi.python.org/pypi/numexpr


什么是 NumExpr ？
----------------

NumExpr 是一个加速数学表达式评估器的 NumPy 加速库。
使用这个加速库，让你的表达式在阵列操作上有显著的提升
（例如 :code:`'3*a+4*b'`），并且使用更少的内存，
这是与同样的计算在 Python 中对比而来。

另外，具备多线程能力，这让你可以充分利用多核处理器资源，
通用中与 NumPy 相比有明显的性能标量提升。

不止于此， numexpr 可以使用 Intel's VML 库（向量数学库，
正常情况集成在了英特尔的数学内核库里，即 MKL）。
这会进一步加速超前表达式的运算。


NumExpr 是如何实现高性能的？
-------------------------------------

主要原因是 NumExpr 避免了分配内存导致的中间效应。
这也是与 NumPy 技高一筹的地方。这种结果在缓存利用率上
达到了更好的效果，并且减少了内存访问。通用中由于此特效，
 NumExpr 最适合与大数据一起工作。

NumExpr 对表达式的语法分析是自身的操作代码实现的，
这些操作代码都会集成到虚拟计算计中。阵列操作都分解成
皮卡式操作，这样容易满足 CPU 的缓存，并且都传递给虚拟机。
虚拟机再再每个皮卡上应用这些操作。表达式中的所有短器和
持续性操作也都因此成为皮卡式操作，这个价值带来的就是性能提升。
这些皮卡式操作都分布在 CPU 多核上使用，结果就是高平行代码执行效率。

 NumExpr 的结果就是获得你的电脑最大计算能力给智能阵列计算。
同样的 NumPy 计算速度通常在 0.95X 左右（例如非常简单的表达式
:code:`'a + 1'`），以及速度 4x (是相对多层化的表达式
:code:`'a*b-4.1*a > 2.5*b'`），NumPy 更高的速度可以在
一些函数和多层化数学操作上实现（某些环境下提升到 15x ）。

NumExpr 性能最佳体现在矩阵计算上，因为矩阵太大了，难以满足 L1 CPU 缓存。
为了得到更好的性能，许多不同的速度可以实现在你的操作系统上，
运行提供的标杆可以得知。


用法
-----

::

  >>> import numpy as np
  >>> import numexpr as ne

  >>> a = np.arange(1e6)   # Choose large arrays for better speedups
  >>> b = np.arange(1e6)

  >>> ne.evaluate("a + 1")   # a simple expression
  array([  1.00000000e+00,   2.00000000e+00,   3.00000000e+00, ...,
           9.99998000e+05,   9.99999000e+05,   1.00000000e+06])

  >>> ne.evaluate('a*b-4.1*a > 2.5*b')   # a more complex one
  array([False, False, False, ...,  True,  True,  True], dtype=bool)

  >>> ne.evaluate("sin(a) + arcsinh(a/b)")   # you can also use functions
  array([        NaN,  1.72284457,  1.79067101, ...,  1.09567006,
          0.17523598, -0.09597844])

  >>> s = np.array(['abba', 'abbb', 'abbcdef'])
  >>> ne.evaluate("'abba' == s")   # string arrays are supported too
  array([ True, False, False], dtype=bool)


文档
-------------

请阅读官方文档 `numexpr.readthedocs.io <https://numexpr.readthedocs.io>`_
其中包含了一份用户指导、标杆结果，和 API 手册。


作者
-------

阅读 `AUTHORS.txt <https://github.com/pydata/numexpr/blob/master/AUTHORS.txt>`_


协议
-------

NumExpr 库是建立在 `MIT <http://www.opensource.org/licenses/mit-license.php>`_ 协议之下进行分发。


.. Local Variables:
.. mode: text
.. coding: utf-8
.. fill-column: 70
.. End:
