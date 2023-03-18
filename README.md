#  IXIA-C ONE con tres Arista cEOS usando Containerlab

Se va a desplegar la siguiente topología
![Topología](https://github.com/LauSeVe/containerlab_IXIA-CONE_cEOS/blob/main/images/IxiaCeos.svg)

Para esto, se necesita:
- Tener descargada e instalada una versión de container lab
- Tener la imagen que despliega un contenedor Arista cEOS 


## Instalación containerlab
Para instalar la última versión de containerlab se puede realizar el siguiente comando:
~~~~
bash -c "$(curl -sL https://get.containerlab.dev)"
~~~~

Contine un script que instala automaticamente ContainerLab dependiendo del sistema operativo utiliado
https://github.com/srl-labs/containerlab/blob/main/get.sh



## Descarga de la imagen cEOS

Para descargarse la imagen de Arista cEOS es necesario tener una cuenta en https://www.arista.com.
Ahí hay que aceptar el vEOS / cEOS License Agreement para que te permitan descargarla. 

Para descargar la imagen: 
https://www.arista.com/en/support/software-download

Las pruebas se han hecho con la imagen cEOS64-lab-4.29.0.2F, ya que a 20ene23 es la última versión.

Para importar el archivo con docker:
~~~
docker import cEOS64-lab-4.29.0.2F.tar.tar ceos:4.29.0.2F
~~~


## ¿Como desplegar el escenario?
Para desplegar el escenario hay que ejecutar el comando containerlab:
~~~~
sudo containerlab deploy --topo ixiaceos.clab.yml 
~~~~

Las configuraciones de los routers y de IXIA-C ONE se ejecutan automaticamente al ejecutar el comando anterior. 



## Accediendo a los contenedores
Si se quiere entrar en el interfaz Cli de configuración de cualquier cEOS:
~~~
docker exec -it clab-ixiaceos-ceos1 Cli
enable
conf t
~~~
Como IXIA-C ONE es un contenedor de contenedores, si se quiere acceder:
~~~
sudo docker exec -i -t clab-ixiaceos-ixia-c /bin/bash
~~~


## Prueba mandar paquetes IXIA-C ONE
Se incluye también una prueba para comprobar como IXIA-C ONE manda paquetes y los recibe, siendo capaz de sacar metricas.

Para esto primero se necesita tener instalado go

~~~
cd /usr/local
sudo curl -OL https://golang.org/dl/go1.19.4.linux-amd64.tar.gz
sudo rm -rf /usr/local/go && sudo tar -xzf go1.19.4.linux-amd64.tar.gz
export PATH=$PATH:/usr/local/go/bin
go version
~~~
Se usa esta version ya que se trata de la última estable en el momento (16dic23).

Para ejecutar esta prueba se necesita saber cual es la dirección MAC asignada al interfaz eth2 del cEOS1.

Esta se puede obtener facilmente de dos modos:
- Se puede encontrar en el terminal cuando se ha desplegado el escenario (esto se ha configurado en el propio archivo clab que genera la topologia de containerlab)
- En la propia configuraciñon del contenedor viendo cual es la dirección Ethernet del interfaz e1-1

~~~
docker exec -i -t clab-ixia3srl-srl1 /bin/bash
ip addr show e1-1 | grep "ether\b" | awk '{print $2}'
~~~

Para ejecutar la prueba se tener que realizar el siguiente comando:
go run ipv4_forwarding.go -dstMac="<MAC cEOS e1-1>"

## Destruir el escenario
Para destruir el escenario creado con containerlab se puede realizar el siguiente comando:
~~~
sudo containerlab destroy
~~~
