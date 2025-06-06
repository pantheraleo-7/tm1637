# TM1637

A modern Python library for interfacing with TM1637-based 4-digit 7-segment LED display modules, using the Linux GPIO character device interface (`gpiod`).

## Overview

The TM1637 is a driver IC commonly used in 4-digit 7-segment LED display modules. This library offers a clean, Pythonic interface for controlling these displays from any Linux-based platform with GPIO support, including Raspberry Pi, BeagleBone, and other single-board computers.

## Features

- Hardware-agnostic implementation using the modern Linux GPIO character device interface
- Automatic detection of the available GPIO chip device for the given clock and data I/O lines
- Uses the official [`libgpiod`](https://git.kernel.org/pub/scm/libs/libgpiod/libgpiod.git/about/) Python bindings: [`gpiod v2`](https://pypi.org/project/gpiod/)
- Text display
- Scrolling text
- Hexadecimal display (0x0000 to 0xffff)
- Number display (-999 to 9999 or two -9 to 99)
- Temperature display (-9 to 99 with degree Celsius or -9.9 to 99.9 with degree)
- Support for decimal point displays through `TM1637Decimal` class

## Installation

```bash
pip install python-tm1637
```

## Requirements

- Python 3.9 or later
- `gpiod` 2.3.0 or later
- Linux-based system with GPIO pins

## Hardware Connections

|     TM1637     |      Pin      |
|----------------|---------------|
|       CLK      |     GPIO X    |
|       DIO      |     GPIO Y    |
|       VCC      |   3.3V or 5V  |
|       GND      |     Ground    |
> where `TM1637(clk=X, dio=Y)`

## Quick Start

```python
import time

import tm1637


# Initialize display connected to GPIO 23 & 24 at /dev/gpiochip4
display = tm1637.TM1637(clk=23, dio=24, chip_path="/dev/gpiochip4")

# Write segments directly
display.write([127, 127, 127, 127]) # 8888
time.sleep(1)

# Turn off all LEDs
display.write([0, 0, 0, 0])
time.sleep(1)

# Display text
display.show("HIYA")
time.sleep(1)

# Display a number
display.number(8888)
time.sleep(1)
```

## API Reference

### `TM1637` Class

#### Constructor

```python
TM1637(clk, dio, *, chip_path="/dev/gpiochip*", brightness=7)
```

- `clk`: GPIO pin number for the clock line
- `dio`: GPIO pin number for the data I/O line
- `brightness`: Initial brightness level (0-7, default: 7)
- `chip_path`: Path to the GPIO chip device (defaults to auto-detection)

#### Properties
- `brightness`: Get or set the display brightness
- `chip_path`: Get the path to GPIO chip device in use
- `clk`: Get the GPIO pin number of the clock line passed to constructor
- `dio`: Get the GPIO pin number of the data I/O line passed to constructor

#### Methods

- `write(segments, pos=0)`: Write raw segments
- `show(string, sep=False, fill=" ")`: Display a string
- `scroll(string, delay=0.25)`: Scroll a string across the display
- `hex(val)`: Display a hexadecimal value (0x0000 to 0xffff)
- `number(num, zero_pad=True)`: Display an integer value (-999 to 9999)
- `numbers(num1, num2, sep=True, zero_pad=True)`: Display two integers values (-9 to 99)
- `temperature(num)`: Display a temperature integer value
- `temperature_decimal(num)`: Display a temperature decimal value

#### Static Methods

- `encode_char(char)`: Convert a character to its segment representation
- `encode_digit(digit)`: Convert a digit to its segment representation
- `encode_string(string)`: Convert a string to an array of segments

### `TM1637Decimal` Class

Extends `TM1637` to enable support for display modules having a decimal point after each digit.

#### Static Methods

- `encode_string(string)`: Convert a string [with periods in-between] to an array of segments

## Permissions

To access the GPIO chip device without root permissions, the user must be in the `gpio` group. To add the current user to the `gpio` group:

```bash
sudo usermod -aG gpio $USER
sudo reboot
```

## Examples

### Clock Display

```python
import time

import tm1637


display = tm1637.TM1637(clk=23, dio=24)

colon = True
while True:
    time.sleep(1 - time.time()%1)
    lt = time.localtime()
    display.numbers(lt.tm_hour, lt.tm_min, sep=colon)
    colon = not colon
```

### Temperature Display

```python
import tm1637


display = tm1637.TM1637(clk=23, dio=24)

display.temperature(23) # 23*C
display.temperature_decimal(23.5) # 23:5*
```

### Scrolling Text

```python
import tm1637


display = tm1637.TM1637(clk=23, dio=24)

display.scroll("HELLO WORLD", delay=0.1)
```

## License

MIT License - check the [LICENSE](LICENSE) file for details.

## Sources

- [Original MicroPython implementation by Mike Causer](https://github.com/mcauser/micropython-tm1637)
- [Python port for Raspberry Pi by Patrick Palma](https://github.com/depklyon/raspberrypi-tm1637)
