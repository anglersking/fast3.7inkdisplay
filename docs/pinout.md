# 引脚与丝印对照

## 屏幕接口

ESP32-L 的 **P5** 与 DESPI-C02 的 **P2** 是同一组信号，两板丝印顺序一致，8 pin 直接对插。

物理方向上，pin 1 是 `3.3V` 那一端；板上丝印从上往下读依次是 `BUSY / RES / D/C / CS / SCK / SDI / GND / 3.3V`。

| P5 / P2 | 丝印 | ESP32 GPIO | 方向 | 说明 |
| --- | --- | --- | --- | --- |
| 1 | 3.3V | — | 电源 | 转接板与屏幕供电 |
| 2 | GND | GND | 电源 | 公共地 |
| 3 | SDI | GPIO23 | 输出 | SPI MOSI，写命令/数据 |
| 4 | SCK | GPIO18 | 输出 | SPI 时钟，示例用 10 MHz |
| 5 | CS | GPIO27 | 输出 | 片选，低有效 |
| 6 | D/C | GPIO14 | 输出 | 0 = 命令，1 = 数据 |
| 7 | RES | GPIO12 | 输出 | 复位，低有效，上电至少保持 10 ms |
| 8 | BUSY | GPIO13 | 输入 | 0 = 忙，1 = 空闲 |

屏幕只写不读，**MISO 不接屏幕**。

## 示例代码里的引脚

`firmware/GDEY037T03_Arduino/Display_EPD_W21_spi.h`：

```c
#define isEPD_W21_BUSY digitalRead(A14)      // BUSY
#define EPD_W21_RST_0   digitalWrite(A15,LOW) // RES
#define EPD_W21_DC_0    digitalWrite(A16,LOW) // D/C
#define EPD_W21_CS_0    digitalWrite(A17,LOW) // CS
```

这里的 `A14 ~ A17` 是板子丝印，同时正好等于 Arduino ESP32 核心里的 `A14 ~ A17` 宏：

| 丝印 | A 宏 | GPIO |
| --- | --- | --- |
| A14 | A14 | GPIO13 |
| A15 | A15 | GPIO12 |
| A16 | A16 | GPIO14 |
| A17 | A17 | GPIO27 |

所以这份厂商示例不用改引脚就能在 ESP32-L 上跑。换成别的板子时，把上面 4 个宏换成实际 GPIO 即可。

## 板载 SPI 总线

ESP32-L 上屏幕、字库芯片、Flash 芯片挂在同一条 SPI 总线上，靠各自的 CS 区分：

| 信号 | GPIO | 接到哪里 |
| --- | --- | --- |
| SCK | GPIO18 | 屏幕 / 字库 / Flash |
| MOSI | GPIO23 | 屏幕 / 字库 / Flash |
| MISO | GPIO19 | 字库 / Flash |
| CS（屏幕） | GPIO27 | P5 / P2 pin 5 |
| CS（字库 GT30L32S4W） | GPIO15 | U1 |
| CS（Flash W25Q16） | GPIO2 | U4 |

二次开发时如果复用这条总线，注意 **GPIO15 / GPIO2 不要同时拉低**，否则字库和 Flash 会一起被选中。

## 排针丝印（A / T 两套标号）

主板把 IO 引到两侧排针，同一根信号有 `A` 和 `T` 两套丝印，对应关系如下（只列本方案用得到的部分）：

| 丝印 | GPIO | 丝印 | GPIO |
| --- | --- | --- | --- |
| A0 | GPIO36 | A11 / T1 | GPIO0 |
| A3 | GPIO39 | A12 / T2 | GPIO2 |
| A4 / T9 | GPIO32 | A13 / T3 | GPIO15 |
| A5 / T8 | GPIO33 | A14 / T4 | GPIO13 |
| A6 | GPIO34 | A15 / T5 | GPIO12 |
| A7 | GPIO35 | A16 / T6 | GPIO14 |
| A10 / T0 | GPIO4 | A17 / T7 | GPIO27 |
| A18 | GPIO25 | A19 | GPIO26 |

## 电源、跳线与测试点

| 位置 | 作用 |
| --- | --- |
| ESP32-L P2 | 电源开关（ON / OFF） |
| ESP32-L P3 | 转接板 3.3 V 供电跳线，**必须短接** |
| ESP32-L P4 | 外部电源输入（5 V / 3.3 V / GND） |
| ESP32-L TEST1 | 串入电流表测主板电流（需关掉电源开关） |
| ESP32-L TEST2 | 串入电流表测转接板/屏幕电流（需先拔掉 P3 跳线） |
| DESPI-C02 P3 | RESE 拨码开关（**和上面 ESP32-L 的 P3 不是一回事**） |
| DESPI-C02 TP | PREVGH / PREVGL / VCOM / GND 电压测试点 |

## 注意事项

- **GPIO12 是 strapping 引脚（MTDI）**，复位瞬间必须为低，否则 ESP32 可能起不来。ESP32-L 已经处理好；自己画板时 RES 线上不要加上拉，或者加下拉。
- **BUSY 是输入**，初始化前先 `pinMode(BUSY, INPUT)`，其余 EPD 引脚设为输出。
- 屏幕 FPC 带电插拔可能直接损坏屏幕，插拔前先断电。
- 转接板 RESE 选错（比如给 UC8253 用了 2.2 Ω）时，屏幕可能完全不刷新，不是代码问题。
