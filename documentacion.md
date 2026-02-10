# Manual de Despliegue de Entornos DevSecOps con Dev Containers

**Autor:** [Tu Nombre]
**Asignatura:** Administración de Sistemas / DevSecOps
**Entorno:** Ubuntu 22.04 LTS (Nativo) | CPU Intel i3 | 16GB RAM

---

## 1. Introducción y Fundamentos

El objetivo de esta práctica es implementar entornos de desarrollo reproducibles y aislados utilizando **Dev Containers**. [cite_start]Esta tecnología soluciona el problema de "funciona en mi máquina" al desacoplar las herramientas de desarrollo del sistema operativo anfitrión[cite: 6].

### Estrategia de Imágenes
[cite_start]Basándonos en la investigación realizada, el uso de imágenes propias (*custom images*) es una práctica estándar en entornos empresariales para garantizar la seguridad y el cumplimiento normativo[cite: 25]. [cite_start]Hemos optado por la estrategia de **Construcción vía Dockerfile**, lo que nos permite versionar la infraestructura como código (IaC) junto con el proyecto[cite: 38].

---

## 2. Preparación del Host (Ubuntu 22.04)

Debido a las limitaciones de hardware (CPU i3), se ha optado por utilizar **Docker Engine nativo** en Linux en lugar de Docker Desktop para eliminar la sobrecarga de virtualización.

### 2.1 Resolución de Conflictos y Dependencias
Durante la instalación, se detectó un conflicto con los módulos del kernel de VirtualBox (`virtualbox-dkms`) que bloqueaba el gestor de paquetes.

**Comandos ejecutados para la reparación e instalación:**

```bash
# 1. Eliminación del módulo conflictivo de VirtualBox
sudo apt-get remove virtualbox-dkms

# 2. Reconfiguración de paquetes pendientes
sudo dpkg --configure -a

# 3. Instalación limpia de Docker Engine
sudo apt-get install docker.io docker-compose -y

# 4. Configuración de permisos (para ejecutar sin sudo)
sudo usermod -aG docker $USER
newgrp docker
```

> **[INSERTAR PANTALLAZO 1: Terminal mostrando `docker run hello-world` con éxito]**
> *Evidencia de que el motor Docker está operativo tras la reparación.*

---

## 3. Implementación de Entornos

Se han configurado tres entornos distintos, cada uno demostrando una competencia clave de DevSecOps documentada en la investigación.

### 3.1 Entorno Scripting (Python) - Competencia: Seguridad
Para este entorno, se prioriza la seguridad siguiendo el principio de **mínimo privilegio**. [cite_start]En lugar de ejecutar el contenedor como `root`, se configura un usuario estándar[cite: 125].

* [cite_start]**Estrategia:** Creación de usuario no-root (`vscode`) alineado con el UID 1000 del host para evitar problemas de permisos de archivos[cite: 127].
* [cite_start]**Imagen Base:** `python:3.12-slim-bookworm`[cite: 116].

**Archivo:** `script-python/.devcontainer/Dockerfile`

```dockerfile
FROM python:3.12-slim-bookworm

# Evitar archivos temporales y buffer
ENV PYTHONDONTWRITEBYTECODE=1
ENV PYTHONUNBUFFERED=1

# SEGURIDAD: Creación de usuario no-root
ARG USERNAME=vscode
ARG USER_UID=1000
ARG USER_GID=$USER_UID

RUN groupadd --gid $USER_GID $USERNAME \
    && useradd --uid $USER_UID --gid $USER_GID -m $USERNAME \
    # Instalación de sudo y git
    && apt-get update \
    && apt-get install -y sudo git \
    # Configuración de sudo sin contraseña para el usuario
    && echo "$USERNAME ALL=(ALL) NOPASSWD:ALL" > /etc/sudoers.d/$USERNAME \
    && chmod 0440 /etc/sudoers.d/$USERNAME

# Instalación de herramientas de calidad
RUN pip install --no-cache-dir black pylint pytest

# Cambiar al usuario seguro
USER $USERNAME
```

**Archivo:** `script-python/.devcontainer/devcontainer.json`

