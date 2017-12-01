
GNURadio的几个例子
====================

这些例子安装在$prefix/share/doc/gnuradio-$version/。

Dial Tone
------------

使用两种模块

* gr::analog::sig_source_f，生成 :math:`350Hz,440Hz`。
* gr::audio::sink，将输出连接到语音系统，以采样率生成输出信号。audio sink可以设置两个输出。

.. code:: c

    sig_source_f (freq = 350) -->
                              audio.sink
    sig_source_f (freq = 440) -->

dial_tone.py文件：

.. code:: python

    from gnuradio import gr
    from gnuradio import audio
    from gnuradio.eng_option import eng_option
    from optparse import OptionParser

    try:
        from gnuradio import analog
    except ImportError:
        sys.stderr.write("Error: Program requires gr-analog.\n")
        sys.exit(1)

    class my_top_block(gr.top_block):

        def __init__(self):
            gr.top_block.__init__(self)

            parser = OptionParser(option_class=eng_option)
            parser.add_option("-O", "--audio-output", type="string", default="",
                            help="pcm output device name.  E.g., hw:0,0 or /dev/dsp")
            parser.add_option("-r", "--sample-rate", type="eng_float", default=48000,
                            help="set sample rate to RATE (48000)")
            (options, args) = parser.parse_args()
            if len(args) != 0:
                parser.print_help()
                raise SystemExit, 1

            sample_rate = int(options.sample_rate)
            ampl = 0.1

            src0 = analog.sig_source_f(sample_rate, analog.GR_SIN_WAVE, 350, ampl)
            src1 = analog.sig_source_f(sample_rate, analog.GR_SIN_WAVE, 440, ampl)
            dst = audio.sink(sample_rate, options.audio_output)
            self.connect(src0, (dst, 0))
            self.connect(src1, (dst, 1))

    if __name__ == '__main__':
        try:
            my_top_block().run()
        except KeyboardInterrupt:
            pass

FM Modulator
--------------

这个例子用GRC或者GRC和Python完成。我们用GRC生成一个FM信号，然后用GRC程序或者Python程序解码。

Modulator
~~~~~~~~~~~

首先用控制台命令 "gnuradio-companion"运行GRC。用图形接口创建我们的流程图。这里先不详细介绍GRC接口，仅仅利用GRC程序生成数据。
GRC运行之后，打开“fm_tx.grc”。流程图中，生成了电话音频率，求和，重采样，以便我们可以通过整数倍的上采样得到宽带FM信号，然后输出。

实际例子只是为了生成FM电话音信号，保存到文件中。不需要文件太大，只要能说明问题即可。

* gr::blocks::head，模块限制进入文件的采样数量。一旦有了N个采样点，流程图就被终止了。
* gr::blocks::skiphead忽略最初的M个采样点，避免滤波器的跳变和群延时。

运行程序，可以使用"Build->Execute"，或者点击工具栏上的齿轮。程序运行一会儿，一旦控制台显示了"nitems"参数设置的item数量。
在一个"dummy.dat"文件中有FM的复数采样点。

Demodulator
~~~~~~~~~~~~~~
GRC程序"fm_rx.grc"，对应着python脚本，"fm_demod.py"。

* gr::blocks::file_source，读取文件 
* gr::analog::quadrature_demod_cf，把文件复数的FM信号转换成浮点信号。

我们把200bps输入信号重采样到44.1kbps(语音速率)。因为这个重采样不能使用整数倍的抽取速率，所以我们使用了任意速率重采样器，
gr::filter::pfb_arb_resampler_fff。将输出的信号以 :math:`44.1Khz`的采样率滤波到 :math:`15Khz`的带宽，然后利用
gr::audio::sink 输出。