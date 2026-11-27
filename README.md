# ACC DAP-UART Debugger (STM32G431) – Hardware

A small, low-cost **USB-C debug probe** that combines a **CMSIS-DAP (SWD) debugger** and a **USB-to-UART bridge** in a single USB device. It connects to the target with a 10-pin **Tag-Connect TC2050** cable.

When plugged into a PC, the board shows up as:
- a **CMSIS-DAP v2 probe** (works with OpenOCD, `west flash`, `west debug`, GDB)
- a **virtual COM port** (USB CDC-ACM) that forwards the target's UART

One cable to the PC, one cable to the target.

> **Firmware:** this repository only contains the hardware. The Zephyr RTOS firmware is here:
> [acc-dap-uart-debugger-stm32g431cbtx-fw](https://github.com/Nightshadow1258/acc-dap-uart-debugger-stm32g431cbtx-fw)
>
> **Blog post:** [DIY Debugger: CMSIS-DAP + UART over a Single USB-C Cable](https://blog.preisch.cloud/) <!-- TODO: replace with the post's URL once published -->

---

## Features

| | |
|---|---|
| **MCU** | STM32G431CBT6 (Cortex-M4, 170 MHz, 128 KB Flash, 32 KB RAM, LQFP48) |
| **USB** | USB Type-C, USB 2.0 Full-Speed, crystal-less (internal HSI48) |
| **Power** | Bus-powered from USB 5 V, no external supply |
| **Debug interface** | SWD (SWCLK, SWDIO, NRST) via bit-banged GPIO |
| **UART bridge** | USART2, default 115200 baud |
| **Target connector** | Tag-Connect TC2050 (10-pin, no header needed on the target) |
| **Indicators** | Power LED, status LED |
| **Enclosure** | Two-part 3D-printable case (FreeCAD) |

---

## Repository Structure

```
ACC_DAP-UART-Debugger_STM32G431CBTx/
├── Dokumentation/   # Schematic PDF, images, notes
├── KiCAD/           # KiCad project (ACC_DAP-UART-Debugger_STM32G431CBTx.kicad_pro)
└── FreeCAD/         # Enclosure design (top + bottom part)
```

---

## Target Connector Pinout (TC2050)

Pin names are from the **target MCU's side**.

| Pin Name   | Pin | Pin | Pin Name   |
|------------|:---:|:---:|------------|
| VDD        | 1   | 2   | GND        |
| UART_RXD   | 3   | 4   | UART_TXD   |
| JTMS_SWDIO | 5   | 6   | JTCK_SWCLK |
| JTDO_SWO   | 7   | 8   | JTDI       |
| TBD/Free   | 9   | 10  | JTRST      |

- The target's `UART_RXD` (pin 3) connects to the debugger's **TX**, and the target's `UART_TXD` (pin 4) to the debugger's **RX**.
- Pin 9 is reserved for future use.
- This debugger uses **SWD only**. The JTAG-only pins (`JTDI`, `JTRST`) and `SWO` are part of the connector standard but not used by the current firmware.

To use the debugger with your own board, place the **TC2050-IDC-NL** footprint (no-legs version, or the legged version for retention clips) and route it according to the table above.

---

## MCU Pin Assignment

| Function | MCU Pin | Notes |
|---|---|---|
| Target SWCLK | `PA5` | GPIO, push-pull |
| Target SWDIO | `PA4` | GPIO, pull-up |
| Target NRST  | `PB0` | GPIO |
| Target UART TX | `PA2` | USART2_TX |
| Target UART RX | `PA3` | USART2_RX |
| USB D- / D+ | `PA11` / `PA12` | USB FS |

---

## Enclosure

The case was designed in **FreeCAD** around the STEP model exported from KiCad.

- Two parts (top + bottom)
- Screws go in from the bottom and sit in recessed mounting points
- The top part holds **M2 threaded inserts** and has a cut-out for the TC2050 connector

---

## Getting Started

1. **Order / assemble** the PCB from the files in `KiCAD/`.
2. **Flash the firmware** over USB DFU (STM32 ROM bootloader):
   - hold `BOOT0`, plug in USB, release `BOOT0`
   - flash using the instructions in the [firmware repository](https://github.com/Nightshadow1258/acc-dap-uart-debugger-stm32g431cbtx-fw)
3. **Connect** the TC2050 cable to your target and check that the probe is detected:
   ```bash
   openocd -f interface/cmsis-dap.cfg -f target/stm32f4x.cfg
   ```
4. **Use it in a Zephyr project** by adding this to the target's `board.cmake`:
   ```cmake
   board_runner_args(openocd --config interface/cmsis-dap.cfg --config target/<target>.cfg)
   ```
   then run `west flash --runner openocd`.

### Tested targets
| Target | Status |
|---|---|
| STM32F4 ([Stepper Controller V1.0](https://blog.preisch.cloud/stepper-controller-v1-0/)) | Working |
| nRF52840 ([Sensor Node](https://blog.preisch.cloud/blog-sensor-node-v4-00/)) | Working |
| STM32G4 ([Motor Controller](https://blog.preisch.cloud/diy-motor-controller-part-i-hardware-design-bringup/)), incl. external flash | To test |

---

## License
This hardware design is licensed under the CERN-OHL-P v2
