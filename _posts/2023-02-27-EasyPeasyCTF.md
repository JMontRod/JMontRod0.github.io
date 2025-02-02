---
layout: article
title: EasyPeasyCTF
mode: immersive
tags: TryHackMe
aside:
  toc: true
article_header:
  type: cover
  image:
    src: /EasyPeasy.jpg
---

Esta es una máquina de tryHackMe
<!--more-->
---


# Puertos
Para sacar los puertos he utilizado nmap con el siguiente comando
~~~
sudo nmap -sV -T5 -p- 10.10.154.97
~~~

Esto me ha dado los siguientes resultados
~~~
80/tcp    open  http    nginx 1.16.1
6498/tcp  open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
65524/tcp open  http    Apache httpd 2.4.43 ((Ubuntu))
~~~

Con lo cual ya sabemos que hay 3 servicios abiertos

## 1ºServicio
En este servicio accedemos a el utilizando la ip del servidor, la cual hemos utilizado antes para hacer el nmap
Lo unico que se encuentra aqui es un nginx, pero sin nada configurado, así que para buscar mas directorios, he decidido usar gobuster para hacerle Fuzzing
~~~
gobuster dir -u 10.10.154.97 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
~~~

Con este comando el primer directorio que nos encuentra es /hidden
~~~
/hidden               (Status: 301) [Size: 169] [--> http://10.10.154.97/hidden/]
~~~    

No ha encontrado ningun otro directorio pero a su vez he realizado otro gobuster del directorio que me ha encontrado
~~~
gobuster dir -u 10.10.154.97/hidden -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
~~~

El directorio que nos encuentra esta vez es /whatever
~~~
/whatever             (Status: 301) [Size: 169] [--> http://10.10.154.97/hidden/whatever/]      
~~~


Una vez dentro de este directorio en el inspector de codigo podemos ver que hay un texto cifrado en Base64
~~~
ZmxhZ3tmMXJzN19mbDRnfQ==
~~~

Lo decodifico utilizando el comando base64
~~~
echo "ZmxhZ3tmMXJzN19mbDRnfQ==" | base64 -d > 1ºFlag
~~~

Y veo el resultado con cat
~~~
cat 1ºFlag 
XXXX{XXXX}
~~~

Aqui acaba el 1º servicio.
## 2ºServicio
Para entrar en este servicio nos dirigimos a
~~~
http://10.10.154.97:65524/
~~~

Mirando el inspector y buscando flag, encontramos la tercera flag           
~~~              
Fl4g 3 : XXX{XXXXXX}
~~~  

Aqui al mirar en robots.txt
~~~
http://10.10.154.97:65524/robots.txt 
~~~

Encontramos otro texto cifrado
~~~
a18672860d0510e5ab6699730763b250
~~~

Para poder ver en que esta cifrado utilizamos hash-identifier
~~~
hash-identifier a18672860d0510e5ab6699730763b250
Possible Hashs:
[+] MD5
~~~

Asi que decodificamos md5 en la siguiente pagina
https://md5hashing.net/hash/md5/

Y nos da que el texto descifrado que era este
~~~
XXXX{XXXXX}
~~~

Volvemos a
~~~
http://10.10.154.97:65524/
~~~
Y aqui en el inspector al buscar hidden podemos ver que hay otro texto cifrado

~~~
<p hidden="">its encoded with ba....:ObsJmP173N2X6dOrAgEAL0Vu</p>
~~~

El cual es un codigo en base62, al descifrarlo obtenemos lo siguiente

~~~
/XXXXXXX
~~~

El cual es nuestro siguiente directorio a mirar, en este al mirar en el inspector vemos el siguiente texto cifrado

~~~
940d71e8655ac41efb5f8ab850668505b86dd64186a66e57d1483e7f5fe6fd81
~~~

Este texto lo podemos descifrar con el siguiente comando utilizando el diccionario proporcionado por la maquina

~~~
echo “940d71e8655ac41efb5f8ab850668505b86dd64186a66e57d1483e7f5fe6fd81” > hash.txt
~~~

Utilizando john para descifrarla

~~~
john --wordlist=Downloads/easypeasy.txt --format=GOST hash.txt
~~~

Esto nos da una contraseña

~~~
XXXXXXX
~~~
Ahora sacamos la imagen que hay en esta url

~~~
http://10.10.154.97:65524/XXXXX/
~~~
Y le hacemos un steghide para comprobar si tiene algo

~~~
steghide --extract -sf binarycodepixabay.jpg
~~~
Nos pedira una contraseña y aqui ponemos la contraseña conseguida anteriormente y nos pondra lo siguiente

~~~
username:boring
password:
01101001 01100011 01101111 01101110 01110110 01100101 01110010 01110100 01100101 01100100 01101101 01111001 01110000 01100001 01110011 01110011 01110111 01101111 01110010 01100100 01110100 01101111 01100010 01101001 01101110 01100001 01110010 01111001
~~~

La contraseña esta vez esta en binario por lo que la desciframos en la siguiente pagina https://cryptii.com/pipes/binary-decoder y nos da la siguiente contraseña

~~~
XXXXXXX
~~~

Aqui acaba lo que podemos obtener del 2º servicio
## 3º Servicio
Con la contraseña obtenida en el 2º servicio podemos intentar acceder al servidor ssh de la siguiente forma

~~~
ssh boring@10.10.154.97 -p 6498
~~~

Al poner esto escribimos yes y ponemos la contraseña 

~~~
XXXX
~~~

A continuacion nos aparece esto

~~~
You Have 1 Minute Before AC-130 Starts Firing
XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
!!!!!!!!!!!!!!!!!!I WARN YOU !!!!!!!!!!!!!!!!!!!!
boring@kral4-PC:~$ 
~~~

Con esto vemos que ya estamos dentro del ssh y podemos comprobar que hay aqui haciendo un ls

~~~
boring@kral4-PC:~$ ls
user.txt
~~~

Despues le hacemos un cat para ver el contenido

~~~
boring@kral4-PC:~$ cat user.txt
 User Flag But It Seems Wrong Like Its Rotated Or Something
 XXXX{XXXX}
~~~ 



Como se puede ver la flag esta cifrada, en este caso con un ROT13 y con esta pagina lo sacamos https://brianur.info/cifrado-caesar/, dandonos la siguiente flag

~~~
XXXX{XXXX}
~~~

Y aqui acaba por ahora esta maquina


