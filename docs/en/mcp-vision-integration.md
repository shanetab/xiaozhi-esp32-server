# Vision Model Usage Guide

This tutorial is divided into two parts:
- Part 1: Running xiaozhi-server in single-module mode with vision model enabled
- Part 2: How to enable the vision model in full-module mode

Before enabling the vision model, you need to prepare three things:
- You need to prepare a device with a camera, and this device has implemented camera calling functionality in the Xiaoge repository. For example, `Lichuang·Practical ESP32-S3 Development Board`
- Upgrade your device firmware to version 1.6.6 or higher
- You have successfully run the basic conversation module

## Running xiaozhi-server in Single-Module Mode with Vision Model

### Step 1: Confirm Network Settings
Since the vision model will start on port 8003 by default.

If you are running with Docker, please confirm that your `docker-compose.yml` file maps port `8003`. If not, update to the latest `docker-compose.yml` file.

If you are running from source code, confirm that the firewall allows port `8003`.

### Step 2: Select Your Vision Model
Open your `data/.config.yaml` file and set your `selected_module.VLLM` to a vision model. Currently, we support vision models that are compatible with the `openai` interface. `ChatGLMVLLM` is one such model compatible with `openai`.

```
selected_module:
  VAD: ..
  ASR: ..
  LLM: ..
  VLLM: ChatGLMVLLM
  TTS: ..
  Memory: ..
  Intent: ..
```

Assuming we use `ChatGLMVLLM` as the vision model, we need to first log into the [Zhipu AI](https://bigmodel.cn/usercenter/proj-mgmt/apikeys) website to apply for an API key. If you have already applied for a key before, you can reuse it.

Add this configuration to your config file. If it already exists, just set your api_key.

```
VLLM:
  ChatGLMVLLM:
    api_key: your_api_key
```

### Step 3: Start the xiaozhi-server Service
If running from source code, start with the command:
```
python app.py
```
If running with Docker, restart the container:
```
docker restart xiaozhi-esp32-server
```

After starting, the following log will be output:

```
2025-06-01 **** - OTA interface is           http://192.168.4.7:8003/xiaozhi/ota/
2025-06-01 **** - Vision analysis interface is        http://192.168.4.7:8003/mcp/vision/explain
2025-06-01 **** - Websocket address is       ws://192.168.4.7:8000/xiaozhi/v1/
2025-06-01 **** - =======The above address is a websocket protocol address, do not access with browser=======
2025-06-01 **** - To test websocket, start digital-human module and open browser interactive test
2025-06-01 **** - =============================================================
```

After starting, use a browser to open the `vision analysis interface` link shown in the logs. What does it display? If you're on Linux without a browser, you can execute this command:
```
curl -i your vision analysis interface
```

Normally, it will display:
```
MCP Vision interface is running normally, vision explanation interface address is: http://xxxx:8003/mcp/vision/explain
```

Please note, if you are deploying on a public network or with Docker, you must modify this configuration in your `data/.config.yaml` file:
```
server:
  vision_explain: http://your ip or domain:port/mcp/vision/explain
```

Why? Because the vision explanation interface needs to be sent to the device. If your address is a LAN address or Docker internal address, the device cannot access it.

Assuming your public address is `111.111.111.111`, then `vision_explain` should be configured as:
```
server:
  vision_explain: http://111.111.111.111:8003/mcp/vision/explain
```

If your MCP Vision interface is running normally and you have successfully accessed the `vision explanation interface address` using a browser, proceed to the next step.

### Step 4: Wake Up the Device
Say to the device: "Please open the camera and tell me what you see"

Pay attention to the xiaozhi-server log output to see if there are any errors.

## Enabling Vision Model in Full-Module Mode

### Step 1: Confirm Network Settings
Since the vision model will start on port 8003 by default.

If you are running with Docker, please confirm that your `docker-compose_all.yml` file maps port `8003`. If not, update to the latest `docker-compose_all.yml` file.

If you are running from source code, confirm that the firewall allows port `8003`.

### Step 2: Confirm Your Configuration File
Open your `data/.config.yaml` file and confirm that your configuration file structure matches `data/config_from_api.yaml`. If it differs or is missing items, please complete it.

### Step 3: Configure Vision Model API Key
We need to first log into the [Zhipu AI](https://bigmodel.cn/usercenter/proj-mgmt/apikeys) website to apply for an API key. If you have already applied for a key before, you can reuse it.

Log into the `Smart Console`, click `Model Configuration` in the top menu, then click `Vision-to-Language Model` in the left sidebar, find `VLLM_ChatGLMVLLM`, click the Modify button, and in the pop-up, enter your API key in the `API Key` field, then click Save.

After saving successfully, go to the agent you want to test, click `Configure Role`, and in the opened content, check if `Vision Large Language Model (VLLM)` has selected the vision model you just configured. Click Save.

### Step 4: Start the xiaozhi-server Module
If running from source code, start with the command:
```
python app.py
```
If running with Docker, restart the container:
```
docker restart xiaozhi-esp32-server
```

After starting, the following log will be output:

```
2025-06-01 **** - Vision analysis interface is        http://192.168.4.7:8003/mcp/vision/explain
2025-06-01 **** - Websocket address is       ws://192.168.4.7:8000/xiaozhi/v1/
2025-06-01 **** - =======The above address is a websocket protocol address, do not access with browser=======
2025-06-01 **** - To test websocket, start digital-human module and open browser interactive test
2025-06-01 **** - =============================================================
```

After starting, use a browser to open the `vision analysis interface` link shown in the logs. What does it display? If you're on Linux without a browser, you can execute this command:
```
curl -i your vision analysis interface
```

Normally, it will display:
```
MCP Vision interface is running normally, vision explanation interface address is: http://xxxx:8003/mcp/vision/explain
```

Please note, if you are deploying on a public network or with Docker, you must modify this configuration in your `data/.config.yaml` file:
```
server:
  vision_explain: http://your ip or domain:port/mcp/vision/explain
```

Why? Because the vision explanation interface needs to be sent to the device. If your address is a LAN address or Docker internal address, the device cannot access it.

Assuming your public address is `111.111.111.111`, then `vision_explain` should be configured as:
```
server:
  vision_explain: http://111.111.111.111:8003/mcp/vision/explain
```

If your MCP Vision interface is running normally and you have successfully accessed the `vision explanation interface address` using a browser, proceed to the next step.

### Step 5: Wake Up the Device
Say to the device: "Please open the camera and tell me what you see"

Pay attention to the xiaozhi-server log output to see if there are any errors.

> [!NOTE] 
> This translation requires manual review as it involves technical documentation that may benefit from human verification for accuracy.