# Deployment Architecture Diagram
![Please refer to the full module installation architecture diagram](../docs/images/deploy2.png)

# Method 1: Running Full Modules with Docker
Starting from version 0.8.2, the docker images released by this project only support `x86 architecture`. If you need to deploy on a CPU with `arm64 architecture`, you can compile the `arm64 image` on your local machine according to [this tutorial](docker-build.md).

## 1. Install Docker

If your computer does not have Docker installed, you can install it following the tutorial here: [Docker Installation](https://www.runoob.com/docker/ubuntu-docker-install.html)

There are two ways to install Docker with full modules. You can [use the lazy script](./Deployment_all.md#11-lazy-script) (author [@VanillaNahida](https://github.com/VanillaNahida)) 
The script will automatically download the required files and configuration files for you. You can also use [manual deployment](./Deployment_all.md#12-manual-deployment) to build from scratch.

### 1.1 Lazy Script
Deployment is simple. You can refer to the [video tutorial](https://www.bilibili.com/video/BV17bbvzHExd/). The text tutorial is as follows:
> [!NOTE]  
> Currently only supports one-click deployment on Ubuntu server. Other systems have not been tried and may have some strange bugs.

Connect to the server using an SSH tool and execute the following script with root permissions:
```bash
sudo bash -c "$(wget -qO- https://ghfast.top/https://raw.githubusercontent.com/xinnan-tech/xiaozhi-esp32-server/main/docker-setup.sh)"
```

The script will automatically complete the following operations:
> 1. Install Docker
> 2. Configure image source
> 3. Download/pull images
> 4. Download speech recognition model files
> 5. Guide configuration of the server side

After execution, configure briefly, then refer to the most important 3 things mentioned in [4. Run the Program](#4.-run-the-program) and [5. Restart xiaozhi-esp32-server](#5.-restart-xiaozhi-esp32-server). After completing these three configurations, you can use the system.

### 1.2 Manual Deployment

#### 1.2.1 Create Directory

After installation, you need to find a directory for this project to store configuration files. For example, we can create a new folder called `xiaozhi-server`.

After creating the directory, you need to create `data` folder and `models` folder under `xiaozhi-server`. The `models` folder should also contain a `SenseVoiceSmall` folder.

The final directory structure is as follows:

```
xiaozhi-server
  ├─ data
  ├─ models
     ├─ SenseVoiceSmall
```

#### 1.2.2 Download Speech Recognition Model Files

This project's speech recognition model defaults to using the `SenseVoiceSmall` model for speech-to-text conversion. Because the model is large, it needs to be downloaded separately. After downloading, place the `model.pt` file in the `models/SenseVoiceSmall` directory. Choose one of the two download routes below.

- Route 1: Download from ModelScope [SenseVoiceSmall](https://modelscope.cn/models/iic/SenseVoiceSmall/resolve/master/model.pt)
- Route 2: Download from Baidu Netdisk [SenseVoiceSmall](https://pan.baidu.com/share/init?surl=QlgM58FHhYv1tFnUT_A8Sg&pwd=qvna) Extraction code: 
  `qvna`

#### 1.2.3 Download Configuration Files

You need to download two configuration files: `docker-compose_all.yaml` and `config_from_api.yaml`. These files need to be downloaded from the project repository.

##### 1.2.3.1 Download docker-compose_all.yaml

Open [this link](../main/xiaozhi-server/docker-compose_all.yml) with a browser.

On the right side of the page, find the button named `RAW`. Next to the `RAW` button, find the download icon, and click the download button to download the `docker-compose_all.yml` file. Download it to your `xiaozhi-server`.

Or simply execute `wget https://raw.githubusercontent.com/xinnan-tech/xiaozhi-esp32-server/refs/heads/main/main/xiaozhi-server/docker-compose_all.yml` to download.

After downloading, return to this tutorial and continue.

##### 1.2.3.2 Download config_from_api.yaml

Open [this link](../main/xiaozhi-server/config_from_api.yaml) with a browser.

On the right side of the page, find the button named `RAW`. Next to the `RAW` button, find the download icon, and click the download button to download the `config_from_api.yaml` file. Download it to your `xiaozhi-server` folder's `data` folder, then rename the `config_from_api.yaml` file to `.config.yaml`.

Or simply execute `wget https://raw.githubusercontent.com/xinnan-tech/xiaozhi-esp32-server/refs/heads/main/main/xiaozhi-server/config_from_api.yaml` to download and save.

After downloading the configuration files, we confirm that the files in `xiaozhi-server` should be as follows:

```
xiaozhi-server
  ├─ docker-compose_all.yml
  ├─ data
    ├─ .config.yaml
  ├─ models
     ├─ SenseVoiceSmall
       ├─ model.pt
```

If your file directory structure is also above, continue downward. If not, check if you missed any steps.

## 2. Backup Data

If you have previously successfully run the Smart Console and saved your key information, please copy important data from the Smart Console first. Because during upgrade, some data might be overwritten.

## 3. Clear Previous Version Images and Containers

Next, open a command-line tool, use `Terminal` or `Command Line` tool to enter your `xiaozhi-server` and execute the following commands:

```
docker compose -f docker-compose_all.yml down

docker stop xiaozhi-esp32-server
docker rm xiaozhi-esp32-server

docker stop xiaozhi-esp32-server-web
docker rm xiaozhi-esp32-server-web

docker stop xiaozhi-esp32-server-db
docker rm xiaozhi-esp32-server-db

docker stop xiaozhi-esp32-server-redis
docker rm xiaozhi-esp32-server-redis

docker rmi ghcr.nju.edu.cn/xinnan-tech/xiaozhi-esp32-server:server_latest
docker rmi ghcr.nju.edu.cn/xinnan-tech/xiaozhi-esp32-server:web_latest
```

## 4. Run the Program

Execute the following command to start the new version containers:

```
docker compose -f docker-compose_all.yml up -d
```

After execution, execute the following command to view log information:

```
docker logs -f xiaozhi-esp32-server-web
```

When you see log output, it means your `Smart Console` has started successfully.

```
2025-xx-xx 22:11:12.445 [main] INFO  c.a.d.s.b.a.DruidDataSourceAutoConfigure - Init DruidDataSource
2025-xx-xx 21:28:53.873 [main] INFO  xiaozhi.AdminApplication - Started AdminApplication in 16.057 seconds (process running for 17.941)
http://localhost:8002/xiaozhi/doc.html
```

Please note that at this moment only the `Smart Console` is running. If the `xiaozhi-esp32-server` port 8000 reports an error, don't worry about it for now.

At this time, you need to use a browser to open the `Smart Console`, link: http://127.0.0.1:8002, register the first user. The first user is the super administrator, and subsequent users are regular users. Regular users can only bind devices and configure agents; super administrators can manage models, users, and parameters.

Next, you need to do three important things:

### The First Important Thing

Use the super administrator account to log into the Smart Console. In the top menu, find `Parameter Management`, find the first data in the list, parameter code is `server.secret`, copy it to `Parameter Value`.

`server.secret` needs to be explained. This `parameter value` is very important; its function is to let our `Server` side connect to `manager-api`. `server.secret` is a randomly generated key that is automatically generated each time the manager module is deployed from scratch.

After copying the `parameter value`, open the `.config.yaml` file in the `data` directory under `xiaozhi-server`. At this time, your configuration file content should look like this:

```
manager-api:
  url:  http://127.0.0.1:8002/xiaozhi
  secret: your server.secret value
```

1. Copy the `parameter value` of `server.secret` from the `Smart Console` to the `secret` in the `.config.yaml` file.

2. Since you are deploying with Docker, change the `url` to the following `http://xiaozhi-esp32-server-web:8002/xiaozhi`

3. Since you are deploying with Docker, change the `url` to the following `http://xiaozhi-esp32-server-web:8002/xiaozhi`

4. Since you are deploying with Docker, change the `url` to the following `http://xiaozhi-esp32-server-web:8002/xiaozhi`

Similar to this effect:
```
manager-api:
  url: http://xiaozhi-esp32-server-web:8002/xiaozhi
  secret: 12345678-xxxx-xxxx-xxxx-123456789000
```

After saving, continue with the second important thing.

### The Second Important Thing

Use the super administrator account to log into the Smart Console. In the top menu, find `Model Configuration`, then in the left sidebar click `Large Language Model`, find the first data `Zhipu AI`, click the `Modify` button.

After the modification box pops up, fill in the `API Key` you registered with `Zhipu AI` into the `API Key`. Then click Save.

## 5. Restart xiaozhi-esp32-server

Next, open a command-line tool, use `Terminal` or `Command Line` tool to input:
```
docker restart xiaozhi-esp32-server
docker logs -f xiaozhi-esp32-server
```

If you can see logs similar to the following, it's a sign that the Server has started successfully.

```
25-02-23 12:01:09[core.websocket_server] - INFO - Websocket address is      ws://xxx.xx.xx.xx:8000/xiaozhi/v1/
25-02-23 12:01:09[core.websocket_server] - INFO - =======The above address is a websocket protocol address, please do not visit with a browser=======
25-02-23 12:01:09[core.websocket_server] - INFO - If you want to test websocket, please start the digital-human module and open browser interaction testing
25-02-23 12:01:09[core.websocket_server] - INFO - =======================================================
```

Since you are deploying the full modules, you have two important interfaces to write into the esp32.

OTA Interface:
```
http://your local network IP of the host:8002/xiaozhi/ota/
```

Websocket Interface:
```
ws://your host's IP:8000/xiaozhi/v1/
```

### The Third Important Thing

Use the super administrator account to log into the Smart Console. In the top menu, find `Parameter Management`, find the parameter code `server.websocket`, enter your `Websocket Interface`.

Use the super administrator account to log into the Smart Console. In the top menu, find `Parameter Management`, find the parameter code `server.ota`, enter your `OTA Interface`.

Next, you can start operating your esp32 device. You can `compile your own esp32 firmware` or configure using `Xiege's compiled firmware version 1.6.1 or above`. Either choice is fine.

1. [Compile your own esp32 firmware](firmware-build.md).

2. [Configure custom server based on Xiege's compiled firmware](firmware-setting.md).

# Method 2: Running Full Modules Locally from Source Code

## 1. Install MySQL Database

If MySQL is already installed on this machine, you can directly create a database named `xiaozhi_esp32_server` in the database.

```sql
CREATE DATABASE xiaozhi_esp32_server CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
```

If MySQL is not installed, you can install MySQL via Docker:

```
docker run --name xiaozhi-esp32-server-db -e MYSQL_ROOT_PASSWORD=123456 -p 3306:3306 -e MYSQL_DATABASE=xiaozhi_esp32_server -e MYSQL_INITDB_ARGS="--character-set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci" -e TZ=Asia/Shanghai -d mysql:latest
```

## 2. Install Redis

If Redis is not installed, you can install Redis via Docker:

```
docker run --name xiaozhi-esp32-server-redis -d -p 6379:6379 redis
```

## 3. Run manager-api Program

3.1 Install JDK21, set JDK environment variables

3.2 Install Maven, set Maven environment variables

3.3 Use VSCode programming tool, install Java environment related plugins

3.4 Use VSCode programming tool to load the manager-api module

In `src/main/resources/application-dev.yml`, configure database connection information:

```
spring:
  datasource:
    username: root
    password: 123456
```

In `src/main/resources/application-dev.yml`, configure Redis connection information:

```
spring:
    data:
      redis:
        host: localhost
        port: 6379
        password:
        database: 0
```

3.5 Run the main program

This project is a SpringBoot project, and the startup method is:
Open `Application.java` and run the `Main` method to start

```
Path address:
src/main/java/xiaozhi/AdminApplication.java
```

When you see log output, it means your `manager-api` has started successfully.

```
2025-xx-xx 22:11:12.445 [main] INFO  c.a.d.s.b.a.DruidDataSourceAutoConfigure - Init DruidDataSource
2025-xx-xx 21:28:53.873 [main] INFO  xiaozhi.AdminApplication - Started AdminApplication in 16.057 seconds (process running for 17.941)
http://localhost:8002/xiaozhi/doc.html
```

## 4. Run manager-web Program

4.1 Install nodejs

4.2 Use VSCode programming tool to load the manager-web module

Enter the manager-web directory in the terminal:

```
npm install
```

Then start:

```
npm run serve
```

Please note, if your manager-api interface is not at `http://localhost:8002`, please modify the path in `main/manager-web/.env.development` during development.

After successful operation, you need to use a browser to open the `Smart Console`, link: http://127.0.0.1:8001, register the first user. The first user is the super administrator, and subsequent users are regular users. Regular users can only bind devices and configure agents; super administrators can manage models, users, and parameters.

Important: After registration is successful, use the super administrator account to log into the Smart Console. In the top menu, find `Model Configuration`, then in the left sidebar click `Large Language Model`, find the first data `Zhipu AI`, click the `Modify` button.

After the modification box pops up, fill in the `API Key` you registered with `Zhipu AI` into the `API Key`. Then click Save.

Important: After registration is successful, use the super administrator account to log into the Smart Console. In the top menu, find `Model Configuration`, then in the left sidebar click `Large Language Model`, find the first data `Zhipu AI`, click the `Modify` button.

After the modification box pops up, fill in the `API Key` you registered with `Zhipu AI` into the `API Key`. Then click Save.

Important: After registration is successful, use the super administrator account to log into the Smart Console. In the top menu, find `Model Configuration`, then in the left sidebar click `Large Language Model`, find the first data `Zhipu AI`, click the `Modify` button.

After the modification box pops up, fill in the `API Key` you registered with `Zhipu AI` into the `API Key`. Then click Save.

## 5. Install Python Environment

This project uses `conda` to manage dependency environments. If it's inconvenient to install `conda`, you need to install `libopus` and `ffmpeg` according to your actual operating system.

If you are sure to use `conda`, then after installing, start executing the following commands.

Important Tip! Windows users can install `Anaconda` to manage environments. After installing `Anaconda`, search for keywords related to `anaconda` in the `Start` menu, find `Anaconda Prompt`, and run it with administrator privileges. As shown in the figure below.

![conda_prompt](./images/conda_env_1.png)

After running, if you can see the word `(base)` in front of the command line window, it means you have successfully entered the `conda` environment. Then you can execute the following commands.

![conda_env](./images/conda_env_2.png)

```
conda remove -n xiaozhi-esp32-server --all -y
conda create -n xiaozhi-esp32-server python=3.10 -y
conda activate xiaozhi-esp32-server

# Add Tsinghua source channel
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/conda-forge

conda install libopus -y
conda install ffmpeg -y

# When deploying in Linux environment, if you encounter missing libiconv.so.2 dynamic library errors, please install with the following command
conda install libiconv -y
```

Please note, the above commands are not successful with a single execution. You need to execute them step by step, and after each step is executed, check the output logs to see if it was successful.

## 6. Install Project Dependencies

You first need to download the source code of this project. The source code can be downloaded with the `git clone` command. If you are not familiar with the `git clone` command.

You can open this address with a browser `https://github.com/xinnan-tech/xiaozhi-esp32-server.git`

After opening, find a green button on the page that says `Code`, click it, and you will see the `Download ZIP` button.

Click it to download the project source code compressed package. After downloading to your computer, unzip it. At this point, its name might be `xiaozhi-esp32-server-main`. You need to rename it to `xiaozhi-esp32-server`. In this file, go into the `main` folder, then into the `xiaozhi-server` folder. Remember this directory `xiaozhi-server`.

```
# Continue using conda environment
conda activate xiaozhi-esp32-server
# Enter your project root directory, then into main/xiaozhi-server
cd main/xiaozhi-server
pip config set global.index-url https://mirrors.aliyun.com/pypi/simple/
pip install -r requirements.txt
```

### 7. Download Speech Recognition Model Files

This project's speech recognition model defaults to using the `SenseVoiceSmall` model for speech-to-text conversion. Because the model is large, it needs to be downloaded separately. After downloading, place the `model.pt` file in the `models/SenseVoiceSmall` directory. Choose one of the two download routes below.

- Route 1: Download from ModelScope [SenseVoiceSmall](https://modelscope.cn/models/iic/SenseVoiceSmall/resolve/master/model.pt)
- Route 2: Download from Baidu Netdisk [SenseVoiceSmall](https://pan.baidu.com/share/init?surl=QlgM58FHhYv1tFnUT_A8Sg&pwd=qvna) Extraction code:
  `qvna`

## 8. Configure Project Files

Use the super administrator account to log into the Smart Console, and in the top menu find `Parameter Management`. Find the first data in the list, parameter code is `server.secret`, copy it to `Parameter Value`.

`server.secret` needs to be explained. This `parameter value` is very important; its function is to let our `Server` side connect to `manager-api`. `server.secret` is a randomly generated key that is automatically generated each time the manager module is deployed from scratch.

If your `xiaozhi-server` directory does not have `data`, you need to create the `data` directory. If your `data` directory does not have a `.config.yaml` file, you can copy the `config_from_api.yaml` file from the `xiaozhi-server` directory to `data` and rename it to `.config.yaml`.

After copying the `parameter value`, open the `.config.yaml` file in the `data` directory under `xiaozhi-server`. At this time, your configuration file content should look like this:

```
manager-api:
  url: http://127.0.0.1:8002/xiaozhi
  secret: your server.secret value
```

Copy the `parameter value` of `server.secret` from the `Smart Console` to the `secret` in the `.config.yaml` file.

Similar to this effect:
```
manager-api:
  url: http://127.0.0.1:8002/xiaozhi
  secret: 12345678-xxxx-xxxx-xxxx-123456789000
```

## 9. Run the Project

```
# Ensure you execute in the xiaozhi-server directory
conda activate xiaozhi-esp32-server
python app.py
```

If you can see logs similar to the following, it's a sign that the project service has started successfully.

```
25-02-23 12:01:09[core.websocket_server] - INFO - Server is running at ws://xxx.xx.xx.xx:8000/xiaozhi/v1/
25-02-23 12:01:09[core.websocket_server] - INFO - =======The above address is a websocket protocol address, please do not visit with a browser=======
25-02-23 12:01:09[core.websocket_server] - INFO - If you want to test websocket, please start the digital-human module and open browser interaction testing
25-02-23 12:01:09[core.websocket_server] - INFO - =======================================================
```

Since you are deploying the full modules, you have two important interfaces.

OTA Interface:
```
http://your computer's local network IP:8002/xiaozhi/ota/
```

Websocket Interface:
```
ws://your computer's local network IP:8000/xiaozhi/v1/
```

Please be sure to write these two interface addresses into the Smart Console. They will affect websocket address distribution and automatic upgrade functionality.

1. Use the super administrator account to log into the Smart Console. In the top menu, find `Parameter Management`, find the parameter code `server.websocket`, enter your `Websocket Interface`.

2. Use the super administrator account to log into the Smart Console. In the top menu, find `Parameter Management`, find the parameter code `server.ota`, enter your `OTA Interface`.

Next, you can start operating your esp32 device. You can `compile your own esp32 firmware` or configure using `Xiege's compiled firmware version 1.6.1 or above`. Either choice is fine.

1. [Compile your own esp32 firmware](firmware-build.md).

2. [Configure custom server based on Xiege's compiled firmware](firmware-setting.md).

# Common Issues

The following are some common issues for reference:

1. [Why does Xiaozhi recognize many Korean, Japanese, and English words when I speak?](./FAQ.md)<br/>
2. [Why does "TTS task error, file not found" appear?](./FAQ.md)<br/>
3. [TTS often fails and times out](./FAQ.md)<br/>
4. [WiFi can connect to self-built server, but 4G mode cannot connect](./FAQ.md)<br/>
5. [How to improve Xiaozhi's response speed in conversation?](./FAQ.md)<br/>
6. [I speak slowly, and Xiaozhi often interrupts me](./FAQ.md)<br/>

## Deployment-related Tutorials

1. [How to automatically pull the latest code of this project, compile, and start](./dev-ops-integration.md)<br/>
2. [How to deploy MQTT gateway to enable MQTT+UDP protocol](./mqtt-gateway-integration.md)<br/>
3. [How to integrate with Nginx](https://github.com/xinnan-tech/xiaozhi-esp32-server/issues/791)<br/>

## Expansion-related Tutorials

1. [How to enable mobile phone number registration for Smart Console](./ali-sms-integration.md)<br/>
2. [How to integrate HomeAssistant for smart home control](./homeassistant-integration.md)<br/>
3. [How to enable vision model for photo recognition](./mcp-vision-integration.md)<br/>
4. [How to deploy MCP endpoint](./mcp-endpoint-enable.md)<br/>
5. [How to connect to MCP endpoint](./mcp-endpoint-integration.md)<br/>
6. [How to enable voiceprint recognition](./voiceprint-integration.md)<br/>
7. [News plugin source configuration guide](./newsnow_plugin_config.md)<br/>
8. [Weather plugin usage guide](./weather-integration.md)<br/>

## Voice Cloning, Local Voice Deployment-related Tutorials

1. [How to clone voice in Smart Console](./huoshan-streamTTS-voice-cloning.md)<br/>
2. [How to deploy integrated index-tts local voice](./index-stream-integration.md)<br/>
3. [How to deploy integrated fish-speech local voice](./fish-speech-integration.md)<br/>
4. [How to deploy integrated PaddleSpeech local voice](./paddlespeech-deploy.md)<br/>

## Performance Testing Tutorials

1. [Speed testing guide for each component](./performance_tester.md)<br/>
2. [Regular public test results](https://github.com/xinnan-tech/xiaozhi-performance-research)<br/>

> [!NOTE] 
> This translation requires manual review as it involves technical documentation that may benefit from human verification for accuracy.