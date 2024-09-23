## WRITEUP DE LA MAQUINA BORAZUWARAHCTF DE DOCKERLABS EN ESPAÑOL

Sitio web - https://dockerlabs.es/#/

Hacemos un escaneo de puertos mediante la herramienta nmap, esto casi siempre es el comienzo para la resolucion de una maquina o CTF.

```shell
└─$ nmap -sV -sC 172.17.0.2 -oN outputscan.txt
```
"-sC" Muestra la versión del servicio del puerto abierto.
"-sV" Ejecuta scripts propios de Nmap que, entre otras cosas, descubren información detallada sobre los servicios.
"172.17.0.2" Es la dirección IP de la máquina víctima.
"-oN outputscan.txt" Sirve para que te copie el resultado del escaneo en el fichero que tú le indiques (no hay que crearlo antes, lo crea solo). Esto lo hago por si más tarde quiero volver a ver el escaneo pero no quiero tener que hacerlo de nuevo.

En este caso no he utlizado ni "-p-", ni "-n", ni "--min-rate 5000" ya que me tardaba mucho hacer el escaneo y para algo tan sencillo es mas eficaz y rapido ejecutar un comando mas corto.

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

Observamos que los puertos 21, 22 y 80 de los servicios ftp, ssh, http respectivamente estan abiertos, vamos a tirar por observar la direccion ip en algun buscador ya que el servicio http esta abierto. Vemos la siguiente página:
<br><br>
![foto de la pagina web](paginaweb.png)
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