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


general_work()和work()
---------------------------

general_work()和work()是block工作的核心函数，数据流的操作都在这里完成。


.. code-block:: cpp

    int block::general_work(int noutput_items,
        gr_vector_int &ninput_items,
        gr_vector_const_void_star &input_items,
        gr_vector_void_star &output_items)


input_items 是一个vector包含一组指针指向input buffer。output_items 是一个vector包含一组指针指向output buffer。general_work不指定输入输出的关系，只是指定输入和输出的数量。noutput_items是最小的output数量。ninput_items是input buffer。


.. code-block:: cpp

    int block::work(int noutput_items, 
        gr_vector_const_void_star &input_items,
        gr_vector_void_star &output_items)


work函数指定了input和output的关系。通过noutput_items确定ninput_items。


Scheduler block
---------------------
调度器会处理buffer，block的需求，消息和stream tags。blocks有几个需求：alignment，output multiple，forecast，history。

.. image:: ../fig/scheduler-4.png

* alignment: 将输出对齐到一定倍数，不一定保证
* output multiple：将输出对齐到一定倍数，保证实现。如不满足会等待。
* forecast：ninput_items_required[i]告诉对于每个输出需要多少输入。
* history：set_history()，scheduler控制了buffer的长度。如果我们将history设置为N，那么buffer里的前N个数据中的N-1个数据为历史数据（即使你已经用过了）。history保证了buffer里至少有N-1个数据。

.. image:: ../fig/scheduler-history.png

当我们给定输出的数据数量noutput_items，那么我们可以计算输入数据量ninput_items_required[i]：

.. code-block:: cpp

    //forecast()
    ninput_items_required[i]=noutput_items+history()-1; // default
    ninput_items_required[i]=noutput_items*decimation()+history()-1; // Decim
    ninput_items_required[i]=noutput_items/interpolation()+history()-1; // Interp


经过这样的forecast设置，可以保证输入满足输出的需求。

**Buffer and Controlling flow and latency**


.. code-block:: cpp

    // Caps the maximum noutput_items.
    // Will round down to nearest output multiple, if set.
    // Does not change the size of any buffers.
    set_max_noutput_items(int)
    // Sets the maximum buer size for all output buers.
    // Buffer calculations are based on a number of factors, this limits overall size.
    // On most systems, will round to nearest page size.
    set_max_output_buffer(long)
    // Sets the minimum buer size for all output buers.
    // On most systems, will round to nearest page size.
    set_min_output_buffer(long)

 **Scheduler manages the Data stream Condition**

 * 计算input有多少可用的点
 * 计算output有多空间
 * 确定限制条件: history, alignment, forecast
 * call general_work，给block恰当的指针和数据
 * 从general_work的返回值更新指针


Scheduler Flow Chart
---------------------------

调度器会为每个模块创建一个线程。