# Infraestructura SIE

Este repositorio contiene la configuración necesaria para levantar el ecosistema de Sistemas de Información Empresariales mediante Docker.

## 📂 Estructura del Repositorio

* `odoo/`:
    * `addons/`: Carpeta para tus módulos personalizados.
    * `config/`: Contiene el fichero `odoo.conf` de configuración.
* `suitecrm/`:
    * `SuiteCRM-8.9.1/`: Código fuente de la aplicación.
    * `languages/`: Contiene el pack de idioma español (.zip) listo para instalar tras lanzar el servicio. Puedes descargar y añadir aquí otros idiomas si lo deseas (más abajo se describe cómo hacerlo).
    * `upload/`: Carpeta local para persistir archivos subidos al CRM.
    * `Dockerfile`: Instrucciones de construcción de la imagen de PHP personalizada.
* `bonita/`: 
    * `exports/`: Directorio recomendado para guardar tus ficheros de Bonita Studio (`.bos`, `.bar`,...).
* `n8n/`:
    * `workflows/`: Carpeta donde exportar tus flujos exportados manualmente (`.json`) desde n8n.
* `docker-compose.yml`: Archivo principal para orquestar todos los servicios.
* `Guía de Instalación_SIE.pdf`: Documento complementario con información general y pasos para el proceso de instalación.

## 🛠️ Requisitos y Herramientas Externas

Antes de comenzar, asegúrate de tener instaladas las siguientes herramientas en tu equipo:

