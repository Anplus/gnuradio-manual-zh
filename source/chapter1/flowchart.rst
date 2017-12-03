
处理流图
=============

操作流程图
-----------

GNURadio基本的结构就是flowchart。flowchart是有向无环图，有一个或者多个source(输入采样点)，一个或者多个sin(输出采样点或者终止)。

每个程序必须至少创建一个'top_blcok'作为flowchart的顶层结构。这个结构提供了很多全局的方法，'start,' 'stop,' and 'wait'。

GNU Radio的应用创建gr_top_block来实例化blocks，连接模块，然后开始gr_top_block。下面给出一个FIR滤波器的例子。

.. code:: python

    from gnuradio import gr, blocks, filter, analog
    class my_topblock(gr.top_block):
        def __init__(self):
            gr.top_block.__init__(self)
            amp = 1
            taps = filter.firdes.low_pass(1, 1, 0.1, 0.01)
            self.src = analog.noise_source_c(analog.GR_GAUSSIAN, amp)
            self.flt = filter.fir_filter_ccf(1, taps)
            self.snk = blocks.null_sink(gr.sizeof_gr_complex)
            self.connect(self.src, self.flt, self.snk)
    if __name__ == "__main__":
        tb = my_topblock()
        tb.start()
        tb.wait()

'tb.start()'开始了数据流流过flowchart，'tb.wait()'等价于知道gr_top_block结束后等待线程'join'。
可以利用 'run' 方法来替换这两个方法的先后调用。

延迟和吞吐量
-------------
GNU Radio运行一个调度器来优化吞吐量。动态调度器使得成块的数据通过blocks从source流到sink。数据块的大小和信号处理的速度有关。
对于每个block，能够处理的数据量和输出buffer的空间和输入buffer中已经收到的数据量有关。

这样操作的结果就是，一个模块可能申请了很多数据来处理(规模可能到几千个采样点)。
从速度的角度来看，这样使得大部分的处理时间都用在了处理数据使得系统更有效率。
申请小的数据块就意味着会向调度器多次申请。
这样做的副作用就是当block处理大量数据的时候会出现延迟。

为了解决这个问题，gr_top_block可以限制block可以收到的数据量，也就是上一个block的需要输出的数据量。
一个block只能得到比这个数量少的采样点输入，所以相当于一个block的最大延迟。
通过限制每次调用申请的数据量，我们相当于增加了调度器的负担，所以降低了全局效率。

可以按照如下方式限制输出数据量：

.. code:: python

    tb.start(1000)
    tb.wait()
    # or
    tb.run(1000)

使用这个方法，我们设置了一个全局的item数量限制。每个block可以通过'set_max_noutput_items(m)'，重写这个限制。

.. code:: python

    tb.flt.set_max_noutput_items(2000)
    tb.run(1000)

在一些情况下，可能想要限制输出缓存的大小。这个能够防止要输出的数据量过大超过了上限，而使得新的输出延迟。
你可以为每个block的每个输出端口设置输出延迟。

.. code:: python

    tb.blk0.set_max_output_buffer(2000)
    tb.blk1.set_max_output_buffer(1, 2000)
    tb.start()
    print tb.blk1.max_output_buffer(0)
    print tb.blk1.max_output_buffer(1)



动态配置流程图
--------------
在通信系统运行的时候，经常需要根据输入信号改变系统的状态，这时候需要更新流图。更新流图有三步：
* 锁定，停止运行，处理数据
* 更新
* 解锁

