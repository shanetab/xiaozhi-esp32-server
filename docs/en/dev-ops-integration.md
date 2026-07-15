# Full Module Source Code Deployment Automatic Upgrade Method

This tutorial is for enthusiasts who have deployed the full module source code, showing how to use automatic commands to automatically pull source code, automatically compile, and automatically start port operations. This achieves the most efficient system upgrades.

The test platform of this project `https://2662r3426b.vicp.fun` has been using this method since it opened, with good results.

The tutorial can refer to the Bilibili blogger `Bile Labs`'s video tutorial: [《Open Source Xiao Zhi Server xiaozhi-server Automatic Update and Latest Version MCP Access Point Configuration Tutorial》](https://www.bilibili.com/video/BV15H37zHE7Q)

# Start Conditions
- Your computer/server is running Linux operating system
- You have already completed the entire process
- You like to follow the latest features but find manual deployment a bit troublesome and expect an automatic update method

The second condition must be met because certain files involved in this tutorial, such as JDK, Node.js environment, Conda environment, etc., are only available after you have completed the entire process. If you have not completed it, when I mention a certain file, you may not know what it means.

# Tutorial Effect
- Solves the problem of not being able to pull the latest project source code domestically
- Automatically pulls code and compiles frontend files
- Automatically pulls code, compiles Java files, automatically kills the 8002 port, and automatically starts the 8002 port
- Automatically pulls Python code, automatically kills the 8000 port, and automatically starts the 8000 port

# Step 1: Choose Your Project Directory

For example, I plan my project directory to be a new blank directory. If you don't want to make mistakes, you can follow my example:
```
/home/system/xiaozhi
```

# Step 2: Clone This Project

At this point, you need to execute the first command to pull the source code. This command is suitable for servers and computers with domestic networks, no need to bypass firewall:
```
cd /home/system/xiaozhi
git clone https://ghproxy.net/https://github.com/xinnan-tech/xiaozhi-esp32-server.git
```

After execution, your project directory will have a new folder `xiaozhi-esp32-server`, which is the project source code.

# Step 3: Copy Basic Files

If you have already completed the entire process, you will be familiar with the funasr model file `xiaozhi-server/models/SenseVoiceSmall/model.pt` and your private configuration file `xiaozhi-server/data/.config.yaml`.

At this point, you need to copy the `model.pt` file to the new directory. You can do this:
```
# Create required directories
mkdir -p /home/system/xiaozhi/xiaozhi-esp32-server/main/xiaozhi-server/data/

cp your original .config.yaml full path /home/system/xiaozhi/xiaozhi-esp32-server/main/xiaozhi-server/data/.config.yaml
cp your original model.pt full path /home/system/xiaozhi/xiaozhi-esp32-server/main/xiaozhi-server/models/SenseVoiceSmall/model.pt
```

# Step 4: Create Three Automatic Compilation Files

## 4.1 Automatic Compilation of manager-web Module

In the `/home/system/xiaozhi/` directory, create a file named `update_8001.sh` with the following content:
```
cd /home/system/xiaozhi/xiaozhi-esp32-server
git fetch --all
git reset --hard
git pull origin main


cd /home/system/xiaozhi/xiaozhi-esp32-server/main/manager-web
npm install
npm run build
rm -rf /home/system/xiaozhi/manager-web
mv /home/system/xiaozhi/xiaozhi-esp32-server/main/manager-web/dist /home/system/xiaozhi/manager-web
```

After saving, execute the permission command:
```
chmod 777 update_8001.sh
```

After execution, continue below.

## 4.2 Automatic Compilation and Running of manager-api Module

In the `/home/system/xiaozhi/` directory, create a file named `update_8002.sh` with the following content:
```
cd /home/system/xiaozhi/xiaozhi-esp32-server
git pull origin main


cd /home/system/xiaozhi/xiaozhi-esp32-server/main/manager-api
rm -rf target
mvn clean package -Dmaven.test.skip=true
cd /home/system/xiaozhi/

# Find the process ID occupying port 8002
PID=$(sudo netstat -tulnp | grep 8002 | awk '{print $7}' | cut -d'/' -f1)

rm -rf /home/system/xiaozhi/xiaozhi-esp32-api.jar
mv /home/system/xiaozhi/xiaozhi-esp32-server/main/manager-api/target/xiaozhi-esp32-api.jar /home/system/xiaozhi/xiaozhi-esp32-api.jar

# Check if process ID was found
if [ -z "$PID" ]; then
  echo "No process occupying port 8002 found"
else
  echo "Found process occupying port 8002, process ID: $PID"
  # Kill the process
  kill -9 $PID
  kill -9 $PID
  echo "Process $PID killed"
fi

nohup java -jar xiaozhi-esp32-api.jar --spring.profiles.active=dev &

tail tail -f nohup.out
```

After saving, execute the permission command:
```
chmod 777 update_8002.sh
```

After execution, continue below.

## 4.3 Automatic Compilation and Running of Python Project

In the `/home/system/xiaozhi/` directory, create a file named `update_8000.sh` with the following content:
```
cd /home/system/xiaozhi/xiaozhi-esp32-server
git pull origin main

# Find the process ID occupying port 8000
PID=$(sudo netstat -tulnp | grep 8000 | awk '{print $7}' | cut -d'/' -f1)

# Check if process ID was found
if [ -z "$PID" ]; then
  echo "No process occupying port 8000 found"
else
  echo "Found process occupying port 8000, process ID: $PID"
  # Kill the process
  kill -9 $PID
  kill -9 $PID
  echo "Process $PID killed"
fi
cd main/xiaozhi-server
# Initialize conda environment
source ~/.bashrc
conda activate xiaozhi-esp32-server
pip install -r requirements.txt
nohup python app.py >/dev/null &
tail -f /home/system/xiaozhi/xiaozhi-esp32-server/main/xiaozhi-server/tmp/server.log
```

After saving, execute the permission command:
```
chmod 777 update_8000.sh
```

After execution, continue below.

# Daily Updates

After setting up all the scripts, for daily updates, we just need to execute the following commands in sequence to achieve automatic updates and startups:
```
cd /home/system/xiaozhi
# Update and start Java program
./update_8001.sh
# Update web program
./update_8002.sh
# Update and start Python program
./update_8000.sh


# Later, if you want to view Java logs, execute the following command
tail -f nohup.out
# Later, if you want to view Python logs, execute the following command
tail -f /home/system/xiaozhi/xiaozhi-esp32-server/main/xiaozhi-server/tmp/server.log
```

# Notes

The test platform `https://2662r3426b.vicp.fun` uses nginx for reverse proxy. The detailed nginx.conf configuration can [be referenced here](https://github.com/xinnan-tech/xiaozhi-esp32-server/issues/791)

## Common Issues

### 1. Why is port 8001 not visible?
Answer: 8001 is used in development environments for running the frontend port. If you are deploying on a server, it is not recommended to use `npm run serve` to start the 8001 port to run the frontend, but instead compile it into HTML files as shown in this tutorial, then use nginx to manage access.

### 2. Do I need to manually update SQL statements each time I update?
Answer: No, because the project uses **Liquibase** to manage database versions and will automatically execute new SQL scripts.

> [!NOTE] 
> This translation requires manual review as it involves technical documentation that may benefit from human verification for accuracy.