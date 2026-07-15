# MCP Endpoint Deployment and Usage Guide

This tutorial includes 3 sections:
- 1. How to deploy the MCP endpoint service
- 2. How to configure the MCP endpoint in full-module deployment
- 3. How to configure the MCP endpoint in single-module deployment

# 1. How to Deploy the MCP Endpoint Service

## Step 1: Download the MCP Endpoint Project Source Code

Open the [MCP endpoint project address](https://github.com/xinnan-tech/mcp-endpoint-server) in your browser.

After opening, find the green button labeled `Code` on the page, click it, and you will see the `Download ZIP` button.

Click it to download the project source code zip package. After downloading to your computer, extract it. At this point, its name might be called `mcp-endpoint-server-main`.

You need to rename it to `mcp-endpoint-server`.

## Step 2: Start the Program

This is a very simple project, recommended to run with Docker. However, if you don't want to use Docker to run it, you can refer to [this page](https://github.com/xinnan-tech/mcp-endpoint-server/blob/main/README_dev.md) to run with source code. The following is the Docker running method:

```
# Enter the root directory of this project source code
cd mcp-endpoint-server

# Clear cache
docker compose -f docker-compose.yml down
docker stop mcp-endpoint-server
docker rm mcp-endpoint-server
docker rmi ghcr.nju.edu.cn/xinnan-tech/mcp-endpoint-server:latest

# Start Docker container
docker compose -f docker-compose.yml up -d
# View logs
docker logs -f mcp-endpoint-server
```

At this point, the logs will output logs similar to the following:
```
250705 INFO-=====The following addresses are the Smart Console/Single Module MCP endpoint addresses====
250705 INFO-Smart Console MCP parameter configuration: http://172.22.0.2:8004/mcp_endpoint/health?key=abc
250705 INFO-Single module deployment MCP endpoint: ws://172.22.0.2:8004/mcp_endpoint/mcp/?token=def
250705 INFO-=====Please select the appropriate deployment for use, do not disclose to anyone======
```

Please copy both interface addresses:
 
Since you deployed with Docker, you must not directly use the addresses above!

Since you deployed with Docker, you must not directly use the addresses above!

Since you deployed with Docker, you must not directly use the addresses above!

First, copy the addresses to a draft, and you need to know what your computer's LAN IP is. For example, my computer's LAN IP is `192.168.1.25`, then
the original interface addresses
```
Smart Console MCP parameter configuration: http://172.22.0.2:8004/mcp_endpoint/health?key=abc
Single module deployment MCP endpoint: ws://172.22.0.2:8004/mcp_endpoint/mcp/?token=def
```
need to be changed to
```
Smart Console MCP parameter configuration: http://192.168.1.25:8004/mcp_endpoint/health?key=abc
Single module deployment MCP endpoint: ws://192.168.1.25:8004/mcp_endpoint/mcp/?token=def
```

After making the changes, use your browser to directly access the `Smart Console MCP parameter configuration`. When the browser shows code similar to the following, it indicates success.
```
{"result":{"status":"success","connections":{"tool_connections":0,"robot_connections":0,"total_connections":0}},"error":null,"id":null,"jsonrpc":"2.0"}
```

Please keep the two `interface addresses` above, they will be needed in the next step.

# 2. How to Configure the MCP Endpoint in Full-Module Deployment

First, you need to enable the MCP endpoint function. In the Smart Console, click the top `Parameter Dictionary`, then in the dropdown menu, click the `System Function Configuration` page. On the page, check `MCP Endpoint`, then click `Save Configuration`. In the `Role Configuration` page, click the `Edit Function` button to see the `MCP Endpoint` function.

If you are using full-module deployment, use administrator account to log into the Smart Console, click the top `Parameter Dictionary`, and select the `Parameter Management` function.

Then search for parameter `server.mcp_endpoint`, its value should currently be `null`.
Click the Modify button, paste the `Smart Console MCP parameter configuration` obtained from the previous step into the `Parameter Value`. Then save.

If saving is successful, everything is going well, and you can check the effect in the agent. If not successful, it means the Smart Console cannot access the MCP endpoint, most likely due to firewall or incorrect LAN IP.

# 3. How to Configure the MCP Endpoint in Single-Module Deployment

If you are using single-module deployment, find your configuration file `data/.config.yaml`.

Search for `mcp_endpoint` in the configuration file. If not found, add the `mcp_endpoint` configuration. For example, mine looks like this:
```
server:
  websocket: ws://your ip or domain:port/xiaozhi/v1/
  http_port: 8002
log:
  log_level: INFO

# More configurations may exist here...

mcp_endpoint: your endpoint websocket address
```

At this point, paste the `Single Module Deployment MCP Endpoint` obtained from the "How to Deploy the MCP Endpoint Service" section into `mcp_endpoint`. For example:
```
server:
  websocket: ws://your ip or domain:port/xiaozhi/v1/
  http_port: 8002
log:
  log_level: INFO

# More configurations may exist here

mcp_endpoint: ws://192.168.1.25:8004/mcp_endpoint/mcp/?token=def
```

After configuration, starting the single module will output logs similar to the following:
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

As shown above, if it outputs similar `MCP endpoint is` with `ws://192.168.1.25:8004/mcp_endpoint/mcp/?token=abc`, it indicates successful configuration.

> [!NOTE] 
> This translation requires manual review as it involves technical documentation that may benefit from human verification for accuracy.