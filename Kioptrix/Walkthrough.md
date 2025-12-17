We start our Pentest with scanning locally using any of both methods:

`sudo arp-scan -l`
<img width="1374" height="436" alt="527826540-1ef5e4bf-421a-4062-9af3-dc5663a7822d" src="https://github.com/user-attachments/assets/c3c72070-27e2-46b5-b495-84e56403ec6a" />

`sudo netdiscover -i eth0 -r 10.0.2.0/24`
<img width="1181" height="316" alt="527826606-82e7622e-7954-46f1-b6b8-51d723a975f2" src="https://github.com/user-attachments/assets/dfb316f9-05d0-49d0-a786-a89de26c3fe1" />

the first 3 IPs are related to My Personal VM, so now we know that the victim's IP is
`10.0.2.5`

Let's Scan the IP we found using `nmap`

`sudo nmap -T4 -A 10.0.2.5`
<img width="1069" height="962" alt="Pasted image 20250918110527" src="https://github.com/user-attachments/assets/fd160d33-9fe8-4790-8f42-f8deef82b17e" />


Notice many services are shown:
`ssh | http | rcpbind | netbios-ssn | ssl/https`

Lets search the version of each service used respectively.
1) `ssh | OpenSSH 2.9p2 (protocol 1.99)`
[Exploit-DB](https://www.exploit-db.com/exploits/21402) | [Rapid7](https://www.rapid7.com/db/modules/exploit/multi/ssh/sshexec/)

It seems that it needs a password to authenticate so it might be hard to exploit, Let's keep it aside and try to exploit the machine using the next service

2) `http | Apache httpd 1.3.20 ((Unix)  (Red-Hat/Linux) mod_ssl/2.8.4 OpenSSL/0.9.6b)`

search for exploits using `searchsploit`

`searchsploit apache 1.3.20`
<img width="2536" height="736" alt="Pasted image 20250918112035" src="https://github.com/user-attachments/assets/c37ca505-8837-431c-8b82-770834b45805" />

Here, we can see that this version of Apache is vulnerable to `OpenFuckV2`. So, we can copy that code, compile it and try to gain access to the machine.

Bear in mind that the Kernel used in the victim's machine was visible in the scan
`Redhat/Linux 1.3.20`

after compiling the exploit we now want to run it
<img width="452" height="247" alt="Pasted image 20250918113603" src="https://github.com/user-attachments/assets/754a7419-f28c-4827-bc94-44076c8b2299" />

`./z 0x6b 10.0.2.5 -c 10`

We're In!

<img width="1057" height="857" alt="Pasted image 20250918113831" src="https://github.com/user-attachments/assets/c81df113-d22e-4970-a2e1-edb8009b7de1" />
<img width="227" height="158" alt="Pasted image 20250921235829" src="https://github.com/user-attachments/assets/f8237c3b-a243-4bcc-a2b6-62181f4dbddc" />

if you have noticed, i can access the machine, but as a non-root user : `apache`
and this is because of the error in the first pic after `./z 0x6b 10.0.2.5 -c 10`

this is because its trying to access an essential file for the exploit, but since the machine cant connect to the internet, it can't download it 
<img width="1053" height="274" alt="Pasted image 20250922000149" src="https://github.com/user-attachments/assets/88c3ae72-3e30-44de-9a87-73de6f53d546" />

the file named `ptrace-kmod.c`
so i downloaded it locally on my kali VM, started my Apache server and modified a line in the exploit code, that it instead of downloading this file from the server on the internet, it simply downloads it locally from my machine

compile the file after saving, try running it
<img width="1167" height="1065" alt="Pasted image 20250922000421" src="https://github.com/user-attachments/assets/a8aff65f-6c58-43e9-a5c6-e5918b0a49b9" />

ROOT! 🥳


3) `netbio-ssn | Samba smbd`
Search for exploits, but first we need to detect the smb version to be able to do that. this can be done using Metasploit.

<img width="1543" height="585" alt="Pasted image 20250918114722" src="https://github.com/user-attachments/assets/cf10263c-9636-43c4-bb69-347d05c4ddfd" />

search `Samba 2.2.1a` for exploits

[Exploit-DB](https://www.exploit-db.com/exploits/10) | [Rapid7](https://www.rapid7.com/db/modules/exploit/linux/samba/trans2open/)

let's use the Rapid7 method
<img width="1287" height="1031" alt="Pasted image 20250918120547" src="https://github.com/user-attachments/assets/9a33de23-b7e6-4b8d-a4a4-0291f72a3892" />

And We're In as root!
