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
## Pasos para crear un backup automaatizado

    - Crear un directorio para backups
        sudo mkdir -p /opt/backup-scripts/
    - Crear el archivo donde pondremos el script
        sudo touch mattermost_postgres_backup.sh
    - Darle al usuario actual el control sobre los archivos y directorios
        sudo chown -R $USER:$USER /opt/backup-scripts/mattermost_postgres_backup.sh
    - Entrar al directorio creado y editar el archivo
        Sudo nano mattermost_postgres_backup.sh
    - Escribir lo siguiente:
        #!/bin/bash

        # --------------------------------------------
        # 1. Backup de postgres_data
        # --------------------------------------------
                # Directorios donde se guardarán los backups
                PRIMARY_BACKUP_DIR_POSTGRES="/home/mhadmin/backups_mattermost"
                SECONDARY_BACKUP_DIR_POSTGRES="/mnt/carpeta_compartida/postgres_data"

                # Crear los directorios si no existen
                mkdir -p "$PRIMARY_BACKUP_DIR_POSTGRES"
                mkdir -p "$SECONDARY_BACKUP_DIR_POSTGRES"

                # Nombre del archivo de backup con fecha
                BACKUP_FILE_POSTGRES="db_postgres_data_backup_$(date +%Y-%m-%d).tar.gz"

                # Ejecutar el comando de docker con ambos volúmenes montados
                docker run --rm \
                    -v db_postgres_data:/data \
                    -v "$PRIMARY_BACKUP_DIR_POSTGRES":/backup1_postgres \
                    -v "$SECONDARY_BACKUP_DIR_POSTGRES":/backup2_postgres \
                    alpine sh -c "tar czvf /backup1_postgres/$BACKUP_FILE_POSTGRES /data && cp /backup1_postgres/$BACKUP_FILE_POSTGRES /backupp2_postgres/"
                
                # Eliminar backups antiguos (conserva los últimos 4) en ambos directorios
                find "$PRIMARY_BACKUP_DIR_POSTGRES" -name "db_postgres_data_backup_*.tar.gz" -mtime +4 -exec rm {} \;
                find "$SECONDARY_BACKUP_DIR_POSTGRES" -name "db_postgres_data_backup_*.tar.gz" -mtime +4 -exec rm {} \;
        # --------------------------------------------
        # 2. Backup de mattermost_data
        # --------------------------------------------

                # Directorios de backup
                PRIMARY_BACKUP_DIR_MATTERMOST_DATA="/home/mhadmin/backups_mattermost/mattermost_data"
                SECONDARY_BACKUP_DIR_MATTERMOST_DATA="/mnt/carpeta_compartida/mattermost_data"

                # Crear directorios si no existen
                mkdir -p "$PRIMARY_BACKUP_DIR_MATTERMOST_DATA"
                mkdir -p "$SECONDARY_BACKUP_DIR_MATTERMOST_DATA"

                # Nombre del archivo de backup
                BACKUP_FILE_MATTERMOST_DATA="mattermost_data_backup_$(date +%Y-%m-%d).tar.gz"

                # Ejecutar Docker con DOS volúmenes montados:
                docker run --rm \
                    -v mattermost_data:/data \
                    -v "$PRIMARY_BACKUP_DIR_MATTERMOST_DATA":/backup_primary_mattermost_data \
                    -v "$SECONDARY_BACKUP_DIR_MATTERMOST_DATA":/backup_secondary_mattermost_data \
                    alpine sh -c "tar czvf /backup_primary_mattermost_data/$BACKUP_FILE_MATTERMOST_DATA /data && cp /backup_primary_mattermost_data/$BACKUP_FILE_MATTERMOST_DATA /backup_secondary_mattermost_data/"
                
                # Eliminar backups antiguos (más de 4 días) en AMBAS ubicaciones
                find "$PRIMARY_BACKUP_DIR_MATTERMOST_DATA" -name "mattermost_data_backup_*.tar.gz" -mtime +4 -exec rm {} \;
                find "$SECONDARY_BACKUP_DIR_MATTERMOST_DATA" -name "mattermost_data_backup_*.tar.gz" -mtime +4 -exec rm {} \;

        # --------------------------------------------
        # 3. Backup de mattermost_config
        # --------------------------------------------

                # Directorios de backup
                PRIMARY_BACKUP_DIR_MATTERMOST_CONFIG="/home/mhadmin/backups_mattermost/mattermost_config"
                SECONDARY_BACKUP_DIR_MATTERMOST_CONFIG="/mnt/carpeta_compartida/mattermost_config"

                # Crear directorios si no existen
                mkdir -p "$PRIMARY_BACKUP_DIR_MATTERMOST_CONFIG"
                mkdir -p "$SECONDARY_BACKUP_DIR_MATTERMOST_CONFIG"

                # Nombre del archivo backup
                BACKUP_FILE_MATTERMOST_CONFIG="mattermost_config_backup_$(date +%Y-%m-%d).tar.gz"

                # Comando de docker para generar los backups
                docker run --rm \
                    -v mattermost_config:/config \
                    -v "$PRIMARY_BACKUP_DIR_MATTERMOST_CONFIG":/backup_primary_mattermost_config \
                    -v "$SECONDARY_BACKUP_DIR_MATTERMOST_CONFIG":/backup_secondary_mattermost_config \
                    alpine sh -c "tar czvf /backup_primary_mattermost_config/$BACKUP_FILE_MATTERMOST_CONFIG /config && cp /backup_primary_mattermost_config/$BACKUP_FILE_MATTERMOST_CONFIG /backup_secondary_mattermost_config/"
                
                # Eliminar los backups antiguos de más de 4 días
                find "$PRIMARY_BACKUP_DIR_MATTERMOST_CONFIG" -name "mattermost_config_backup_*.tar.gz" -mtime +4 -exec rm {} \;
                find "$SECONDARY_BACKUP_DIR_MATTERMOST_CONFIG" -name "mattermost_config_backup_*.tar.gz" -mtime +4 -exec rm {} \;

    - Guardar los cambios: ctrl + O Enter
    - Salir del archivo: ctrl + X
    - Editar el cron:
        sudo crontab -e
    - Escribir esto al final:
        0 2 * * * /opt/backup-scripts/mattermost_postgres_backup.sh > /var/log/mattermost_backup.log 2>&1
    - Guardar los cambios: ctrl + O Enter
    - Salir del archivo: ctrl + X