# Single-Module Deployment Firmware OTA Auto-Upgrade Configuration Guide

This tutorial will guide you on how to configure the firmware OTA auto-upgrade function in a **single-module deployment** scenario, achieving automatic device firmware updates.

If you have already used **full-module deployment**, please ignore this tutorial.

## Function Introduction

In single-module deployment, xiaozhi-server has built-in OTA firmware management functionality that can automatically detect device versions and issue firmware upgrades. The system will match and push the latest firmware version based on device model and current version.

## Prerequisites

- You have successfully completed **single-module deployment** and are running xiaozhi-server
- The device can connect to the server normally

## Step 1: Prepare Firmware Files

### 1. Create Firmware Storage Directory

Firmware files need to be placed in the `data/bin/` directory. If this directory does not exist, create it manually:

```bash
mkdir -p data/bin
```

### 2. Firmware File Naming Convention

Firmware files must follow the following naming format:

```
{device_model}_{version}.bin
```

**Naming Convention Explanation:**
- `device_model`: The device model name, such as `lichuang-dev`, `bread-compact-wifi`, etc.
- `version`: Firmware version number, must start with a digit, supports digits, letters, periods, underscores, and hyphens, such as `1.6.6`, `2.0.0`, etc.
- File extension must be `.bin`

**Naming Examples:**
```
bread-compact-wifi_1.6.6.bin
lichuang-dev_2.0.0.bin
```

### 3. Place Firmware Files

Copy the prepared firmware files (.bin files) to the `data/bin/` directory:

Important: The upgrade bin file is `xiaozhi.bin`, not the full firmware file `merged-binary.bin!

Important: The upgrade bin file is `xiaozhi.bin`, not the full firmware file `merged-binary.bin!

Important: The upgrade bin file is `xiaozhi.bin`, not the full firmware file `merged-binary.bin! 

```bash
cp xiaozhi.bin data/bin/device_model_version.bin
```

For example:
```bash
cp xiaozhi.bin data/bin/bread-compact-wifi_1.6.6.bin
```

## Step 2: Configure Public Access Address (Only Required for Public Deployment)

**Note: This step is only for single-module public deployment scenarios.**

If your xiaozhi-server is deployed publicly (using a public IP or domain name), **you must** configure the `server.vision_explain` parameter, as the OTA firmware download address will use the domain name and port configured here.

If you are using LAN deployment, you can skip this step.

### Why Configure This Parameter?

In single-module deployment, when the system generates the firmware download address, it uses the domain name and port configured in `vision_explain` as the base address. If not configured or configured incorrectly, the device will not be able to access the firmware download address.

### Configuration Method

Open `data/.config.yaml` file, find the `server` configuration section, and set the `vision_explain` parameter:

```yaml
server:
  vision_explain: http://your domain or IP:port/mcp/vision/explain
```

**Configuration Examples:**

LAN deployment (default):
```yaml
server:
  vision_explain: http://192.168.1.100:8003/mcp/vision/explain
```

Public domain deployment:
```yaml
server:
  vision_explain: http://yourdomain.com:8003/mcp/vision/explain
```

### Important Notes

- The domain name or IP must be an address that the device can access
- If using Docker deployment, do not use Docker internal addresses (such as 127.0.0.1 or localhost)
- If you are using nginx reverse proxy, please fill in the external address and port number, not the port number on which this project runs

## Common Issues

### 1. Device Not Receiving Firmware Updates

**Possible Causes and Solutions:**

- Check if the firmware file naming follows the rules: `{model}_{version}.bin`
- Check if the firmware file is correctly placed in the `data/bin/` directory
- Check if the device model matches the model in the firmware filename
- Check if the firmware version number is higher than the device's current version
- Check server logs to confirm if the OTA request is handled normally

### 2. Device Reports Download Address Not Accessible

**Possible Causes and Solutions:**

- Check if the domain name or IP configured in `server.vision_explain` is correct
- Confirm the port number configuration is correct (default 8003)
- If using public deployment, ensure the device can access this public address
- If using Docker deployment, ensure internal addresses (127.0.0.1) are not used
- Check if the firewall has opened the corresponding ports
- If using nginx reverse proxy, please fill in the external address and port number, not the port number on which this project runs

### 3. How to Confirm Device Current Version

Check OTA request logs, the logs will show the version number reported by the device:

```
[ota_handler] - Device AA:BB:CC:DD:EE:FF firmware is already the latest: 1.6.6
```

### 4. Firmware Files Not Taking Effect After Placement

The system has a 30-second cache time (default), you can:
- Wait 30 seconds and then let the device initiate an OTA request
- Restart the xiaozhi-server service
- Adjust the `firmware_cache_ttl` configuration to a shorter time

> [!NOTE] 
> This translation requires manual review as it involves technical documentation that may benefit from human verification for accuracy.