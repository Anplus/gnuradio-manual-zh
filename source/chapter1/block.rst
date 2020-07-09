模块类型
=============

概述
------------


利用gnuradio的框架，用户可以创建多种类型的模块来实现特定的数据处理。

- Synchronous Blocks (1:1)
- Decimation Blocks (N:1)
- Interpolation Blocks (1:M)
- Basic (a.k.a. General) Blocks (N:M)

Synchronous Blocks (1:1)
---------------------------

同步模块一个端口的的消耗和输出的点数量是一样的。零输入的同步模块叫做source，零输出的同步模块叫做sink。

.. code-block:: cpp

    #include <gr_sync_block.h> 
    class my_sync_block : public gr_sync_block
    {
        public:
        my_sync_block(...):
            gr_sync_block("my block", 
                        gr_make_io_signature(1, 1, sizeof(int32_t)),
                        gr_make_io_signature(1, 1, sizeof(int32_t)))
        {
            //constructor stuff
        }

        int work(int noutput_items,
                gr_vector_const_void_star &input_items,
                gr_vector_void_star &output_items)
        {
            //work stuff...
            return noutput_items;
        }
    };

- noutput_items是输入和输出的缓冲区大小
- 输入的签名gr_make_io_signature(0, 0, 0)使得模块为source 
- 输出的签名gr_make_io_signature(0, 0, 0)使得模块为sink

gr_make_io_signature(int min_streams, int max_streams, int sizeof_stream_item)控制了接口的数量，以及接口的数据大小。下面给出了python的模块样例：

.. code-block:: python

    class my_sync_block(gr.sync_block):
        def __init__(self):
            gr.sync_block.__init__(self,
                name = "my sync block",
                in_sig = [numpy.float32, numpy.float32],
                out_sig = [numpy.float32],
            )
        def work(self, input_items, output_items):
            output_items[0][:] = input_items[0] + input_items[1]
            return len(output_items[0])

input_items和output_items是包含列表的列表。input_items的每个端口有一个输入采样点向量。output_items也是一个向量可以将输出点存起来。output_items[0]的长度等于C++中noutput_items。

- in_sig=None的时候，模块为source
- out_sig=None的时候，模块为sink。这时候要用len(input_items[0])
- 不像C++中的gr::io_signature类，python可以直接创建指定数据类型的list


Basic Block
--------------


.. code-block::cpp

    #include <gr_block.h>

    class my_basic_block : public gr_block
    {
    public:
    my_basic_adder_block(...):
        gr_block("another adder block",
                in_sig,
                out_sig)
    {
        //constructor stuff
    }

    int general_work(int noutput_items,
                    gr_vector_int &ninput_items,
                    gr_vector_const_void_star &input_items,
                    gr_vector_void_star &output_items)
    {
        //cast buffers
        const float* in0 = reinterpret_cast(input_items[0]);
        const float* in1 = reinterpret_cast(input_items[1]);
        float* out = reinterpret_cast(output_items[0]);

        //process data
        for(size_t i = 0; i < noutput_items; i++) {
        out[i] = in0[i] + in1[i];
        }

        //consume the inputs
        this->consume(0, noutput_items); //consume port 0 input
        this->consume(1, noutput_items); //consume port 1 input
        //this->consume_each(noutput_items); //or shortcut to consume on all inputs

        //return produced
        return noutput_items;
    }
    };