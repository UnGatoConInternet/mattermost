# Documentación de: **Configuración de Mattermost**
## 2. Configurar el servicio de mattermost
## (cualquier modificación debe realizarse tanto en el servicio de mattermost desde
##  System Console y en config.json)
### Configurar Web Server
    
#### "Please check connection, Mattermost unreachable..."
    Si este mensaje aparece en la parte superior de mattermost en color rojo
    el error puede deberse a que esta mal escrito o no se ha escrito la url del sitio
    en la configuración de mattermost
    Comprueba tanto en el archivo env como en docker-compose.yml y en config.json que
    la url de tu dominio este bien escrito
    En env.magicchat:
        MM_SERVICESETTINGS_SITEURL=https://cgnovachat.info
    En docker-compose.yml:
        - MM_SERVICESETTINGS_SITEURL=https://cgnovachat.info
    En config.json:
        "SiteURL": "https://cgnovachat.info"

## Creación de un tunel mediante cloudflare ##
6. **Instalación de cloudflare para**

9. **Correr el tunel**
    -Usar el siguiente comando
    cloudflared tunnel run tunelprueba

10. **Link para registrar usuarios**

11. **Correr el tunel creado conn docker compose** Corregir
    - Editar el archivo docker-compose.yml:
        cd docker
        nano docker-compose.yml
    - Otorga permisos para que docker pueda leer el certificado de cloudflare
        - chmod 644 ~/.cloudflared/cert.pem  # -rw-r--r--
    - Agregar el servicio de cloudflared con el comando para correr el tunel:
    services:
    cloudflared:
        image: cloudflare/cloudflared:latest
        container_name: cloudflared-tunnel
        restart: unless-stopped
        volumes:
        - ./config:/etc/cloudflared  # Carpeta local para persistir credenciales/config
        command: tunnel --loglevel debug run tunelprueba

## Crear un volumen donde se guarde la información para hacer un volumen persistente ## 
    - Crear el volumen de postgres
        - docker volume create <NombreDelVolumen>
        - Ejemplo docker volume create db_posgres_test1

    - Correr el siguiente comando (cambiar db_posgres_test1 con el nombre del volumen que creaste)
        - docker run -d --name db_posgres_test1 --mount type=volume,src=db_posgres_test1,dst=/data/db postgres:16

    - Inspeccionar el volumen creado
        - docker volume inspect db_posgres_test1
        - Resultado esperado:
            [
                {
                    "CreatedAt": "2025-04-21T16:56:39Z",
                    "Driver": "local",
                    "Labels": null,
                    "Mountpoint": "/var/lib/docker/volumes/db_posgres_test1/_data",
                    "Name": "db_posgres_test1",
                    "Options": null,
                    "Scope": "local"
                }
            ]
    - Detener y eliminar el contenedor de postgres
        - docker stop mattermost-test_db-test_1
        - docker rm mattermost-test_db-test_1
    - Volver a crear el contenedor de docker con el volumen persistente
        - docker run -d --name mattermost-test_db-test_1 --mount type=volume,src=db_posgres_test1,dst=/var/lib/postgresql/data -e POSTGRES_DB=mattermost -e POSTGRES_USER=mmuser -e POSTGRES_PASSWORD=mmuser-password -v /backups:/backups --network docker_default --health-cmd="pg_isready -U mmuser" --health-interval=10s --health-timeout=5s --health-retries=3 postgres:13-alpine

## Crear un respaldo de un volumen ##
- Crear una carpeta en la que guardar los respaldos
    - mkdir Nombre_de_carpeta
- Usar el sigueinte comando
    - docker run --rm -v <VolumenNombre>:/data -v $(pwd):/backup alpine tar czvf /backup/<NombreDelBackup>$(date +%Y-%m-%d).tar.gz /data
    - Ejemplo:
        docker run --rm -v db_postgres_data:/data -v $(pwd):/backup alpine tar czvf /backup/db_postgres_data_backup_$(date +%Y-%m-%d).tar.gz /data
        o
        docker run --rm -v db_postgres_data:/data -v $(pwd):/backup alpine tar czvf /backup/db_postgres_data_backup_$(date +%Y-%m-%d).tar.gz /data
        o
        docker run --rm -v db_postgres_data:/data -v $(pwd):/backup alpine tar czvf /backup/db_postgres_data_backup_$(date +%Y-%m-%d).tar.gz /data

