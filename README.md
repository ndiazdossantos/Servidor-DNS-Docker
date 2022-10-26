# Práctica 1 - DNS Linux

# Imágenes necesarias

Para poder comenzar con la práctica primeramente debemos instalar las imágenes de:

* **Bind9**

```
sudo apt install bind9
```

* **DNSutils**


```
sudo apt install dnsutils
```


# Levantar contenedor

Para levantar el contenedor debemos crear un documento llamado ***docker-compose.yml*** donde en el definiremos todas las carácterísticas del mismo.

```
version: "3.3"
services:
  bind9:
    container_name: asir_bind9
    image: internetsystemsconsortium/bind9:9.16
    ports:
      - 5300:53/udp
      - 5300:53/tcp
    networks:
      bind9_subnet:
        ipv4_address: 10.1.0.254
    volumes:
      - /home/noe/DNS/conf:/etc/bind
      - /home/noe/DNS/zonas:/var/lib/bind
  asir_cliente:
    container_name: asir_clliente
    image: alpine
    networks:
      - bind9_subnet
    stdin_open: true #docker run -i
    tty: true #docker run -t
    dns:
      - 10.1.0.254 #dns a usar
networks:
  bind9_subnet:
    external: true

```
## Parámetros a tener en cuenta

* **Version**

_En el especificamos la versión del mismo._

* **Services**

_Dentro del mismo definiremos en ***container_name*** el nombre de nuestra imagen y en ***image*** la referencia de nuestra imagen con su versión para levantar el servicio._

_En el apartado de ***networks*** es muy importante a tener en cuenta que debemos crear anteriormente una subred con su rango de IPS y pasarela de entrada y salida para posteriormente utilizarla como un parámetro._

_Para ello utilizaremos el comando:_

```
  docker network create --subnet 10.1.0.0/24 --gateway 10.1.0.1 bind9_subnet

```

_Posteriormente a tenerla creada ya podemos asignarle una ip concreta dentro del apartado de ***ipv4_address***, en nuestro caso 10.1.0.254_

_También tendremos que definir los **volumes** para designar ```/etc/bind``` hacia nuestro directorio ```conf```, haciendo referencia a nuestros ficheros de configuración y ```/var/lib/bind``` hacia ```zonas``` haciendo referencia a las zonas del servidor DNS._

_Posteriormente definimos el nombre del contenedor y campos básicos de configuración_

# Creación fichero de configuración

Para crear el fichero de configuración para bind9 no tomaremos como referencia el que figura en [GuíaBind9](https://ubuntu.com/server/docs/service-domain-name-service-dns) si no que dividiremos el fichero de configuración en dos, en ```named.conf.local``` (contiene la configuración de la zona DNS) y ```named.conf.options``` (contiene la configuración del servidor bind9) lo cual lo especificaremos en otro fichero más denominado ```named.conf```

En ```named.conf``` indicamos esta configuración principal:

```
include "/etc/bind/named.conf.options";
include "/etc/bind/named.conf.local";
```

En el fichero de configuración de la zona DNS ```named.conf.local``` especificaremos lo siguiente:

```
zone "sdfadfe2.com." {
        type master;
        file "/var/lib/bind/db.sdfadfe2.com";
        allow-query{
            any;
        };
};'       
```
_Nuestra zona y la zona creada que se hace referencia_

Y en el fichero ```named.conf.options``` especificamos la configuración de bind9:

```
options {
        directory "/var/cache/bind";

        forwarders {
            8.8.8.8;
            8.8.4.4;
        };

        forward only;

        listen-on { any; };

        listen-on-v6 { any; };

        allow-query{ 
            any;
         };

};     
```
_Siendo los **forwarders** la configuración  predeterminada para resolver la IP, en nuestro caso indicamos los servidores de Google._


# Configuración de la ZONA DNS

Para ello en la carpeta que hemos creado de ZONA creamos un archivo precedido de **db.nombreZona.dominio** siguiendo en nuestro caso ```db.sdfadfe2.com``` donde indicaremos los registros para dicha zona.

Como se nos indicó definiremos registros de: NS, A, CNAME, TXT, SOA


