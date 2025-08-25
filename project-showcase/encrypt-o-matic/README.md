# Encrypt-O-Matic - A Windows/Linux Executable Encryptor Tool 🛡️

## Overview :globe_with_meridians:

**Encrypt-O-Matic** is a cross-platform command-line tool for encrypting Windows or Linux executables, rendering them unusable until decrypted. It supports three encryption algorithms — **AES**, **ChaCha20**, and **Twofish** — and allows users to encrypt either a single file, or all executable files within a directory (recursive mode). 

The tool supports optional compression using **Zstandard**, customizable artificial delay using a hash-based operation, and time-delayed decryption. Decryption can occur either:
- immediately after entering the correct password, or
- automatically after a configurable timer expires (via a pre-derived key).

A minimal decryptor stub is bundled with the encrypted file and is used at runtime to prompt for decryption. After successful decryption, the original encrypted file is replaced, and the executable is launched automatically. The tool includes platform-specific logic and supports both Windows `.exe` files and Linux executables.

---

## Table of Contents :book:
- [Overview](#overview-globe_with_meridians)
- [Technologies Used](#technologies-used-wrench)
- [Setup Instructions](#setup-instructions-computer)
- [Usage Guide](#usage-guide-running)
- [Bonus Features](#bonus-features-gift)
- [Key Concepts](#key-concepts-key)
- [Notes and Warnings](#notes-and-warnings-warning)
- [Author](#author-construction_worker)

---

## Technologies Used :wrench:

- **Programming Language**: Python 3.12.3
    - `venv` for isolated environments
    - Cross-platform scripting and file I/O
    - Easy integration with modern encryption libraries

- **Encryption Libraries**:
    - `cryptography`: for AES-GCM and ChaCha20-Poly1305 (authenticated encryption)
    - `twofish`: manually included and modified version for Twofish-CBC (legacy block cipher)

- **Compression**:
    - `zstandard`: for optional compression of embedded payloads before encryption

- **CLI and Interactive Prompt**:
    - `argparse`: for structured argument parsing
    - `prompt_toolkit`: for secure, interactive password prompts in terminal

- **Executable Packaging**:
    - `pyinstaller`: for converting scripts into standalone Windows/Linux executables

- **Build Environment**:
    - **Visual Studio Code** + **WSL Ubuntu**: used for core development
    - **Windows 10 Virtual Machine** via **VirtualBox**: used for testing, building, and validating `.exe` behaviour

- **Version Control**:
    - Git + Gitea

---

## Setup Instructions :computer:

This section helps you set up a development and testing environment for Encrypt-O-Matic.

The project is cross-platform, but its core functionality is focused on **Windows executables** (`.exe`). For this reason, testing is best done in a **Windows environment**.

---

### Requirements

To build and run the program, you will need:

- A working **Windows environment** (real PC or virtual machine)
- **Python 3.10+**
- **Git** (to clone the repository)
- **Optional but recommended**: Virtual environment (`venv`)
- ~50 GB of free disk space

I strongly recommend setting up a **Windows Virtual Machine (VM)**. See instructions below.

---

### Step-by-Step Setup in VirtualBox

This guide walks you through installing a Windows 10 VM using Oracle VirtualBox and configuring it for development.

---

#### Prerequisites

1. **Oracle VirtualBox**  
   Download and install from: https://www.virtualbox.org/

2. **Windows 10 ISO file**  
   Download from the official site:  
   https://www.microsoft.com/en-us/software-download/windows10ISO

3. At least **50 GB of free disk space**

---

#### 1. Create a New VM

- Open VirtualBox -> Click **New**
- Name, e.g.: `encrypt-o-matic`
- Type: `Microsoft Windows`
- Version: `Windows 10 (64-bit)`

Configure hardware:

- On **Hardware** tab: 
    - **Base Memory**: 4096 MB or more
- On **Hard Disk** tab: 
    - Select: **Create Virtual Hard Disk Now** 
    - Size: **50 GB** (recommended), dynamically allocated
    - Type: **VDI (VirtualBox Disk Image)**
- Click **Finish**

---

#### 2. Mount the ISO File

- Select your new VM, go to **Settings** -> **Storage**
- Under "Controller: SATA" (or "Controller: IDE", depending on what version of VirtualBox you are working on):
    - Click on the "Empty" CD icon
- On the right-hand side, look for the CD icon drop-down menu (next to "Optical Drive" label)
- Click it → Choose **"Choose a Disk File…"**
- Browse to and select your **Windows 10 ISO** file
- After selecting the ISO, "Empty" should change to something like:
    ```
    Win10_22H2_English_x64v1.iso
    ```
- Click **OK** to save.

---

#### 3. Enable Networking

- Go to **Settings** -> **Network** -> Adapter 1:
  - ✅ Enable Network Adapter
  - Attached to: **Bridged Adapter** (or **NAT** if that doesn't work)
  - Name: your physical network (Wi-Fi or Ethernet)
  - Click **OK**

---

#### 4. Install Windows

Start the VM and follow the installer:

- Language: English  
- Time/keyboard: appropriate
- Click **Next** > **Install now**

- On **Activation Screen**:
    - Click "**I don't have a product key**"

- On **Select Operating System**:
    - Choose: **Windows 10 Pro** (recommended for developer features)

- Accept license -> Next

- On **Installation Type**
    - Choose: **Custom: Install Windows only (advanced)**

- Install Windows on the 50 GB virtual disk (may take approx. 5–15 minutes)

---

#### 5. Set Up a Local Account (Offline Mode)

To avoid linking a Microsoft account:

**If no internet:**
- The installer will let you choose "**Offline Account**" or "**Limited setup**"

**If internet is connected:**
- Press `Shift + F10` to open a command prompt
    - Type:
    ```bash
    oobe\bypassnro
    ```
    - Press Enter -> VM reboots -> Choose **"I don't have internet"**
- OR: Power off the VM, and then in VirtualBox:
    - Settings -> Network -> **Uncheck "Enable Network Adapter"** -> reboot VM 
    - After reboot, it should offer **"I don't have internet"**

Then:
- Choose **"Continue with limited setup"**, or click **"Limited experience"**, if available
- Create a **local username**, and (optionally) a **password** 
- Decline unnecessary services

---

#### 6. First Boot Setup

After Windows finishes installing:
- Re-enable the network (if you disabled it earlier) (see step 3.)
- Install Guest Additions (optional: for copy/paste + screen resizing):
    - VM menu -> Devices -> Insert Guest Additions CD Image
    - Open CD inside VM and run the installer
- Install software, as needed:
    - Python
    - Git
    - VS Code (optional)

---

### Python Setup 

Python is required to build and run the project.

---

#### Python for Windows (Inside the VM)

1. Download from: https://www.python.org/downloads/windows/
2. Choose the latest stable version (e.g. `python-3.13.x-amd64.exe`)
3. Run the installer:
   - ✅ Check **Add python.exe to PATH**
   - Click **Customize installation** -> keep all defaults
   - ✅ Check **Install Python for all users**
   - Click **Install**

4. Open Command Prompt and check:
   ```bash
   python --version
   ```
    You should see: `Python 3.13.x`

---

#### Python for Linux (WSL/Ubuntu)

```bash
sudo apt update
sudo apt install python3 python3-pip python3-venv
```

---

### Optional: Install Development Tools

---

#### Git for Windows

1. Download: https://git-scm.com/
2. Install with recommended defaults:
    - ✅ Use VS Code or Notepad as editor
    - ✅ Enable Git from command line
    - ✅ Keep default line endings (Checkout Windows-style, commit Unix-style)
    - ✅ Use MinTTY for terminal
    - ✅ Enable file system caching

3. Confirm installation:
    ```bash
    git --version
    ```
    You should see something like: `git version 2.50.1.windows.1`

---

#### Visual Studio Code (Optional)

1. Download: https://code.visualstudio.com/
2. During installation:
    - ✅ Add to PATH
    - ✅ Add "Open with Code" to context menu

---

#### Shared Folder with Your OS (Optional)

- To easily sync files:
- In VirtualBox -> VM Settings -> Shared Folders:
- Add your project folder (~/encrypt-o-matic/)
- Check: "Auto-mount" + "Make permanent"
- Inside Windows, it'll appear as `Z:\` or similar.

---

### Clone the Repo and Build the Project

---

#### 1. Clone the Repository

In Windows Command Prompt (or Visual Studio Code, if you please), clone the repo: 
```bash
git clone https://github.com/HennaVenho/encrypt-o-matic.git
cd encrypt-o-matic
```
---

#### 2. Create a Virtual Environment (recommended)

- Windows (CMD): 
    ```bash
    python -m venv venv
    ``` 

- Linux/macOS: 
    ```bash
    python3 -m venv venv
    ``` 

> ❗ If you're unsure which one to use, run `python --version` and `python3 --version` to see which one is available

---

#### 3. Activate the Virtual Environment

- Windows:
    ```bash
    venv\Scripts\activate
    ```

- Linux/macOS:
    ```bash
    source venv/bin/activate
    ```
---

#### 4. Install Required Python Libraries

```bash
pip install -r requirements.txt
```
---

#### 5. Build the Executables

- Windows:
    ```bash
    build_windows.bat
    ```

- Linux (set executable permission with `chmod +x <file>` if it isn't set):
    ```bash
    chmod +x build_linux.sh
    ./build_linux.sh
    ```

The resulting file (`encrypt-o-matic.exe` or `encrypt-o-matic`) will appear in your current directory.

---

### Exiting the Environment

When you're done using the tool:

- To **exit the Python virtual environment**, run:
    ```bash
    deactivate
    ```

- To **shut down your Windows VM** (if using VirtualBox):
    1. Inside the VM, click **Start** -> **Power** -> **Shut down**.
    2. Alternatively, in VirtualBox: right-click the VM -> **Stop** -> **ACPI Shutdown**.

    - This ensures that your environment is saved cleanly and doesn't consume background resources.

- To **completely remove the VM from your system**:
    1. In VirtualBox Manager, right-click the VM -> **Remove**
    2. Click "**Delete all files**" when prompted
        - ⚠️ This will permanently delete the VM and its virtual hard disk (VHD/VDI)
    3. Optionally delete any leftover folders in:
        - `~/VirtualBox VMs/`
        - and any manually downloaded ISO files
        

After the setup is done, please check [Usage Guide](#usage-guide) for instructions on how to run the program.   

---

## Usage Guide :running:

After building the program, you'll find the main executable in your project root directory:

- On Windows, it's:
    ```bash
    encrypt-o-matic.exe
    ```

- On Linux, as Linux does not conventionally use `.exe`, it's:
    ```bash
    ./encrypt-o-matic
    ```

The usage examples below assume you're on **Windows**. If you're on Linux, remember to prepend `./` to the command (e.g. `./encrypt-o-matic` instead of `encrypt-o-matic.exe`) and use forward slashes (`/`) instead of backslashes (`\`) in file paths.

---

### Show Help / Usage Instructions

To view available options: 
```console
encrypt-o-matic.exe -h
```

You will see output like this:
```console
usage: encrypt-o-matic.exe [-h] [-m] [-c] [target_app] [encryption_algorithm] [size_manipulation] [custom_variable] [duration]

Encrypt-O-Matic: A Windows/Linux Executable Encryptor Tool

positional arguments:
  target_app            Path to the target Windows/Linux application
  encryption_algorithm  Encryption algorithm to use (AES, ChaCha20, or Twofish)
  size_manipulation     Amount in MB to increase file size
  custom_variable       Custom operation variable (e.g. 100000 or range like 0-100000)
  duration              Encryption duration in minutes

options:
  -h, --help            show this help message and exit
  -m, --multi           Enable multi-file mode (encrypts all executables in directory)
  -c, --compress        Compress the input file with Zstd
```
---

### Creating Safe Test Files

To test the tool safely, create copies of system executables — **don't encrypt original system files**.

**On Windows**

1. Locate e.g. the `calc.exe` file:
    ```bash
    where calc.exe
    ```

2. Copy it into a test folder:
    ```bash
    mkdir testfiles
    copy C:\Windows\System32\calc.exe testfiles\test-calc.exe
    ```

**On Linux**

1. Copy a system utility like `whoami` or `ls` into the test folder:
    ```bash
    mkdir testfiles
    cp /usr/bin/whoami testfiles/fake-whoami
    chmod +x testfiles/fake-whoami
    ```

---

### Example Commands

These examples show how to run the program with various options:

---

#### 1. Minimal AES Encryption (no size change, no timer)

```bash
encrypt-o-matic.exe testfiles\test-calc.exe aes 0 0 0
```
Encrypts `test-calc.exe` using **AES** without increasing file size, without custom operation, and no timer.

---

#### 2. Twofish + Inflate + Timer

```bash
encrypt-o-matic.exe testfiles\test-calc.exe twofish 3 0 1
```
Encrypts with **Twofish**, adds 3 MB of random junk data, no Operation X, and enables a 1-minute timer before automatic decryption.

---

#### 3. ChaCha20 + Inflate + Timer + Custom Operation X + Compression

```bash
encrypt-o-matic.exe -c testfiles\test-calc.exe chacha20 5 100000 2
```
Encrypts using **ChaCha20**, adds 5 MB inflate, runs Operation X with 100000 bytes, sets a 2-minute timer, and compresses the data before encryption.

---

#### 4. Multi-File Encryption (Directory Mode)

```bash
encrypt-o-matic.exe -m -c testfiles aes 0 0 0
```
Recursively encrypts all `.exe` files in the `testfiles` directory using **AES**, with compression, and no extra options.

---

### Decryption Instructions

To decrypt an encrypted file:

```bash
testfiles\test-calc.exe
```
The executable will prompt you for the **master password** used during encryption — or automatically decrypt itself if a timer was set.

⚠️ **Do not delete the encrypted** `.exe` during the decryption process. The program replaces itself with the decrypted version after launching.

---

## Bonus Features :gift:

**Multi-File Mode:**
- Activated using `-m` or `--multi` flag.
- Recursively traverses the given directory and **encrypts all valid executable files**:
    - On **Windows**, targets `.exe` files.
    - On **Linux**, targets files with the **executable permission bit** set.
- Each file is encrypted **independently** using its own **random salt and IV/nonce**, ensuring better cryptographic separation and security per file.

**Per-File Key Separation:**
- Even in multi-file mode, **each file gets its own unique encryption key** derived from:
    - The user-provided password
    - A random salt (using PBKDF2 key derivation)
- Prevents key reuse and avoids leaking patterns across multiple files.

**Compression Support (Zstandard):**
- Optional compression can be enabled using `-c` or `--compress`.
- Uses **Zstandard**, a high-performance compression algorithm designed for speed and efficiency.
- Compression reduces the size of the encrypted payload and adds entropy, which may make the binary harder to reverse engineer.
- Decompression happens **automatically during decryption** if compression was used.

**Linux Compatibility:**
- While the tool was primarily designed to encrypt **Windows `.exe` files**, it also supports:
    - Executable binaries in **Linux** environments
    - Full **cross-platform encryption and decryption** via shared logic
- Tested on **Ubuntu (via WSL)**, ensuring the tool is not locked to Windows-only workflows.

**Seamless Decryption and Execution:**
- After successful decryption (by password or timer), the decrypted executable is:
    1. Written to disk as a temporary copy
    2. Replaces the original encrypted file cleanly
    3. Automatically launched, restoring user experience
- This functionality is powered by a separate helper binary: `payload_replacer.exe`.

**Custom Operation X: Simulated Processing Delay:**
- A custom CPU-intensive hashing operation (`Operation X`) is used to simulate legitimate application behaviour.
- Runs **twice** during encryption:
    1. After reading the original file (simulates data pre-processing)  
    2. After encrypting and before writing the output (simulates result validation or saving)
- Purpose:
    - Makes the tool behaviour appear **less suspicious** by introducing realistic processing time.
    - Reduces chances of triggering basic behavioural heuristics in real-world scenarios.

---

## Key Concepts :key:

<details id="main-evasion-methods-used-by-malware">
<summary>Main Evasion Methods Used by Malware</summary>

- Malware often uses several techniques to avoid detection by antivirus software and other security tools. These are known as evasion methods. Some of the most common include:

    - **Code Obfuscation**: Makes the code harder to analyze by humans or automated tools. This can include renaming variables to meaningless names, inserting dead code, or using unusual control flows that still behave the same.

    - **Encryption**: The actual malicious payload is encrypted, and only decrypted at runtime. This prevents antivirus engines from detecting known malicious code through static analysis (e.g. signature matching). The decryptor stub is typically simple and harder to detect.

    - **Polymorphism**: Every time the malware is distributed or run, its code changes — even though the functionality stays the same. This helps evade signature-based detection, since the binary will have a different hash or structure each time.

    - **Packing**: Similar to encryption, but often uses compression techniques. The executable is "packed" into a smaller, non-readable form and then unpacked in memory at runtime.

    - **Anti-debugging & Anti-VM techniques**: The malware detects whether it is being analyzed in a debugger or virtual machine. If so, it may change its behaviour, exit early, or hide key functions.

</details>

<details id="signature-based-detection-and-its-limitations">
<summary>Signature-based Detection and Its Limitations</summary>

- **Signature-based detection** is a traditional antivirus technique that identifies malware by looking for specific patterns of bytes (signatures) known to be associated with malicious code.

- These signatures can include:
    - Specific sequences of machine instructions
    - Hashes of known malicious files
    - File metadata patterns or binary structures

- **Limitations of Signature-Based Detection:**
    - **Cannot detect new or unknown malware (zero-day threats)**: If the malware's signature is not already in the database, it will not be detected.
    
    - **Vulnerable to minor changes in code**: Even small modifications to the file (e.g. changing variable names, adding NOPs, reordering code) can evade detection.
    
    - **Easily bypassed by encryption or packing**: Obfuscated or encrypted payloads can hide known signatures until runtime.
    
    - **Struggles against polymorphic and metamorphic malware**: These techniques cause the malware's appearance to change on every execution, invalidating static signatures.
    
    - **Depends on frequent updates**: Signature databases must be continuously updated to remain effective.

</details>

<details id="principles-of-behavioural-analysis-in-malware-detection">
<summary>Principles of Behavioural Analysis in Malware Detection</summary>

- **Behavioural analysis** focuses on detecting malware based on how it *acts* during execution, rather than how it looks on disk.
- It observes the runtime behaviour of programs in real-time or inside a sandboxed environment to identify suspicious or malicious actions.

- **Examples of suspicious behaviours:**
    - Modifying or deleting system files or registry keys
    - Spawning unexpected child processes (e.g. PowerShell, cmd)
    - Connecting to known or unknown **Command and Control (C2)** servers
    - Injecting code into other processes
    - Encrypting many files in quick succession (typical ransomware pattern)

- **Strengths:**
    - Can detect obfuscated or encrypted malware that bypasses signature-based detection
    - More resilient to polymorphic/mutating malware variants

- **Limitations:**
    - May cause **false positives**, flagging legitimate software as malicious
    - Can be **resource-intensive**, especially in sandboxed environments
    - Malware may include **delays, evasions, or sandbox detection logic** to avoid behaving maliciously until it detects it's in a real environment

</details>

<details id="heuristic-analysis-and-its-role-in-malware-detection">
<summary>Heuristic Analysis and Its Role in Malware Detection</summary>

- **Heuristic analysis** is a detection technique that uses rules, patterns, and educated guesses to identify potentially malicious behaviour or file characteristics — even if the exact malware signature is unknown.

- **What it looks for:**
    - Suspicious API calls (e.g. functions commonly used in **process injection**, **keylogging**, or **file wiping**)
    - Abnormal memory usage or control flow patterns
    - Suspicious code constructs (e.g. packed code that decrypts itself in memory)
    - Use of low-level system features in unexpected ways
    - Presence of malformed headers or intentionally broken structures in executable files

- **Advantages:**
    - Can detect **new**, **unknown**, or **modified** malware variants that signature-based methods would miss
    - Provides early warning signs of suspicious code

- **Limitations:**
    - Prone to false positives, especially with custom or experimental software
    - May be evaded by malware that mimics normal software behaviour
    - Doesn't prove a file *is* malware — only that it *looks suspicious*

</details>

<details id="different-encryption-algorithms-and-their-use-cases">
<summary>Different Encryption Algorithms and Their Use Cases</summary>

- In this project, the user can choose between **AES**, **ChaCha20**, and **Twofish** for encrypting executables. These are all symmetric encryption algorithms, meaning the **same key** is used for both encryption and decryption.

- **AES** (Advanced Encryption Standard):
    - **Type**: Symmetric block cipher
    - **Block size**: 128 bits
    - **Key sizes**: 128, 192, or 256 bits
    - **Mode used**: **GCM** (Galois/Counter Mode), which provides both encryption and authentication
    
    - **Strengths:**
        - Very fast and secure
        - Supported natively by most modern CPUs (hardware acceleration)
        - Resistant to known attacks
        - Widely adopted across many industries
    
    - **Use Cases:**
        - File and disk encryption
        - SSL/TLS communications
        - Cloud storage security
        - VPNs and encrypted databases

- **ChaCha20 (+ Poly1305)**: 
    - **Type**: Symmetric stream cipher
    - **Mode**: Authenticated encryption with Poly1305
    - **Design**: Focuses on simplicity and constant-time operations

    - **Strengths:**
        - Extremely fast on systems without hardware **AES support**
        - Resistant to **timing attacks** due to constant-time design
        - Ideal for mobile and embedded environments
    - **Use Cases:**
        - TLS encryption in browsers (e.g. used by Google Chrome)
        - Secure messaging apps (e.g. Signal)
        - VPNs like WireGuard
        - IoT and mobile devices

- **Twofish**:
    - **Type**: Symmetric block cipher
    - **Block size**: 128 bits
    - **Key size**: Up to 256 bits
    - **Mode used**: CBC with PKCS#7 padding (in this project)
    - **Strengths:**
        - Was one of the five AES finalists (lost to Rijndael)
        - Open, flexible, and still considered secure
        - Efficient in hardware and constrained environments
    - **Use Cases:**
        - Rare in modern systems, but useful for:
            - Embedded systems
            - Niche security tools
            - Educational use where alternative ciphers are desired

</details>

<details id="articulate-the-choice-of-language-and-supporting-libraries-used">
<summary>Articulate the Choice of Language and Supporting Libraries Used</summary>

- **Language Choice: Python**
    - I chose **Python** for this project due to its combination of development speed, readability, and the rich ecosystem of mature libraries:
        - I have prior experience with Python, so I could focus on implementing and testing encryption logic.
        - Python makes cross-platform scripting and file manipulation easier, which supports the project's **multi-platform (Windows and Linux)** goals.
        - Python allows for **clean separation of logic**, which helped me organize encryption, payload embedding, decompression, CLI input, and file replacement in modular files.
    - While Python isn't usually the first choice for performance-critical malware evasion tasks, it is ideal for building a **proof-of-concept (PoC) encryption and payload delivery tool** like this.

- **Supporting Libraries Used:**
    - **`cryptography`**  
        - Main cryptographic library used for **AES-GCM** and **ChaCha20-Poly1305** encryption and key derivation (PBKDF2).

    - **`twofish`**  
        - Used for **Twofish-CBC** encryption. 
        - This module is **not supported** by popular crypto libraries anymore, so I manually **modified and embedded a cleaned version** from a PyPI source into the project.

    - **`zstandard`**  
        - Provides efficient compression and decompression using the **Zstandard algorithm**. Optional for encrypted payloads. 

    - **`pyinstaller`**  
        - Converts the Python scripts into **standalone executables** that run on Windows and Linux without requiring Python installed.    

    - **`prompt_toolkit`**  
        - Enables a responsive and interruptible **password prompt** in the decryptor stub, allowing for **interactive use alongside timer-based decryption**.

    - Python built-in libraries: 
        - **`argparse`**: Used for handling **command-line arguments** and validating input (single-file vs. multi-file mode). 
        - **`hashlib`**: Provides **SHA-256 hashing** for verifying payload integrity before decryption. 
        - **`base64`**: Used to **encode the timer-based decryption key** for safe storage in the payload.
        - **`getpass`**: Built-in way to securely **prompt for passwords without echoing** them to the terminal.
        - **`threading`**, **`time`**: Used to run the **decryption timer and password prompt in parallel**. Ensures either timer or correct password can trigger decryption.
        - **`os`**: Used throughout the program for **file paths**, **permission checks**, **process environment**, etc.
        - **`sys`**: Handles **system exit**, argument access, and runtime behaviour (e.g. when built into `.exe` with PyInstaller).
        - **`shutil`**: Used to **copy and replace** files on disk, especially when swapping encrypted executables with decrypted ones. 
        - **`subprocess`**: Launches the decrypted executable after successful decryption. Also used to invoke the `payload_replacer` helper.
        - **`tempfile`**: Used to **extract helper executables** (like `payload_replacer.exe`) into a **temporary system directory** safely.
        - **`struct`**: Packs and unpacks **binary payload structure** (e.g. encryption metadata, timer key, etc.) in a compact, predictable format.
        - **`importlib.util`**: Used for **dynamic module inspection** in the patched `twofish` integration.

</details>

<details id="custom-operation-and-how-it-increases-the-tools-ability-to-stay-undetected">
<summary>Custom Operation and How It Increases the Tool's Ability to Stay Undetected</summary>

- The custom operation implemented is called **Operation X**.
- It is a **deliberate time-wasting function** that repeatedly hashes a random set of bytes using the **SHA-256** algorithm.
- It is run **twice** during encryption:
    - **After reading the input executable**, and
    - **After encryption but before writing the bundled stub to disk**.
- This simulates time-consuming computation like what legitimate programs often do (e.g. loading, processing), thus helping the malware:
    - **Appear less suspicious** in behavioural analysis.
    - **Delay sandbox analysis**, especially in fast timeout-based environments.
    - **Avoid detection heuristics** that might flag immediate I/O or suspicious API calls.
- The delay is adjustable via the `custom_variable` CLI argument:
    - This controls how many random bytes are hashed, affecting total CPU time wasted.
    - For example, `100000` bytes leads to ~1 second delay on a dev machine.

</details>

<details id="detection-or-defeat-by-antivirus-software">
<summary>Detection or Defeat by Antivirus Software</summary>

- Antivirus software can detect or block this tool through a variety of methods:

- **Signature-based Detection**
    - Scans files for known byte patterns, such as:
        - Encrypted payload headers with predictable structure
        - Presence of cryptographic constants or known instruction sequences
        - Imports of specific cryptography or compression libraries (e.g. `zstandard`, `cryptography`)
        - Static strings like `"decryptor_launcher"` or `"payload_replacer"` included in the stub

- **Heuristic Detection**
    - Looks for suspicious characteristics including:
        - Executables that grow significantly in size (size manipulation)
        - Use of password prompts and timers (common in ransomware-like behaviour)
        - Custom hashing loops (Operation X) that increase CPU usage without clear reason
        - Obvious file-writing and file-replacing operations (overwrite + relaunch behaviour)

- **Behavioural Analysis (Sandboxing)**
    - Executes the binary and monitors behaviour at runtime:
        - Writing decrypted executables to disk
        - Deleting/replacing original `.exe` after payload extraction
        - Launching a secondary process (`payload_replacer`) to modify files
        - Delayed execution patterns triggered by timers
        - Self-deletion behaviour after execution (`os.remove(sys.argv[0])` in replacer)

- **How the Tool Could Be Improved to Reduce Detectability**
    - While this project doesn't aim to create undetectable malware, you could reduce AV flagging by:
        - **Randomizing** binary section layout or stub binary structure
        - **Renaming helper tools** and avoiding fixed filenames
        - **Encoding strings or imports dynamically**
        - **Embedding payloads differently** (e.g. inside alternate PE sections)
        - **Using in-memory execution** instead of writing decrypted files to disk
        - **Avoiding static time delays** and replacing them with event-driven logic

</details>

<details id="password-based-key-derivation-function-PBKDF-choice-and-implementation">
<summary>Password-based Key Derivation Function (PBKDF) Choice and Implementation</summary>

- This project uses **PBKDF2 (Password-Based Key Derivation Function 2)** implemented via the `PBKDF2HMAC` class from the `cryptography.hazmat.primitives.kdf.pbkdf2` module.

- The PBKDF2 function strengthens the user-provided password by performing many iterations of hashing and applying a **cryptographic salt**, producing a secure binary key for encryption.

- **Key Implementation Details:**
    - **Algorithm**: HMAC-SHA-256
    - **Iterations**: 200,000
    - **Salt**: 16 random bytes per encryption
    - **Key length**: 32 bytes (256 bits), suitable for AES-256, ChaCha20, and Twofish
    - **Backend**: Uses `default_backend()` for cryptographic operations

- **Why PBKDF2 with High Iterations?**
    - Makes brute-force attacks computationally expensive.
    - Adding 200,000 iterations slows down key generation just enough to deter attackers without harming legitimate performance.
    - Using a **per-file random salt** ensures that even identical passwords generate different encryption keys.

- **Password Usage in This Tool:**
    - The password is prompted securely from the user (no echo).
    - It is **never stored or transmitted**.
    - The derived key is used **in-memory only**, then discarded.

</details>

<details id="importance-of-salting-in-password-hashing">
<summary>Importance of Salting in Password Hashing</summary>

- Salting adds a **random value** (salt) to a password before hashing or deriving a cryptographic key. This is essential for:
    - **Preventing precomputed attacks** such as rainbow table lookups
    - **Ensuring uniqueness**: even if two users or files use the same password, their derived keys or hashes will be different
    - **Defeating mass brute-force attacks** by forcing the attacker to calculate hashes separately for each salt

- **How It's Done in This Project:**
    - Each encryption generates a fresh 16-byte random salt
    - This salt is stored **alongside the encrypted payload**, so decryption can correctly derive the same key
    - The salt is used as input to **PBKDF2-HMAC-SHA256**, making brute-force attacks far more computationally expensive and user-specific

- **Why It's Important:**
    - Without salting, two files encrypted with the same password would generate identical key material, allowing attackers to correlate data, deduce patterns, or reuse cracked passwords across files.

</details>

<details id="ensuring-the-integrity-of-the-encrypted-file">
<summary>Ensuring the integrity of the encrypted file</summary>

- The tool **appends a SHA-256 hash** of the encrypted payload during the encryption process.
- Upon decryption, this hash is re-calculated and compared against the embedded value.
- If the hash does not match, the program halts with an integrity failure.
- This protects against:
    - Tampering during transfer
    - Incomplete payloads
    - Wrong passwords (especially for unauthenticated encryption like Twofish-CBC)
- This solution complements authenticated encryption modes (like AES-GCM and ChaCha20-Poly1305) and fills the gap for non-AEAD modes.

</details>

<details id="increasing-the-file-size-without-corrupting-the-executable">
<summary>Increasing the File Size Without Corrupting the Executable</summary>

- The tool supports optional **file size manipulation** using the `size_manipulation` argument (in megabytes).
- This feature adds a **block of random "junk" bytes** to the encrypted payload before embedding it into the final executable.

- **Purpose:**
    - **Evasion**: Increases resistance to static analysis or signature-based detection that relies on file size.
    - **Obfuscation**: Makes files less predictable and adds entropy to the payload.

- **How it's safely implemented:**
    - The random bytes are:
        - **Appended at the end of the actual ciphertext**
        - **Length is recorded** in the encrypted payload metadata
    - During decryption:
        - The decryptor uses the **exact ciphertext length** (from the payload header) and **ignores the junk data**
        - This ensures **correct decryption and prevents corruption**
    - No changes are made to the executable's internal structure or headers, preserving format compatibility.

- This approach guarantees that:
    - The encrypted file can still be decrypted without errors.
    - The decrypted executable runs as expected.
    - Repeated structure remains valid even if size manipulation is applied.

</details>

<details id="multi-file-mode-efficient-directory-traversal-and-per-file-encryption"> <summary>Multi-File Mode: Efficient Directory Traversal and Per-File Encryption</summary>

- Multi-file mode enables recursive encryption of all executable files in a given directory.
- Implemented using `os.walk()` and per-file filters:
    - `.exe` extension on Windows
    - Executable permission check on Linux
- Each file is:
    - Validated (empty check, double-encryption check)
    - Assigned a unique key
    - Individually processed, with errors logged but not fatal
- Ensures efficiency, reliability, and scalability of encryption across larger file sets.

</details>

<details id="single-key-vs-multiple-keys">
<summary>Single key vs multiple keys</summary>

- **Single Key for All Files:**
    - ✅ *Simplicity*: Only one key to derive and store.
    - ✅ *Performance*: Saves time by avoiding repeated key derivation.
    - ❌ *Security Risk*: If the single key is leaked or brute-forced, **all files** are compromised.
    - ❌ *Detectability*: Repeated use of the same key and structure may produce detectable patterns.

- **Unique Key Per File (Used in this project):**
    - ✅ *Higher Security*: Each file uses a different key derived from the password + unique salt.
        - Even if one encrypted file is decrypted, the others remain secure.
    - ✅ *Better Obfuscation*: Each encrypted file looks different at the binary level.
    - ❌ *Slight Performance Cost*: Slightly slower due to per-file key derivation with PBKDF2.
    - ❌ *Complexity*: More metadata must be stored in each payload.

- **My implementation uses per-file unique keys**. In both single-file and multi-file mode, each file gets its own random 16-byte salt and key, derived from the master password using PBKDF2 with 200,000 iterations.

</details>

<details id="compression-and-decompression-flow"> <summary>Compression and Decompression Flow</summary>

- The tool supports optional **compression before encryption** via the `-c`/`--compress` flag, using the efficient **Zstandard (Zstd)** algorithm.
- If enabled:
    - The input file is compressed before encryption, reducing payload size and improving obfuscation.
    - A **compression flag** is embedded in the payload.
- During decryption:
    - This flag is checked after successful decryption.
    - If set, the payload is automatically **decompressed** using Zstandard.
- This ensures:
    - Safe and **reversible integration**
    - Correct processing order: **decrypt -> decompress -> run**
    - Compatibility with **large or redundant binaries**
    - The final decrypted file is **identical and runnable** just like the original.

</details>

---

## Notes and Warnings :warning:

**Multi-file Mode Warning:**

Be careful when using the `-m`/`--multi` mode!
This feature recursively traverses all directories under the one you provide and encrypts every executable it finds.
If you accidentally point it to a sensitive system directory, you could render your VM or system unstable or unbootable.
Always test on sample files in isolated folders. Avoid running multi-mode in `C:\` or any real application folder.

**Antivirus False Positives:**

Because this tool manipulates executables and includes encryption, password prompts, and stub launchers, some antivirus tools may flag it as suspicious or require a scan before running.
The `.exe` builds were generated on a clean, isolated Windows VM, and contain no malicious code — this is a known side effect of working with encrypted binary payloads.

**Decryption Behaviour:**

The tool replaces the encrypted file with the decrypted version once a correct password or timer is triggered. If you plan to compare before/after results, be sure to keep a backup of the original for testing.

---

## Author :construction_worker:
Henna Venho