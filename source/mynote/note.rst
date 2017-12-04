
类型转换
================

* ishort_to_complex模块：interleaved_short_to_complex ，交叉short类型转complex


控制接口
=========

为GNURadio创建分布式应用。这个模块使得内部Blocks可以连接到一个输出流，远程画图；
以及输出可以被设置，监听，绘制的变量。模块在ctrlport命名空间内，可以被如下方式访问。

.. code:: python

    from gnuradio import ctrlport

注意
==========

SBX子板的自动DC控制，在直接做ASK调制的时候，有长连1的时候，
SBX子板会将长1调整成直流分量，从而使得ASK调制在接受端看起来是相反的。
