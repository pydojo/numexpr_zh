==================
安装 Numexpr
==================

在 Unix 系统上安装 Numexpr 要知道一些指令。
对于 Windows 系统来说，最好就是从二进制文件来安装。
不管如何做到的，你应该注意，项目正在进行中，
我们不能提供含有 MKL 支持的 Windows 二进制安装包。


建造
========

本 `Numexpr` 版本需要 Python 2.6 以上的版本，
以及 NumPy 1.6 以上的版本。

用标准的 Python 方法来建造，命令行中输入::

  $ python setup.py build
  $ python setup.py install

安装完毕，你可以测试一下 `numexpr` 库，命令行中输入::

  $ python -c "import numexpr; numexpr.test()"


开启英特尔公司的 MKL 支持
============================

numexpr 库支持英特尔公司的 MKL 库。这就让你在
英特尔架构上的性能获得更好效果，主要是在评估超前
函数上（三角学公式，幂运算，等等多层化数学计算公式）。
同时也开启了 numexpr 库使用多核 CPU 的利用率。

如果你的电脑中已经安装了 Intel's MKL 库的话，
只要复制分发包中的 `site.cfg.example` 文件内容到
 `site.cfg` 文件中，并且编辑稍后给出的正确指导即可。
正确指导是关于如何找到你电脑中所安装的 MKL 库的位置。
完成这些，你可以继续使用上面的常用建造指令来安装了。

注意，在建造过程中的消息内容，这样你会知道是否检测到 MKL 库。
最后，你可以检查你的电脑运算加速效果，
通过运行 `bench/vml_timing.py` 脚步方式
（你可以用不同的参数值来与
`set_vml_accuracy_mode()` 和 `set_vml_num_threads()` 
函数玩一会儿，在脚本中你会看到有不同的性能效果）。



.. Local Variables:
.. mode: text
.. coding: utf-8
.. fill-column: 70
.. End:
