![OpenShrooly logo](https://github.com/user-attachments/assets/9fa4725c-00c0-42ce-ba5d-a9dcd2bbc7c9)


**OpenShrooly** is a very early experimental replacement firmware for the Shrooly mushroom growing device.  
The intent is to convert the Shrooly into a **Wi-Fi–only device**, controllable from a local web interface or integrated with [Home Assistant](https://www.home-assistant.io/).

Presently this is only a proof of concept. It can control the lights/fans and monitor the temperature and humidity but it's not ready for actually growing mushrooms with.

⚠️ **Warning:** Use at your own risk! This project is experimental and may brick your device.

---

## Prerequisites

Before flashing, install the following tools:

- **ESPHome** (for building firmware):  
  [ESPHome installation guide](https://esphome.io/guides/installing_esphome.html)

- **Home Assistant** (optional, for integration):  
  [Home Assistant installation guide](https://www.home-assistant.io/installation/)

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
     **SSID:** `Shrooly Setup`  
   - Connect to this AP and a **captive portal** will let you enter your home Wi-Fi details.  
   - Once configured, it will reboot and join your network.

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

---

## Next Steps

- Access the built-in web interface at the Shrooly’s IP on your LAN.  You might be able to use `http://shrooly` or `http://shrooly.local` if mDns works on your home network, otherwise you might have to look in your router's DHCP allocationst o find the address
- Experiment with sensors, lights. Automation is coming soon
