# Documentació para solucionar el problema de las descargas de archivos en mac

## Este problema ocurre debido a que la versión disponible para la app de escritorio es la versión 5.12.1 y desde versiones anteriores se dejo de actualizar todas las funciones para que funcionen correctamente en mac.

## El problema es que a pesar de darle permisos a la mac para que mattermost use la carpeta de descargas, las descargas no se iniciaban

### Solución
    - Entra al archivo config.json
    - Escribe lo siguiente dentro de FileSettings:
    "FileSettings": {

        "CustomUserAgent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/605.1.15 (KHTML, like Gecko)"

    }

## El código anterior fuerza a la app Mattermost (en este caso, la versión de escritorio para Mac) a identificarse ante el servidor como si fuera un navegador web y permite realizar las descargas

### Comprobar en la mac que Mattermost tenga permisos para acceder a la carpeta de Descargas
    - Entrar a Configuración del sistema > Privacidad y seguridad > Archivos y carpetas
    - Asegurarse de que Mattermost está en la lista y Capeta Descargas esta activado
    - Si no está la aplicación de mattermost entre las opciones, entra a mattermost e intenta descargar un archivo, debe de saltar la ventana para conceder permisos y debes permitir que acceda a Descargas y repetir el primer paso (Entrar a configuración del sistema)