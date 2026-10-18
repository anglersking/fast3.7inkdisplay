# 硬件资料索引

本目录放的是原厂 PDF，未做修改，方便对照原理图和引脚。

| 文件 | 版本 / 日期 | 内容 | 原始文件名 |
| --- | --- | --- | --- |
| [ESP32-L_SCH.pdf](ESP32-L_SCH.pdf) | Rev C / 2022-10-08 | ESP32-L 主板原理图（ESP32-S 模块、CH340G、ASM1117-3.3、SPI 外设、P5 屏幕接口） | `ESP32-L_SCH.pdf` |
| [DESPI-C02_SCH_V1.0.pdf](DESPI-C02_SCH_V1.0.pdf) | V1.0 | DESPI-C02 转接板原理图（FPC24、升压、RESE 选择） | `DESPI-C02_SCH V1.0.pdf` |
| [DESPI-C02_Connector_Board_V1.1.pdf](DESPI-C02_Connector_Board_V1.1.pdf) | 手册 V1.1 / 规格 V2.0（2025-10-26） | 转接板说明：功能框图、RESE、测试点、驱动电路设计建议 | `DESPI-C02 Connector Board for E-paper Display V1.1.pdf` |
| [ESP32-L_C02_Eval_Kit_V2.0.pdf](ESP32-L_C02_Eval_Kit_V2.0.pdf) | V2.0 / 2022-07-07 | ESP32-L + DESPI-C02 评估套件：供电、跳线、RESE、烧录 | `E-Paper Display Evalution Kit for ESP32.pdf` |
| [ESP32-L_C102_Eval_Kit_V2.0.pdf](ESP32-L_C102_Eval_Kit_V2.0.pdf) | V2.0 / 2023-10-17 | ESP32-L + DESPI-C102 评估套件：Arduino 环境、板型选择、烧录步骤 | `ESP32-L(C102)-E-Paper Display Evalution Kit for E.pdf` |
| [ESP32-L_SPI_EPaper_WiFi_Manual_V1.0.pdf](ESP32-L_SPI_EPaper_WiFi_Manual_V1.0.pdf) | V1.0 / 2025-02-12 | ESP32-L WiFi 图传：ImageToWiFi（PC）、WiFi-W2（手机）、ImageToServer（远程） | `SPI Full Series E-Paper WiFi Local and Remote Driver User Manual.pdf` |

## 说明

- ESP32-L(C102) 那份手册是给 1.02 寸屏的套件写的，但主板、Arduino 环境、烧录流程和本项目完全一样，可以当 ESP32-L 的通用上手文档看。
- 屏幕 GDEY037T03 的规格书体积较大（约 4.4 MB），没有放进仓库，见 [Good Display 下载页](https://www.good-display.com/companyfile/879.html)。
- 资料版权归大连佳显电子（Dalian Good Display）所有，仅作学习与二次开发参考。
