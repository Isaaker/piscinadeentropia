---
title: "Pi hole"
date: "2024-03-21"
---

# Introducción a Pi-Hole®

![](../../images/pi-hole-logo.jpg)

Hola de nuevo, después de unos meses sin escribir queria volver al blog actualizando y añadiendo unas entradas que tenia pendientes, estas son las entradas sobre Pi-Hole®.

## ¿Qué es Pi-Hole®?

Pi-Hole® es un programa basado en la tecnología DNS que se puede ejecutar tanto en Linux como en una imagen docker (normalmente se usa en Raspberry Pi) y que como principal objetivo bloquea los diferentes dominios y servicios que el usuario no quiere que sean accesibles, permitiendo asi el bloqueo de anuncios y webs peligrosas.

Además Pi-Hole® es un proyecto de codigo abierto por lo que puedes consultar y colaborar en su codigo de forma abierta y libre.

## ¿Cómo funciona Pi-Hole®?

Como ya vengo explicando, Pi-Hole® es un programa facilmente instalable en sistemas linux o mediante una imagen docker, y que permite el bloqueo de dominios mediante tecnología DNS.

El usuario conecta los diferentes dispositivos que quiere que reciban la protección de Pi-Hole®, mediante los ajustes de red cambiando la opción DNS a la dirección IP del equipo que esta ejecutando Pi-hole®. Y Pi-hole® lo que hace es recibir todas las peticiones de conexion del dispositivo a los diferentes dominios, filtrando estas mediante las listas y reglas establecidas en los ajustes (que trataremos más adelante), en caso de que la petición sea valida la dejara pasar (devolvera al dispositivo la ip del servidor que ha solicitado) y en caso contrario "cortara" la conexión para que el dispositivo no se pueda conectar a ese dominio (devolverá la IP 0.0.0.0 que no es una IP valida).

![](../../images/esquema_pihole.jpg)

## ¿Qué es el protocolo DNS?

Para comprender todo esto un poco mejor, os dejo aquí una breve descripción de que es el protocolo DNS:

El sistema de Nombres de Dominio o Domain Name Server en inglés es una tecnología desarrollada en los años 80 y que tiene como proposito asociar nombres de dominio a las direcciones IPs de los servidores o dispositivos en Internet y en Redes en general.

Una buena forma de entender esto seria con los lugares reales:

* Un lugar (ejemplo: mi casa) = Un dominio (ejemplo: piscinadeentropia.es)

* Una dirección (ejemplo: 1 Apple Park Way) = Una dirección IP (ejemplo: 1.1.1.1)

El lugar seria el dominio y la dirección seria la IP.

Es por esto que necesitamos un DNS, ya que lo que tiene el DNS es una base de datos que interelaciona las direcciónes IP con los nombres de dominio, así cuando tu tecleas en la URL piscinadeentropia.es tu navegador sabe donde esta el servidor de la web y se puede conectar.

Por último, para los que querais saber como se elaboran estas bases de datos, cuando los propietarios de un dominio cambian su IP, lo que se hace es que se conecta a su proovedor de dominio y lo actualiza, y es este el que se encarga de informar a los servidores DNS de que esto ha cambiado, por lo que este tipo de cambios pueden tardar en propagarse un poco.

**No solo hay entradas de tipo IP en los servidores DNS también puede haber texto(TXT) que se usa para verificar que eres el propietario del dominio y puedes cambiar los registros o de tipo CNAME para redirigir a otros dominios.**

## ¿Dónde puedo usar Pi-Hole®?

Otro punto importante a tratar es donde de puede instalar este programa, para ello veremos los distintos requerimientos que tiene el programa:

### Sistemas Operativos Compatibles

