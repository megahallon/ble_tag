# Building

Download nRF SDK 17 and softdevice S140, unpack in this directory. The directory should be called sdk.

https://www.nordicsemi.com/Products/Development-software/nRF5-SDK/Download

Important: The SPI driver in sdk/modules/nrfx/drivers/src/nrfx_spim.c contains a delay on 200ms after setting SS pin. Look for nrf_delay_ms(200). This line should be removed or SPI access will be very slow. Using hardware SS pin activation would be a better solution.

Project files are provided for Segger Embedded Studio

To use round board, enable NRF_CKAA define seen in beginning of main file (normally commented out). The only difference currently is GPIO setup to accelerometer.

# Programming

Programming is done using a Segger J-Link Basic or similar device. A 5-pin 1.27mm header is available on the board for this purpose. For simple connection a clamp like https://www.adafruit.com/product/5434 is suggested. The pins are as follows, first pad is at middle of board and last at corner: Vref, SWCLK, GND, SWDIO, N_RESET.
Vref selects the voltage level used by the programmer. It is connected directly to VDDH. This means that this pin also can be used to power the board. However as 1.8v VDD is used internally it should be selected as Vref, although VDDH seems to work as well. To simplify this setup a small board is available. Using jumpers the board can be configured to select Vref and voltage to board Vref. See below for schematics and example setup.

![bild](https://github.com/seerelectronics/ble_tag/assets/61621920/752427a9-33f1-435b-8db5-f5405943a3ad)

![bild](https://github.com/seerelectronics/ble_tag/assets/61621920/fe332da2-648e-4ee0-98c0-fce6769b22ec)

## CR2032 board

New board with CR2032 battery holder should not be powered from programming card, always use battery. Programming otherwise works as before.

The internal DC/DC regulators should be disabled for this board, meaning that the following functions in power_management_init should be changed:

    nrf_power_dcdcen_vddh_set(false);
    nrf_power_dcdcen_set(false);


## CR2032 board v2

Second version of CR2032 board has corrections done on power supply. This version can be identified but looking at programming header which is moved to opposite side of battery holder. On v1 the header is located next to the holder. Internal voltage is now 1.8v, if using programming card for power Vout must be set to 1.8v. Vref must also be set to 1.8v.

Internal DC/DC regulator should be enabled for this board, however vddh regulator should be disabled as VDD and VDDH are both connected to 1.8v.

## GATTs

Base UUID: 00000001-8334-43-97-af7e-bfca78d70c67

This id must only be used for testing.

* SENSOR_READ 0x0002: When notify is active XYZ samples are sent. As each component is 10 bits the data is compressed so that each sample fits in a 4 byte word. The format is: 00<10 bit X><10 bit Y><10 bit Z>.

* STEP_COUNTER 0x0003: When notify is active step counter is sent as a 16 bit value

* ODR 0x0004: Set data rate, 8 => 1Hz, 9 => 12.5 Hz, 10 => 25 Hz. Default is 25 Hz. See ODR register in datasheet for more information.

* ODR_AVG 0x0005: Current does nothing

* FS 0x0006: Full scale output, 0 => 2g, 1 => 16g, 2 => 4g, 3 => 8g. 2g is default. See FS register in datasheet for more information.

* HP_FILTER 0x0007: Set high pass filter (FDS_SLOPE bit), 1 => on, 0 => off

* STEP_MINTHS 0x0008: Min threshold, 6 bits

* STEP_DEBOUNCE 0x0009: Debounce time/threshold, byte value is written in PEDO_DEB_REG register.

* STEP_COUNT_DELTA 0x000: Byte written in STEP_COUNT_DELTA register.

Step counter is also set as manufacturer data in advertisement message, using the format StepLow-StepHigh-00-00.
