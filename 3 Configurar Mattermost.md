# Documentación de: **Configuración de Mattermost**

> **Nota importante**: algunas modificaciones pueden realizarse desde System Console de mattermost y otras modificaciones deben ser en el archivo config.json

## 1. Configurar Web Server

    - Es posible configurar Web Server tanto desde la aplicación de mattermost como mediante config.json
    - Inicia sesión como usuario con rol administrador en mattermost
    - Haz click en el cuadrado en la esquina superior izquierda (debajo de los 3 puntos)
    - Selecciona: System Console
    - En el menú izquierdo ve a: Environment > Web Server
    - Editar lo siguiente:
        Site URL: https://cgnovachat.info/ #url de su dominio
        Listen Address: :8065             #Por defecto mattermots usa 8065
    - Guardar los cambios en SAVE

> **Nota importante**: No es necesario realizar cambios en Connection Security, TLS Certificate File y TLS Key File, el tunel ya proporciona HTTPS

> **Requisitos**: Tener una cuenta en sendingblue.com (ahora se llama brevo.com)

## 2. Configurar SMTP desde system console de mattermosts

    - Es posible configurar Web Server desde la propia aplicación de mattermost
    - Inicia sesión como usuario con rol administrador en mattermost
    - Haz click en el cuadrado en la esquina superior izquierda (debajo de los 3 puntos)
    - Selecciona: System Console
    - En el menú izquierdo ve a: Environment > SMTP
    - Poner la siguiente información:
        SMTP Server:                smtp-relay.sendinblue.com # Es el servicio de envio de correos
        SMTP Servver Port:          587 # Es el puerto estandar
        Enable SMTP Authentication: True # Mattermost usará credenciales para conectarse al servidor SMTP.
        SMTP Server Username:       cgnova.kitsu@gmail.com # correo autorizado para enviar emails
        SMTP Server Password:       <La contraseña dada por sendinblue.com> # La proporcionada por SendinBlue (Brevo)
        Connection Security:        STARTTLS # Encripta la conexión entre Mattermost y el servidor SMTP
        Skip Server...:             True # Omite la verificación del certificado SSL (útil en entornos de prueba, pero no recomendado en producción).
        Enable Security Alerts:     True # Mattermost notificará sobre problemas de seguridad
    - Guardar los cambios en SAVE
    - Ir a System Console > Notifications
    - Poner la siguiente información:
        Enable Email Notifications:     True
        Enable Email Batching:          True
        Notification Display Name:      Mattermost
        Notification From Address:      cgnova.kitsu@gmail.com
        Support Email Address:          cgnova.kitsu@gmail.com
        Notification Reply-To Address:  cgnova.kitsu@gmail.com

> **Credenciales**:Estas son las credenciales de brevo
    Environment="MAIL_SERVER=smtp-relay.sendinblue.com"
    Environment="MAIL_PORT=587"
    Environment="MAIL_USERNAME=cgnova.kitsu@gmail.com"
    Environment="MAIL_PASSWORD=T6rp3Bct2wEAHq18"
    Environment="MAIL_DEFAULT_SENDER=cgnova.kitsu@gmail.com"
    Environment="MAIL_USE_TLS=True"

> **Nota importante**: Si el SMTP Servver Port es 587 entonces Connection Security debe ser STARTTLS. Si el SMTP Servver Port es 443 entonces Connection Security debe ser TLS.

## (alternativa) Configurar SMTP desde config.json ##
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

## Configurar Push Notification Server
    - Haz click en el cuadrado en la esquina superior izquierda (debajo de los 3 puntos)
    - Selecciona: System Console
    - En el menú izquierdo ve a: Environment > Push Notification Server
    - Poner la siguiente información:
        Enable Push Notifications:  Use TPNS connection to send notifications to iOS and Android
    - Push Notification Server: https://push-test.mattermost.com
    - Guardar los cambios en SAVE

## Configurar Session Lengths
    - Extend session length with activity: True
    - Terminate Sessions on Password Change: True

> **Nota importante**: El siguiente paso es para producción

## Configurar Developer Settings
    - Haz click en el cuadrado en la esquina superior izquierda (debajo de los 3 puntos)
    - Selecciona: System Console
    - En el menú izquierdo ve a: Environment > Developer
    - Poner la siguiente información:
        Enable Testing Commands:    False
        Enable Developer Mode:  False
        Enable Client Performance Debugging: False
    - Guardar los cambios en SAVE

## Configurar Customization
    - Haz click en el cuadrado en la esquina superior izquierda (debajo de los 3 puntos)
    - Selecciona: System Console
    - En el menú izquierdo ve a: Site configuration > Customization
    - Poner la siguiente información:
        Site Name: CGNOVA
        Site description: (añadir una descripción)
        (lo siguiente es opcional)
        Enable Custom Branding: True
        Custom Brand Image: select image # Elige una imagen de tu dispositivo para usar en el branding
        Custom Brand text: (texto del branding)
    - Guardar los cambios en SAVE

## Configurar localization
     Haz click en el cuadrado en la esquina superior izquierda (debajo de los 3 puntos)
    - Selecciona: System Console
    - En el menú izquierdo ve a: Site configuration > localization
    - Poner la siguiente información:
    Available Languages:    Englis (US) Español (Alpha)
    Enable Experimental Locales:    True
    - Guardar los cambios en SAVE

> **Aclaración**: Esta configuración permite a los usuarios cambiar el idioma en el que verán el servicio de mattermos, de forma estandar la interfaz les aparecerá en ingles, si desea que de forma estandar el servicio aparezca en otro idioma, modifique la opción "Default Client Language" para el lenguaje de la interfaz de los usuarios que se creen una cuenta nueva o "Default Server Language" para cambiar el idioma de todas las cuentas

## Configurar Public Links
    - Haz click en el cuadrado en la esquina superior izquierda (debajo de los 3 puntos)
    - Selecciona: System Console
    - En el menú izquierdo ve a: Site configuration > Public Links
    - Poner la siguiente información:
        Enable Public File Links: True
    - Guardar los cambios en SAVE

## Configurar Signup
    - Haz click en el cuadrado en la esquina superior izquierda (debajo de los 3 puntos)
    - Selecciona: System Console
    - En el menú izquierdo ve a: Authentication > Signup
    - Poner la siguiente información:
        Enable Account Creation:    True
        Enable Open Server: 
        Enable Email Invitations:   True
    - Guardar los cambios en SAVE

## Configurar Email 
    - Haz click en el cuadrado en la esquina superior izquierda (debajo de los 3 puntos)
    - Selecciona: System Console
    - En el menú izquierdo ve a: Authentication > Email
    - Poner la siguiente información:
        Enable account creation with email: True
        Require Email Verification: False
        Enable sign-in with email: True
        Enable sign-in with username: True
    - Guardar los cambios en SAVE

## Configurar Password
    - Haz click en el cuadrado en la esquina superior izquierda (debajo de los 3 puntos)
    - Selecciona: System Console
    - En el menú izquierdo ve a: Authentication > Password
    - Poner la siguiente información:
        Minimum Password Length:    8
        Password Requirements:  ☑ At least one lowercase letter
                                ☑ At least one uppercase letter
                                ☑ At least one symbol (e.g. "~!@#$%^&*()")
        
        Maximum Login Attempts: 10
        Enable Forgot Password Link:    True
    - Guardar los cambios en SAVE

    

