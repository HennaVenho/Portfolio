# 🧬 Binary Fusion

## :earth_africa: Overview

**Binary Fusion** is a command-line tool that fuses multiple ELF executables into a single self-contained binary. During the fusion process, the payloads can optionally be compressed using Zstandard to reduce the size of the final file.

When the fused binary is executed, it silently and sequentially runs each embedded payload. If payloads were compressed, they are automatically decompressed at runtime. Outputs are redirected based on their index — to `stdout`, `stderr`, or other file descriptors — enabling redirection-aware usage and stealthy execution chains.

You can optionally use the `--diagnose` flag to inspect the ELF headers of input and output files (e.g. architecture and file type). A `tests/quick_tests.sh` script is provided for fast validation, and the `demo/` folder contains examples and payloads for testing and demonstration purposes.

Built for a Cybersecurity module focused on binary formats and malware techniques, **Binary Fusion** demonstrates how **benign and malicious binaries can be concealed within a single executable** — simulating techniques used in:
- Dropper malware
- Runtime loaders & injection
- Fileless attack techniques

> 🔒 This fusion mechanism enables potentially malicious payloads to be embedded inside otherwise innocuous software — fulfilling the red-team learning goals of the project.

The tool is fully self-contained and works entirely on ELF binaries. It is implemented in:
- **Python** — for fusion logic and metadata building
- **C** — for the runtime ELF loader stub

---

