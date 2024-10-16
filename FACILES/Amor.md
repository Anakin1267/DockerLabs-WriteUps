## WRITEUP DE LA MÁQUINA AMOR DE DOCKERLABS EN ESPAÑOL  

Sitio web - https://dockerlabs.es/#/

Hacemos un escaneo de puertos mediante la herramienta nmap, esto casi siempre es el comienzo para la resolución de una máquina o CTF.

`└─$ nmap -sV -sC 172.17.0.2 -oN outputscan.txt`

- **"-sC"** Muestra la versión del servicio del puerto abierto.  

- **"-sV"** Ejecuta scripts propios de Nmap que, entre otras cosas, descubren información detallada sobre los servicios.  

- **"172.18.0.2"** Es la dirección IP de la máquina víctima.  

- **"-oN outputscan.txt"** Sirve para que te copie el resultado del escaneo en el fichero que tú le indiques (no hay que crearlo antes, lo crea solo). Esto lo hago por si más tarde quiero volver a ver el escaneo pero no quiero tener que hacerlo de nuevo.  

> En este caso no he utlizado ni "-p-", ni "-n", ni "--min-rate 5000" ya que para algo tan sencillo es más eficaz y rapido ejecutar un comando más corto.

```
└─$ nmap -sCV --min-rate 5000 172.17.0.2
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-10-15 12:42 EDT
Nmap scan report for 172.17.0.2
Host is up (0.00024s latency).
Not shown: 998 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.6p1 Ubuntu 3ubuntu13 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 7e:72:b6:8b:5f:7c:23:64:dc:15:21:32:5f:ce:40:0a (ECDSA)
|_  256 05:8a:a7:27:0f:88:b9:70:84:ec:6d:33:dc:ce:09:6f (ED25519)
80/tcp open  http    Apache httpd 2.4.58 ((Ubuntu))
|_http-server-header: Apache/2.4.58 (Ubuntu)
|_http-title: SecurSEC S.L
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 13.24 seconds
```

Vemos que el puerto 22 y 80 de los servicios ssh y http respectivamente estan abiertos. Empezaremos por ver que sitio web encontramos con la IP víctima (172.17.0.2):  

![Pagina Web](paginaweb.png)  

He usado gobuster para ver si había algún directorio o archivo oculto en el sitio web, pero no encontramos nada, solo un directorio `/javascript` al cual no podemos acceder porque no tenemos permisos.

Siempre hay que observar a fondo la información que tenemos, en este caso la pagina web contiene mucho texto del cual sacamos nombres clave como `Carlota` y `Juan`, ya tenemos dos posibles víctimas para hacer una conexion ssh.

> Os adelanto que haciendo fuerza bruta solo encontramos la contraseña de carlota, lo digo para ahorraros tiempo.

Vamos a hacer un ataque de fuerza bruta mediante la herramienta `hydra` para intentar encontrar la contraseña de `Carlota`:  
  
`└─$ hydra -l carlota -P /home/kali/Desktop/rockyou.txt ssh://172.17.0.2 -t 16
`  

- **hydra**: Es una herramienta para realizar ataques de fuerza bruta a servicios como SSH, HTTP, FTP, etc.  

- **-l carlota**: (L de *login*) Especifica el nombre de usuario a utilizar en el ataque de fuerza bruta. En este caso, es "carlota".  

- **-P /home/kali/Desktop/rockyou.txt**: (P de *Password list*) Define la ruta al archivo que contiene una lista de posibles contraseñas (en este caso, el archivo `rockyou.txt`) que contiene mas de un millon de contraseñas típicas.  

- **ssh://172.17.0.2**: Indica el protocolo (SSH) y la dirección IP (172.17.0.2) del servidor objetivo.  

- **-t 16**: Establece el número de hilos (threads) que se utilizarán para realizar el ataque, en este caso, 16 hilos en paralelo.  
  
Aquí el resultado:  

