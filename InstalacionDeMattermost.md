# Documentación de Instalación de: **Mattermost-team-edition:10.5.4  en Ubuntu Server**

**Requisitos**
Esta instalación se realizo en Ubuntu Server 24.04.2 LTS
Tener instalado Docker version 26.1.3
Tener instalado docker-compose version 1.29.2

## 1 Instalación de los contenedores de mattermost y postgres en Docker
### 1. **Clonar el repositorio oficial de mattermost para docker**: 
    - Usar el siguiente comando:
    git clone https://github.com/mattermost/docker 
    - Entrar en el directorio de Docker con el comando:
        cd docker
    - Copiar el archivo de ejemplo de variables de entorno:
        cp env.example .env
    - Comprobar que se haya creado el archivo env.example 
    - Cambiar el nombre de env.example:
        mv env.example env.<NombreDeSuDominio>
    * Ejemplo: mv env.example env.maggichat
### 2. ** Configurar los archivos env y docker-compose **
    - Entrar al archivo env para editarlo:
        nano env.magicchat
    - Editar las siguientes lineas:
        DOMAIN=<url de su dominio>
        - TZ=<su zona horaria>
        POSTGRES_USER=<usuario de postgres>
        POSTGRES_PASSWORD=<contraseña del usuario de postgres>
        MATTERMOST_IMAGE=mattermost-team-edition
        MATTERMOST_IMAGE_TAG=10.5.4
        MATTERMOST_CONTAINER_READONLY=false
        MATTERMOST_APP_PORT=8065
        MM_SQLSETTINGS_DATASOURCE=postgres://${POSTGRES_USER}:${POSTGRES_PASSWORD}@postgres:5432/${POSTGRES_DB}?sslmode=disable&connect_timeout=10
        MM_SERVICESETTINGS_SITEURL=<url se su sitio web>
    * Revisar "Configuración de env.magicchat" para ver la configuración actual
    - Entrar al archivo docker-compose.yml
        nano docker-compose.yml
    - Editar las siguientes lineas:
        postgres:
            read_only: false
            enviroment:
                - TZ=<su zona horaria>
        mattermost:
            entrypoint: sh -c "chmod 755 /entrypoint.sh && /entrypoint.sh mattermost"
            image: mattermost/mattermost-team-edition:10.5.4
            ports:
                - "8065:8065"
                - "8444:8443"
                - "80:80"
                - "443:443"
            enviroment:
                - TZ=<su zona horaria>
                - MM_SQLSETTINGS_DATASOURCE=postgres://${POSTGRES_USER}:${POSTGRES_PASSWORD}@postgres:5432/${POSTGRES_DB}?sslmode=disable&connect_timeout=10
                - MM_SERVICESETTINGS_SITEURL=<url de su sitio web>
    networks:
        default:
            name: mattermost_network
    * Revisar "configuración de docker-compose" para ver la configuración actual
    ** Los cambios en los archivos env y docker-compose no deben generar conflictos, revisar que la información en uno sea la misma que en el otro
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
    ** Para asignar el socket al grupo docker:
        sudo chown root:docker /var/run/docker.sock
        - Reinicia el servidor:
            sudo systemctl restart docker
    - Fusiona los siguientes dos archivos de Docker Compose y ejecuta los servicios:
        docker compose -f docker-compose.yml -f docker-compose.without-nginx.yml up -d
    El código anterior debió poner en a correr el contenedor de mattermost y de postgres
    De no ser así realice el siguiente paso
### 4. ** Iniciar los contenedores de mattermost y postgres**
    Docker-compose up -d
    - Comprobar el estado de los contenedores de mattermost y de postgres:
    docker ps
#### Soluciónes al estado restarting en los contenedores 
#### 1 Comprobar los logs por el estado restarting en los contenedores
    Para saber que causa el estado restarting en los contenedores use los siguientes comandos
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
    - Puede deberse a problemas de permiso en los volumenes montados, se puede arreglar dando permisos con el siguiente comando: 
    sudo chown -R 2000:2000 /ruta/con/falta/de/permisos
    sudo chown -R 2000:2000 /home/mhadmin/mattermost/volumes/app/
    * Puede darle permisos a las rutas desde el archivo docker-compose.yml*
    * Ejemplo: /ruta/hacia/el/documento:rw
                o
                chmod -R 755 /ruta/hacia/el/documento
    * Puede hacer que en el docker-compose.yml se le asigne a un usuario y grupo los contenedores *
                chown -R 2000:2000 /ruta/
**Si en los logs del contenedor de mattermost dice que no puede conectarse a la base de datos**
    Si entre los mensajes aparece que el usuario de mattermost ha fallado al intentar conectarse a la base de datos
    -Comprobar que en el archivo env y docker-compose el usuario de postgres sea el mismo
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
    Si el problema persiste puede ser que se hayan borrado las credenciales el usuario mediante el cual mattermost se conecta con postgres
    para solucionarlo basta con cambiarle la contraseña a la de la variable de env
    POSTGRES_PASSWORD=<contraseña>
    Con el contenedor de postgrest en estado up:
    - Conectate a postgres con el usuario en env
        docker exec -it <nombre_del_contenedor> psql -U <usuario> -d <base_de_datos>
    - Cambiar la contraseña del usuario de mattermost
        ALTER USER postgres WITH PASSWORD 'tu_nueva_contraseña_segura';
### 5. **Comprobar que el servicio sea accesible en local**
    - Abrir el navegador e ingresar en la url http://<dirección><puerto> 
    * Ejemplo: http://192.168.1.7:8065