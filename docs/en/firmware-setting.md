# Configuring Custom Server Based on Xiege's Pre-compiled Firmware

## Step 1: Confirm Version
Burn the [version 1.6.1 or above firmware](https://github.com/78/xiaozhi-esp32/releases) that has been pre-compiled by Xiege.

## Step 2: Prepare Your OTA Address
If you followed the tutorial and are using full module deployment, you should have an OTA address.

At this point, please open your OTA address with a browser, for example, my OTA address:
```
https://2662r3426b.vicp.fun/xiaozhi/ota/
```

If it displays "OTA interface is running normally, websocket cluster count: X". Then proceed to the next step.

If it displays "OTA interface is not running normally", it's likely that you haven't configured the `Websocket` address in the `Smart Console`. Then:

- 1. Log into the Smart Console with super administrator privileges
- 2. Click `Parameter Management` in the top menu
- 3. In the list, find the `server.websocket` item and enter your `Websocket` address. For example, mine is:
```
wss://2662r3426b.vicp.fun/xiaozhi/v1/
```

After configuration, refresh your OTA interface address with the browser to see if it's normal. If not, confirm again whether the Websocket is starting normally and whether the Websocket address is configured.

## Step 3: Enter Network Configuration Mode
Enter the device's network configuration mode. At the top of the page, click "Advanced Options", then enter your server's `ota` address, and click Save. Restart the device.

![Refer to OTA Address Setting](../docs/images/firmware-setting-ota.png)

## Step 4: Wake Up Xiao Zhi and Check Log Output

Wake up Xiao Zhi and check if the log is outputting normally.

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