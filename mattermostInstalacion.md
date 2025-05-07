# Instalación de `Mattermost en Ubuntu Server`

## Requisitos
- Tener instalado Docker en Ubuntu Server
- Tener instalado Docker Compose
prueba


## Pasos de instalación
1. **Instalación de mattermost y postgres en Ubuntu server**:  
    - Copiar el repositorio de github con el comando:
        git clone https://github.com/mattermost/docker 
    - Entrar en Docker con el comando:
        cd docker
    - Ejecute el siguiente comando para copiar el archivo .env del repositorio:
        cp env.example .env
    - Comprobar que se haya creado el archivo env.example 
        ls -l ##Debe aparecer el nombre del archivo##
    - Se le debe cambiar el nombre al archivo .env por env.NombreDeSuDominio
        mv env.example env.NombreDeSuDominio
    - Despues entrar al archivo para editarlo:
        nano env.magicchat
    - Editar esta linea DOMAIN= y agregar la ip de su servidor
        Ejemplo: DOMAIN=192.179.8.5
    - Añade tu usuario al grupo docker:
        sudo groupadd docker
        sudo usermod -aG docker $USER
        newgrp docker  # O cierra y abre la terminal
        sudo reboot
    - Asignar el socket al grupo docker:
        sudo chown root:docker /var/run/docker.sock
    - Revisar los permisos del socket:
        ls -l /var/run/docker.sock ##Resultado esperado: srw-rw---- 1 root docker 0 Apr 10 15:58 /var/run/docker.sock##
    - Reinicia el servidor:
        sudo systemctl restart docker
    - Fusiona los siguientes dos archivos de Docker Compose y ejecuta los servicios en background:
        docker compose -f docker-compose.yml -f docker-compose.without-nginx.yml up -d
    - Realiza la comprobación:
        getent group docker  # Debe mostrar el grupo
        groups              # Debe listar "docker" entre tus grupos
    - Revisa los permisos:
        ls -l /var/run/docker.sock
        - Si no muestra docker en los permisos puedes ajustar los permisos con:
        sudo chown root:docker /var/run/docker.sock
    - Comprobar la instalación de los contenedores de mattermost y de postgres:
        -Dentro de docker: escribir ls -l
## Si aparece en el contenedor de postgres Restarting
    - Correr los siguientes comandos:
        docker inspect docker-postgres-1
**Si en postgres aparece "ReadonlyRootfs": true**
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
        - Usa docker logs docker-postgres-1 para revisar los logs y obtener más información si el contenedor de postgres sigue diciendo restarting
**Si aparece en el contenedor de mattermost Restarting**
    - Puede deberse a problemas de permiso en los volumenes montados, se puede arreglar dando permisos con el siguiente comando: sudo chown -R 2000:2000 /home/mhadmin/mattermost/volumes/app/ 

2. **Configuración de mattermost**
    - Editar el archivo :
        -
3. **Correr los contenedores de mattermost y postgres**
    - Usar el comando para correr ambos contenedores
        - docker start docker_mattermost_1 docker_postgres_1

4. **Comprobar que el servicio sea accesible en local**
    - Abrir el navegador e ingresar en la url http://<dirección><puerto> 
    #Ejemplo: http://localhost:8080 #
5. **Instalar mattermost en un ordenador**
    - Descargar mattermost en la página:

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



        