```json
{
  "name": "Python Seguro",
  "build": { "dockerfile": "Dockerfile" },
  "remoteUser": "vscode",
  "customizations": {
    "vscode": {
      "settings": {
        "python.defaultInterpreterPath": "/usr/local/bin/python",
        "python.linting.enabled": true
      },
      "extensions": ["ms-python.python"]
    }
  },
  "postCreateCommand": "pip install --user -r requirements.txt"
}
```

> **[INSERTAR PANTALLAZO 2: Terminal con `whoami` y `python --version`]**
> *Se verifica que el usuario activo es 'vscode' y no 'root'.*

---

### 3.2 Entorno Backend (.NET) - Competencia: Productividad
El objetivo es reducir el tiempo de configuración diario.

* **Estrategia:** Pre-instalación de herramientas globales en la imagen ("hornear" la imagen). [cite_start]Se incluye `dotnet-ef` (Entity Framework) directamente en el Dockerfile para que esté disponible inmediatamente[cite: 89].
* [cite_start]**Imagen Base:** `mcr.microsoft.com/dotnet/sdk:9.0-bookworm-slim`[cite: 79].

**Archivo:** `backend-net/.devcontainer/Dockerfile`

```dockerfile
FROM [mcr.microsoft.com/dotnet/sdk:9.0-bookworm-slim](https://mcr.microsoft.com/dotnet/sdk:9.0-bookworm-slim)

# Instalación de dependencias del sistema y herramienta EF Core
RUN apt-get update && apt-get install -y git procps curl \
    && dotnet tool install --global dotnet-ef

# Añadir herramientas al PATH es crítico para que la terminal las reconozca
ENV PATH="$PATH:/root/.dotnet/tools"

# Certificados de desarrollo HTTPS
RUN dotnet dev-certs https --clean && dotnet dev-certs https --trust
```

**Archivo:** `backend-net/.devcontainer/devcontainer.json`

```json
{
  "name": "Backend .NET Core",
  "build": { "dockerfile": "Dockerfile" },
  "forwardPorts": [5000, 5001],
  "postCreateCommand": "dotnet --version",
  "remoteUser": "root"
}
```

> **[INSERTAR PANTALLAZO 3: Terminal ejecutando `dotnet --list-sdks`]**
> *Confirmación de que el SDK 9.0 está listo para compilar.*

---

### 3.3 Entorno Frontend (Angular) - Competencia: Rendimiento
[cite_start]El desafío principal en contenedores de Node.js es la lentitud del sistema de archivos al sincronizar la carpeta `node_modules` en sistemas Windows/macOS, aunque en Linux nativo es menos crítico, es una buena práctica de aislamiento[cite: 106].

* **Estrategia:** Uso de **Volúmenes Nombrados (Named Volumes)**. [cite_start]Se configura el `devcontainer.json` para montar `node_modules` en un volumen interno de Docker[cite: 107].
* [cite_start]**Imagen Base:** `node:22-bookworm-slim` emparejada con Angular 19 para evitar errores de versiones[cite: 100, 111].

**Archivo:** `frontend-angular/.devcontainer/Dockerfile`

```dockerfile
FROM node:22-bookworm-slim

# Instalación global de Angular CLI fijando versión
RUN npm install -g @angular/cli@19.0.0

RUN apt-get update && apt-get install -y git

# Uso del usuario predeterminado de Node
USER node
```

**Archivo:** `frontend-angular/.devcontainer/devcontainer.json`

```json
{
  "name": "Angular Frontend",
  "build": { "dockerfile": "Dockerfile" },
  "remoteUser": "node",
  "forwardPorts": [4200],
  "mounts": [
    "source=node_modules_cache,target=/workspace/node_modules,type=volume"
  ]
}
```

> **[INSERTAR PANTALLAZO 4: Terminal ejecutando `ng version`]**
> *Verificación de Angular CLI v19 instalado sobre Node v22.*

---

## 4. Conclusiones y Referencias

Mediante esta práctica se ha logrado:
1.  **Recuperar un entorno inestable:** Solucionando conflictos de dependencias en Linux.
2.  [cite_start]**Implementar Seguridad:** Usando usuarios no privilegiados en Docker[cite: 125].
3.  [cite_start]**Optimizar Rendimiento:** Gestionando volúmenes para dependencias pesadas[cite: 109].
4.  **Documentar como Código:** Toda la infraestructura está definida en archivos, lista para ser versionada en Git.

**Referencias:**
*Informe de Investigación: Configuración de Dev Containers Personalizados.*
