## WRITEUP DE LA MÁQUINA INJECTION DE DOCKERLABS EN ESPAÑOL

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

```
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-09-19 03:06 EDT
Nmap scan report for 172.17.0.2
Host is up (0.00028s latency).
Not shown: 65533 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.6 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 72:1f:e1:92:70:3f:21:a2:0a:c6:a6:0e:b8:a2:aa:d5 (ECDSA)
|_  256 8f:3a:cd:fc:03:26:ad:49:4a:6c:a1:89:39:f9:7c:22 (ED25519)
80/tcp open  http    Apache httpd 2.4.52 ((Ubuntu))
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
|_http-title: Iniciar Sesi\xC3\xB3n
|_http-server-header: Apache/2.4.52 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 15.68 seconds
```

Observamos que el puerto 22 y 80 (ssh y http respectivamente) estan abiertos, primero vamos a ver que página web encontramos con 172.17.0.2, vemos un login:

![loginsql](https://github.com/user-attachments/assets/8d5f5080-24af-45a7-9c48-6dc7326d7e37)

Probamos a intentar loguearnos con algun usuario o contraseña y nos dice "credenciales erroneas". Vamos a hacer un poco de SQL injection, para ello vamos a hacer una Inyección tautológica. Este tipo de ataque se aprovecha de una expresión condicional que siempre es verdadera, en este caso 1=1, lo que permite a un atacante acceder o manipular la base de datos de una aplicación.

```SQL
' or 1=1 -- -
```

" ' " cierra una cadena de texto que el usuario está ingresando. En este caso esa "comilla" cierra la cadena de login que esta esperando, espera que pongas un nombre, pero con " ' " cierras esa parte de la consulta.
<br><br>
" or 1=1 " es una condición que siempre es verdadera, forzando a que toda la consulta SQL retorne un resultado verdadero.
<br><br>
" -- " es un comentario en SQL que indica que lo que sigue en la línea no será ejecutado, anulando el resto de la consulta original.

" - " el segundo guión adicional es porque algunos motores de SQL no aceptan como comentario un -- sin nada despues. Para enetenerlo se podría hacer lo mismo poniendo cualquier otro caracter, por ejemplo " ' or 1=1 -- x" y funcionaria igual.

Si no lo has entendido, aquí estaría la consulta en SQl como lo estaría interpretando la página de login.

```SQL
SELECT * FROM usuarios WHERE usuario = '' OR 1=1 -- ' AND contraseña = '';
```
Haciendo esa inyección tautológica conseguimos entrar, y observamos esta página:

![entramossql](https://github.com/user-attachments/assets/a12b530b-9a69-4906-b666-11bd5cd679c9)

Sabiendo el usuario (dylan) y la contraseña (KJSDFG789FGSDF78) podemos hacer la conexión ssh.

```
└─$ sudo ssh dylan@172.17.0.2
```
Ahora como siempre toca escalar privilegios, peeeero, esta vez ```sudo -l ``` no funciona, nos dice que no se encuentra el comando, hay otra forma de buscar los binarios para intentar escalar privilegios. Mediante el comando:
``` find / -perm -4000 2>/dev/null ```, lo desgloso:

"find "Este es el comando principal que se utiliza para buscar archivos y directorios en una jerarquía de directorios.
<br><br>
"/" Este es el directorio donde comenzará la búsqueda. En este caso, se está buscando desde la raíz del sistema de archivos, lo que significa que se revisarán todos los directorios y subdirectorios.
<br><br>
"-perm"
Este es un criterio de búsqueda que especifica que se están buscando archivos con un permiso especial. 
<br><br>
"-4000" se refiere al bit SUID (Set User ID). Cuando un archivo tiene este permiso, se ejecuta con los privilegios del propietario del archivo en lugar de los del usuario que lo ejecuta. Esto puede ser útil, pero también puede ser un riesgo de seguridad si no se gestiona adecuadamente.
<br><br>
"2>/dev/null" Esta parte del comando redirige los mensajes de error (que normalmente se mostrarían en la consola) a
/dev/null que es un "agujero negro" en el sistema donde se descartan los datos. Esto significa que si hay errores (por ejemplo, si no tienes permiso para acceder a ciertos directorios), esos mensajes no se mostrarán en la salida.

Este es el resultado:

```
/usr/bin/umount
/usr/bin/env
/usr/bin/chfn
/usr/bin/su
/usr/bin/newgrp
/usr/bin/passwd
/usr/bin/gpasswd
/usr/bin/chsh
/usr/bin/mount 
```

Que estos binarios tengan el bit SUID activad no significa que sean escalables. El binario "/usr/bin/env" se que es escalable por máquinas anteriores asi que la busco en GTFObins. Y veo que debo ejecutar el siguiente comando:

``` ./env /bin/sh -p ```

``` 
# whoami
root 
```

Ya somos root, muchas gracias por seguir este writeup!
