# Documentación de Instalación de: **Creación de un tunel mediante cloudflare**

**Requisitos**
Tener una cuenta de cloudflare
Tener un domino de cloudflare
El dominio debe tener un DNS tipo CNAME

## Instalación de cloudflared
    - Usa este comando para descargar la última versión del paquete de Cloudflared directamente desde github:

        wget https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-amd64.deb
    
    -  Usa el siguiente comando para instalar cloudflared

        sudo dpkg -i cloudflared-linux-amd64.deb
    
    - Comprueba la versión

        cloudflared --version

## Autentifica el tunel con tu cuenta de cloudflare

    - Usa el siguiente comando para iniciar sesión con cloudflare:

        cloudflared tunnel login

    - Entra a la url del mensaje, inicia sesión con tus credenciales de cloudflare y elige el dominio que usarás para mattermost

    - Después se generará un certificado en ~/.cloudflared/cert.pem

## Crear el tunnel

    - Usa el siguiente comando para crear el tunel

        cloudflared tunnel create mattermost-tunnel

> **Nota importante**: El comando anterior también generará un UUID del túnel y un archivo de credenciales en ~/.cloudflared/<UUID>.json

## Crea un archivo de configuración

    - Entra a la carpeta .cloudflared

        cd .cloudflared

    - Crea un archivo .yml con el siguiente comando:

        touch config.yml

    - Editalo:

        sudo nano config.yml

    - Escribe lo siguiente:

        tunnel: tunelprueba #Nombre del tunel
            credentials-file: /home/mhadmin/.cloudflared/0f4dd68f-352e-44d5-8d21-6e8792f2ef7a.json #Ruta hacia las credenciales

            ingress:
            - hostname: cgnovachat.info             #Nombre del host
                service: http://192.168.1.7:8065    #dirección ip y puerto 
            - service: http_status:404

    - Guarda los cambios:

        ctrl + O
        después presiona Enter
    
    - Para salir:

        ctrl + X

## Enrutar el subdominio a Cloudflare

    - Ejecuta este comando para enlazar el subdominio con el túnel:

        cloudflared tunnel route dns <nombre_del_tunel> cgnovachat.info       

> **Nota importante**: service: http_status:404 es para que si alguien intenta acceder a un subdominio o ruta que no está configurado en ingress, recibirá un error 404 en lugar de ser dirigido a otro servicio por error.

## Dar permisos

    - Otorga permisos para que docker pueda leer el certificado de cloudflare
        - chmod 644 ~/.cloudflared/cert.pem  # -rw-r--r--

## Probar el tunel

    - Usa el siguiente comando para iniciar el tunnel:

        cloudflared tunnel run mattermost-tunnel

    - Si la configuración es correcta deberías poder acceder al servicio de mattermost mediante: 
        
        https://mattermost.tudominio.com

## Configurar el túnel como servicio (para que inicie automáticamente)

    - Crea un servicio systemd para que el túnel se ejecute al iniciar el sistema:

        sudo nano /etc/systemd/system/cloudflared.service

    - Escribe la siguiente configuración y editala de ser necesario

        [Unit]
        Description=Cloudflare Tunnel
        After=network.target

        [Service]
        Type=simple
        User=mhadmin    #Tu usuario con el que acceder al servidor
        ExecStart=/usr/local/bin/cloudflared tunnel --config /home/mhadmin/.cloudflared/config.yml run tunelprueba
        Restart=always  #Esto establece que el servicio del tunel se reiniciara siempre que se detenga por error
        RestartSec=5    #Intervalo de espera antes de reiniciar el servicio si esta detenido

        [Install]
        WantedBy=multi-user.target

> **Explicación de ExecStart=/usr/local/bin/cloudflared tunnel --config /home/mhadmin/.cloudflared/config.yml run tunelprueba**:
    ExecStart= → Define el comando que se ejecutará cuando el servicio inicie.
    /usr/local/bin/cloudflared → Ruta del ejecutable de Cloudflared.
    tunnel --config /home/mhadmin/.cloudflared/config.yml → Indica que se usará un archivo de configuración específico (config.yml).
    run tunelprueba → Ejecuta el túnel llamado tunelprueba.

## Habilita e inicia el servicio:

    - Hacer que el sistema reconozca el nuevo servicio

        sudo systemctl daemon-reload

    - Habilita el servicio para que inicie al encender el servidor:

        sudo systemctl enable cloudflared
    
    - Inicia el servicio manualmente (es necesario usar esto si detienes el servicio manualmente)

        sudo systemctl start cloudflared

    - Verifica el estado del servicio

        sudo systemctl status cloudflared

    - Debe aparecer el texto active (running)

    - Comandos útiles para el servicio de tunel:
        Comando	                            Descripción
        sudo systemctl start cloudflared	Inicia el servicio manualmente
        sudo systemctl stop cloudflared	    Detiene el servicio
        sudo systemctl restart cloudflared	Reinicia el servicio
        sudo systemctl status cloudflared	Muestra el estado actual
        sudo journalctl -u cloudflared -f	Muestra los logs en tiempo real

