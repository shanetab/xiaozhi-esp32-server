# Xiao Zhi ESP32 - Open Source Server and HomeAssistant Integration Guide

[TOC]

-----

## Introduction

This document will guide you on how to integrate the ESP32 device with HomeAssistant.

## Prerequisites

- `HomeAssistant` is installed and configured
- The model I chose this time is: the free ChatGLM, which supports function call function invocation

## Pre-operations (Required)

### 1. Get HA Network Address Information

Please visit your Home Assistant network address, for example, my HA address is 192.168.4.7, and the port is the default 8123, then open in the browser:

```
http://192.168.4.7:8123
```

> Manual method to query HA's IP address** (only when Xiao Zhi esp32-server and HA are deployed on the same network device [such as the same WiFi])**:
>
> 1. Enter Home Assistant (frontend).
>
> 2. Click the bottom-left **Settings** → **System** → **Network**.
>
> 3. Scroll to the bottom `Home Assistant Website` area, in `Local Network`, click the `eye` button to see the current IP address being used (such as `192.168.1.10`) and network interface. Click `Copy Link` to copy directly.
>
>    ![image-20250504051716417](images/image-ha-integration-01.png)

Or, if you have set up a directly accessible Home Assistant OAuth address, you can also access it directly in the browser:

```
http://homeassistant.local:8123
```

### 2. Log into `Home Assistant` to Get the Development Key

Log into `HomeAssistant`, click on the bottom-left avatar → Profile, switch to the Security navigation bar, scroll to the bottom `Long-Lived Access Token` to generate the api_key, and copy and save it. The following methods will all need to use this api key, which only appears once (small tip: you can save the generated QR code image and scan it later to retrieve the api key).

## Method 1: Xiao Zhi Community-Contributed HA Calling Function

### Function Description

- If you need to add new devices in the future, this method requires manually restarting the `xiaozhi-esp32-server` server-side to update device information **(important)**.
- You need to ensure that `Xiaomi Home` has been integrated in HomeAssistant and that MiHome devices have been imported into `HomeAssistant`.
- You need to ensure that `xiaozhi-esp32-server Smart Console` can function normally.
- My `xiaozhi-esp32-server Smart Console` and `HomeAssistant` are deployed on the same machine on a different port, version is `0.3.10`:
  ```
  http://192.168.4.7:8002
  ```

### Configuration Steps

#### 1. Log into `HomeAssistant` to Organize the List of Devices to Control

Log into `HomeAssistant`, click on the bottom-left Settings, then enter `Devices & Services`, and click on `Entities` at the top.

Then search for switches related to your control in the entities list. After the results appear, click on one result in the list. A switch interface will appear.

Try clicking the switch to see if the development will follow our clicks to turn on/off. If it can be operated, it means it's connected to the network.

Next, find the settings button in the switch panel, click it to view the switch's `Entity Identifier`.

We open a notepad and organize the data in this format:

Location + comma + device name + comma + `Entity Identifier` + semicolon

For example, I'm at work, I have a toy lamp, and its identifier is switch.cuco_cn_460494544_cp1_on_p_2_1, so I write this line of data:

```
Work,Toy Lamp,switch.cuco_cn_460494544_cp1_on_p_2_1;
```

Of course, in the end, I might want to operate two lamps, so my final result is:

```
Work,Toy Lamp,switch.cuco_cn_460494544_cp1_on_p_2_1;
Work,Desk Lamp,switch.iot_cn_831898993_socn1_on_p_2_1;
```

This string of characters is called the "Device List Character" and should be saved for later use.

#### 2. Log into `Smart Console`

![image-20250504051716417](images/image-ha-integration-06.png)

Using administrator account, log into `Smart Console`. In `Agent Management`, find your agent, then click `Configure Role`.

Set the intent recognition to `External Large Model Intent Recognition` or `Large Model Autonomous Function Call`. At this point you will see a `Edit Function` on the right. Click the `Edit Function` button, and a `Function Management` box will pop up.

In the `Function Management` box, you need to check `HomeAssistant Device Status Query` and `HomeAssistant Device Status Modification`.

After checking, click on `HomeAssistant Device Status Query` in `Selected Functions`, then in `Parameter Configuration` configure your `HomeAssistant` address, key, and device list character.

After editing, click `Save Configuration`. At this point, the `Function Management` box will hide, and then you click save the agent configuration.

After saving successfully, you can wake up the device for operation.

#### 3. Wake up Device for Control

Try saying to the esp32, "Turn on XXX lamp"

## Method 2: Using Home Assistant's Voice Assistant as an LLM Tool

### Function Description

- This method has a significant disadvantage — **this method cannot use the function_call plugin capabilities of the Xiao Zhi open source ecosystem**, because using Home Assistant as Xiao Zhi's LLM tool will transfer the intent recognition capability to Home Assistant. However, **this method allows you to experience the native Home Assistant operation function, and Xiao Zhi's chat capability remains unchanged**. If you really don't mind, you can use [Method 3](##Method-3:-Using-Home-Assistant's-MCP-Service-(Recommended)), which supports the Home Assistant functions the most.

### Configuration Steps:

#### 1. Configure Home Assistant's Large Model Voice Assistant

