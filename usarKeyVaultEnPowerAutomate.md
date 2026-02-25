# 🔐 Guía de Integración: Azure Key Vault y Power Automate
Este documento detalla los pasos necesarios para habilitar, configurar y consumir secretos de Azure Key Vault desde flujos de Power Automate (Cloud y Desktop) utilizando el modelo de permisos RBAC, permitiendo ejecuciones desatendidas seguras.

## Fase 1: Preparación de la Suscripción en Azure
Antes de que Power Automate pueda comunicarse con Azure, es obligatorio registrar el proveedor de recursos en la suscripción.

Ingresa al Portal de Azure.

Ve al servicio de Suscripciones (Subscriptions) y selecciona la suscripción correspondiente (ej. 47ba6a2f-1d42-4950-b95a-cde3bb28d1eb).

En el menú lateral izquierdo, bajo la sección Configuración, selecciona Proveedores de recursos (Resource providers).

En la barra de búsqueda, escribe Microsoft.PowerPlatform.

Selecciónalo y haz clic en el botón Registrar (Register) en la parte superior.

Nota: Espera unos minutos hasta que el estado cambie de Registering a Registered.

## Fase 2: Creación del Key Vault y Permisos de Administrador
Incluso siendo el Owner (Dueño) de la suscripción, Azure separa el control del recurso del acceso a los datos internos (secretos).

Ve a Grupos de recursos y selecciona tu grupo (ej. RPA).

Crea un nuevo Key Vault (ej. RPAkey) asegurándote de que el modelo de permisos esté configurado en Control de acceso basado en roles de Azure (RBAC).

Una vez creado, entra al Key Vault y ve a Control de acceso (IAM).

Haz clic en + Agregar > Agregar asignación de roles.

Selecciona el rol Oficial de secretos de Key Vault (Key Vault Secrets Officer). Este rol permite crear, ver y administrar los secretos.

En la pestaña Miembros, asígnate este rol a tu propio usuario/correo de Azure.

Haz clic en Revisar y asignar.

⏳ Importante: Los cambios de RBAC tardan entre 5 y 10 minutos en propagarse. Espera este tiempo antes de intentar crear tu primer secreto.

## Fase 3: Permisos para Power Automate (Dataverse)
Para que Power Automate pueda extraer la credencial, el motor interno de la plataforma (Dataverse) necesita permisos de lectura explícitos sobre el Key Vault.

Dentro de tu Key Vault (RPAkey), ve nuevamente a Control de acceso (IAM).

Haz clic en + Agregar > Agregar asignación de roles.

Selecciona el rol Usuario de secretos de Key Vault (Key Vault Secrets User). Este rol permite solo lectura, garantizando que el flujo no pueda reescribir ni borrar la llave.

En la pestaña Miembros, selecciona Usuario, grupo o entidad de servicio y haz clic en + Seleccionar miembros.

En el buscador, escribe Dataverse (o busca el ID 00000007-0000-0000-c000-000000000000) y selecciona la aplicación.

Haz clic en Revisar y asignar.

⏳ Importante: Nuevamente, espera de 5 a 15 minutos para que este permiso surta efecto en la nube.

## Fase 4: Configuración en Power Automate
A. Crear la Credencial en la Nube
Ve al portal web de Power Automate.

Asegúrate de estar en el Entorno (Environment) correcto.

En la sección de credenciales, crea una nueva apuntando a Azure Key Vault.

Llena los datos exactamente como aparecen en Azure:

Subscription ID: 47ba6a2f-1d42-4950-b95a-cde3bb28d1eb

Resource Group: RPA

Key Vault Name: RPAkey

Secret Name: El nombre exacto de tu secreto.

B. Consumir la Credencial en Power Automate Desktop (PAD)
Para que la máquina local pueda recuperar la contraseña de la nube de forma segura:

Registrar la máquina: Abre la aplicación principal de Power Automate, ve a Configuración > Máquina y haz clic en Registrar máquina.

Reiniciar el servicio: Haz clic derecho en el ícono de Power Automate en la bandeja del sistema de Windows (junto al reloj) y selecciona Salir para limpiar la caché de conexiones. Vuelve a abrir la aplicación.

Abre tu flujo en el diseñador de PAD.

Arrastra la acción Obtener credencial (Get credential).

Selecciona la credencial creada en el paso A. Al ejecutar, puede tomar unos 7 a 10 segundos recuperar el valor desde Azure.

Uso en el flujo: Utiliza la variable generada llamando a sus propiedades (ej. %GettedCredential.Password%). El valor se manejará como Sensible y no dejará rastros en los logs.

🛠️ Solución de Problemas Comunes
Error: "The operation is not allowed by RBAC" al crear un secreto: Tu usuario no tiene el rol de Key Vault Secrets Officer asignado, o el permiso aún no se ha propagado.

Error: "Could not verify your user permissions... Microsoft.PowerPlatform provider is registered": Falta completar la Fase 1 (Registrar el proveedor en la suscripción).

Error: "Error genérico de cliente" en PAD al obtener credencial: Ocurre por caché atascada tras registrar la máquina o falta de propagación de RBAC de Dataverse. Cierra PAD por completo desde la bandeja del sistema y vuelve a intentarlo.

Firewall de Azure: Si persisten los errores de conexión, verifica en el Key Vault > Redes que el acceso público esté permitido temporalmente para descartar bloqueos de IP.
