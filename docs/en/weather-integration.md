# Weather Plugin Usage Guide

## Overview

The weather plugin `get_weather` is one of the core features of the Xiaozhi ESP32 voice assistant, supporting voice queries for weather information across the country. The plugin is based on the HeWeather API, providing real-time weather and 7-day weather forecast functions.

## API Key Application Guide

### 1. Register HeWeather Account

1. Visit [HeWeather Console](https://console.qweather.com/)
2. Register an account and complete email verification
3. Log into the console

### 2. Create Application to Get API Key

1. After entering the console, click the right-side ["Project Management"](https://console.qweather.com/project?lang=zh) → "Create Project"
2. Fill in the project information:
   - **Project Name**: Such as "Xiaozhi Voice Assistant"
3. Click Save
4. After the project is created, click "Create Credentials" in this project
5. Fill in the credential information:
    - **Credential Name**: Such as "Xiaozhi Voice Assistant"
    - **Authentication Method**: Select "API Key"
6. Click Save
7. Copy the `API Key` in the credentials, this is the first critical configuration information

### 3. Get API Host

1. In the console, click ["Settings"](https://console.qweather.com/setting?lang=zh) → "API Host"
2. View the dedicated `API Host` address assigned to you, this is the second critical configuration information

The above operations will provide two important configuration items: `API Key` and `API Host`

## Configuration Methods (Choose One)

### Method 1. If you are using Xiaozhi Console deployment (Recommended)

1. Log into Xiaozhi Console
2. Enter the "Role Configuration" page
3. Select the agent to configure
4. Click the "Edit Functions" button
5. Find the "Weather Query" plugin in the parameter configuration area on the right
6. Check "Weather Query"
7. Fill the first critical configuration `API Key` you copied into the `Weather Plugin API Key` field
8. Fill the second critical configuration `API Host` you copied into the `Developer API Host` field
9. Save the configuration, then save the agent configuration

### Method 2. If you are deploying only the single-module xiaozhi-server

Configure in `data/.config.yaml`:

1. Fill the first critical configuration `API Key` you copied into the `api_key` field
2. Fill the second critical configuration `API Host` you copied into the `api_host` field
3. Fill your city into the `default_location` field, for example `Guangzhou`

```yaml
plugins:
  get_weather:
    api_key: "Your HeWeather API Key"
    api_host: "Your HeWeather API Host Address"
    default_location: "Your Default Query City"
```

> [!NOTE] 
> This translation requires manual review as it involves technical documentation that may benefit from human verification for accuracy.