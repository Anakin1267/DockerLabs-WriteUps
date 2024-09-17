## WRITEUP DE LA MAQUINA FIRSTHACKING DE DOCKERLABS EN ESPAÑOL

Sitio web - https://dockerlabs.es/#/

```shell
└─$ nmap -sC -sV --min-rate 5000 172.17.0.2 -oN outputscan.txt
```
"-sC" Muestra la versión del servicio del puerto abierto.
<br>
"-sV" Ejecuta scripts propios de Nmap que, entre otras cosas, descubren información detallada sobre los servicios.
<br>
"--min-rate 5000" Ajusta la tasa mínima de envío de paquetes a 5000 paquetes por segundo, acelerando el escaneo.
<br>
"172.17.0.2" Es la dirección IP de la máquina víctima.
<br>
"-oN outputscan.txt" Sirve para que te copie el resultado del escaneo en el fichero que tú le indiques (no hay que crearlo antes, lo crea solo). Esto lo hago por si más tarde quiero volver a ver el escaneo pero no quiero tener que hacerlo de nuevo.

En este caso no he utlizado el -p- y el -n ya que me tardaba mucho hacer el escaneo y para algo tan sencillo es mas eficaz y rapido ejecutar un comando mas corto.

```shell
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-09-17 11:12 EDT
Nmap scan report for 172.17.0.2
Host is up (0.000099s latency).
Not shown: 999 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 2.3.4
Service Info: OS: Unix

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 1.35 seconds
```
Vemos que el puerto 21 del servicio ftp está abierto, en este caso nos vamos a fijar en la versión de dicho servicio (vsftpd 2.3.4). Para explotar esta versión nos guiaremos con un repositorio de github muy bien explicado del usuario " Hellsender01", aqui la url: https://github.com/Hellsender01/vsftpd_2.3.4_Exploit . Aquí encontraremos la explicación de dicha versión del servicio ftp y la vulnerabilidad de este, asi como los requisitos para poder ejecutar el exploit de python y el propio exploit. 

Ejecutarlo es tan sencillo como tener instalada la biblioteca "pwntools" que proporciona herramientas para el desarrollo de exploits y la explotación de vulnerabilidades, para ello ejecutamos el comando:
```shell
sudo python3 -m pip install pwntools
```
"sudo" Permite ejecutar el comando con privilegios de superusuario. En sistemas basados en Unix (como Linux), esto es necesario para instalar paquetes en el sistema global.
<br><br>
"python3 -m pip" Ejecuta pip (el gestor de paquetes para Python) usando la versión de Python 3. 
<br><br>
El flag "-m" permite ejecutar un módulo como un script.
<br><br>
"install" Indica que pip debe instalar el paquete.
<br><br>
"pwntools" Es el nombre del paquete que quieres instalar.
<br>
Ahora con los requerimientos listos, toca descargar el exploit y un par de comandos mas (explicados más adelante)

```shell
git clone https://github.com/Hellsender01/vsftpd_2.3.4_Exploit.git
cd vsftpd_2.3.4_Exploit/
chmod +x exploit.py
```
"git clone https://github.com/Hellsender01/vsftpd_2.3.4_Exploit.git" Clona (es decir, crea una copia) de un repositorio de Git en tu máquina local. El repositorio se encuentra en la URL que proporcionas.
<br><br>
"cd vsftpd_2.3.4_Exploit/" Cambia el directorio actual a la carpeta vsftpd_2.3.4_Exploit/, que es el directorio creado por el comando git clone.
<br><br>
"chmod +x exploit.py" El comando chmod (change mode) modifica los permisos del fichero o directorio que indiques, si añades un "+x" despues del chmod conseguiras dar permisos de ejecucion tanto para el propietario, grupo y otros.
<br><br>
Ahora solo falta ejecutarlo para conseguir ser root, para ello escribimos:
```shell
sudo python3 exploit.py 172.17.0.2 
```
"sudo" permite ejecutar comandos con privilegios de superusuario.
<br>
"python3" ejecuta python y en concreto la version python3.
<br>
"exploit.py" indica el archivo python (.py) que se va a ejecutar.
<br>
"171.17.0.2" es la ip de la maquina víctima.

```shell
[+] Got Shell!!!
[+] Opening connection to 172.17.0.2 on port 21: Done
[*] Closed connection to 172.17.0.2 port 21
[+] Opening connection to 172.17.0.2 on port 6200: Done
[*] Switching to interactive mode
$ whoami
root
```
Somos root! muchas gracias por seguir este writeup!