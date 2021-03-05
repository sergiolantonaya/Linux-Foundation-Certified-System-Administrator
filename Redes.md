# Linux Foundation Certified System Administrator

## Redes

### 1. Configuración de redes y resolución de nombres de forma estática y dinámica (Configure networking and hostname resolution statically or dynamically)

#### IP

**`ip`** es el principal comando para gestionar la configuración de red. Algunos ejemplos de uso del comando **`ip`** son:

    user@ubuntu:/$ ip addr show

    user@ubuntu:/$ ip link set eth0 down
    
    user@ubuntu:/$ ip addr add 192.168.0.2/24 dev eth0

#### NETPLAN

**`netplan`** es la principal herramienta para modificar la configuración de red. Para poder trabajar con **netplan** es necesario modificar el contenido del siguiente fichero:

  `/etc/netplan/00-installer-config.yaml`
  
A continuación se muestra un ejemplo del contenido incluido en los ficheros de configuración de la herramienta **netplan**:

```bash
# This is the network config written by 'subiquity'
network:
    ethernets:
        eth0:
            dhcp4: true
    version: 2
```

Una vez que el contenido del fichero ha sido modificado con la configuración deseada, es necesario aplicar los cambios al sistema mediante los comandos asociados a la herramienta **netplan**:

```bash
user@ubuntu:/$ sudo netplan generate

user@ubuntu:/$ sudo netplan apply
```

#### HOSTNAME

Es posible conocer el nombre asignado a una máquina de las formas siguientes:
  
`user@ubuntu:/$ cat /etc/hostname`
   
`user@ubuntu:/$ hostname`
  
`user@ubuntu:/$ hostnamectl`
  
Para modificar el nombre de la máquina es posible emplear uno de los métodos siguientes:

  Editar el contenido del fichero `/etc/hostname`.
  
  Ejecutar el comando `hostnamectl set-hostname hostname`.

Es necesario reiniciar la máquina para que los cambios sean aplicados.

#### HOSTS

El fichero `/etc/hosts` permite realizar asignaciones estáticas entre nombres de máquina y direcciones IP, siendo este un mecanismo de resolución de nombres previo al sistema DNS. 

A continuación se muestra un contenido de ejemplo para el fichero `/etc/hosts` en una máquina llamada `ubuntu`:

```bash
127.0.0.1 localhost
127.0.1.1 ubuntu

# The following lines are desirable for IPv6 capable hosts
::1     ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
```

El fichero `/etc/resolv.conf` contiene entradas para los servidores DNS que se desean emplear durante el proceso de resolución de nombres.

A continuación se muestra un contenido de ejemplo para el fichero `/etc/resolv.conf`:

```bash  
nameserver 127.0.0.53
options edns0
search mshome.net
```

Es posible añadir mas de una entrada `nameserver`como mecanismo de respaldo, contando asi con una servidor DNS primario y otro servidor DNS secundario.

### 2. Configurar los servicios de red para iniciarse automáticamente en el arranque (Configure network services to start automatically at boot)

#### NETWORK MANAGER

La herramienta `network manager` es un servicio que permite administrar los dispositivos de red y las conexiones, asegurando al conectividad de la máquina siempre que sea posible.

La gestión de la red en los sistemas **Ubuntu** actuales es reponsabilidad del sistema interno `systemd` y de las herramientas y servicios asociados `netplan` y `networkd`. Sin embargo, si la herrmienta `network manager` se encuentra instalada, está tomará el control de todos los dispositivos de red y centralizará su gestión en un fichero de configuración.

Es posible manejar el funcionamiento de `network manager` mediante el comando `systemctl` al igual que se hace con cualquier otro servicio (aunque pueden observarse diferencias en función de la distribución empleada):

* `systemctl start systemd-networkd`
* `systemctl enable systemd-networkd`
* `systemctl stop systemd-networkd`
* `systemctl disable systemd-networkd`
* `systemctl status systemd-networkd`
* `systemctl restart systemd-networkd`

### 3. Implementar filtrados de paquetes (Implement packet filtering)

Un filro de paquetes es un software que examina las cabeceras de los paquetes que se transmiten a través de la red y decide que acciones realizar sobre el paquete completo:

* DROP: descarta el paquete.
* ACCEPT: aceptar el paquete.

