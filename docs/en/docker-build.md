# Local Docker Image Compilation Method

The project now uses GitHub's `automatic Docker image compilation` feature. If you pull images released by the project, you don't need to compile images yourself, so you can ignore this document.

If you have modified the source code and want to deploy and run using `Docker`, you can follow these steps:

## 1. Environment Preparation

Install Docker:
```bash
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

## 2. Compile Images

After you have modified the code and need to compile new images, follow these steps:

Prepare your `username` and `new version number`.
- This `username` is the username you registered on `Docker Hub`, for example `xiaozhi`. Of course, if you don't need to push to `Docker Hub`, you can define it freely.
- This `new version number` is the version of the image you're compiling, for example `1.2.3`, you can customize it or use a date format (such as `20260609`) mainly to distinguish it from the currently running version number, and also to make it easy to recall when you built it next time. Do not use the same version number as the one currently running on your local machine.

Enter the root directory of the `xiaozhi-esp32-server` project to compile the server and web images:

```bash
cd project root directory

# Compile server image
docker build -f Dockerfile-server -t your_username/xiaozhi-esp32-server:new_version_number .

# Compile web image
docker build -f Dockerfile-web -t your_username/xiaozhi-esp32-server-web:new_version_number .
```

## 3. Modify Docker Compose Configuration

```bash
cd main/xiaozhi-server
```

Edit the `docker-compose_all.yml` file, replacing the image version with the one you just compiled:

```yaml
services:
  xiaozhi-esp32-server:
    image: your_username/xiaozhi-esp32-server:new_version_number   # Modify to your image address
    ...

  xiaozhi-esp32-server-web:
    image: your_username/xiaozhi-esp32-server-web:new_version_number   # Modify to your image address
    ...
```

## 4. Restart Services

```bash
# Stop old containers
docker compose -f docker-compose_all.yml down

# Start new containers
docker compose -f docker-compose_all.yml up -d
```

## 5. Verification

Check logs to confirm the service started normally:

```bash
# Check server logs
docker logs -f -n 50 xiaozhi-esp32-server

# Check web logs
docker logs -f -n 50 xiaozhi-esp32-server-web
```

> [!NOTE] 
> This translation requires manual review as it involves technical documentation that may benefit from human verification for accuracy.