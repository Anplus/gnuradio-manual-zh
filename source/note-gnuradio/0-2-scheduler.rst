GNURadio scheduler
======================

GNURadio的scheduler是数据流调度的核心。这个部分的文档很少，本文主要参考 gnuradio-note_ 的slides，以及笔者的一些源码阅读。

.. _gnuradio-note: http://www.trondeau.com/blog/2013/9/15/explaining-the-gnu-radio-scheduler.html

首先看一个例子。两个数据流经过同步模块，再经过十倍欠采样模块，最后输出。

.. image:: ../fig/scheduler-1.png

每个input和output会维护一个buffer。output利用Wptr指针写数据，input利用Rptr指针读取数据。

.. image:: ../fig/scheduler-2.png

Decimator需要足够的输入来计算输出。

.. image:: ../fig/scheduler-3.png


general_work和work
---------------------

·· code-block:: cpp

    int block::general_work(int noutput_items,
        gr_vector_int &ninput_items,
        gr_vector_const_void_star &input_items,
        gr_vector_void_star &output_items)

input_items 是一个vector包含一组指针指向input buffer。output_items 是一个vector包含一组指针指向output buffer。general_work不指定输入输出的关系，只是指定输入和输出的数量。noutput_items是最小的output数量。ninput_items是input buffer。

·· code-block:: cpp

    int block::work(int noutput_items, 
        gr_vector_const_void_star &input_items,
        gr_vector_void_star &output_items)

work函数指定了input和output的关系。通过noutput_items确定ninput_items。


Scheduler
---------------------
调度器会处理buffer，block的需求，消息和stream tags。blocks有几个需求：alignment，output multiple，forecast，history。

.. image:: ../fig/scheduler-4.png

* alignment: 将输出对齐到一定倍数，不一定保证
* output multiple：将输出对齐到一定倍数，保证实现。如不满足会等待。
* forecast：告诉对于每个输出需要多少输入。
* history：set_history(nitems+1)，设置了读取指针的位置，确保我们可以写多于noutput_items的更多数据。

