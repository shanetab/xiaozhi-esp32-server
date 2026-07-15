# MQTT Gateway Deployment Tutorial

The `xiaozhi-esp32-server` project can be simply modified to work with Xiaoge's open-source [xiaozhi-mqtt-gateway](https://github.com/78/xiaozhi-mqtt-gateway) project to achieve MQTT+UDP connection with Xiao Zhi hardware.

This tutorial is divided into three parts. You can choose the corresponding part based on whether you are using full-module or single-module deployment to connect to the MQTT gateway:
- Part 1: Deploying the MQTT Gateway
- Part 2: Full-module operation to achieve MQTT+UDP connection with Xiao Zhi hardware
- Part 3: Single-module operation of xiaozhi-server to achieve MQTT+UDP connection with Xiao Zhi hardware

## Preparation Stage

Prepare your `xiaozhi-server`'s `mqtt-websocket` connection address. Based on your original `websocket address`, add the `?from=mqtt_gateway` character to get the `mqtt-websocket` address.

1. If you are deploying from source code, your `mqtt-websocket` address is:
```
ws://127.0.0.1:8000/xiaozhi/v1/?from=mqtt_gateway
```

2. If you are deploying with Docker, your `mqtt-websocket` address is:
```
ws://your host LAN IP:8000/xiaozhi/v1/?from=mqtt_gateway
```

## Important Notice

If you are deploying on a server, ensure that ports `1883`, `8884`, and `8007` are all open. Port `8884` uses the `UDP` protocol, while the others use `TCP`.

## Part 1: Deploying the MQTT Gateway

