
Flowchart
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

上面的接口blk0所有端口被设置成缓存为2000个items，而blk1只有端口1被设置了，其余为默认值。

注意：

* 在运行的开始，缓存的大小就被配置好了。
* 一旦flowgraph开始，缓存长度对于一个block的值是不能被更改的，即使是lock()/unlock()。如果要改变缓存大小，必须删除block再重新建立。
* 这可能影响到吞吐量。
* 真实的缓存大小实际上是依赖于最小系统粒度。理论上就是一个页的大小，通常是4096bytes。这就意味着，由指令设置的缓存大小最终会四舍五入到最接近的系统粒度上。

动态配置流程图
--------------
在通信系统运行的时候，经常需要根据输入信号改变系统的状态，这时候需要更新流图。更新意味着改变结构，不独立的参数设置。
例如， gr::blocks::add_const_cc中改变加的常量大小可以由调用'set_k(k)'完成。

更新流图有三步：

* 锁定，停止运行，处理数据
* 更新
* 解锁

下面的例子展示了一个流图，首先加入两个gr::analog::noise_source_c，然后由gr::blocks::sub_cc替代gr::blocks::add_cc。

.. code:: python

    from gnuradio import gr, analog, blocks
    import time
    class mytb(gr.top_block):
        def __init__(self):
            gr.top_block.__init__(self)
            self.src0 = analog.noise_source_c(analog.GR_GAUSSIAN, 1)
            self.src1 = analog.noise_source_c(analog.GR_GAUSSIAN, 1)
            self.add = blocks.add_cc()
            self.sub = blocks.sub_cc()
            self.head = blocks.head(gr.sizeof_gr_complex, 1000000)
            self.snk = blocks.file_sink(gr.sizeof_gr_complex, "output.32fc")
            self.connect(self.src0, (self.add,0))
            self.connect(self.src1, (self.add,1))
            self.connect(self.add, self.head)
            self.connect(self.head, self.snk)
        def main():
            tb = mytb()
            tb.start()
            time.sleep(0.01)
            # Stop flowgraph and disconnect the add block
            tb.lock()
            tb.disconnect(tb.add, tb.head)
            tb.disconnect(tb.src0, (tb.add,0))
            tb.disconnect(tb.src1, (tb.add,1))
            # Connect the sub block and restart
            tb.connect(tb.sub, tb.head)
            tb.connect(tb.src0, (tb.sub,0))
            tb.connect(tb.src1, (tb.sub,1))
            tb.unlock()
            tb.wait()
        if __name__ == "__main__":
            main()

在更新flowchart的时候，最大输出items数量也可以被更改。一个block也可以调用'unset_max_noutput_items()' 来解锁限制恢复到全局值。
下面的例子扩展了上面的例子，增加了设置最大输出items数量。

.. code:: python

    from gnuradio import gr, analog, blocks
    import time
    class mytb(gr.top_block):
        def __init__(self):
            gr.top_block.__init__(self)
            self.src0 = analog.noise_source_c(analog.GR_GAUSSIAN, 1)
            self.src1 = analog.noise_source_c(analog.GR_GAUSSIAN, 1)
            self.add = blocks.add_cc()
            self.sub = blocks.sub_cc()
            self.head = blocks.head(gr.sizeof_gr_complex, 1000000)
            self.snk = blocks.file_sink(gr.sizeof_gr_complex, "output.32fc")
            self.connect(self.src0, (self.add,0))
            self.connect(self.src1, (self.add,1))
            self.connect(self.add, self.head)
            self.connect(self.head, self.snk)
        def main():
            # Start the gr_top_block after setting some max noutput_items.
            tb = mytb()
            tb.src1.set_max_noutput_items(2000)
            tb.start(100)
            time.sleep(0.01)
            # Stop flowgraph and disconnect the add block
            tb.lock()
            tb.disconnect(tb.add, tb.head)
            tb.disconnect(tb.src0, (tb.add,0))
            tb.disconnect(tb.src1, (tb.add,1))
            # Connect the sub block
            tb.connect(tb.sub, tb.head)
            tb.connect(tb.src0, (tb.sub,0))
            tb.connect(tb.src1, (tb.sub,1))
            # Set new max_noutput_items for the gr_top_block
            # and unset the local value for src1
            tb.set_max_noutput_items(1000)
            tb.src1.unset_max_noutput_items()
            tb.unlock()
            tb.wait()
        if __name__ == "__main__":
            main()