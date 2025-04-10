#Creando una Capa de AWS Lambda con Geopandas para Python 3.12 Usando Docker (¡Guía para Principiantes!)

¿Necesitas usar Geopandas en tus funciones Lambda de AWS con Python 3.12, pero te encuentras con problemas de compatibilidad? ¡No te preocupes! Crear una "capa" (Lambda Layer) personalizada es la solución. Las capas te permiten empaquetar librerías y dependencias para que tus funciones Lambda las puedan usar sin exceder los límites de tamaño.

Usaremos Docker para asegurarnos de que todo sea compatible con el entorno de ejecución de Lambda (Amazon Linux 2023). ¡Vamos paso a paso!

¿Qué necesitas?

Tener Docker instalado en tu computadora.
Una cuenta de AWS.
La consola de AWS (AWS CLI) configurada (opcional, pero útil).
Paso 1: Prepara tu Carpeta de Proyecto    

Crea una carpeta nueva en tu computadora. Llamémosla geo-lambda-layer.   
Dentro de esta carpeta, crea un archivo llamado requirements.txt.   
Paso 2: Define las Dependencias (requirements.txt)    

Abre el archivo requirements.txt y añade las librerías de Python que necesitarás. Para Geopandas, usualmente basta con poner:

Plaintext

# requirements.txt
pandas
geopandas
# Nota: Geopandas instalará automáticamente otras librerías necesarias como shapely, fiona, pyproj, numpy, etc. [cite: 5]
# Puedes especificar versiones si lo necesitas, ej: geopandas==0.14.3 [cite: 5]
¡Guarda el archivo!    

Paso 3: Crea el Dockerfile    

En la misma carpeta geo-lambda-layer, crea un archivo llamado Dockerfile (¡sin extensión!).  Este archivo le dice a Docker cómo construir nuestro entorno.   

Pega el siguiente contenido en tu Dockerfile:

Dockerfile

# Dockerfile [cite: 7]

# Usamos la imagen base oficial de AWS Lambda para Python 3.12 [cite: 7]
# Elige la arquitectura correcta: x86_64 (la más común) o arm64 (si usas AWS Graviton) [cite: 7]
# Para x86_64:
FROM public.ecr.aws/lambda/python:3.12-x86_64 [cite: 7]
# Para arm64:
# FROM public.ecr.aws/lambda/python:3.12-arm64

# Creamos la estructura de carpetas que Lambda espera para las capas [cite: 7]
RUN mkdir -p /opt/python/lib/python3.12/site-packages

# --- ¡NUEVO PASO IMPORTANTE!: Instalar dependencias del sistema --- [cite: 7]
# Geopandas necesita la librería 'expat'. La instalamos aquí.
RUN echo "Instalando dependencias del sistema..." && \
    dnf upgrade -y && \
    dnf install -y expat findutils && \ # Instala expat y findutils [cite: 7]
    echo "Copiando librerías necesarias a la capa..." && \
    # Buscamos y copiamos los archivos de libexpat del sistema a la capa [cite: 8]
    # La ubicación común en AL2023 x86_64 es /usr/lib64/. Ajusta si usas arm64. [cite: 8, 9]
    mkdir -p /opt/python/lib/ && \ # Crea el directorio /opt/python/lib si no existe [cite: 9]
    cp -L /usr/lib64/libexpat.so* /opt/python/lib/ && \ # Copia los archivos .so de libexpat [cite: 9]
    dnf clean all && \ # Limpia la caché de dnf [cite: 9]
    echo "Dependencias del sistema procesadas y librerías copiadas." [cite: 9]

# Copiamos el archivo de requerimientos al contenedor [cite: 10]
COPY requirements.txt . [cite: 10]

# Actualizamos pip e instalamos los paquetes en el directorio correcto [cite: 11]
# ¡OJO! Usa el 'platform tag' correcto para tu arquitectura [cite: 11]
# Para x86_64: manylinux2014_x86_64
# Para arm64: manylinux2014_aarch64
RUN pip install --upgrade pip && \
    pip install \
        --platform manylinux2014_x86_64 \ # Cambia a manylinux2014_aarch64 si usas arm64 [cite: 11]
        --target=/opt/python/lib/python3.12/site-packages \
        --implementation cp \
        --python-version 3.12 \
        --only-binary=:all: \
        --requirement requirements.txt [cite: 11]

# --- ¡NUEVO PASO!: Reducir el tamaño de la Capa --- [cite: 11]
# Las capas tienen límites de tamaño, ¡así que vamos a limpiarla!
RUN echo "Iniciando reducción de tamaño..." [cite: 12]
# Eliminar caché de Python y archivos compilados [cite: 12]
RUN find /opt/python/lib/python3.12/site-packages -name "__pycache__" -type d -exec rm -rf {} + [cite: 12]
RUN find /opt/python/lib/python3.12/site-packages -name "*.pyc" -type f -delete [cite: 12]

# Eliminar carpetas de pruebas comunes [cite: 12]
RUN find /opt/python/lib/python3.12/site-packages -type d -name "tests" -prune -exec rm -rf {} + [cite: 12]
RUN find /opt/python/lib/python3.12/site-packages -type d -name "test" -prune -exec rm -rf {} + [cite: 12]

