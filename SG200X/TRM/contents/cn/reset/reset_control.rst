复位控制
--------

芯片内部存在三个层级的复位管理模块对整个芯片、子系统，各个功能模块的复位进行管理。

.. _diagram_reset_block:
.. figure:: ../../../../media/image6.png
	:align: center

	复位管理模块框图

Reset Ctrl Level 1 电路负责系统上电复位功能。上电复位 (POR) 由实时时钟模块配合全局电源管理与晶振时序生成。详情参照章节 :ref:`section_rtc`。来源由以下途径：

-  上电复位

-  过热保护复位

-  看门狗复位。RCT_CTRL0.hw_wdg_rst_en 为 1, (见 :ref:`table_rtc_ctrl0`) 且 sys_ctrl_reg.reg_sw_root_reset_en 的 bit[0] 为 0 (见 :ref:`table_sys_ctrl_reg`) 情况下 Watchdog 定时器超时触发系统复位。

Reset Ctrl Level 2 电路负责产生系统硬复位 (System Hard Reset)，对芯片全局包含子系统及各功能模块进行硬复位，来源由以下途径：

-  看门狗复位, sys_ctrl_reg.reg_sw_root_reset_en 的 bit[0] 为 1 (见 :ref:`table_sys_ctrl_reg`) 情况下 Watchdog 定时器超时触发系统复位。

-  外部复位管脚 (RSTN)，内建去抖动电路 (Debounce), RSTN 高低电平有效信号须达 6.56ms。

Reset Ctrl Level 3 电路负责提供实现软复位控制相应的复位配置寄存器 (Reset CRG)，具体参考 :ref:`section_reset_configure_registers`。包含：

-  系统软复位 : 复位全芯片，除少部分电路及 RTC 内部电路。

-  处理器子系统复位 : 复位处理器及处理器子系统。操作寄存器 SOFT_CPUAC_RSTN 可对处理器及子系统做软复位，配置寄存器写 0 后，复位控制器会等待 24us 延时后才触发相应处理器复位。这段期间处理器应结束对总线之访问，以避免复位后总线挂死。触发复位后对应之复位信号会持续 8us 后自动解除，处理器及处理器子系统完成复位并开始启动。

-  功能子系统及功能模块复位: 复位各功能子系统及功能模块。操作寄存器 SOFT_RSTN_0 ~ 3，可对各功能模块进行软复位。复位配置为低电平有效，复位信号并不会自动清除，故软件配置相应寄存器为 0 触发复位后，亦需配置为 1 解除复位。复位前须确保各功能模块及功能子系统内置 DMA 对总线访问与处理器对模块之访问处于闲置状态。否则将使复位失败易造成系统挂死。
