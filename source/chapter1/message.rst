
message passing
=================

Introduction
-------------
GNURadio的最初设计是为了处理数据流，比特和采样点为基本的处理单位。为了传输控制信息，元数据，包结构，Gnuradio引入了标签流。
标签流和数据流是并行处理的。标签流是为了存储metadata，控制信息的。标签和采样点是关联的，和数据流一起传输。这个模型使得模块可以识别一些特殊的事件，采取一定的措施。
缺点是：标签流只能单向流动，而且只能在模块的work函数访问。优点是，标签流和数据流是等同步的。

设计这个机制的两个目的是：

- 下行的模块可以回传数据给上行模块
- 外部程序可以用这个接口和GNURADIO通信

这个模块严重依赖多态类型实现（PMT）。

Message Passing API
----------------------
message passing的接口在gr::basic_block中得到了实现，gr::basic_block是所有block的父类。
每个block都有一个消息队列可以存储消息，并向下传输消息。
而且可以区分输入和输出端口。

端口是在构造器中声明的：

.. code:: c++

    void message_port_register_in(pmt::pmt_t port_id)
    void message_port_register_out(pmt::pmt_t port_id)

每个接口有一个端口id。其他block要和这个端口通信，接收或者发送message，必须订阅这个端口。subscribe的API如下：

.. code:: python

    void message_port_pub(pmt::pmt_t port_id,pmt::pmt_t msg);
    void message_port_sub(pmt::pmt_t port_id,pmt::pmt_t target);
    void message_port_unsub(pmt::pmt_t port_id,pmt::pmt_t target);

任何block订阅了另一个block的输出端口后，会在发布消息的时候收到消息。
在一个block内部，当他要发布消息的时候，他会向每个订阅了他输出的端口的block的消息队列发送消息。


Message Handler Functions
---------------------------
订阅了block的消息的端口后， 必须声明一个处理方法。 
利用gr::basic_block::message_port_register_in订阅了一个端口后，我们必须把这个端口绑定到一个消息处理器上。
这个部分利用的是boost的bind函数：

.. code:: c++

    set_msg_handler(pmt::pmt_t port_id, 
                    boost::bind(&block_class::message_handler_function, this, _1));

- 'port_id' 是输入的端口id
- 'block_class::message_handler_function'是处理这个端口消息的函数
- this和_1是boost绑定函数的

.. code:: c++

    void block_class::message_handler_function(pmt::pmt_t msg);

Connecting Messages through the Flowgraph
---------------------------------------------


这个机制的接口是独立于数据流的，所以创建模块的时候不需要


Code Examples
----------------------------------
下面利用gr::blocks::message_debug 和 gr::blocks::tagged_stream_to_pdu 。
gr::blocks::message_debug模块是用来调试消息传递的模块。有三个输入口：

- print，打印所有信息到标准输出流。
- store，把消息存储到list里，和gr::blocks::message_debug::get_message(int i)连接，取出第i个消息。
- pdu_print，把PDU消息转化成标准流。

.. code:: c++

    {
        message_port_register_in(pmt::mp("print"));
        set_msg_handler(pmt::mp("print"),
        boost::bind(&message_debug_impl::print, this, _1));
        message_port_register_in(pmt::mp("store"));
        set_msg_handler(pmt::mp("store"),
        boost::bind(&message_debug_impl::store, this, _1));
        message_port_register_in(pmt::mp("print_pdu"));
        set_msg_handler(pmt::mp("print_pdu"),
        boost::bind(&message_debug_impl::print_pdu, this, _1));
    }