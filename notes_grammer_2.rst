第一章
^^^^^^
ADC 可协助用户将数字信号转为模拟内容。

若 ESP32 作为从机进行全双工通信，则不需要主机发送命令和地址，直接进行数据交换。但此时需要注意 的是，读/写操作开始时，CS 需要提前至少一个 SPI 时钟长度拉低；读/写结束后，CS 需要至少延迟一个 SPI 时 钟长度拉高。

相关资源
^^^^^^^^
请见 `项目术语表<https://github.com/NatashaLi-ESPRESSIF/test_proj/blob/master/notes_grammer_22.rst>`_ 。