# 硬件说明

## 系统组成

```
Type-C 5V ──> ESP32-L（ESP32-WROOM-32D + CH340G + 3.3V LDO）
                  │  P5：3.3V / GND / SDI / SCK / CS / D/C / RES / BUSY
                  ▼
             DESPI-C02（电平转接 + 升压电路 + RESE 选择）
                  │  P1：24 pin FPC
                  ▼
             GDEY037T03（3.7" / 416×240 / UC8253）
```

整机 5 V 输入，ESP32 和逻辑用 3.3 V；屏幕需要的高压（VGH / VGL / VCOM）由转接板上的升压电路产生，不需要额外电源。

## 屏幕：GDEY037T03

厂商标识里的 `A32-GDEY037T03-FP4G` 表示 3.7 寸、4 灰阶、FPC 接口。

| 参数 | 值 |
| --- | --- |
| 尺寸 | 3.7 inch |
| 分辨率 | 416 × 240（竖屏原生扫描 240 × 416） |
| PPI | 129.8 |
| 驱动 IC | UC8253 |
| 接口 | 4 线 SPI |
| 灰阶 | 最大 4 级（全刷 2 s 模式下） |
| 全刷 | 2 s |
| 快刷 | 1.5 s |
| 局刷 | 0.3 s |
| 刷新功耗 | 18 mW |
| 待机功耗 | 0.003 mW |
| 工作电压 | 3.3 V |
| 工作温度 | 0 ~ 50 ℃ |
| 存储温度 | -25 ~ 70 ℃ |
| 有效区域 | 81.54 × 47.04 mm |
| 外形尺寸 | 92.99 × 53 × 1.0 mm |
| FPC | 24 pin / 0.5 mm 间距 |

特性：

- 双稳态，断电后画面保持，不刷新时基本不耗电。
- 纯反射、无背光，视角 >170°，硬涂层防眩光。
- 波形存在 IC 内部 OTP 里，初始化时不需要 MCU 下发波形。
- 自带温度传感器，也支持外接 I²C 温度传感器做波形补偿。

刷新时间是在 25 ℃ 下测的，低温下会变慢，这是电子纸的正常现象。

## 驱动板：ESP32-L

| 项目 | 说明 |
| --- | --- |
| MCU | ESP32-WROOM-32D（双核 240 MHz，520 KB SRAM，WiFi 2.4 GHz） |
| 供电 | Type-C 5 V，ASM1117-3.3 转 3.3 V，F1 500 mA 保险丝 |
| 串口 | CH340G + DTR/RTS 自动下载电路，只有 RST 按键 |
| 屏幕接口 | P5，8 pin |
| 板上外设 | 字库 GT30L32S4W（CS = GPIO15）、Flash W25Q16（CS = GPIO2）、电源/串口指示灯 |
| 调试 | TEST1 / TEST2 电流测试点、P3 供电跳线、P4 外部电源输入 |
| 尺寸 | 70 × 54 mm |

几个容易踩的点：

- **P3 跳线必须短接**，它给转接板（也就是屏幕）供 3.3 V，拔掉跳线屏幕就不亮。
- 板子只有 RST 一个按键，下载靠 CH340 的 DTR/RTS 自动进 bootloader，不需要手动按 BOOT。
- 屏幕、字库、Flash 共用 SPI：SCK = GPIO18、MOSI = GPIO23、MISO = GPIO19，分别用 GPIO27 / GPIO15 / GPIO2 做片选。
- ESP32 只支持 2.4 GHz WiFi，5 GHz 连不上。

## 转接板：DESPI-C02

| 项目 | 说明 |
| --- | --- |
| 功能 | 24 pin FPC 转 8 pin 排针，兼容 13.3 寸及以下的 24 pin SPI 屏 |
| 供电 | 3.3 V |
| 尺寸 | 41 × 22 mm |
| 升压 | L1 10 µH / 1 A、Q1 Si1308EDL、D1~D3 MBR0530、4.7 µF / 25 V |
| 选择开关 | P3 拨码选择 RESE |
| 测试点 | PREVGH、PREVGL、VCOM、GND |