```
└─$ hydra -l carlota -P /home/kali/Desktop/rockyou.txt ssh://172.17.0.2 -t 5 
Hydra v9.5 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2024-10-16 10:29:36
[DATA] max 5 tasks per 1 server, overall 5 tasks, 14344399 login tries (l:1/p:14344399), ~2868880 tries per task
[DATA] attacking ssh://172.17.0.2:22/
[22][ssh] host: 172.17.0.2   login: carlota   password: babygirl
1 of 1 target successfully completed, 1 valid password found
```  

<br>

---

<br>

Teniendo ya la contraseña de carlota, accedemos a su usuario mediante conexión ssh ya que el puerto 22 al estar abierto nos lo permite, `ssh carlota@172.17.0.2`.  

> La terminal a la que nos conectamos no ha pasado por el llamado "**_Tratamiento de la TTY_**" y no es del todo funcional.    

Aquí una explicación de que es la TTY, que es hacer el tratamiento de esta, y algún comando sencillo junto a su explicación.  

- **¿Qué significa TTY?:** 

TTY son las siglas de Teletypewriter o teletipo. Hace referencia a una terminal de texto en sistemas UNIX/Linux, que permite interactuar con el sistema operativo a través de comandos. Originalmente, estas terminales estaban conectadas físicamente al sistema.  

- **¿Qué significa hacer el tratamiento de la TTY?:** 

Tratamiento de la TTY significa manejar o "tomar control" de una terminal para asegurar que los comandos que se ejecutan interactúen correctamente con ella. Esto es importante en el pentesting cuando necesitas que tu shell remota se comporte como una terminal completamente funcional  

- **`export term=XTERM:`**  

Le estás diciendo al sistema que te trate como si estuvieras usando una terminal Xterm, lo que puede mejorar la compatibilidad de la sesión interactiva, especialmente en conexiones remotas o shells restringidas.

- **`script /dev/null -c bash:`**  

El comando inicia una nueva sesión de terminal ejecutando un shell Bash dentro de ella, sin guardar un registro de la sesión (porque redirige la salida a /dev/null). Esto se utiliza para obtener una terminal interactiva completa cuando la actual es limitada o no tiene todas las funcionalidades de una TTY.  

<br>

---

<br>

Con estos dos últimos comandos conseguimos una shell mas interactiva e intuitiva.  
Rebuscando un poco por los directorios del usuario **carlota**, encontramos una foto "imagen.jpg", exactamente en esta ruta: `/home/carlota/Desktop/fotos/vacaciones/`.  

En un principio pense en observar los metadatos de la foto con **_exixtool_** (Spoiler: no van por ahí los tiros) pero para eso necesitaba pasarme el archivo de la imágen del usuario carlota a mi usuario local. 

Para ello, gracias a que el servicio ssh está disponible, el protocolo **_SCP_** (Secure Copy Protocol) tambien está disponible.  

> SCP (Secure Copy Protocol) es una herramienta que permite transferir archivos de manera segura entre dos sistemas a través de una red, utilizando el protocolo SSH (Secure Shell) para la autenticación y encriptación de los datos.

`scp usuario@servidor:/ruta/archivo.jpg /ruta/destino/local/` es el comando que vamos a necesitar (Cambiarlo para ajustarlo a vuestras rutas). Es importante saber que este comando hay que ejecutarlo en la sesión de nuestro usuario personal, NO carlota.  

```
└─$ scp carlota@172.17.0.2:/home/carlota/Desktop/fotos/vacaciones/imagen.jpg .
carlota@172.17.0.2's password: 
imagen.jpg                              100%   51KB   5.1MB/s   00:00
```

<br>

---

<br>

Gracias a esta máquina aprendí lo que es la **_"Esteganografía"_** .  

> La esteganografía en hacking es la técnica de ocultar información dentro de archivos aparentemente inofensivos, como imágenes, videos o audio, para transmitir datos sin levantar sospechas, haciéndola útil para evadir detección en ataques o comunicación encubierta.