**Para restaurar**
    - Detener los contenedores
        -Ejemplo: docker stop mattermost-test_db-test_1
    - Ejecutar el siguiente comando para restaurar un volumen
        - docker run --rm \
            -v <VolumenNombre>:/data \
            -v /dirrecion/a/<NombreDeLaCarpetaDeLosBackups>:/backup \
            alpine sh -c "rm -rf /data/* && tar xzvf /backup/<NombreDelBackup> -C /data --strip-components=1"
        - Ejemplo
            docker run --rm \
            -v db_postgres_data:/data \
            -v /home/mhadmin/backups_mattermost:/backup \
            alpine sh -c "rm -rf /data/* && tar xzvf /backup/db_postgres_data_backup_2025-04-21.tar.gz -C /data --strip-components=1"

        - Ejemplo
            docker run --rm \
            -v db_postgres_data:/data \
            -v /home/mhadmin/backups_mattermost:/backup \
            alpine sh -c "rm -rf /data/* && tar xzvf /backup/db_postgres_data_backup_2025-05-07.tar.gz -C /data --strip-components=1"

## Crear un respaldo automatizado mediante cron ##
    - Crear un script para el comando
        sudo mkdir -p /opt/backup-scripts
    - Editar el archivo que tendrá el script
        sudo nano /opt/backup-scripts/<NombreDelArchivo>
        Ejemplo:
        sudo nano /opt/backup-scripts/mattermost_postgres_backup.sh
    - Editar con la siguiente información:
        #!/bin/bash
        # Directorio donde se guardarán los backups
        BACKUP_DIR="/home/mhadmin/backups_mattermost"  # Cambia esto a tu ruta deseada
        # Crear el directorio si no existe
        mkdir -p "$BACKUP_DIR"
        # Nombre del archivo de backup con fecha
        BACKUP_FILE="db_postgres_data_backup_$(date +%Y-%m-%d).tar.gz"
        # Ejecutar el comando de docker para hacer el backup
        docker run --rm \
        -v db_postgres_data:/data \
        -v "$BACKUP_DIR":/backup \
        alpine tar czvf "/backup/$BACKUP_FILE" /data
        # Opcional: Eliminar backups antiguos (ejemplo: conservar solo últimos 7 días)
        find "$BACKUP_DIR" -name "db_postgres_data_backup_*.tar.gz" -mtime +7 -exec rm {} \;
    - Hacer el script ejecutable dandole permisos:
        sudo chmod +x /opt/backup-scripts/mattermost_postgres_backup.sh
    - Configura el cron:
        sudo crontab -e
    - Agrega la siguiente línea al final para que el script se ejecute diariamente a las 2 AM
        0 2 * * * /opt/backup-scripts/mattermost_postgres_backup.sh > /var/log/mattermost_backup.log 2>&1
    

## Crear un volumen para mattermost_config ##
    - #!/bin/bash
        # Directorio donde se guardarán los backups de mattermost_config
        MATTERCONFIG_BACKUP_DIR="/home/mhadmin/backups_mattermost/mattermost_config"  # Cambia esto a tu ruta deseada
        # Crear el directorio si no existe
        mkdir -p "$MATTERCONFIG_BACKUP_DIR"
        # Nombre del archivo de backup con fecha
        MATTERCONFIG_BACKUP_FILE="mattermost_config_backup_$(date +%Y-%m-%d).tar.gz"
        # Ejecutar el comando de docker para hacer el backup
        docker run --rm \
        -v mattermost_config:/config \
        -v "$MATTERCONFIG_BACKUP_DIR":/backup \
        alpine tar czvf "/backup/$MATTERCONFIG_BACKUP_FILE" /config
        # Opcional: Eliminar backups antiguos (ejemplo: conservar solo últimos 3 días)
        find "$MATTERCONFIG_BACKUP_DIR" -name "mattermost_config_backup_*.tar.gz" -mtime +3 -exec rm {} \;

