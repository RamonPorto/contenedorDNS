Práctica contenedorDNS

# 1. Primero Creamos el docker-compose.yml

***Fichero docker-compose.yml:***
~~~

services:
  asir_bind9:
    container_name: asir_bind9
    image: internetsystemsconsortium/bind9:9.16
    ports:
      - "53:53/tcp"
      - "53:53/udp"
    networks:
      bind9_subnet:
        ipv4_address: 172.28.5.1
    volumes:
      - ./conf:/etc/bind
      - ./zonas:/var/lib/bind
networks:
  bind9_subnet:
    external: true

~~~

Primero crearemos el docker compose para poder crear el contenedor del servidor

# 2. Después creamos la red "bind9_subnet"

Comando para crear la red:
~~~

$ docker network create \
  --driver=bridge \
  --subnet=172.28.0.0/16 \
  --ip-range=172.28.5.0/24 \
  --gateway=172.28.5.254 \
  bind9_subnet

~~~

Posteriormente creamos la subred mencionada en el docker compose.

# 3.Creación del directorio conf y sus archivos correspondientes.

***Fichero named.conf:***
~~~

include "/etc/bind/named.conf.options";
include "/etc/bind/named.conf.local";
include "/etc/bind/named.conf.default-zones";

~~~

***Fichero named.conf.default-zones:***
~~~

// prime the server with knowledge of the root servers
zone "." {
	type hint;
	file "/usr/share/dns/root.hints";
};

// be authoritative for the localhost forward and reverse zones, and for
// broadcast zones as per RFC 1912

zone "localhost" {
	type master;
	file "/etc/bind/db.local";
};

zone "127.in-addr.arpa" {
	type master;
	file "/etc/bind/db.127";
};

zone "0.in-addr.arpa" {
	type master;
	file "/etc/bind/db.0";
};

zone "255.in-addr.arpa" {
	type master;
	file "/etc/bind/db.255";
};

~~~

***Fichero named.conf.local:***
~~~

zone "asircastelao.int" {
	type master;
	file "/var/lib/bind/db.asircastelao.int";
	allow-query {
		any;
		};
	};

~~~

***Fichero named.conf.options:***
~~~

options {
	directory "/var/cache/bind";

	forwarders {
	 	8.8.8.8;
		1.1.1.1;
	 };
	 forward only;

	listen-on { any; };
	listen-on-v6 { any; };

	allow-query {
		any;
	};
};

~~~

Creamos el directorio "conf" al que hacemos referencia en el docker compose y creamos los ficheros de configuración

# 4. Creación del directorio zonas y el fichero db.asircastelao.int

***Fichero db.asircastelao.int:***
~~~

$TTL 38400	; 10 hours 40 minutes
@		IN SOA	ns.asircastelao.int. some.email.address. (
				10000002   ; serial
				10800      ; refresh (3 hours)
				3600       ; retry (1 hour)
				604800     ; expire (1 week)
				38400      ; minimum (10 hours 40 minutes)
				)
@		IN NS	ns.asircastelao.int.
ns		IN A		172.28.5.1
test	IN A		172.28.5.4
alias	IN CNAME	test
texto	IN TXT		mensaje

~~~



*Todos estos ficheros se encuentran en el repositorio de la práctica en github.*

# 5. Prueba de que funciona correctamente

Para confirmar que está creado correctamente debemos instalar la herramienta dig en el contenedor con los siguientes comandos:

~~~

apt update

apt install dnsutils

~~~

# 6. Creación del cliente

Para crear el cliente los debemos añadir en el archivo docker-compose.yml

Nuestro fichero debería ser el siguiente:

~~~

services:
  asir_bind9:
    container_name: asir_bind9
    image: ubuntu/bind9
    ports:
      - "53:53/tcp"
      - "53:53/udp"
      #Mapeo de puertos
    networks:
      bind9_subnet:
        ipv4_address: 172.28.5.1
    volumes:
      - ./conf:/etc/bind
      - ./zonas:/var/lib/bind
      #Para mapear los directorios
  cliente:
    container_name: asir_cliente_dns
    image: alpine
    platform: linux/amd64
    tty: true
    stdin_open: true
    dns:
      - 172.28.5.1
    networks:
      bind9_subnet:
        ipv4_address: 172.28.5.33
networks:
  bind9_subnet:
    external: true

~~~

# 7.Creación de los contenedores

Para crear ambos contenedores deberemos volver a ejecutar el archivo docker-compose con el siguiente comando:

~~~

docker compose -f docker-compose.yml up

~~~

*Agregamos -f para indicar el fichero docker-compose que queremos ejecutar*