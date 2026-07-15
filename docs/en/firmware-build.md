# ESP32 Firmware Compilation

## Step 1: Prepare Your OTA Address

If you are using version 0.3.12 of this project, whether using simple Server deployment or full module deployment, you will have an OTA address.

Since the OTA address setting methods differ between simple Server deployment and full module deployment, please choose the appropriate method below:

### If You're Using Simple Server Deployment
At this point, please open your OTA address with a browser, for example, my OTA address:
```
http://192.168.1.25:8003/xiaozhi/ota/
```
If it displays "OTA interface is running normally, the websocket address sent to the device is: ws://xxx:8000/xiaozhi/v1/"

You can start the `digital-human` module, then open `index.html` to test if it can connect to the websocket address output by the OTA page.

If you cannot access it, you need to modify the `server.websocket` address in the configuration file `.config.yaml`, restart, and then test again until `index.html` can access normally.

After success, proceed to Step 2.

### If You're Using Full Module Deployment
At this point, please open your OTA address with a browser, for example, my OTA address:
```
http://192.168.1.25:8002/xiaozhi/ota/
```

If it displays "OTA interface is running normally, websocket cluster count: X". Then proceed to Step 2.

If it displays "OTA interface is not running normally", it's likely that you haven't configured the `Websocket` address in the `Smart Console`. Then:

- 1. Log into the Smart Console with super administrator privileges
- 2. Click `Parameter Management` in the top menu
- 3. In the list, find the `server.websocket` item and enter your `Websocket` address. For example, mine is:
```
ws://192.168.1.25:8000/xiaozhi/v1/
```

After configuration, refresh your OTA interface address with the browser to see if it's normal. If not, confirm again whether the Websocket is starting normally and whether the Websocket address is configured.

## Step 2: Configure Environment
First, follow this tutorial to set up the project environment [《Windows Setup ESP IDF 5.3.2 Development Environment and Compile Xiao Zhi》](https://icnynnzcwou8.feishu.cn/wiki/JEYDwTTALi5s2zkGlFGcDiRknXf)

## Step 3: Open Configuration File
After configuring the compilation environment, download the xiaozhi-esp32 project source code from Xiege.

Download xiaozhi-esp32 project source code [here](https://github.com/78/xiaozhi-esp32).

After downloading, open the `xiaozhi-esp32/main/Kconfig.projbuild` file.

## Step 4: Modify OTA Address

Find the `default` content of `OTA_URL` and change `https://api.tenclass.net/xiaozhi/ota/`
to your own address. For example, my interface address is `http://192.168.1.25:8002/xiaozhi/ota/`, so I change the content to this.

Before modification:
```
config OTA_URL
    string "Default OTA URL"
    default "https://api.tenclass.net/xiaozhi/ota/"
    help
        The application will access this URL to check for new firmwares and server address.
```
After modification:
```
config OTA_URL
    string "Default OTA URL"
    default "http://192.168.1.25:8002/xiaozhi/ota/"
    help
        The application will access this URL to check for new firmwares and server address.
```

## Step 5: Set Compilation Parameters

Set compilation parameters:
```
# Terminal command line enters xiaozhi-esp32 root directory
cd xiaozhi-esp32
# For example, I'm using esp32s3 board, so set the compilation target to esp32s3. If your board is another model, replace with the corresponding model
idf.py set-target esp32s3
# Enter menu configuration
idf.py menuconfig
```

After entering menu configuration, go to `Xiaozhi Assistant` and set `BOARD_TYPE` to your board's specific model.
Save and exit, then return to the terminal command line.

## Step 6: Compile Firmware

```
idf.py build
```

## Step 7: Package BIN Firmware

```
cd scripts
python release.py
```

After executing the above packaging command, the firmware file `merged-binary.bin` will be generated in the `build` directory under the project root directory.
This `merged-binary.bin` is the firmware file to be flashed onto the hardware.

Note: If the second command reports "zip" related errors, please ignore this error, as long as the firmware file `merged-binary.bin` is generated in the `build` directory, it won't have much impact on you. Please continue.

## Step 8: Flash Firmware
Connect the ESP32 device to the computer, use Chrome browser to open the following URL:

```
https://espressif.github.io/esp-launchpad/
```

Open this tutorial [Flash Tool/Web Browser Flash Firmware (No IDF Development Environment)](https://ccnphfhqs21z.feishu.cn/wiki/Zpz4wXBtdimBrLk25WdcXzxcnNS).
Scroll to: `Method 2: ESP-Launchpad Browser WEB Flash` starting from `3. Flash Firmware/Download to Development Board`, and follow the tutorial steps.

After successful flashing and network connection, wake up Xiao Zhi with the wake word and pay attention to the console information output by the server side.

## Common Issues
The following are some common issues for reference:

[1. Why does Xiao Zhi recognize many Korean, Japanese, and English words when I speak?](./FAQ.md)

[2. Why does "TTS task error, file not found" appear?](./FAQ.md)

[3. TTS often fails and times out](./FAQ.md)

[4. WiFi can connect to self-built server, but 4G mode cannot connect](./FAQ.md)

[5. How to improve Xiao Zhi's response speed?](./FAQ.md)

[6. I speak slowly, and Xiao Zhi often interrupts me](./FAQ.md)

[7. I want to control lights, air conditioners, remote power on/off through Xiao Zhi](./FAQ.md)

> [!NOTE] 
> This translation requires manual review as it involves technical documentation that may benefit from human verification for accuracy.