# how to create a menu of predefined options with tuilib

### context:

I made a python module with the curses library because I felt like there were no high-level options

if you're new to programming, high level is typically easier than low level because low level is closer to hardware meaning memmory management, memmory pointers, unreadable syntax, take assembly you need to know if your cpu uses intel or amd syntax, if your cpu is x86 x64 or x86_64 and thats just to know what version of assembly you need to learn you also need to know your os if you're going to use syscalls which is a necesity, you need to know if your os is 64 bit or 32 bit because 64 bit programs run better on 64 bit oses but can't run at all on 32 bit oses and yeah this is compilcated not to mention in asssembly you need to know memmory adresses, like mess up one number you accidentally messed up your kernel, oh wait you don't know what a kernel is? it basically tells your computer 'hey theres an operating system here' which is different from a boot sector which tells your computer 'hey when you turn on you can boot into this drive' and you get the point oh and you also need to know what assembler you're using, an assembler translates assembly to machine code

take this x86_64 nasm 64 bit hello world program:


```assembly

extern GetStdHandle
extern WriteFile
extern ExitProcess

section .rodata

msg db "Hello World!", 0x0d, 0x0a

msg_len equ $-msg
stdout_query equ -11

section .data

stdout dw 0
bytes_written dw 0

section .text

global start

start:
    mov rcx, stdout_query
    call GetStdHandle
    mov [rel stdout], rax

    mov  rcx, [rel stdout]
    mov  rdx, msg
    mov  r8, msg_len
    mov  r9, bytes_written
    push qword 0
    call WriteFile

    xor rcx, rcx
    call ExitProcess
```

now you know low level, high level is closer to human readable langauge, eg take a look at this python code

```python
for i in range(5):
    print(f"iteration {i}")
```

you know that it will output:

iteration 1

iteration 2

iteration 3

iteration 4

iteration 5

well, close it actually starts at 0 because python uses 0 based indexing, it may sound counterintuitive but in my opinion its more intuitive because 1 based indexing means that 0 is the last element which doesn't make sense

but edit the code:

```python
for i in range(5):
    print(f"iteration {i + 1}")
```

and it outputs as originally expected

and just a note: 

because these definitions are not mutually exclusive(close but not quite) it's technically possible to create a programming language that is both low level and high level but really difficult since theese definitions are 99.9% of the way to being mutually exclusive

this how to guide will show you how to create a menu where the user can select from predefined options in the high level python library I made

# steps:

1. install python
click customize install

enable the options below(windows only)

install python for all users

add python to path

precompile standard library


on linux/macos run the following script with 3.13.2 as the first argument:

```bash
#!/bin/bash

# Install Python for all users with precompiled standard library
set -euo pipefail

# Configuration
PYTHON_VERSION=$1
INSTALL_PREFIX="/usr/local"
SOURCE_DIR="/tmp/python-src"

# Check for root privileges
if [ "$EUID" -ne 0 ]; then
    echo "ERROR: This script must be run as root or with sudo"
    exit 1
fi

# Create temporary source directory
rm -rf "${SOURCE_DIR}"
mkdir -p "${SOURCE_DIR}"
cd "${SOURCE_DIR}"

# Install platform-specific build dependencies
if [ "$(uname)" == "Darwin" ]; then
    # macOS setup
    echo "Installing macOS dependencies..."
    if ! xcode-select -p >/dev/null 2>&1; then
        echo "Xcode Command Line Tools required. Installing..."
        xcode-select --install
        read -p "Press Enter when Xcode installation completes, CTRL-C to abort"
    fi

elif [ "$(uname)" == "Linux" ]; then
    # Debian/Ubuntu Linux setup
    echo "Installing Linux build dependencies..."
    apt-get update
    apt-get install -y build-essential zlib1g-dev libncurses5-dev \
        libgdbm-dev libnss3-dev libssl-dev libsqlite3-dev libreadline-dev \
        libffi-dev libbz2-dev
else
    echo "Unsupported operating system"
    exit 1
fi

# Download and verify Python source
echo "Downloading Python ${PYTHON_VERSION}..."
TARBALL="Python-${PYTHON_VERSION}.tgz"
wget "https://www.python.org/ftp/python/${PYTHON_VERSION}/${TARBALL}"
tar -xzf "${TARBALL}"
cd "Python-${PYTHON_VERSION}"

# Build and install Python with optimizations
echo "Configuring Python build..."
./configure \
    --prefix="${INSTALL_PREFIX}" \
    --enable-optimizations \
    --with-ensurepip=install \
    --enable-shared \
    --with-system-expat \
    --with-system-ffi \
    --enable-ipv6

echo "Compiling Python (this may take a while)..."
make -j "$(nproc)"

echo "Installing Python system-wide..."
make altinstall

# Precompile standard library
echo "Precompiling Python standard library..."
PYTHON_BINARY="${INSTALL_PREFIX}/bin/python${PYTHON_VERSION%.*}"
"${PYTHON_BINARY}" -m compileall "/usr/local/lib/python${PYTHON_VERSION%.*}"

# Update shared library cache
ldconfig >/dev/null 2>&1 || true

# Verify installation
echo "Python installation complete:"
"${PYTHON_BINARY}" --version

echo -e "\nAdd Python to your current session's PATH with:"
echo "export PATH=${INSTALL_PREFIX}/bin:\$PATH"
echo -e "\nFor permanent access, add the above line to your shell profile"

# Cleanup
rm -rf "${SOURCE_DIR}"
```

2. install curses(on windows only)

```bash
pip install windows-curses
```

3. install tuilib

on windows and macos

```bash
pip install tuilib
```

on linux
```bash
python3 -m venv e
```
replace e with the path to your venv

then run

```bash
source e
```
once again replace e with the path to your venv

then run:

```bash
python3 -m pip install -U tuilib 
```

4. make sure tuilib in one of the following directories(folders):

\<where your python path is located\>\\lib\\site-packages\\tuilib or \<where your python path is\>\\lib\\tuilib

5. import it in your python script
```python
import tuilib
```
alternatively scince it only has 1 class
```python
from tuilib import tui
```

6. create your python list of items

example
```python
mylist = ["item 1","item 2","item 3","item 4","item 5"]
```

7. define a function
```python
def myfunc():
   ...
```

8. use the built in list_selector method

```python
def myfunc(stdscr):
   tui.list_selector(stdscr,mylist)
```

9. (optional: if you want to store this information) assign a variable to the output

```python
x = tui.list_selector(stdscr,mylist)
```

10. call the
```python
tui.main(arg1:function)
```

method on your function

11. only if your terminal isn't in the folder where your script is:

```bash
cd ~/scripts/myscript
```

replace ~/scripts/myscript with your actual script path

on windows that would be:

```bash
cd C:/users/<username>/myscript
```

replace \<username> with your username
or if your file isn't in the directory go to the current one

12. run the script!

```bash
python3 filename.py
```

where filename.py is your actual filename

completed example code:

```python
from tuilib import tui
mylist = ["item 1","item 2","item 3","item 4","item 5"]
def myfunc(stdscr):
   option = tui.list_selector(stdscr,mylist)
tui.main(myfunc)
```

## note


as far as I'm aware python dosn't support chromeos so you have to enable the linux dev enviroment in settings > linux


click turn on

for a more detailed instruction on how to enable linux on chromeos and chromebooks check out this video gemini recommended:

[video](https://www.youtube.com/watch?v=LY11WTKE6m0&t=1)
