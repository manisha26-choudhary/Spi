 SPI Master (FSM Based)
This project implements a **Serial Peripheral Interface (SPI) Master** using a **Finite State Machine (FSM)** in SystemVerilog. The design provides a simple way to transmit an **8-bit data word** over SPI using the `MOSI`, `SCLK`, and `CS` signals.

The SPI Master cycles through four states: **Idle**, **Start Transmission**, **Data Transmission**, and **End Transmission**, controlled by an FSM.

## Module Interface

### **Ports**

| Name        | Direction | Type   | Description                       |
| ----------- | --------- | ------ | --------------------------------- |
| `clk`       | Input     | `wire` | System clock                      |
| `rst`       | Input     | `wire` | Active-high reset (synchronous)   |
| `tx_enable` | Input     | `wire` | Starts SPI transmission when high |
| `mosi`      | Output    | `reg`  | Master Out Slave In data line     |
| `cs`        | Output    | `reg`  | Chip select (active low)          |
| `sclk`      | Output    | `wire` | Serial clock for SPI              |

---

## FSM States

| State      | Encoding | Description                                                       |
| ---------- | -------- | ----------------------------------------------------------------- |
| `idle`     | 2'b00    | SPI idle, `cs = 1`, `mosi = 0`. Waits for `tx_enable`.            |
| `start_tx` | 2'b01    | Assert chip select (`cs = 0`) and prepare clock for transmission. |
| `tx_data`  | 2'b10    | Shift out 8-bit data (`din`) on MOSI, MSB first.                  |
| `end_tx`   | 2'b11    | Deassert chip select, end transaction, return to idle.            |

## Data Format

* Default transmit data: `din = 8'b10101010` (`0xAA`).
* Data is transmitted **MSB-first**.
* `mosi` updates according to `bit_count` during the `tx_data` state.

## Clocking

* `sclk` is generated internally from the input clock and toggles depending on the state.
* The SPI clock is derived using a **3-bit counter (`count`)** that divides and phases the input clock.

## Operation Flow

1. On reset → system enters **Idle** state.
2. When `tx_enable = 1` → FSM moves to **Start Transmission** (`start_tx`).
3. After setup, FSM enters **Data Transmission** (`tx_data`) and shifts 8 bits onto `mosi`.
4. Once all 8 bits are transmitted → FSM transitions to **End Transmission** (`end_tx`).
5. After cleanup, FSM returns to **Idle**.

## Example Waveform

A typical transmission sequence looks like:

* `cs` goes low (slave selected).
* `sclk` toggles for each bit.
* `mosi` shifts out `10101010`.
* After 8 bits, `cs` goes high again.
