---
title: Enumerando AD
published: true
---

![Imagen AD](/assets/WindowsAD.jpg)

# Enumerando el AD

## Busqueda de información en sus DNS:
El mejor sitio para hacer la búsqueda (en mi opinión) es [Hurrican Electric]https://bgp.he.net/dns/ , cuentan con una herramienta que funciona muy bien. Ofrece un registro completo de las DNS y de las diferetens IPs asignadas que pueda tener un dominio público. Ya seque la mayoría prefiere herramientas propias como nslookup pero en mi opinión esta es la uqe mejor funciona. 

## Busqueda de información pública (archivos):
Podemos encontrar todo tipo de archivos publicados por una empresa, para ello debemos buscar en nuestro buscador el tipode archivo y la url en la que podríamos buscarlo. Estos son algunos de los tipos de archivos que podemos buscar [Tipos de archivos](https://developers.google.com/search/docs/crawling-indexing/indexable-file-types), prefiero no seguir indagando en esto porque creo que lo mejor será crear una sección solo para el OSINT.
![Imagen DuckDuck go](https://github.com/BDLBC/bdlbc.github.io/_site/assets/PDF_search.png)

## Busquda de credenciales filtradas:
A menudo en la enumeración del Direcotrio Activo veremos que las mismas herramientas que utilizan los pentesters también son utilizadas por el blueteam/equipo defensivo, un ejemplo de ello es esta herramienta para la [busqueda de credenciales filtradas](https://dehashed.com/), en ella podemos ver si las contraseñas asociadas a nuestras cuentas están filtradas, tengamos en cuenta que aunque nuestro dominio no sea vulnerado peude que si los correos privados de los trabajadores de la empresa y en caso de que se reutilicen las credenciales ya se habría vulnerado un usuario de dominio.

## Enumeración del directorio:

### Kerbrute:
> Herramienta para la enumeración de usuarios de dominio.
**Kerbrute** se apoya en el fallo de autenticación de kerberos que no suele generar logs de alerta, esto nos permite enumerar un gran número de usaurios de una manerra muy rápida, o al menos más rápida y silenciosa de lo que lo haríamos con otras herramientas como CrackMapExec. Para poder utilizarlo debermos pasarle una lista de usuarios potencialmente válidos, esta lisa la podemos obtener de múltiples maneras, en GitHub podemos encontrar repositorios con lista como [insidetrust](https://github.com/insidetrust/statistically-likely-usernames) cargada con más de 4000 nombre en inglés. En caso de querer algo más profundo (lo recomendable) es buscar de manera más inteligente, por ejemplo dentro del perfil de Linkdln de los trabajadores o extrabajadores, para este fin podemos utilizar la herramienta [Crosslinked](https://github.com/m8sec/CrossLinked), nos permite rastrear toda la red para conseguir estos nombres de usuario de dominio. 

En este [repo](https://www.hackingarticles.in/a-detailed-guide-on-kerbrute/) podemos encontrar un guía sobre la descarga, configuración y uso de la herramienta 

![Teoría de Kerbrute](https://github.com/BDLBC/bdlbc.github.io/blob/master/_site/assets/kerbrute.png)

```Python 
Happy Hacking
```

```Python 
Nos vemos pronto con la siguiente herramienta
```
 
