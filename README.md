# Examen1_1-Ev._SRI_MarcosAlonso
Crea un repositorio para contestar todo o exame.
Este repositorio ten que conter tódo-los ficheiros necesarios para xustifica-la túas respostas.

Contesta as seguintes preguntas, xustificándoas, nun README.md

## 1. Explica métodos para 'abrir' unha consola/shell a un contenedor en execución.

Para meternos na consola dun contenedor xa en execucion debemos facel algo tal que asi: `docker exec -it <container id o nome> /bin/bash(ou o interprete que use, por exemplo sh)` 

------------------Tamen podemos facer un `docker run -it <image_name> sh` que é basicamente o mesmo.  
E por ultimo podemos facer un `docker attach <container id o nome>`

## 2. No contenedor anterior (en execución), qué opciones tes que ter usado ó arrincalo para poder interactuar coas súas entradas e salidas
Para meterte na consola, podes facer un contenedor previo como demonio co -d (exemplo: docker run -d alpine sh) para que asi podas poñer na mesma terminar un dos anteriores comandos ou iniciar o contenedor sen o -d y noutra terminal poñer o comando.  
As opcions que se usan son -i para mantener a consola aberta e o -t que crea a propia consola.  
Un exemplo de que inicie un contenedor e metanos na terminal ao momento seria o seguinte: `docker run -it --name interactivo alpine sh` (crea un sistema alpine e metenos na sua consola directamente)



## 3. Cómo sería un ficheiro docker-compose para que dous contenedores se comuniquen entre si nunha rede só deles?
```
services:
  contenedor1:
    image: ubuntu
    networks:
      redcompartida:

  contenedor2:
    image: alpine
    networks:
      redcompartida:

networks:
  redcompartida:
    driver: bridge
```
Este docker compose.yaml xa crea a red e fai que esten conectados mediante a mesma pero tamen poderiamos crear a rede antes de executar o docker compose (co comando `docker network create redcompartida`) e poñer no final do .yaml que non cree a rede co seguiente comando:
```
networks:
  bind9_subnet:
    external: true
    
```
## 4. Qué tes que engadir ó ficheiro anterior para que un contenedor teña unha IP fixa?
Pois engadimos a configuracion da rede en engadimos as IP fixas que queiramos (a continuacion deixo o .yaml outra vez explicado): 
```
services:
  contenedor1:
    image: ubuntu
    networks:
      redcompartida:
        ipv4_address: "192.168.1.2" #engadimos a ip fixa 192.168.1.2

  contenedor2:
    image: alpine
    networks:
      redcompartida:
        ipv4_address: "192.168.1.3" #engadimos a ip fixa 192.168.1.3

networks:
  redcompartida:
    driver: bridge
    ipam:
      config:
        - subnet: "192.168.1.0/24"
```

## 5. Qué comando de consola podo usar para sabe-las ips dos contenedores anteriores?
O primeiro comando de doker que pode ser util e o seguinte xa que lista as IP da rede que acabamos de crear: `docker network inspect redcompartida`
-------dig ???


## 6. Cál é a funcionalidade do apartado "ports" en docker compose?
A funcionalidade é a de mapea os portos do contenedor ao host (para que se podan conectar ao exterior) e funciona da seguinte manera `80:81` = `host:contenedor`


## 7. Para qué serve o rexistro CNAME? Pon un exemplo
O rexistro CNAME permite crear un alias dun dominio a outro.
`www.exemplo.com. IN CNAME exemplo.com.`  
Por si non queda claro vou deixar a explicacion dunha linea dunha zona de unha practica mia onde a zona chamabase pepe.int
```
test IN A 192.28.5.4      ; Rexistro A. O dominio "test.pepe.int" apunta á IP 192.28.5.4.
  
alias IN CNAME test       ; Rexistro CNAME. O alias "alias.pepe.int" apunta a "test.pepe.int".
```

## 8. Cómo podo facer para que a configuración dun contenedor DNS non se borre se creo outro contenedor?
Para que a informacion non se borre utilizamos volumes. A continuacion vou deixar un exemplo dun .yaml dun contenedor dns que fixen con bind9:
```
services:
  servidor:
    container_name: server  # Nome do contedor para o servidor DNS
    image: ubuntu/bind9  # Imaxe base que proporciona BIND9 para configurar DNS
    platform: linux/amd64  # Especificamos a plataforma, útil para evitar conflitos en máquinas ARM
    ports:
      - 54:53  # Mapeamos o porto 54 do host ao porto 53 (DNS) do contedor
    networks:
      bind9_subnet:  # Engadimos o contedor á rede personalizada 'bind9_subnet'
        ipv4_address: 192.28.5.4  # Enderezo IP estático asignado ao servidor DNS


    volumes: #-----ESTA É A PARTE IMPORTANTE DA PREGUNTA DONDE CREO DOUS VOLUMES EN LOCAL PARA QUE SE GARDEN AS CONFIGURACIONS-----
      - ./conf:/etc/bind/  # Montamos o directorio local './conf' dentro do contedor como '/etc/bind'
      - ./zonas:/var/lib/bind/  # Montamos o directorio local './zonas' dentro do contedor como '/var/lib/bind'

```