## Crear un volumen para mattermost_data ##    
    - #!/bin/bash
        # Directorio donde se guardarán los backups de mattermost_data
        MATTERDATA_BACKUP_DIR="/home/mhadmin/backups_mattermost/mattermost_data"  # Cambia esto a tu ruta deseada
        # Crear el directorio si no existe
        mkdir -p "$MATTERDATA_BACKUP_DIR"
        # Nombre del archivo de backup con fecha
        MATTERDATA_BACKUP_FILE="mattermost_data_backup_$(date +%Y-%m-%d).tar.gz"
        # Ejecutar el comando de docker para hacer el backup
        docker run --rm \
        -v mattermost_data:/data \
        -v "$MATTERDATA_BACKUP_DIR":/backup \
        alpine tar czvf "/backup/$MATTERDATA_BACKUP_FILE" /data
        # Opcional: Eliminar backups antiguos (ejemplo: conservar solo últimos 3 días)
        find "$MATTERDATA_BACKUP_DIR" -name "mattermost_data_backup_*.tar.gz" -mtime +3 -exec rm {} \;

## Cómo hacer que se realice una restauración automaticamente ##
    - Editar el archivo que tendrá el script
        sudo nano /opt/backup-scripts/<NombreDelArchivo>
        Ejemplo:
        sudo nano /opt/backup-scripts/restore_mattermost.sh
    - Editar con la siguiente información:
        #!/bin/bash
        docker run --rm \
            -v mattermost_data:/data \
            -v /home/mhadmin/backups_mattermost/mattermost_data:/backup \
            alpine sh -c "rm -rf /data/* && tar xzvf /backup/mattermost_data_backup_$(date +\%Y-\%m-\%d).tar.gz -C /data --strip-components=1"

        docker run --rm \
            -v mattermost_config:/config \
            -v /home/mhadmin/backups_mattermost/mattermost_config:/backup \
            alpine sh -c "rm -rf /config/* && tar xzvf /backup/mattermost_config_backup_$(date +\%Y-\%m-\%d).tar.gz -C /config --strip-components=1"
        
        docker run --rm \
            -v db_postgres_data:/data \
            -v /home/mhadmin/backups_mattermost:/backup \
            alpine sh -c "rm -rf /data/* && tar xzvf /backup/db_postgres_data_backup_$(date +\%Y-\%m-\%d).tar.gz -C /data --strip-components=1"

        - Ejemplo
            docker run --rm \
            -v db_postgres_data:/data \
            -v /home/mhadmin/backups_mattermost:/backup \
            alpine sh -c "rm -rf /data/* && tar xzvf /backup/db_postgres_data_backup_2025-05-07.tar.gz -C /data --strip-components=1"
    - Hacerlo ejecutable
        sudo chmod +x /opt/backup-scripts/restore_mattermost.sh
    - Editar el cron
        sudo crontab -e
    - Añadir el siguiente texto al final
        # Restauración diaria a las 2:30 AM (30 minutos después para asegurar que el backup terminó)
        30 2 * * * /home/mhadmin/restore_mattermost.sh

## Cómo hacer que el tunel se mantenga de forma persistente ##
    - Verificar que cloudflared este instalado correctamente
        which cloudflared
    - Crear un archivo de servicio para systemd
        sudo nano /etc/systemd/system/cloudflared.service
    - Adaptar el siguiente texto:
        [Unit]
        Description=Cloudflare Tunnel
        After=network.target

        [Service]
        Type=simple
        # Usuario bajo el que se ejecuta (puede ser 'root' o un usuario dedicado)
        User=root
        # Comando para iniciar el túnel (reemplaza 'tunelprueba' con el nombre de tu túnel)
        ExecStart=/usr/local/bin/cloudflared tunnel run tunelprueba
        # Reinicia automáticamente si falla
        Restart=always
        # Tiempo de espera antes de reiniciar (5 segundos)
        RestartSec=5
        # Opcional: Variables de entorno (ejemplo para configuración con credenciales)
        # Environment=CLOUDFLARED_ORIGIN_CERT=/ruta/a/tu/cert.pem

        [Install]
        WantedBy=multi-user.target

    - Ejemplo:
        [Unit]
        Description=Cloudflare Tunnel
        After=network.target

        [Service]
        Type=simple
        User=mhadmin
        ExecStart=/usr/local/bin/cloudflared tunnel --config /home/mhadmin/.cloudflared/config.yml run tunelprueba
        Restart=always
        RestartSec=5

        [Install]
        WantedBy=multi-user.target
    - Hacer que el sistema reconozca el nuevo servicio
        sudo systemctl daemon-reload
    - Habilitar el servicio para que se inicie automaticamente al arrancar el servidor
        sudo systemctl enable cloudflared
    - Iniciar el servicio automaticamente
        sudo systemctl start cloudflared
    - Verificar que el servicio este funcionando
        sudo systemctl status cloudflared
        - Debe aparecer el texto active (running)
    - Comandos útiles para administrar el servicio
        Comando	                            Descripción
        sudo systemctl start cloudflared	Inicia el servicio manualmente
        sudo systemctl stop cloudflared	    Detiene el servicio
        sudo systemctl restart cloudflared	Reinicia el servicio
        sudo systemctl status cloudflared	Muestra el estado actual
        sudo journalctl -u cloudflared -f	Muestra los logs en tiempo real
