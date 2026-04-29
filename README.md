# clocksync (Rmni2 Fork) 

**ESP32 Fake Radio Clock Station**

Video:
[![Watch the video](https://img.youtube.com/vi/pbbZUizgu_k/maxresdefault.jpg)](https://youtu.be/pbbZUizgu_k)

This project is a fork from SASAKI Taroh (tarohs)

`clocksync` allows an ESP32 to emulate various Low-Frequency (LF) time signal stations, allowing you to sync radio-controlled clocks (JJY, WWVB, DCF77, MSF, etc.) even if you are out of range of the actual transmitters.

This project is a fork of `clocksync` project by SASAKI Taroh (tarohs), customized for more dynamic configurations and support for esp32-c3 super mini

### What's New in the Rmni2 Fork

- **Dynamic Configuration**: WiFi SSID, Password, and Timezone (TZ) can now be updated on the fly via Serial or Web commands.
- **Access Point (AP) Fallback**: If the ESP32 cannot connect to the configured WiFi network, it will broadcast its own `hotspot` at ip `1.2.3.4`.
- **ESP32-C3 Support**: just mapped different pins for the esp32-c3

## Features

- **Multi-Station Support**: Emulates JJY (Japan), WWVB (USA), DCF77 (Germany), MSF (UK), BSF (Taiwan), and BPC (China).
- **Web Control**: Simple web interface for status monitoring and configuration.
- **Home Assistant Integration**: REST API for scheduling transmission (e.g., run only at night).
- **Precise Timing**: Uses ESP32 hardware timers for accurate carrier generation and modulation.
- **OTA Updates**: Update firmware wirelessly via Arduino IDE.

## Getting Started

### 1. Hardware Setup

**Supported Boards**:
- **NO NAME ESP32-C3 SuperMini** (Recommended)
- **Generic ESP32 / ESP32-S3**: Compatible with standard DevKits.

**Wiring Diagram**:
```text
           ESP32 / M5Atom
          +-------------+
          |             |
          |     GND [ ]----(Resistor 220-330Ω)----+
          |             |                         |
          | GPIO 32 [ ]-----------------+         |
          |             |               |         |
          +-------------+               |         |
                                   [ Antenna ]    |
                                   [  Coil   ]    |
                                        |         |
                                        +---------+
```

1.  **Pin**: Connect one end of your antenna to `GPIO 6` (default) If Generic esp32 use `GPIO 32` (default).
2.  **Ground**: Connect the other end of the antenna to a **Current Limiting Resistor** (220Ω - 330Ω), and then to `GND`.
    *   *Why a resistor?* It protects your ESP32 from drawing too much current, as the antenna coil has very low resistance.
3.  **Antenna Types**:
    *   **Ferrite Rod (Recommended)**: Scavenge one from an old radio clock or buy a ferrite antenna tuned to your target frequency. **Important**: You must match the frequency! (Tested range > 1m)
        *   **60 kHz**: For WWVB (USA), MSF (UK), JJY (Japan).
        *   **40 kHz**: For JJY (Japan).
        *   **77.5 kHz**: For DCF77 (Germany), BSF (Taiwan).
        *   **68.5 kHz**: For BPC (China).
    *   **Wire Loop**: A simple coil of wire (e.g., 30 turns of magnet wire around a water bottle). Short range, but works for any frequency.
4.  **Placement**: Place your target watch/clock **inside** or **immediately next to** the antenna coil. This is a low-power near-field emulator; range is typically < 10cm.

### 2. Software Configuration
1.  **Upload the Firmware**:
    *   Go to releases.
    *   Download the correct bin file that matches your esp32 type.
    *   Use an esp32 tool, heres some web tools: 
        ```
        https://esptool.spacehuhn.com/
        https://www.espboards.dev/tools/program/
        ```
    *   Set offset to 0x0 and flash

2. **Wifi**:
    *   In the Serial Monitor/terminal.
    *   Use command ss to set your SSID (Wifi network name). 
        
        Example: My Wifi's SSID is "My Wifi's Name"
        ```
        ssMy Wifi's Name
        ```
    *   Use command ps to set your PASS. 
    
        Example: My Wifi's password is "My Wifi's Password"
        ```
        psMy Wifi's Password
        ```

3.  **Timezone**:
    *   In the Serial Monitor/terminal.
    *   Use command `tz` then your POSIX TZ string (not Olson names like `America/Los_Angeles`). example:
        ```
        tzPST8PDT,M3.2.0,M11.1.0
        ```

    *   **WWVB (USA)**: Set TZ to your US timezone. WWVB frame time is always UTC (via `gmtime()`), so there is no double-offset risk — TZ only affects the DST status bits.

        | Region | POSIX TZ string |
        | :--- | :--- |
        | Pacific (LA, SF, Seattle) | `PST8PDT,M3.2.0,M11.1.0` |
        | Mountain (Denver, Phoenix†) | `MST7MDT,M3.2.0,M11.1.0` |
        | Central (Chicago, Dallas) | `CST6CDT,M3.2.0,M11.1.0` |
        | Eastern (NYC, Miami) | `EST5EDT,M3.2.0,M11.1.0` |
        | Hawaii | `HST10` |

        †Arizona (no DST): use `MST7` instead.

    *   **Other stations**: These transmit local time directly, so TZ must match the station's native timezone:

        | Station | Region | POSIX TZ string |
        | :--- | :--- | :--- |
        | JJY | Japan | `JST-9` |
        | DCF77 | Germany / Central Europe | `CET-1CEST,M3.5.0/2,M10.5.0/3` |
        | MSF | United Kingdom | `GMT0BST,M3.5.0/1,M10.5.0` |
        | BSF | Taiwan | `CST-8` |
        | BPC | China | `CST-8` |

    *   **Lookup**: Find any POSIX TZ string by city name in [this table](https://github.com/nayarsystems/posix_tz_db/blob/master/zones.csv). On Mac/Linux: `tail -1 /etc/localtime`
4. **Finalizing**:
    * look at the commands below to set your desired station.
    * once you're done with everything, reboot.

### 3. Usage

Once running, the device acts as a time signal transmitter.
- **Status LED**: Indicates transmission is active.
- **Button**: Press to toggle transmission ON/OFF.
- **Web UI**: Visit `http://clocksync.local` (or the device IP) to view status.

## Command Reference

Control the device via Serial Monitor (115200 baud) or via HTTP (`http://clocksync.local/cmd?c=<command>`).

| Command | Description | Example |
| :--- | :--- | :--- |
| **Stations** | | |
| `sj` | Set station to **JJY (40 kHz)** (Fukushima) | `sj` |
| `sk` | Set station to **JJY (60 kHz)** (Fukuoka) | `sk` |
| `sw` | Set station to **WWVB (60 kHz)** (USA) | `sw` |
| `sd` | Set station to **DCF77 (77.5 kHz)** (Germany) | `sd` |
| `sm` | Set station to **MSF (60 kHz)** (UK) | `sm` |
| `st` | Set station to **BSF (77.5 kHz)** (Taiwan) | `st` |
| `sc` | Set station to **BPC (68.5 kHz)** (China) | `sc` |
| **Control** | | |
| `e1` / `e0` | **Enable / Disable Transmission** (Carrier & Logic) | `e1` |
| `pNN` | Set Radio Output Pin (GPIO `NN`) | `p25` |
| `g0` - `g3` | Set Drive Strength (0=Weakest, 3=Strongest) | `g2` |
| **Settings** | | |
| `ss` | Set SSID | `ssMy Wifi's Name` |
| `ps` | Set PASS | `ssMy Wifi's Password` |
| `tz` | Set POSIX Timezone String | `tzPST8PDT,M3.2.0,M11.1.0` |
| `y1` / `y0` | NTP Sync On / Off | `y1` |
| `l1` / `l0` | LED On / Off | `l1` |
| `x0` - `x2` | DCF/MSF DST Override (0=Std, 1=DST, 2=Auto) | `x2` |
| **Manual Time** | | |
| `dYYMMDD` | Manually set Date (Year, Month, Day) | `d231225` |
| `tHHmmSS` | Manually set Time (Hour, Min, Sec) | `t123000` |
| **Debug** | | |
| `h` | Show Help | `h` |
| `f` | Frequency Self-Test (Requires jumper `PIN_RADIO` -> `PIN_MEAS`) | `f` |
| `status` | (HTTP only) Get text status summary | |

## Supported Stations

| Station | Freq (kHz) | Location | Notes |
| :--- | :--- | :--- | :--- |
| **JJY** | 40 / 60 | Japan | Default is end-Low symbol shape. |
| **WWVB** | 60 | USA | UTC framing; DST bits derived from TZ setting. |
| **DCF77** | 77.5 | Germany | "Start-Low" encoding. |
| **MSF** | 60 | UK | Includes parity and DST bits. |
| **BSF** | 77.5 | Taiwan | Quaternary encoding (uncertified). |
| **BPC** | 68.5 | China | Quaternary encoding (uncertified). |

## Troubleshooting
*   **Clock isn't syncing**:
    *   **Proximity**: Sometimes, the signal is too strong for the watch, with a coil, you usually don't need to be touching.
    *   **Interference**: LED power supplies and monitors cause noise. Move away from them.
    *   **Volume**: Increase "Volume" (Drive Strength) to `g3` using the Serial/Web command.
    *   **Antenna**: Ensure your antenna wire isn't broken and the resistor is secure.
*   **Clock shows wrong time**:
    *   **Timezone**: Make sure you have the correct `TZ`. check your Watches GMT feature. For WWVB, use a US POSIX timezone (see setup step 2).
    *   **DST off by 1 hour (WWVB)**: If TZ is set to `"UTC0"`, DST bits will always be 0 and your clock won't spring forward. Set TZ to your US timezone.

## Technical Details

For deep dives into the signal encoding and protocol specifications used by this emulator, see [HOW_IT_WORKS.md](./HOW_IT_WORKS.md).

## Credits

- **Original Author**: SASAKI Taroh (tarohs) - [nisejjy](https://github.com/tarohs/nisejjy)
- **Reference**: [txtempus](https://github.com/hzeller/txtempus) by Henner Zeller (used for WWVB frame verification).
