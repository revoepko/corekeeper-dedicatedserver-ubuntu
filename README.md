# **Core Keeper Dedicated Server Installation and Configuration Guide**
---

**The original Korean text of this repository can be found at the link below.**

*https://ssoobaakbaaa.tistory.com/5*

---

## **1\. Preparing the System**

### 1.1 Environment Information

This guide is tailored for `Ubuntu 24.04.1 LTS`.

### 1.2 Creating a User Account

Create a dedicated user account to manage `steamCMD` and the `Core Keeper Dedicated Server`.

```bash
sudo adduser corekeeper
```

#### 1.3 Setting Up Package Repositories

Install `steamCMD` through the package repository.

```bash
sudo add-apt-repository multiverse; sudo dpkg --add-architecture i386; sudo apt update
sudo apt install steamcmd
```

---

### **2\. Installing Core Keeper Dedicated Server**

#### 2.1 Installing with steamCMD

Use `steamCMD` to install both the `Core Keeper Dedicated Server` and `Steamworks SDK Redist`.

##### 2.1.1 Installing Core Keeper Dedicated Server

```bash
steamcmd +force_install_dir /home/corekeeper/server +login anonymous +app_update 1963720 validate +quit
```

##### 2.1.2 Installing Steamworks SDK Redist
```bash
steamcmd +force_install_dir /home/corekeeper/sdk +login anonymous +app_update 1007 validate +quit
```

---

### **3\. Setting Up a Systemd Service**

#### 3.1 Modifying the Launch Script

Edit the script file located at `/home/corekeeper/server/_launch.sh` as follows:

```bash
sed -i 's|Steamworks SDK Redist/linux64|sdk|g' /home/corekeeper/server/_launch.sh
```

#### 3.2 Creating a Systemd Service File

Create a service file to manage the `Core Keeper Dedicated Server` with `systemd`.

##### 3.2.1 Service File Creation

Add the following content to /etc/systemd/system/corekeeper.service:

```bash
sudo vi /etc/systemd/system/corekeeper.service
```

Service File Content:

```bash
[Unit]
Description=Core Keeper Dedicated Server
After=network.target

[Service]
Type=simple
User=corekeeper
WorkingDirectory=/home/corekeeper/server
ExecStartPre=/bin/sleep 5
ExecStart=/home/corekeeper/server/_launch.sh
KillMode=process
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
```

##### 3.2.2 Reloading and Enabling the Service

Reload `systemd`, start the service, and enable it to run on boot:

```bash
sudo systemctl daemon-reload
sudo systemctl start corekeeper.service
sudo systemctl enable corekeeper.service
```

---

### **4\. Importing an Existing World**

#### 4.1 World File Paths

Copy an existing world to the server directory.

##### 4.1.1 Linux Server World Path
```
/home/corekeeper/.config/unity3d/Pugstorm/Core Keeper/DedicatedServer/worlds
```

##### 4.1.2 Windows Single-Player World Path
```
C:\Users\{User}\AppData\LocalLow\Pugstorm\Core Keeper\Steam\253642204\worlds
```

##### 4.1.3 Windows Dedicated Server World Path
```
C:\Users\{User}\AppData\LocalLow\Pugstorm\Core Keeper\DedicatedServer\worlds
```

> Note:
> After a test run, delete the newly created files and replace them with your existing world files.
---

### **5\. Setting Up System Monitoring Tools**

#### 5.1 Installing Additional Packages

Install `screen` and `glances` for system monitoring:

```bash
sudo apt install -y screen glances
```

#### 5.2 Running Glances for Monitoring
Run `glances` in the background using `screen`.

##### 5.2.1 Starting Glances in the Background
```bash
screen -dmS corekeeper bash -c "sudo glances"
```

##### 5.2.2 Connecting to the Screen Session
```bash
screen -r corekeeper
```

##### 5.2.3 Detaching the Screen Session (Keep Running)
```
Press Ctrl + A, D
```

---

### **6\. Overall Execution Flow**
1. `steamCMD` and the `Core Keeper Dedicated Server` run under the `corekeeper` user account.
2. The `corekeeper.service` is managed by `systemd`, starting automatically and restarting if the server crashes.
3. Use `screen` and `glances` for real-time system monitoring.
