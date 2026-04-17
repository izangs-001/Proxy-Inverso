# Mi Documentación de la Práctica: Proxies con Nginx

En este documento explico paso a paso todo lo que he hecho para montar la infraestructura, los comandos que he utilizado y los problemas que me han ido surgiendo por el camino.

## Paso 1: Creación de la estructura del proyecto
Lo primero que he hecho ha sido organizar mi espacio de trabajo. He creado una carpeta principal y dentro dos carpetas específicas: una para el contenido web y otra para la configuración de Nginx.
- **Comandos usados:** `mkdir -p practica_proxy/html practica_proxy/nginx`
- **Por qué lo he hecho:** Para tenerlo todo bien organizado y que luego sea más fácil montar los volúmenes en Docker sin mezclar archivos de configuración con archivos de la web.

## Paso 2: Preparación del contenido web (HTML, CSS e Imagen)
He creado un archivo `index.html` sencillo y un `style.css` para que la web no se vea vacía. También he aprovechado una imagen que ya tenía en mi equipo para usarla como recurso compartido.
- **Problema encontrado:** Al principio creé un archivo de vídeo vacío con el comando `touch`, pero al probar la web en el navegador, el reproductor daba error porque el archivo no tenía formato real.
- **Solución:** He usado `ffmpeg` para generar un vídeo de prueba de 5 segundos con un patrón de colores. Así me aseguro de que el proxy sirve correctamente archivos multimedia reales.
- **Comando usado:** `ffmpeg -f lavfi -i testsrc=duration=5:size=640x360:rate=30 -c:v libx264 practica_proxy/html/video.mp4`

<img width="1277" height="1005" alt="imagen" src="https://github.com/user-attachments/assets/a8c72b7a-ff56-4f77-aa59-9f001935ec4c" />

## Paso 3: Configuración del Proxy Inverso y el Balanceo
He configurado el archivo `nginx/default.conf`. He definido un bloque `upstream` llamado "backends" donde he metido los dos servidores Apache.
- **Por qué lo he hecho:** Para cumplir con el requisito de Round Robin. Al no especificar ningún método, Nginx reparte las peticiones una a cada uno por defecto.
- **Mejora personal:** He añadido cabeceras personalizadas (`add_header`) para que el servidor me diga en todo momento qué IP de Apache me está respondiendo y si la respuesta viene de la caché o no. Esto me ha facilitado muchísimo las pruebas.

<img width="1655" height="738" alt="imagen" src="https://github.com/user-attachments/assets/e88d0df4-825e-4d35-9deb-a9990fc18248" />
<img width="1627" height="836" alt="imagen" src="https://github.com/user-attachments/assets/6dd41f22-cb6a-4ed5-a039-ed569cb57b3f" />


## Paso 4: Orquestación con Docker Compose
He escrito el archivo `docker-compose.yml` para levantar los tres contenedores a la vez: `apache1`, `apache2` y el `nginx_proxy`.
- **Por qué lo he hecho:** Para no tener que levantar cada contenedor a mano. He usado una red propia para que el proxy pueda comunicarse con los Apache usando sus nombres de contenedor. También he configurado un volumen compartido para que los dos Apache lean de la misma carpeta `html/`.

## Paso 5: Pruebas de Balanceo de Carga
He comprobado que el balanceo funcionaba correctamente haciendo varias peticiones seguidas.
- **Comando usado:** `curl -I http://localhost:8080`
- **Lo que he observado:** Gracias a la cabecera `X-Backend-Server` que configuré antes, he podido ver cómo la IP del servidor de fondo iba cambiando en cada refresco de página.

<img width="541" height="937" alt="imagen" src="https://github.com/user-attachments/assets/3ac03428-b59c-441d-bd3f-5203c8878ab4" />


## Paso 6: Configuración y prueba de la Memoria Caché
Finalmente, he activado la caché en el proxy para que no tenga que pedirle siempre el contenido a los Apache.
- **Problema encontrado:** Al probar con el navegador, a veces me salía siempre `MISS`. Me he dado cuenta de que el navegador enviaba peticiones pidiendo no usar caché.
- **Solución:** He configurado Nginx para que ignore las cabeceras de control de caché del cliente y del servidor Apache (`proxy_ignore_headers`), obligándole a guardar el contenido durante 10 minutos.



---
**Realizado por:** Izan Gómez Solano