## 9. Engade unha zoa tendaelectronica.int no teu docker DNS que teña:
### - www á IP 172.16.0.1  
Feito na liña 15 en zonas
### - owncloud sexa un CNAME de www  
Feito na liña 16 en zonas
### - un rexistro de texto có contido "1234ASDF"  
Feito na liña 17 en zonas
### Comproba que todo funciona có comando "dig"
Hay duas formas de instalar estas ferramientas de red (dig):  
1. Unha vez feito o `docker compose up` debemos facer noutra terminal un `docker exec -it ubuntu /bin/bash` para asi entrar na terminal do cliente de ubuntu e poñer o comando `apt-get update && apt-get install -y dnsutils` 

2. Poñendo directamente no .yaml la configuracion para que asi xa nos teñamos que facer o update e install a man (esta e a opcion que elexin xa que é mais comoda):
```
    command: /bin/bash -c "apt-get update && apt-get install -y dnsutils && bash" 
```  
A continucion deixarei a resposta dos digs ( aunque tamen adxuntarei fotos ):   
Primeiro facemos un `docker exec -it cliente_ubuntu /bin/bash`.  
Comprobamos o `dig www.tendaelectronica.int`
```
root@9e5450e69055:/# dig www.tendaelectronica.int

; <<>> DiG 9.18.28-0ubuntu0.24.04.1-Ubuntu <<>> www.tendaelectronica.int
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 46024
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: d60525865552af2e010000006737900984c810926876154c (good)
;; QUESTION SECTION:
;www.tendaelectronica.int.	IN	A

;; ANSWER SECTION:
www.tendaelectronica.int. 38400	IN	A	172.16.0.1

;; Query time: 4 msec
;; SERVER: 127.0.0.11#53(127.0.0.11) (UDP)
;; WHEN: Fri Nov 15 18:16:41 UTC 2024
;; MSG SIZE  rcvd: 97
```

Comprobamos o `dig owncloud.tendaelectronica.int` e que o cname funciona:

```
root@9e5450e69055:/# dig owncloud.tendaelectronica.int

; <<>> DiG 9.18.28-0ubuntu0.24.04.1-Ubuntu <<>> owncloud.tendaelectronica.int
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 55729
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 2, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: 310b2811faebf16c0100000067379091c529663d8ac1054a (good)
;; QUESTION SECTION:
;owncloud.tendaelectronica.int.	IN	A

;; ANSWER SECTION:
owncloud.tendaelectronica.int. 38400 IN	CNAME	www.tendaelectronica.int.
www.tendaelectronica.int. 38400	IN	A	172.16.0.1

;; Query time: 0 msec
;; SERVER: 127.0.0.11#53(127.0.0.11) (UDP)
;; WHEN: Fri Nov 15 18:18:57 UTC 2024
;; MSG SIZE  rcvd: 120

```

Compobamos o `dig texto.tendaelectronica.int TXT` (poño texto antes da consulta xa que no arquivo de zonas o teno configrado asi )


```
root@9e5450e69055:/# dig texto.tendaelectronica.int TXT

; <<>> DiG 9.18.28-0ubuntu0.24.04.1-Ubuntu <<>> texto.tendaelectronica.int TXT
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 61116
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: 9b977b6c59e0845101000000673791d49fdc5d32636f4240 (good)
;; QUESTION SECTION:
;texto.tendaelectronica.int.	IN	TXT

;; ANSWER SECTION:
texto.tendaelectronica.int. 38400 IN	TXT	"1234ASDF"

;; Query time: 4 msec
;; SERVER: 127.0.0.11#53(127.0.0.11) (UDP)
;; WHEN: Fri Nov 15 18:24:20 UTC 2024
;; MSG SIZE  rcvd: 104


```
### Mostra nos logs que o servicio funciona ben usando a saída da terminal ó levantar o compose ou có comando "docker logs [nomeContenedorOuID]"
(o apartado 9 realízase na máquina virtual)

