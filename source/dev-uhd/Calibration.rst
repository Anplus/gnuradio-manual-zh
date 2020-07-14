Self-Calibration
====================

UHD提供了软件矫正IQ不平衡和直流偏差的工具。偏差的结果被保存在用户home路径下的一个CSV文件。
当用户重新启动UHD,GnuRadio会先读取矫正文件。

完成自矫正，首先需要断开所有USRP射频连接。
.. code:: bash
    uhd_cal_rx_iq_balance: - minimizes RX IQ imbalance vs. LO frequency
    uhd_cal_tx_dc_offset: - minimizes TX DC offset vs. LO frequency
    uhd_cal_tx_iq_balance: - minimizes TX IQ imbalance vs. LO frequency