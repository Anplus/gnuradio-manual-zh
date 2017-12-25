
USRP环境配置
==============

这里主要介绍使用USRP平台，上位机是ubuntu的环境配置。从容易到难列出配置环境的几种方式，最后针对几种不同的软件无线电平台给出特有的步骤。

SDR Live
----------

最简单的方式，利用官方的SDR Live镜像，直接安装配有完整环境的系统盘。需要注意的是，这样安装的系统只能在线读写，每次启动都会复原。

https://wiki.gnuradio.org/index.php/GNU_Radio_Live_SDR_Environment

使用自动化脚本
-------------

这种方式也很方便。前人写好的自动化脚本完成了从源代码的下载，编译和安装几个步骤。过程很慢，要有心理准备。

.. code:: bash

    wget http://www.sbrac.org/files/build-gnuradio && chmod a+x ./build-gnuradio && ./build-gnuradio

其中有可能遇到几种问题：

* 路径访问权限报错。

.. code:: bash

    chmod: changing permissions of './build-gnuradio': Operation not permitted

需要在一个有完全访问权限的路径使用脚本。

* 依赖的包找不到，笔者遇到了两个。

.. code:: bash

    Failed to find package 'libzmq1-dev' in known package repositories
    Failed to find package 'python-wxgtk2.8' in known package repositorie

wkgtk是wx-gui的包，如果这里没有装成功，之后wx-gui的组件就不能使用了，如果使用了wx-gui，会报出如下错误。

.. code:: bash

    wxgui-python
    TypeError: Error when calling the metaclass bases
        multiple bases have instance lay-out conflict

之所以找不到这个包是因为ubuntu16.04不再有这个包了，是wkgtk3.0。
这里首先获取apt源的列表，安装历史版本2.8。
关于libzmq1-dev这个包没有找到合适的解决方案，但是这个不影响使用。

.. code::bash

    echo "deb http://archive.ubuntu.com/ubuntu wily main universe" | sudo tee /etc/apt/sources.list.d/wily-copies.list
    sudo apt update
    sudo apt install python-wxgtk2.8
    sudo rm /etc/apt/sources.list.d/wily-copies.list
    sudo apt update

Pybombs
----------

Gnuradio推荐的方式是利用官方发布的pybombs工具安装gnuradio及其依赖。
但是实际操作的时候，pybombs的bug还是很多，这里并不推荐。

E310/E312的环境配置
------------------------------

* 需要重新针对E310/312构建UHD驱动。

https://kb.ettus.com/Software_Development_on_the_E310_and_E312

.. code:: bash

    git clone https://github.com/EttusResearch/uhd.git
    cd ./host
    mkdir build 
    cmake ./ -DENABLE_E300=ON
    make install -j8    

-j8是为了使用多核，速度会快些。

* Remote login E312

.. code:: bash

    ssh root@192.168.10.10

* 切换网络模式

.. code:: bash

    usrp_e3x0_network_mode

* 开启另一个终端，查找设备。

.. code:: bash

    uhd_find_devices --args="addr=192.168.10.10"

如果上面的构建失败就会出现

.. code:: bash

    No UHD Devices Found

FPGA版本不兼容
~~~~~~~~~~~~~~~
E31x系列比较烦人的是内部有一个linux系统，也要配置环境。
如果内部系统用的FPGA版本和外部控制电脑不一致，虽然UHD驱动仍然可以找到设备，调试的时候就会报错。

.. code:: bash

    RuntimeError: RuntimeError: Expected FPGA compatibility number 16.x, but got 14.0:
    The FPGA build is not compatible with the host code build.
    Please run:

        "/usr/local/lib/uhd/utils/uhd_images_downloader.py"

当然按照他给的方案，直接下载uhd镜像是肯定不行的。用UHD工具查看FPGA版本。

.. code:: bash

    uhd_find_devices 
    linux; GNU C++ version 5.4.0 20160609; Boost_105800; UHD_003.010.002.000-3-g122bfae1

    --------------------------------------------------
    -- UHD Device 0
    --------------------------------------------------
    Device Address:
        type: e3x0
        addr: 192.168.10.10
        name: 
        serial: 30CCCC1
        product: 30675

E310,E312,E313的FPGA的硬件版本都是e3x0。

* 方案一，降低本机的UHD版本

注意到本机的UHD版本是3.11.1，与E312内部的版本不同。
这里选择将本机的UHD版本降低到与E312一致，这时候运行程序的时候会出现GnuRadio Companion的UHD组件冲突，需要重新编译GnuRadio。

.. code:: bash

    _uhd.swig: undefined symbol: _ZN3uhd4usrp10multi_usrp7ALL_LOSB5cxx11E

再利用E312的测试程序。

.. code:: bash

    rx_ascii_art_dft ­­--freq 88.1e6 ­­--rate 400e3 ­­--gain 30 ­­--ref­-lvl ­-30

* 方案二，升级USRP的UHD版本。

- 首先格式化SD卡

http://www.arthurtoday.com/2013/10/ubuntu-mkdosfs-format-sd-card.html

.. code:: bash

    sudo umount /dev/sdb1
    sudo mkdosfs -F 32 -v /dev/sdb1

Read more: http://www.arthurtoday.com/2013/10/ubuntu-mkdosfs-format-sd-card.html#ixzz52HCPfTVQ  

- 然后写入SD卡的镜像文件

.. code:: bash

    sudo dd if=sdimage-gnuradio-dev.direct of=/dev/<yoursdcard> bs=1M

<yoursdcard>可以用fdisk -l或者df看到。

- 配置ip信息
USB串口进入设备，在设备内更新网络配置文件。

.. code:: bash

    cd etc/network
    vi interface

在auto eth0后面加入

.. code:: bash  

    iface eth0 inet static
        address 192.168.10.10
        network 255.255.255.0
        gateway 192.168.10.1



BladeRF环境配置
================

BladeRF有详细的官方windows教程，很难做错，这里就毋庸赘言了。主要介绍BladeRF在ubuntu的环境配置。

同样官方给了easy安装版本。
https://github.com/Nuand/bladeRF/wiki/Getting-Started:-Linux#Easy_installation_for_Ubuntu_The_bladeRF_PPA

遇到的问题可能有：

* FPGA not laoded

.. code:: bash

    bladeRF-cli -i
    bladeRF> info

    Serial #:                 19a3df66ec02993409cd516b0ef169ff
    VCTCXO DAC calibration:   0x925f
    FPGA size:                115 KLE
    FPGA loaded:              no
    USB bus:                  3
    USB address:              2
    USB speed:                SuperSpeed
    Backend:                  libusb
    Instance:                 0

从BladeRF官网下载对应的FPGA镜像 https://www.nuand.com/fpga.php 。

.. code:: bash

    $ bladeRF-cli -L hostedx115-latest.rbf
    Writing FPGA to flash for autoloading...
    [INFO @ usb.c:498] Erasing 55 blocks starting at block 4
    ...
    [INFO @ usb.c:617] Done reading 13952 pages
    Done.
    $ bladeRF-cli -l hostedx115-latest.rbf
    Loading fpga...
    Done.
