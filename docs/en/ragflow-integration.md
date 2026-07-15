# ragflow Integration Guide

This tutorial consists of two parts:

- Part 1: How to deploy ragflow
- Part 2: How to configure ragflow API in Xiaozhi Console

If you're already familiar with ragflow and have deployed it, you can skip the first part and go directly to the second part. However, if you need guidance on deploying ragflow to work with `xiaozhi-esp32-server` using `mysql` and `redis` infrastructure services to reduce resource costs, you should start from the first part.

# Part 1: How to Deploy ragflow

## Step 1: Confirm MySQL and Redis are Accessible

ragflow requires a `mysql` database dependency. If you've previously deployed the `Xiaozhi Console`, you already have `mysql` installed. You can reuse it.

You can test whether you can access `mysql`'s `3306` port and `redis`'s `6379` port using the `telnet` command on the host machine:

```shell
telnet 127.0.0.1 3306

telnet 127.0.0.1 6379
```

If you can access both ports, you can skip the following content and proceed to Step 2.

If you cannot access these ports, you need to recall how your `mysql` was installed.

If your mysql was installed using installation packages, it may have network isolation. You may need to resolve access to the `3306` port of `mysql` first.

If your `mysql` was installed via the project's `docker-compose_all.yml`, you need to find the `docker-compose_all.yml` file you used to create the database, and modify the following content:

Before modification:
```yaml
  xiaozhi-esp32-server-db:
    ...
    networks:
      - default
    expose:
      - "3306:3306"
  xiaozhi-esp32-server-redis:
    ...
    expose:
      - 6379
```

After modification:
```yaml
  xiaozhi-esp32-server-db:
    ...
    networks:
      - default
    ports:
      - "3306:3306"
  xiaozhi-esp32-server-redis:
    ...
    ports:
      - "6379:6379"
```

Note that you need to change `expose` to `ports` for both `xiaozhi-esp32-server-db` and `xiaozhi-esp32-server-redis`. After modification, you need to restart. Here's the command to restart mysql:

```shell
# Enter the folder where your docker-compose_all.yml is located, for example mine is xiaozhi-server
cd xiaozhi-server
docker compose -f docker-compose_all.yml down
docker compose -f docker-compose.yml up -d
```

After restarting, test accessing `mysql`'s `3306` port from the host machine:

```shell
telnet 127.0.0.1 3306

telnet 127.0.0.1 6379
```

Under normal circumstances, you should be able to access these ports.

## Step 2: Create Database and Tables

If you can access the mysql database from your host machine, create a database named `rag_flow` and a user `rag_flow` with password `infini_rag_flow`:

```sql
-- Create database
CREATE DATABASE IF NOT EXISTS rag_flow CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;

-- Create user and grant privileges
CREATE USER IF NOT EXISTS 'rag_flow'@'%' IDENTIFIED BY 'infini_rag_flow';
GRANT ALL PRIVILEGES ON rag_flow.* TO 'rag_flow'@'%';

-- Refresh privileges
FLUSH PRIVILEGES;
```

## Step 3: Download the ragflow Project

Find a folder on your computer to store the ragflow project. For example, I use `/home/system/xiaozhi`.

You can use the `git` command to download the ragflow project to this folder. This tutorial uses version `v0.22.0` for installation and deployment:

```
git clone https://ghfast.top/https://github.com/infiniflow/ragflow.git
cd ragflow
git checkout v0.22.0
```

After downloading, enter the `docker` folder:

```shell
cd docker
```

Modify the `docker-compose.yml` file under `ragflow/docker` to remove the `depends_on` configuration from `ragflow-cpu` and `ragflow-gpu` services, to remove the `mysql` dependency for `ragflow-cpu` service.

Before modification:
```yaml
  ragflow-cpu:
    depends_on:
      mysql:
        condition: service_healthy
    profiles:
      - cpu
  ...
  ragflow-gpu:
    depends_on:
      mysql:
        condition: service_healthy
    profiles:
      - gpu
```

After modification:
```yaml
  ragflow-cpu:
    profiles:
      - cpu
  ...
  ragflow-gpu:
    profiles:
      - gpu
```

Next, modify the `docker-compose-base.yml` file under `ragflow/docker` to remove the `mysql` and `redis` configurations:

For example, before removal:
```yaml
services:
  minio:
    image: quay.io/minio/minio:RELEASE.2025-06-13T11-33-47Z
    ...
  mysql:
    image: mysql:8.0
    ...
  redis:
    image: redis:6.2-alpine
    ...
```

After removal:
```yaml
services:
  minio:
    image: quay.io/minio/minio:RELEASE.2025-06-13T11-33-47Z
    ...
```

## Step 4: Modify Environment Variable Configuration

Edit the `.env` file under `ragflow/docker` folder. Find the following configurations and modify each one individually!

The following modification to the `.env` file will cause 60% of users to skip the `MYSQL_USER` configuration, leading to ragflow startup failure, so it needs to be emphasized three times:

Emphasis #1: If your `.env` file doesn't have `MYSQL_USER` configuration, please add this item to the configuration file!

Emphasis #2: If your `.env` file doesn't have `MYSQL_USER` configuration, please add this item to the configuration file!

Emphasis #3: If your `.env` file doesn't have `MYSQL_USER` configuration, please add this item to the configuration file!

