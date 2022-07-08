---
layout: post
title:  "Variables de entorno"
date:   2022-07-08
excerpt: "Explotación de la Variable PATH."
project: true
tag:
- Pentesting
- Linux
- Explotación
- Shell
comments: true
---
~~~~~~~~

##VARIABLE PATH:

PATH es una variable de entorno que especifica directorios que contienen programas ejecutables. 

Cuando un usuario ejecuta cualquier comando en la terminal, como el comando ls(lista directorios y archivos), pwd(muestra directorio actual de trabajo)... el sistema busca esos archivos ejecutables en los directorios especificados por la variable PATH en el orden especificado. 
Cuando ejecutamos un comando, el shell busca en los directorios de la variable, uno por uno, hasta que encuentra el directorio donde se encuentra el programa ejecutable y asi no tener que llamarlos por su ruta absoluta.

Podemos ver la ruta del PATH con los directorios especificados de la siguiente forma:

-`echo $PATH`

Para consultar todas las variables de entorno que tenemos definidas en nuestro sistema:

-`$env`

Para agregar una ruta al PATH y que podamos ejecutar cualquier binario o ejecutable solo teclenado su nombre sin tener que recurri a escribir su ruta absoluta,
tendremos que agregar dicha ruta al archivo de configuracion, normalmente ~/.bashrc

Tambien podemos hacerlo directamente desde consola de la siguiente manera:

-`export PATH=$PATH:/ruta`

##EXPLOTACION VARIABLE PATH:

Existe una manera de explotacion mediante esta variable. Por ejemplo, si tenemos un binario SUID (Ejecuta programas con los privilegios del propietario) el cual ejecuta un comando como root. El truco está en volver a escribir la variable PATH en una ubicación de nuestra elección. Cuando el binario SUID llame al sistema para buscar y ejecutar el comando, ejecutara el que hayamos escrito en su lugar. De esta manera nos ejecutara nuestro comando con los mismos privilegios que el propietario del archivo SUID original, ósea como root.

EJEMPLO-->

Localizamos un comando, en este caso "ls" el cual se ejecuta con permisos de root. Vamos a un directorio donde tengamos permisos de escritura como puede ser /tmp.
Una vez alli, creamos nuestro comando ejecutable de imitación. (Esta vez vamos a querer que nos ejecute una shell de bash suplantando el comando "ls").

-`echo [comando queramos ejecutar] > [nombre comando original]`
-`echo “/bin/bash” > ls`

Le damos permisos de ejecución a este nuevo comando

-`chmod +x ls`

Cambiamos la variable PATH, para que apunte al directorio donde tenemos almacenado nuestro comando de imitación ls 

-`export PATH=/tmp:$PATH`

Esto indica y pone en primer lugar del PATH y donde primero va a buscar el sistema a la ruta /tmp donde tenemos nuestro ejecutable "ls" de imitación con el comando shell.

Ahora cada vez que ejecutemos el comando ls, nos abrirá una shell de bash como usuarios root. En caso de querer usar el comando original para listar contenido, deberemos usar el comando ls desde su raíz original /bin/ls donde se encuentra el ejecutable ls real o restablecer la variable PATH.

Para restablecer la variable PATH a sus valores predeterminados:

exportPATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:















