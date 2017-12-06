
Metadata Information
=====================

Introduction
-------------

元数据文件在文件头有额外的元数据存储着有关采样点类型的信息。
原始文件，二进制文件不带有任何额外信息。所以这类文件必须被特殊处理。
系统中的任何改变，比如采样率或者接受机的频率都没有在文件中体现，元数据的文件头解决了这类问题。

我们利用gr::blocks::file_meta_sink写入元数据文件，利用gr::blocks::file_meta_source读取元数据文件。

元数据文件的文件头描述了一个数据分片的信息。比如，item size，数据类型(comples)，采样率，首个采样点的时间戳，
文件头的大小和分片大小。

第一个静态区保存着：

* version: (char) version number (usually set to METADATA_VERSION)
* rx_rate: (double) Stream's sample rate
* rx_time: (pmt::pmt_t pair - (uint64_t, double)) Time stamp (format from UHD)
* size: (int) item size in bytes - reflects vector length if any.
* type: (int) data type (enum below)
* cplx: (bool) true if data is complex
* strt: (uint64_t) start of data relative to current header
* bytes: (uint64_t) size of following data segment in bytes

额外的分局存储在每一个收到的tags里。

* rx_rate: the sample rate of the stream.
* rx_time: the time stamp of the first item in the segment.

在一个文件中的数据类型是不会变得。因为GNU Radio的block只能在构造函数的IO signature设置数据额类型，所以之后的数据类型改变不会被接受。

元数据文件的类型
~~~~~~~~~~~~~~~~~

GNU Radio支持两种：

* inline：headers和数据在同一行
* detached：headers在一个单独的header file里

inline是标准的方法。如果使用detached方法，headers简单地插入到detached header file；数据文件是标准的无中断的原始二进制格式。

更新headers
~~~~~~~~~~~~~

实现
~~~~~

结构
------

Header Information
~~~~~~~~~~~~~~~~~~~~~~

额外信息
~~~~~~~~~

使用
------

例子
-------
例子在
* gr-blocks/examples/metadata
GRC例子

* file_metadata_sink: create a metadata file from UHD samples.
* file_metadata_source: read the metadata file as input to a simple graph.
* file_metadata_vector_sink: create a metadata file from UHD samples.
* file_metadata_vector_source: read the metadata file as input to a simple graph.