P1 的 24 pin FPC 中，和代码相关的几个信号（按原理图）：

| Pin | 信号 | Pin | 信号 |
| --- | --- | --- | --- |
| 14 | SDI | 12 | CS |
| 13 | SCLK | 11 | D/C |
| 10 | RES | 9 | BUSY |
| 8 | BS | 3 | RESE |
| 2 | GDR | 24 | VCOM |

其余为升压与电源脚（VDD / VSS / VCI / VDDIO / VPP / VSH / VSL / PREVGH / PREVGL / VGH / VGL 等），具体定义见 [DESPI-C02 原理图](hardware/DESPI-C02_SCH_V1.0.pdf)。

> 不同屏厂的 24 pin 定义可能不同，换屏时以屏幕规格书为准。

## RESE 选择

RESE 是升压电路电流反馈的采样电阻，选错会导致屏幕刷新异常甚至完全不刷新。DESPI-C02 上用 P3 拨码切换：

| 屏幕类型 | RESE |
| --- | --- |
| UC（UltraChip）系列 IC，如 UC8253、UC8151D、UC8276、UC8179 | **0.47 Ω** |
| SSD（Solomon Systech）系列 IC | 2.2 Ω（V1.1 手册；早期文档写 3 Ω） |
| 黑白红黄四色屏 | 2.2 Ω（同上） |

**本项目用 0.47 Ω。**

## 供电与功耗

- 建议 5 V / 1 A 以上，刷新瞬间升压电路有电流尖峰，供电不足会刷新失败或花屏。
- 测主板电流：关掉电源开关，电流表串到 TEST1。
- 测屏幕电流：保持开机，拔掉 P3 跳线，电流表串到 TEST2，测完记得插回。
- 待机功耗 0.003 mW，做电池方案时主要看 ESP32 深睡电流，屏幕基本可以忽略。

## 自己画板时的注意点

- FPC 座选 24 pin、0.5 mm、上接或上下接。
- UC 系列升压电感 10 µH / 1 A，SSD 系列 47 µH / 500 mA。
- MOSFET 推荐 Si1304BDL / Si1308EDL，可用 AO3400 替代。
- 肖特基二极管选 MBR0530 同级别。
- 升压电容的耐压要够，耐压不足会在升压时击穿。
- 元件参数以屏幕规格书为准，DESPI-C02 上的值只是参考设计。

## 相关资料

| 文件 | 内容 |
| --- | --- |
| [ESP32-L_SCH.pdf](hardware/ESP32-L_SCH.pdf) | ESP32-L 原理图 |
| [DESPI-C02_SCH_V1.0.pdf](hardware/DESPI-C02_SCH_V1.0.pdf) | DESPI-C02 原理图 |
| [DESPI-C02_Connector_Board_V1.1.pdf](hardware/DESPI-C02_Connector_Board_V1.1.pdf) | DESPI-C02 说明书（RESE、测试点、设计建议） |
| [ESP32-L_C02_Eval_Kit_V2.0.pdf](hardware/ESP32-L_C02_Eval_Kit_V2.0.pdf) | ESP32-L + DESPI-C02 评估套件说明 |
| [ESP32-L_C102_Eval_Kit_V2.0.pdf](hardware/ESP32-L_C102_Eval_Kit_V2.0.pdf) | ESP32-L + DESPI-C102 评估套件说明（Arduino 环境、烧录步骤同样适用） |
| [ESP32-L_SPI_EPaper_WiFi_Manual_V1.0.pdf](hardware/ESP32-L_SPI_EPaper_WiFi_Manual_V1.0.pdf) | WiFi 图传手册（PC 端 ImageToWiFi、手机端 WiFi-W2、远程 ImageToServer） |
