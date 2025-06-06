# Documentación de Instalación de: **Mattermost-team-edition:10.5.4  en Ubuntu Server**

**Requisitos**
Tener instalado Docker version 26.1.3
Tener instalado docker-compose version 1.29.2
Esta instalación se realizo en Ubuntu Server 24.04.2 LTS

## Instalación de los contenedores de mattermost y postgres en Docker
### 1. **Clonar el repositorio oficial de mattermost para docker**: 
    - Usar el siguiente comando:
    git clone https://github.com/mattermost/docker 

    - Entrar en el directorio de Docker con el comando:
        cd docker

    - Comprobar que se haya creado el archivo env.example 
        ls -l
        * Debe aparecer env.example

> **Nota importante**: Si no se creó el archivo env se puede copiar:
        cp env.example .env
        -Puede cambiar el nombre de env.example si se quiere:
        mv env.example env.<CambioDeNombre>
        *Ejemplo: mv env.example env.maggichat

### 2. ** Configurar el archivo env**
> **Nota importante**: En el archivo llamado "Configuración de env" se encuentra una configuración que puede copiarse

    - Entrar a su archivo env para editarlo:
        nano env.example

    - Editar las siguientes lineas:
        DOMAIN=<url de su dominio>
        *Ejemplo: cgnovachat.info o ip
        - TZ=<su zona horaria>
        *Ejemplo: America/Cancun

> **Nota importante**:Si se quiere puede dejar los valores de usuario postgres default

        - Cambiar los valores de usuario de
        postgres:

        POSTGRES_USER=<usuario de postgres>
        # Ejemplo: mmuser
        POSTGRES_PASSWORD=<contraseña del usuario>
        # Ejemplo: mmuser_password

        - Cambiar los siguientes valores según la version que tenga de mattermost:

> **Nota opcional**: si tiene una versión enterprise escriba MATTERMOST_IMAGE=mattermost-enterprise-edition

        MATTERMOST_IMAGE=mattermost-team-edition
        MATTERMOST_IMAGE_TAG=10.5.4

        MATTERMOST_CONTAINER_READONLY=false

> **Nota importante**: read_only: false permite que se puedan escribir datos temporales y también sirve para los logs

        MM_SQLSETTINGS_DATASOURCE=postgres://${POSTGRES_USER}:${POSTGRES_PASSWORD}@postgres:5432/${POSTGRES_DB}?sslmode=disable&connect_timeout=10

### 2.1 ** Configurar el archivo docker-compose.yml **
> **Nota importante**: En el archivo llamado Configuración de docker-compose se ecuentra una configuración que puede copiarse

    - Entrar al archivo docker-compose.yml
        nano docker-compose.yml

    - Editar las siguientes lineas:
        postgres:
            security_opt:
                - "no-new-privileges:false"

> **Nota importante**: no-new-privileges:false permite cambiar el UID/GID para realizar cambios de permisos en rutas

            read_only: false

> **Nota importante**: read_only: false permite que se puedan escribir datos temporales y también sirve para los logs

            environment:
                - TZ=<su zona horaria> 
                # Ejemplo: America/Cancun

    -   Escribir la siguiente linea:
            healthcheck:
                test: ["CMD-SHELL", "pg_isready -U ${POSTGRES_USER} -d ${POSTGRES_DB}"]
                interval: 5s
                timeout: 5s
                retries: 5

        mattermost:
    -   Escribir el siguiente entrypoint:
            entrypoint: sh -c "chmod 755 /entrypoint.sh && /entrypoint.sh mattermost"
    -   Abajo de image escribir los puertos que se van a usar
            ports:
                - "8065:8065"
                - "8444:8443"
                - "80:80"
                - "443:443"

> **Nota importante**: el puerto 8065 es para HTTP interno de mattermost , 8444 para HTTP alternativo (sin proxy), 80 para redirección HTTP a HTTPS y 443 para HTTPS seguro (acceso público)

    -   Editar las siguientes lineas
            security_opt:
                - "no-new-privileges:false"
            environment:
                - TZ=<su zona horaria>
                # Ejemplo: America/Cancun
                - MM_SQLSETTINGS_DRIVERNAME=postgres
                - MM_SQLSETTINGS_DATASOURCE=postgres://${POSTGRES_USER}:${POSTGRES_PASSWORD}@postgres:5432/${POSTGRES_DB}?sslmode=disable&connect_timeout=10
                - MM_BLEVESETTINGS_INDEXDIR=${MM_BLEVESETTINGS_INDEXDIR}
                #MM_SERVICESETTINGS_SITEURL=${MM_SERVICESETTINGS_SITEURL}
                # Ejemplo: https://cgnovachat.info
                # Se comenta MM_SERVICESETTINGS_SITEURL para poder editar la url desde system console y que la url del sitio pueda cambiarse por los administradores

    -   Agregar lo siguiente:
    networks:
        default:
            name: mattermost_network

    ** Los cambios en los archivos env y docker-compose no deben generar conflictos, revisar que la información en el .env sea la misma que en el .yml y no sea contradictoria, por ejemplo: tener en .env DOMAIN=cgnovachat.com y en .yml - MM_SERVICESETTINGS_SITEURL=https://cgnovachat.org

### 3. ** Comprobación de grupos y permisos ** 
    - Comprueba que tu usuario pertenece al grupo docker:
        groups

    * Resultado esperado: docker debe aparecer en la lista
    ** Para agregar tu usuario al grupo docker:
        sudo groupadd docker
        sudo usermod -aG docker $USER (mhadmin)
        newgrp docker  # O cierra y abre la terminal
        sudo reboot

    - Revisar los permisos del socket:
        ls -l /var/run/docker.sock

    * Resultado esperado: srw-rw---- 1 root docker 0 Apr 10 15:58 /var/run/docker.sock
    - Para asignar el socket al grupo docker:
        sudo chown root:docker /var/run/docker.sock
        - Reinicia el servidor:
            sudo systemctl restart docker

