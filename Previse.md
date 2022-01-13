# Box : Previse Writeup
Previse is one of challenge on HTB that already very long. It also categorized as easy. In the end more than 14K Players have solved the user and the root flag and given 30 points.
# Enumeration
First of all I need to scan open ports that Available in the machine Target.\
In this case I am going to use nmap to scan open targets with the commmands
> nmap -sV -O -p- <IP Targets> \
>> -sV = scan services\
>> -O = Scan OS version\
>> -p = Scan ports range , when - is entered , the port ranging from 0 to 65500
> 
After that the results shown like picture below\
![Screenshot1](https://i.imgur.com/mp7F7My.png)\
As you can see, there is ssh and site protocol. To make it easier to work with, I will add the IP from the image to /etc/hosts \
Since I am on HTTP, it means I assume this as website.
&nbsp;
### Directory Busting
Next, I am going to do directory busting on the websites. You can use any tools, from dirbuster,gobuster,ffuf. But in this case, I will use ffuf since it's much faster than dirbuster.\
![Screenshot2](https://i.imgur.com/4RzUNjw.png)\
In this type , I am going to use FFUF with dictionary wordlist from dirbuster/directory-2.3-medium.txt\
There is one thing that interesting which is accounts.php, maybe this is redirect to credential mechanism or something. Turns out , everytime that I visit that, it automatically redirects to login.php. Which means there is some redirections happen here.
# Exploitation
In this part , I am going to do some exploitation. There is two flags, User Flag and System Flag. User Flag is flag that come when you can get into the system. System flag is the flag that when you are successfully set yourself as a root / Administrator.\
Based on HTB Documentation, user flag located in /home/{user}/ \
Meanwhile root flag is located in /root.
## User Flag
### Intercepting connnection
As I know, 302 Requests it means redirections. So I need to change the response header to 200 OK. We am going to use burpsuite for changing the redirection. By going to Match and Replace.\
![Screenshot3](https://i.imgur.com/WGo57gg.png)\
and then I add the rules to change any redirection to OK by add this rules.\
![Screenshot4](https://i.imgur.com/nv68RF9.png)\
After I add those rules, then I can see the real accounts.php. In this section I am seeing account registration that seems show that only the person who have admin can access the website.\
![Screenshot5](https://i.imgur.com/VKhPxlK.png)\
I will try to create one account and suddenly account creation was successful. It means I can turn off site redirections.\
![Screenshot6](https://i.imgur.com/9IiPdvE.png)\
After that I can login to the credential that I already created.
&nbsp;
### Analysing Potential Vulnerability
After exploring the sites , I found something interesting it's called sitesbackup.zip which I think it's a source code of the websites.\
![Screenshot7](https://i.imgur.com/tJToyII.png)\
I analyze the source code and found interesting findings.\
![Screenshot8](https://i.imgur.com/KchJBRd.png)\
It's the SQL Credential, we are going to keep that on a side.\
After digging entire source code, I found there is a potential RCE (Remote Code Execution)\
![Screenshot9](https://i.imgur.com/a5Bdjw3.png)\
The code exec() it means execute and there isn't any sanitation in the code that could lead to RCE. If you wonder, where this is happened. Turns out it happened in delimeter in logging actions.\
![Screenshot10](https://i.imgur.com/0FMyiR3.png)
### Getting RCE by using Reverse Shell
It means I can execute RCE by interecepting the data sent using burpsuite and then before the delimter response is send, I add the Reverse shell.\
![Screenshot11](https://i.imgur.com/XAyG3TM.png)\
and we got the RCE\
![Screnshot12](https://i.imgur.com/P6zNXkz.png)
### Dumping SQL Information
First I am trying to access the SQL server that we find previously, after that we successfully login to the SQL Database server. Before that , I need to see the database that available on the system. 
> show databases;
> 
![Screenshot13](https://i.imgur.com/3mvczqT.png)\
As I can see there are some databases. I am going to see previse database by using this command
> use previse;
>
After that I can go to see the table list that available in the database.\
![Screenshot14](https://i.imgur.com/eo4ia7a.png)\
As you can see there is two tables accounts and files, since we are going to see the acccounts file I am going to execute this command.\
> SELECT * FROM accounts;
>
![Screenshot15](https://i.imgur.com/nGuoOqH.png)
We get the credential but it hashed. So I am going to dehashed the password.
### Dehashing Credential
Based on this [CheatSheet Algorithm Hash](https://hashcat.net/wiki/doku.php?id=example_hashes) I am seeing this as MD5 crypt. So by using hashcat, I am going to guess the password.\
In this case, I am going to use rockyou wordlist since it's listed the most used password.\
After executing using hashcat, I finally got the password for m4lwhere.\
![Screenshot16](https://i.imgur.com/VoVd0BJ.png)
### Catching the User Flag
Based on HTB documentation , the flag for user flag is always in /home/{user}/user.txt \
So I can cd /home and found m4lwhere folder which is the user is m4lwhere. Trying to use the credential that we already dehashed and we successfully logged in.\
![Screenshot17](https://i.imgur.com/dEHI3VE.png)\
after that I Successfullly found the FLAG for user FLAG.\
![Screenshot18](https://i.imgur.com/hWOQ9gr.png)
## System FLAG
### Enumeration for Privelenge Escalation
First of all , I am going to check if there is anything that m4lwhere available to run as sudo by typing sudo -l.\
![Screenshot19](https://i.imgur.com/XSefgu1.png)\
This file is not supposed to be run as root. Let's see what is contain from access_backup.sh\
![Screenshot20](https://i.imgur.com/mvGwo8p.png)\
As I can see, it's executing gzip. This is bad practice as you should not executing based on the process name but based on the file location.\
We can achieve this by creating fake gzip that create reverse shell (again) and then connect to my second reverse shell.
### Privelege Escalation
First I create reverse shell program that executing reverse shell. After that , set chmod +x to the gzip that I already create. After that, create Environment path by executing export PATH=$(pwd):$PATH\
![Screenshot21](https://i.imgur.com/RkUQtrG.png)
### Getting the flag
After that , I executing the access_backup.sh with sudo in the same folder with gzip. Don't forget to prepare the netcat for listener to the reverse shell.\
![Screenshot22](https://i.imgur.com/d67Gaso.png)\
Access Obtained!\
![Screenshot23](https://i.imgur.com/JnnkCTK.png)\
After that we are go to /root and we find the FLAG! SOLVED!\
![Screenshot24](https://i.imgur.com/21CDAI5.png)