Y como ya os dije antes, los metadatos no iban a ser los protagonistas hoy. Existe una magnifica herramienta llamada **_steghide_** (Stegnography Hide) la cual nos va a extraer lo que este dentro de la imagen.jpg que ya tenemos en nuestro directorio personal.

`steghide extract -sf imagen.jpg` donde:  

- **_steghide_**: Invoca la herramienta para realizar operaciones de esteganografía.

- **_extract_**: Extrae información secreta de un _Stego file_.

- **_-sf_**: Sirve para indicar que el archivo que escribamos despues es un _stego file_ que no es más que el archivo que contiene la informacion secreta. 

- **_imagen.jpg_** El archivo del cual vamos a extraer la información oculta.  

Nos pedirá una contraseña, no os comais la cabeza, no tiene. Pulsando enter extraerá un fichero llamado `secret.txt`:  

```
└─$ steghide extract -sf imagen.jpg 
Enter passphrase: 
wrote extracted data to "secret.txt".
```

<br>

---

<br>

Con el comando `cat secret.txt` nos muestra el contenido de secret.txt el cual contiene lo que parece un mensaje codificado en **_base64_**. Pero... porque en base64? Aquí algunas razones para considerarlo:  

- Las cadenas codificadas en Base64 suelen ser más largas que los datos originales, ya que el formato aumenta el tamaño al convertir datos binarios a texto.  

- La presencia del símbolo de igual (=) al final de la cadena es un indicador típico de Base64. Se utiliza para el relleno cuando la longitud de los datos no es un múltiplo de 3, ayudando a que la cadena sea de longitud adecuada.

Para desencriptarla ejecutad `echo "ZXNsYWNhc2FkZXBpbnlwb24=" | base64 -d`  

- **_echo_**: Este comando se utiliza en la línea de comandos para imprimir (o mostrar) el texto que le sigas.  

- **_ZXNsYWNhc2FkZXBpbnlwb24=_**: Es el texto que se quiere imprimir. En este caso, es una cadena codificada en Base64.

- **|**: El símbolo | se conoce como pipe. Se utiliza para redirigir la salida de un comando como entrada a otro comando. En este caso el resultado de echo... es la entrada de `base64 -d`  

- **_base64_**: Es un comando que se utiliza para codificar o decodificar datos en Base64.  

- **_-d_**: Es una opción que significa "decode" (decodificar). Indica que deseas decodificar la entrada en lugar de codificarla.  

<br>

Resultado: `eslacasadepinypon`

<br>

---

<br>

Teniendo ya lo que se intuye que es la contraseña de alguno de los usuarios que encontramos en la sesión de carlota, `carlota  oscar  ubuntu`, intentaremos logearnos.  

Nos conectamos como `oscar` con el comando `su oscar` ("su" significa Switch User) y poniendo la contraseña que acabamos de decodificar nos hemos conectado como _Oscar_.  

Toca escalar privilegios (no lo hicimos con Carlota porque no tiene permisos para ejecutar _sudo_), con `sudo -l` lo cual nos lista los binarios que podemos ejecutar con los privilegios de root sin tener la contraseña de este:  

```
$ whoami
oscar
$ sudo -l
Matching Defaults entries for oscar on 88bfc36c261d:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin,
    use_pty

User oscar may run the following commands on 88bfc36c261d:
    (ALL) NOPASSWD: /usr/bin/ruby
$ 
```  

<br>

Como siempre vamos a la página [enlace GTFObins](https://gtfobins.github.io/) y entramos a la opción de **_sudo_** del binario **_ruby_** , debemos ejecutar: `sudo ruby -e 'exec "/bin/sh"'` y ya está...  

<br>

```
$ sudo ruby -e 'exec "/bin/sh"'
# whoami
root
```

<br>

---

<br>

Ya estaría, muchas gracias por seguir este writeup tan extenso y con tanta teoría, adios!! 😊😊