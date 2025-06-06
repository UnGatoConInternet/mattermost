# Documentació para crear un volumen mediante samba para tener una carpeta compartida

**Requisitos**
Tener instalado samba

## Pasos para instalar samba
    - Actualiza el sistem
        sudo apt update && sudo apt upgrade -y
    - Instala Samba
        sudo apt install samba -y
    - Edita el archivo de configuración de Samba
        sudo nano /etc/samba/smb.conf
    - Al final del archivo escribe lo siguiente:
        [Volumen_de_backups_de_mattermost]
                path = /home/mhadmin/backups_mattermost
                browsable = yes
                writeable = yes
                guest ok = no
                valid users = mhadmin
                force user = mhadmin
                create mask = 0775
                directory mask = 0775
                inherit permissions = yes
                inherit owner = yes
    - Guardar los cambios: ctrl + O Enter
    - Salir del archivo: ctrl + X

    - Crea la dirección donde estará carpeta compartida
        sudo mkdir -p /mnt/carpeta_compartida
    - Asigna permisos
        sudo chmod -R 0775 /mnt/carpeta_compartida
    - Agrega tu usuario a samba
        sudo smbpasswd -a tu_usuario_ubuntu
    - Reinicia Samba y habilita el servicio
        sudo systemctl restart smbd
        sudo systemctl enable smbd
## Montar la carpeta compartida de synology
    - Instalar cifs-utils (para montar recursos SMB)
        sudo apt update && sudo apt install cifs-utils -y
    - Crear un archivo para guardar las credenciales
        sudo nano /root/.smbcreds
    - Poner las credenciales de synology
        username=techadmin
        password=M4gic01_#
    - Editar fstab para el montaje automatico
        sudo nano /etc/fstab
    - Agregar lo siguiente al final del archivo
        //192.168.1.100/SERVICES_BACKUP/Mattermost_Backup  /mnt/carpeta_compartida  cifs  credentials=/root/.smbcreds,vers=2.0,uid=1000,gid=1000,noauto,x-systemd.automount,_netdev  0  0
    - Guardar los cambios: ctrl + O Enter
    - Salir del archivo: ctrl + X
    - Darle permisos al archivo
        sudo chmod 644 fstab
    - Probar la configuración
        sudo mount -a
    
    