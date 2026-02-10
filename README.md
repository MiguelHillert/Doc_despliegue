# Estandarización de Entornos de Desarrollo (Dev Containers)

![Docker](https://img.shields.io/badge/docker-%230db7ed.svg?style=for-the-badge&logo=docker&logoColor=white)
![Visual Studio Code](https://img.shields.io/badge/Visual%20Studio%20Code-0078d7.svg?style=for-the-badge&logo=visual-studio-code&logoColor=white)
![Linux](https://img.shields.io/badge/Linux-FCC624?style=for-the-badge&logo=linux&logoColor=black)

Este repositorio contiene las **plantillas oficiales de configuración**, la documentación técnica y las guías de implementación para los entornos de desarrollo contenerizados de la compañía.

El objetivo es eliminar la fricción en la configuración de entornos locales ("en mi máquina funciona"), garantizar la seguridad mediante prácticas de **DevSecOps** y optimizar el uso de recursos en equipos con hardware limitado.

## 📂 Contenido del Repositorio

Este proyecto proporciona configuraciones de **Infraestructura como Código (IaC)** para los siguientes stacks tecnológicos:

| Stack | Tecnología Base | Enfoque Principal | Configuración |
| :--- | :--- | :--- | :--- |
| **Scripting** | Python 3.12 | **Seguridad** (Usuario no-root `vscode`) | [Ver Código](./script-python) |
| **Backend** | .NET 9.0 SDK | **Productividad** (Herramientas pre-instaladas) | [Ver Código](./backend-net) |
| **Frontend** | Angular 19 / Node 22 | **Rendimiento** (Volúmenes para `node_modules`) | [Ver Código](./frontend-angular) |

## 📚 Documentación Técnica

Para una explicación detallada sobre la arquitectura, las decisiones de diseño (Dockerfiles personalizados vs. imágenes genéricas) y el procedimiento paso a paso de implementación, consulta el informe técnico completo:

👉 **[LEER INFORME TÉCNICO DE IMPLEMENTACIÓN](./INFORME_TECNICO.md)**

*(Nota: Asegúrate de que el archivo markdown largo que generamos antes se llame `INFORME_TECNICO.md` o actualiza este enlace)*

## 🚀 Inicio Rápido

### Prerrequisitos
* **Sistema Operativo:** Ubuntu 22.04 LTS (Recomendado) o Windows/macOS.
* **Motor:** Docker Engine (Linux Nativo) o Docker Desktop.
* **IDE:** Visual Studio Code con la extensión [Dev Containers](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers).

### Cómo usar estas plantillas

1.  **Clonar el repositorio:**
    ```bash
    git clone [https://github.com/tu-usuario/dev-containers-standards.git](https://github.com/tu-usuario/dev-containers-standards.git)
    ```
2.  **Copiar la configuración:**
    Copia la carpeta `.devcontainer` del stack que necesites a la raíz de tu proyecto.
    * *Ejemplo:* Si tienes un proyecto de Python, copia `script-python/.devcontainer` a tu carpeta de proyecto.
3.  **Iniciar:**
    Abre tu proyecto en VS Code y selecciona **"Reopen in Container"** cuando aparezca la notificación.

## 🛠️ Estructura del Repositorio

```text
├── img/                   # Evidencias y capturas de pantalla
├── INFORME_TECNICO.md     # Manual detallado de implementación
└── README.md              # Este archivo
