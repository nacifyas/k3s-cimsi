# Atomflare 2 (CIMSI)
Repositorio para los manifiestos del trabajo de investigación de la asignatura CIMSI de la Universidad de Sevilla. 
Titulación, Ingeniería Informática Tecnologías Informáticas.

# Endpoints
Acceso a las distintas interfaces de la infraestructura de Atomflare 
- VPN Dashboard: https://wireguard-cimsi.atomflare.net
- Nextcloud: https://cloud-cimsi.atomflare.net
- Portainer: https://portainer-cimsi.atomflare.net
- Pihole Dashboard: https://pihole-cimsi.atomflare.net/admin

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

## [VPN](https://wg-cimsi.atomflare.net)
En Atomflare se usa la VPN para que sea la única forma de acceder al protocolo SSH.
En la máquina actual de Digital Ocean, se ha cambiado el puerto por defecto de SSH, y además, se ha restringido tal tráfico, a que sea exclusivamente acceptado por conexiones provenientes de la VPN.

Entre las distintas opciones de VPN, se ha optado por Wireguard. Al ser un proyecto moderno, con lo que cuenta con los estándares de IP-Sec más actualizados, y es muy ligero para la máquina, debido a que el tráfico pr VPN será el de los administradores únicamente.

El [enlace](https://wg-cimsi.atomflare.net) lleva a la interfaz gráfica para facilitar la gestión de usuario con acceso a la VPN. Las credenciales de acceso son:
```
https://wg-cimsi.atomflare.net
contraseña: cimsi2022
```

## [Pihole](https://pihole-cimsi.atomflare.net/admin)
Pihole es el DNS de la organización. El sistema operativo que se ha escogido en la máquina es [Ubunutu Server 22.04 LTS](https://releases.ubuntu.com/22.04/) y éste, entre otros sistemas operativos Linux, vienen con un proceso que resuelve peticiones DNS internas. Este es [systemd-resolved](https://wiki.archlinux.org/title/systemd-resolved).

Se ha procedido a parar y descativar este servicio, porque tiene ocupado el puerto 53 de la máquina imposibilitando usar puerto para otro servicio DNS.

Con el puerto 53 disponible, y `systemd-resolved` desactivado, se puede especificar en al archivo `/etc/resolv.conf` los dns a los cuales debe recurrir la máquina para resolución de nombres. Con Pihole instalado, el contenido de `resolv.conf` queda como a continuación:
```
nameserver 127.0.0.53
nameserver 1.1.1.1
nameserver 1.0.0.1
```
Esto le indica a la máquina que preferentemente siempre recurra al DNS 127.0.0.1 en el puerto 53, donde Pihole estará escuchando, para resolver sus peticiones DNS. Si no funciona, que recurra como backup a `1.1.1.1` de Cloudflare.
Se pueden añadir tantos servidores backup como se desee.

Pihole es un [DNS Sinkhole](https://en.wikipedia.org/wiki/DNS_sinkhole) sin llegar a ser un DNS como tal. Porque más bien, sería como un DNS Proxy, debido a que Pihole en sí, no puede resolver nombres de dominio, sino, pasa la petición a otro DNS superior (Upstream DNS) con la diferencia, de que se disponer de una interfaz, con la cual se puede decidir, qué nombres resolver y cuales bloquear.

Pihole también, puede servir como un pequeño DNS local.

Más información sobre sus funcionalidades figuran en la [documentación oficial](https://docs.pi-hole.net/).

Accediendo a la dashboard, la clave es:
```conf
https://pihole-cimsi.atomflare.net/admin
contraseña: cimmsi2022
```
Pihole permite obersvar, gestionar y monotorizar el todo el tráfico DNS que se realiza en Atomflare. Pudiendo selectivamente bloquear ciertos dominios y ofreciendo más control que `systemd-resolved`.

Para el despliegue de Pihole se ha tenido en cuenta la privacidad y la seguridad del sistema. Por ello, el tráfico DNS está limitado por firewall solo para el acceso local (vpn y localhost). Y como upstream DNS, se ha desplegado [Unbound](https://www.nlnetlabs.nl/projects/unbound/about/) un servicio DNS autónomo que accede directamente a los servidores autoritativos, sin la necesidad de exponer el tráfico a un proveedor intermediario como Google (`8.8.8.8` y `8.8.4.4`) o Cloudflare (`1.1.1.1` y `1.0.0.1`).

En caso de que Pihole no funcione, o haga falta mantenimiento, no se pierde DNS del sistema. Ya que según se ah configurado en el `/etc/resolv.conf`, se recurrirá automáticamente a otros DNS en caso de emergencia.

## [Portainer](https://portainer-cimsi.atomflare.net)
Portainer es una interfaz para entornos de contenerización. Originlmente fue diseñado para Docker, y en sus versiones recientes, incorpora ventajas para entornos de Kuberntes.

Es una interfaz que ofrece una variedad de ventajas, y que se usan tanto en Atomflare real, como en este trabajo de investigación.

Para empezar, Portainer ofrece control de usuarios con roles, con lo que se puede distribuir la gestión y el mantenimiento del sistema, sin ofrece credenciales SSH ni túneles VPN, permitiendo el acceso regulado desde la UI por (casi) cualquier navegador web.

Además, nos permite hacer desplegues, elminar o crear recursos con facilidad y flexibilidad. Cuenta con un editor integrado online, con corrector de sintaxis, para poner los manifiestos desde la interfaz. Pero lo más brillante, es que nos permite usar la técnica Gitops con facilidad. Así podremos tener un control de versiones de los manifiestos, y poder desplegarlos de forma mucho más organizada y automática.

# Otros Aspectos

## Distribución de escalabilidad
Destaca mencionar que la infraestructura fue creada teniendo en mente, su escalabilidad. En concreto, escalado horizontal, añadiendo nuevos nodos al sistema. Para tal caso, es conveniente que al integrar nuevos nodos, éstos distribuyan la carga de los pods en ejecución.

Para ello, se ha ofrecido una heurística basada en etiquetas en los manifiestos. Por experiencia, se sabe que ciertos pods, como Nextcloud, consumen muchos recursos. Por ello, se le ha colocado la etiqueta `load: high` a los pods que más recursos consumen. Teniendo estos pods identificados, se ha implementado una política de ["Pod Topology Spread"](https://kubernetes.io/docs/concepts/scheduling-eviction/topology-spread-constraints/)
```yaml
topologySpreadConstraints:
- maxSkew: 1
  topologyKey: kubernetes.io/hostname
  whenUnsatisfiable: ScheduleAnyway
  labelSelector:
    matchLabels:
      load: high
```
Aquellas aplicaciones que tengan esta configuración en su manifiesto, van a hacer que el [Scheduler](https://kubernetes.io/docs/concepts/scheduling-eviction/kube-scheduler/) minimice el número de pods con la etiqueta `load:high` en el mismo nodo, guíando así a la distribución de pods de forma equitativa por los nodos.

## Recuperación de servicios
Al menos en la distrivución [k3s](https://docs.k3s.io/), si cae un nodo con pods en él, este espera durante 300 segundos hasta comenzar una evicción de los pods. Así para los pods más críticos, se ha establecido una política que reduce ese tiempo significativamente (como para el caso de VPN)
```yaml
tolerations:
- key: "node.kubernetes.io/unreachable"
  operator: "Exists"
  effect: "NoExecute"
  tolerationSeconds: 20
- key: "node.kubernetes.io/not-ready"
  operator: "Exists"
  effect: "NoExecute"
  tolerationSeconds: 20
```
