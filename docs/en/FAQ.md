# Frequently Asked Questions ❓

### 1. Why does Xiao Zhi recognize many Korean, Japanese, and English words when I speak? 🇰🇷

Recommendation: Check if `models/SenseVoiceSmall` already has the `model.pt` file. 
If not, you need to download it, see here [Download speech recognition model files](Deployment.md#model-files)

### 2. Why does "TTS task error, file not found" appear? 📁

Recommendation: Check if you have correctly installed the `libopus` and `ffmpeg` libraries using `conda`.

If not installed, install them:
```
conda install conda-forge::libopus
conda install conda-forge::ffmpeg
```

### 3. TTS often fails and times out ⏰

Recommendation: If `EdgeTTS` often fails, first check if you're using a proxy (VPN). If you are, try turning off the proxy and test again;  
If using Volcano Engine's Doubao TTS, when it often fails, it's recommended to use the paid version, as the test version only supports 2 concurrent connections.

### 4. WiFi can connect to self-built server, but 4G mode cannot connect 🔐

Reason: Xiege's firmware requires secure connections for 4G mode.

Solution: There are currently two methods to solve this. Choose one:
1. Modify the code. Refer to this video for resolution https://www.bilibili.com/video/BV18MfTYoE85
2. Use nginx to configure SSL certificate. Refer to tutorial https://icnt94i5ctj4.feishu.cn/docx/GnYOdMNJOoRCljx1ctecsj9cnRe

### 5. How to improve Xiao Zhi's response speed? ⚡

This project's default configuration is a low-cost solution. It's recommended that beginners first use the default free models to solve the "runable" issue, then optimize for "fast running".  
To improve response speed, you can try replacing components. Starting from version `0.5.2`, the project supports streaming configuration. Compared to earlier versions, response speed improves by approximately `2.5 seconds`, significantly improving user experience.

| Module Name | Entry-level Free Setup | Streaming Configuration |
|:---:|:---:|:---:|
| ASR (Speech Recognition) | FunASR (Local) | 👍XunfeiStreamASR (Xunfei Streaming) |
| LLM (Large Model) | glm-4-flash (Zhipu) | 👍qwen-flash (Ali百炼) |
| VLLM (Visual Large Model) | glm-4v-flash (Zhipu) | 👍qwen3.5-flash (Ali百炼) |
| TTS (Speech Synthesis) | EdgeTTS (Microsoft) | 👍HuoshanDoubleStreamTTS (Volcano Streaming) |
| Intent (Intent Recognition) | function_call (Function Call) | function_call (Function Call) |
| Memory (Memory Function) | mem_local_short (Local Short-term Memory) | mem_local_short (Local Short-term Memory) |

If you're concerned about component timing, please refer to the [Xiao Zhi Component Performance Test Report](https://github.com/xinnan-tech/xiaozhi-performance-research) to test actual performance in your environment following the test methods described in the report.

### 6. I speak slowly, and Xiao Zhi often interrupts me 🗣️

Recommendation: In the configuration file, find the following section and increase the value of `min_silence_duration_ms` (for example, change it to `1000`):

```yaml
VAD:
  SileroVAD:
    threshold: 0.5
    model_dir: models/snakers4_silero-vad
    min_silence_duration_ms: 700  # If speech pauses are longer, you can increase this value
```

### 7. Deployment-related Tutorials
1. [How to perform minimal deployment](./Deployment.md)<br/>
2. [How to perform full module deployment](./Deployment_all.md)<br/>
3. [How to deploy MQTT gateway to enable MQTT+UDP protocol](./mqtt-gateway-integration.md)<br/>
4. [How to automatically pull the latest code of this project, compile, and start](./dev-ops-integration.md)<br/>
5. [How to integrate with Nginx](https://github.com/xinnan-tech/xiaozhi-esp32-server/issues/791)<br/>
6. [How to compile your own Docker image after modifying the code](./docker-build.md)<br/>

### 8. Firmware Compilation Related Tutorials
1. [How to compile Xiao Zhi firmware yourself](./firmware-build.md)<br/>
2. [How to modify OTA address based on Xiege's pre-compiled firmware](./firmware-setting.md)<br/>
3. [How to configure firmware OTA auto-upgrade for single-module deployment](./ota-upgrade-guide.md)<br/>

### 9. Expansion-related Tutorials
1. [How to enable mobile phone number registration for Smart Console](./ali-sms-integration.md)<br/>
2. [How to integrate HomeAssistant for smart home control](./homeassistant-integration.md)<br/>
3. [How to enable vision model for photo recognition](./mcp-vision-integration.md)<br/>
4. [How to deploy MCP endpoint](./mcp-endpoint-enable.md)<br/>
5. [How to connect to MCP endpoint](./mcp-endpoint-integration.md)<br/>
6. [How MCP methods get device information](./mcp-get-device-info.md)<br/>
7. [How to enable voiceprint recognition](./voiceprint-integration.md)<br/>
8. [News plugin source configuration guide](./newsnow_plugin_config.md)<br/>
9. [RAGFlow integration guide](./ragflow-integration.md)<br/>
10. [How to deploy context provider](./context-provider-integration.md)<br/>
11. [How to integrate PowerMem smart memory](./powermem-integration.md)<br/>
12. [How to configure weather plugin for weather queries](./weather-integration.md)<br/>
13. [How to enable device calling plugin](./device-call-guide.md)<br/>
14. [How to enable web search function](./web-search-integration.md)<br/>

### 10. Digital Human-related Tutorials
1. [Digital human digital-human startup method](./digital-human-wakeword.md)<br/>
2. [How to deploy digital human digital-human on N100 mini PC](./all-in-one-digital-human-setup.md)<br/>

### 11. Voice Cloning, Local Voice Deployment Related Tutorials
1. [How to clone voice in Smart Console](./huoshan-streamTTS-voice-cloning.md)<br/>
2. [How to deploy integrated index-tts local voice](./index-stream-integration.md)<br/>
3. [How to deploy integrated fish-speech local voice](./fish-speech-integration.md)<br/>
4. [How to deploy integrated PaddleSpeech local voice](./paddlespeech-deploy.md)<br/>

### 12. Performance Testing Tutorials
1. [Speed testing guide for each component](./performance_tester.md)<br/>
2. [Regular public test results](https://github.com/xinnan-tech/xiaozhi-performance-research)<br/>

### 13. More questions, you can contact us for feedback 💬

You can submit your questions in [issues](https://github.com/xinnan-tech/xiaozhi-esp32-server/issues).

> [!NOTE] 
> This translation requires manual review as it involves technical documentation that may benefit from human verification for accuracy.