# Desarrollo de un entorno integrado para arquitecturas 5G
 
En el presente repositorio se ve una configuración de red básica utilizando la herramienta [free5GC](https://www.free5gc.org) la cual permite crear una red centralizada 5G definida en 3GPP (R15) esto usando contenedores [_docker compose_](https://docs.docker.com/compose/).
 
Todos los créditos a los desarrolladores de la herramienta.
 
 
**Tabla de contenidos:**
 
- [Desarrollo de un entorno integrado para arquitecturas 5G](#desarrollo-de-un-entorno-integrado-para-arquitecturas-5g)
  - [Requisitos](#requisitos)
  - [Instalación](#instalación)
    - [Componentes básicos](#componentes-básicos)
    - [GTP5G kernel module](#gtp5g-kernel-module)
    - [Docker](#docker)
  - [Inicialización](#inicialización)
  - [Solución de problemas comunes](#solución-de-problemas-comunes)
  - [Visualización  de datos](#visualización--de-datos)
  - [NF dependencias y puertos](#nf-dependencias-y-puertos)
  - [Referencias](#referencias)
  - [Anexos](#anexos)
    - [Consola web](#consola-web)
 
 
***
 
## Requisitos
 
Puede ver los requisitos recomendados en la documentación oficial
[requisitos recomendados free5gc](https://github.com/free5gc/free5gc/wiki/Environment)
 
- **Software utilizado**
  - OS: GNU Linux Ubuntu 18.04.3
  - gcc 7.3.04
  - Golang 1.14.4 linux/amd64
  - kernel 5.0.0-23-generic
  
- **Hardware utilizado**
  - CPI: AMD Ryzen 7 2700X
  - RAM: 16GB
  - HDD: 180GB
  
***
 
## Instalación
 
### Componentes básicos
 
1. Verificar la versión del kernel de linux 
 
    ```
    uname -r
    5.0.0-23-generic
    ```
2. Instalación de Golang
    - Verificamos la versión de `Go`
      ```
        go version
      ```
    - Removemos si tenemos versiones distintas he instalamos `Go 1.14.4` 
      ```
        # this assumes your current version of Go is in the default location
        sudo rm -rf /usr/local/go
        wget https://dl.google.com/go/go1.14.4.linux-amd64.tar.gz
        sudo tar -C /usr/local -zxvf go1.14.4.linux-amd64.tar.gz
      ``` 
    - Para instalar esta version desde cero
        ```
            wget https://dl.google.com/go/go1.14.4.linux-amd64.tar.gz
            sudo tar -C /usr/local -zxvf go1.14.4.linux-amd64.tar.gz
            mkdir -p ~/go/{bin,pkg,src}
            # The following assume that your shell is bash
            echo 'export GOPATH=$HOME/go' >> ~/.bashrc
            echo 'export GOROOT=/usr/local/go' >> ~/.bashrc
            echo 'export PATH=$PATH:$GOPATH/bin:$GOROOT/bin' >> ~/.bashrc
            echo 'export GO111MODULE=auto' >> ~/.bashrc
            source ~/.bashrc
        ```
3. Soporte para el usuario
    ```
    sudo apt -y update
    sudo apt -y install git gcc g++ cmake autoconf libtool pkg-config libmnl-dev libyaml-dev
    go get -u github.com/sirupsen/logrus
 
    ```   
4. Soporte de red
    ```
    sudo sysctl -w net.ipv4.ip_forward=1
    sudo iptables -t nat -A POSTROUTING -o <dn_interface> -j MASQUERADE
    sudo iptables -A FORWARD -p tcp -m tcp --tcp-flags SYN,RST SYN -j TCPMSS --set-mss 1400
    sudo systemctl stop ufw
    ```
    _**Nota**: para saber el `<dn_interface>` consultar el siguiente enlace_:
    - [Linux Show / Display Available Network Interfaces](https://www.cyberciti.biz/faq/linux-list-network-interfaces-names-command/)
  
5.  Si desea continuar la intalacion para utilizarlo de manera local puede consultar el enlace [free5gc installation](https://github.com/free5gc/free5gc/wiki/Installation)
 
### GTP5G kernel module
El host debe utilizar el kernel `5.0.0-23-generic`. Y debería contener el módulo del kernel `gtp5g`.
```
git clone https://github.com/PrinzOwO/gtp5g.git
cd gtp5g
make
sudo make install
```
 
### Docker 
 
La instalación básica de docker puede verse en el siguiente enlace:
- [Docker Engine](https://docs.docker.com/engine/install/)
 
Para instalar docker compose puede seguir la siguiente guia:
 
- [Docker Compose](https://docs.docker.com/compose/install/)
 
Como dato para las pruebas se utilizaron las siguientes versiones
```
sudo apt-get install docker-ce=5:18.09.0~3-0~ubuntu-bionic docker-ce-cli=5:18.09.0~3-0~ubuntu-bionic containerd.io
```
 
***
## Inicialización
 
Para este caso como es necesario una interfaz de túnel es necesario instanciar los contenedores con **privilegios de administrador**.
```
$ git clone https://github.com/free5gc/free5gc-compose.git
$ cd free5gc-compose
$ make base
$ docker-compose build
$ sudo docker-compose up # Recommend use with tmux to run in foreground mode
$ sudo docker-compose up -d # Run in background mode if needed
```
***
## Solución de problemas comunes
 
En casos es necesario realizar tomas de datos almacenados mediante `mongodb`
 
```
$ docker exec -it mongodb mongo
> use free5gc
> db.subscribers.drop()
> exit # (Or Ctrl-D)
```
***
## Visualización  de datos
Podemos ver los logs de cada imagen con  `docker logs` . Por ejemplo para ver los datos del **SMF** podemos hacerlo de la siguiente manera:
 
```
docker logs smf
```
***
 
## NF dependencias y puertos
 
| NF | Exposed Ports | Dependencies | Dependencies URI |
|:-:|:-:|:-:|:-:|
| amf | 8000 | nrf | nrfUri: https://nrf:8000 |
| ausf | 8000 | nrf | nrfUri: https://nrf:8000 |
| nrf | 8000 | db | MongoDBUrl: mongodb://db:27017 |
| nssf | 8000 | nrf | nrfUri: https://nrf:8000/,<br/>nrfId: https://nrf:8000/nnrf-nfm/v1/nf-instances |
| pcf | 8000 | nrf | nrfUri: https://nrf:8000 |
| smf | 8000 | nrf, upf | nrfUri: https://nrf:8000,<br/>node_id: upf1, node_id: upf2, node_id: upf3 |
| udm | 8000 | nrf | nrfUri: https://nrf:8000 |
| udr | 8000 | nrf, db | nrfUri: https://nrf:8000,<br/>url: mongodb://db:27017 |
| n3iwf | N/A | amf, smf, upf |  |
| upf1 | N/A | pfcp, gtpu, apn | pfcp: upf1, gtpu: upf1, apn: internet |
| upf2 | N/A | pfcp, gtpu, apn | pfcp: upf2, gtpu: upf2, apn: internet |
| upfb (ulcl) | N/A | pfcp, gtpu, apn | pfcp: upfb, gtpu: upfb, apn: intranet |
| webui | 5000 | db | MongoDBUrl: mongodb://db:27017  |
***
## Referencias
- https://github.com/open5gs/nextepc/tree/master/docker
- https://github.com/abousselmi/docker-free5gc
- https://github.com/free5gc/free5gc-compose/
 
 
## Anexos
 
### Consola web
 
 
![img](/images/webconsole.jpg)
 
```
URL: http://localhost:5000
Username: admin
Password: free5gc
```
 
 
 