```env
# Port settings
SVR_WEB_HTTP_PORT=8008           # HTTP port
SVR_WEB_HTTPS_PORT=8009          # HTTPS port
# MySQL configuration - Modify to your local MySQL information
MYSQL_HOST=host.docker.internal  # Use host.docker.internal to let containers access host services
MYSQL_PORT=3306                  # Local MySQL port
MYSQL_USER=rag_flow              # Username created above, add this line if missing
MYSQL_PASSWORD=infini_rag_flow   # Password set above
MYSQL_DBNAME=rag_flow            # Database name

# Redis configuration - Modify to your local Redis information
REDIS_HOST=host.docker.internal  # Use host.docker.internal to let containers access host services
REDIS_PORT=6379                  # Local Redis port
REDIS_PASSWORD=                  # If your Redis has no password, leave as is; otherwise, enter the password
```

Note: If your Redis has no password, you also need to modify `service_conf.yaml.template` under `ragflow/docker` folder to replace `infini_rag_flow` with an empty string.

Before modification:
```shell
redis:
  db: 1
  password: '${REDIS_PASSWORD:-infini_rag_flow}'
  host: '${REDIS_HOST:-redis}:6379'
```

After modification:
```shell
redis:
  db: 1
  password: '${REDIS_PASSWORD:-}'
  host: '${REDIS_HOST:-redis}:6379'
```

## Step 5: Start the ragflow Service

Execute the command:

```shell
docker-compose -f docker-compose.yml up -d
```

After successful execution, you can use `docker logs -n 20 -f docker-ragflow-cpu-1` to check the logs of the `docker-ragflow-cpu-1` service.

If there are no errors in the logs, the ragflow service has started successfully.

## Step 6: Register an Account

You can access `http://127.0.0.1:8008` in your browser and click `Sign Up` to register an account.

After successful registration, you can click `Sign In` to log into the ragflow service. If you want to disable ragflow registration so others can't register accounts, you can set the `REGISTER_ENABLED` configuration item to `0` in the `.env` file under `ragflow/docker` folder.

```dotenv
REGISTER_ENABLED=0
```

After modification, restart the ragflow service:

```shell
docker-compose -f docker-compose.yml down
docker-compose -f docker-compose.yml up -d
```

## Step 7: Configure Models for ragflow Service

You can access `http://127.0.0.1:8008` in your browser, click `Sign In` to log into the ragflow service. Click on the avatar in the top right corner to access the settings page.

First, click on `Model Providers` in the left navigation bar to enter the model configuration page. Under the `Optional Models` search box on the right, select `LLM` from the list, choose your model provider, click `Add`, and enter your API key.

Then, select `TEXT EMBEDDING`, choose your model provider from the list, click `Add`, and enter your API key.

Finally, refresh the page and click on `Set Default Models` to set your LLM and Embedding models. Please confirm that your API key has activated the corresponding services. For example, if I use an embedding model from vendor xxx, I need to check if this model requires a purchased resource package to use on the vendor's official website.

# Part 2: Configure ragflow Service

## Step 1: Login to ragflow Service

You can access `http://127.0.0.1:8008` in your browser and click `Sign In` to log into the ragflow service.

Then click on the avatar in the top right corner to access the settings page. In the left navigation bar, click on the `API` function, then click the "API Key" button. A modal will appear.

In the modal, click the "Create new Key" button to generate an API Key. Copy this `API Key`, you'll use it later.

## Step 2: Configure in Xiaozhi Console

Ensure your Xiaozhi Console version is `0.8.7` or above. Log in to the Xiaozhi Console with a super administrator account.

First, you need to enable the knowledge base feature. In the top navigation bar, click `Parameter Dictionary`, then in the dropdown menu, click `System Function Configuration` page. Check `Knowledge Base`, click `Save Configuration`. You'll then see the `Knowledge Base` feature in the navigation bar.

In the top navigation bar, click `Model Configuration`, then in the left navigation bar, click `Knowledge Base`. Find `RAG_RAGFlow` in the list, click the `Edit` button.

In `Service Address`, enter `http://your ragflow service LAN IP:8008`, for example, if my ragflow service's LAN IP is `192.168.1.100`, I would enter `http://192.168.1.100:8008`.

In `API Key`, enter the API Key you copied earlier.

Finally, click the save button.

## Step 3: Create a Knowledge Base

Log in to the Xiaozhi Console with a super administrator account. In the top navigation bar, click `Knowledge Base`, then click the `New` button in the lower left corner of the list. Fill in a name and description for the knowledge base. Click Save.

To improve the large model's understanding and recall capabilities of the knowledge base, it's recommended to fill in a meaningful name and description when creating a knowledge base. For example, if you're creating a knowledge base about `Company Introduction`, the knowledge base name could be `Company Introduction`, and the description could be `Information about the company including basic information, service items, contact number, address, etc.`.

After saving, you can see this knowledge base in the knowledge base list. Click the `View` button of the knowledge base you just created to enter the knowledge base details page.

In the knowledge base details page, click the `New` button in the lower left corner to upload documents to the knowledge base.

After uploading, you can see the uploaded document in the knowledge base details page. You can then click the `Parse` button of the document to parse it.

After parsing, you can view the sliced information. In the knowledge base details page, click the `Recall Test` button to test the knowledge base's recall/retrieval function.

## Step 4: Use ragflow Knowledge Base with XiaoZhi

Log in to the Xiaozhi Console. In the top navigation bar, click `Agent`, find the agent you want to configure, and click the `Configure Role` button.

On the left side of Intent Recognition, click the `Edit Functions` button to open a modal. In the modal, select the knowledge base you want to add. Save.

> [!NOTE] 
> This translation requires manual review as it involves technical documentation that may benefit from human verification for accuracy.