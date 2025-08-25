# Objetivo: Demostrar el uso avanzado de volúmenes para persistencia y compartición de datos.
## Descripción:
1. Crea un contenedor writer que escriba la fecha actual cada 10 segundos en un archivo /data/timestamp.log
2. Crea un contenedor reader que lea y muestre el contenido actual del archivo cada vez que ejecutes un comando
3. Si detienes y eliminas ambos contenedores, al recrearlos deben poder seguir accediendo a los datos anteriores
4. Implementa una solución que permita hacer backup del volumen sin detener los contenedores
### Prueba de éxito:
 * Los datos persisten entre reinicios de contenedores
 * Puedes crear un backup del volumen y restaurarlo en otro volumen

##Solución:

Creamos un script llamado ``date.sh`` para writer de la siguiente manera. 
````
While true
do
    date >> /data/timestamp.log
    sleep 10
done

````
Este script crea en un archivo timestamp.log que va imprimiendo la fecha cada 10 segundos.

Con base a este archivo, se crea un Dockerfile para writer de la siguiente manera.

````
FROM nginx:latest
RUN date.sh date.sh
RUN chmod + x date.sh
CMD ["bash", "date.sh"]
````
Posteriormente se construye la imagen.

````
sudo docker build -t writer .
````

Se crea un volumen con el fin de garantizar la compartición de datos.

````
sudo docker volume create volume
````
Se crea un contenedor writer con base a la imagen y el volumen creado.

````
sudo docker run -d --name writer -v volume:/data writer
````
* volume:/data representa el punto de montaje al interior del contenedor, todo lo que se escriba allí persistirá gracias al volumen. 

Posteriormente, se crea la imagen de reader con ayuda del siguiente Dockerfile.

````
FROM nginx:latest
CMD ["tail", "-f", "/dev/null"]
````
Lo anterior, se realiza con el fin de persistir el contenedor que sea creado.

Se ejecuta el comando.

````
sudo docker build -t reader .
````
Con el fin de construir la imagen.

Luego, es creado el contenedor de reader con ayuda del siguiente comando.

````
sudo docker run -d --name reader -v volume:/data -v $(pwd)/backup reader
````
* Se establece un punto de montura que apunta a la carpeta actual del host (pwd), todo lo que se escriba en backup persistirá y se verá en la carpeta actual del host gracias a dicho volumen.

Luego, se conecta al contenedor reader con ayuda del comando.

````
sudo docker exec -it reader /bin/bash
````
Al interior del contenedor en la carpeta la ``/data`` visualizo la existencia de timestamp.log y su constante actualización gracias al contenedor writer con ayuda de cat.








