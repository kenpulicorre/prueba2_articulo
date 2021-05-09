# LXC/LXD

<img src="imagen/new.png" alt="Mini Shell Remoto Cliente/Servidor" width="650"/>

Esta carpeta contiene el paso a paso para acceder al servidor MySQL en un ambiente LXD
Ejecutar sobre una versión del kernel de linux 5.10.19

## Configuracion inicial

- Una vez se tenga instalado LXC y LXD y se inicie el comando `sudo lxd init` se recomienda dejar las configuraciones por defecto.
- Es importante que se intale el servidor `MySQL` en el host de la maquina donde trabaje. Para ello puede utilizar el comando:

-Instalar MySQL: `sudo apt install mysql-server `

## Procedimiento

- 1-Creacion de un nuevo contenedor (LXC) , para implementar `MySQL`, se toma como ejemplo la version `ubuntu:18.04` , y el contenedor se denota con el nombre `n-cont-mysql`, este nombre varia segun el gusto del programador:

-Crear contenedor : `lxc launch ubuntu:18.04 n-cont-mysql`

- 2-Instalar dentro del contenedor MySQL:

Inicio de sesion en el contenedor creado (n-cont-mysql):

-Abrir contenedor : `lxc exec n-cont-mysql bash`

Dentro del contenedor crear o instalar _MySQL_:

-Actualizacion en contenedor: `sudo apt update `

-Instalar MySQL en contenedor:`sudo apt install mysql-server`

-Inicie y Verifique estado del servicio MySQL: `systemctl start mysql` luego `systemctl status mysql`, se obtiene lo siguiente:

<img src="imagen/start_status_mysql.png" alt="Start_status" width="650"/>

- 3-Para que el servidor acepte conexiones de equipos remotos convendrá substituir dentro de las configuraciones del MySQL el `bind-address` cambiando de `127.0.0.1` por `0.0.0.0` para que el servidor abra un puerto en todas las interfaces de red.

  -Ubicarse dentro del contenedor (creado con MySQL) con: `lxc exec n-cont-mysql bash`

  -valide y compruebe las direcciones de red : `netstat -plnt`, si no ha configurado aun el acceso remoto, se tiene lo siguiente:
  <img src="imagen/bind_address_127.0.0.1.3306.png" alt="bind_address_127.0.0.1.3306" width="650"/>

  -Configurar el acceso remoto, abro _mysqld.cnf_, para configurar los cambio deseados: ` vi /etc/mysql/mysql.conf.d/mysqld.cnf`

Cambio de _bind-address_ de _127.0.0.1 por 0.0.0.0_
<img src="imagen/bind_address.png" alt="bind-adress" width="650"/>

-Reinicie _MySQL_, para que se apliquen los cambios: `systemctl restart mysql`

-Si no se aplican cambios pare `systemctl stop mysql` y luego aplique `systemctl restart mysql`

-valide y compruebe las direcciones de red nuevamente: `netstat -plnt`, si hizo los cambios debe tener lo siguiente:
<img src="imagen/bind_address_0.0.0.0.3306.png" alt="bind_address_0.0.0.0.3306" width="650"/>

- 4- Obtener la ip del host y contenedor para hacer la comunicación:

-Obtener la ip del Host , para ello ir a la terminal del host y el comando: `ifconfig`
<img src="imagen/iphost.png" alt="iphost" width="650"/>

-Obtener la ip del contenedor con MySQL, para ello hay dos formas:

\* Una opcion es desde la terminal del contenedor del ejemplo (`n-cont-mysql`) y digitando el comando: `ifconfig`

\* Otra opcion es desde el host, con el comando: `lxc list`

<img src="imagen/ipcontenedor.png" alt="iphost" width="650"/>

- 5- Creando flujo de informacion desde el MySQL creado en el contenedor hacia el host:

- -
-
-

- _Creacion de contenedor_

- Instalar lxc: `sudo apt install lxc`
- Instalar lxd: `sudo snap install lxd`
- Inicializar lxd con: `lxd init`
- Crear cualquier contenedor: `lxc launch imageserver:imagename instancename`
- Instanciar cualquier contenedor: `lxc start instancename`
  [ver mas](https://linuxcontainers.org/lxd/getting-started-cli/)

## Ejecución

Después de configurar lxd, para ejecutar las pruebas en lxd se debe ejecutar el archivo `scripts/stress_lxd.sh` el cual realiza lo siguiente

- Crear contenedor de prueba: `lxc launch ubuntu:20.10 lxdtest`
- Instanciar contenedor de prueba: `lxc start lxdtest`
- Instalar stress-ng en el contenedor: `lxc exec lxdtest -- snap install stress-ng`

### Stress-ng

EL comando que ejecuta todas las pruebas es el siguiente para el contenedor $CONTAINER y el tiempo de ejecución $TIMEOUT

`lxc exec $CONTAINER -- stress-ng --cpu 4 --timeout $TIMEOUT --metrics --log-file /root/stress_test.log`

Los resultados del test quedan guardados en /root/stress_test.log en el contenedor, y pueden consultarse extrayendo el archivo mediante

`lxc file pull $CONTAINER/root/stress_ng.log .`