## Solucion a SMTP ##
    - Entrar en la ruta: /docker/volumes/app/mattermost/config
    - Comprobar que se encuentre dentro el archivo  config.json
        ls -l
    - Editar el archivo
        sudo nano config.json
    - Bajar hasta ver el siguiente apartado: "EmailSettings"
    - Editarlo con la siguiente información:
        "EmailSettings": {
        "EnableSignUpWithEmail": true,
        "EnableSignInWithEmail": true,
        "EnableSignInWithUsername": true,
        "SendEmailNotifications": true,
        "UseChannelInEmailNotifications": false,
        "RequireEmailVerification": false,
        "FeedbackName": "Mattermost",
        "FeedbackEmail": "cgnova.kitsu@gmail.com",
        "ReplyToAddress": "cgnova.kitsu@gmail.com",
        "FeedbackOrganization": "",
        "EnableSMTPAuth": false,
        "SMTPUsername": "cgnova.kitsu@gmail.com",
        "SMTPPassword": "T6rp3Bct2wEAHq18",
        "SMTPServer": "smtp-relay.sendinblue.com",
        "SMTPPort": "587",
        "SMTPServerTimeout": 10,
        "ConnectionSecurity": "STARTTLS",
        "SendPushNotifications": false,
        "PushNotificationServer": "",
        "PushNotificationContents": "full",
        "PushNotificationBuffer": 1000,
        "EnableEmailBatching": false,
        "EmailBatchingBufferSize": 256,
        "EmailBatchingInterval": 30,
        "EnablePreviewModeBanner": true,
        "SkipServerCertificateVerification": false,
        "EmailNotificationContentsType": "full",
        "LoginButtonColor": "#0000",
        "LoginButtonBorderColor": "#2389D7",
        "LoginButtonTextColor": "#2389D7"
    }
    - Guardar con: 
        ctrl + O
        Enter
    - Salir con:
        ctrl + X
    - Entrar a mattermost: https://cgnovachat.info
    - Ir a System Console > SMTP
    - Poner la siguiente información:
        SMTP Server:                smtp-relay.sendinblue.com
        SMTP Servver Port:          587
        Enable SMTP Authentication: True
        SMTP Server Username:       cgnova.kitsu@gmail.com
        SMTP Server Password:       <La contraseña dada por sendinblue.com>
        Connection Security:        STARTTLS
        Skip Server...:             True
        Enable Security Alerts:     True
    - Guardar los cambios en SAVE
    - Ir a System Console > Notifications
    - Poner la siguiente información:
        Enable Email Notifications:     True
        Enable Email Batching:          True
        Notification Display Name:      Mattermost
        Notification From Address:      cgnova.kitsu@gmail.com
        Support Email Address:          cgnova.kitsu@gmail.com
        Notification Reply-To Address:  cgnova.kitsu@gmail.com

