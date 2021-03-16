---
layout: post
title: STM32 development on Linux
date: 2018-08-28 23:29 +0800
tags:
  - Development
---

STM32 Development on Linux.

### Installation

1. Install the compiler

   ```shell
   sudo apt install gcc-arm-none-eabi
   # none: no vendor, not targeting any operating system
   # eabi: ARM embedded ABI
   ```

2. Install ST-Link to get connected to the board (drivers and utils)

   ```shell
   git clone --depth 1 https://github.com/texane/stlink.git
   # then build and install yourself
   ```

3. Install STM32CubeMX for project code generation

  [https://www.st.com/en/development-tools/stm32cubemx.html](https://www.st.com/en/development-tools/stm32cubemx.html)


### Compiling

  - Notice: `-mfloat-abi={soft,hard}`
  - Flashing to the board
    ```shell
    st-flash erase
    st-flash --reset write data.bin 0x8000000
    ```
