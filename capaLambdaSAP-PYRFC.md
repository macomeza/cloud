Guía Definitiva: Creación de AWS Lambda Layer para SAP RFC (Python 3.12)
Esta guía detalla el procedimiento para empaquetar el SAP NW RFC SDK y la librería pyrfc utilizando Docker. Este método garantiza la compatibilidad binaria estricta con Amazon Linux 2023, el sistema operativo base del runtime de Python 3.12 en AWS Lambda.

🛠️ 1. Prerrequisitos
WSL con Ubuntu (o cualquier entorno Linux).

Docker instalado y corriendo.

El archivo del SDK de SAP: nwrfc750P_18-70002752.zip (descargado del portal de SAP).

📁 2. Preparación del Entorno Local
Crea un directorio de trabajo y mueve el archivo ZIP del SDK dentro de él.

```
mkdir -p ~/lambda-sap-build
cd ~/lambda-sap-build

# Asegúrate de que el ZIP esté aquí
ls -la nwrfc750P_18-70002752.zip
```
🐳 3. Creación del Dockerfile
Crea un archivo llamado Dockerfile en el mismo directorio. Este archivo utilizará la imagen oficial de AWS para compilar la librería nativamente.

```
nano Dockerfile
```
Pega el siguiente contenido exacto:

Dockerfile
```
# Usamos la imagen oficial de AWS Lambda para Python 3.12 (Amazon Linux 2023)
FROM public.ecr.aws/sam/build-python3.12:latest

# Crear directorios con la estructura estricta que exige Lambda
RUN mkdir -p /opt/layer/python && \
    mkdir -p /opt/layer/lib && \
    mkdir -p /opt/nwrfcsdk

# Copiar el ZIP del SDK al contenedor
COPY nwrfc750P_18-70002752.zip /tmp/

# Descomprimir el SDK y mover los archivos .so a la carpeta lib de la Layer
RUN unzip /tmp/nwrfc750P_18-70002752.zip -d /opt/ && \
    cp /opt/nwrfcsdk/lib/*.so* /opt/layer/lib/

# Configurar variables de entorno requeridas por el compilador de pyrfc
ENV SAPNWRFC_HOME=/opt/nwrfcsdk
ENV LD_LIBRARY_PATH=/opt/nwrfcsdk/lib

# Instalar pyrfc forzando la compilación desde las fuentes (--no-binary)
# Esto garantiza que se enlace con el glibc nativo de Amazon Linux 2023
RUN pip install --target /opt/layer/python --no-binary pyrfc pyrfc==3.3.1

# Directorio de trabajo y comando de salida
WORKDIR /opt/layer
CMD ["zip", "-r", "/tmp/sap_layer_al2023.zip", "python", "lib"]
```
🏗️ 4. Construcción y Extracción del ZIP
Construye la imagen de Docker y luego ejecuta un contenedor efímero para extraer el ZIP generado hacia tu máquina local.

```
# 1. Construir la imagen (puede tardar un par de minutos)
docker build -t build-sap-layer .

# 2. Ejecutar el contenedor y montar la carpeta actual para extraer el ZIP
docker run --rm -v $(pwd):/tmp build-sap-layer
```
¡Listo! Ahora tienes el archivo sap_layer_al2023.zip en tu carpeta, 100% compatible con AWS.

☁️ 5. Configuración en AWS Lambda
1. Ve a Lambda > Layers y crea una nueva capa subiendo el archivo sap_layer_al2023.zip. Selecciona x86_64 y Python 3.12.

2. Asocia la capa a tu función Lambda.

3. En la función, ve a Configuration > Environment variables y añade:

Key: ```LD_LIBRARY_PATH```

Value: ```/opt/lib:/var/lang/lib:/lib64:/usr/lib64:/var/runtime:/var/runtime/lib:/var/task:/var/task/lib```

🧠 6. Plantilla Base de Código
Gracias a la compilación nativa en Docker, las importaciones se realizan de forma estándar, manteniendo el código limpio:

Python
```
import re
import ast
import boto3
import os
import json
import pandas as pd
from pyrfc import Connection

# Importaciones locales SIEMPRE absolutas (sin punto inicial)
from utils.compress import comprimir_payload

# --- HANDLER ---
def lambda_handler(event, context):
    # Lógica de conexión y ejecución de BAPIs
    pass
```
