# ejabber2

```

Dockerfile
Crea con tu editor texto favorito, un archivo sin extensión llamado Dockerfile y dentro copiar este contenido.

FROM arm32v7/debian:latest
MAINTAINER angel <ugeekpodcast@gmail.com>

RUN apt-get -y update; \
    apt-get -y upgrade; \
    apt-get -y install apt-utils \
    ejabberd; 


EXPOSE 5280
EXPOSE 5222
EXPOSE 5269

VOLUME /etc/ejabberd/


ENTRYPOINT service ejabberd start && /bin/bash
CMD ["bash"]
La primera línea, FROM, nos descargará de Docker Hub la distro debian para arm32v7.
MAINTAINER, son los datos de quien va a mantener el contenedor. En este caso no es necesario, ya que mi intención no es subirlo al Docker Hub, pero vamos a ponerlo.
RUN, Comando que se ejecutaran del mismo modo que un script en bash. Actualizamos repositorios e instalamos todo aquello que necesitamos para crear nuestro servidor.
EXPOSE, son los puertos que quedarán abiertos en el contenedor.
VOLUME, es la carpeta del contenedor que deseamos que quede accesible desde fuera del contenedor. Esto es ideal en este caso para dejar el archivo de configuración fuera del contenedor y así si después actualizamos el Docker o se rompe, restaurando a partir de la imagen. Dejando el archivo de configuración, todo funcionará como si no hubiera pasado nada.
ENTRYPOINT, Comando que ejecutaremos una vez montado el contenedor.
Creando la Imagen
Nos situaremos en la carpeta donde está el archivo Dockerfile y ejecutaremos el siguiente comando:

docker build -t ejabberd_imagen:dockerfile .

Veamos si se ha creado la imagen
Con este comando veremos todas las imágenes que tenemos en el servidor. La última, corresponderá la imagen que hemos creado:

docker images

Tenemos una nueva imagen de 155Mb con el nombre que habíamos puesto!!!

REPOSITORY                TAG                 IMAGE ID            CREATED             SIZE
ejabberd_imagen           dockerfile          983bc10a3e5b        47 seconds ago      155MB
Creando del Contenedor
Ya tenemos nuestra imagen, ahora vamos a crear el contenedor. A la imagen la nombramos ejabberd_imagen y al contenedor, lo pondremos el nombre de ejabberd

Solo hay 2 valores que tiene que añadir:

$USUARIO : Escribe tu usuario
$IMAGE_ID : Escribe el número de IMAGE ID, que te apareció en el paso anterior.
docker run -dti --name ejabberd --restart=unless-stopped -p 5280:5280 -p 5222:5222 -v /home/$USUARIO/docker/ejabberd_config:/etc/ejabberd/ $IMAGE_ID

En la carpeta /home/$USUARIO/docker/ejabberd_config, encontraremos el archivo de configuración del servidor.

Entrar en el Docker
Ahora vamos a entrar al interior del Docker. Una vez dentro de este, estaremos en una versión de debian, con todo lo justo para que funcione nuestro servidor y para que sea del mínimo tamaño.

docker exec -i -t ejabberd /bin/bash

ejabberd es el nombre que habíamos dado a nuestro contenedor.

Vamos a configurar nuestro servidor
Una vez dentro, mediante este comando, daremos nombre al servidor, nombre de administrador y contraseña del mismo.

dpkg-reconfigure ejabberd

Es posible que de error. Si es así, pararemos el servidor y lo volveremos a iniciar.

Detener el servidor

service ejabberd stop

Iniciar el servidor

service ejabberd start

Configurando el servidor
Ahora nos conectamos a https://ip:5280/admin, que es la web de administración del servicio. Nos pedirá para acceder el nombre del servidor y usuario del administrador, que hemos puesto en el paso anterior.

Nombre de usuario: admin@localhost Contraseña: password

Archivo de configuración está en: /etc/ejabberd/ejabberd.yml

Añadir usuarios
Para añadir usuarios desde la interfaz de administración web, seleccionaremos en el menú Dominios Virtuales -> Dominio -> Usuarios. Ahí, añadiremos nuestros usuarios y sus contraseñas. Esto sería en el caso que únicamente el administrador del servidor, sea el único que permita el registro de usuarios, tal como viene por defecto. También podríamos hacer que cada usuario que quiera, pueda registrarse.

Aumentar la velocidad de transferencia de archivos. TRAFFIC SHAPERS
Ejabberd/xmpp, no está diseñado para transferir archivos, aún así, notarás que la transferencia de archivos es muy lenta y es debido a que el archivo de configuración tiene limitación en la transferencia por defecto, por si el número de usuarios que van a utilizar el servidor es muy grande. También tiene limitado el tamaño máximo de archivo a compartir. Desmarca mediante # o personaliza a tu gusto los parametros en ejabberd.yml, Recuerda que puedes acceder desde fuera del contenedor, gracias al volumen que creamos al crear el mismo.

shaper:
  ##
  ## The "normal" shaper limits traffic speed to 1000 B/s
  ##
##########  normal: 10000000

  ##
  ## The "fast" shaper limits traffic speed to 50000 B/s
  ##
##########  fast: 50000

##
## This option specifies the maximum number of elements in the queue
## of the FSM. Refer to the documentation for details.
##
#max_fsm_queue: 1000
Podemos personalizar absolutamente nuestro servidor desde este archivo, pero es muy complejo. Visita la web de ejabberd para ver ejemplos y sobretodo, haz una copia del archivo antes de modificarlo.

Web ejabberd
https://www.ejabberd.im/


```
