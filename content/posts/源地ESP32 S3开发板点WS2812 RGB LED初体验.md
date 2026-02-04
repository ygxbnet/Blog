---
title: "源地ESP32 S3开发板点WS2812 RGB LED初体验"
date: 2026-02-04T20:25:54+08:00
draft: false

# 文章短链接
slug: esp32 s3 first use
# 分类
categories: ["硬件开发"]
# 标签
tags:
  - ESP32
  - ESP32-S3
# 文章图片
featuredImage: https://img.ygxb.net/i/2026/02/04/69833c9941e22.webp
---

## 前言

这几天回看博客，上次更新都已经两年前了，正好这几天闲来无聊把以前的 ESP32 S3 又翻出来玩了玩，就来水一篇博客吧，也许后面还会出好几个 ESP32 的玩法（应该）

这次探索之旅并不顺利，中间踩了不少坑。

本次开发环境：CLion PlatformIO

## 准备

开发板是我几年前买的一块源地 ESP32 S3 开发板，这次放假闲来无聊就翻出来折腾折腾

<img src="https://img.ygxb.net/i/2026/02/04/698341189b6da.webp" alt="privacyMosaicImage_a358299d-8d5b-4b58-b36c-92bad81ed03e" style="zoom:25%;" />

看到开发板上自带一颗 RGB 灯就想着看能不能点，结果一上来就遇到了第一个坑，淘宝商品详情页的 RGB 灯标注的引脚是 GPIO47

<img src="https://img.ygxb.net/i/2026/02/04/698347fcba1ca.webp" alt="image-20260204212201748" style="zoom:50%;" />

因为这个板子是兼容乐鑫官方的 [ESP32-S3-DevKitC-1](https://docs.espressif.com/projects/esp-dev-kits/zh_CN/latest/esp32s3/esp32-s3-devkitc-1/index.html#esp32-s3-devkitc-1) 开发板，我就去文档查了一下，结果一查这下更迷糊了

官方开发饭有 V1.0 和 V1.1 两个版本，RGB 灯管脚分别是 GPIO48 和 GPIO38，好家伙，加上源地的一起一共出了 48 47 38 这三个管脚，后来看了代码中的定义才确定是 **GPIO48**，顺便确认了乐鑫的接线图和源地的这块板子是通用的

<img src="https://img.ygxb.net/i/2026/02/04/6983489c6bbfd.webp" alt="ESP32-S3_DevKitC-1_pinlayout" style="zoom:50%;" />

## 教程

源地的这块开发板板载的是 **WS2812 RGB LED**，直接像正常点LED那样输出高电平是没法点亮的，要用 PWM 信号点亮（原理见[一文带你了解WS2812原理及驱动-CSDN博客](https://blog.csdn.net/Xhw3f586/article/details/132295552)）

网上大部分教程都是用 **Adafruit NeoPixel** 库点亮，但其实根本不用库，用自带的函数就能点，完整代码如下

```c++
#include <Arduino.h>
void setup() {
    neopixelWrite(RGB_BUILTIN, 64, 0, 0); // 红
}
```

` neopixelWrite` 是自带的驱动 **WS2812 RGB LED** 的函数

- RGB_BUILTIN 是 RGB LED 的管脚，默认是 GPIO48（虽然源代码对 RGB_BUILTIN 定义为 `49+48=97`，但这不影响，这是转换后的GPIO48）
- 64 是表示红色亮度，范围为0-64
- 0 是表示绿色亮度，范围为0-64
- 0 是表示蓝色亮度，范围为0-64

```c++
neopixelWrite(RGB LED 管脚, 红色亮度, 绿色亮度, 蓝色亮度);
```

---

会基本电灯后就能玩点不一样的了，比如彩色呼吸灯

```c++
#include <Arduino.h>

void setup() {
}

void loop() {
    for (int i = 1; i < RGB_BRIGHTNESS; i++) {
        neopixelWrite(RGB_BUILTIN, i, 0, 0); // 红
        delay(10);
    }
    for (int i = RGB_BRIGHTNESS; i > 1; i--) {
        neopixelWrite(RGB_BUILTIN, i, 0, 0); // 红
        delay(10);
    }
    for (int i = 1; i < RGB_BRIGHTNESS; i++) {
        neopixelWrite(RGB_BUILTIN, 0, i, 0); // 绿
        delay(10);
    }
    for (int i = RGB_BRIGHTNESS; i > 1; i--) {
        neopixelWrite(RGB_BUILTIN, 0, i, 0); // 绿
        delay(10);
    }
    for (int i = 1; i < RGB_BRIGHTNESS; i++) {
        neopixelWrite(RGB_BUILTIN, 0, 0, i); // 蓝
        delay(10);
    }
    for (int i = RGB_BRIGHTNESS; i > 1; i--) {
        neopixelWrite(RGB_BUILTIN, 0, 0, i); // 蓝
        delay(10);
    }
}

```

![IMG_20260204_215311](https://img.ygxb.net/i/2026/02/04/69834fa40b553.webp)

## 参考文档

[ESP32-S3-DevKitC-1 v1.1 - ESP32-S3 - — esp-dev-kits latest 文档](https://docs.espressif.com/projects/esp-dev-kits/zh_CN/latest/esp32s3/esp32-s3-devkitc-1/user_guide_v1.1.html#id19)

[ESP32-S3 onboard RGB LED - Projects / Programming - Arduino Forum](https://forum.arduino.cc/t/esp32-s3-onboard-rgb-led/1198754/13)

