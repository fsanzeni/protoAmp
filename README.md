# 100W USB-C PD Audio Amplifier with Integrated DSP

This repository contains the hardware design files, documentation, and firmware logic for a 100W audio amplifier board. The system integrates USB-C Power Delivery (PD) for power management, a dedicated digital signal processor (DSP) for audio manipulation, a Bluetooth Low Energy (BLE) microcontroller for system management, and a high-efficiency Class-D amplification stage.

This platform is designed for active speaker systems, custom crossover networks, and integrated audio processing applications requiring a compact footprint and standardised power inputs.

---

## System Architecture and Major Components

The board design relies on a specialised set of integrated circuits to handle power negotiation, wireless connectivity, digital signal processing, and audio amplification.

### USB Power Delivery: HUSB238

The [HUSB238](https://en.hynetek.com/2421.html) serves as the USB Type-C PD sink controller. It handles the low-level negotiation with the connected USB-C power source to request the necessary voltage and current (up to 20V at 5A, yielding 100W).

* **Protocol:** Compliant with USB PD 3.0 and Type-C 1.4 specifications.
* **Interface:** Interfaced via I2C, allowing the system MCU to read negotiated power capabilities and dynamically limit the amplifier's maximum output power to prevent source overcurrent.
* **Functionality:** Operates independently of the MCU for fundamental power requests, ensuring rapid voltage ramp-up and system stability upon cable insertion. Please note: without writing some code and pushing it via the exposed I2C lines, the HUSB238 can only output a maximum of 20V @ 3A (60W). This is a safety feature. Please see e.g., [Adafruit's](https://github.com/adafruit/Adafruit_HUSB238) code to figure out how to request up to 100W.

### Digital Signal Processor: ADAU1701

Audio processing is handled by the Analog Devices [ADAU1701](https://www.analog.com/en/products/adau1701.html), a fully programmable 28-/56-bit audio DSP.

* **Processing:** Executes audio algorithms at a high sample rate, suitable for bi-quad filters, dynamic range compression, time alignment, and active crossovers.
* **I/O Routing:** Features two built-in ADCs and four DACs, allowing for stereo input and up to four channels of analog output (e.g., routing separate high-frequency and low-frequency channels to the amplification stage).
* **Programming:** Configurable via Analog Devices' SigmaStudio. The DSP boots from an external EEPROM or is initialised via the I2C bus by the system MCU.

### System Controller & Wireless: STM32WBA65

The STMicroelectronics [STM32WBA65CIU6](https://www.st.com/en/microcontrollers-microprocessors/stm32wba65ci.html) acts as the central brain of the board. It is an ultra-low-power Arm Cortex-M33 microcontroller with integrated Bluetooth LE Audio capabilities.

* **System Management:** Acts as the I2C master to configure the HUSB238 (via solder jumpers) and the ADAU1701 (loading DSP parameters or switching presets).
* **Connectivity:** Handles Bluetooth Low Energy (BLE) connections for audio streaming and remote configuration, effectively acting as the bridge between mobile applications and the onboard DSP.

### Audio Amplification Stage

The output stage uses a Class-D amplifier topology configured to deliver up to 100W of total output power, distributed as either a bridged mono (PBTL) output or a standard stereo configuration - or a combination of both.

I'm using here a pair of synced [TPA3126D2DADR](https://www.ti.com/product/TPA3126D2/part-details/TPA3126D2DADR) - one configured as a 50-W stereo amp, the second as a 100-W amp. Obviously you can't request their max wattage at the same time, so I've included a PLIMIT circuit for both. You can also limit the individual amp's power limits via the DSP / SigmaStudio, but better safe than sorry!

---

## License

This project is licensed under the MIT License.