**You need to configure Home Assistant's voice assistant or large model tool in advance.**

#### 2. Get Home Assistant's Language Assistant's Agent ID

1. Enter the Home Assistant page. Click on the left `Developer Tools`.
2. In the opened `Developer Tools`, click the `Actions` tab (as shown in operation 1), in the option bar `Actions`, find or enter `conversation.process (Conversation - Process)` and select `Conversation: Process` (as shown in operation 2).

![image-20250504043539343](images/image-ha-integration-02.png)

3. In the page, check the `Agent` option, in the `Conversation Agent` that becomes illuminated, select the voice assistant name you configured in step one, as shown in the figure. My configured one is `ZhipuAi` and select it.

![image-20250504043854760](images/image-ha-integration-03.png)

4. After selection, click the `Enter YAML Mode` button at the bottom left of the form.

![image-20250504043951126](images/image-ha-integration-04.png)

5. Copy the value of the agent-id, for example, in the figure my is `01JP2DYMBDF7F4ZA2DMCF2AGX2` (for reference only).

![image-20250504044046466](images/image-ha-integration-05.png)

6. Switch to the `config.yaml` file of the Xiao Zhi open source server `xiaozhi-esp32-server`. In the LLM configuration, find Home Assistant, and set your Home Assistant's network address, Api key, and the agent_id just queried.

7. Modify the `selected_module` attribute in `config.yaml` to set the `LLM` to `HomeAssistant` and `Intent` to `nointent`.

8. Restart the Xiao Zhi open source server `xiaozhi-esp32-server` to use normally.

## Method 3: Using Home Assistant's MCP Service (Recommended)

### Function Description

- You need to integrate and install the HA integration in Home Assistant — [Model Context Protocol Server](https://www.home-assistant.io/integrations/mcp_server/) in advance.

- This method, like Method 2, is a solution provided by HA official. Unlike Method 2, you can use Xiao Zhi open source server's `xiaozhi-esp32-server` open source community plugins, and also allow you to use any LLM large model that supports function_call function.

### Configuration Steps

#### 1. Install Home Assistant's MCP Service Integration

Integration official website — [Model Context Protocol Server](https://www.home-assistant.io/integrations/mcp_server/).

Or follow the following manual operations:

> - Go to **Settings > Devices & Services** on the Home Assistant page.
>
> - In the bottom right, select the **[Add Integration](https://my.home-assistant.io/redirect/config_flow_start?domain=mcp_server)** button.
>
> - Select **Model Context Protocol Server** from the list.
>
> - Follow the on-screen instructions to complete the setup.

#### 2. Configure Xiao Zhi Open Source Server MCP Configuration Information

Enter the `data` directory and find the `.mcp_server_settings.json` file.

If there is no `.mcp_server_settings.json` file in your `data` directory:
- Please copy the `mcp_server_settings.json` file from the root directory of the `xiaozhi-server` folder to the `data` directory and rename it to `.mcp_server_settings.json`
- Or [download this file](https://github.com/xinnan-tech/xiaozhi-esp32-server/blob/main/main/xiaozhi-server/mcp_server_settings.json), download it to the `data` directory and rename it to `.mcp_server_settings.json`

Modify the content in this part of `"mcpServers"`:

```json
"Home Assistant": {
      "command": "mcp-proxy",
      "args": [
        "http://YOUR_HA_HOST/mcp_server/sse"
      ],
      "env": {
        "API_ACCESS_TOKEN": "YOUR_API_ACCESS_TOKEN"
      }
},
```

Note:

1. **Replace Configuration:**
   - Replace `YOUR_HA_HOST` in `args` with your HA service address. If your service address already includes https/http (for example, `http://192.168.1.101:8123`), then just fill in `192.168.1.101:8123`.
   - Replace `YOUR_API_ACCESS_TOKEN` in `env` with the api key you obtained earlier.
2. **If you are adding the configuration within the `mcpServers` brackets and there are no new `mcpServers` configurations afterward, you need to remove the trailing comma `,`**, otherwise it might fail to parse.

**Final effect reference (refer below):**

```json
 "mcpServers": {
    "Home Assistant": {
      "command": "mcp-proxy",
      "args": [
        "http://192.168.1.101:8123/mcp_server/sse"
      ],
      "env": {
        "API_ACCESS_TOKEN": "abcd.efghi.jkl"
      }
    }
  }
```

#### 3. Configure Xiao Zhi Open Source Server System Configuration

1. **Select any LLM large model that supports function_call as Xiao Zhi's LLM chat assistant (but don't choose Home Assistant as the LLM tool)**. This time I chose: the free ChatGLM, which supports functioncall function invocation, but sometimes calls are unstable. If you prefer stability, recommend setting the LLM to: DoubaoLLM, using the specific model_name: doubao-1-5-pro-32k-250115.

2. Switch to the `config.yaml` file of the Xiao Zhi open source server `xiaozhi-esp32-server`, set your LLM large model configuration, and adjust the `Intent` in `selected_module` configuration to `function_call`.

3. Restart the Xiao Zhi open source server `xiaozhi-esp32-server` to use normally.

> [!NOTE] 
> This translation requires manual review as it involves technical documentation that may benefit from human verification for accuracy.