| Sistema Operativo | Version | Arquitectura |
|-------------------|---------|--------------|
| Raspberry Pi OS (Raspbian)| Buster o Bullseye |ARM|
| Armbian OS | Cualquiera | ARM, x86_64 o riscv64 |
| Ubuntu | 20.x, 22.x o 23.x | ARM o x86_64 |
| Debian | 10, 11 o 12 | ARM, x86_64 o i386 |
| Fedora | 36, 37 o 38 | ARM o x86_64 |
| CentOS Stream | 8 o 9 | x86_64 |

### Requisitos de Hardware

- Mínimo 2 GB de espacio en disco, aunque se recomienda tener 4 GB.
- Mínimo de 512 MB de RAM

### Requisitos de Red

#### IP Estática

Es importante mencionar que pese a que no es necesario establecer la IP de la maquina que ejecuta Pi-Hole® como el DNS primario (que se puede hacer para no tener que cambiar los ajustes DNS de todos los dispositivos, si que es necesario que el Pi-Hole® cuente con una IP estática (que no cambie). Para ello, necesitará realizar una reserva DCHP en su router.

**Para saber más sobre esto, consulté la página web de soporte del fabricante de su router.**

#### Puertos

Pi-Hole® también necesita usar los siguientes puertos para poder funcionar correctamente:

| Puerto | Servicio | Protocolo | Uso |
|--------|----------|-----------|-----|
| 53 (DNS) | pihole-FTL | TCP/UDP | Es el puerto donde se reciben las peticiones DNS de los dispositivos|
| 67 (DCHP) | pihole-FTL | IPv4 UDP | Es un puerto de uso opcional y que se activa para poder ofrecer el servicio de DCHP |
| 547 (DCHPv6) | pihole-FTL | IPv6 UDP | Es un puerto de uso opcional y que se activa para poder ofrecer el servicio de DCHP |
| 80 (HTTP) | lighthttpd | TCP | De uso opcional pero muy recomendado ya que nos permite conectarnos y cambiar los ajustes de pihole desde un navegador |
| 4711 | pihole-FTL | TCP | Es un puerto obligatorio, ya que es para uso interno del programa (para conectar a la API), no se debe de poder acceder a el desde otro equipo |

#### Firewalls

En caso de que use firewall en su maquina, la documentación de pi-hole recomienda añadir las siguientes excepciones:

**IPTable**

IPv4

```
iptables -I INPUT 1 -s 192.168.0.0/16 -p tcp -m tcp --dport 80 -j ACCEPT
iptables -I INPUT 1 -s 127.0.0.0/8 -p tcp -m tcp --dport 53 -j ACCEPT
iptables -I INPUT 1 -s 127.0.0.0/8 -p udp -m udp --dport 53 -j ACCEPT
iptables -I INPUT 1 -s 192.168.0.0/16 -p tcp -m tcp --dport 53 -j ACCEPT
iptables -I INPUT 1 -s 192.168.0.0/16 -p udp -m udp --dport 53 -j ACCEPT
iptables -I INPUT 1 -p udp --dport 67:68 --sport 67:68 -j ACCEPT
iptables -I INPUT 1 -p tcp -m tcp --dport 4711 -i lo -j ACCEPT
iptables -I INPUT -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT
```

IPv6

```
ip6tables -I INPUT -p udp -m udp --sport 546:547 --dport 546:547 -j ACCEPT
ip6tables -I INPUT -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT
```

**FirewallD**

```
firewall-cmd --permanent --add-service=http --add-service=dns --add-service=dhcp --add-service=dhcpv6
firewall-cmd --permanent --new-zone=ftl
firewall-cmd --permanent --zone=ftl --add-interface=lo
firewall-cmd --permanent --zone=ftl --add-port=4711/tcp
firewall-cmd --reload
```

**ufw**

IPv4

```
ufw allow 80/tcp
ufw allow 53/tcp
ufw allow 53/udp
ufw allow 67/tcp
ufw allow 67/udp
```

IPv6

```
ufw allow 546:547/udp
```

### Versión de docker

Tembién hay una version de docker disponible, con lo que puede ejecutar Pi-Hole® en sistemas que no son compatibles directamente pero que si cumplen con los requisitos de docker.

- Requisitos de Docker en Linux: [Ver](https://docs.docker.com/engine/install/#supported-platforms)
- Requisitos de Docker en Mac: [Ver](https://docs.docker.com/desktop/install/mac-install/#system-requirements)
- Requisitos de Docekr en Windows: [Ver](https://docs.docker.com/desktop/install/windows-install/#system-requirements)

**Puedes encontrar la imagen de docker [aquí](https://github.com/pi-hole/docker-pi-hole)**

### Información adicional

El script de instalación de Pi-Hole® lleva a cabo un proceso de verificación para comporbar si tu sistema cuenta con los requisitos de instalación aqui mencionados.

Comandos para comprobar las características de tu sistema Linux:

- Comprobar el Sistema Operativo y su version: ```cat /etc/os-release```
- Obtener información de la CPU: ```lscpu```
- Comprobar espacio libre en los discos conectados: ```df```
- Comprobar el tamaño de la RAM: ```free -m```

[Para saber más puedes leer aquí la documentación oficial](https://docs.pi-hole.net/main/prerequisites)

## Beneficios
Pi-Hole® ofrece gran cantidad de funciones y beneficios a sus usuarios:

- Bloquear Servicios y Dominios no deseados, anuncios y contenido no seguro.
- Te da mayor privacidad, ya que algunos ISP (Internet Service Provider) y DNS públicos recaban datos acerca de que conexiones realizas para vender despues esos datos.
- Mayor seguridad, bloquea algunos esquemas y tipos de conexion que pueden no ser seguros.
- Te permite hacer uso de DNSSEC, pudiendo comprobar si las entradas DNS estan firmadas.
- Puedes controlar la actividad en red de tus dispositivos.
- Puedes crearte uns servidos DCHP propio en caso de no tenerlo.
- Puede ayudarte a tener una velocidad de conexion mayor
- Y por último, no solo es gratis si no que no comparte ningun tipo de datos y solo recoje los que tu desees.

## Capturas de Pi-Hole®
**Capturas obtenidas de la página web de pi-hole**

![Pi-Hole Vista General](https://wp-cdn.pi-hole.net/wp-content/uploads/2020/04/Dashboard.png)
![DCHP Config](https://wp-cdn.pi-hole.net/wp-content/uploads/2018/12/Screenshot-2018-12-19-17.39.58.png)
![Lista de Bloqueo manual](https://wp-cdn.pi-hole.net/wp-content/uploads/2020/02/Screenshot-2018-12-22-11.17.24.png)
![Log de Peticiones](https://wp-cdn.pi-hole.net/wp-content/uploads/2018/12/Screenshot-2018-12-19-17.50.40.png)
![Estadísticas](https://wp-cdn.pi-hole.net/wp-content/uploads/2020/08/long-term-stats1-e1597224606922.png)
![Registro de auditoria](https://wp-cdn.pi-hole.net/wp-content/uploads/2020/08/audit-log1.png)
![Modos de privacidad](https://wp-cdn.pi-hole.net/wp-content/uploads/2020/02/Screenshot-2018-12-20-12.30.34.png)
![Otros Ajustes](https://wp-cdn.pi-hole.net/wp-content/uploads/2020/02/Screenshot-2018-12-20-12.26.29.png)

## Enlaces oficiales de Pi-Hole®

- [Página web oficial](https://pi-hole.net)
- [Foro](https://discourse.pi-hole.net)
- [Reddit](https://www.reddit.com/r/pihole/)
- [X](https://twitter.com/The_Pi_Hole)

## Despedida

Y esto es todo por ahora, intentare continuar con los artículos de pihole y el siguiente sera un pequeño manual de instalación junto a una explicación amplia de características y todo lo que se le puede personalizar a este programa.