### 4. ** Iniciar los contenedores de mattermost y postgres**
    - Para iniciar los contenedores de mattermost y postgres mediante el archivo docker-compose.yml:
        cd docker

> **Nota importante**: el siguiente comando indica a docker-compose cuál es el .env para que detecte las variables

        docker-compose --env-file env.magicchat up -d
    - Comprobar el estado de los contenedores de mattermost y de postgres:
        docker ps

#### Soluciónes al estado restarting en los contenedores 
#### 1 Comprobar los logs por el estado restarting en los contenedores
    - Para saber que causa el estado restarting en los contenedores use los siguientes comandos:

        docker inspect <nombre del contenedor de postgres>
        o
        docker logs <nombre del contenedor de postgres>
        - para mattermost:
        docker inspect <nombre del contenedor de mattermost>
        o
        docker logs <nombre del contenedor de matttermost>

    * Ejemplo   docker inspect docker_mattermost_1/docker_postgres_1
                o
                docker logs docker_mattermost_1/docker_postgres_1
##### Solución por problemas de permisos de lectura en postgres
**Si al realizar dockeer inspect en el contenedor de postgres aparece "ReadonlyRootfs": true**

    - Edita el archivo docker-compose.yml
        nano /home/usuario/docker/docker-compose.yml

    - Edita el permiso de ReadonlyRootfs": true, por lo siguiente: ReadonlyRootfs": false

    - Guarda los cambios:
        En nano: Presiona Ctrl + O, luego Enter para guardar, y Ctrl + X para salir

    - Recrea los contenedores:
        docker-compose down
        docker-compose up -d

    - Verifica el estado de los contenedores:
        docker ps

        - Usa docker logs docker_postgres_1 para revisar los logs y obtener más información si el contenedor de postgres sigue diciendo restarting

##### Solución por problemas de permisos en el contenedor de mattermost 
**Si en los logs del contenedor de mattermost dice que faltan permisos**

    Este error puede producirse por falta de permisos en las rutas a las que necesita acceder mattermost

    - puede deberse a que es un archivo el que no tiene permisos, para ese caso ubique el archivo sin permisos en los logs del contenedor y use:
    cd /ruta/hasta/el/archivo/sin/permisos/
    sudo chown 2000:2000 <nombre del archivo>
    * (2000:2000 es el UID/GID predeterminado de Mattermost)

    - Puede deberse a problemas de permiso en los volumenes montados, se puede arreglar dando permisos con el siguiente comando: 

    sudo chown -R 2000:2000 /ruta/con/falta/de/permisos
    sudo chown -R 2000:2000 /home/mhadmin/mattermost/volumes/app/

    * Puede darle permisos a las rutas desde el archivo docker-compose.yml*

    * Ejemplo: /ruta/hacia/el/documento:rw
                o
                chmod -R 755 /ruta/hacia/el/documento
    * Puede hacer que desde el docker-compose.yml se le asigne un usuario y grupo a los contenedores *
                chown -R 2000:2000 /ruta/
**Si en los logs del contenedor de mattermost dice que no puede conectarse a la base de datos**

    * Este error puede surgir al realizar un restore de la data de postgres
    Si entre los mensajes aparece que el usuario de mattermost ha fallado al intentar conectarse a la base de datos

    -Comprobar que en el archivo env y docker-compose el usuario de postgres sea el mismo
    * Ejemplo: 
    POSTGRES_USER=mmuser
    POSTGRES_PASSWORD=mmuser_password
    POSTGRES_DB=mattermost

    - En config.json comprobar que DataSource use la información del usuario y la base de datos correctamente
        "SqlSettings": {
            "DriverName": "postgres",
            "DataSource": "postgres://<Usuario>:<Contraseña>@<Servicio:puerto>/<base_de_datos>?sslmode=disable\u0026connect_timeout=10\u0026binary_parameters=yes",
        }
    Comprobar que esto este corregido tambien en el archivo env y docker-compose
        "postgres://<Usuario>:<Contraseña>@<Servicio:puerto>/<base_de_datos>?sslmode=disable\u0026connect_timeout=10\u0026binary_parameters=yes"

    * Si el problema persiste puede ser que se hayan borrado las credenciales el usuario mediante el cual mattermost se conecta con postgres
    para solucionarlo basta con cambiarle la contraseña a la de la variable de env
    POSTGRES_PASSWORD=<contraseña>
    Con el contenedor de postgrest en estado up:
    - Comprueba que la contraseña puesta en el env sea correcto
        docker inspect <nombre_contenedor_postgres> | grep -A 10 "POSTGRES_USER"

    - Conectate a postgres con el usuario en env
        docker exec -it <nombre_del_contenedor_de_postgres> psql -U <usuario> -d <base_de_datos>
    * Ejemplo:
        docker exec -it docker_postgres_1 psql -U mmuser -d mattermost 

    - Cambiar la contraseña del usuario de mattermost
        ALTER USER <usuario> WITH PASSWORD '<la_contraseña_puesta_en_el_env_o_yml>';
    - Volver a crear el contenedor
        docker-compose up -d

### 5. **Comprobar que el servicio sea accesible en local**
    - Abrir el navegador e ingresar en la url http://<dirección><puerto> 
    * Ejemplo: http://192.168.1.7:8065