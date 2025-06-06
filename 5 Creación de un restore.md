# Creación de un backup automatizado para los volumenes data de postgres, data y config de mattermost

## Requisitos
    - Tener instalado cron
        - Para instalar cron
            sudo apt update
            sudo apt install cron
        - Activar el servicio de cron
            sudo systemctl enable cron
        - Hacer que se inicie automaticamente al encender el servidor
            sudo systemctl start cron
## Pasos para crear un restore automaatizado
    - Crear un directorio para backups
            sudo mkdir -p /opt/backup-scripts/
        - Crear el archivo donde pondremos el script
            sudo touch restore_mattermost.sh
        - Darle al usuario actual el control sobre los archivos y directorios
            sudo chown -R $USER:$USER /opt/backup-scripts/restore_mattermost.sh
        - Entrar al directorio creado y editar el archivo
            Sudo nano restore_mattermost.sh
        - Escribir lo siguiente:
            #!/bin/bash
                    /usr/bin/docker run --rm \
                        -v mattermost_data:/data \
                        -v /home/mhadmin/backups_mattermost/mattermost_data:/backup \
                        alpine sh -c "rm -rf /data/* && tar xzvf /backup/mattermost_data_backup_$(date +\%Y-\%m-\%d).tar.gz -C /data --strip-components=1"

                    /usr/bin/docker run --rm \
                        -v mattermost_config:/config \
                        -v /home/mhadmin/backups_mattermost/mattermost_config:/backup \
                        alpine sh -c "rm -rf /config/* && tar xzvf /backup/mattermost_config_backup_$(date +\%Y-\%m-\%d).tar.gz -C /config --strip-components=1"

                    /usr/bin/docker run --rm \
                        -v db_postgres_data:/data \
                        -v /home/mhadmin/backups_mattermost:/backup \
                        alpine sh -c "rm -rf /data/* && tar xzvf /backup/db_postgres_data_backup_$(date +\%Y-\%m-\%d).tar.gz -C /data --strip-components=1"

            # Forzar el cambio de contraseña del usuario de Mattermost
                    /usr/bin/docker exec docker_postgres_1 psql -U mmuser -d mattermost -c "ALTER USER mattermost WITH PASSWORD 'mmuser_password';"
            # Reiniciar los servicios
                    /usr/bin/docker-compose -f /home/mhadmin/mattermostrepo/docker/docker-compose.yml --env-file /home/mhadmin/mattermostrepo/docker/env.magicchat up -d
    - Guardar los cambios: ctrl + O Enter
    - Salir del archivo: ctrl + X
    - Editar el cron:
        sudo crontab -e
    - Escribir esto al final:
        #56 17 * * * /opt/backup-scripts/restore_mattermost.sh > /var/log/mattermost_restore.log 2>&1
    - Guardar los cambios: ctrl + O Enter
    - Salir del archivo: ctrl + X