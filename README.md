# ESP32 Homekit Multi Sensor

## What is it?

This is a simple ESP32 WIFI connected temperature, humidity, AQI, and light sensor based on the BME680 and an LDR. It can also fall back to a DHT22 with no AQI readings. It uses the ESP32 Homekit SDK to provide multiple sensors for HomeKit. There is battery support but I have disabled it as I am not using a battery.

## How this Works

There is one oddity with the Espressif homekit implementation - it does not automatically connect to the WIFI network like every other Homekit devices. Most homekit devices use Bluetooth to transfer over the WIFI credentials. The ESP version does not. This means the WIFI creds must either be hard coded into the device or the user must first use the Espressif BLE or SoftAP provisioning app on a iPhone. However, this also means that the HomeKit configuration is much faster as the WIFI network is already setup. 

The light sensor is just an LDR that caps out at 100 lux, so the effective extent of its measurements is dark room, indoor lighting, and sun.

The code configures the following services that make up the accessory:
- Temperature sensor
- Humidity Sensor
- AQI Sensor
- Light Sensor
- Battery level

It borrows the from the [LIPO Battery Capacity code](https://github.com/G6EJD/LiPo_Battery_Capacity_Estimator).

This version is migrated to ESP-IDF 5.0 with the new ADC driver. It uses BSEC 2.4.0.0 and the latest version of the BME68X sensor api as of 05/30/2023.

esptool.py --chip esp32 p com7 erase_flash

## Setting up the Device

This project is based on the ESP-IDF.

Make sure to run the updatemodules.sh script if you did not get the code recursively. It has a dependancy on the Espressif HomeKit SDK. There is a conflict with the button library between the HomeKit SDK and the ESP-IDF-LIB! Remove the button folder in the ESP-IDF-LIB to avoid this issue.

First, configure idf with menuconfig:

```bash
idf.py menuconfig
```

Make sure to set the GPIO pins to what you intend to use. Also, use the hard coded Homekit code, but make sure there are no duplicates on your network. If you use the WIFI Autoprovisioning (bluetooth version is recommended), this the Apple App Store and search for the Espressif BLE Provisioning tool. This will be required to configure WIFI. Some pins and settings are now also set with macros at the top of the file.

Flash and run the monitor:

```bash
idf.py flash
idf.py monitor
```

The monitor is required for the first time to get the QR codes for WIFI provisioning and Homekit setup.

With the monitor running, you will get a screen full of data and two QR codes. The first QR code is for homekit, and the second is for the Espressif provisioning tool. Open the Espressif BLE provisioning tool, and press the Provision Device button. You will be asked to scan the second QR code (provisioning QR code). Next, enter your WIFI password. Watch the monitor. If will indicate if provisioning was succcesful as will the app. If you mess up the password, hold the boot button down for 10 seconds to wipe the configuration, and start over. Once the WIFI is working, open the Homekit app on your iPhone, and add an accessory. Scan the first QR code with Home app, and it should find the GarageDoor accessory. At this point, it's fairly quick to add the accessory. Because of all the different sensors in this device, it's a good idea to use "separate tiles" to display them. Open the setting for the garage door item in the Home app, and select the separate files item.

That is it. You can now use the Home app or Siri to control the device. The default names are ESP "item name". You can change them on your Home app.

## Resetting the ESP device

There is an button handler tied to GPIO0 which on most devices is attached to the boot button. Holding the button has two effects:
- hold for 3 seconds to reset the WIFI setup
- hold for 10 seconds to wipe the WIFI and homekit config and start over (factory default)

For a reset, one does not necessary need the monitor. You can manually add the WIFI password and manually add the device to homekit....but the monitor makes things easier.
