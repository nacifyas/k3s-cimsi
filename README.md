### Atomflare
Repositorio para los manifiestos del trabajo de investigación de la asignatura CIMSI de la Universidad de Sevilla. 
Titulación, Ingeniería Informática Tecnologías Informáticas.

### Endpoints
Acceso a las distintas interfaces de la infraestructura de Atomflare 
- VPN Dashboard: https://wireguard-cimsi.atomflare.net
- Nextcloud: https://cloud-cimsi.atomflare.net
- Portainer: https://portainer-cimsi.atomflare.net
- Pihole Dashboard: https://pihole-cimsi.atomflare.net

### Servicios
## [Nextcloud](https://cloud-cimsi.atomflare.net)
Nextcloud es un servicio de cloud, que ofrece diversas aplicaciones de almacenamiento en la nube, así como aplicaciones colaborativas. Es muy parecido a servicios propietarios como [OneDrive](https://www.microsoft.com/en-us/microsoft-365/onedrive/online-cloud-storage), [Dropbox](https://www.dropbox.com/) o [Google Drive](https://www.google.com/drive/).

Según [el reositorio oficial de Nextcloud](https://github.com/nextcloud) en Atomflare se ha optado por desplegar la versión FPM. Debido a que esta es la más modular, ya que separa el front-end del backend. Permitiendo el despliegue por microservicios. Todo esto, conlleva a múltiples ventajas a la hora de desplegar en entornos de Kubernetes. Ofreciendo ventajas de escalado de la aplicación.

Nextcloud (FPM) en Atomflare se compone de:
- Un núcleo. Este contiene la implementación de WebDav y el backend del sistema.
- Un servidor web Nginx, que se encarga del FrontEnd de Nextcloud.
- Una base de datos SQL. En concreto se desplega Postgresql.
- Una base de datos no relacional, como caché. En concreto Redis. Esta no fue desplegada en este ejemplo, por simplicidad del sistema y al ser característica de sistemas con alto volumen de Lectura-Escritura. No obstante, en la versión original de Nextcloud (https://cloud.atomflare.net) al ser usada por usuarios reales, sí viene con Redis.

Los manifiestos de Nextcloud para kubernetes se componen de despliegue varias replicas de su frontend, con Nginx, y una sola replica (o un stateful-set) para el núcleo de Nextcloud-fpm. Los demás servicios como la base de datos SQL fueron desplegados de forma independiente, en su propio namespace.

