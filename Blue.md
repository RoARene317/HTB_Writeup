# HackTheBox : Machine - Blue
Blue was a machine in HTB, it's also categorized as easy. In the end more than 27K people solve it and based on the charts , most people say that this problem was a piece of cake.
# How I found this machine
So this machine I found as already retired machine as I tried one of retired machine due to I tried the VIP in HackTheBox.
# Enumeration
Doing some enumeration and found something interesting.\
![Screenshot1](https://i.imgur.com/dB3sd4f.png)\
It's looks like the OS is windows and the system is Windows 7+. From the name of the machine, maybe this machine is about EternalBlue?
# EternalBlue
If you don't know, EternalBlue is a [BufferOverflow](https://en.wikipedia.org/wiki/Buffer_overflow) exploits that infect [Windows XP](https://en.wikipedia.org/wiki/Windows_XP) up to [Windows 10](https://en.wikipedia.org/wiki/Windows_10) that could lead Remote Code Execution (RCE). It first found by [NSA](https://en.wikipedia.org/wiki/National_Security_Agency) and then leaked by one of group hackers.
# Exploitation
In this case we are going to use Metasploit module of [EternalBlue](https://www.rapid7.com/db/modules/exploit/windows/smb/ms17_010_eternalblue/) the exploitation process is really simple as shown in the picture below.\
![Screenshot2](https://i.imgur.com/kCkz0e9.png)\
as you can see the setup is really simple and the exploitation progress is shown below.\
![Screenshot3](https://i.imgur.com/6MIA05P.png)\
We got the shell and now we can grab the flag\
![Screenshot4](https://i.imgur.com/g7jnOQV.png)\
![Screenshot5](https://i.imgur.com/z6tNbBe.png)\
I got both the flag really easy. SOLVED
