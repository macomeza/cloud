# Qué son las capas en Lambda y para qué me sirven?

Son una característica que permite añadir dependencias o librerías para ser utilizada por nuestras funciones.

## Pre requisitos

Tener instalado Python en nuestra computadora.
Tener instalado WSL (Windows Subsystem for Linux).
## Cómo se crean?

Las librerías que podemos subir a los layers de Lambda tienen algunos requisitos básicos, deben ser un .zip y deben tener todos sus folders dentro de una carpeta llamada python (con todas minúsculas). Una forma sencilla de hacerlo es desde el WSL si usamos Windows.

Ingresar a WSL, teclear lo siguiente:

```
python -m pip install geopandas -t python/
zip -r geopandas.zip python
```
En el ejemplo, deseamos hacer un zip para crear un layer con geopandas. Para instalar otra librería, reemplazan el nombre de la librería. Deben ser pacientes, pues algunas librerías pueden tomar algunos minutos para descargarse en sus computadoras. El nombre del zip puede hacer referencia a la librería y versión (opcionalmente).

Ingresamos a Lambda, luego en el menú lateral hacemos clic, luego seleccionamos Layers, luego le damos clic en "Create Layer". Le damos un nombre (añadir la versión para que sea fácil saber si es compatible con lo demás), luego podemos cargar un archivo zip o podemos precargar la libreria a un bucket, si escogemos la opción de cargar desde el bucket, debemos tener el arn de nuestra libreria en .zip.

Luego seleccionamos la arquitectura (lo más seguro es que sea x86_64.

En runtimes compatibles, marquemos los que soporta la librería, lo más común es que sean las últimas 5 versiones de Python, esto dará flexibilidad al programador a poder usar el Layer en funciones nuevas y existentes.

Finalmente le damos clic en "Create".

## Cómo se utilizan?

En nuestra función Lambda, en la parte superior veremos el nombre de la función seguida de Layers, le daremos clic en Layers, nos llevará al bloque donde podemos ver si hay Layer pre asociados o bien le podemos dar clic en "Add a layer" para añadir la que acabamos de crear.

Usaremos la opción de Custom layers y en la lista seleccionaremos la de nuestro interés y en versión se recomienda seleccionar la más reciente. Finalmente le damos clic en "Add".

Con esto ya podemos hacer el import hacia la librería de nuestro interés.