1.  **Docker Desktop:** [Descargar](https://www.docker.com/products/docker-desktop/). **Imprescindible.** Es el motor que permite ejecutar todos los servicios (Odoo, SuiteCRM, etc.) contenidos en este repositorio.
2.  **Bonita Studio 2023.2:** [Descargar](https://www.bonitasoft.com/es/old-versions). Necesario para diseñar y modelar tus procesos de negocio, que posteriormente se ejecutarán en el motor (Bonita Runtime) incluido en el `docker-compose.yml`.
3.  **Git:** [Descargar](https://git-scm.com/downloads). **Recomendado.** Permite mantener tu repositorio actualizado y gestionar versiones. Si no deseas usarlo, puedes descargar el repositorio como un archivo ZIP.
4.  **Cuenta de GitHub:** Necesaria para crear y alojar tu propio repositorio a partir de la plantilla que proporcionamos.

## 🚀 Inicio Rápido para Alumnos

1.  **Crear tu propio repositorio:** Inicia sesión en GitHub y pulsa el botón verde **"Use this template"** arriba a la derecha en el repositorio del curso.
2.  **Nombre del repositorio:** Es OBLIGATORIO que siga el formato: `sie-UVUS` (siendo `UVUS` tu propio UVUS).
3.  **Privacidad:** Privado (si fuera necesario el profesor te pedirá que lo añadas como colaborador).
4.  **Clonar o Descargar:** Usa `git clone <tu-nueva-url>` o descarga el ZIP y descomprímelo.
5.  **Arrancar:** Entra en la carpeta desde una terminal y ejecuta: `docker compose up -d --build`.
6.  **Verificar:** Una vez que Docker Desktop indique que los contenedores están en verde, comprueba que puedes acceder a:
    * **Odoo:** [http://localhost:8069](http://localhost:8069)
    * **SuiteCRM:** [http://localhost:8080/public](http://localhost:8080/public)
    * **Bonita Runtime:** [http://localhost:8081/bonita](http://localhost:8081/bonita)
    * **n8n:** [http://localhost:5678](http://localhost:5678)
    * **smtp4dev:** [http://localhost:3000](http://localhost:3000)
    * **pgAdmin:** [http://localhost:5050](http://localhost:5050)

> **Nota sobre `--build`:** El parámetro `--build` solo es necesario la primera vez o si se modifica el `Dockerfile`. No te preocupes por tus datos; gracias a los volúmenes de Docker, no perderás configuraciones ni archivos aunque detengas los contenedores o reconstruyas la imagen.

## ⚙️ Configuración de Instalación y Red

### A. Instalación de SuiteCRM (Asistente Web)
A diferencia del resto de herramientas, SuiteCRM debe terminar de instalarse una vez lanzado el servicio. Para ello debemos acceder a la aplicación y seguir los pasos del asistente de instalación, que nos pedirá algunos datos que facilitamos a continuación.

Acceso: `http://localhost:8080/public`
* **Database Host:** `db_suitecrm`
* **Database Name:** `suitecrm_db`
* **Database User:** `suitecrm_user`
* **Database Password:** `suitecrm_pass`
* **URL Instance:** `http://localhost:8080/public`

### B. Idioma Español y Otros Idiomas en SuiteCRM
Para poner SuiteCRM en español, sigue estos pasos:
1. **Instalación:** Ve a **Admin** > **Module Loader**, sube el archivo `.zip` que está en la carpeta `suitecrm/languages/`, pulsa **Install** y luego **Commit**.
2. **Verificación:** Ve a **Admin** > **Languages**. Comprueba que el idioma "Spanish" aparece en la columna **Enabled**. Si no es así, muévelo y guarda.
3. **Selección:** Cierra sesión. En la pantalla de Login, verás un selector para elegir "Español".

> **Descarga de traducciones:** Puedes encontrar los paquetes de idioma listos para descargar en [SuiteCRM Translations (SourceForge)](https://sourceforge.net/projects/suitecrmtranslations/files/). Para las versiones más recientes o para colaborar en la traducción, visita [SuiteCRM Crowdin](https://crowdin.com/project/suitecrmtranslations).

### C. smtp4dev (Servidor de correo electrónico)
Entre las herramientas se encuentra un servidor de correo "fake2 (smtp4dev), que simula el envío de correos electrónicos para hacer pruebas sin necesidad de enviarlos realmente. Será util para configurar las demás herramientas y probar funcionalidades como la creación de nuevos usuarios en SuiteCRM de manera que reciban su contraseña de acceso por email.
También podemos usarlo para que nuestros procesos de negocio en Bonita Studio y Bonita Runtime incluyan una tarea que envíe un email, pero debemos tener en cuenta que la configuración será distinta en ambas herramientas al estar ejecutándose fuera y dentro de Docker respectivamente. A continuación mostramos los parámetros a usar en cada caso.
| Configuración | Desde Docker (Odoo/SuiteCRM/n8n/Bonita Runtime) | Desde fuera de Docker (Bonita Studio) |
| :--- | :--- | :--- |
| **Servidor (Host)** | `smtp4dev` | `localhost` |
| **Puerto SMTP** | `25` | `2525` |

> **Ver Emails:** Accede a [http://localhost:3000](http://localhost:3000) para ver los correos capturados.

### D. Bonita Runtime (BPM)
Tras lanzar el servicio el sistema no tendrá Organización, BDM ni procesos, que deberán ser desplegados por el usuario denominado "superadministrador", que es el único que inicialmente puede iniciar sesión con las credenciales `install / install`.

### E. Odoo (ERP)
Al acceder a Odoo por primera vez podremos crear una primera base de datos para gestionar nuestra organización (podemos crear varias, por ejemplo para realizar pruebas). En este punto tendremos que usar la **"Master password"** definida en el fichero `odoo/config/odoo.conf` que por defecto es `admin_password`. También será necesaria para crear nuevas bases de datos o realizar operaciones sobre las bases de datos que ya tengamos creadas. 

#### Módulos Personalizados en Odoo (Addons)
Si has añadido una carpeta de módulo en `odoo/addons/`, sigue estos pasos para que aparezca:
1. **Activar Modo Desarrollador:** Ve a **Ajustes** y, al final de la página, pulsa en **Activar modo desarrollador**.
2. **Actualizar Lista de Aplicaciones:** Ve al menú **Aplicaciones** y, en la barra superior, pulsa en **Actualizar lista de aplicaciones** > **Actualizar**.
3. **Instalar:** Busca tu módulo en el buscador (quita el filtro "Aplicaciones" si no aparece) y pulsa **Instalar**.
> **Nota:** Si has hecho cambios en el código Python del módulo, debes reiniciar el contenedor con `docker compose restart odoo`. Si solo has cambiado XML/CSS, basta con **Actualizar** el módulo desde la interfaz.

### F. n8n (iPaaS)
Esta herramienta nos permite crear flujos de trabajo que integren diferentes servicios de los que tengamos lanzados en esta infraestructura empresarial. Requerirá crear una cuenta local la primera vez que accedamos.

### G. pgAdmin (Gestión de Bases de Datos)
Usaremos esta herramienta para poder acceder directamente a las bases de datos PostgreSQL que usan Odoo y Bonita Runtime usando los siguientes datos:
* **Email:** `admin@sie.com`
* **Password:** `admin`

Para añadir los servidores, haz clic derecho en **Servers** > **Register** > **Server...** y usa la siguiente configuración:

**1. Servidor Odoo:**
* **General (Name):** `Odoo DB`
* **Connection (Host name/address):** `db_odoo`
* **Username:** `odoo`
* **Password:** `odoo_pass`

**2. Servidor Bonita:**
* **General (Name):** `Bonita DB`
* **Connection (Host name/address):** `db_bonita`
* **Username:** `bonita`
* **Password:** `bpm`

## Alternativas (Instalación Local)
Si por limitaciones de hardware o problemas de otra índole tu equipo no permite ejecutar Docker, hay otras opciones para instalar y ejecutar estas mismas herramientas por separado:

* Odoo: Visita la web oficial y descarga el instalador nativo o usa la versión cloud con restricciones en [https://www.odoo.com/es/page/download](https://www.odoo.com/es/page/download).
* SuiteCRM: Descarga los archivos desde la web oficial de SuiteCRM [https://suitecrm.com/download](https://suitecrm.com/download). Requiere un servidor con PHP y MySQL (como XAMPP). 
* n8n: Usa la versión community de la herramienta siguiendo las indicaciones de la documentación oficial [https://docs.n8n.io/choose-n8n](https://docs.n8n.io/choose-n8n).
* Bonita Runtime: Puedes descargarla desde la misma URL desde la que se descarga Bonita Studio, y recuerda que ambas deben ser de la misma versión. Si no pudiese llevarse a cabo la instalación de Bonita Runtime, Bonita Studio incluye un servidor local para pruebas rápidas que te permitirá validar los procesos que se diseñen.
* smtp4dev: En el repositorio oficial en GitHub podemos encontrar ficheros de instalación para diferentes sistemas operativos [https://github.com/rnwood/smtp4dev/releases](https://github.com/rnwood/smtp4dev/releases).
* pgAdmin: Podemos descargarla desde [https://www.pgadmin.org](https://www.pgadmin.org)

## FAQ y Resolución de Problemas
* Error "Forbidden" en SuiteCRM: SuiteCRM 8 requiere acceder a través de la carpeta pública. Asegúrate de usar la URL completa: [http://localhost:8080/public](http://localhost:8080/public).
* ¿Se borra mi trabajo si cierro Docker Desktop o apago el equipo? No. Los datos persisten en los volúmenes definidos en el `docker-compose.yml`, tanto los internos de Docker como los ligados a las carpetas locales de tu proyecto.
* ¿Cómo detengo los servicios? Ejecuta `docker compose stop` en la carpeta del proyecto (aunque no es estrictamente necesario).
* ¿Problemas con la virtualización? Si Docker no arranca, verifica en la BIOS que la "Virtualización" (VT-x o AMD-V) esté habilitada.
* ¿Cómo empiezo de cero con un servicio instalado eliminando todos los datos creados hasta la fecha?
    1. Páralo todo con `docker compose down` desde la carpeta del proyecto.
    2. Saca el listado de volúmenes con `docker volume ls`.
    3. Elimina los relacionados con ese servicio con `docker volume rm [VOLUMEN_A_ELIMINAR]`.
    4. Vuelve a lanzar los servicios con `docker compose up -d`.
* ¿Puedo eliminar un servicio por completo? (por ejemplo, porque hayamos modificado la configuración y necesitemos construirlo desde cero). Sí, con `docker rm -f [SERVICIO]` (Vuelve a lanzarlo con `docker compose up -d --build`).
* ¿Puedo ver los logs generados en los servicios? Sí, con `docker compose logs [SERVICIO]`.
* ¿Por qué tenemos dos servicios de PostgreSQL (para Odoo y para Bonita Runtime)? De esta forma, si tuviéramos que eliminar uno o diera cualquier problema no perderíamos los datos del otro.