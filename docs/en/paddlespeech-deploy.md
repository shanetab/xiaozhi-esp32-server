# PaddleSpeechTTS Integration with Xiaozhi Service

## Key Notes
- Advantages: Local offline deployment, fast speed
- Disadvantages: As of September 25, 2025, the default model is a Chinese model that does not support text-to-speech for English. If English text is included, no sound will be produced. If you need to support both Chinese and English, you'll need to train your own model.

## I. Basic Environment Requirements
Operating System: Windows / Linux / WSL 2

Python Version: 3.9 or higher (please adjust according to Paddle official tutorial)

Paddle Version: Official latest version   ```https://www.paddlepaddle.org.cn/install```

Dependency Management Tool: conda or venv

## II. Start the PaddleSpeech Service
### 1. Clone source code from PaddleSpeech official repository
```bash 
git clone https://github.com/PaddlePaddle/PaddleSpeech.git
```
### 2. Create Virtual Environment
```bash
conda create -n paddle_env python=3.10 -y
conda activate paddle_env
```
### 3. Install Paddle
Due to different CPU architectures and GPU architectures, please establish the environment according to the Python versions supported by Paddle official
```
https://www.paddlepaddle.org.cn/install
```

### 4. Enter PaddleSpeech Directory
```bash
cd PaddleSpeech
```
### 5. Install PaddleSpeech
```bash
pip install pytest-runner -i https://pypi.tuna.tsinghua.edu.cn/simple

# Use any of the following commands
pip install paddlepaddle -i https://mirror.baidu.com/pypi/simple
pip install paddlespeech -i https://pypi.tuna.tsinghua.edu.cn/simple
```
### 6. Use command to automatically download voice model
```bash
paddlespeech tts --input "Hello, this is a test"
```
This step will automatically download the model cache to the local .paddlespeech/models directory

### 7. Modify tts_online_application.yaml Configuration
Refer to directory ```"PaddleSpeech\demos\streaming_tts_server\conf\tts_online_application.yaml"```
Select the ```tts_online_application.yaml``` file with an editor, and set ```protocol``` to ```websocket```

### 8. Start the Service
```yaml
paddlespeech_server start --config_file ./demos/streaming_tts_server/conf/tts_online_application.yaml
# Official default startup command:
paddlespeech_server start --config_file ./conf/tts_online_application.yaml
```
Please adjust the startup command according to the actual directory of your ```tts_online_application.yaml```. When you see the following logs, the startup is successful:
```
Prefix dict has been built successfully.
[2025-08-07 10:03:11,312] [   DEBUG] __init__.py:166 - Prefix dict has been built successfully.
INFO:     Started server process [2298]
INFO:     Waiting for application startup.
INFO:     Application startup complete.
INFO:     Uvicorn running on http://0.0.0.0:8092 (Press CTRL+C to quit)
```

## III. Modify Xiaozhi Configuration File
### 1.```main/xiaozhi-server/core/providers/tts/paddle_speech.py```

### 2.```main/xiaozhi-server/data/.config.yaml```
Using single-module deployment
```yaml
selected_module:
  TTS: PaddleSpeechTTS
TTS:
  PaddleSpeechTTS:
      type: paddle_speech
      protocol: websocket 
      url:  ws://127.0.0.1:8092/paddlespeech/tts/streaming  # TTS service URL address, pointing to local server [websocket default ws://127.0.0.1:8092/paddlespeech/tts/streaming]
      spk_id: 0  # Speaker ID, 0 usually represents the default speaker
      sample_rate: 24000  # Sample rate [websocket default 24000, http default 0 automatic selection]
      speed: 1.0  # Speech speed, 1.0 represents normal speed, >1 means faster, <1 means slower
      volume: 1.0  # Volume, 1.0 represents normal volume, >1 means louder, <1 means softer
      save_path:   # Save path
```
### 3. Start Xiaozhi Service
```py
python app.py
```
After starting `python start.py` under `main/digital-human`, open `http://127.0.0.1:8006/index.html` to test connection and send messages, checking if PaddleSpeech has output logs

Log Output Reference:
```
INFO:     127.0.0.1:44312 - "WebSocket /paddlespeech/tts/streaming" [accepted]
INFO:     connection open
[2025-08-07 11:16:33,355] [    INFO] - sentence: Hey, why are you chatting with me all of a sudden?
[2025-08-07 11:16:33,356] [    INFO] - The durations of audio is: 2.4625 s
[2025-08-07 11:16:33,356] [    INFO] - first response time: 0.1143045425415039 s
[2025-08-07 11:16:33,356] [    INFO] - final response time: 0.4777836799621582 s
[2025-08-07 11:16:33,356] [    INFO] - RTF: 0.19402382942625715
[2025-08-07 11:16:33,356] [    INFO] - Other info: front time: 0.06514096260070801 s, first am infer time: 0.008037090301513672 s, first voc infer time: 0.04112648963928223 s,
[2025-08-07 11:16:33,356] [    INFO] - Complete the synthesis of the audio streams
INFO:     connection closed
```

> [!NOTE] 
> This translation requires manual review as it involves technical documentation that may benefit from human verification for accuracy.