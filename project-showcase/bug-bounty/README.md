# Bug Bounty - Penetration Testing :beetle:

## Overview :globe_with_meridians:

**Bug Bounty** project demonstrates a full penetration test against the Damn Vulnerable Web Application (DVWA) running on Metasploitable 2. The test includes reconnaissance, brute force login, SQL injection, file upload vulnerability, reverse shell access, privilege escalation, and a chained exploit combining multiple vulnerabilities. It is conducted in an isolated VirtualBox lab using Kali Linux.

---

## Table of Contents :book:
- [Overview](#overview-globe_with_meridians)
- [Technologies Used](#technologies-used-wrench)
- [Setup Instructions](#setup-instructions-computer)
- [Usage Guide](#usage-guide-running)
- [Bonus Features](#bonus-features-gift)
- [Key Concepts](#key-concepts-key)
- [Ethical Disclaimer](#ethical-disclaimer-bulb)
- [Author](#author-construction_worker)

---

## Technologies Used :wrench:

- VirtualBox VMs: 
    - Metasploitable 2 
    - Kali Linux 
- Pentest tools in Kali VM: 
    - Reconnaissance: `nmap`, `netstat`, `tcpdump`, `Wireshark`, `Zenmap`
    - Brute Force Attack: `hydra`
    - SQL Injection Attack: `sqlmap`, `searchsploit`
    - File Upload Attack: `php-reverse-shell.php`, `netcat`, `nmap`
- Python Virtual Environment
- Draw.io for Network Map Creation
- Visual Studio Code
- WSL (Ubuntu)
- Version Control System (Git)
- Repository (Gitea)
- VCS Client (Bash)

---

## Setup Instructions :computer:

### Requirements

- VirtualBox installed
- Kali Linux VM (attacker)
- Metasploitable 2 VM (target)
- Bridged network configuration for both VMs
- Shared folder (optional) between Kali and host for saving reports
- Python 3.9 or higher (for bonus brute force script)


### Setting Up the Environment

#### 1. Clone the Repository and Navigate to It
```
git clone https://github.com/HennaVenho/bug-bounty.git
cd bug-bounty
```

#### 2. Import and Run both VMs in VirtualBox

- If you need to install VirtualBox, you can download it from <a href="https://www.virtualbox.org">https://www.virtualbox.org</a>

    **A: Metasploitable 2 in VirtualBox**

    1. Download Metasploitable 2 

        - from <a href="https://sourceforge.net/projects/metasploitable/">https://sourceforge.net/projects/metasploitable/</a>

    2. Import into VirtualBox

        - Extract the Metasploitable 2 ZIP to a safe folder (e.g. `C:/VMs/Metasploitable2/`)

        - In VirtualBox: 
            - Click New 
                - Name it e.g. `Metasploitable 2`
                - Choose: 
                    - Type: Linux 
                    - Subtype and version: Ubuntu (32-bit)
            - Set Memory Size: 
                - Set RAM to 1024 MB or 2048 MB (Metasploitable doesn't need much)
            - In the "Hard Disk" screen:
                - Select **Use an existing virtual hard disk file**
                - Click the 📂 icon and browse to the `Metasploitable.vmdk` file you just extracted
                - Select it, then click **Create**

    3. Network Configuration (required!)

        - Go to VirtualBox Settings → Network
        - Set Adapter 1 to:
            - ✅ Enable Network Adapter
            - Attached to: Bridged Adapter
            - Name: your active internet adapter (e.g. Wi-Fi)

    4. Start Metasploitable 2

        - Login with default credentials:
            ```console
            Username: msfadmin
            Password: msfadmin
            ```

        - Find the IP address:
            ```console
            ifconfig
            ```

        - Look for the `inet` line under `eth0`, e.g.:
            ```console 
            inet 192.168.0.11
            ```


    **B: Kali Linux in VirtualBox**

    1. Download Kali Linux

        - When VirtualBox is already installed, you can download Kali Linux VirtualBox image from: <a href="https://www.kali.org/get-kali/#kali-virtual-machines">https://www.kali.org/get-kali/#kali-virtual-machines</a>

    2. Extract the `.7z` File

        - Use a tool like 7-Zip or WinRAR to extract the `.7z` archive
        - After extracting, you should see something like: 
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
                - Name: Kali Linux
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

    4. Adjust Kali Linux VM Settings

        - Select your Kali VM → Click **Settings**

            A. Display > Screen
            - Video Memory: 128 MB
            - Enable 3D Acceleration: ✅

            B. Network
            - Set Adapter 1 to:
                - ✅ Enable Network Adapter
                - Attached to: Bridged Adapter
                (This must match what Metasploitable is using)
                - Name: your active internet adapter (e.g. Wi-Fi)
            - Click **OK**

    5. Start Kali Linux

        - Click **Start** on the VM.

        - Login with default credentials:
            ```console
            Username: kali
            Password: kali
            ```

        - On the Kali desktop, click the **Terminal** icon (looks like a black screen), and run the following command to check the Kali VM IP address: 
            ```bash
            ip a
            ```
        - Look for the `inet` line under `eth0`, e.g.:
            ```console 
            inet 192.168.0.18
            ```

#### 3. Confirm that Kali VM and Metasploitable VM are on the same subnet and can ping each other

- Make sure Kali Linux VM IP is in the same subnet as Metasploitable 2 (e.g. both `192.168.0.x`).

    - Confirm connectivity between Kali Linux VM and Metasploitable 2 VM. From Kali terminal:
        ```bash
        ping <metasploitable_ip>
        ```
    - Optionally you can test DVWA:
        ```bash
        firefox http://<metasploitable_ip>/dvwa/
        ```

#### 4. Start the Apache service on Metasploitable 2

- DVWA runs on a built-in Apache server, so ensure that the Apache web server is running.
    - In the Metasploitable 2 terminal, run:
        ```bash
        sudo /etc/init.d/apache2 start
        ```

#### 5. Access DVWA from the browser in Kali

- From Kali Linux browser, go to:
    ```
    http://<metasploitable_ip>/dvwa/
    ```
    - If it works, you should see the DVWA login page.

#### 6. Login to DVWA using the default credentials:
```
Username: admin
Password: password
```
- If login works, you'll land on the main DVWA dashboard.

#### 7. Set DVWA security level to Medium

- From the menu in DVWA:

    1. Click "DVWA Security"
    2. From the "Script Security" dropdown:
        - Set Security Level to "medium"
    3. Click "Submit"

- You're now working with realistic but still vulnerable settings — perfect for pentesting practice.

#### 8. Install Python and Pip, if needed

- The bonus brute force script is written in Python, so if you want to run it, please ensure you have Python installed, or download it from <a href="https://www.python.org/downloads/">python.org</a>, OR for Linux, check your distro's package manager for the following packages:

    - python3
    - python3-pip
    - python3-venv (optional, but strongly recommended, as it will allow you to create a virtual environment for the installed dependencies, and thus avoid any version incompatibilities with other projects)

- For Ubuntu-based distros you can use the following command:
    ```console
    sudo apt install python3 python3-pip python3-venv
    ```

- Create a Virtual Environment:
    ```console
    python3 -m venv venv
    ```

- Activate the Virtual Environment:
    - WSL/Linux/macOS:
        ```console
        source venv/bin/activate
        ```
    - Windows (PowerShell):
        ```console
        venv\Scripts\Activate
        ```

- Install Dependencies (required):
    ```console
    pip install -r scripts/requirements.txt
    ```

- Optional: Select the Python Interpreter in VS Code:
    - Press `Ctrl+Shift+P`, search `Python: Select Interpreter`
    - Choose: `venv/bin/python` (or `venv\Scripts\python.exe`)

- After the setup is done, please check [Usage Guide](#usage-guide-running) for how to run the Python brute force script.   


### Safety Tips While Using Bridged Adapters for the VMs

1. Be Aware of Other Devices on the Network
    - Only target the IP of Metasploitable

2. Disable Services You Don't Use
    - Don't leave reverse shells or listeners running indefinitely (`nc -lvnp`)
    - Shutdown Metasploitable when you're not using it

3. Temporarily Disabling Your OS Firewall
    - Only disable firewall if necessary (e.g. to test ping or reverse shell from Metasploitable to Kali), and always turn it back on afterward

4. Clean up Exploits After Testing
    - If you upload a reverse shell to DVWA, delete it after the test so it doesn't stay open by accident.


### (Optional) Create a Shared Folder Between Kali and Host for Easier Results Sharing 

- This creates a live folder that both Kali and your host OS (e.g. Windows/WSL) can access.

1. Shut down your Kali VM

2. In VirtualBox, go to Kali VM > Settings > Shared Folders

3. Click the folder icon → Add a shared folder
    - Folder Path: Choose a real folder on your Host (e.g. Windows) system (e.g. `C:\Users\YourName\KaliShare`)
    - Folder Name: e.g. `KaliShare` (this will be used in Kali)
    - Check: ✅ Auto-mount 

4. Start Kali again

- In Kali, mount it:
    - If not auto-mounted, run:
        ```bash
        sudo mkdir /mnt/KaliShare
        sudo mount -t vboxsf KaliShare /mnt/KaliShare
        ```
    - A. Now you can copy a file from Kali to your OS:
        ```bash
        cp <filename> /mnt/KaliShare/
        ```
        - You'll find `<filename>` in your OS folder
    - B. You can also copy a file from your OS to Kali by placing it in the shared folder and running in Kali: 
        ```bash
        cp /mnt/KaliShare/<filename> <file-name-in-Kali>
        ```

---

## Usage Guide :running:

- All tool outputs, screenshots, and custom scripts are provided in the `/reports`, `/screenshots`, and `/scripts` folders.

- The Python brute force script `/scripts/brute_force_dvwa.py` performs a brute force attack on DVWA's login form using local username and password lists. 
    - To run the Python script, make sure the following wordlist files exist in the `scripts/` directory:
        - `users.txt` — A list of usernames, one per line (e.g. `gordonb`, etc.)
        - `passwords.txt` — A list of passwords, one per line (e.g. `abc123`, etc.)

    ⚠️ These files are excluded from the repository to follow best practices. Please create them yourself before running the script.

    - Run the script in the Kali environment, or directly from your OS like this:
        ```bash
        cd scripts
        python3 brute_force_dvwa.py <metasploitable_ip>
        ```

- [Pentest_Report.md](./Pentest_Report.md) details the penetration tests run, steps to reproduce the tests, and an analysis of the results, as well as recommendations for mitigation. 

---

## Bonus Features :gift:

- Reconnaissance is done using several tools to confirm the network mapping: `nmap`, `netstat`, `tcpdump`, `Wireshark`, `Zenmap`
- A Python script for brute forcing, which performs the same function as the Hydra attack
- SQL Injection done using both `sqlmap` and manual browser-based injection
- File Upload Attack is accomplished with prebuilt PHP shell `php-reverse-shell.php`, but several tests were made creating my own PHP test file
- Root access gained with vulnerable binary: `/usr/bin/nmap`, but other methods are also considered, and instructions are given for creating a new user with root privileges by editing `/etc/passwd` 
- Chained exploit using Brute Force followed by SQLi
    - Brute Force gives access
    - SQL Injection leverages that access to compromise the backend
    - Together, they lead to data exposure and authenticated attack surfaces you couldn't reach otherwise.

---

## Key Concepts :key:

<details id="what-penetration-testing-is-and-how-it-is-proactively-used-in-cybersecurity">
<summary>What Penetration Testing Is and How It Is Proactively Used in Cybersecurity</summary>

- Penetration testing (or "pentesting," or ethical hacking) is a controlled simulation of a cyberattack on a system, web app, or network. The goal is to identify security vulnerabilities before attackers can exploit them.
- It is used proactively to strengthen an organization's defenses by revealing weak spots in authentication, access controls, software, or configurations.
- Think of it like hiring ethical hackers to break in legally, so real hackers don't get the chance to do it first.

</details>

<details id="ethical-guidelines-of-penetration-testing">
<summary>Ethical Guidelines of Penetration Testing</summary>

- Pentesting must always be performed with proper authorization and within a legally approved scope.
- The tester must avoid damaging systems, stealing data, or exploiting vulnerabilities beyond the agreed-upon scope.
- Any discovered vulnerabilities must be reported responsibly.
- In this project, the pentest is conducted in a deliberately insecure environment (DVWA on Metasploitable 2), which is designed for ethical practice and learning.

</details>

<details id="reasons-behind-choosing-attacking-platform">
<summary>Reasons Behind Choosing Attacking Platform</summary>

- I chose Kali Linux because it comes pre-installed with many industry-standard pentesting tools like `nmap`, `hydra`, `sqlmap`, and `netcat`.
- Full Kali virtual machine is used in VirtualBox, instead of WSL, to enable all Kali tools and full networking capabilities, which was necessary for tools like `netcat` and `tcpdump` (e.g. raw sockets and network interfaces are not so well supported in Kali WSL)

</details>

<details id="network-map-and-potential-weaknesses">
<summary>Network Map and Potential Weaknesses</summary>

- A network map is a visual representation of devices and services running in a given network.
- In this project, it helps identify open ports, running services, and the path to the vulnerable DVWA target.
- Weaknesses include outdated services, exposed ports (like SSH, FTP, MySQL), and services with known vulnerabilities.
- Wireshark Network Traffic Capture captured HTTP POST requests to `/dvwa/login.php`, and the capture clearly shows username/password in plaintext.
- Although the initial `tcpdump` capture did not reveal HTTP credentials due to timing, it demonstrated ARP (Address Resolution Protocol) traffic and network chatter between the VMs. This was documented as a failed attempt, which is also valuable in pentesting documentation.

</details>

<details id="principles-and-methods-of-brute-forcing">
<summary>Principles and Methods of Brute Forcing</summary>

- Brute force attacks try many combinations of usernames and passwords until they find valid credentials.
- Tools like `hydra` can automate this using a wordlist of possible usernames and passwords.
- It's most effective when there's no lockout mechanism or rate limiting on login attempts.
- Kali also comes built-in with ready wordlists, but using long wordlists provides an extremely long processing time. 
- Brute force attacks are not always efficient. In this test, I confirmed that using publicly available DVWA credentials was faster and more practical than running large brute force wordlists.

</details>

<details id="how-the-brute-forcing-tool-identifies-valid-usernames-and-passwords">
<summary>How the Brute Forcing Tool Identifies Valid Usernames and Passwords</summary>

- The tool I used for brute forcing, Hydra works as follows: 
    1. Sends login requests using combinations of usernames and passwords, and analyzes server responses after each login attempt.
    2. Checks the response body or status code from the server
    3. Identifies a change in behaviour that signals a successful login:
        - If the page contains "Login failed", that attempt failed
        - If the response changes (e.g. a redirect,  a different message, or no "Login failed" text), it signals valid credentials
- If credentials are correct, Hydra shows:
    ```plaintext
    [80][http-post-form] host: 192.168.0.11   login: pablo   password: letmein
    ```
- I also wrote a Python script that performs a brute force login by checking for the same "Login failed" message, demonstrating how this detection logic can be replicated in code.

</details>

<details id="principles-and-methods-of-sql-injection">
<summary>Principles and Methods of SQL Injection</summary>

- SQL injection (SQLi) occurs when user input is directly included in a database query without sanitization.
- An attacker can manipulate the SQL query to extract or modify data by injecting code into input fields.
- Manual techniques (like `' OR 1=1--`) or automated tools like `sqlmap` can be used to exploit this.

</details>

<details id="what-further-actions-can-be-taken-once-the-database-schemas-have-been-obtained">
<summary>What Further Actions Can Be Taken Once the Database Schemas Have Been Obtained</summary>

- After discovering schemas (i.e. databases and tables), attackers can:
    - Extract sensitive data (users, passwords, config settings)
    - Modify data (e.g. change user roles)
    - Identify further vulnerabilities or weak configurations
    - Chain the SQLi with other attacks (like account takeover or privilege escalation)

</details>

<details id="principles-and-methods-of-file-upload-attacks">
<summary>Principles and Methods of File Upload Attacks</summary>

- File upload attacks happen when an application allows users to upload files without proper validation.
- If a malicious file (e.g. a PHP reverse shell) is uploaded and executed, the attacker can gain remote access to the server.
- Exploits typically involve bypassing file extension checks or MIME type validation.
- In this project, file upload bypass was tested by uploading `.php.jpg` files and renaming them after upload. This revealed that filename-based checks are not secure when file permissions or post-processing is insufficient.

</details>

<details id="how-the-reverse-shell-works">
<summary>How the Reverse Shell Works</summary>

- A reverse shell is a shell session initiated from the target back to the attacker's machine.
- The target machine connects to a listener (e.g. `nc -lvnp 4444`) on the attacker's machine.
- Once connected, the attacker gets a terminal on the target system, often as the user running the vulnerable app.
- I used a modified `php-reverse-shell.php` to open a connection back to Kali via port 4444. After executing the payload, a listener using `nc -lvnp 4444` received a shell as user `www-data`.

</details>

<details id="various-privilege-escalation-methods">
<summary>Various Privilege Escalation Methods</summary>

- Privilege escalation is the process of gaining higher-level access, such as root.
- Common methods include:
    - Exploiting misconfigured services (e.g. cron jobs, writable `/etc/passwd`)
    - Kernel exploits
    - Using leaked passwords or reused credentials
    - SUID binaries and scripts
- I used the interactive mode of a SUID binary (`nmap`) to escalate from `www-data` to `root`.

- Common Linux Privilege Escalation Methods:

| Method                     | Description                                                                                                  | Example                                                    |
| -------------------------- | ------------------------------------------------------------------------------------------------------------ | ---------------------------------------------------------- |
| **SUID binaries**          | Programs with the SUID bit run as their owner (e.g. root). Exploiting an insecure one can give root access.  | `/usr/bin/nmap` with interactive shell                     |
| **Cron jobs**              | Misconfigured scheduled tasks may run scripts you can write to.                                              | If `/etc/cron.d/job` runs `script.sh` in a writable folder |
| **Kernel exploits**        | Older kernels may be vulnerable to known exploits.                                                           | Exploiting CVE with public PoC like Dirty COW              |
| **Writable config files**  | Files like `/etc/passwd` or `/etc/sudoers` may be misconfigured or writable.                                 | Add a new root user or enable passwordless sudo            |
| **Password reuse**         | Cracked credentials (e.g. from SQLi) reused for SSH or `su`.                                                 | `su gordonb`, then try their password from earlier         |
| **Misconfigured sudo**     | If user can run any command with `sudo` without password.                                                    | `sudo nmap` or `sudo less` → root                          |
| **PATH hijacking**         | If root runs a script that calls `cp`, and attacker can change PATH to use a malicious `cp`.                 | Used in cron or init scripts                               |
| **World-writable scripts** | Scripts run by root but writable by normal users.                                                            | `/etc/init.d/myscript` owned by `www-data`                 |


</details>

<details id="recommendations-on-how-found-vulnerabilities-can-be-mitigated">
<summary>Recommendations on How Found Vulnerabilities Can Be Mitigated</summary>

- Enforce strong password policies and lockouts to prevent brute force attacks.
- Sanitize and parameterize all SQL queries to prevent injection.
- Validate and restrict file uploads by checking file types, extensions, and applying strict permissions.
- Regularly patch systems and services to eliminate known vulnerabilities.
- Disable or limit the use of SUID binaries like `nmap` unless explicitly needed. 
- Implement server-side validation for file uploads and hash passwords using stronger algorithms like bcrypt.

</details>

<details id="how-the-recommendations-improve-the-security-of-the-application">
<summary>How the Recommendations Improve the Security of the Application</summary>

- These recommendations reduce the risk of unauthorized access, data leaks, and system compromise.
- Strong authentication and input validation directly address the exploited vulnerabilities.
- Keeping systems updated and hardened helps defend against both known and emerging threats.

</details>

<details id="chained-exploits">
<summary>Chained Exploits</summary>

- A chained exploit means combining two or more vulnerabilities, where the first one enables the second. Think of it as:
    - Initial foothold → deeper control → full compromise
- Instead of just showing each vulnerability in isolation, you exploit them in a sequence.

- Chained Exploit used: Brute Force → SQL Injection
    - I first brute-forced a login using `hydra` and obtained valid credentials for a standard user (`gordonb:abc123`).
    - Once logged into DVWA, I was able to access the vulnerable SQL injection page.
    - By using `sqlmap` with the authenticated session cookie, I successfully dumped database contents, including user credentials and password hashes.
- This demonstrates how weak login protections can be chained with SQL vulnerabilities to compromise backend data.

</details>

---

## Ethical Disclaimer :bulb:

All testing was conducted in a controlled environment with consent. No real systems were harmed or accessed.

---

## Author :construction_worker:
Henna Venho