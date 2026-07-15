# All-in-One Digital Human Setup Guide

This project is used to deploy a complete digital human display system on x86 architecture devices (such as mini PCs, industrial computers, and ordinary computers), achieving the following functions:
- Automatically enter Kiosk full-screen browser on startup to display a specified webpage
- Run wake word detection service in the background, supporting voice interaction

> **Note**: This document uses the **Intel N100 Mini PC (Tianhong QN10-100B4)** as an example for deployment demonstration. Other x86 devices can refer to this for adjustments (note network configuration and audio device differences).

## Suitable Environment

| Project | Description |
|---------|-------------|
| Example Hardware | Tianhong QN10-100B4 (Intel N100) |
| Operating System | Ubuntu 24.04 LTS (Noble Numbat) |
| Example User | xz (please replace with actual information) |
| Network | Wi-Fi connection, fixed IP (can be changed to wired as needed) |

## Deployment Process

1. System initialization (source replacement, network connection)
2. Install graphics components and Kiosk browser
3. Configure auto-login and graphical interface
4. Deploy wake word service (Python environment + microphone)
5. Optimize boot speed and hide startup information

---

### System Initialization (Source Replacement, Network Connection)

```
sudo cp /etc/apt/sources.list /etc/apt/sources.list.bak

sudo tee /etc/apt/sources.list > /dev/null <<EOF
deb http://mirrors.aliyun.com/ubuntu/ noble main restricted universe multiverse
# deb-src http://mirrors.aliyun.com/ubuntu/ noble main restricted universe multiverse

deb http://mirrors.aliyun.com/ubuntu/ noble-security main restricted universe multiverse
# deb-src http://mirrors.aliyun.com/ubuntu/ noble-security main restricted universe multiverse

deb http://mirrors.aliyun.com/ubuntu/ noble-updates main restricted universe multiverse
# deb-src http://mirrors.aliyun.com/ubuntu/ noble-updates main restricted universe multiverse

deb http://mirrors.aliyun.com/ubuntu/ noble-proposed main restricted universe multiverse
# deb-src http://mirrors.aliyun.com/ubuntu/ noble-proposed main restricted universe multiverse

deb http://mirrors.aliyun.com/ubuntu/ noble-backports main restricted universe multiverse
# deb-src http://mirrors.aliyun.com/ubuntu/ noble-backports main restricted universe multiverse
EOF

echo 'Acquire::ForceIPv4 "true";' | sudo tee /etc/apt/apt.conf.d/99force-ipv4
```

Install network management tools (can be ignored if already present)

Bash

```
sudo apt update
sudo apt install network-manager -y
sudo systemctl start NetworkManager
sudo systemctl enable NetworkManager
```

Set Wi-Fi password, fixed IP

> **Reminder**: The Wi-Fi name, password, and IP address in the following commands are examples. Please replace them with your actual information.

Bash

```
sudo nmcli device wifi connect "MERCURY_1812" password "12345678"

sudo nmcli connection modify "MERCURY_1812" ipv4.addresses "192.168.0.86/24" ipv4.gateway "192.168.0.1" ipv4.dns "8.8.8.8,114.114.114.114" ipv4.method "manual"

sudo nmcli connection up "MERCURY_1812"
```

### Step 1: Install Core Graphics Components and Browser

Here we adhere to "minimalism", firmly refusing to install unnecessary desktop environments (such as GNOME/KDE), only installing underlying drivers, the lightest window manager (Openbox), mouse hiding tools, and Chromium browser.

Bash

```
sudo timedatectl set-timezone Asia/Shanghai


sudo apt install net-tools vim fonts-wqy-microhei fonts-wqy-zenhei alsa-utils pulseaudio -y
sudo apt install --no-install-recommends xserver-xorg x11-xserver-utils xinit openbox unclutter -y

wget https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb
sudo apt install ./google-chrome-stable_current_amd64.deb -y
rm google-chrome-stable_current_amd64.deb

sudo apt purge snapd -y

```

### Step 2: Configure Auto Login at TTY1 on Boot

To avoid the awkwardness of manually entering account passwords, we modify the systemd service to automatically log in as the `xz` user as soon as the system boots. We use a one-click write command to completely avoid issues caused by improper operations with `nano` or `vi` that might result in failed saves.

**1. Create configuration directory:**

Bash

```
sudo mkdir -p /etc/systemd/system/getty@tty1.service.d/
```

**2. Write auto-login rules:**

Bash

