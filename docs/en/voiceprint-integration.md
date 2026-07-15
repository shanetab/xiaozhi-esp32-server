# Voiceprint Recognition Enable Guide

This tutorial consists of 3 parts:
- 1. How to deploy the voiceprint recognition service
- 2. How to configure the voiceprint recognition interface when deploying all modules
- 3. How to configure the voiceprint recognition in minimal deployment

# 1. How to Deploy the Voiceprint Recognition Service

## Step 1: Download the Voiceprint Recognition Project Source Code

Open the [Voiceprint Recognition Project Address](https://github.com/xinnan-tech/voiceprint-api) in your browser.

After opening, find the green button labeled `Code` and click it. You will then see the `Download ZIP` button.

Click it to download the project source code zip package. After downloading to your computer and extracting it, its name may be `voiceprint-api-main`. You need to rename it to `voiceprint-api`.

## Step 2: Create Database and Table

Voiceprint recognition requires a `mysql` database dependency. If you have previously deployed the `Xiaozhi Console`, you already have `mysql` installed. You can reuse it.

You can test whether you can access `mysql`'s `3306` port using the `telnet` command on the host machine:
```
telnet 127.0.0.1 3306
```
If you can access the 3306 port, please ignore the following content and proceed to Step 3.

If you cannot access it, you need to recall how your `mysql` was installed.

If your mysql was installed using installation packages, it means your `mysql` has network isolation. You may need to first resolve access to the `3306` port of `mysql`.

If your `mysql` was installed via the project's `docker-compose_all.yml`. You need to find the `docker-compose_all.yml` file you used to create the database and modify the following content:

Before modification:
```
  xiaozhi-esp32-server-db:
    ...
    networks:
      - default
    expose:
      - "3306:3306"
```

After modification:
```
  xiaozhi-esp32-server-db:
    ...
    networks:
      - default
    ports:
      - "3306:3306"
```

Note that you need to change `expose` to `ports` for `xiaozhi-esp32-server-db`. After modification, you need to restart. Here's the command to restart mysql:
```
# Enter the folder where your docker-compose_all.yml is located, for example mine is xiaozhi-server
cd xiaozhi-server
docker compose -f docker-compose_all.yml down
docker compose -f docker-compose.yml up -d
```

After restarting, test accessing `mysql`'s `3306` port from the host machine:
```
telnet 127.0.0.1 3306
```

Under normal circumstances, you should be able to access these ports.

## Step 3: Create Database and Table

If your host machine can access the mysql database, then create a database named `voiceprint_db` and a `voiceprints` table in mysql.

```
CREATE DATABASE voiceprint_db CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;

USE voiceprint_db;

CREATE TABLE voiceprints (
    id INT AUTO_INCREMENT PRIMARY KEY,
    speaker_id VARCHAR(255) NOT NULL UNIQUE,
    feature_vector LONGBLOB NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    INDEX idx_speaker_id (speaker_id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

## Step 4: Configure Database Connection

Enter the `voiceprint-api` folder and create a folder named `data`.

Copy the `voiceprint.yaml` from the root directory of `voiceprint-api` to the `data` folder, and rename it to `.voiceprint.yaml`.

Next, you need to configure the database connection in `.voiceprint.yaml`:

```
mysql:
  host: "127.0.0.1"
  port: 3306
  user: "root"
  password: "your_password"
  database: "voiceprint_db"
```

Note! Since your voiceprint recognition service is deployed using docker, the `host` needs to be filled with the local network IP of the machine where `mysql` is located.

Note! Since your voiceprint recognition service is deployed using docker, the `host` needs to be filled with the local network IP of the machine where `mysql` is located.

Note! Since your voiceprint recognition service is deployed using docker, the `host` needs to be filled with the local network IP of the machine where `mysql` is located.

## Step 5: Start the Program

This is a very simple project and is recommended to be run using docker. However, if you don't want to use docker, you can refer to [this page](https://github.com/xinnan-tech/voiceprint-api/blob/main/README.md) to run it from source code. Here's the docker running method:

```
# Enter the root directory of this project source code
cd voiceprint-api

# Clear cache
docker compose -f docker-compose.yml down
docker stop voiceprint-api
docker rm voiceprint-api
docker rmi ghcr.nju.edu.cn/xinnan-tech/voiceprint-api:latest

# Start docker container
docker compose -f docker-compose.yml up -d
# View logs
docker logs -f voiceprint-api
```

At this point, the logs will output similar logs:
```
250711 INFO-🚀 Start: Production environment service startup (Uvicorn), listening address: 0.0.0.0:8005
250711 INFO-============================================================
250711 INFO-Voiceprint interface address: http://127.0.0.1:8005/voiceprint/health?key=abcd
250711 INFO-============================================================
```

Please copy the voiceprint interface address:
```
Since you are deploying with docker, do not directly use the above address!
Since you are deploying with docker, do not directly use the above address!
Since you are deploying with docker, do not directly use the above address!

Copy the address and put it in a draft. You need to know your computer's local network IP. For example, my computer's local network IP is `192.168.1.25`, so
my original interface address
```
http://127.0.0.1:8005/voiceprint/health?key=abcd

```
needs to be changed to
```
http://192.168.1.25:8005/voiceprint/health?key=abcd
```

After making the changes, use your browser to directly access the `Voiceprint Interface Address`. When the browser shows code similar to the following, it means it's successful:
```
{"total_voiceprints":0,"status":"healthy"}
```

Please keep the modified `Voiceprint Interface Address` handy, you'll need it in the next step.

# 2. How to Configure Voiceprint Recognition When Deploying All Modules

## Step 1: Configure Interface

First, you need to enable the voiceprint recognition feature. In Xiaozhi Console, click the top `Parameter Dictionary`, and in the dropdown menu, click the `System Function Configuration` page. Check `Voiceprint Recognition` on the page, and click `Save Configuration`. You will then see the `Voiceprint Recognition` button on the new agent card.

If you are deploying all modules, log into Xiaozhi Console with an administrator account, click the top `Parameter Dictionary`, and select the `Parameter Management` function.

Then search for the parameter `server.voice_print`. At this time, its value should be `null`. Click the Modify button, and paste the `Voiceprint Interface Address` obtained in the previous step into the `Parameter Value`. Then save.

If you save successfully, everything is going well, and you can view the effect in the agent. If not, it means that Xiaozhi Console cannot access the voiceprint recognition, and it's very likely due to network firewall or incorrect local network IP.

## Step 2: Set Agent Memory Mode

Enter your agent's role configuration and set the memory to `Local Short-term Memory`. Make sure to enable `Report Text + Voice`.

## Step 3: Chat with Your Agent

Power on your device and have a normal conversation with it using normal speech rate and tone.

## Step 4: Set Voiceprint

In Xiaozhi Console, on the `Agent Management` page, in the agent's panel, there is a `Voiceprint Recognition` button. Click it. There is an `Add` button at the bottom. You can register voiceprints for someone's speech.

In the pop-up box, it is recommended to fill in the `Description` property, which can be the person's profession, personality, hobbies, etc. This helps the agent analyze and understand the speaker.

## Step 5: Chat with Your Agent Again

Power on your device and ask it, "Do you know who I am?" If it can answer correctly, the voiceprint recognition function is working properly.

# 3. How to Configure Voiceprint Recognition in Minimal Deployment

## Step 1: Configure Interface

Open the `xiaozhi-server/data/.config.yaml` file (create if it doesn't exist), and then add/modify the following content:

```
# Voiceprint Recognition Configuration
voiceprint:
  # Voiceprint interface address
  url: Your Voiceprint Interface Address
  # Speaker Configuration: speaker_id, name, description
  speakers:
    - "test1,Zhang San,Zhang San is a programmer"
    - "test2,Li Si,Li Si is a product manager"
    - "test3,Wang Wu,Wang Wu is a designer"
```

Paste the `Voiceprint Interface Address` obtained in the previous step into `url`, and then save.

The `speakers` parameter should be added according to requirements. Please note that the `speaker_id` parameter here will be used in the next step for voiceprint registration.

## Step 2: Register Voiceprint

If you have already started the voiceprint service, visit `http://localhost:8005/voiceprint/docs` in your local browser to view the API documentation. This only explains how to use the voiceprint registration API.

The API address for registering voiceprint is `http://localhost:8005/voiceprint/register`, with POST request method.

The request header needs to include Bearer Token authentication, where the token is the part after `?key=` in the `Voiceprint Interface Address`. For example, if my voiceprint registration address is `http://127.0.0.1:8005/voiceprint/health?key=abcd`, then my token is `abcd`.

The request body includes the speaker ID (speaker_id) and WAV audio file (file). An example request is as follows:

```
curl -X POST \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN" \
  -F "speaker_id=your_speaker_id_here" \
  -F "file=@/path/to/your/file" \
  http://localhost:8005/voiceprint/register
```

Here, `file` is the audio file of the speaker speaking, and `speaker_id` needs to match the `speaker_id` in the first step configuration. For example, if I want to register Zhang San's voiceprint, and I filled Zhang San's `speaker_id` as `test1` in `.config.yaml`, then when registering Zhang San's voiceprint, the request body fills `speaker_id` as `test1`, and `file` fills in the audio file of Zhang San speaking.

## Step 3: Start Service

Start the Xiaozhi server and voiceprint service to use it normally.

> [!NOTE] 
> This translation requires manual review as it involves technical documentation that may benefit from human verification for accuracy.