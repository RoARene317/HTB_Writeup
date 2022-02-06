# HackTheBox: Horizontall
Horizontall is one of challenge that kinda easy to solve. I learn so much in this challenge.

# Enumeration
First of all is important to do any enumeration. In this case I am going to nmap the client by using this syntax.
> nmap -sV -O -p- 10.10.11.105
>> -sV = Scan Services.\
>> -O = Scan operating system.\
>> -p = Scan ports.\
>>

After that I got some interesting results.\
![Screenshot1](https://i.imgur.com/wAOOC2k.png)\
There is port 80 (Apache HTTP) and also openSSH.\
Trying open the website it shows as below.\
![Screenshot2](https://i.imgur.com/dcOhYtk.png)
There is something in the website, so I am going to directory busting the websites. In this case I will use FFUF since it's faster.\
![Screenshot3](https://i.imgur.com/pNkbrjg.png)\
Nothing interesting in here , so I am stuck in here. After asking several person, I should do subdomain busting.\
In this case I will be using gobuster because it's much cleaner and much more readable than dirbuster.\
![Screenshot4](https://i.imgur.com/wJCsniH.png)\
I found one subdomain that related, it's http://api-prod.Horizontall.htb/ \
After that I find a next problem.\
![Screenshot5](https://i.imgur.com/OdnbCtK.png)\
Yeah server not found, this is starting to annoy me a little bit. To solve this , you need to add http://api-prod.Horizontall.htb/ to the host list (/etc/hosts)\
After that found a site that writes welcome and a logo.\
![Screenshot6](https://i.imgur.com/iGYzANA.png)
Now let's do directory busting on http://api-prod.Horizontall.htb/ \
But this time I will be using FeroxBuster. It shows results as below\
![Screenshot7](https://i.imgur.com/oNlcDkh.png)\
The admin one is really interesting , so I visit the links http://api-prod.Horizontall.htb/ and then found that it redirects to strapi CMS. Since the level is easy , possibly there is a Strapi CMS vulnerability.
# Exploitation
Like previous writeup , there is two flags. User flag and root flag.\
User Flag = when you get access to the machine successfully\
Root / System Flag = When you get access to the machine as admin successfully
## User FLAG
For the user FLAG, I am going to use CVE-2019-18818 and CVE-2019-19609. It's basically RCE authenticated and Reset password Vulnerability Unauthenticated.\
First of all I need to reset the password by using CVE-2019-18818.
![Screenshot8](https://i.imgur.com/Wrymej7.png)\
User password reset successfully, keep in mind to save the JWT (JSON Web Token) as this important to do authenticated RCE.\
Next I am going to use CVE-2019-19609 for authenticated RCE.\
Remember to don't forget to execute the netcat as listener to listen any reverse shell.\
![Screenshot9](https://i.imgur.com/FRWRPIl.png)\
Back to the netcat , and I got RCE connection.\
![Screenshot10](https://i.imgur.com/YZNsulE.png)\
Go to the target directory , I can see the User FLAG.\
![Screenshot11](https://i.imgur.com/2b5CtdK.png)\
SOLVED For user FLAG.
## Root FLAG
### Enumeration for Privelege Escalation
For enumerating possible privelege escalation , I am going to use [PEASS-ng](https://github.com/carlospolop/PEASS-ng). In this case I'll download and then serve to the target machine.\
To move the file from my machine to target machine, I need to serve as http server like below\
![Screenshot12](https://i.imgur.com/mTqV1eU.png)\
After that I can wget the file, but before that I need to move the directory to /tmp since it means temporary.\
![Screenshot13](https://i.imgur.com/fu6kjrs.png)\
Actually there is a lot of information that you can dig here, but the one that for me is very interesting is the network statistics.\
![Screenshot14](https://i.imgur.com/Q4SnPfj.png)\
Based on the [Port list](https://en.wikipedia.org/wiki/List_of_TCP_and_UDP_port_numbers) I know that 3306 is MySQL server, 80 is HTTP, 22 is SSH , 1337 is strapi , but what is port 8000?\
Trying to get the informationb by using curl, I know it's Laravel v8 (PHP v7.4.18)\
![Screenshot15](https://i.imgur.com/ZR3Mzqo.png)
### Exploitation
Searching for the exploitation, I found exploit that is pretty new (CVE-2021-3129) and could lead to RCE.\
So first I need to download PHPGGC and the exploit.\
After I download the exploit and the PHPGGC I run the exploit, in short it will be like down below.\
![Screenshot16](https://i.imgur.com/B2IVNLM.png)\
and I got the flag. SOLVED.