1. Clone the [modified xiaozhi-mqtt-gateway project](https://github.com/xinnan-tech/xiaozhi-mqtt-gateway.git):
```bash
git clone https://ghfast.top/https://github.com/xinnan-tech/xiaozhi-mqtt-gateway.git
cd xiaozhi-mqtt-gateway
```

2. Install dependencies:
```bash
npm install
npm install -g pm2
```

3. Configure `config.json`:
```bash
cp config/mqtt.json.example config/mqtt.json
```

4. Edit the configuration file config/mqtt.json and replace the `mqtt-websocket` address you obtained in the "Preparation Stage" with the `chat_servers` entry. For example, for source code deployed `xiaozhi-server`, the configuration would be:

```
{
    "production": {
        "chat_servers": [
            "ws://127.0.0.1:8000/xiaozhi/v1/?from=mqtt_gateway"
        ]
    },
    "debug": false,
    "max_mqtt_payload_size": 8192,
    "mcp_client": {
        "capabilities": {
        },
        "client_info": {
            "name": "xiaozhi-mqtt-client",
            "version": "1.0.0"
        },
        "max_tools_count": 128
    }
}
```

5. Create a `.env` file in the project root directory and set the following environment variables:
```
PUBLIC_IP=your-ip         # Server public IP
MQTT_PORT=1883            # MQTT server port
UDP_PORT=8884             # UDP server port
API_PORT=8007             # Management API port
MQTT_SIGNATURE_KEY=test   # MQTT signature key
SERVER_SECRET=Te1st12134  # Server secret, please keep consistent with Smart Console (server.secret) or xiaozhi-server (server.auth_key)
```

Please note that the `PUBLIC_IP` configuration must match your actual public IP. If you have a domain name, use that instead.

`MQTT_SIGNATURE_KEY` is the key used for MQTT connection authentication. It is best to set it to something complex, at least 8 characters long and containing both uppercase and lowercase letters. This key will be used again later.

- Do not use simple passwords like `123456`, `test`, etc.
- Do not use simple passwords like `123456`, `test`, etc.
- Do not use simple passwords like `123456`, `test`, etc.

`SERVER_SECRET` is the authentication information used to generate the websocket connection.

1. If you are using full-module deployment and have set `server.auth.enabled` to `true` in Smart Console parameter management, then `SERVER_SECRET` should match the Smart Console's `server.secret`.

2. If you are using single-module deployment and have set `server.auth.enabled` to `true` in your configuration file, then `SERVER_SECRET` should match the `server.auth_key` in your configuration file.

6. Start the MQTT gateway
```
# Start the service
pm2 start ecosystem.config.js

# View logs
pm2 logs xz-mqtt
```

When you see the following log, it indicates the MQTT gateway has started successfully:
```
0|xz-mqtt  | 2025-09-11T12:14:48: MQTT server is listening on port 1883
0|xz-mqtt  | 2025-09-11T12:14:48: UDP server is listening on x.x.x.x:8884
```

To restart the MQTT gateway, execute the following command:
```
pm2 restart xz-mqtt
```

## Part 2: Full-module Operation to Achieve MQTT+UDP Connection with Xiao Zhi Hardware

Check the version number at the bottom of your Smart Console homepage to confirm if your Smart Console version is `0.7.7` or higher. If not, you need to upgrade the Smart Console.

1. In the Smart Console, click `Parameter Management` at the top, search for `server.mqtt_gateway`, click Edit, and fill in the `PUBLIC_IP`+`:`+`MQTT_PORT` you set in the `.env` file. For example:
```
192.168.0.7:1883
```

2. In the Smart Console, click `Parameter Management` at the top, search for `server.mqtt_signature_key`, click Edit, and fill in the `MQTT_SIGNATURE_KEY` you set in the `.env` file.

3. In the Smart Console, click `Parameter Management` at the top, search for `server.udp_gateway`, click Edit, and fill in the `PUBLIC_IP`+`:`+`UDP_PORT` you set in the `.env` file. For example:
```
192.168.0.7:8884
```

4. In the Smart Console, click `Parameter Management` at the top, search for `server.mqtt_manager_api`, click Edit, and fill in the `PUBLIC_IP`+`:`+`API_PORT` you set in the `.env` file. For example:
```
192.168.0.7:8007
```

After completing the above configuration, you can use the curl command to verify if your OTA address will issue MQTT configuration. Replace `http://localhost:8002/xiaozhi/ota/` in the following command with your OTA address:
```
curl 'http://localhost:8002/xiaozhi/ota/' \
  -H 'Content-Type: application/json' \
  -H 'Client-Id: 7b94d69a-9808-4c59-9c9b-704333b38aff' \
  -H 'Device-Id: 11:22:33:44:55:66' \
  --data-raw $'{\n  "application": {\n    "version": "1.0.1",\n    "elf_sha256": "1"\n  },\n  "board": {\n    "mac": "11:22:33:44:55:66"\n  }\n}'
```

If the returned content includes MQTT-related configuration, it indicates successful configuration. Similar to this:
```
{"server_time":{"timestamp":1757567894012,"timeZone":"Asia/Shanghai","timezone_offset":480},"activation":{"code":"460609","message":"http://xiaozhi.server.com\n460609","challenge":"11:22:33:44:55:66"},"firmware":{"version":"1.0.1","url":"http://xiaozhi.server.com:8002/xiaozhi/otaMag/download/NOT_ACTIVATED_FIRMWARE_THIS_IS_A_INVALID_URL"},"websocket":{"url":"ws://192.168.4.23:8000/xiaozhi/v1/"},"mqtt":{"endpoint":"192.168.0.7:1883","client_id":"GID_default@@@11_22_33_44_55_66@@@7b94d69a-9808-4c59-9c9b-704333b38aff","username":"eyJpcCI6IjA6MDowOjA6MDowOjA6MSJ9","password":"Y8XP9xcUhVIN9OmbCHT9ETBiYNE3l3Z07Wk46wV9PE8=","publish_topic":"device-server","subscribe_topic":"devices/p2p/11_22_33_44_55_66"}}
```

Since MQTT information is issued via the OTA address, you only need to ensure your OTA address can connect to the server normally and then restart and wake up the device.

After waking up, monitor the mqtt-gateway logs to confirm if there are successful connection logs.
```
pm2 logs xz-mqtt
```

## Part 3: Single-module Operation of xiaozhi-server to Achieve MQTT+UDP Connection with Xiao Zhi Hardware

Open your `data/.config.yaml` file, find `mqtt_gateway` under `server` and fill in the `PUBLIC_IP`+`:`+`MQTT_PORT` you set in the `.env` file. For example:
```
192.168.0.7:1883
```

Find `mqtt_signature_key` under `server` and fill in the `MQTT_SIGNATURE_KEY` you set in the `.env` file.

Find `udp_gateway` under `server` and fill in the `PUBLIC_IP`+`:`+`UDP_PORT` you set in the `.env` file. For example:
```
192.168.0.7:8884
```

After completing the above configuration, you can use the curl command to verify if your OTA address will issue MQTT configuration. Replace `http://localhost:8002/xiaozhi/ota/` in the following command with your OTA address:
```
curl 'http://localhost:8002/xiaozhi/ota/' \
  -H 'Device-Id: 11:22:33:44:55:66' \
  --data-raw $'{\n  "application": {\n    "version": "1.0.1",\n    "elf_sha256": "1"\n  },\n  "board": {\n    "mac": "11:22:33:44:55:66"\n  }\n}'
```

If the returned content includes MQTT-related configuration, it indicates successful configuration. Similar to this:
```
{"server_time":{"timestamp":1758781561083,"timeZone":"GMT+08:00","timezone_offset":480},"activation":{"code":"527111","message":"http://xiaozhi.server.com\n527111","challenge":"11:22:33:44:55:66"},"firmware":{"version":"1.0.1","url":"http://xiaozhi.server.com:8002/xiaozhi/otaMag/download/NOT_ACTIVATED_FIRMWARE_THIS_IS_A_INVALID_URL"},"websocket":{"url":"ws://192.168.1.15:8000/xiaozhi/v1/"},"mqtt":{"endpoint":"192.168.1.15:1883","client_id":"GID_default@@@11_22_33_44_55_66@@@11_22_33_44_55_66","username":"eyJpcCI6IjE5Mi4xNjguMS4xNSJ9","password":"fjAYs49zTJecWqJ3jBt+kqxVn/x7vkXRAc85ak/va7Y=","publish_topic":"device-server","subscribe_topic":"devices/p2p/11_22_33_44_55_66"}}
```

Since MQTT information is issued via the OTA address, you only need to ensure your OTA address can connect to the server normally and then restart and wake up the device.

After waking up, monitor the mqtt-gateway logs to confirm if there are successful connection logs.
```
pm2 logs xz-mqtt
```

> [!NOTE] 
> This translation requires manual review as it involves technical documentation that may benefit from human verification for accuracy.