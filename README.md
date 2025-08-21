![OpenShrooly logo](https://github.com/user-attachments/assets/9fa4725c-00c0-42ce-ba5d-a9dcd2bbc7c9)


**OpenShrooly** is a very early experimental replacement firmware for the Shrooly mushroom growing device.  
The intent is to convert the Shrooly into a **Wi-Fi–only device**, controllable from a local web interface or integrated with [Home Assistant](https://www.home-assistant.io/).

Presently this is only a proof of concept. It can control the lights/fans and monitor the temperature and humidity but it's not ready for actually growing mushrooms with.

⚠️ **Warning:** Use at your own risk! This project is experimental and may brick your device.

---

## Prerequisites

Before flashing, install the following tool:

- **esptool.py** (for backing up and flashing):  
  [esptool documentation](https://docs.espressif.com/projects/esptool/en/latest/)  
  Install via pip:  
  ```bash
  pip install esptool
  ```

---

## Backing Up Your Shrooly Firmware

These steps assume Linux. On Windows your port may look like `COM3`, on macOS it may look like `/dev/cu.usbserial...`.

1. **Check flash size**  
   Connect your Shrooly to USB and run:
   ```bash
   esptool --port /dev/ttyACM0 flash_id
   ```
   Expected output:
   ```
   Detected flash size: 16MB
   ```
   If you see anything else, proceed with caution — you may have a hardware variant.

2. **Backup the full flash**  
   Dump the entire 16MB flash to a file:
   ```bash
   esptool --port /dev/ttyACM0 read_flash 0x000000 0x1000000 shrooly_backup.bin
   ```
   This will take some time and produce a 16,777,216-byte file.

   ✅ **Keep this backup safe!**  
   - You’ll need it to restore the original firmware.  
   - It contains your device serial number and Wi-Fi config — don’t share it without sanitizing.

---

## Flashing the Experimental Firmware

1. **Download the firmware**  
   Get the latest development build:  
   [OpenShrooly dev-latest release](https://github.com/grahamsz/openshrooly/releases/tag/dev-latest)

   Download `shrooly.factory.bin` (The `shrooly.ota.bin` is only useful if you are runnnig an earlier OpenShrooly build).

2. **Flash with esptool**  
   Example:
   ```bash
   esptool --chip esp32s3 --port /dev/ttyACM0 --baud 460800 write_flash -z 0x0 shrooly.factory.bin
   ```

3. **First boot**  
   - After flashing, the device will create a Wi-Fi access point:  
     **SSID:** `OpenShrooly`  
   - Connect to this AP and browse to http://192.168.4.1 and you can enter your home WiFi details
   - (Sometimes ESP home boots up and fails to start it's DHCP server, power cycling sometimes helpes here or you can connect with a static IP like 192.168.4.2)
   - Once configured, it will reboot and join your network. 
   - Access the built-in web interface at the Shrooly’s IP on your LAN.  You might be able to use `http://shrooly` or `http://shrooly.local` if mDns works on your home network, otherwise you might have to look in your router's DHCP allocations to find the address

<img width="975" height="1403" alt="image" src="https://github.com/user-attachments/assets/63dbbed0-dc08-4b18-aca1-c779fbf2dcf4" />

---

## Next development steps

- Allow set up of custom cultivation program with on/off times, target humidity, custom light levels
- Tune a [PID](https://en.wikipedia.org/wiki/Proportional%E2%80%93integral%E2%80%93derivative_controller) to allow more stable humidity control than the Shrooly does currently. (I expect this will reduce condensation)
- Provide instructions for integrating with Home Assistant, which in turn would allow the use of a smart plug and heating mat to maintain a stable temperature.
- Figure out how to port across the existing cultivation programs that shrooly has
- Maybe make the water sensor work. I have some notes about how it communicates with I2C but haven't been able to figure out why ESPHome isn't working


## Things I don't plan to do

 - Build a physical interface to allow control via the buttons and eInk screen. This is hypothetically possible - we'd need to figure out the pins for the SPI eInk interface and then ESPHome should be quite able to drive it
 - Anything with bluetooth or the original Shrooly app

---

## Restoring the Original Firmware

If you need to roll back to the stock Shrooly software:

1. **Ensure you have your backup file** (`shrooly_backup.bin`).  
   This was created in the backup step above.

2. **Flash the backup back onto the device**  
   ```bash
   esptool --chip esp32s3 --port /dev/ttyACM0 --baud 460800 write_flash -z 0x0 shrooly_backup.bin
   ```

3. **Reboot**  
   After flashing, disconnect and power cycle the device. It should boot back into the original factory firmware.
