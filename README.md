# Atomflare
Repositorio para los manifiestos del trabajo de investigación de la asignatura CIMSI de la Universidad de Sevilla. 
Titulación, Ingeniería Informática Tecnologías Informáticas.

# Endpoints
Acceso a las distintas interfaces de la infraestructura de Atomflare 
- VPN Dashboard: https://wireguard-cimsi.atomflare.net
- Nextcloud: https://cloud-cimsi.atomflare.net
- Portainer: https://portainer-cimsi.atomflare.net
- Pihole Dashboard: https://pihole-cimsi.atomflare.net

# Servicios
## [Nextcloud](https://cloud-cimsi.atomflare.net)
Nextcloud es un servicio de cloud, que ofrece diversas aplicaciones de almacenamiento en la nube, así como aplicaciones colaborativas. Es muy parecido a servicios propietarios como [OneDrive](https://www.microsoft.com/en-us/microsoft-365/onedrive/online-cloud-storage), [Dropbox](https://www.dropbox.com/) o [Google Drive](https://www.google.com/drive/).

Según [el reositorio oficial de Nextcloud](https://github.com/nextcloud) en Atomflare se ha optado por desplegar la versión FPM. Debido a que esta es la más modular, ya que separa el front-end del backend. Permitiendo el despliegue por microservicios. Todo esto, conlleva a múltiples ventajas a la hora de desplegar en entornos de Kubernetes. Ofreciendo ventajas de escalado de la aplicación.

Nextcloud (FPM) en Atomflare se compone de:
- Un núcleo. Este contiene la implementación de WebDav y el backend del sistema.
- Un servidor web Nginx, que se encarga del FrontEnd de Nextcloud.
- Una base de datos SQL. En concreto se desplega Postgresql.
- Una base de datos no relacional, como caché. En concreto Redis. Esta no fue desplegada en este ejemplo, por simplicidad del sistema y al ser característica de sistemas con alto volumen de Lectura-Escritura. No obstante, en la versión original de Nextcloud (https://cloud.atomflare.net) al ser usada por usuarios reales, sí viene con Redis.

Los manifiestos de Nextcloud para kubernetes se componen de despliegue varias replicas de su frontend, con Nginx, y una sola replica (o un stateful-set) para el núcleo de Nextcloud-fpm. Los demás servicios como la base de datos SQL fueron desplegados de forma independiente, en su propio namespace.

Para la versión de prueba de cimsi, las credenciales de acceso son:
```
https://cloud-cimsi.atomflare.net
usuario: atomflare
contraseña: cimsi2022
```

##[VPN](wg-cimsi.atomflare.net)
En Atomflare se usa la VPN para que sea la única forma de acceder al protocolo SSH.
En la máquina actual de Digital Ocean, se ha cambiado el puerto por defecto de SSH, y además, se ha restringido tal tráfico, a que sea exclusivamente acceptado por conexiones provenientes de la VPN.

Entre las distintas opciones de VPN, se ha optado por Wireguard. Al ser un proyecto moderno, con lo que cuenta con los estándares de IP-Sec más actualizados, y es muy ligero para la máquina, debido a que el tráfico pr VPN será el de los administradores únicamente.

El [enlace] lleva a la interfaz gráfica para facilitar la gestión de usuario con acceso a la VPN. Las credenciales de acceso son:
```
https://wg-cimsi.atomflare.net
contraseña: cimsi2022
```

##[Pihole](https://pihole-atomflare.net)
Pihole es el DNS de la organización. El sistema operativo que se ha escogido en la máquina es [Ubunutu Server 22.04 LTS](https://releases.ubuntu.com/22.04/) y éste, entre otros sistemas operativos Linux, vienen con un proceso que resuelve peticiones DNS internas. Este es [systemd-resolved](https://wiki.archlinux.org/title/systemd-resolved).

Se ha procedido a parar y descativar este servicio, porque tiene ocupado el puerto 53 de la máquina imposibilitando usar puerto para otro servicio DNS.

Con el puerto 53 disponible, y `systemd-resolved` desactivado, se puede especificar en al archivo `/etc/resolv.conf` los dns a los cuales debe recurrir la máquina para resolución de nombres. Con Pihole instalado, el contenido de `resolv.conf` queda como a continuación:
```
nameserver 127.0.0.53
nameserver 1.1.1.1
nameserver 1.0.0.1
```
Esto le indica a la máquina que preferentemente siempre recurra al DNS 127.0.0.1 en el puerto 53, donde Pihole estará escuchando, para resolver sus peticiones DNS. Si no funciona, que recurra como backup a `1.1.1.1` de Cloudflare.
