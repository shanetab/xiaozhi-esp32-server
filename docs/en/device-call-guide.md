# Inter-Device Calling Plugin Usage Guide

## Overview

The device calling function allows two configured devices to communicate bidirectionally through voice/data channels. When Device A calls Device B, the system implements the following process:

```
Device A → Authorization Verification → MQTT Gateway → Device B Remote Wake-up → Connection Establishment → Call Establishment
```

## Prerequisites for Using This Function

1. You must have at least two devices, and each device model must be `ESP32-S3`, as only `ESP32-S3` supports the remote wake-up function.
2. Your devices must have `two microphones`. However, if your device only has `a single microphone`, you can still experience this function, but there will be significant lag.
3. You must deploy this project using [Full Module Deployment](Deployment_all.md), as you need the Smart Console to manage device permissions and communication.
4. You must install and configure the `MQTT Gateway Service` deployed after `May 27, 2026` (see [MQTT Gateway Integration](mqtt-gateway-integration.md)). If you have already deployed the MQTT Gateway Service, please confirm that the code version is after `May 27, 2026`.

These are the hard requirements for using this function. The following sections will provide detailed instructions.

## Configuration Steps

### Step 1: Enable Address Book Function

1. Confirm that your Smart Console version is `0.9.4` or higher.
2. Log into the Smart Console backend.
3. Go to **System Function Configuration**.
4. In the left function list, check **Address Book**.
5. Click **Save Configuration** to confirm.

### Step 2: Configure Device-to-Device Calling Permissions

1. In the Smart Console top menu, click **Address Book**.
2. In the left intelligent agent, select your Device A from the device list (support search by MAC address or nickname).
3. In the right detail panel, find the target device B's name setting, such as **"Xiao Wang"**.
4. Check the **Calling Permission** checkbox for Device B.
5. Click **Save**.

**Bidirectional Authorization Explanation:** For devices A and B to communicate with each other, permissions must be configured on both sides in the Smart Console. For example:
- In Device A's configuration, check Device B → Device A can communicate with Device B.
- In Device B's configuration, check Device A → Device B can communicate with Device A.

### Step 3: Add Calling Tool to Agent Configuration

1. In the Smart Console top menu, click **Agent Management**.
2. In the intelligent agent related to device contact configuration, click **Edit Role**.
3. In the right detail panel, click **Edit Functions**.
4. Check the **Device Calling Device** tool.
5. Click **Save Configuration** to confirm.
6. Click **Save Configuration** again externally to restart the device.

### Step 4: Add Remote Wake-up Tool on Firmware Side

1. Add the remote wake-up tool MCP to the [xiaozhi-esp32](https://github.com/78/xiaozhi-esp32) codebase, with version support from 2.1.0 to 2.2.6 (version from May 29, 2026).
2. Add remote wake-up function declaration in the application.h file:
    ```cpp
    void RemoteWakeup(const std::string& reason);
    ```
3. Add remote wake-up function in the application.cc file:
    ```cpp
    void Application::RemoteWakeup(const std::string& reason){
        if (!protocol_) {
            return;
        }

        auto state = GetDeviceState();
        
        if (state == kDeviceStateIdle) {
            audio_service_.EncodeWakeWord();

            if (!protocol_->IsAudioChannelOpened()) {
                SetDeviceState(kDeviceStateConnecting);
                if (!protocol_->OpenAudioChannel()) {
                    audio_service_.EnableWakeWordDetection(true);
                    return;
                }
            }
            std::string wake_word = reason;
    #if CONFIG_USE_AFE_WAKE_WORD || CONFIG_USE_CUSTOM_WAKE_WORD
            // Encode and send the wake word data to the server
            while (auto packet = audio_service_.PopWakeWordPacket()) {
                protocol_->SendAudio(std::move(packet));
            }
            // Set the chat state to wake word detected
            protocol_->SendWakeWordDetected(wake_word);
            SetListeningMode(aec_mode_ == kAecOff ? kListeningModeAutoStop : kListeningModeRealtime);
    #else
            // Set flag to play popup sound after state changes to listening
            // (PlaySound here would be cleared by ResetDecoder in EnableVoiceProcessing)
            play_popup_on_listening_ = true;
            SetListeningMode(aec_mode_ == kAecOff ? kListeningModeAutoStop : kListeningModeRealtime);
    #endif
        } else if (state == kDeviceStateSpeaking) {
            AbortSpeaking(kAbortReasonWakeWordDetected);
            SetDeviceState(kDeviceStateIdle);
        } else if (state == kDeviceStateActivating) {
            SetDeviceState(kDeviceStateIdle);
        }
    }
    ```
4. Add remote wake-up tool in the mcp_server.cc file:
    ```cpp
    AddUserOnlyTool("self.remote_wakeup", "Remote wakeup function with configurable parameters",
        PropertyList({
            Property("reason", kPropertyTypeString, "Wakeup reason"),
        }),
        [this](const PropertyList& properties) -> ReturnValue {
            std::string reason = properties["reason"].value<std::string>();
            ESP_LOGI(TAG, "Wakeup reason=%s", reason.c_str());
            auto& app = Application::GetInstance();
            app.RemoteWakeup(reason);
            return true;
    ```
5. Complete firmware flashing according to the [Firmware Compilation and Flashing Guide](firmware-build.md).
6. Regardless of whether your device has single or dual microphones, please check the AEC function during compilation!
7. Regardless of whether your device has single or dual microphones, please check the AEC function during compilation!
8. Regardless of whether your device has single or dual microphones, please check the AEC function during compilation!

### Step 5: Configure MQTT Gateway Service

1. Deploy the MQTT Gateway Service, refer to [MQTT Gateway Integration Documentation](mqtt-gateway-integration.md).
2. If already deployed, confirm the code version is from May 27, 2026.

## Call Process Description

Prepare two devices, and after configuring communication permissions in the Smart Console and adding the calling tool in the agent, speak to one Xiao Zhi: "Call XXX" and observe if Device B responds.

## Common Issues

### Q: Device B is not responding to calls?

- Check if Device B is online (Smart Console device status).
- Confirm that Device B's firmware has correctly integrated the remote wake-up tool.
- Check if the MQTT Gateway connection is normal.
- Verify that bidirectional permission configuration is complete.

### Q: Shows "No calling permission"?

- Confirm in the Smart Console that Device A has checked Device B's calling permission.
- Confirm that the configuration has been saved (not just modified without saving).

### Q: How to confirm the Address Book function is enabled?

- If the Smart Console top menu shows an "Address Book" entry, it indicates it's enabled.

### Q: I call "Zhang Shan" but it keeps recognizing as "Zhang San", what should I do?

- Check the documentation for the ASR service you're using to confirm if it supports hotword recognition.
- If you're using `FunASRServer`, you can add "Zhang Shan" to the hotword file in the container, then restart the container.
- If you're using `Volcano Engine` service, you can add a hotword file in the `Volcano Engine console`, then go back to the Smart Console's `Model Configuration page` and configure the `hotword file name` in the `Volcano Engine TTS`.

> [!NOTE] 
> This translation requires manual review as it involves technical documentation that may benefit from human verification for accuracy.