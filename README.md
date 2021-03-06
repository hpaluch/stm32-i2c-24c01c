# Connecting STM32 Nucleo board to I2C EEPROM 24C01C

> WARNING!
>
> The program will overwrite all data on connected EEPROM 24C01C!

This program now shows basic EEPROM operations:
* cleaning whole EEPROM using `HpStm_24c_WritePattern(0xff);`
* writing test data to EEPROM using `Hpstm_24c_WriteToEeprom()`
  Please notice that this function must write data carefully not crossing
  page boundary (16-bytes for `24C01C` - can be different for other
  `24x01x` models!!!)
* reading and dumping whole EEPROM using `Hpstm_ReadAndDumpEEPROM()`

NOTE: There is no wrapper for reading EEPROM data because
it is completely implemented in provided HAL
function `HAL_I2C_Mem_Read()`

Other functions:
* green LED after init
* red LED in case of fatal error (halting CPU)
* blinking blue LED in `while(1)` loop (after all EEPROM operations succesfully finished)
* UART output on init or error.

Here is picture of my prototype circuit:

![STM32 Nucleo with I2C EEPROM 24C01C and Logic Analyzer](https://github.com/hpaluch/stm32-i2c-24c01c/blob/master/assets/stm32-i2c-24c01c-logic-analyzer.jpg?raw=true)

![Schematic of STM32 Nucleo with I2C EEPROM 24C01C](https://github.com/hpaluch/stm32-i2c-24c01c/blob/master/assets/ExpressPCB/nucleo-i2c-24c01c.png?raw=true) 

NOTE: If you never used the `ST NUCLEO F767ZI` board please
read my
article [Getting started with ST NUCLEO F767ZI Board][Getting started with ST NUCLEO F767ZI Board]

# Setup

Required Hardware:
* [STM NUCLEO-F767ZI][STM NUCLEO-F767ZI] development board with Cortex-M7 CPU. 
  Ordered [STM NUCLEO-F767ZI - Amazon.de][STM NUCLEO-F767ZI - Amazon.de]
* [24C01C][24C01C] - I2C EEPROM. Ordered [24C01C-I/P - Tme.eu][24C01C-I/P - Tme.eu]
* 1x 100nF ceramic capacitor
* 2x 4k7 resistor
* any suitable bread-board for prototyping  

Recommended Hardware:
* `Logical Analyzer`.
   Ordered [KeeYees USB Logic Analyzer - Amazon.de][KeeYees USB Logic Analyzer - Amazon.de]
* [Amphenol 2x5 cable - Tme.eu][Amphenol 2x5 cable - Tme.eu] to
  reduce strain on Logical Analyzer's connector.
* [Bread board wires - Tme.eu][Bread board wires - Tme.eu] to
  wire parts on bread board
* [Elegoo Jumper Wires - Amazon.de][Elegoo Jumper Wires - Amazon.de] - to
  interconnect boards and Analyzer etc.

Required Software:
* [STM32CubeF7][STM32CubeF7] Firmware package required to build this project.
  Must be unpacked to `C:\ARM` directory as `c:\ARM\STM32Cube_FW_F7_V1.14.0`.
* [System Workbench for STM32][System Workbench for STM32] development IDE

Recommended Software:
* [STM32CubeMX][STM32CubeMX] - this project was generated using this tool.
  You can open `stm32-i2c-24c01c.ioc` project file and customize it.
* [Putty][Putty] to watch info/error messages from ARM program. Putty
  is perfect replacement for Hyperterminal (which is no longer included with recent Windows)
* [Sigrok PulseView][Sigrok PulseView] most popular software for `Logical Analyzer`

# Build

Use standard:
* right-click on project name in `Project Explorer`
* select `Build project`


# Run/Debug

To quickly run this program you can just use:
* right-click on project name in `Project Explorer`
* select `Target` -> `Program chip...`
* select compiled binary
* ensure that `Reset after program` is checked.

To debug this program use common procedure as
described on [Getting started with ST NUCLEO F767ZI Board] 

To see info/error messages on UART you need to:
* get `COMx` port number in `Device Manager`:
  - expand `Ports (COM & LPT)`
  - notice COM port number, for example `STMicroelectronics STLink Virtual COM Port (COM3)`

In Putty: 
* create new connection type `Serial` with:
  - Serial line: `COMx` (replace `x` with your COM port number)
  - Speed: 115200
* in `Connection` -> `Serial` tree use:
  - Data bits: `8`
  - Stop bits: `1`
  - Parity: `None`
  - Flow control: `None`
* apply/save etc... and connect


On program startup you should see UART output like:
```
Starting program on ../Src/main.c:241
Program build date: Mar  2 2019 10:40:52
GCC version: 7.2.1 20170904 (release) [ARM/embedded-7-branch revision 255204]
Init complete.
Cleaning EEPROM...
Done in 99 [ms]
Test string is: 'Hello! SysTick=115'
Writing test data to EEPROM...
Done in 23 [ms]
Reading whole EEPROM - 128 bytes
Done in 12 [ms]
Dump of buffer at 0x0x20000094,  bytes 128
0x0000 ff ff ff ff ff 48 65 6c 6c 6f 21 20 53 79 73 54 .....Hello! SysT
0x0010 69 63 6b 3d 31 31 35 00 ff ff ff ff ff ff ff ff ick=115.........
0x0020 ff ff ff ff ff ff ff ff ff ff ff ff ff ff ff ff ................
0x0030 ff ff ff ff ff ff ff ff ff ff ff ff ff ff ff ff ................
0x0040 ff ff ff ff ff ff ff ff ff ff ff ff ff ff ff ff ................
0x0050 ff ff ff ff ff ff ff ff ff ff ff ff ff ff ff ff ................
0x0060 ff ff ff ff ff ff ff ff ff ff ff ff ff ff ff ff ................
0x0070 ff ff ff ff ff ff ff ff ff ff ff ff ff ff ff ff ................
```

# Logic Analyzer output

Using [KeeYees USB Logic Analyzer][KeeYees USB Logic Analyzer - Amazon.de] here
are following outputs.

EEPROM write (writing test string `Hello! ...` starting on EEPROM address `0x5`:

![STM32 - I2C EEPROM 24C01C page write](https://github.com/hpaluch/stm32-i2c-24c01c/blob/master/assets/PulseView/stm32-i2c-24c01c-page-write.png?raw=true) 

NOTES:
* The warning `Warning: Page write crossed...` is harmless in case 
  of `24C01C` EEPROM (it has 16-byte page buffer as described
  in [24C01C data-sheet][24C01C]. However if you will use
  different EEPROM version
  you may need to correct macro:
  
  ```c
  #define HPSTM_24C_WRITE_PAGE_SIZE 16 
  ```
  
  in `main.c`

  WARNING! Original Atmel `24C01` will NOT work, because it has
  different Address format (using `Device address` as
  `EEPROM data address` - violating I2C standard) -
  see [Atmel 24C01][Atmel 24C01] data-sheet to verify it by yourself.

* There is absolutely perfect 100Khz timing as can be seen on right corner "Cursors"

Reading all data from EEPROM:

![STM32 - I2C EEPROM 24C01C read all](https://github.com/hpaluch/stm32-i2c-24c01c/blob/master/assets/PulseView/stm32-i2c-24c01c-read-all.png?raw=true) 


# Random notes

* The `SCA` and `SCL` must have pull-up resistor (tested `4k7`) tied to `+5V` otherwise
  EEPROM will refuse to work.

* `int __io_putchar(int ch)` redefinition (required to redirect `printf(3)`
   to UART will not work until generated `syscalls.c` is added to 
   source files (without it there are hardcoded null stubs, without
   weak symbol `__io_putchar` to be redefined)

# Resources

You can also look 
on [How to access I2C EEPROM 24C01C from LC CH341A USB Adapter][How to access I2C EEPROM 24C01C from LC CH341A USB Adapter]
to compare this solution to version using USB to I2C Adapter.

[Sigrok PulseView]: https://sigrok.org/wiki/Main_Page
[Elegoo Jumper Wires - Amazon.de]: https://www.amazon.de/Elegoo-Steckbr�cken-Breadboard-Jumperkabel-Wiederverwendbare-Farbig/dp/B06XBHXWBD/
[Bread board wires - Tme.eu]: https://www.tme.eu/en/details/wjw-70b/universal-pcbs/wisher-enterprise/
[Amphenol 2x5 cable - Tme.eu]: https://www.tme.eu/en/details/fc10150-0/ribbon-cables-with-idc-connectors/amphenol/
[24C01C-I/P - Tme.eu]: https://www.tme.eu/en/details/24c01c-i_p/serial-eeprom-memories-integ-circ/microchip-technology/ 
[Atmel 24C01]: https://dflund.se/~triad/krad/entrega/at24c01.pdf
[24C01C]: http://ww1.microchip.com/downloads/en/devicedoc/21201k.pdf
[KeeYees USB Logic Analyzer - Amazon.de]: https://www.amazon.de/KeeYees-Logic-Analyzer-Farben-Arduino/dp/B07K6G55WG/
[STM32CubeF7]: https://www.st.com/en/embedded-software/stm32cubef7.html
[System Workbench for STM32]: http://www.openstm32.org/System%2BWorkbench%2Bfor%2BSTM32
[STM32CubeMX]: https://www.st.com/content/st_com/en/products/development-tools/software-development-tools/stm32-software-development-tools/stm32-configurators-and-code-generators/stm32cubemx.html

[STM NUCLEO-F767ZI - Amazon.de]: https://www.amazon.de/dp/B072MMZZBK/
[STM NUCLEO-F767ZI]: https://www.st.com/content/st_com/en/products/evaluation-tools/product-evaluation-tools/mcu-eval-tools/stm32-mcu-eval-tools/stm32-mcu-nucleo/nucleo-f767zi.html
[Getting started with ST NUCLEO F767ZI Board]: https://github.com/hpaluch/hpaluch.github.io/wiki/Getting-started-with-ST-NUCLEO-F767ZI-Board
[STM32CubeF7]: https://www.st.com/en/embedded-software/stm32cubef7.html
[Putty]: https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html
[Getting started with ST NUCLEO F767ZI Board]: https://github.com/hpaluch/hpaluch.github.io/wiki/Getting-started-with-ST-NUCLEO-F767ZI-Board
[How to access I2C EEPROM 24C01C from LC CH341A USB Adapter]: https://github.com/hpaluch/ch341-i2c-24c01c

