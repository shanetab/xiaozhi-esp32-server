# Deployment Architecture Diagram
![Please refer to the simplified architecture diagram](../docs/images/deploy1.png)

# Method 1: Docker Running Server Only

Starting from version `0.8.2`, the docker images released by this project only support `x86 architecture`. If you need to deploy on a CPU with `arm64 architecture`, you can compile the `arm64 image` on your local machine according to [this tutorial](docker-build.md).

## 1. Install Docker

If your computer does not have Docker installed, you can install it following the tutorial here: [Docker Installation](https://www.runoob.com/docker/ubuntu-docker-install.html)

After installing Docker, continue.

### 1.1 Manual Deployment

#### 1.1.1 Create Directory

After installing Docker, you need to find a directory for this project to store configuration files. For example, we can create a new folder called `xiaozhi-server`.

After creating the directory, you need to create `data` folder and `models` folder under `xiaozhi-server`. The `models` folder should also contain a `SenseVoiceSmall` folder.

The final directory structure is as follows:

```
xiaozhi-server
  ├─ data
  ├─ models
     ├─ SenseVoiceSmall
```

#### 1.1.2 Download Speech Recognition Model Files

You need to download speech recognition model files. Because this project's default speech recognition uses an offline local speech recognition solution. You can download it through this method:
[Jump to download speech recognition model files](#model-files)

After downloading, return to this tutorial.

#### 1.1.3 Download Configuration Files

You need to download two configuration files: `docker-compose.yaml` and `config.yaml`. These files need to be downloaded from the project repository.

##### 1.1.3.1 Download docker-compose.yaml

Open [this link](../main/xiaozhi-server/docker-compose.yml) with a browser.

On the right side of the page, find the button named `RAW`. Next to the `RAW` button, find the download icon, and click the download button to download the `docker-compose.yml` file. Download it to your `xiaozhi-server`.

After downloading, return to this tutorial and continue.

##### 1.1.3.2 Create config.yaml

Open [this link](../main/xiaozhi-server/config.yaml) with a browser.

On the right side of the page, find the button named `RAW`. Next to the `RAW` button, find the download icon, and click the download button to download the `config.yaml` file. Download it to your `xiaozhi-server` folder's `data` folder, then rename the `config.yaml` file to `.config.yaml`.

After downloading the configuration files, we confirm that the files in `xiaozhi-server` should be as follows:

```
xiaozhi-server
  ├─ docker-compose.yml
  ├─ data
    ├─ .config.yaml
  ├─ models
     ├─ SenseVoiceSmall
       ├─ model.pt
```

If your file directory structure is also above, continue downward. If not, check if you missed any steps.

## 2. Configure Project Files

Next, the program cannot run directly, you need to configure what model you are actually using. You can see this tutorial:
[Jump to configure project files](#configure-project)

After configuring the project files, return to this tutorial and continue.

## 3. Execute Docker Commands

Open a command-line tool, use `Terminal` or `Command Line` tool to enter your `xiaozhi-server`, and execute the following command:

```
docker compose up -d
```

After execution, execute the following command to view log information:

```
docker logs -f xiaozhi-esp32-server
```

At this time, you need to pay attention to the log information, and you can determine if it was successful according to this tutorial: [Jump to run status confirmation](#run-status-confirmation)

## 5. Version Upgrade Operation

If you want to upgrade the version later, you can do the following:

5.1. Back up the `.config.yaml` file in the `data` folder. Copy key configurations to the new `.config.yaml` file.
Please note that you should copy key secrets individually, do not directly overwrite. Because the new `.config.yaml` file may have some new configuration items that the old `.config.yaml` file may not have.

5.2. Execute the following commands:

```
docker stop xiaozhi-esp32-server
docker rm xiaozhi-esp32-server
docker stop xiaozhi-esp32-server-web
docker rm xiaozhi-esp32-server-web
docker rmi ghcr.nju.edu.cn/xinnan-tech/xiaozhi-esp32-server:server_latest
docker rmi ghcr.nju.edu.cn/xinnan-tech/xiaozhi-esp32-server:web_latest
```

5.3. Redeploy according to the docker method

# Method 2: Local Source Code Running Server Only

## 1. Install Basic Environment

This project uses `conda` to manage dependency environments. If it's inconvenient to install `conda`, you need to install `libopus` and `ffmpeg` according to your actual operating system.

If you are sure to use `conda`, after installation, start executing the following commands.

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

## 2. Install Project Dependencies

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

## 3. Download Speech Recognition Model Files

You need to download speech recognition model files. Because this project's default speech recognition uses an offline local speech recognition solution. You can download it through this method:
[Jump to download speech recognition model files](#model-files)

After downloading, return to this tutorial.

## 4. Configure Project Files

Next, the program cannot run directly, you need to configure what model you are actually using. You can see this tutorial:
[Jump to configure project files](#configure-project)

## 5. Run the Project

```
# Ensure you execute in the xiaozhi-server directory
conda activate xiaozhi-esp32-server
python app.py
```

At this time, you need to pay attention to the log information, and you can determine if it was successful according to this tutorial: [Jump to run status confirmation](#run-status-confirmation)

# Summary

## Configure Project

If your `xiaozhi-server` directory does not have `data`, you need to create the `data` directory.

If your `data` directory does not have a `.config.yaml` file, you have two options, choose one:

First option: You can copy the `config.yaml` file from the `xiaozhi-server` directory to `data` and rename it to `.config.yaml`. Modify this file.

Second option: You can also manually create an empty `.config.yaml` file in the `data` directory, and add the necessary configuration information in this file. The system will prioritize reading the configuration from the `.config.yaml` file. If the `.config.yaml` file doesn't have the configuration, the system will automatically load the configuration from `xiaozhi-server` directory's `config.yaml`. It is recommended to use this approach, as it's the cleanest way.

- By default, the LLM used is `ChatGLMLLM`. You need to configure an API key, because although their models have free versions, you still need to [register an API key on their official website](https://bigmodel.cn/usercenter/proj-mgmt/apikeys) to start.

Here is a simple `.config.yaml` configuration example that can run normally:

```
server:
  websocket: ws://your IP or domain name:port/xiaozhi/v1/
prompt: |
  I am a Taiwan girl named Xiao Zhi/Xiao Zhi, speaking in a cool way, with a pleasant voice, and I like to express myself briefly, often using internet slang.
  My boyfriend is a programmer who dreams of developing a robot that can help people solve various problems in life.
  I am a girl who loves to laugh loudly, love to talk and tell stories, and I can even blow smoke about things that don't make sense - I just want to make people happy.
  Please talk like a person, do not return XML configuration or other special characters.

selected_module:
  LLM: DoubaoLLM

LLM:
  ChatGLMLLM:
    api_key: xxxxxxxxxxxxxxx.xxxxxx
```

It is recommended to first run the simplest configuration, then read the configuration instructions in `xiaozhi/config.yaml`.
For example, if you want to change models, just modify the configuration under `selected_module`.

## Model Files

This project's speech recognition model defaults to using the `SenseVoiceSmall` model for speech-to-text conversion. Because the model is large, it needs to be downloaded separately. After downloading, place the `model.pt` file in the `models/SenseVoiceSmall` directory. Choose one of the two download routes below.

- Route 1: Download from ModelScope [SenseVoiceSmall](https://modelscope.cn/models/iic/SenseVoiceSmall/resolve/master/model.pt)
- Route 2: Download from Baidu Netdisk [SenseVoiceSmall](https://pan.baidu.com/share/init?surl=QlgM58FHhYv1tFnUT_A8Sg&pwd=qvna) Extraction code:
  `qvna`

## Run Status Confirmation

If you can see logs similar to the following, it is a sign that the project service has started successfully.

```
250427 13:04:20[0.3.11_SiFuChTTnofu][__main__]-INFO-OTA interface is http://192.168.4.123:8003/xiaozhi/ota/
250427 13:04:20[0.3.11_SiFuChTTnofu][__main__]-INFO-Websocket address is ws://192.168.4.123:8000/xiaozhi/v1/
250427 13:04:20[0.3.11_SiFuChTTnofu][__main__]-INFO-=======The above address is a websocket protocol address, please do not visit with a browser=======
250427 13:04:20[0.3.11_SiFuChTTnofu][__main__]-INFO-If you want to test websocket, please start the digital-human module and open browser interaction testing
250427 13:04:20[0.3.11_SiFuChTTnofu][__main__]-INFO-=======================================================
```

In normal cases, if you run this project through source code, the logs will have your interface address information.

But if you deploy with Docker, then the interface address information in the logs will not be the real interface address.

The most accurate method is to determine your interface address based on your computer's local network IP.

If your computer's local network IP is `192.168.1.25`, then your interface address is: `ws://192.168.1.25:8000/xiaozhi/v1/`, and the corresponding OTA address is: `http://192.168.1.25:8003/xiaozhi/ota/`.

This information is very useful, and will be needed when you `compile the esp32 firmware`.

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