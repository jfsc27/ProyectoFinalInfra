# ProyectoFinalInfra
Repositorio con la bitacora


INTRODUCCION
El presente proyecto tiene como objetivo implementar una infraestructura funcional y persistente utilizando tecnologías de almacenamiento y contenedorización en un entorno Linux corriendo sobre una máquina virtual creada en VirtualBox.

Durante el desarrollo se configuraron arreglos RAID, volúmenes lógicos mediante LVM, sistemas de archivos montados de manera persistente y contenedores Docker que utilizan estos volúmenes para asegurar la integridad y permanencia de los datos incluso después de reinicios del sistema o de los servicios.

La implementación se realizó siguiendo buenas prácticas de administración de sistemas y con un enfoque orientado a la tolerancia a fallos y la persistencia de datos.


REQUERIMIENTOS DEL PROYECTO
El proyecto exigía la construcción de un entorno que cumpliera con los siguientes requisitos:

Configuración de Ubuntu dentro de VirtualBox.

Creación y conexión de 9 discos virtuales: 3 para Apache, 3 para MySQL y 3 para Nginx.

Implementación de tres arreglos RAID 1, cada uno con un disco repuesto (spare).

Creación de volúmenes LVM sobre cada arreglo RAID.

Formateo y montaje persistente de los LVM mediante /etc/fstab.

Instalación de Docker.

Construcción de imágenes personalizadas de Apache y Nginx mediante Dockerfiles.

Creación de contenedores Apache, Nginx y MySQL con volúmenes persistentes.

Pruebas de persistencia de archivos y datos después de reinicios.

Documentación completa del proceso en una bitácora.


CONFIGURACION DEL VIRTUAL BOX
Para cumplir con los requisitos del sistema, se creó una máquina virtual en VirtualBox con Ubuntu y se agregaron nueve discos virtuales adicionales.
Cada disco fue configurado como almacenamiento independiente para simular dispositivos físicos.

Esta separación permitió implementar los RAIDs y LVM de manera correcta, emulando un entorno real de producción donde los servicios utilizan almacenamiento redundante y tolerante a fallos.

La verificación inicial con lsblk permitió confirmar la correcta detección de los discos por parte del sistema operativo.


CONFIGURACION RAID
Se utilizó la herramienta mdadm para la creación y administración de los arreglos RAID.

Cada servicio (Apache, MySQL y Nginx) cuenta con su propio RAID 1 compuesto por dos discos en espejo y un disco repuesto (spare).
El uso de espejo garantiza que, si un disco falla, los datos permanecen íntegros en el disco redundante, mientras que el repuesto puede entrar en funcionamiento automáticamente.

Los comandos utilizados permitieron limpiar firmas previas, crear los arreglos, verificar su estado y guardar la configuración de forma persistente.

El uso de cat /proc/mdstat fue fundamental para validar el correcto funcionamiento de los RAIDs.


CONFIGURACION LVM
Sobre cada RAID se implementó LVM para permitir flexibilidad en la administración del almacenamiento.
Cada RAID fue inicializado como un Physical Volume (PV), luego se creó un Volume Group (VG) y finalmente un Logical Volume (LV).

Este enfoque permitió:

Manejar el espacio de manera más eficiente.

Poder escalar o extender volúmenes en el futuro.

Facilitar el formateo y montaje de los sistemas de archivos.

Los comandos pvs, vgs y lvs fueron utilizados para visualizar y verificar la estructura LVM creada.


FORMATEO Y MONTAJE PERSISTENTE
Cada LV fue formateado utilizando el sistema de archivos ext4.
Posteriormente se crearon los puntos de montaje necesarios:

/mnt/lvm_apache/www

/mnt/lvm_mysql

/mnt/lvm_nginx/html

El uso de estos directorios permitió asociar directamente los datos de los servicios a volúmenes persistentes ubicados en discos administrados mediante RAID y LVM.

La configuración de /etc/fstab permitió que los volúmenes se montaran automáticamente al iniciar el sistema, asegurando persistencia incluso después de reinicios.


INSTALACION Y CONFIGURACION DE DOCKER
Docker fue instalado mediante el script oficial, garantizando que se utilizara la versión más reciente y estable del servicio.

Una vez instalado, se verificó su estado y funcionamiento mediante:

docker --version

systemctl status docker

Se confirmó que Docker estuviera activo y habilitado en el sistema.


CREACION DE IMAGENES
Se desarrollaron dos Dockerfiles:

Apache

Basado en httpd:2.4, con un archivo HTML propio copiado al directorio del servidor.

Nginx

Basado en nginx:alpine, también con su propio archivo HTML.

Estas imágenes personalizadas permitieron demostrar la capacidad de modificar el contenido servido por los contenedores.


CONSTRUCCION DE CONTENEDORES
Se construyeron imágenes personalizadas para Apache y Nginx desde sus respectivos Dockerfiles mediante docker build.

Luego se crearon contenedores para:

Apache → puerto 8080
Nginx → puerto 8081
MySQL → puerto 3306

Cada contenedor fue creado con la directiva -v para mapear directamente los directorios de los volúmenes LVM hacia las rutas internas del contenedor.

Esto permitió que los datos se mantuvieran incluso si se eliminaba o recreaba el contenedor.


PRUEBAS DE PERSISTENCIA
Se realizaron pruebas de persistencia modificando los archivos HTML de Apache y Nginx directamente en los directorios montados.
Luego se reiniciaron los contenedores y se comprobó que los cambios permanecían visibles, demostrando que los volúmenes estaban funcionando correctamente.

Para MySQL se creó una base de datos llamada prueba mediante el cliente interno del contenedor.
Tras reiniciar el contenedor y reiniciar completamente el sistema operativo, se confirmó que la base de datos persistía.

Estas pruebas demostraron que:

Los volúmenes LVM estaban correctamente montados.

Los contenedores estaban utilizando los volúmenes asignados.

Los datos eran persistentes después de reinicios.


CONCLUSIONES
El proyecto permitió implementar un entorno completo de infraestructura que combina:

Tolerancia a fallos → gracias a RAID 1

Flexibilidad de almacenamiento → mediante LVM

Portabilidad y aislamiento → mediante contenedores Docker

Persistencia garantizada → gracias al uso de volúmenes montados en los contenedores

La integración de estas tecnologías demuestra una arquitectura sólida, escalable y confiable, similar a la utilizada en entornos empresariales.

El sistema final cumple al 100% con los requerimientos establecidos en el proyecto, y se documentó de forma completa en esta bitácora.