## :book: Table of Contents 
- [Overview](#earth_africa-overview)
- [Technologies Used](#wrench-technologies-used)
- [Setup Instructions](#jigsaw-setup-instructions)
- [Usage Guide](#rocket-usage-guide)
- [Bonus Features](#gift-bonus-features)
- [Key Concepts](#key-key-concepts)
- [Author](#female_detective-author)

---

## :wrench: Technologies Used 

| Component                             | Description                                                     |
| ------------------------------------- | --------------------------------------------------------------- |
| **Python 3.12.3**                     | Core fusion logic (header parsing, compression, output control) |
| **GCC (C Compiler)**                  | Used to compile the custom loader binary (written in C)         |
| **Zstandard (Zstd)**                  | Optional compression of payloads to reduce fused binary size    |
| **Makefiles**                         | Used to compile examples and loader with correct flags          |
| **Bash scripts**                      | `setup.sh` and `quick_tests.sh` to automate build & test tasks  |
| **Visual Studio Code + WSL (Ubuntu)** | Primary development environment                                 |
| **Git + Gitea**                       | Version control and remote hosting                              |

---

## :jigsaw: Setup Instructions

This project requires Python and C build tools. Please follow the steps below to set up and run **Binary Fusion** locally.

---

### Requirements

To build and run the program, you will need:

| Tool                  | Required       | Description                                                                |
| --------------------- | -------------- | -------------------------------------------------------------------------- |
| Python 3.10+          | ✅ Yes         | Used to run the fusion tool (requires `zstandard`)                         |
| `build-essential`     | ✅ Yes         | Provides `gcc`, `make`, and `ld` for compiling the ELF loader              |
| Git                   | ✅ Yes         | Used to clone the repository                                               |
| `venv`                | ⭐ Recommended | Isolates Python dependencies for clean environments                        |
| `file`, `xxd`, `tree` | 🛠️ Optional    | Useful command-line tools for inspecting ELF binaries and debugging output |

---

### 1. Install System Packages (Ubuntu / WSL2 / Debian)

```bash
sudo apt update
sudo apt install python3 python3-pip python3-venv build-essential -y
```

Optional CLI tools:
```bash
sudo apt install file xxd tree -y
```

---

### 2. Clone the Repository

```bash
git clone https://github.com/HennaVenho/binary-fusion.git
cd binary-fusion
```

---

### 3. Set Up Python Environment and Install Dependencies

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt 
``` 
This installs the `zstandard` compression library and other dependencies.

---

### 4. Run Setup Script

This script compiles everything and creates the CLI tool `./fuser`.

```bash
chmod +x setup.sh       # Executable rights should be set automatically but run this if needed
./setup.sh
```

It will:
- Compile examples (`jules`, `vincent`) -> `build/`
- Build the C ELF loader (`loader/`) -> `build/`
- Create the `./fuser` entrypoint (wrapper around the Python CLI)

You can now run:
```bash
./fuser build/jules build/vincent -o output/fused_jv
```

---

### 5. Run Quick Tests (Optional)

To verify that everything works correctly:

```bash
chmod +x tests/quick_tests.sh       # Executable rights should be set automatically but run this if needed
./tests/quick_tests.sh
```

This script:
- Fuses `jules` + `vincent`
- Validates ELF headers
- Runs the fused binary twice:
    - Once directly
    - Once with output redirection (`stdout` -> `tests/out1.txt`, `stderr` -> `tests/out2.txt`)
- Prints the outputs

✅ A passing test shows the tool is working end-to-end.

---

### 6. Exiting the Python Virtual Environment

When you're done using the tool:

```bash
deactivate
```

---

### Troubleshooting

If you get permission errors running `./fuser` or e.g. `./output/fused_output`, make sure they are executable:
```bash
chmod +x fuser
chmod +x output/fused_output
```

After the setup is done, please check [Usage Guide](#running-usage-guide) for instructions on how to run the program.   

---

## :rocket: Usage Guide

Binary Fusion fuses multiple ELF executables into a single binary that executes all payloads in order. You can optionally compress them and redirect their output to separate streams.

> ⚠️ Fusing already fused binaries is not officially supported at this time. Execution order may vary due to nested loaders and fork timing. This is a known limitation.

---

### Display Help

Run the following to view the CLI options:
```bash
./fuser -h
```

This will print:
```bash
usage: ./fuser [-h] [-o OUTPUT] [-c] [-d] [binaries ...]

Binary Fusion: Fuse Multiple ELF Executables into a Single Fused Binary

positional arguments:
  binaries              Paths to the payload binaries to fuse (e.g. build/jules build/vincent ...).

options:
  -h, --help            show this help message and exit
  -o OUTPUT, --output OUTPUT
                        Path to output fused binary (default: output/fused_output)
  -c, --compress        Compress the size of the fused binary with Zstd
  -d, --diagnose        Print ELF header info for input and output files
```

---

### Copy Real Linux Binaries (e.g. `ls`, `whoami`)

You can safely test with real executables by making *copies* of trusted system binaries (never fuse original `/usr/bin/...` files directly):
```bash
cp /usr/bin/ls build/ls-copy
cp /usr/bin/whoami build/whoami-copy
chmod +x build/ls-copy build/whoami-copy
```

---

### Example Usage Scenarios

Below are tested usage examples. You can find more in [demo/USAGE_EXAMPLES.md](demo/USAGE_EXAMPLES.md).


#### Basic Fusion

Fuse two example payloads (`jules` and `vincent`) using the default output path:
```bash
./fuser build/jules build/vincent
./output/fused_output
```

#### Specify Custom Output File

Fuse 3 payloads and save under a custom name with `-o <filename>`:
```bash
./fuser build/jules build/vincent build/ls-copy -o output/fused_multi
./output/fused_multi
```

#### Redirect Output (1 file per payload)

Redirect each program's output:
```bash
./output/fused_multi > output/jules.txt 2> output/vincent.txt 3> output/ls.txt
```

Inspect the output files: 
```bash
cat output/jules.txt
cat output/vincent.txt
cat output/ls.txt
```

#### Fuse with Compression

Use `-c` or `--compress` to enable Zstandard compression:
```bash
./fuser build/jules build/vincent build/ls-copy build/whoami-copy -o output/fused_compressed -c
./output/fused_compressed
```

And optionally redirect all outputs:
```bash
./output/fused_compressed > output/jules.txt 2> output/vincent.txt 3> output/ls.txt 4> output/whoami.txt
```

Optional: To verify the size difference between compressed and uncompressed fused files, use e.g. `ls -lh output/`

#### Print Diagnostics

The `-d` or `--diagnose` flag prints ELF header details:
```bash
./fuser build/jules build/vincent -o output/fused_diagnose -d
```

Example output:
```bash
[*] Validating ELF headers...
[build/jules]
  Magic     : b'\x7fELF'
  Arch      : 62 (expected 0x3E for x86_64)
  File Type : ET_EXEC (Executable file)
  ELF class : ELFCLASS64 (2)
  Endianness: Little-endian (1)
...
```

#### Want to Just Sanity-Check the Tool?

See the [Setup Instructions](#jigsaw-setup-instructions) for how to run the automated quick test (`tests/quick_tests.sh`).

---

## :gift: Bonus Features

In addition to basic two-binary fusion, **Binary Fusion** includes several extra capabilities:

- **Zstandard Compression Support**
    - Use `-c` or `--compress` to compress each payload before embedding.
    - At runtime, decompression is handled **automatically**, with no user interaction required.
    - Greatly reduces the size of large binaries, especially useful for large payload chains.

- **Output Redirection Support**
    - Each original payload's output can be redirected to a separate file using standard shell redirection:
        ```bash
        ./fused_binary > out1.txt 2> out2.txt 3> out3.txt ...
        ```

- **Flexible Output Naming**
    - Use `-o <filename>` or `--output <filename>` to customize the fused binary name and path.
    - If omitted, a **smart auto-incrementing default** (`output/fused_output`, `output/fused_output1`, etc.) is used to prevent accidental overwrites.

- **N-Way Fusion (Multiple Executables)**
    - Fuse **more than two** ELF executables in a single run.
    - No hardcoded limits: 2, 3, 4, or more payloads are all supported.

- **Works with Real System Executables**
    - Supports both **statically linked** and **dynamically linked** binaries (e.g. `ls`, `whoami`, `date`, `hostname`).
    - Useful for testing real-world payload fusion scenarios.

- **Diagnostic Mode**
    - Use `-d` or `--diagnose` to print parsed ELF header metadata for all input and output binaries.
    - Prints and validates:
        - ELF magic number
        - File type (`e_type`): e.g. `ET_EXEC`, `ET_DYN`
        - Architecture (`e_machine`): e.g. `EM_X86_64`
        - ELF class (`ei_class`): 32-bit vs 64-bit (`ELFCLASS64`)
        - Endianness (`ei_data`): little-endian vs big-endian

- **Quick Test Script**
    - Run `./tests/quick_tests.sh` to:
        - Auto-compile test binaries (`jules`, `vincent`)
        - Fuse them into `output/fused_jv`
        - Validate headers and output redirection
        - Show output and verify correctness

- **One-Step Setup**
    - Run `./setup.sh` to:
        - Build the loader and examples
        - Create the `./fuser` entrypoint
        - Set correct executable permissions

---

## :key: Key Concepts

<details id="concept-of-binary-file-formats-and-their-importance-in-software-development-and-cybersecurity">
<summary>Concept of Binary File Formats and Their Importance in Software Development and Cybersecurity</summary>

- Binary file formats define how software is stored and interpreted by the operating system. These formats are crucial in both development and cybersecurity.

#### What is a Binary File Format?

- A binary file format (like **ELF** on Linux, **PE** on Windows, or **Mach-O** on macOS) specifies the **structure** of an executable file, including:
    - Magic number (file identity)
    - Entry point (where execution begins)
    - Architecture (e.g. x86_64)
    - File type (e.g. executable or shared object)
    - Offsets and sizes of code, data, and metadata sections

- These formats allow the OS loader to correctly map, interpret, and execute the program.

#### Why It Matters in Software Development

- Developers rely on binary formats to **build, link, and deploy** software.
- Understanding them enables debugging, inspection (e.g. `readelf`, `objdump`), and compatibility checks (e.g. 32-bit vs 64-bit).

#### Why It Matters in Cybersecurity

- Binary formats are **critical for reverse engineering, malware analysis**, and detecting obfuscation. Attackers often:
    - Embed hidden payloads inside executables
    - Modify headers to evade detection
    - Craft "droppers" or "packers" that unpack code at runtime

#### How This Project Uses ELF Internals

- This project works directly with ELF files by:
    - Parsing ELF headers manually (e.g. `e_type`, `e_machine`)
    - Validating architecture and file type before fusing
    - Injecting multiple ELF binaries into a single host
    - Optionally compressing payloads and decompressing them at runtime
    - Appending **custom metadata** at the end of the ELF file

- By doing this, the project shows how ELF structure can be manipulated to **embed multiple programs into one**, which is both a powerful technique and a cybersecurity concern.

</details>


<details id="my-choice-of-executable-format">
<summary>My Choice of Executable Format</summary>

#### Why I Chose ELF

- The **Executable and Linkable Format (ELF)** is the native binary format used on Linux, including under WSL and many Unix-like systems.

- It was chosen for this project because:
    - **Native to Linux** — default format for `.out`, `.so`, and all executables.
    - **Well-documented** — with clear specifications, byte offsets, and field meanings.
    - **Easily inspectable** — tools like `readelf`, `objdump`, and `file` make analysis easy.
    - **Flexible** — ELF supports multiple file types (e.g. executables, shared objects, core dumps).
    - **Widely used in malware analysis and reverse engineering**.

#### Supported ELF Types

- The tool currently supports two ELF types:

    | ELF Type   | Description                                 | Supported?  |
    |------------|---------------------------------------------|-------------|
    | `ET_EXEC`  | Statically linked executable binaries       | ✅ Yes      |
    | `ET_DYN`   | Dynamically linked shared/PIE executables   | ✅ Yes      |

    > Note: Many system programs like `ls` or `whoami` are compiled as PIEs (Position Independent Executables: `ET_DYN`). These are still fully executable and are supported by the fusion loader.

#### Implementation Notes

- Each ELF binary is **parsed**, **validated**, and **embedded** using raw byte-level access.
- The loader is itself a valid **64-bit ELF executable**.
- The metadata block is appended directly at EOF and located by a trailing offset pointer.

- This format enabled full control over execution behaviour while staying compatible with Linux tooling and runtime expectations.

</details>


<details id="tools-used-for-this-project">
<summary>Tools Used for This Project</summary>

#### Core Tools and Technologies

| Tool                          | Role in Project                                                                |
|-------------------------------|--------------------------------------------------------------------------------|
| **Python 3.12.3**             | Main fusion logic, compression, CLI, metadata handling                         |
| **zstandard (Python module)** | Used to compress ELF payloads during fusion                                    |
| **GCC**                       | Used to compile the ELF loader (`loader/`) and example binaries (`examples/`)  |
| **Make**                      | Simplifies and automates C compilation for loader and examples                 |
| **C (loader.c)**              | The embedded runtime stub that executes payloads at runtime                    |
| **VS Code + WSL Ubuntu**      | Development environment (Linux-based toolchain + editor integration)           |
| **Bash scripting**            | Used in `setup.sh` and `tests/quick_tests.sh` to automate setup and testing    |

#### Auxiliary Tools (Optional but Useful)

- `file`, `readelf`, `xxd`, `tree` — CLI tools for analyzing ELF headers and filesystem layout.
- `chmod`, `cat`, output redirection (`>`, `2>`, `3>`, etc.) — used during demonstration of output handling.

#### Design Rationale

- **Python** was chosen for its speed of development, built-in support for binary data, and clean CLI implementation.
- **C** was chosen for the loader because it needed to produce a raw, self-contained ELF executable without external dependencies.
- **Make** keeps build automation reproducible and cross-platform.
- **zstandard** was added for real-world compression, balancing performance and file size.

</details>


<details id="key-fields-in-the-main-header-of-elf">
<summary>Key Fields in the Main Header of ELF</summary>

#### ELF Header Overview

- The **ELF (Executable and Linkable Format)** header is located at the beginning (offset `0x00`) of every ELF binary. It contains the OS loader's instructions on how to interpret the file.

- Key fields include:

    | Field        | Offset | Size | Description                                                         |
    |--------------|--------|------|---------------------------------------------------------------------|
    | `e_ident`    | `0x00` | 16 B | Magic number (`\x7FELF`), bitness (32/64), endianness, ABI info     |
    | `ei_class`   | `0x04` |  1 B | ELF class: `1 = ELFCLASS32`, `2 = ELFCLASS64`                       |
    | `ei_data`    | `0x05` |  1 B | Endianness: `1 = little-endian`, `2 = big-endian`                   |
    | `e_type`     | `0x10` |  2 B | File type — e.g. `ET_EXEC`, `ET_DYN` (shared object / PIE)          |
    | `e_machine`  | `0x12` |  2 B | Target architecture — e.g. `EM_X86_64` (64-bit x86)                 |
    | `e_entry`    | `0x18` |  8 B | Entry point virtual address (where execution starts)                |
    | `e_phoff`    | `0x20` |  8 B | Offset to program header table (used to load segments into memory)  |
    | `e_shoff`    | `0x28` |  8 B | Offset to section header table (used for linking, debugging)        |
    | `e_flags`    | `0x30` |  4 B | Architecture-specific flags (usually 0 for x86_64)                  |
    | `e_ehsize`   | `0x34` |  2 B | ELF header size in bytes (typically 64 for 64-bit ELF)              |

#### Fields Used in This Project

- My Python parser extracts and validates:
    - **`e_ident`: magic number, class, and endianness**
    - **`ei_class`**: must be `2 = ELFCLASS64`
    - **`ei_data`**: must be `1 = little-endian`
    - **`e_type`**: must be `ET_EXEC` or `ET_DYN` to be runnable
    - **`e_machine`**: must match `EM_X86_64`

- This validation ensures correct runtime behaviour and avoids segmentation faults caused by format or architecture mismatches.

#### Demo Tip:

- To inspect these fields manually, run:
    ```bash
    readelf -h build/jules
    ```

</details>


<details id="how-the-tool-differentiates-between-executable-shared-object-and-relocatable-files-in-elf">
<summary>How the Tool Differentiates Between Executable, Shared Object, and Relocatable Files in ELF</summary>

#### ELF File Type Detection (`e_type`)

- The tool uses the `e_type` field in the ELF header to identify the file type. This 2-byte field is located at offset `0x10` in the header and is parsed using `struct.unpack_from('<H', ...)` in Python.

#### Supported ELF Types

| Type        | Hex   | Description                                                                   |
|-------------|-------|-------------------------------------------------------------------------------|
| `ET_EXEC`   | 0x02  | **Executable file** (e.g. statically linked binaries like `jules`, `vincent`) |
| `ET_DYN`    | 0x03  | **Shared object** — used for `.so` files, but also for PIE executables        |
| `ET_REL`    | 0x01  | **Relocatable object** — `.o` files, not directly executable                  |

- ✅ **Accepted types**: `ET_EXEC` and `ET_DYN`
- ❌ **Rejected types**: `ET_REL`, `ET_NONE`, and others

#### Why PIE (`ET_DYN`) Is Supported

- Most Linux system binaries are built as PIEs (Position Independent Executables), which show up as `ET_DYN` in the ELF header — even though they are fully runnable.
    - Examples: `/usr/bin/ls`, `/usr/bin/whoami`, `/usr/bin/bash`

- Therefore, **`validate_parsed_elf_header()` accepts both `ET_EXEC` and `ET_DYN`**, ensuring compatibility with modern Linux binaries.

#### Key Takeaway

- The tool **intentionally allows both statically linked (`ET_EXEC`) and PIE-style (`ET_DYN`) binaries**, but strictly **rejects non-runnable ELF files** like object files or static libraries.

- Invalid types produce helpful errors:
    ```bash
    [!] Validation failed: build/loader.o: Not a runnable ELF binary — detected type: ET_REL (Relocatable object)
    ```

</details>


<details id="modifying-the-entry-point-and-ensuring-execution-flow">
<summary>Modifying the Entry Point and Ensuring Execution Flow</summary>

#### How Execution Is Controlled in This Project

- Rather than modifying the `e_entry` (entry point address) of an existing ELF binary, the tool **uses a custom-built C loader stub**, compiled into a standalone ELF executable.

- This **loader becomes the entry point** of the fused binary and controls all execution.

#### Why This Approach?

- Directly modifying an existing binary's `e_entry` or injecting shellcode can easily corrupt it, break relocations, or violate ELF constraints.

- By creating a clean, valid loader ELF from scratch, I:
    - Avoid overwriting or injecting code into fragile binaries
    - Guarantee predictable behaviour
    - Support both `ET_EXEC` (e.g. statically-linked) and `ET_DYN` (e.g. PIE) formats

#### How the Loader Executes the Payloads

- At runtime, the `loader` stub:
    1. Reads the **metadata** appended to the end of the fused binary to find: 
        - Offsets and sizes of each embedded payload
        - Flags (e.g. compression enabled)
    2. Extracts each payload into a temporary file
    3. If the payload is compressed, decompresses it with Zstandard
    4. Sets execute permission on the temporary file
    5. **Forks and executes each payload in order**:
        - Uses `fork()` (or `clone()`) to create a subprocess
        - Uses `execl()` to run the binary
        - Uses `waitpid()` to block until the child exits

    - This guarantees that payloads are **run in order**, one after the other.

#### Result

- The fused binary is a fully valid ELF executable.
- Its only `e_entry` is the custom loader — the loader then executes each payload as a subprocess.
- This ensures **safe, deterministic, sequential execution** of all embedded binaries, without modifying their internal headers.

</details>


<details id="fused-binary-layout-and-how-section-conflicts-are-avoided">
<summary>Fused Binary Layout and How Section Conflicts Are Avoided</summary>

#### Challenge: Combining Executables Without Section Conflicts

- ELF binaries contain sections and segments that define how code and data are loaded into memory.

- Naively merging `.text`, `.data`, or `.bss` from multiple binaries can cause:
    - Virtual memory collisions
    - Broken relocation entries
    - Segfaults or undefined behaviour at runtime

#### Solution: Dropper-Style Layout (No Section Merging)

- Instead of modifying or merging ELF section layouts, this project uses a **simple and robust layout strategy**:
    ```
    [loader ELF] + [payload 1] + [payload 2] + ... + [metadata trailer]
    ```

- Each payload is stored as a raw byte blob:
    - No headers are modified
    - No sections are merged
    - Each payload retains its original internal structure

- The loader extracts and executes each payload as an isolated ELF binary:
    - It writes the blob to disk
    - Sets execute permissions
    - Uses `fork()` + `execl()` to run it in a new process

#### Why This Approach Is Safer

- This design:
    - Avoids ELF corruption
    - Supports multiple payloads regardless of their internal layouts
    - Handles both `ET_EXEC` and `ET_DYN` types
    - Works even if payloads contain overlapping virtual sections

- It also mirrors the behaviour of many real-world malware droppers and loaders:
    - Embed -> Extract -> Execute
    - No need for complex relocation or re-linking logic

#### Summary

- While the tool doesn't perform "basic section concatenation" in the traditional linker sense, it achieves the same outcome:
    - Multiple executables run from one binary
    - Section conflicts are entirely avoided by design

</details>


<details id="validation-steps-to-ensure-the-fused-binary-is-structurally-valid-and-executable">
<summary>Validation Steps to Ensure the Fused Binary is Structurally Valid and Executable</summary>

#### Validation at Fusion Time (Python Tool)

- Before generating the fused binary, `fuser.py` performs strict validation for all input files:
    - **Magic number**: Must begin with `\x7fELF` (4-byte ELF identifier)
    - **Bitness**: Only 64-bit ELF files are supported (`ELFCLASS64`)
    - **Endianness**: Only little-endian ELF files are accepted (`ei_data = 1`)
    - **File type**: Must be a runnable executable (`ET_EXEC`) or PIE (`ET_DYN`)
    - **Architecture**: Must match `EM_X86_64` — all inputs must use the same target architecture
    - **Offset computation**: Dynamically calculates payload offsets to prevent overlap
    - **Metadata creation**: Stores offsets, sizes, and compression flags in a compact header at the end of the fused file

- These checks prevent malformed ELF inputs, format mismatches, or invalid payloads from entering the fusion process, and ensure that the resulting fused binary has a valid ELF structure and embeds its payloads in a predictable, aligned, and recoverable way.

#### Validation at Runtime (C Loader Stub)

- Once the fused binary is executed:
    1. The loader opens its own file via `/proc/self/exe`
    2. Reads the metadata footer to find each payload
    3. Verifies sizes and flags, decompresses if needed
    4. Extracts and writes each binary to a temporary location
    5. Applies `chmod +x` to ensure execution rights
    6. Executes the binaries **sequentially** using `fork()` -> `execl()` -> `waitpid()`

- This runtime pipeline guarantees that both ELF structure and embedded payloads are handled correctly without any direct mutation of their headers or sections.

</details>


<details id="handling-matching-architecture-and-bitness">
<summary>Handling Matching Architecture and Bitness</summary>

#### Why Architecture Compatibility Matters

- All payloads in a fused binary must be compatible with the same CPU architecture and bit width (e.g. 64-bit).

- Mixing 32-bit and 64-bit ELF files (or other mismatched architectures like ARM vs x86_64) would lead to:
    - Loader crashes
    - OS runtime errors (e.g. "Exec format error")
    - Inability to execute the embedded binaries reliably

#### What This Tool Enforces

- At **fusion time**, the tool enforces two strict requirements:

    | Field         | Value Required         | Description                              |
    |---------------|------------------------|------------------------------------------|
    | `ei_class`    | `ELFCLASS64 (2)`       | Ensures that input binaries are 64-bit   |
    | `e_machine`   | `EM_X86_64 (0x3E)`     | Requires x86_64 architecture             |

- These values are read from the ELF header and compared across all input binaries.
- If any mismatch is detected, the fusion process fails with a descriptive error.

#### Why 32-bit ELF Files Are Rejected

- The tool currently supports only **64-bit ELF files**.

- Supporting 32-bit (`ELFCLASS32`) would require major changes to:
    - The loader's compilation and execution logic
    - Memory alignment and ABI compatibility

- Instead of partial support, the tool **explicitly rejects** incompatible files.

#### Example Error

```bash
[!] Validation failed: build/thirtytwo: Incompatible ELF file — ELF class: ELFCLASS32 (1) (expected ELFCLASS64); Architecture: 0x3 (expected EM_X86_64 / 0x3E)
```

</details>


<details id="testing-methodology-for-verifying-correct-sequential-execution-of-both-programs-in-the-fused-binary">
<summary>Testing Methodology for Verifying Correct Sequential Execution of Both Programs in the Fused Binary</summary>

#### Smoke Testing with `quick_tests.sh`

- To verify that the fused binary executes all embedded payloads **in the correct order**, I created an automated test script:
    ```bash
    tests/quick_tests.sh`
    ```
- It performs the following steps:
    1. **Compiles the test programs**:
        - `jules`: Prints `"What do they call it?"`
        - `vincent`: Prints `"They call it a Royale with Cheese."`
    2. **Fuses the binaries**:
        ```bash
        ./fuser build/jules build/vincent -o output/fused_jv
        ```
    3. **Executes the fused binary**:
        ```bash
        ./output/fused_jv > tests/out1.txt 2> tests/out2.txt
        ```
    4. **Validated output**:
        - `tests/out1.txt` contains Jules's line
        - `tests/out2.txt` contains Vincent's line

    - This confirms **deterministic sequential execution** based on input order.

#### Manual and Edge Case Testing

- I also performed extensive manual testing to verify reliability under real-world conditions:
    - **Execution order validation with `strace`**:
        - Used `strace -f -o strace_fused.log ./output/fused_jv` to inspect syscalls.
        - Verified presence of `clone()`, `execve()`, and `wait4()` to confirm fork-like behaviour and correct sequential execution.
    
    - **Tested with and without compression**:
        - Fused binaries using `--compress` and confirmed correct output and decompression.

    - **Failure case testing**:
        - Ran binaries where one payload returns non-zero exit code (`exit(1)`), confirming graceful error handling.

    - These manual checks demonstrate the tool's resilience and runtime correctness across common edge cases.

</details>


<details id="fusing-incompatible-executables">
<summary>Fusing Incompatible Executables</summary>

#### Input Validation and Error Reporting

- The tool performs strict **ELF validation** on all inputs before attempting fusion:
    - ELF magic check (`\x7fELF`)
    - 64-bit architecture only (`ELFCLASS64`)
    - Little-endian format only
    - Allowed ELF file types: `ET_EXEC` and `ET_DYN`
    - Matching architecture (`e_machine == EM_X86_64`)

- These checks are handled in `elf_parser.py` (`validate_elf()`), and any mismatch or corrupted input produces a clear error message:
    ```bash
    [!] Validation failed: build/thirtytwo: Incompatible ELF file — ELF class: ELFCLASS32 (1) (expected ELFCLASS64); Architecture: 0x3 (expected EM_X86_64 / 0x3E)
    ```

#### Examples of Invalid Inputs

| Test Case                     | Result                                                                    |
|-------------------------------|---------------------------------------------------------------------------|
| File is not ELF               | "Not a valid ELF file (missing ELF magic number)"                         |
| Wrong bitness (32-bit)        | "Incompatible ELF file — ELF class: ELFCLASS32 (1) (expected ELFCLASS64)" |
| Wrong endianness              | "Only little-endian ELF files are supported"                              |
| Wrong type (`ET_REL`)         | "Not a runnable ELF binary — detected type: ET_REL (Relocatable object)"  |
| Wrong architecture (`ARM`)    | "Incompatible ELF file — Architecture: 0x28 (expected EM_X86_64 / 0x3E)"  |

#### Goal

- The goal is to prevent invalid fusions that would:
    - Break the output binary structure
    - Cause runtime crashes or loader errors
    - Waste developer time due to silent corruption

- This validation ensures **reliable, predictable, and secure fusion** of compatible executables.

</details>


<details id="files-with-non-standard-section-names">
<summary>Files with Non-Standard Section Names</summary>

#### Why Section Names Don't Matter in This Project

- ELF binaries contain **section headers**, each with a name (`.text`, `.data`, `.bss`, etc.).
- These names are often used by linkers or debuggers — but they are **not required for execution**, and may be missing or obfuscated (e.g. in stripped or packed binaries).

#### Robustness by Design

- My fusion tool **does not parse or rely on section names**.
- Instead, it:
    - Validates the ELF magic, architecture, and file type
    - Treats each payload as a self-contained, executable binary blob
    - Appends them to the loader with metadata for offset/size

#### Result

- Binaries with renamed, non-standard, or missing section names (e.g. `.foo`, `.bar`, `.xyz123`) are **accepted and executed normally**.
- This makes the tool:
    - More flexible
    - More compatible with real-world binaries (especially stripped ones)
    - Less likely to break due to harmless cosmetic changes

</details>


<details id="preserving-basic-section-permissions-in-fused-binaries"> 
<summary>Preserving Basic Section Permissions in Fused Binaries</summary>

#### ELF Permissions Overview

- ELF files use section and segment headers to mark memory regions as:
    - **R** = Readable
    - **W** = Writable
    - **X** = Executable

- These permissions are critical for program correctness:
    - Code sections (`.text`) must be `X`
    - Data sections (`.data`, `.bss`) must be `W`

#### Design Decision: No Section Merging

- This tool **never edits or merges ELF sections**.

- Each input binary is:
    - Validated
    - Appended as a raw blob
    - Extracted and executed by the loader in isolation

- Because the original files are untouched:
    - Their section and segment permissions are preserved exactly

#### Result

- The fused binary:
    - Maintains correct permissions for all internal payloads
    - Doesn't risk introducing security issues from permission mismatches
    - Avoids ELF corruption from incorrect flag changes

</details>


<details id="optimization-techniques-used-to-minimize-padding-and-improve-efficiency-in-the-fused-binary">
<summary>Optimization Techniques Used to Minimize Padding and Improve Efficiency in the Fused Binary</summary>

#### Binary Layout Strategy

- The fused binary is designed to be **compact and efficient**:
    - Each embedded ELF payload is stored **immediately after** the previous one.
    - There is **no padding** inserted between the loader, payloads, or metadata.
    - Offsets are calculated precisely during fusion, avoiding wasteful space.

#### Impact

- This keeps the **file size small**, improves **disk caching**, and ensures that metadata is always in a predictable location.
- Since the tool does not rely on memory mapping or section headers, this compact layout works reliably without impacting execution.

#### Possible Future Optimizations

- While the current version does not implement padding or alignment, future improvements might include:
    - **Page-aligned payloads**:
        - Aligning payloads to `4096` bytes (typical page size) for more efficient memory mapping.
        - Useful if direct memory execution is introduced later.
    
    - **Compressed and packed layouts**:
        - Use binary packing techniques or section-level compression.
        - Requires additional logic in the runtime loader.

- These improvements would enhance startup speed or reduce disk usage — but at the cost of additional complexity and runtime overhead.

</details>


<details id="compression-algorithm-and-runtime-decompression"> 
<summary>Compression Algorithm and Runtime Decompression</summary>

#### Compression Strategy

- The tool supports **optional payload compression** via the `--compress` (`-c`) flag.

- It uses the modern and efficient **Zstandard** algorithm:
    - Implemented using the `zstandard` Python module.
    - Compression is applied **individually** to each payload before embedding.
    - The tool marks compressed payloads using a **per-payload metadata flag** (`0x80`).

#### Runtime Decompression

- During execution:
    - The **C loader** reads the payload metadata.
    - If the compression flag is set, it extracts the compressed blob to a temporary file (e.g. `/tmp/compressed_payload_ABC123`).

    - The loader then launches a **Python decompression script** (`decompress.py`):
        - This decompresses the blob to a new ELF file (`/tmp/decompressed_payload_ABC123`).
        - Finally, the loader executes the decompressed binary.
    
    - If the flag is not set, it executes the extracted payload directly.

#### Advantages

- This hybrid architecture:
    - Keeps the **C loader lightweight** and maintainable.
    - Leverages **Python's mature Zstandard bindings** for compression and decompression.
    - Allows **automatic decompression** with no user intervention or additional flags at runtime.

- Especially useful when fusing:
    - Large binaries
    - Long chains of payloads

#### Impact on Performance

- **Reduces fused binary size** significantly.
- **Slightly increases startup time**, due to decompression and disk I/O.
- Designed for **modularity** — compression logic is isolated in `compression.py` and `decompress.py`.

</details>


<details id="how-the-tool-manages-file-descriptors-and-output-redirection-for-each-payload"> 
<summary>How the Tool Manages File Descriptors and Output Redirection for Each Payload</summary>

#### Design Goals

- Each payload's output should be independently redirectable by the user.
- Output should be written to the **correct file descriptor** without requiring any flags or arguments.

#### Implementation

- The loader captures output for each payload using a custom `run_exec_capture()` function in `loader.c`:
    - Uses `pipe()` + `fork()` + `dup2()` to redirect child output.
    - Stores and returns the output string and exit code.

- Output is routed as follows:
    - Payload 0 -> `stdout` (fd 1)
    - Payload 1 -> `stderr` (fd 2)
    - Payload 2 -> fd 3
    - Payload 3 -> fd 4
    - ... and so on.

- Before writing, the loader verifies each fd is open and writable

#### Example Usage

- For more than 2 payloads:
    ```bash
    ./fused_all > p0.txt 2> p1.txt 3> p2.txt 4> p3.txt 5> p4.txt
    ```

#### Temp Files and Runtime Handling

- Each payload is extracted to a temp file using `mkstemp()`:
    - `/tmp/XXXXXX` for uncompressed
    - `/tmp/decompressed_payload_XXXXXX` for compressed

- All temporary files are **deleted** after execution to avoid clutter.

#### Benefits

- Supports arbitrary number of payloads with no configuration needed.
- Each program's output is clearly separated.
- Minimal overhead and works across different shells and OS configurations.

</details>

---

## :female_detective: Author
Henna Venho
