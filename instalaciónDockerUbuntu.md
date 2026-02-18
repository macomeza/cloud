## Instalación de Docker en WSL (Ubuntu)

Antes de comenzar con la creación de la capa, asegúrate de tener Docker instalado y funcionando en tu instancia de Ubuntu.

### Instalación de Docker Engine
Ejecuta los siguientes comandos en tu terminal de Ubuntu:

```
# 1. Actualizar el índice de paquetes e instalar dependencias iniciales
sudo apt-get update
sudo apt-get install -y ca-certificates curl gnupg

# 2. Agregar la clave GPG oficial de Docker
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL [https://download.docker.com/linux/ubuntu/gpg](https://download.docker.com/linux/ubuntu/gpg) | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg

# 3. Configurar el repositorio oficial
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] [https://download.docker.com/linux/ubuntu](https://download.docker.com/linux/ubuntu) \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# 4. Instalar Docker
sudo apt-get update
sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

Configuración de permisos (Post-instalación)
Para ejecutar Docker sin necesidad de usar sudo en cada comando, añade tu usuario al grupo docker:

```
sudo usermod -aG docker $USER
```
Importante: Debes cerrar la terminal de WSL y volver a abrirla para que este cambio surta efecto.

Iniciar el servicio
En WSL, es posible que el servicio de Docker no inicie automáticamente. Puedes verificarlo e iniciarlo con:

```
sudo service docker start
```