Vou copiar é pegar o resultado do comado do `docker compose up`

```
postgres@debian:~/Examen1_1-Ev._SRI_MarcosAlonso$ docker compose up
[+] Running 2/0
 ✔ Container cliente_ubuntu  Created                                                     0.0s 
 ✔ Container servidor_DNS    Created                                                     0.0s 
Attaching to cliente_ubuntu, servidor_DNS
servidor_DNS    | Starting named...
servidor_DNS    | exec /usr/sbin/named -u "bind" "-g" ""
servidor_DNS    | 15-Nov-2024 18:23:02.978 starting BIND 9.18.28-0ubuntu0.24.04.1-Ubuntu (Extended Support Version) <id:>
servidor_DNS    | 15-Nov-2024 18:23:02.978 running on Linux x86_64 5.10.0-33-amd64 #1 SMP Debian 5.10.226-1 (2024-10-03)
servidor_DNS    | 15-Nov-2024 18:23:02.978 built with  '--build=x86_64-linux-gnu' '--prefix=/usr' '--includedir=${prefix}/include' '--mandir=${prefix}/share/man' '--infodir=${prefix}/share/info' '--sysconfdir=/etc' '--localstatedir=/var' '--disable-option-checking' '--disable-silent-rules' '--libdir=${prefix}/lib/x86_64-linux-gnu' '--runstatedir=/run' '--disable-maintainer-mode' '--disable-dependency-tracking' '--libdir=/usr/lib/x86_64-linux-gnu' '--sysconfdir=/etc/bind' '--with-python=python3' '--localstatedir=/' '--enable-threads' '--enable-largefile' '--with-libtool' '--enable-shared' '--disable-static' '--with-gost=no' '--with-openssl=/usr' '--with-gssapi=yes' '--with-libidn2' '--with-json-c' '--with-lmdb=/usr' '--with-gnu-ld' '--with-maxminddb' '--with-atf=no' '--enable-ipv6' '--enable-rrl' '--enable-filter-aaaa' '--disable-native-pkcs11' 'build_alias=x86_64-linux-gnu' 'CFLAGS=-g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/build/bind9-CTg8aa/bind9-9.18.28=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/build/bind9-CTg8aa/bind9-9.18.28=/usr/src/bind9-1:9.18.28-0ubuntu0.24.04.1 -fno-strict-aliasing -fno-delete-null-pointer-checks -DNO_VERSION_DATE -DDIG_SIGCHASE' 'LDFLAGS=-Wl,-Bsymbolic-functions -flto=auto -ffat-lto-objects -Wl,-z,relro -Wl,-z,now' 'CPPFLAGS=-Wdate-time -D_FORTIFY_SOURCE=3'
servidor_DNS    | 15-Nov-2024 18:23:02.978 running as: named -u bind -g
servidor_DNS    | 15-Nov-2024 18:23:02.978 compiled by GCC 13.2.0
servidor_DNS    | 15-Nov-2024 18:23:02.978 compiled with OpenSSL version: OpenSSL 3.0.13 30 Jan 2024
servidor_DNS    | 15-Nov-2024 18:23:02.978 linked to OpenSSL version: OpenSSL 3.0.13 30 Jan 2024
servidor_DNS    | 15-Nov-2024 18:23:02.978 compiled with libuv version: 1.48.0
servidor_DNS    | 15-Nov-2024 18:23:02.978 linked to libuv version: 1.48.0
servidor_DNS    | 15-Nov-2024 18:23:02.978 compiled with libxml2 version: 2.9.14
servidor_DNS    | 15-Nov-2024 18:23:02.978 linked to libxml2 version: 20914
servidor_DNS    | 15-Nov-2024 18:23:02.978 compiled with json-c version: 0.17
servidor_DNS    | 15-Nov-2024 18:23:02.978 linked to json-c version: 0.17
servidor_DNS    | 15-Nov-2024 18:23:02.978 compiled with zlib version: 1.3
servidor_DNS    | 15-Nov-2024 18:23:02.978 linked to zlib version: 1.3
servidor_DNS    | 15-Nov-2024 18:23:02.978 ----------------------------------------------------
servidor_DNS    | 15-Nov-2024 18:23:02.978 BIND 9 is maintained by Internet Systems Consortium,
servidor_DNS    | 15-Nov-2024 18:23:02.978 Inc. (ISC), a non-profit 501(c)(3) public-benefit 
servidor_DNS    | 15-Nov-2024 18:23:02.978 corporation.  Support and training for BIND 9 are 
servidor_DNS    | 15-Nov-2024 18:23:02.978 available at https://www.isc.org/support
servidor_DNS    | 15-Nov-2024 18:23:02.978 ----------------------------------------------------
servidor_DNS    | 15-Nov-2024 18:23:02.978 found 6 CPUs, using 6 worker threads
servidor_DNS    | 15-Nov-2024 18:23:02.978 using 6 UDP listeners per interface
servidor_DNS    | 15-Nov-2024 18:23:02.978 DNSSEC algorithms: RSASHA1 NSEC3RSASHA1 RSASHA256 RSASHA512 ECDSAP256SHA256 ECDSAP384SHA384 ED25519 ED448
servidor_DNS    | 15-Nov-2024 18:23:02.978 DS algorithms: SHA-1 SHA-256 SHA-384
servidor_DNS    | 15-Nov-2024 18:23:02.978 HMAC algorithms: HMAC-MD5 HMAC-SHA1 HMAC-SHA224 HMAC-SHA256 HMAC-SHA384 HMAC-SHA512
servidor_DNS    | 15-Nov-2024 18:23:02.978 TKEY mode 2 support (Diffie-Hellman): yes
servidor_DNS    | 15-Nov-2024 18:23:02.978 TKEY mode 3 support (GSS-API): yes
servidor_DNS    | 15-Nov-2024 18:23:02.978 loading configuration from '/etc/bind/named.conf'
servidor_DNS    | 15-Nov-2024 18:23:02.978 unable to open '/etc/bind/bind.keys'; using built-in keys instead
servidor_DNS    | 15-Nov-2024 18:23:02.978 looking for GeoIP2 databases in '/usr/share/GeoIP'
servidor_DNS    | 15-Nov-2024 18:23:02.978 using default UDP/IPv4 port range: [32768, 60999]
servidor_DNS    | 15-Nov-2024 18:23:02.986 using default UDP/IPv6 port range: [32768, 60999]
servidor_DNS    | 15-Nov-2024 18:23:02.986 listening on IPv4 interface lo, 127.0.0.1#53
Ign:1 http://archive.ubuntu.com/ubuntu noble InRelease
Ign:2 http://archive.ubuntu.com/ubuntu noble-updates InRelease
Ign:3 http://security.ubuntu.com/ubuntu noble-security InRelease
servidor_DNS    | 15-Nov-2024 18:23:02.986 listening on IPv4 interface eth0, 172.16.0.4#53
servidor_DNS    | 15-Nov-2024 18:23:02.986 IPv6 socket API is incomplete; explicitly binding to each IPv6 address separately
servidor_DNS    | 15-Nov-2024 18:23:02.986 listening on IPv6 interface lo, ::1#53
servidor_DNS    | 15-Nov-2024 18:23:02.986 generating session key for dynamic DNS
servidor_DNS    | 15-Nov-2024 18:23:02.986 sizing zone task pool based on 1 zones
servidor_DNS    | 15-Nov-2024 18:23:02.986 none:99: 'max-cache-size 90%' - setting to 6661MB (out of 7401MB)
servidor_DNS    | 15-Nov-2024 18:23:02.986 set up managed keys zone for view _default, file 'managed-keys.bind'
servidor_DNS    | 15-Nov-2024 18:23:02.986 configuring command channel from '/etc/bind/rndc.key'
servidor_DNS    | 15-Nov-2024 18:23:02.986 command channel listening on 127.0.0.1#953
servidor_DNS    | 15-Nov-2024 18:23:02.986 configuring command channel from '/etc/bind/rndc.key'
servidor_DNS    | 15-Nov-2024 18:23:02.986 command channel listening on ::1#953
servidor_DNS    | 15-Nov-2024 18:23:02.986 not using config file logging statement for logging due to -g option
servidor_DNS    | 15-Nov-2024 18:23:02.986 managed-keys-zone: loaded serial 0
servidor_DNS    | 15-Nov-2024 18:23:02.986 zone tendaelectronica.int/IN: loaded serial 10000002
servidor_DNS    | 15-Nov-2024 18:23:02.986 all zones loaded
servidor_DNS    | 15-Nov-2024 18:23:02.986 running
Hit:3 http://security.ubuntu.com/ubuntu noble-security InRelease
Hit:4 http://archive.ubuntu.com/ubuntu noble-backports InRelease
Hit:1 http://archive.ubuntu.com/ubuntu noble InRelease
Hit:2 http://archive.ubuntu.com/ubuntu noble-updates InRelease
Reading package lists... Done
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
```



