# GDEY037T03 Arduino 示例

来源：大连佳显（Good Display）出料包 `A32-GDEY037T03-FP4G-20240815`，原样归档，只做了目录整理。

## 文件

| 文件 | 作用 |
| --- | --- |
| `GDEY037T03_Arduino.ino` | 主程序：全刷 / 快刷 / 4 灰 / 局刷演示 |
| `Display_EPD_W21.h` | 分辨率定义与对外接口 |
| `Display_EPD_W21.cpp` | UC8253 初始化、刷屏、局部刷新、深睡 |
| `Display_EPD_W21_spi.cpp` / `.h` | 4 线 SPI 读写与引脚宏 |
| `Ap_29demo.h` | 演示图片与数字数组（约 650 KB） |

## 参数

| 项目 | 值 |
| --- | --- |
| 分辨率 | `EPD_WIDTH = 240`、`EPD_HEIGHT = 416` |
| 单色帧大小 | `240 × 416 / 8 = 12480` 字节 |
| 4 灰帧大小 | `240 × 416 × 2 / 8 = 24960` 字节 |
| SPI | 10 MHz，Mode 0，MSB first |
| 引脚 | BUSY = A14、RES = A15、D/C = A16、CS = A17 |

## 主要函数

| 函数 | 说明 |
| --- | --- |
| `EPD_Init()` | 全刷初始化（VCOM 间隔 `0x50 = 0x97`） |
| `EPD_Init_Fast()` | 快刷初始化，`0xE5 = 0x5F`，约 1.5 s |
| `EPD_Init_Part()` | 局刷初始化，`0xE5 = 0x6E` |
| `EPD_Init_4G()` | 4 灰初始化，`0xE5 = 0x5A` |
| `EPD_WhiteScreen_ALL()` | 全刷一张图 |
| `EPD_WhiteScreen_ALL_Fast()` | 快刷一张图 |
| `EPD_WhiteScreen_ALL_4G()` | 4 灰刷一张图 |
| `EPD_SetRAMValue_BaseMap()` | 局刷前写入基图（必须） |
| `EPD_Dis_Part()` / `EPD_Dis_PartAll()` | 局部刷新 |
| `EPD_Dis_Part_Time()` | 局刷时钟演示 |
| `EPD_DeepSleep()` | 掉电 + 深睡，刷完必须调用 |

## 数据格式

**单色**（`gImage_1` 等，12480 字节）：

- 每像素 1 bit，MSB 在前；
- `1` = 白，`0` = 黑；
- 每行 240 像素 = 30 字节，共 416 行。

**4 灰**（`gImage_4G1`，24960 字节）：

- 每像素 2 bit，MSB 在前；
- `0xC0` = 白，`0x80` = 灰 1，`0x40` = 灰 2，`0x00` = 黑。

面板按 240（宽）× 416（高）扫描，横屏图要先旋转 90° 再取模。

## 运行

1. Arduino IDE 打开 `GDEY037T03_Arduino.ino`；
2. 板型选 **WEMOS LOLIN32**，选对 COM 口；
3. 上传；
4. 断电 → 插好屏幕 FPC 和转接板 → 上电。

## 注意事项

- 示例跑完停在 `while(1)`，只执行一轮，需要循环就把它去掉。
- 每次刷完都调用了 `EPD_DeepSleep()`，不要删。
- 快刷 / 局刷连续 5 次以内要做一次全刷，否则残影会累积。
- 全刷会闪烁几次（清残影），属正常现象；局刷不闪。
- 换引脚只改 `Display_EPD_W21_spi.h` 里的 4 个宏。
- 代码版权归大连佳显电子所有，这里仅作学习与二次开发参考。
