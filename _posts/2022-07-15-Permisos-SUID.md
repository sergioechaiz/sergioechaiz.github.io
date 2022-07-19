---
layout: post
title:  "Permisos SUID"
date:   2022-07-15
excerpt: "Abuso Permisos SUID"
project: true
tag:
- Linux
- Permisos SUID
- Hacking
- Pentesting
- Explotación
comments: true
---
## PERMISOS SUID (Set User ID)

En Linux, SUID(establecer ID de usuario del propietario en la ejecución) es un tipo especial de permiso de archivo otorgado a un archivo. SUID otorga permisos temporales a un usuario para ejecutar el programa/archivo con los permisos del propietario del archivo, en lugar del usuario que lo ejecuta. Es decir, si lo creo root tendremos permisos root.

Por ejemplo, el archivo binario para cambiar su contraseña (/usr/bin/passwd), tiene el bit SUID. Esto se debe a que para cambiar su contraseña, deberá escribir en el archivo shadowers al que no tiene acceso, pero root sí. Por lo que se le otorga temporalmente privilegios de root para realizar los cambios correctos.

La forma de identificar que archivos poseen permisos SUID es mediante una "s" en el primer permiso de ejecucion del propietario.

Para asignar permisos SUID a un archivo deberemos ejecutar los siguientes comandos:

`sudo chmod u+s [Nombre archivo]`

`sudo chmod 4775 [Nombre archivo]`

Una vez asignados los permisos SUID, si listamos el archivo. podemos observar que en la primera casilla de los permisos del propietario aparece una "s" en vez de una "x".

De igual modo podemos quitar los permisos de la siguiente forma:

`sudo chmod u-s [Nombre archivo]`

`sudo chmod 0755 [Nombre archivo]`

Buscar permisos SUID:

`find / -user root -perm -4000 -exec ls -ldb {} \;`

`find / -perm -u=s 2>/dev/null`

    *find /: búsqueda desde la raíz
    *-perm: indicamos en base a que permisos se hará la búsqueda
    *-u=s: indicamos los permisos SUID
    *2>/dev/null: indicamos que los errores los mande al agujero negro en Linux


Una vez conocemos que son y como funcionan los permisos SUID, vamos a ver como en ocasiones, estos permisos pueden ser utilizados y ser explotados para escalar privilegios. Una de las paginas por excelencia para buscar posibles vectores de escalada de privilegios segun los binarios es GTFOBins

[LINK](https://gtfobins.github.io/)/GTFOBins


POC EXPLOTACION ARCHIVO CON PERMISOS SUID -->

He elegido un binario ubicado en /usr/bin llamado "base64" para este ejemplo. 

Lo primero de todo será concederle permisos SUID a este binario como vimos antes.

![cambio permisos](https://githubraw.com/H4ckM1nd/h4ckm1nd.github.io/master/Capturas/PERMISOS-SUID/cambio%20permisos%20suid%20a%20base64.png)

Vamos a comprobar como con la sentencia de busqueda de archivos con este tipo de permisos, indicada mas arriba, efectivamente nos encuentra el nuevo binario "base64" con los privilegios SUID.

![busqueda permisos](https://githubraw.com/H4ckM1nd/h4ckm1nd.github.io/master/Capturas/PERMISOS-SUID/busqueda%20archivos%20con%20permisos%20SUID.png)

Una vez localizado el binario que vamos a explotar y que vemos que cuenta con los privilegios de root, vamos a proceder a la explotacion de dicho binario. Buscando en la pagina GTFOBins encontramos un posible exploit para este binario en encuestion.

![gtfobins](https://githubraw.com/H4ckM1nd/h4ckm1nd.github.io/master/Capturas/PERMISOS-SUID/BASE64%20gtfobins.png)

Comprobamos por ejemplo que si intentamos acceder a leer el archivo /etc/shadow no contamos con los permisos suficientes para su lectura. Es ahora cuando entra en juego el exploit que encontramos en GTFOBins para el binario "base64" y que nos permitira leer este archivo sin ser root. Simplemente ejecutamos los comandos que nos aparecen en GTFOBins en nuestra terminal y... Done !!! Hemos posido leer un archivo del propietario root.

![explotacion](https://githubraw.com/H4ckM1nd/h4ckm1nd.github.io/master/Capturas/PERMISOS-SUID/explotacion%20binario%20base64%20suid.png)


Al igual que hemos sido capaces de leer un archivo del propietario root siendo usuarios normales, tambien es posible directamente convertirnos en root y ejecutar una shell como usuarios root con maximos privilegios.

Como habeis podido comprobar, mediante estos archivos que cuentan con permisos SUID es posible la explotacion y escalada de privilegios. Por lo que hay que tener mucho cuidado con ellos y a que archivos/binarios se le otorgan.