De esta manera, al conectar una máquina a una red es posible permitir el tipo de tráfico que se desea permitir o restringir. Las princiaples herramientas para gestionar el filtrado de paquetes en Linux son `firewalld` e `iptables`.

#### FIREWALL

Un cortafuegos es un mecanismo que permite aplicar restricciones a los paquetes que se transmiten a través de la red.

La gestión del cortafuegos en Linux corre a cargo del propio Kernel, y la herramienta de administración asociada es `netfilter`. Adicionalmente, es necesario manejar unas tablas de reglas que permiten establecer las restricciones que se desean aplicar:

* INPUT: reglas que establecen los paquetes que podrán acceder al sistema.
* OUTPUT: reglas que establecen los paquetes que podrán abandonar el sistema.
* FORWARD:
* PREROUTING:
* POSTROUTING:

Las reglas incluidas en las tablas anteriores pueden ser encadenadas, y su evaluación se lleva a cabo de una forma ordenada:

1. Cuando una regla se cumple las demás reglas incluidas en la cadena son descartadas.
2. Si ninguna regla se cumple, se aplican políticas por defecto.
3. Las políticas por defecto son:
    1. ACCEPT
    2. DROP

#### IPTABLES

La herramienta `iptables` permite gestionar el funcionamiento del cortafuegos. Mediante `iptables` es posible crear las reglas y las cadenas de reglas que se emplearan para el procesamiento ordenador de las comunicaciones de red.

De hecho es posible reemplazar el servicio `firewalld` con `iptables` siguiendo el siguiente procedimiento:

1. Comprobar el estado de `iptables`: `sudo systemctl status iptables`
2. Detener `iptables`: `sudo systemctl stop iptables`.
3. Asegurar que el sistema ya no está haciendo uso de `iptables`: `sudo systemctl mask iptables`
4. Comprobar el estado del servicio de cortafuegos: `sudo systemctl status firewalld`
5. Detener el cortafuegos: `sudo systemctl stop firewalld`
6. Deshabilitar el cortafuegos: `sudo systemctl disable firewalld`
7. Eliminar el cortafuegos: `sudo apt-get remove ufw`
8. Iniciar `iptables`: `sudo systemctl start iptables`
9. Habilitar `iptables`: `sudo systemctl enable iptables`

* With this configuration rules must be inserted
* `iptables -P INPUT DROP`
  * Set default policy to DROP for INPUT chain
* iptables rules syntax:
  * `iptables {-A|I} chain [-i/o interface][-s/d ipaddres] [-p tcp|upd|icmp [--dport|--sport nn…]] -j [LOG|ACCEPT|DROP|REJECTED]`
  * `{-A|I} chain`
    * `-A` append as last rule
    * `-I` insert. This require a number after chain that indicate rule position
  * `[-i/o interface]`
    * E.g. `-i eth0` - the package is received (input) on the interface eth0
  * `[-s/d ipaddres]`
    * `-s` Source address. ipaddres can be an address or a subnet
    * `-d` Destination address. ipaddres can be an address or a subnet
  * [-p tcp|upd|icmp [--dport|--sport nn…]]
    * `-p` protocol
    * `--dport` Destination port
    * `--sport` Source port
  * `-j [LOG|ACCEPT|DROP|REJECTED]`
    * `ACCEPT` accept packet
    * `DROP` silently rejected
    * `REJECTED` reject the packet with an ICMP error packet
    * `LOG` log packet. <u>Evaluation of rules isn't blocked.</u>

* E.g.
* `iptables -A INPUT -i lo -j ACCEPT` 
  * Accept all inbound loopback traffic
* `iptables -A OUTPUT -o lo -j ACCEPT`
  * Accept all outbound loopback traffic
* `iptable -A INPUT -p tcp --dport 22 -j ACCEPT`
  * Accept all inbound traffic for tcp port 22
* `iptable -A OUTPUT -m state --state ESTABLISHED,RELATED -j ACCEPT`
  * This is a rule that is used to ACCEPT all traffic generated as a response of an inbound connection that was accepted. E.g. if incoming traffic for web server on port 80 was accepted, this rule permits to response traffic to exit from system without inserting specific rules in OUTPUT chain

* **NOTE** file `/etc/services` contains a list of well know ports with services name

