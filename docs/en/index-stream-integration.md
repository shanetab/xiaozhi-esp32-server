# IndexStreamTTS Integration Guide

## Environment Preparation
### 1. Clone the Project
```bash 
git clone https://github.com/Ksuriuri/index-tts-vllm.git
```
Enter the extracted directory
```bash
cd index-tts-vllm
```
Switch to the specified version (using VLLM-0.10.2 historical version)
```bash
git checkout 224e8d5e5c8f66801845c66b30fa765328fd0be3
```

### 2. Create and Activate Conda Environment
```bash 
conda create -n index-tts-vllm python=3.12
conda activate index-tts-vllm
```

### 3. Install PyTorch (Version 2.8.0 Required)
#### Check the highest version supported by the graphics card and the actual installed version
```bash
nvidia-smi
nvcc --version
``` 
#### Highest CUDA version supported by the driver
```bash
CUDA Version: 12.8
```
#### Actual installed CUDA compiler version
```bash
Cuda compilation tools, release 12.8, V12.8.89
```
#### Then the corresponding installation command (pytorch defaults to the 12.8 driver version)
```bash
pip install torch torchvision
```
PyTorch version 2.8.0 is required (corresponding to vllm 0.10.2). For specific installation instructions, please refer to: [pytorch official website](https://pytorch.org/get-started/locally/)

### 4. Install Dependencies
```bash 
pip install -r requirements.txt
```

### 5. Download Model Weights
### Option 1: Download Official Weight Files and Convert
This is the official weight file, download it to any local path, supporting IndexTTS-1.5 weights  
| HuggingFace                                                   | ModelScope                                                          |
|---------------------------------------------------------------|---------------------------------------------------------------------|
| [IndexTTS](https://huggingface.co/IndexTeam/Index-TTS)        | [IndexTTS](https://modelscope.cn/models/IndexTeam/Index-TTS)        |
| [IndexTTS-1.5](https://huggingface.co/IndexTeam/IndexTTS-1.5) | [IndexTTS-1.5](https://modelscope.cn/models/IndexTeam/IndexTTS-1.5) |

Taking ModelScope installation method as an example  
#### Please Note: Git needs to be installed and initialized with lfs enabled (if already installed, you can skip)
```bash
sudo apt-get install git-lfs
git lfs install
```
Create model directory and pull the model
```bash 
mkdir model_dir
cd model_dir
git clone https://www.modelscope.cn/IndexTeam/IndexTTS-1.5.git
```

#### Model Weight Conversion
```bash 
bash convert_hf_format.sh /path/to/your/model_dir
```
For example, if you downloaded the IndexTTS-1.5 model to the model_dir directory, execute the following command
```bash
bash convert_hf_format.sh model_dir/IndexTTS-1.5
```
This operation will convert the official model weights to transformers library compatible versions, saved in the vllm folder under the model weights path, making it convenient for subsequent vllm library to load model weights.

### 6. Adjust API Interface to Match This Project
The interface return data is not compatible with the project and needs to be adjusted to directly return audio data
```bash
vi api_server.py
```
```bash 
@app.post("/tts", responses={
    200: {"content": {"application/octet-stream": {}}},
    500: {"content": {"application/json": {}}}
})
async def tts_api(request: Request):
    try:
        data = await request.json()
        text = data["text"]
        character = data["character"]

        global tts
        sr, wav = await tts.infer_with_ref_audio_embed(character, text)

        return Response(content=wav.tobytes(), media_type="application/octet-stream")
        
    except Exception as ex:
        tb_str = ''.join(traceback.format_exception(type(ex), ex, ex.__traceback__))
        print(tb_str)
        return JSONResponse(
            status_code=500,
            content={
                "status": "error",
                "error": str(tb_str)
            }
        )
```

### 7. Write a Shell Startup Script (Please Note This Should Run in the Corresponding Conda Environment)
```bash 
vi start_api.sh
```
### Paste the following content and press :wq to save  
#### The /home/system/index-tts-vllm/model_dir/IndexTTS-1.5 in the script should be modified to the actual path
```bash
# Activate conda environment
conda activate index-tts-vllm 
echo "激活项目conda环境"
sleep 2
# Find the process ID occupying port 11996
PID_VLLM=$(sudo netstat -tulnp | grep 11996 | awk '{print $7}' | cut -d'/' -f1)

# Check if the process ID is found
if [ -z "$PID_VLLM" ]; then
  echo "没有找到占用11996端口的进程"
else
  echo "找到占用11996端口的进程，进程号为: $PID_VLLM"
  # First try normal kill, wait 2 seconds
  kill $PID_VLLM
  sleep 2
  # Check if the process is still running
  if ps -p $PID_VLLM > /dev/null; then
    echo "进程仍在运行，强制终止..."
    kill -9 $PID_VLLM
  fi
  echo "已终止进程 $PID_VLLM"
fi

# Find processes related to VLLM::EngineCore
GPU_PIDS=$(ps aux | grep -E "VLLM|EngineCore" | grep -v grep | awk '{print $2}')

# Check if the process ID is found
if [ -z "$GPU_PIDS" ]; then
  echo "没有找到VLLM相关进程"
else
  echo "找到VLLM相关进程，进程号为: $GPU_PIDS"
  # First try normal kill, wait 2 seconds
  kill $GPU_PIDS
  sleep 2
  # Check if the process is still running
  if ps -p $GPU_PIDS > /dev/null; then
    echo "进程仍在运行，强制终止..."
    kill -9 $GPU_PIDS
  fi
  echo "已终止进程 $GPU_PIDS"
fi

# Create tmp directory (if not exists)
mkdir -p tmp

# Run api_server.py in the background, redirect logs to tmp/server.log
nohup python api_server.py --model_dir /home/system/index-tts-vllm/model_dir/IndexTTS-1.5 --port 11996 > tmp/server.log 2>&1 &
echo "api_server.py 已在后台运行，日志请查看 tmp/server.log"
```
Give the script execute permissions and run the script
```bash 
chmod +x start_api.sh
./start_api.sh
```
Logs will be output in tmp/server.log, you can view the log status with the following command
```bash
tail -f tmp/server.log
```
If the graphics card memory is sufficient, you can add the startup parameter --gpu_memory_utilization in the script to adjust the GPU memory usage ratio, default value is 0.25

## Voice Configuration
index-tts-vllm supports registering custom voices through configuration files, supporting single voice and mixed voice configuration.  
Configure custom voices in the assets/speaker.json file in the project root directory
### Configuration Format Description
```bash
{
    "Speaker Name 1": [
        "Audio File Path 1.wav",
        "Audio File Path 2.wav"
    ],
    "Speaker Name 2": [
        "Audio File Path 3.wav"
    ]
}
```
### Note (After Configuring Roles, Restart the Service for Voice Registration)
After adding, you need to add the corresponding speaker in the Smart Console (single module, then replace the corresponding voice)

> [!NOTE] 
> This translation requires manual review as it involves technical documentation that may benefit from human verification for accuracy.