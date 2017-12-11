
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
SBX子板的DC控制
----------------
SBX子板的自动DC控制，在直接做ASK调制的时候，有长连1的时候，
SBX子板会将长1调整成直流分量，从而使得ASK调制在接受端看起来是相反的。

.. code:: python

    self.source.set_auto_dc_offset(False)


SBX全双工
--------

SBX子板有两个天线：TX/RX和RX2，但是子板上只有一个transmit和一个receive。
- 做全双工的时候，必须由RX2做接收，而且收发的工作频率可以不一致
- 不支持两路都接收