#### FIREWALLD

El servicio `firewalld` emplea la herramienta `iptables` para gestionar las reglas aplicadas por el cortafuegos.

Es posible gestionar el funcionamiento del servicio `firewalld` madiente el comando `firewall-cmd`.

* firewalld is enabled by default in CentOS
* It works with zone, *public* is default zone
* The *zone* is applied to an interface
  * The idea is that we can have safe zone, e.g. bound to an internal interface, and unsafe zone, e.g. bound to external interfaces internet facing
* `firewall-cmd --list-all` show current configuration
  * services -> service that are allowed to use interface
  * ports -> ports that are allowed to use interface
* `firewall-cmd --get-services` shows the list of default services
  * The services are configured in `/urs/lib/firewalld/services`
  * `/urs/lib/firewalld/services` contains xml file with service configuration

* `firewall-cmd --add-service service` add service to current configuration
  * **NOTE**: it isn't a permanent configuration
* `firewall-cmd --reload` reload firewalld configuration
  * **NOTE**: If a service was added with previous command now it is disappeared
* `firewall-cmd --add-service service --permanent`  add service to configuration as permanent
  * **NOTE**: Now if firewalld configuration is reloaded service it is still present
* `firewall-cmd --add-port 4000-4005/tcp` Open TCP ports from 4000 to 4005
* `firewall-cmd --permanent --direct --add-rule ipv4 filter OUTPUT 1 -p tcp -m tcp --dport 80 -j ACCEPT`
  * Add a firewall rule using iptables syntax
  * This add permanently a rule as first in OUTPUT chain to allow connections to TCP destination port 80

### 4. Arrancar, detener y comprobar el estado de los servicios de red (Start, stop, and check the status of network services)

Los servicios de red se controlan de la misma forma que el resto de servicios a través del comando `systemctl`.

Adicionalmente, existen aglunas herramientas que permiten conocer el estado de las comunicaciones de red que tienen lugar en la máquina, destacando la herramienta `netstat`.

Para poder instalar las herramientas `netstat` es necesario ejecutar el siguiente comando:

```bash
sudo apt-get install net-tools
```

Una vez instalado se pueden conocer los puertos abiertos en la máquina, así como el proceso que hace uso de cada puerto.

```bash
sudo netsat -tln
```

### 5. Enrutamiento estático de tráfico IP (Statically route IP traffic)

* `ip route show`
  * Print route
  * Alternative command `route -n`
* `ip route add 192.0.2.1 via 10.0.0.1 [dev interface]`
  * Add route to 192.0.2.1 through 10.0.0.1. Optionally interface can be specified
* To make route persistent, create a *route-ifname* file for the interface through which the subnet is accessed, e.g eth0:
  * `vi /etc/sysconfig/network-scripts/route-eth0`
  * Add line `192.0.2.1 via 10.0.0.101 dev eth0`
  * `service network restart` to reload file

* `ip route add 192.0.2.0/24 via 10.0.0.1 [dev ifname]` 
  * Add a route to subnet 192.0.2.0/24

* To configure system as route forward must be enabled
  * `echo 1 > /proc/sys/net/ipv4/ip_forward`
  * To make configuration persistent
    * `echo net.ipv4.ip_forward = 1 > /etc/sysctl.d/ipv4.conf`

### 6. Sincronización de fecha y hora mediante servicios de red (Synchronize time using other network peers)

* In time synchronization the concept of Stratum define the accuracy of server time.
* A server with Stratum 0 it is the most reliable
* A server synchronized with a Stratum 0 become Stratum 1
* Stratum 10 is reserved for local clock. This means that it is not utilizable
* The upper limit for Stratum is 15
* Stratum 16 is used to indicate that a device is unsynchronized
* Remember that time synchronization between servers is a slowly process

CHRONYD

* Default mechanism to synchronize time in CentOS
* Configuration file `/etc/chrony.conf`
* `server` parameters are servers that are used as source of synchronization
* `chronyc sources` contact server and show them status
* `chronyc tracking` show current status of system clock

NTP

* The old method of synchronization. To enable it Chronyd must be disabled
* Configuration file `/etc/ntp.conf`
* `server` parameters are servers that are used as source of synchronization
* `ntpq -p` check current status of synchronization
