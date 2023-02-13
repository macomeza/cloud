# Arquitectura de cómputo en nube

La nube nos ofrece muchísimas opciones, entre ellas nos ofrece múltiples tipos de cómputo, dependiendo de nuestro caso de uso podemos aprovechar las ventajas que nos ofrecen los siguientes 3 tipos.

## Máquina virtual

Es la solución más común en los centros de datos instalados en casa, consiste en compartir recursos (memoria, procesador y almacenamiento) de un servidor físico en varias instancias que corren su propio sistema operativo y cumplen sus funciones específicas.

Un servidor físico tiene un sistema operativo anfitrión que ejecuta un hypervisor y encima de este corre cada instancia con su propio sistema operativo, librerías y aplicaciones.

Caso de uso común, queremos llevar a la nube un servicio / aplicación en poco tiempo. 

El usuario es responsable de administrar actualizaciones del sistema operativo, gestión de seguridad, etc. El proveedor de nube es responsable de mantener el clúster.

## Contenedores / Kubernetes

El contenedor permite virtualización de servidores, la diferencia principal vs las máquinas virtuales es que el servidor tiene un sistema operativo anfitrión, encima de este corre un manejador de contenedores y cada contenedor se encarga de responder por cada aplicación.

Casos de uso, poder tener más control en replicar ambientes (test/dev/prod) y poderlos administrar desde línea de comando y kit de desarrollo de software.

En este tipo de servicio, el cliente es responsable de lo que ejecute cada contenedor, dejando la gestión de actualizaciones del sistema operativo y el gestor de contenedores. 

## Serverless

Es una arquitectura en la cual el usuario se preocupa únicamente del código a ejecutar, sin tomar en cuenta el tipo de sistema operativo, de la cantidad de recursos que se requiera para poder procesar las peticiones.

El proveedor de nube se encarga de toda la infraestructura y el cliente de la funcionalidad.

Casos de uso, microservicios, respuestas avanzadas a eventos.
