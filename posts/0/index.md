---
author: ["Marcelo Borges"]
title: "Diving into LoRa: Initial Experiments with LLCC68 and ESP32"
date: "2024-06-16"
description: "A detailed walkthrough of my first experience with LoRa technology, featuring the LLCC68 module and ESP32-C3 boards. Learn about my setup process, challenges, and initial experiments."
summary: "Discover my initial journey into LoRa technology using the LLCC68 module. This post covers my setup process, the creation of a breadboard adapter, and the results of my first transmitter and receiver test using ESP32-C3 boards."
tags: ["LoRa", "LLCC68", "ESP32", "IoT", "Low Power"]
categories: ["LoRa", "IoT Projects"]
series: ["LoRa Adventures"]
ShowToc: false
TocOpen: true
---

While browsing Aliexpress, I stumbled upon some intriguing LoRa modules and decided to give them a try. I ended up purchasing three [LoRa LLCC68 modules](https://pt.aliexpress.com/item/1005005763543912.html?gatewayAdapt=glo2bra):

![LoRa-CC68-X1 module](images/lora-cc68-x1-module.png#center)

I wanted to create a low-power application utilizing LoRa technology. My background includes working on an NB-IoT project for my master's thesis, so I was eager to explore another low-power wireless technology.

As with any new technology, I began with a simple Hello World application. However, when I attempted to mount the ESP32-C3 with the LoRa modules on a breadboard, I encountered a problem: the LoRa module pins had a 2 mm spacing, making them incompatible with a standard breadboard. To solve this, I designed an [adapter board in KiCad](https://github.com/jmarcelomb/lora-adapter-board) that converts the pin spacing to be breadboard-compatible:

![Adapter board, front view](images/lora-board-front.png#center)

With the LoRa module now breadboard-compatible, I set up two nodes using ESP32-C3 boards: one as a receiver and the other as a transmitter, as shown below:

![Transmitter and Receiver LoRa nodes](images/two-lora-nodes.jpeg#center)

For this experiment, I used the `basic` example from the [nopnop2002/esp-idf-sx126x](https://github.com/nopnop2002/esp-idf-sx126x) repository. The wiring was done according to the following table:

| SX126X/LLCC68 |       | ESP32  | ESP32-S2/S3 | ESP32-C2/C3/C6 |
| :-----------: | :---: | :----: | :---------: | :------------: |
|     BUSY      |  --   | GPIO17 |   GPIO39    |     GPIO3      |
|      RST      |  --   | GPIO16 |   GPIO38    |     GPIO2      |
|     TXEN      |  --   |  N/C   |     N/C     |      N/C       |
|     RXEN      |  --   |  N/C   |     N/C     |      N/C       |
|     MISO      |  --   | GPIO19 |   GPIO37    |     GPIO4      |
|      SCK      |  --   | GPIO18 |   GPIO36    |     GPIO5      |
|     MOSI      |  --   | GPIO23 |   GPIO35    |     GPIO6      |
|      NSS      |  --   | GPIO15 |   GPIO34    |     GPIO7      |
|      GND      |  --   |  GND   |     GND     |      GND       |
|      VCC      |  --   |  3.3V  |    3.3V     |      3.3V      |

Using the ESP-IDF to build the example, I flashed one ESP32 as the transmitter and the other as the receiver. The monitor output is shown below:

![Application monitor output](images/example-monitor-log.png#center)

The poor reception and signal quality were due to the lack of an antenna; the PCB trace was functioning as a antenna. Despite this, it was a successful first interaction with LoRa technology. It demonstrated that working with LoRa isn’t as daunting as it might seem and inspired several potential applications, such as creating open/close end-device sensors for my house gates.

This initial experiment has sparked numerous ideas for future projects, and I’m excited to explore more advanced LoRa applications. Thank you for joining me on this journey into LoRa technology. Stay tuned for more updates and projects!

Happy experimenting!

**jmmb**
