# picoCTF 2018 Writeup
Greetings from Team Point Mass!
This is our writeup for picoCTF 2018 competition. Over the next few weeks, we will put solutions to the problems in this repository.

## Notes
### Flag and URL Differences
(Almost) all of the problems will have differening flags or URLs, which are randomly chosen by picoCTF when you register, presumably as a mechanism to prevent cheating. We will not show links in our writeups to avoid confusion; and in scripts, we will put dummy variables for the hostname and port, allowing the reader to fill them in. For flags that may differ, all of the variations of the flag have a general form. For example:

> `picoCTF{3x@mp1e_fl4g_48375435}`  
> `picoCTF{3x@mp1e_fl4g_19584647}`  
> `picoCTF{3x@mp1e_fl4g_34583985}`  
> `picoCTF{3x@mp1e_fl4g_85479822}`

In this case, we will write the flag as such:

> `picoCTF{3x@mp1e_fl4g_########}`

## Problem List

Name | Points | Category | Status
--- | :---: | :---: | :---:
[Forensics Warmup 1](problems/forensics-warmup-1/forensics-warmup-1.md) | 50 | Forensics | Complete
[Forensics Warmup 2](problems/forensics-warmup-2/forensics-warmup-2.md) | 50 | Forensics |  Complete
[General Warmup 1](problems/general-warmup-1/general-warmup-1.md) | 50 | General | Complete
[General Warmup 2](problems/general-warmup-2/general-warmup-2.md) | 50 | General | Complete
[General Warmup 3](problems/general-warmup-3/general-warmup-3.md) | 50 | General | Complete
[Resources](problems/resources/resources.md) | 50 | Complete
Reversing Warmup 1 | 50 | Reversing | Pending
Reversing Warmup 2 | 50 | Reversing | Pending
Crypto Warmup 1 | 75 | Cryptography | Pending
Crypto Warmup 2 | 75 | Cryptography | Pending
grep 1 | 75 | General | Pending
net cat | 75 | General | Pending
HEEEEEEERE'S Johnny! | 100 | Cryptography | Pending
strings | 100 | General | Pending
pipe | 110 | General | Pending
Inspect Me | 125 | Web Exploitation | Pending
grep 2 | 125 | General | Pending
Aca-Shell-A | 150 | General | Pending
Client Side is Still Bad | 150 | Web Exploitation | Pending
Desrouleaux | 150 |  Forensics | Pending
Logon | 150 | Web Exploitation | Pending
Reading Between the Eyes | 150 | Forensics |  Pending
Recovering From the Snap | 150 | Forensics |  Pending
admin panel | 150 |  Forensics | Pending
assembly-0 | 150 | Reversing | Pending
buffer overflow 0 | 150 | Binary Exploitation | Pending
caesar cipher 1 | 150 | Cryptography | Pending
environ | 150 | General | Pending
hertz | 150 | Cryptography | Pending
hex editor | 150 |  Forensics | Pending
ssh-keyz | 150 | General | Pending
Irish Name Repo | 200 | Web Exploitation | Pending
Mr. Robots | 200 | Web Exploitation | Pending
No Login | 200 | Web Exploitation | Pending
Secret Agent | 200 | Web Exploitation | Pending
Truly an Artist | 200 |  Forensics | Pending
assembly-1 | 200 | Reversing | Pending
be-quick-or-be-dead-1 | 200 | Reversing | Pending
blaise's cipher | 200 | Cryptography | Pending
buffer overflow 1 | 200 | Binary Exploitation | Pending
hertz 2 | 200 | Cryptography | Pending
leak-me | 200 | Binary Exploitation | Pending
now you don't | 200 |  Forensics | Pending
quackme | 200 | Reversing | Pending
shellcode | 200 | Binary Exploitation | Pending
what base is this? | 200 | General | Pending
you can't see me | 200 | General | Pending
Buttons | 250 | Web Exploitation | Pending
Ext Super Magic | 250 |  Forensics | Pending
Lying Out | 250 | Forensics | Pending
Safe RSA | 250 | Cryptography | Pending
The Vault | 250 | Web Exploitation | Pending
What's My Name? | 250 |  Forensics | Pending
absolutely relative | 250 | General | Pending
assembly-2 | 250 | Reversing | Pending
buffer overflow 2 | 250 | Binary Exploitation | Pending
caesar cipher 2 | 250 | Cryptography | Pending
got-2-learn-libc | 250 | Binary Exploitation | Pending
rsa-madlibs | 250 | Cryptography | Pending
be-quick-or-be-dead-2 | 275 | Reversing | Pending
in out error | 275 | General | Pending
Artisinal Handcrafted HTTP 3 | 300 | Web Exploitation | Pending
SpyFi | 300 | Cryptography | Pending
echooo | 300 | Binary Exploitation | Pending
learn gdb | 300 | General | Pending
Flaskcards | 350 | Web Exploitation | Pending
Super Safe RSA | 350 | Cryptography | Pending
authenticate | 350 | Binary Exploitation | Pending
be-quick-or-be-dead-3 | 350 | Reversing | Pending
core | 350 |  Forensics | Pending
got-shell? | 350 | Binary Exploitation | Pending
quackme up | 350 | Reversing | Pending
rop chain | 350 | Binary Exploitation | Pending
roulette | 350 | General | Pending
Malware Shops | 400 | Forensics |  Pending
Radix's Terminal | 400 | Reversing | Pending
assembly-3 | 400 | Reversing | Pending
eleCTRic | 400 | Cryptography | Pending
fancy-alive-monitoring | 400 | Web Exploitation | Pending
keygen-me-1 | 400 | Reversing | Pending
store | 400 | General | Pending
Super Safe RSA 2 | 425 | Cryptography | Pending
Magic Padding Oracle | 450 | Cryptography | Pending
buffer overflow 3 | 450 | Binary Exploitation | Pending
Secure Logon | 500 | Web Exploitation | Pending
echo back | 500 | Binary Exploitation | Pending
script me | 500 | General | Pending
LoadSomeBits | 550 | Forensics |  Pending
are you root? | 550 | Binary Exploitation | Pending
assembly-4 | 550 | Reversing | Pending
gps | 550 | Binary Exploitation | Pending
Flaskcards Skeleton Key | 600 | Web Exploitation | Pending
Help Me Reset 2 | 600 | Web Exploitation | Pending
Super Safe RSA 3 | 600 | Cryptography | Pending
special-pw | 600 | Reversing | Pending
A Simple Question | 650 | Web Exploitation | Pending
can-you-gets-me | 650 | Binary Exploitation | Pending
James Brahm Returns | 700 | Cryptography | Pending
freecalc | 750 | Binary Exploitation | Unsolved
keygen-me-2 | 750 | Reversing | Pending
LambDash 3 | 800 | Web Exploitation | Unsolved
circuit123 | 800 | Reversing | Pending
sword | 800 | Binary Exploitation | Pending
Contacts | 850 | Binary Exploitation | Unsolved
Cake | 900 | Binary Exploitation | Unsolved
Dog or Frog | 900 | General |  Pending
Flaskcards and Freedom | 900 | Web Exploitation | Pending
