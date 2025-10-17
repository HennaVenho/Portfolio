# :running: C-sprint

## :earth_africa: Overview

This repository contains a **pure ANSI C** implementation of a command-line tool called `cache`, developed as part of the **C-Sprint** phase of the Embedded Systems module.

It simulates a lightweight in-memory **key-value store**, supporting up to 20 named tables and 10,000 entries per table. All core operations (`CREATE`, `ADD`, `GET`, `DELETE`, etc.) are handled via a clean CLI interface.

The project focuses on:

- **Manual memory management** with `malloc`/`free`
- **Hash table design** using **FNV-1a** hashing and **separate chaining**
- **Performance-aware string handling** (`memcpy`, `snprintf`)
- **Strict C practices** with `-Wall -Wextra -Werror`
- **Leak-free execution**, verified with Valgrind
- **Full test compatibility** with the official **Kood/Sisu Docker test suite**

This project serves both as a **systems programming learning tool** and a **performance-critical prototype** for future embedded development.

---

## :book: Table of Contents 
- [Overview](#earth_africa-overview)
- [Technologies Used](#wrench-technologies-used)
- [Getting Started](#jigsaw-getting-started)
- [Usage Guide](#rocket-usage-guide)
- [Optimization Notes & Performance Considerations](#high_brightness-optimization-notes--performance-considerations)
- [Author](#female_detective-author)

---

## :wrench: Technologies Used 

- **C (ANSI C, C99)** — all logic implemented in pure C using only allowed standard library headers (`<stdio.h>`, `<stdlib.h>`, `<string.h>`, etc.)
- **Manual Memory Management** — dynamic allocation via `malloc`, `calloc`, and `free`; no memory leaks under Valgrind
- **Hash Table with FNV-1a Hashing** — optimized hash function with separate chaining (linked lists) for collision resolution
- **Modular Code Structure** — split across `main.c`, `commands.c`, `hashtable.c`, and matching headers
- **Safe String Handling** — uses `snprintf()` and `memcpy()` with boundary checks to prevent buffer overflows
- **Docker-based Testing** — validated against Kood/Sisu's official test suite using a provided Docker image
- **Makefile Build System** — strict compilation with `-Wall -Wextra -Werror` to enforce safe coding practices
- **Optimized for Performance** — static inlining, hash lookup improvements, string copy optimizations

---

## :jigsaw: Getting Started

### Clone the Repository

```bash
git clone https://github.com/HennaVenho/c-sprint.git
cd c-sprint
```

---

### To Run the Official Test Suite (Recommended):

1. **Pull the Docker image:**
    ```bash
    docker pull ghcr.io/futurecoders-org/c-sprint:latest
    ```

2. **Run the tests using the provided script:**
    ```bash
    ./test-all.sh
    ```
    This runs all Kood/Sisu-provided tests inside the correct environment.

3. **Or test manually (example for the `cache` task):**
    ```bash
    docker run -v $PWD:/programs ghcr.io/futurecoders-org/c-sprint cache
    ```

---

### To Build and Run the `cache` CLI Program Locally (Optional):

> This is optional and mostly useful for experimenting or adding your own inputs.

1. **Ensure required tools are installed:**
    ```bash
    sudo apt update
    sudo apt install build-essential
    ```
    - This provides `gcc`, `make`, and basic C development tools.

2. **(Optional) Install Valgrind if you want to test memory leaks:**
    ```bash
    sudo apt install valgrind
    ```

3. **Build the program:**
    ```bash
    cd cache
    make
    ```

4. **Run interactively:**
    ```bash
    ./cache
    ```

5. **Run custom test files:**
    ```bash
    ./cache < test_files/test_input.txt
    ./cache < test_files/test_table_limit.txt
    ./cache < test_files/test_key-value_limit.txt
    ```

6. **(Optional) Run with Valgrind:**
    ```bash
    valgrind ./cache < test_files/test_input.txt
    ```

7. **Clean up build artifacts:**
    ```bash
    make clean
    ```

---

## :rocket: Usage Guide

The `cache` program can also be tested interactively or via pre-written test scripts found in `test_files/` (in `/cache` directory).

---

### Running Valgrind on Custom Inputs

To check for memory leaks and errors:
```bash
valgrind ./cache < test_files/test_input.txt
valgrind ./cache < test_files/test_key-value_limit.txt
valgrind ./cache < test_files/test_table_limit.txt
```

---

<details> <summary>🧾 Example Interactive Session Output</summary>

```bash
$> make
$> ./cache
Welcome to Cache.
Type HELP to see the usage.
Type EXIT to exit.

cache> HELP
Available commands:

HELP - HELP
LIST - LIST
CREATE - CREATE <table>
ADD - ADD <table> <key> <value>
COUNT - COUNT <table>
GET - GET <table> <key>
DELETE - DELETE <table> <key>
DROP - DROP <table>
EXIT - EXIT

cache> CREATE table1
TABLE table1 created

cache> CREATE table1         
TABLE table1 already exists

cache> ADD table1 key1 value1
VALUE value1 at KEY key1 added to TABLE table1

cache> ADD table1 key1 value2
VALUE value2 at KEY key1 added to TABLE table1

cache> GET table1 key1
VALUE: value2

cache> COUNT table1
Item count in TABLE table1: 1

cache> LIST
table1

cache> DELETE table1 key1
KEY key1 deleted from TABLE table1

cache> GET table1 key1
KEY key1 doesn't exist in TABLE table1

cache> DELETE table1 key1
KEY key1 doesn't exist in TABLE table1

cache> DROP table1
TABLE table1 deleted

cache> LIST
No tables exist

cache> COUNT table1
TABLE table1 doesn't exist

cache> ADD table1 key1 value1
TABLE table1 doesn't exist

cache> DELETE table1 key1
TABLE table1 doesn't exist

cache> GET table1 key1
TABLE table1 doesn't exist

cache> DROP table1
TABLE table1 doesn't exist

cache> EXIT
Exiting...
```
</details>

---

## :high_brightness: Optimization Notes & Performance Considerations

The `capacity_test` is the most performance-intensive test in the C-sprint `cache` program. It inserts up to **200,000 key-value entries** across multiple hash tables.


#### Confirmed Passing Environment

This implementation **passes all 27 tests**, including the `capacity_test`, in the following environment:

- **Ubuntu 24.04 VM**
- **10 GB RAM, 8 CPU cores**
- **Docker installed directly inside the VM**

---

### Known Timeout Risks

In **constrained or layered environments**, this test may fail due to timeouts — even if functionally correct:

- **WSL (Windows Subsystem for Linux)**
- **Docker Desktop on Windows**
- **Low-resource VMs (e.g. 6 GB RAM, 2 CPUs)**

These setups add **virtualization overhead**, **nested syscall layers**, and may delay memory ops and malloc/free under pressure.

---

### Key Code Optimizations

Several performance improvements were implemented:
- **Efficient String Copying**: Used `snprintf()` and `memcpy()` instead of `strncpy()` to avoid unnecessary overhead.
- **Optimized Hash Function**: Declared `fnv1a_hash()` as `static inline` to eliminate call overhead and improve compiler inlining.
- **Memory Management**: All memory is properly freed, and the code is Valgrind-clean.
- **Collision Resolution**: Used simple and fast separate chaining with linked lists.

---

### Tips for Reliable Testing

If you're running into timeouts:
- Close other apps and background processes
- Prefer **dedicated Linux VMs** or physical machines over WSL/Docker Desktop
- Retry the test — results may vary based on system load

This implementation is fully functional and **passes all correctness and memory leak tests**. Any performance issues under constrained environments are related to runtime overhead — not bugs in the code.

---

## :female_detective: Author

Henna Venho
