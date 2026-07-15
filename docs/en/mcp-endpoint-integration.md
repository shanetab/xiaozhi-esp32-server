# MCP Endpoint Usage Guide

This tutorial uses the mcp calculator function from Xiege's open source as an example to introduce how to connect your custom mcp service to your endpoint.

The premise of this tutorial is that your `xiaozhi-server` has enabled the mcp endpoint function. If you haven't enabled it yet, you can first follow [this tutorial](./mcp-endpoint-enable.md) to enable it.

# How to Add a Simple MCP Function, Such as Calculator Function, to an Agent

### If You Are Using Full-Module Deployment
If you are using full-module deployment, you can enter the Smart Console, Agent Management, click `Configure Role`, and on the right side of `Intent Recognition`, there is an `Edit Function` button.

Click this button. In the pop-up page, at the bottom, there will be `MCP Endpoint`. Normally, it will display the `MCP Endpoint Address` for this agent. Next, we will extend a calculator function based on MCP technology for this agent.

This `MCP Endpoint Address` is very important, you will use it shortly.

### If You Are Using Single-Module Deployment
If you are using single-module deployment and have already configured the MCP endpoint address in the configuration file, then normally, when the single-module deployment starts, it will output logs similar to the following:
```
250705[__main__]-INFO-Initializing component: vad successfully SileroVAD
250705[__main__]-INFO-Initializing component: asr successfully FunASRServer
250705[__main__]-INFO-OTA interface is          http://192.168.1.25:8002/xiaozhi/ota/
250705[__main__]-INFO-Vision analysis interface is     http://192.168.1.25:8002/mcp/vision/explain
250705[__main__]-INFO-MCP endpoint is        ws://192.168.1.25:8004/mcp_endpoint/mcp/?token=abc
250705[__main__]-INFO-Websocket address is    ws://192.168.1.25:8000/xiaozhi/v1/
250705[__main__]-INFO-=======The above address is a websocket protocol address, do not access with browser=======
250705[__main__]-INFO-To test websocket, start digital-human module and open browser interactive test
250705[__main__]-INFO-=============================================================
```

As shown above, the `ws://192.168.1.25:8004/mcp_endpoint/mcp/?token=abc` in the `MCP endpoint is` output is your `MCP Endpoint Address`.

This `MCP Endpoint Address` is very important, you will use it shortly.

## Step 1: Download Xiege's MCP Calculator Project Code

Open the calculator project written by Xiege in your browser: [calculator project](https://github.com/78/mcp-calculator),

After opening, find the green button labeled `Code` on the page, click it, and you will see the `Download ZIP` button.

Click it to download the project source code zip package. After downloading to your computer, extract it. At this point, its name might be called `mcp-calculator-main`.

You need to rename it to `mcp-calculator`. Next, we use the command line to enter the project directory and install dependencies.

```bash
# Enter project directory
cd mcp-calculator

conda remove -n mcp-calculator --all -y
conda create -n mcp-calculator python=3.10 -y
conda activate mcp-calculator

pip install -r requirements.txt
```

## Step 2: Start

Before starting, first copy the MCP endpoint address from your Smart Console agent.

For example, my agent's mcp address is:
```
ws://192.168.1.25:8004/mcp_endpoint/mcp/?token=abc
```

Start by entering the command:
```bash
export MCP_ENDPOINT=ws://192.168.1.25:8004/mcp_endpoint/mcp/?token=abc
```

After entering, start the program:
```bash
python mcp_pipe.py calculator.py
```

### If You Are Using Smart Console Deployment
If you are using Smart Console deployment, after starting, you need to go back to the Smart Console and click Refresh MCP Connection Status. You will then see the list of extended functions.

### If You Are Using Single-Module Deployment
If you are using single-module deployment, after the device connects, it will output logs similar to the following, indicating success:
```
250705 -INFO-Initializing MCP endpoint: wss://2662r3426b.vicp.fun/mcp_e 
250705 -INFO-Sending MCP endpoint initialization message
250705 -INFO-MCP endpoint connection successful
250705 -INFO-MCP endpoint initialization successful
250705 -INFO-Unified tool processor initialization complete
250705 -INFO-MCP endpoint server information: name=Calculator, version=1.9.4
250705 -INFO-MCP endpoint supported tool count: 1
250705 -INFO-All MCP endpoint tools acquired, client ready
250705 -INFO-Tool cache refreshed
250705 -INFO-Currently supported function list: [ 'get_time', 'get_lunar', 'play_music', 'get_weather', 'handle_exit_intent', 'calculator']
```

If `'calculator'` is included, it means the device can call the calculator tool based on intent recognition.

> [!NOTE] 
> This translation requires manual review as it involves technical documentation that may benefit from human verification for accuracy.