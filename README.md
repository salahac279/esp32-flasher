# STM32 to ESP32 USB-to-UART Hardware Flasher

A hardware bridge firmware that configures an **STM32F407VE** as a USB Virtual COM Port (CDC Class) to program an **ESP32** board with a damaged or unpopulated onboard USB-to-Serial interface.
> **Note:** This is a vibe coding project developed with an AI agent as a co-coder.
## Overview

When an ESP32's micro-USB port or USB-to-UART chip fails, standard host flashing utilities (`esptool`, Arduino IDE, PlatformIO) lose connectivity. This project turns an STM32 into a custom serial passthrough bridge.

To handle high-speed flashing without dropping frame bytes or encountering `USBD_BUSY` buffer overruns, the STM32 firmware uses an interrupt-driven **512-byte Circular Ring Buffer** to queue incoming serial bytes from the ESP32 and transmit them cleanly over USB CDC to the host PC.

## Hardware Wiring

| STM32F407 Pin | Signal Name | ESP32 Pin | Description |
| :--- | :--- | :--- | :--- |
| **PA2** | USART2_TX | **RX0** | Transmit data to ESP32 |
| **PA3** | USART2_RX | **TX0** | Receive data from ESP32 |
| **3V3** | Power | **3V3** | 3.3V System power rail |
| **GND** | Ground | **GND** | Common ground reference |

## Manual Bootloader Sequence

Since auto-reset circuitry (DTR/RTS hardware lines) is bypassed, trigger the ESP32 bootloader manually prior to flashing:

1. Connect **IO0** on the ESP32 to **GND**.
2. Tap the **EN** (or **RST**) pin to **GND** to reset the chip into Download Mode.
3. Disconnect **IO0** from **GND**.
4. In your IDE (`Arduino IDE` / `esptool.py`), select the STM32's COM port, set the baud rate to **115200**, and click **Upload**.
5. Once writing completes (`Writing at 0x00010000... (100%)`), tap **EN** to **GND** once more to execute the new code.

## Key Technical Features

- **USB CDC (Virtual COM Port):** Implemented on the native USB_OTG_FS hardware peripheral.
- **Interrupt-Driven UART:** Non-blocking byte collection on `USART2` (`HAL_UART_RxCpltCallback`).
- **Ring Buffer Architecture:** 512-byte FIFO queue in `main.c` prevents packet loss during high-volume read operations.
- **Overrun Recovery:** Automatic hardware error handling (`HAL_UART_ErrorCallback`) resets the `ORE` flag without locking up the bus.

## Directory Structure

```text
├── Core/
│   ├── Src/
│   │   ├── main.c              # Main loop, ring buffer logic, & UART ISRs
│   │   └── stm32f4xx_it.c      # Interrupt service routines
├── USB_DEVICE/
│   └── App/
│       └── usbd_cdc_if.c       # USB CDC data handlers
├── LICENSE                     # MIT License
└── README.md                   # Documentation
