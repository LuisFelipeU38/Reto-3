# ST0263 - Tópicos Especiales de Telemática
#
## Integrantes:
- LUIS FELIPE URQUIJO VARGAS (lfurquijov@eafit.edu.co)

## Profesor
- **Nombre:** ALVARO ENRIQUE OSPINA SANJUAN
- **Correo:** aeospinas@eafit.brightspace.com

# Reto 3 (Aplicación Monolítica con Balanceo y Datos Distribuidos (BD y archivos))

## 1. breve descripción de la actividad

En este reto, desplegaremos una aplicación monolítica usando WordPress en un entorno Docker, con un enfoque en alta disponibilidad y escalabilidad. Utilizaremos Nginx como balanceador de carga para distribuir el tráfico entre dos instancias de WordPress, optimizando así el rendimiento y la fiabilidad. Además, la arquitectura incluirá instancias separadas para la base de datos MySQL y el almacenamiento de archivos mediante NFS, todo alojado en AWS con cinco máquinas virtuales diseñadas para asegurar un funcionamiento continuo y eficiente, además de tener una instancia adicional (bastion host) para acceder a las instancias privadas (Wordpress, base de datos y NFSServer). Este esquema no solo mejora la gestión y el mantenimiento del sistema, sino que también refuerza la seguridad y la capacidad de respuesta frente a altas demandas de usuario.

### 1.1. Que aspectos cumplió o desarrolló de la actividad propuesta por el profesor (requerimientos funcionales y no funcionales)

Aspectos cumplidos de la actividad propuesta por el profesor:

* Aplicación wordpress o app seleccionada dockerizada monolítica en varios nodos que mejore la
disponibilidad de esta aplicación.
* Implementar un balanceador de cargas basado en nginx que reciba el tráfico web https de
Internet con múltiples instancias de procesamiento.
* Tener 2 instancias de procesamiento wordpress o app seleccionada detrás del balanceador de
cargas.
* Tener 1 instancia de bases de datos mysql.
* Tener 1 instancia de archivos distribuidos en NFS.
* Arquitectura propuesta en el documento guía implementada.
* Despliegue correcto con dominio y subdominio propio.

### 1.2. Que aspectos NO cumplió o desarrolló de la actividad propuesta por el profesor (requerimientos funcionales y no funcionales)

* Ninguno

## 2. información general de diseño de alto nivel, arquitectura, patrones, mejores prácticas utilizadas.
Se utilizó la arquitectura presentada a continuación:

[![overview.png](https://i.postimg.cc/q7bf85dG/overview.png)](https://postimg.cc/hJd3nMvJ)

## 3. Descripción del ambiente de desarrollo y técnico: lenguaje de programación, librerias, paquetes, etc, con sus numeros de versiones.

Se utilizó Docker, con sus respectivos docker-compose para contenerizar correctamente las aplicaciones y servicios necesarios, puede encontrar estos archivos en la carpeta `docker` de este repositorio.

## 4. Descripción del ambiente de EJECUCIÓN (en producción) lenguaje de programación, librerias, paquetes, etc, con sus numeros de versiones.

## IP o nombres de dominio en nube o en la máquina servidor.

[![imagen-2024-04-21-230857595.png](https://i.postimg.cc/SxwXrD7c/imagen-2024-04-21-230857595.png)](https://postimg.cc/0z025G4N)

## Descripción y como se configura los parámetros del proyecto

Para configurar el proyecto se deben seguir los siguientes pasos:

### Crear instancia de bastion host

1. Crear una instancia de bastion host en AWS.
2. Crear un par de llaves para acceder a la instancia.
3. Configurar el grupo de seguridad para permitir el acceso a la instancia de bastion host.
4. Acceder a la instancia de bastion host por medio de ssh.

### Crear instancia de balanceador de carga usando Nginx

1. Copia el archivo `docker-compose.yml` y `nginx.conf` en una instancia de balanceador de carga.

2. Configura el archivo `nginx.conf`.

Reemplaza las direcciones IP por las direcciones IP de las instancias de wordpress.

3. Ejecuta el siguiente comando para levantar el servicio de Nginx:

```bash
docker-compose up -d
```
### Crear instancia de Wordpress

1. Copia el archivo `docker-compose.yml` en una instancia de Wordpress.

2. Configura el archivo `docker-compose.yml` con la información.

Reemplaza las variables de entorno por las credenciales de la base de datos.

3. Ejecuta el siguiente comando para levantar el servicio de Wordpress:

```bash
docker-compose up -d
```

### Crear instancia de base de datos

1. Copia el archivo `docker-compose.yml` en una instancia de base de datos.

2. Configura el archivo `docker-compose.yml` con la información:

Reemplaza las variables de entorno por las credenciales de la base de datos.

3. Ejecuta el siguiente comando para levantar el servicio de la base de datos:

```bash
docker-compose up -d
```

### Crear instancia de NFS Server

1. **Instalación de NFS Server**

```bash
sudo apt-get update
sudo apt-get install nfs-kernel-server
```

2. **Configuración de NFS Server**

```bash
sudo mkdir -p var/nfs/worpress
sudo chown nobody:nogroup var/nfs/worpress
sudo chmod 777 var/nfs/worpress
```

3. **Exportación del directorio**

- Añade el directorio a `/etc/exports` con las configuraciones adecuadas para los clientes:
    ```
    /var/nfs/wordpress <IP_clientes>(rw,sync,no_subtree_check)
    ```
- Aplica los cambios: `sudo exportfs -a`.
- Reinicia el servidor NFS: `sudo systemctl restart nfs-kernel-server`.

### Configuración del Cliente NFS en Instancias de WordPress

1. **Instalación del Cliente NFS**:
   - `sudo apt install nfs-common`.
   
2. **Montaje del Directorio NFS**:
- Monta el directorio NFS en el punto de montaje deseado:
    ```
    sudo mkdir -p /var/www/html
    sudo mount <IP_servidor_NFS>:/var/nfs/wordpress /var/www/html
    ```
- Añade la configuración al `/etc/fstab` para montajes automáticos:
    ```
    <IP_servidor_NFS>:/var/nfs/wordpress /var/www/html nfs auto,noatime,nolock,bg,nfsvers=3,intr,tcp,actimeo=1800 0 0
    ```

Esta configuración asegura que las instancias de WordPress puedan compartir de manera eficiente archivos y datos a través del servidor NFS, manteniendo la consistencia entre múltiples servidores para una aplicación escalable y de alta disponibilidad.

## Resultados o pantallazos 

[![imagen-2024-04-21-232233118.png](https://i.postimg.cc/fL7960j2/imagen-2024-04-21-232233118.png)](https://postimg.cc/njzrmrWB)

### Referencias
- [Wordpress with NFS](https://mohsensy.github.io/sysadmin/2020/04/01/wordpress-nfs.html)

- [How To Install TLS/SSL on Docker Nginx Container With Let’s Encrypt](https://macdonaldchika.medium.com/how-to-install-tls-ssl-on-docker-nginx-container-with-lets-encrypt-5bd3bad1fd48)
