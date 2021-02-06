---
title: Writing a password cracker using python
date: 2020-12-21 14:22:11 GMT+2
categories: [Development, Pentesting Tools]
tags: [ethical hacking, python]     # TAG names should always be lowercase
image: /assets/img/posts/passwordcracker/passwordcracker.jpg
---

## What is password cracking ?
Password cracking refers to various measures used to discover computer passwords. This is usually accomplished by recovering passwords from data stored in, or transported from, a computer system. Password cracking is done by either repeatedly guessing the password, usually through a computer algorithm in which the computer tries numerous combinations until the password is successfully discovered. Hackers use password cracking in order to gain unauthorized access to a computer without the computer owner’s awareness.

## Writing the script
### Step 1: Importing required modules
The Python hashlib module is an interface for hashing messages easily. This contains numerous methods which will handle hashing any raw message in an encrypted format. The core purpose of this module is to use a hash function on a string, and encrypt it so that it is very difficult to decrypt it.

```python
#Python code snippet
import hashlib
```

### Step 2: Asking user for input
When the user run the program, we ask him to enter the required inputs which are type_of_hash, file_path, and hash_to_decrypt. Based on those specified values, the program will use the appropriate hash function to execute. Also, we check if the specified file/path is correct or not. If not correct, then we terminate the program. Otherwise, the program proceeds in execution.

```python
#Python code snippet
type_of_hash = str(input('Which type of hash you want to bruteforce ? '))
file_path = str(input('Enter path to the file to bruteforce with: '))
hash_to_decrypt = str(input('Enter hash value to bruteforce: '))

if os.path.exists(file_path) == False:
    print('[!!] That File/Path Doesnt Exist')
    exit(1)
```

### Step 3: Try cracking the hash
We open the specified file then we check the hash type. Based on the hash type, we use the appropriate hash function to crack the hash, whether the hash type is md5, sha1, sha256, or sha512. If the hash type is not the one of the specified hash types, then the program will be terminated. The cracking process is done by iterating over the specified file and hashing each plain text in the file with the hash function that the user specified then we compare each hashed plain text in the file to the specified hash. If we found a match, then we print the plain text password. Otherwise, we print that the password is not in file.

```python
#Python code snippet
with open(file_path, 'r') as file:
    for line in file.readlines():
        if type_of_hash == 'md5':
            hash_object = hashlib.md5(line.strip().encode())
            hashed_word = hash_object.hexdigest()
            if hashed_word == hash_to_decrypt:
                print('Found MD5 Password: ' + line.strip())
                exit(0)

        elif type_of_hash == 'sha1':
            hash_object = hashlib.sha1(line.strip().encode())
            hashed_word = hash_object.hexdigest()
            if hashed_word == hash_to_decrypt:
                print('Found SHA1 Password: ' + line.strip())
                exit(0)

        elif type_of_hash == 'sha256':
            hash_object = hashlib.sha256(line.strip().encode())
            hashed_word = hash_object.hexdigest()
            if hashed_word == hash_to_decrypt:
                print('Found SHA256 Password: ' + line.strip())
                exit(0)

        elif type_of_hash == 'sha512':
            hash_object = hashlib.sha512(line.strip().encode())
            hashed_word = hash_object.hexdigest()
            if hashed_word == hash_to_decrypt:
                print('Found SHA512 Password: ' + line.strip())
                exit(0)

        else:
            print('[!!] Type of Hash is Incorrect.')
            exit(1)

    print('Password Is Not In File.')
```

## The complete script
After assembling all the steps, here is the script.

```python
import hashlib, os

type_of_hash = str(input('Enter type of hash you want to bruteforce (md5, sha1, sha256, sha512): '))
file_path = str(input('Enter path to the file to bruteforce with: '))
hash_to_decrypt = str(input('Enter hash value to bruteforce: '))

if os.path.exists(file_path) == False:
    print('[!!] That File/Path Doesnt Exist')
    exit(1)

with open(file_path, 'r') as file:
    for line in file.readlines():
        if type_of_hash == 'md5':
            hash_object = hashlib.md5(line.strip().encode())
            hashed_word = hash_object.hexdigest()
            if hashed_word == hash_to_decrypt:
                print('Found MD5 Password: ' + line.strip())
                exit(0)

        elif type_of_hash == 'sha1':
            hash_object = hashlib.sha1(line.strip().encode())
            hashed_word = hash_object.hexdigest()
            if hashed_word == hash_to_decrypt:
                print('Found SHA1 Password: ' + line.strip())
                exit(0)

        elif type_of_hash == 'sha256':
            hash_object = hashlib.sha256(line.strip().encode())
            hashed_word = hash_object.hexdigest()
            if hashed_word == hash_to_decrypt:
                print('Found SHA256 Password: ' + line.strip())
                exit(0)

        elif type_of_hash == 'sha512':
            hash_object = hashlib.sha512(line.strip().encode())
            hashed_word = hash_object.hexdigest()
            if hashed_word == hash_to_decrypt:
                print('Found SHA512 Password: ' + line.strip())
                exit(0)

        else:
            print('[!!] Type of Hash is Incorrect.')
            exit(1)

    print('Password Is Not In File.')
```

## Script output
We tested the script over all the specified hash types and it is worked well. As well as, here is the script result when we tested it on the MD5 hash.

![script_output](/assets/img/posts/passwordcracker/script_output.png)
