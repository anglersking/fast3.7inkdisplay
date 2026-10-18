# 快速开始

从零把屏幕点亮，大约需要 15 分钟。

## 准备

硬件：

- GDEY037T03 屏幕（24 pin FPC）
- ESP32-L 驱动板
- DESPI-C02 转接板
- Type-C 数据线（能传数据，不是纯充电线）
- 电脑

软件：

- Arduino IDE 2.x
- ESP32 开发板支持包
- CH340 驱动

## 1. 装环境

1. 从 [arduino.cc](https://www.arduino.cc/en/software) 下载安装 Arduino IDE。
2. 打开 `File > Preferences`，在 `Additional boards manager URLs` 里填：

   ```text
   https://espressif.github.io/arduino-esp32/package_esp32_index.json
   ```

3. `Tools > Board > Boards Manager`，搜索 `esp32`，安装 **esp32 by Espressif Systems**。
4. 装 CH340 驱动（WCH 官网或板卡资料里都有）。装完插上板子，设备管理器里应该出现一个新的 COM 口。

## 2. 接线（全部断电操作）

1. 用镊子或跳线帽**短接 ESP32-L 的 P3**（转接板供电）。
2. 把 DESPI-C02 的 **P3 拨码**拨到 **0.47**（UC8253 用这个值）。
3. 屏幕**显示面朝上、金手指朝上**插进转接板的 P1 座，插到底后压下黑色卡扣。
4. 把转接板的 8 pin 排针插到 ESP32-L 的 P5 座，注意 `3.3V` 对 `3.3V`、`BUSY` 对 `BUSY`。
5. 最后插 Type-C 上电。

> 顺序很重要：先接线，再上电。屏幕 FPC 带电插拔可能直接损坏。

## 3. 选择开发板

| 选项 | 值 |
| --- | --- |
| Board | WEMOS LOLIN32 |
| Port | 板子对应的 COM 口 |
| Upload Speed | 921600（不稳定就降到 115200） |

## 4. 编译下载

打开 [firmware/GDEY037T03_Arduino/GDEY037T03_Arduino.ino](../firmware/GDEY037T03_Arduino/GDEY037T03_Arduino.ino)，点上传。

板子只有 RST 一个按键，下载由 CH340 的 DTR/RTS 自动控制，不需要手动按 BOOT。如果提示连不上：

- 换一个 USB 口，或换一根确认能传数据的线；
- 把上传波特率降到 115200；
- 拔掉重插，等串口重新枚举后再点上传；
- 确认 `Tools > Port` 选的是 CH340 那个口。

## 5. 预期现象

下载完成后板子自动复位，屏幕依次显示：

1. 全刷清屏（白）；
2. 全刷图片（闪几次，约 2 s）；
3. 快刷图片（闪一次，约 1.5 s）；
4. 4 灰图片（约 2 s）；
5. 局刷演示：在 5 个位置刷新 `00:00` 数字（不闪）；
6. 最后进入 deep sleep，画面保持不动。

## 6. 换成自己的图

示例里的图片是 `Ap_29demo.h` 里的字节数组，格式是 **240 × 416、1 bit/像素、1 = 白、0 = 黑**，每个数组 12480 字节。

做法：

1. 准备一张 416 × 240 的横屏图（黑白）。
2. **顺时针或逆时针旋转 90°** 变成 240 × 416，因为面板是按 240 宽、416 行扫描的。
3. 用 image2cpp 或自己写脚本取模：
   - 画布 240 × 416
   - 黑白两色
   - MSB 在前
   - 白 = 1，黑 = 0
4. 把生成的数组替换进 `Ap_29demo.h`，或者新建头文件后改 `.ino` 里的引用。

方向反了就把旋转方向换一下，或者用示例里的 `EPD_Init_180()`。

## 7. 改代码的几个点

| 想做的事 | 改哪里 |
| --- | --- |
| 只跑全刷 | 把 `.ino` 里其他 `#if` 改成 `#if 0` |
| 循环刷新 | 去掉末尾的 `while(1)` |
| 换引脚 | `Display_EPD_W21_spi.h` 里的 `A14 ~ A17` |
| 局刷时钟 | `EPD_Dis_Part_Time(...)` 的坐标和数字数组 |
| 低功耗 | 刷完调用 `EPD_DeepSleep()`，然后 `esp_deep_sleep_start()` |

> 每次刷新前都要重新 `EPD_Init()` / `EPD_Init_Fast()` / `EPD_Init_Part()`，刷完必须 `EPD_DeepSleep()`。这不是可选项，省掉会缩短屏幕寿命。

## 8. 常见问题

| 现象 | 原因 / 处理 |
| --- | --- |
| 上电后屏幕一点反应都没有 | ESP32-L 的 P3 跳线没短接；或屏幕 FPC 没插到底 |
| 屏幕不刷新，但代码在跑 | DESPI-C02 的 RESE 拨码不是 0.47；或供电不足（换 5 V / 1 A 以上电源） |
| 花屏、重影 | 快刷/局刷做太多，跑一次全刷；局刷前没写基图 |
| 图像上下/左右颠倒 | 取模时旋转方向不对，换 90° 方向或用 `EPD_Init_180()` |
| 图像只显示一部分 | 数组长度或每行字节数不对，必须是 30 字节 × 416 行 |
| 串口认不到 | CH340 驱动没装；线是充电线 |
| 下载失败 | 板型不是 WEMOS LOLIN32；降波特率；重插 USB |

更多硬件问题见 [hardware.md](hardware.md)，引脚见 [pinout.md](pinout.md)。

## 9. 官方库（可选）

大连佳显还维护了一个通用库 **ESP32epdx**（Arduino 库管理器里搜得到），带 GUI 绘制、常用型号驱动，适合不想自己抠 UC8253 时序的场景：

- <https://github.com/gooddisplayshare/ESP32epdx>
