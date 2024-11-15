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
- www á IP 172.16.0.1
- owncloud sexa un CNAME de www
- un rexistro de texto có contido "1234ASDF"
Comproba que todo funciona có comando "dig"
Mostra nos logs que o servicio funciona ben usando a saída da terminal ó levantar o compose ou có comando "docker logs [nomeContenedorOuID]"
(o apartado 9 realízase na máquina virtual)