# Eliminar metadatos de distribución (¡CUIDADO! Puede romper algo, prueba comentándolo si falla) [cite: 12]
RUN find /opt/python/lib/python3.12/site-packages -type d -name "*.dist-info" -prune -exec rm -rf {} + [cite: 12]
RUN find /opt/python/lib/python3.12/site-packages -type d -name "*.egg-info" -prune -exec rm -rf {} + [cite: 13]

# Eliminar datos/docs grandes y no esenciales (Ejemplos, ajusta según veas necesario) [cite: 13]
RUN rm -rf /opt/python/lib/python3.12/site-packages/pandas/tests/data || echo "Datos de test de Pandas no encontrados, omitiendo." [cite: 13, 14]
# RUN rm -rf /opt/python/lib/python3.12/site-packages/geopandas/datasets || echo "Datasets de GeoPandas no encontrados, omitiendo." [cite: 14, 15]

# Opcional: Eliminar símbolos de archivos .so (ahorra espacio pero dificulta depurar) [cite: 15, 16]
# Descomenta la siguiente línea si necesitas ahorrar mucho espacio, pero pruébalo sin ella primero. [cite: 17]
# RUN dnf install -y binutils && find /opt/python/lib/python3.12/site-packages -name "*.so" -type f -exec strip {} \; && dnf remove -y binutils && dnf clean all [cite: 18, 19]

RUN echo "Reducción de tamaño finalizada." [cite: 19]
# --- Fin Reducción de Tamaño --- [cite: 20]

# Opcional: Limpiar caché de pip [cite: 20]
RUN rm -rf /root/.cache/pip

# --- fin del Dockerfile --- [cite: 20]
Paso 4: Construye la Imagen Docker y Extrae la Capa

Abre tu terminal o línea de comandos, navega hasta la carpeta geo-lambda-layer y ejecuta estos comandos:    

Construir la imagen Docker: (Esto puede tardar un poco la primera vez)

Bash

docker build -t geo-layer-builder . [cite: 20]
Crear una carpeta para la salida:

Bash

mkdir layer-output
Ejecutar un contenedor temporal para copiar los archivos de la capa:

Bash

# En Linux/macOS:
docker run --rm --entrypoint "" -v "$(pwd)/layer-output:/output" geo-layer-builder cp -r /opt/python /output/ [cite: 21]
# En Windows (PowerShell):
# docker run --rm --entrypoint "" -v "${PWD}/layer-output:/output" geo-layer-builder cp -r /opt/python /output/
# En Windows (CMD):
# docker run --rm --entrypoint "" -v "%CD%/layer-output:/output" geo-layer-builder cp -r /opt/python /output/
Paso 5: Empaqueta la Capa (Archivo ZIP)

Entra a la carpeta de salida:
Bash

cd layer-output [cite: 21]
Verás una carpeta llamada python. Esta es la estructura que Lambda necesita.
Crea el archivo ZIP:
Bash

# En Linux/macOS:
zip -r ../geo-lambda-layer.zip python [cite: 21]
# En Windows (si no tienes 'zip', puedes usar la compresión integrada de Windows o instalar '7-Zip'):
# Click derecho en la carpeta 'python' -> Enviar a -> Carpeta comprimida (en zip). Renombra el ZIP a 'geo-lambda-layer.zip' y muévelo a la carpeta 'geo-lambda-layer'.
Regresa a la carpeta principal (opcional):
Bash

cd ..
Ahora deberías tener un archivo geo-lambda-layer.zip en tu carpeta geo-lambda-layer.

Paso 6: Sube la Capa a AWS Lambda    

Ve a la consola de AWS Lambda.   
En el menú de la izquierda, haz clic en "Capas" (Layers).   
Haz clic en "Crear capa" (Create layer).   
Dale un nombre descriptivo, por ejemplo: GeopandasPython312.   
Sube el archivo geo-lambda-layer.zip que acabas de crear.   
Selecciona la "Arquitectura compatible" (x86_64 o arm64, la que usaste en el Dockerfile).   
Selecciona el "Tiempo de ejecución compatible" (Python 3.12).   
Haz clic en "Crear" (Create).   
Paso 7: Asocia la Capa a tu Función Lambda    

Ve a la página de configuración de tu función Lambda en la consola.   
Baja hasta la sección "Capas" (Layers) y haz clic en "Agregar una capa" (Add a layer).   
Elige "Capas personalizadas" (Custom layers).   
Selecciona la capa que acabas de crear (GeopandasPython312) y la versión (probablemente la 1).   
Haz clic en "Agregar" (Add).   
Paso 8: ¡Configuración Final Importante! Variable de Entorno    

Debido a cómo Geopandas y sus dependencias buscan librerías del sistema (como libexpat.so que copiamos antes), necesitamos decirle a Lambda dónde encontrarla dentro de la capa.

En la página de configuración de tu función Lambda, ve a "Configuración" (Configuration) > "Variables de entorno" (Environment variables).   
Haz clic en "Editar" (Edit).   
Haz clic en "Agregar variable de entorno" (Add environment variable).   
Configúrala así:
Clave (Key): LD_LIBRARY_PATH    
Valor (Value): /opt/python/lib    
Haz clic en "Guardar" (Save).
¡Listo! 🎉

Ahora tu función Lambda con Python 3.12 debería poder importar y usar Geopandas y Pandas sin problemas. ¡Ya puedes empezar a procesar datos geoespaciales en la nube!

Espero que esta guía te sea útil. ¡Mucha suerte con tus proyectos!