```
echo -e "[Service]\nExecStart=\nExecStart=-/sbin/agetty --autologin xz --noclear %I \$TERM" | sudo tee /etc/systemd/system/getty@tty1.service.d/override.conf
```

**3. Reload services and set default boot target:**

Bash

```
sudo systemctl daemon-reload
sudo systemctl set-default multi-user.target
```

### Step 3: Configure Automatic Launch of Graphical Interface After Login

After automatic system login, the default is to stay on a black background with white text command line. We need to configure a script to immediately start the X11 graphical environment upon login.

**1. Trigger the `startx` startup logic:**

Simply append the trigger code to your personal environment configuration file:

Bash

```
cat << 'EOF' >> ~/.bash_profile
if [ -z "$DISPLAY" ] && [ "$(fgconsole)" -eq 1 ]; then
    exec startx
fi
EOF
```

**2. Tell `startx` to start Openbox:**

Bash

```
echo "exec openbox-session" > ~/.xinitrc
```

### Step 4: Configure "Iron Wall" Openbox and Browser

This is the most critical step: disable screen sleep, hide mouse, lock browser in full-screen, and write a "dead loop" to ensure the browser restarts instantly if accidentally closed.

**1. Create Openbox configuration directory:**

Bash

```
mkdir -p ~/.config/openbox
```

**2. Write auto-start script (`autostart`):**

Copy and paste the entire code below (this will automatically write all protection rules to the file):

Bash

```
cat << 'EOF' > ~/.config/openbox/autostart
# Disable screen saver
xset -dpms
xset s noblank
xset s off

# Hide mouse
unclutter -idle 0.1 -root &

# Dead loop to start Chromium (restarts instantly if crashed or closed)
while true; do
    google-chrome \
        --kiosk \
        --no-first-run \
        --no-default-browser-check \
        --disable-infobars \
        --disable-session-crashed-bubble \
        --disable-translate \
        --disable-external-intent-requests \
        --autoplay-policy=no-user-gesture-required \
        --use-fake-ui-for-media-stream \
        "https://www.douyin.com"
    sleep 2
done &
EOF
```

**3. Disable `Alt+F4` exit shortcut:**

To prevent others from forcibly closing windows by plugging in a keyboard, we remove Openbox's default system shortcuts.

Bash

```
cp /etc/xdg/openbox/rc.xml ~/.config/openbox/
sed -i '/<keybind key="A-F4">/,/<\/keybind>/d' ~/.config/openbox/rc.xml
```

### Step 5: Restart and Verify Results

If you unplug the network cable or don't need to wait for all network connections to be established, you can disable the network waiting service to avoid boot lag.

Bash

```
sudo systemctl mask systemd-networkd-wait-online.service
sudo systemctl mask NetworkManager-wait-online.service
```

Hide boot information (GRUB)

Bash

```
sudo sed -i 's/GRUB_CMDLINE_LINUX_DEFAULT=.*/GRUB_CMDLINE_LINUX_DEFAULT="quiet loglevel=3 systemd.show_status=false vt.global_cursor_default=0"/g' /etc/default/grub

echo 'GRUB_TIMEOUT_STYLE="hidden"' | sudo tee -a /etc/default/grub
echo 'GRUB_RECORDFAIL_TIMEOUT=0' | sudo tee -a /etc/default/grub

sudo update-grub
```

Set volume to 100%, then reboot:

Bash

```
amixer -q sset Master 100% unmute
sudo reboot
```

### Deploy Wake Word Service

To deploy the wake word detection service on an all-in-one machine, you need to install the Python environment, upload project files, configure Camera microphone, and set up auto-start.

#### 1. Install Miniconda

```bash
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh
bash Miniconda3-latest-Linux-x86_64.sh -b -p $HOME/miniconda3
~/miniconda3/bin/conda init bash
source ~/.bashrc
rm Miniconda3-latest-Linux-x86_64.sh
```

Ensure automatic entry into conda environment on login

```bash
if ! grep -q '.bashrc' ~/.bash_profile; then
    cat << 'EOF' >> ~/.bash_profile

if [ -f ~/.bashrc ]; then
    . ~/.bashrc
fi
EOF
fi
```

#### 2. Create Python Virtual Environment

```bash
conda create -n test python=3.10 -y
conda activate test
```

If you encounter the "Terms of Service have not been accepted" error, run:

```bash
conda tos accept --override-channels --channel https://repo.anaconda.com/pkgs/main
conda tos accept --override-channels --channel https://repo.anaconda.com/pkgs/r
```

