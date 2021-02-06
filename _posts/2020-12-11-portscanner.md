---
title: Writing a port scanner using python
date: 2020-12-11 01:08:05 GMT+2
categories: [Development, Pentesting Tools]
tags: [ethical hacking, python]     # TAG names should always be lowercase
image: /assets/img/posts/portscanner/portscanner.png
---

## What is port scanning ?
Briefly, Port Scanning is a technique helps the attackers to determine which ports on a network are open and discover services they can exploit to break into a computer system.

## How does port scanner work ?
Port scanner sends requests to every port, asking to connect to a network. The Port Scanner then analyzes responses and classify ports into three categories:
* Open: The host responds, announcing it is listening and open to requests.
* Closed: The host responds, but notes there is no application listening.
* Filtered: The host does not respond to a request due to the packet was dropped as a result of congestion or a firewall.

## Writing the script
### Step 1: Import required modules
1. Socket: Allows us to establish the connection with our target.
2. IPy: Converts the hostname to the right ip address format.
```python
# Python code snippet
import socket
from IPy import IP
```

### Step 2: Asking user for input
We ask the user to enter the target or multiple targets and the port number to be scanned. Then we check in the case of multiple targets, we split the targets and pass them one by one and the port number to the scan function. Else, we pass that only one target and the port number to the scan function.
```python
# Python code snippet
if __name__ == "__main__":
    targets = input('[+] Enter Target/s To Scan (split multiple targets with ,): ')
    port_number = int(input('[+] Enter Number Of Ports You Want To Scan: '))
    if ',' in targets:
        for ip_add in targets.split(','):
            scan(ip_add.strip(' '), port_number)
    else:
        scan(targets, port_number)
```

### Step 3: Implementing required functions
### scan() function
scan function takes two parameters "target" and "port_num". Then it passes the target parameter to check_ip function to handle the case of if the user entered hostname, then we convert it to the right ip address format. After that we iterate over the user specified port numbers and pass the converted_ip and port to the scan_port function.
```python
# Python code snippet
def scan(target, port_num):
    converted_ip = check_ip(target)
    print('\n' + '[Scanning Target...] ' + str(target))
    for port in range(1, port_num):
        scan_port(converted_ip, port)
```

### check_ip() function
check_ip function takes one parameter "ip". Then checks if the user entered the target in the right ip address format, then return the ip back. Else if the user entered the target as a hostname, then it converts the hostname to the right ip address format and return it back.
```python
# Python code snippet
def check_ip(ip):
    try:
        IP(ip)
        return(ip)
    except ValueError:
        return socket.gethostbyname(ip)
```

### get_banner() function
get_banner function grabs the banner by receiving data from the open ports to tell us what service is running over the open ports. And we do not need more than 1024 bytes to grab the banner.
```python
# Python code snippet
def get_banner(s):
    return s.recv(1024)
```

### scan_port() function
scan_port function takes two parameters "ipaddress" and "port". It establishes the connection with our target by sending requests to every port and analyzing responses and then print the open ports with the banners if successed to find the banners or just print the open port without the banners if failed to find the banners. We set timeout of 0.5 to speed up our scan process. But keep in mind, the accuracy of the scan depends on the amount of the timer you set. The lower timeout, the lower accuracy. The higher timeout, the higher accuracy. And at the end of the function we pass, as we do not really interested in closed or filtered ports.
```python
# Python code snippet
def scan_port(ipaddress, port):
    try:
        sock = socket.socket()
        sock.settimeout(0.5)
        sock.connect((ipaddress, port))
        try:
            banner = get_banner(sock)
            print('[+] Open Port ' + str(port) + ' : ' + str(banner.decode().strip('\n').strip('\r')))
        except:
            print('[+] Open Port ' + str(port))
    except:
        pass
```

## The complete script
After assembling all the parts together, here is our complete port scanner script.
```python
import socket
from IPy import IP

def scan(target, port_num):
    converted_ip = check_ip(target)
    print('\n' + '[Scanning Target...] ' + str(target))
    for port in range(1, port_num):
        scan_port(converted_ip, port)

def get_banner(s):
    return s.recv(1024)

def check_ip(ip):
    try:
        IP(ip)
        return(ip)
    except ValueError:
        return socket.gethostbyname(ip)

def scan_port(ipaddress, port):
    try:
        sock = socket.socket()
        sock.settimeout(0.5)
        sock.connect((ipaddress, port))
        try:
            banner = get_banner(sock)
            print('[+] Open Port ' + str(port) + ' : ' + str(banner.decode().strip('\n').strip('\r')))
        except:
            print('[+] Open Port ' + str(port))
    except:
        pass


if __name__ == "__main__":
    targets = input('[+] Enter Target/s To Scan (split multiple targets with ,): ')
    port_number = int(input('[+] Enter Number Of Ports You Want To Scan: '))
    if ',' in targets:
        for ip_add in targets.split(','):
            scan(ip_add.strip(' '), port_number)
    else:
        scan(targets, port_number)
```

## Script output
Our testing setup is built using virtualbox that contains two virtual machines (Kali Linux, Metasploitable). We launch our two VMs, then we execute our script in the kali linux machine to scan the open ports and discover the services that are running on the open ports of the metasploitable machine. And here is our script results.

![portscanner_output](/assets/img/posts/portscanner/portscanner_output.png)

As shown in the image above, our script prints the open ports with the banners that the port scanner successed to find and prints the open ports without the banners that the port scanner failed to find.

