Multiple USRP Connection and Configuration
==============================================

Network Connection
--------------------
- 如果有两张网卡，可以将两张网卡配置在不同网段，然后两个USRP分别配置在两个网段里，然后在一个gnuradio脚本里直接访问。
- 如果多台PC用交换机访问每个USRP，需要将多张网卡配置在一个网段里，还要指定每个网卡的UHD通信端口，
端口不能冲突，否则UHD只能找到一个设备。





MIMO  Wire Connection
-----------------------


Tuning Notes
====================
Two-stage tuning process
--------------------------
一个USRP设备有两级变音：

- RF front-end:从射频前端到中频
- DSP：中频到基频

