第一章
^^^^^^
[ADC]_ 可协助用户将数字信号转为模拟内容。

若 [ESP32]_ 作为从机进行全双工通信，则不需要主机发送命令和地址，直接进行数据交换。但此时需要注意 的是，读/写操作开始时，CS 需要提前至少一个 [SPI]_ 时钟长度拉低；读/写结束后，CS 需要至少延迟一个 SPI 时 钟长度拉高。


Glossary
^^^^^^^^
.. [ADC] ：数模转换器
.. [ESP32] ：一款乐鑫模组
.. [SPI] ：串行外设接口