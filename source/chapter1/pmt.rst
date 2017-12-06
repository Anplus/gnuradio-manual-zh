
多态类型
===========

介绍
------
Polymorphic Types(多态)是一种高级的数据类型，被设计成通用类型用来在block和thread之间传递数据。
在stream tags和message passed接口中使用的很多。下面由一段python代码看pmt的使用方法：

.. code:: python

    >>> import pmt
    >>> P = pmt.from_long(23)
    >>> type(P)
    <class 'pmt.pmt_swig.swig_int_ptr'>
    >>> print P
    23
    >>> P2 = pmt.from_complex(1j)
    >>> type(P2)
    <class 'pmt.pmt_swig.swig_int_ptr'>
    >>> print P2
    0+1i
    >>> pmt.is_complex(P2)
    True

我们利用from_long和from_complex导入了一个长整数和一个复数。但是他们的类型是一样的，都是pmt。
这样我们就可以把这些变量利用swig传入C++。
同样C++代码如下：

.. code:: C++

    #include <pmt/pmt.h>
    // [...]
    pmt::pmt_t P = pmt::from_long(23);
    std::cout << P << std::endl;
    pmt::pmt_t P2 = pmt::from_complex(gr_complex(0, 1)); // Alternatively: pmt::from_complex(0, 1)
    std::cout << P2 << std::endl;
    std::cout << pmt::is_complex(P2) << std::endl;

有两个特点在C++和python都很重要。首先，我们可以很容易的打印pmt的内容。PMT内置了把值转化成string的方法（某些类型的数据不行）。
而且，PMT必须显式的知道他们的类型，所以我们可以查询他们的类型，比如调用is_complex方法。

non-PMT和PMT的转化使用 from_x和to_x方法。

.. code:: python

    pmt::pmt_t P_int = pmt::from_long(42);
    int i = pmt::to_long(P_int);
    pmt::pmt_t P_double = pmt::from_double(0.2);
    double d = pmt::to_double(P_double);

string是一个比较特殊的类型，他的转化是特殊的方法。

.. code:: python

    pmt::pmt_t P_str = pmt::string_to_symbol("spam");
    pmt::pmt_t P_str2 = pmt::intern("spam");
    std::string 
    str = pmt::symbol_to_string(P_str);

pmt::intern是symbol_to_string的另外一种方法。

在python中，我们可以使用弱类型。