## comprobar que datasource tenga los mismos valores que el archivo .env ##
Ejemplo de valores que no coinciden:
en config.json
"SqlSettings": {
        "DriverName": "postgres",
        "DataSource": "postgres://mmuser:mostest@localhost/mattermost_test?sslmode=disable\u0026connect_timeout=10\u0026binary_parameters=yes",

        usuario:mmuser
        contraseña:mostest
        nombre de la base de datos:mattermost_test
        servicio y puerto:localhost

en env.magicchat
usuario:mmuser
contraseña:mmuser_password
DB:mattermost
servicio y puerto:postgres:5432

Los valores de .env deben ser los que se usen config.json
Ejemplo de como debe estar según el .env del ejemplo anterior
"SqlSettings": {
  "DataSource": "postgres://mmuser:mmuser_password@postgres:5432/mattermost?sslmode=disable&connect_timeout=10\u0026binary_parameters=yes",

## Si el contenedor de docker muestra "Restarting" y la contraseña para conectarse a postgres es incorrecta ##
Es probable que las credenciales del usuario con el que mattermost entraba a postgres se hayan eliminado
- Comprueba que la contraseña puesta en el env.magicchat sea correcto
    docker inspect <nombre_contenedor_mattermost> | grep -A 10 "POSTGRES_PASSWORD"
- Ingresar a postgres con el usuario que esta en env.magicchat
    docker exec -it docker_postgres_1 psql -U mmuser -d mattermost -c "\du"
- Remplaza Cambia la contraseña del usuario con la que sale en env.magicchat
    ALTER USER mmuser WITH PASSWORD 'nueva_contraseña';
Reinicia los servicios
    docker-compose down && docker-compose up -d

## Actualización de instrucciones para realizar un Restore de data (manual) ##
- Crear un directorio temporal en el que se pondrá el backup descomprimido
    mkdir -p TEMPORAL_DIR_PARA_RESTAURACION
- Descomprimir el backup en el directorio temporal
    #!/bin/bash

    - Definición de rutas
    BACKUP_FILE="/ruta/al/backup/db_postgres_data_backup_2025-05-07.tar.gz"
    TEMPORAL_DIR="/ruta/al/directorio/temporal/TEMPORAL_DIR_PARA_RESTAURACION"

    1. Crear directorio temporal (si no existe)
    echo "Creando directorio temporal..."
    mkdir -p "$TEMPORAL_DIR"

    2. Descomprimir el backup en el directorio temporal
    echo "Descomprimiendo el respaldo..."
    tar -xzf "$BACKUP_FILE" -C "$TEMPORAL_DIR"

    3. Verificar contenido
    echo "Contenido descomprimido:"
    ls -l "$TEMPORAL_DIR"

    echo "Descompresión completada en $TEMPORAL_DIR"

    # Ejemplo #
    #!/bin/bash

    # Definición de rutas
    BACKUP_FILE="/home/mhadmin/mnt/carpeta_compartida/postgres_data/db_postgres_data_backup_2025-05-15.tar.gz"
    TEMPORAL_DIR="/home/mhadmin/TEMPORAL_DIR_PARA_RESTAURACION"

    # Crear directorio temporal (si no existe)
    echo "Creando directorio temporal..."
    mkdir -p "$TEMPORAL_DIR"

    # Descomprimir el backup en el directorio temporal
    echo "Descomprimiendo el respaldo..."
    tar -xzf "$BACKUP_FILE" -C "$TEMPORAL_DIR"

    # Verificar contenido
    echo "Contenido descomprimido:"
    ls -l "$TEMPORAL_DIR"

    echo "Descompresión completada en $TEMPORAL_DIR"

    # Restaurar desde el directorio temporal#

    #!/bin/bash

    # Definición de rutas
    TEMPORAL_DATA_DIR="/ruta/al/directorio_temporal/TEMPORAL_DIR_PARA_RESTAURACION/data"
    RESTORE_PATH="/ruta/al/data/de/postgres/data"

    # 1. Verificar datos en directorio temporal
    echo "Verificando datos en directorio temporal..."
    if [ ! -d "$TEMPORAL_DATA_DIR" ]; then
        echo "Error: No se encuentra el directorio $TEMPORAL_DATA_DIR"
        exit 1
    fi

    if [ -z "$(ls -A "$TEMPORAL_DATA_DIR")" ]; then
        echo "Error: El directorio data está vacío"
        exit 1
    fi

    # 2. Preparar directorio de destino (con sudo)
    echo "Preparando directorio de destino..."
    echo "Intentando listar contenido (puede requerir contraseña sudo)..."
    sudo ls -l "$RESTORE_PATH" || {
        echo "No se pudo verificar el contenido del directorio destino"
        exit 1
    }

    read -p "¿Estás seguro de borrar el contenido existente? (y/n) " -n 1 -r
    echo
    if [[ ! $REPLY =~ ^[Yy]$ ]]; then
        echo "Operación cancelada por el usuario"
        exit 1
    fi

    # 3. Limpiar y restaurar con permisos elevados
    echo "Limpiando directorio destino (requiere privilegios)..."
    sudo rm -rf "${RESTORE_PATH}"/*

    echo "Restaurando archivos de la base de datos..."
    sudo cp -r "${TEMPORAL_DATA_DIR}"/* "$RESTORE_PATH/"

    # 4. Ajustar permisos (para PostgreSQL en Docker)
    echo "Ajustando permisos..."
    sudo chown -R 999:999 "$RESTORE_PATH"
    sudo chmod -R 700 "$RESTORE_PATH"

    # 5. Verificación final
    echo "Verificando la restauración..."
    sudo ls -l "$RESTORE_PATH"

    echo "Restauración completada con éxito!"
    echo "Revisa los archivos listados arriba antes de iniciar los servicios"

    # Ejemplo #
    #!/bin/bash

    # Definición de rutas
    TEMPORAL_DATA_DIR="/home/mhadmin/TEMPORAL_DIR_PARA_RESTAURACION/data"
    RESTORE_PATH="/home/mhadmin/docker/volumes/db/var/lib/postgresql/data"

    1. Verificar datos en directorio temporal
    echo "Verificando datos en directorio temporal..."
    if [ ! -d "$TEMPORAL_DATA_DIR" ]; then
        echo "Error: No se encuentra el directorio $TEMPORAL_DATA_DIR"
        exit 1
    fi

    if [ -z "$(ls -A "$TEMPORAL_DATA_DIR")" ]; then
        echo "Error: El directorio data está vacío"
        exit 1
    fi

    2. Preparar directorio de destino (con sudo)
    echo "Preparando directorio de destino..."
    echo "Intentando listar contenido (puede requerir contraseña sudo)..."
    sudo ls -l "$RESTORE_PATH" || {
        echo "No se pudo verificar el contenido del directorio destino"
        exit 1
    }

    read -p "¿Estás seguro de borrar el contenido existente? (y/n) " -n 1 -r
    echo
    if [[ ! $REPLY =~ ^[Yy]$ ]]; then
        echo "Operación cancelada por el usuario"
        exit 1
    fi

    3. Limpiar y restaurar con permisos elevados
    echo "Limpiando directorio destino (requiere privilegios)..."
    sudo rm -rf "${RESTORE_PATH}"/*

    echo "Restaurando archivos de la base de datos..."
    sudo cp -r "${TEMPORAL_DATA_DIR}"/* "$RESTORE_PATH/"

    4. Ajustar permisos (para PostgreSQL en Docker)
    echo "Ajustando permisos..."
    sudo chown -R 999:999 "$RESTORE_PATH"
    sudo chmod -R 700 "$RESTORE_PATH"

    5. Verificación final
    echo "Verificando la restauración..."
    sudo ls -l "$RESTORE_PATH"

    echo "Restauración completada con éxito!"
    echo "Revisa los archivos listados arriba antes de iniciar los servicios"

    # Nota #
    - Se puede verificar la ruta de un volumen con el siguiente comando:
    docker inspect <nombre_del_contenedor> | grep "Source"
    Ruta de la data de postgres:
    /home/mhadmin/docker/volumes/db/var/lib/postgresql/data
    Ruta de la configuración de mattermost:
    /home/mhadmin/docker/volumes/app/mattermost/config
    Ruta de la data de mattermost:
    /home/mhadmin/docker/volumes/app/mattermost/data

    # Ruta de los backups #
    - Se puede buscar un archivo con el siguiente comando
    find / -name "nombre_del_archivo" 2>/dev/null
    Ruta de los backups de postgres data
    /home/mhadmin/backups_mattermost/<nombre_del_respaldo_comprimido>
    Ruta de los backups de mattermost data
    /home/mhadmin/backups_mattermost/mattermost_data/<nombre_del_respaldo_comprimido>
    Ruta de los backups de mattermost config
    /home/mhadmin/backups_mattermost/mattermost_config/<nombre_del_respaldo_comprimido>

## Actualización de instrucciones para realizar un Restore (automatico con un script y cron) ##

#!/bin/bash

# Definición de rutas
TEMPORAL_DATA_DIR="/home/mhadmin/TEMPORAL_DIR_PARA_RESTAURACION/config"
RESTORE_PATH="/home/mhadmin/docker/volumes/app/mattermost/config"

# 1. Verificar datos en directorio temporal
echo "Verificando datos en directorio temporal..."
if [ ! -d "$TEMPORAL_DATA_DIR" ]; then
    echo "Error: No se encuentra el directorio $TEMPORAL_DATA_DIR"
    exit 1
fi

if [ -z "$(ls -A "$TEMPORAL_DATA_DIR")" ]; then
    echo "Error: El directorio data está vacío"
    exit 1
fi

# 2. Preparar directorio de destino (con sudo)
echo "Preparando directorio de destino..."
echo "Intentando listar contenido (puede requerir contraseña sudo)..."
sudo ls -l "$RESTORE_PATH" || {
    echo "No se pudo verificar el contenido del directorio destino"
    exit 1
}

read -p "¿Estás seguro de borrar el contenido existente? (y/n) " -n 1 -r
echo
if [[ ! $REPLY =~ ^[Yy]$ ]]; then
    echo "Operación cancelada por el usuario"
    exit 1
fi

# 3. Limpiar y restaurar con permisos elevados
echo "Limpiando directorio destino (requiere privilegios)..."
sudo rm -rf "${RESTORE_PATH}"/*

echo "Restaurando archivos de la base de datos..."
sudo cp -r "${TEMPORAL_DATA_DIR}"/* "$RESTORE_PATH/"

# 4. Ajustar permisos (para PostgreSQL en Docker)
echo "Ajustando permisos..."
sudo chown -R 999:999 "$RESTORE_PATH"
sudo chmod -R 700 "$RESTORE_PATH"

# 5. Verificación final
echo "Verificando la restauración..."
sudo ls -l "$RESTORE_PATH"

echo "Restauración completada con éxito!"
echo "Revisa los archivos listados arriba antes de iniciar los servicios"

## Lista de Pasos para Resolver "Permission Denied" al Detener/Eliminar un Contenedor PostgreSQL
1. Verificar el estado del contenedor y procesos asociados
bash
sudo docker ps -a | grep postgres  # Identifica el nombre del contenedor
ps aux | grep postgres            # Busca procesos de PostgreSQL en el sistema
2. Intentar detener el contenedor normalmente
bash
sudo docker stop docker_postgres_1  # Reemplaza con el nombre de tu contenedor
3. Si falla, forzar la eliminación del contenedor
bash
sudo docker rm -f docker_postgres_1
4. Si persiste el error, matar el proceso manualmente
Obtén el PID del proceso PostgreSQL:

bash
ps aux | grep $(sudo docker inspect -f '{{.State.Pid}}' docker_postgres_1)
Mata el proceso (ejemplo con PID 3628):

bash
sudo kill -9 3628
Vuelve a intentar eliminar el contenedor:

bash
sudo docker rm -f docker_postgres_1
o
docker-compose down

## lo anterior debió eliminar el contenedor
5. Reiniciar el servicio Docker (si hay bloqueos)
bash
sudo systemctl restart docker
6. Verificar y limpiar recursos huérfanos
bash
sudo docker system prune -a --volumes  # Cuidado: elimina contenedores, imágenes y volúmenes no utilizados
7. Si el problema es recurrente, revisar permisos y reinstalar Docker
Asegúrate de que tu usuario esté en el grupo docker:

bash
sudo usermod -aG docker $USER
newgrp docker  # Actualiza la sesión
Si usas Docker instalado via Snap (recomendado migrar a Docker nativo):

bash
sudo snap remove docker
sudo apt-get install docker.io
8. Prevención futura
Evita apagados bruscos del servidor o contenedores.

Usa volúmenes persistentes para datos críticos (como PostgreSQL).

Monitorea los logs:

bash
sudo docker logs docker_postgres_1
Resumen Rápido (Cheatsheet)
Listar procesos y contenedores → ps aux | grep postgres + docker ps -a.

Intentar detener/eliminar → docker stop / docker rm -f.

Matar proceso manual → kill -9 <PID>.

Reiniciar Docker → sudo systemctl restart docker.

Limpiar recursos → docker system prune.

Revisar instalación → Migrar de Snap a Docker nativo si es necesario.

Notas Adicionales
Si el contenedor aloja una base de datos, asegúrate de tener backups antes de forzar su eliminación.

El error suele ocurrir por conflictos entre el proceso dentro del contenedor y el sistema host.

## Resolver problemas de puertos ocupados por docker
# 1. Identificar el qué esta ocupando el puerto
sudo lsof -i :80
COMMAND     PID USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
docker-pr 14509 root    4u  IPv4  63216      0t0  TCP *:http (LISTEN)
docker-pr 14518 root    4u  IPv6  60271      0t0  TCP *:http (LISTEN)
* Probar con los puertos 80, 443, 8065 y 8444
# 2. Obtén los PIDs de los procesos docker-pr (14509 y 14518 en este caso)
sudo kill -9 14509 14518
# 3. Ahora el comando sudo lsof -i :80 ya no debe regresar ningún mensaje