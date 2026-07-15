# Performance Testing Tool Guide for Speech Recognition, Large Language Model, Non-Stream Speech Synthesis, Stream Speech Synthesis, and Vision Models

1. Create a data directory under main/xiaozhi-server directory
2. Create a .config.yaml file under the data directory
3. In the .data/config.yaml file, write the parameters for your speech recognition, large language model, stream speech synthesis, and vision model
4. For example:
```
LLM:
  ChatGLMLLM:
    # Define LLM API type
    type: openai
    # glm-4-flash is free, but you still need to register and fill in the api_key
    # You can find your api key here https://bigmodel.cn/usercenter/proj-mgmt/apikeys
    model_name: glm-4-flash
    url: https://open.bigmodel.cn/api/paas/v4/
    api_key: your chat-glm web key

TTS:

VLLM:

ASR:
```
5. Run performance_tester.py under the main/xiaozhi-server directory:
```
python performance_tester.py
```

> [!NOTE] 
> This translation requires manual review as it involves technical documentation that may benefit from human verification for accuracy.