#### 3. Upload Project Files

Upload the entire `main/digital-human/` directory from your development machine to the all-in-one machine's `~/digital-human/` directory:

```bash
# Run on development machine (replace <all-in-one machine IP> with actual IP)
scp -r main/digital-human/ xz@<all-in-one machine IP>:~/digital-human/
```

#### 4. Install System Dependencies

The wake word service requires audio capture libraries and ALSA PulseAudio plugins:

```bash
sudo apt install libportaudio2 portaudio19-dev libasound2-plugins -y
```

#### 5. Install Python Dependencies

```bash
cd ~/digital-human/wakeword_runtime
pip install numpy
pip install -r requirements.txt
```

#### 6. Download Wake Word Model

The model files are not included in the project and need to be downloaded and configured separately. See the "Model Download" section in [docs/digital-human-wakeword.md](digital-human-wakeword.md).

#### 7. Modify Openbox Auto-start Script

You need to add PulseAudio and Camera microphone configuration to the autostart, and change the Chrome address to the test page.

First, confirm the device name of the Camera microphone in PulseAudio:

```bash
pulseaudio --start
pactl list sources short
```

Find the line containing `USB_Camera`, and note the complete name, for example:

```
alsa_input.usb-SN0002_2K_USB_Camera_46435000_P030D00_SN0002-02.mono-fallback
```

Then overwrite autostart with the complete content (replace `TARGET_MIC` with your actual device name):

```bash
cat << 'EOF' > ~/.config/openbox/autostart
# 1. Start audio service and wait a bit
pulseaudio --start
sleep 1

# 2. Lock the Camera microphone (replace with your actual device name)
TARGET_MIC="alsa_input.usb-SN0002_2K_USB_Camera_46435000_P030D00_SN0002-02.mono-fallback"

# 3. Set as system default microphone
pactl set-default-source "$TARGET_MIC"

# 4. Unmute
pactl set-source-mute "$TARGET_MIC" 0

# 5. Set volume to 100%
pactl set-source-volume "$TARGET_MIC" 100%

# --- Minimal desktop and browser environment configuration ---

# Disable screen saver
xset -dpms
xset s noblank
xset s off

# Hide mouse
unclutter -idle 0.1 -root &

# Dead loop to start browser (restarts instantly if crashed or closed)
while true; do
    google-chrome \
        --kiosk \
        --no-first-run \
        --no-default-browser-check \
        --disable-infobars \
        --disable-session-crashed-bubble \
        --disable-translate \
        --disable-external-intent-requests \
        --autoplay-policy=no-user-gesture-required \
        --use-fake-ui-for-media-stream \
        "http://127.0.0.1:8006/index.html"
    sleep 2
done &
EOF
```

#### 8. Configure Wake Word Service Auto-start

Create a systemd service file to automatically run the wake word service on boot.

First, confirm the current user's UID:

```bash
id -u $(whoami)
```

Then replace `1000` below with the UID you found (usually the first user is 1000):

```bash
sudo tee /etc/systemd/system/digital-human.service << 'EOF'
[Unit]
Description=Digital Human Runtime
After=network.target sound.target

[Service]
Type=simple
User=xz
Environment=XDG_RUNTIME_DIR=/run/user/1000
Environment=PULSE_SERVER=unix:/run/user/1000/pulse/native
WorkingDirectory=/home/xz/digital-human
ExecStartPre=/bin/sleep 10
ExecStart=/home/xz/miniconda3/envs/test/bin/python start.py
Restart=on-failure
RestartSec=10

[Install]
WantedBy=multi-user.target
EOF
```

> **Important Note**:
> - `User=xz` — Replace with your actual username
> - `/run/user/1000` — Replace with your actual UID
> - `WorkingDirectory` and `ExecStart` paths — Replace with your actual deployment paths
> - The PulseAudio environment variables in `Environment` **must be retained**, otherwise the wake word service and browser cannot use the Camera microphone simultaneously

Enable and start the service:

```bash
sudo systemctl daemon-reload
sudo systemctl enable digital-human
sudo systemctl start digital-human
```

#### 9. Common Service Management Commands

```bash
sudo systemctl start digital-human     # Start immediately
sudo systemctl stop digital-human      # Stop
sudo systemctl restart digital-human   # Restart
sudo systemctl status digital-human    # Check status
journalctl -u digital-human -f         # View real-time logs
```

> [!NOTE] 
> This translation requires manual review as it involves technical documentation that may benefit from human verification for accuracy.