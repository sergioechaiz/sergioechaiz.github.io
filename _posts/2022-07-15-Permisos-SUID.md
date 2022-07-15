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

En Linux, SUID ( establecer ID de usuario del propietario en la ejecución)  es un tipo especial de permiso de archivo otorgado a un archivo. SUID otorga permisos temporales a un usuario para ejecutar el programa/archivo con el permiso del propietario del archivo (en lugar del usuario que lo ejecuta).
Por ejemplo, el archivo binario para cambiar su contraseña (/usr/bin/passwd), tiene el bit SUID. Esto se debe a que para cambiar su contraseña, deberá escribir en el archivo shadowers al que no tiene acceso, pero root sí. Por lo que tiene privilegios de root para realizar los cambios correctos.

Buscar permisos SUID: 
`find / -user root -perm -4000 -exec ls -ldb {} \;`
`find / -perm -u=s -type f 2>/dev/null`




