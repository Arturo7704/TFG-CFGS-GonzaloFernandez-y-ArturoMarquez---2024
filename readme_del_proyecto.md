# FastDataAI 🚀

> **Consulta inteligente de bases de datos empresariales mediante lenguaje natural, control de acceso biométrico y arquitectura de servidor securizada.**

**FastDataAI** es una plataforma desarrollada como Trabajo Final de Grado (CFGS en Administración de Sistemas Informáticos en Red - Instituto Nebrija de Formación Profesional, curso 2023-2024). Su objetivo es agilizar la explotación de datos corporativos permitiendo a cualquier empleado realizar consultas complejas sobre bases de datos relacionales en español sin conocimientos previos de SQL, garantizando al mismo tiempo el control de acceso mediante reconocimiento facial y políticas estrictas de seguridad por roles.

---

## ✨ Características Principales

* **Autenticación Biométrica por Roles (Reconocimiento Facial):**
  * Captura y comparación de vectores faciales en tiempo real mediante cámara web.
  * Diferenciación automática entre **Administradores**, **Trabajadores Normales** y **Usuarios Desconocidos**.
  * **Flujo de alta para desconocidos:** Si el sistema no reconoce un rostro, permite capturar una fotografía al pulsar `Enter` y enviarla automáticamente por correo electrónico (SMTP/SSL) al administrador para su validación y asignación de rol.
  * **Auditoría de accesos:** Registro automático en base de datos MySQL con identificador, nombre y marca temporal (*timestamp*) de cada inicio de sesión.
* **Chatbot SQL con Inteligencia Artificial (Text-to-SQL):**
  * Traducción de preguntas en lenguaje natural (español) a consultas SQL ejecutables mediante **LangChain** (`SQLDatabaseChain`) y **OpenAI GPT-3.5-turbo**.
  * **Restricción de privacidad por rol:** Los trabajadores estándar tienen bloqueado el acceso a tablas sensibles (como la tabla `usuarios`), mientras que los administradores cuentan con acceso completo.
* **Panel Web CRUD para Administradores:**
  * Interfaz web en **Flask** para crear, listar (con paginación y filtrado por producto, cliente, país, fecha o factura), editar y eliminar registros de ventas.
  * Reindexación automática de identificadores tras la eliminación de registros mediante expresiones de tabla común (`WITH RECURSIVE`).
* **Infraestructura Securizada y Administración de Sistemas:**
  * Despliegue automatizado como servicios del sistema mediante **systemd** en servidor **Ubuntu Linux**.
  * Endurecimiento (*hardening*) de red mediante firewall **UFW**, habilitando únicamente los puertos estrictamente necesarios.
  * Trazabilidad y monitorización de peticiones HTTP e IPs mediante el módulo `logging` de Python.
  * **Sistema de Copias de Seguridad en Bash:** Herramienta interactiva por consola para realizar backups totales, incrementales y diferenciales (`rsync`), catalogarlos en MySQL, comprimirlos (`tar.gz`), restaurarlos y programarlos vía `cron`.

---

## 🏗️ Arquitectura del Sistema

El sistema sigue una arquitectura cliente-servidor híbrida que separa la captura biométrica local del procesamiento de datos y el motor de IA en el servidor:

```text
┌─────────────────────────┐       1. Petición Web       ┌───────────────────────────────┐
│   Equipo Cliente        │ ──────────────────────────► │   Servidor Ubuntu (VM)        │
│   (Windows / Webcam)    │                             │                               │
│                         │ ◄────────────────────────── │   • Portal Inicio (Flask)     │
│   • Agente escucha HTTP │   2. Orden POST /execute    │   • Chatbot IA (LangChain)    │
│   • OpenCV + dlib       │      (vía curl.sh)          │   • Panel CRUD Admin (Flask)  │
└───────────┬─────────────┘                             └───────┬───────────────┬───────┘
            │                                                   │               │
            │ 3. Registro de acceso                             │ 4. SQL        │ 5. NLP / SQL
            ▼                                                   ▼               ▼
┌─────────────────────────┐                             ┌───────────────┐ ┌─────────────┐
│   MySQL (Auditoría)     │                             │ SQLite (Datos)│ │ OpenAI API  │
│   • Tabla: personas     │                             │ • ventas      │ │ (GPT-3.5)   │
│   • Catálogo: buckups   │                             │ • productos   │ └─────────────┘
└─────────────────────────┘                             │ • clientes    │
                                                        │ • paises      │
                                                        │ • usuarios    │
                                                        └───────────────┘
```

---

## 🛠️ Stack Tecnológico

* **Lenguajes:** Python 3.12, Bash, SQL, HTML/CSS
* **Frameworks e IA:** Flask, LangChain (`langchain_openai`, `langchain_experimental`), OpenAI API (`gpt-3.5-turbo`)
* **Visión Artificial:** OpenCV (`cv2`), `face_recognition` (`dlib`), Tkinter
* **Bases de Datos:** SQLite3 (datos corporativos), MySQL / XAMPP / MySQL Workbench (auditoría de accesos y catálogo de backups)
* **Sistemas y Seguridad:** Ubuntu Linux, `systemd`, `ufw` (Uncomplicated Firewall), `rsync`, `cron`, `python-dotenv` / variables de entorno

---

## 📂 Estructura Recomendada del Repositorio

```text
FastDataAI/
├── cliente_reconocimiento/
│   ├── app2.py                  # Servidor HTTP local a la escucha de peticiones de captura
│   ├── prueba2.py               # Script principal de reconocimiento facial, roles y alertas SMTP
│   ├── reconES.py               # Prototipo con interfaz gráfica Tkinter para gestión de rostros
│   ├── administradores/         # Imágenes de referencia para usuarios administradores (.jpg)
│   └── usuarios_normales/       # Imágenes de referencia para usuarios estándar (.jpg)
├── servidor_web/
│   ├── app.py                   # Portal de bienvenida y disparador del reconocimiento (curl.sh)
│   ├── usuarios.py              # App Flask del chatbot con restricción de tablas sensibles
│   ├── administradores.py       # App Flask del chatbot sin restricciones + gestión CRUD
│   ├── curl.sh                  # Script de comunicación servidor -> cliente
│   ├── tfg.db                   # Base de datos SQLite empresarial (ventas, productos, clientes...)
│   └── templates/               # Plantillas HTML (consultas.html, admin_productos.html, editar_factura.html)
├── scripts_sistemas/
│   ├── menu_backups.sh          # Gestor interactivo de copias de seguridad (total/inc/dif)
│   ├── cron.sh                  # Automatización de tareas programadas
│   └── systemd/                 # Archivos .service para arranque automático en Ubuntu
└── docs/
    └── Memoria_TFG_FastDataAI.pdf
```

---

## 🚀 Instalación y Puesta en Marcha

### 1. Requisitos Previos
* **Servidor:** Máquina Linux (Ubuntu) con Python 3.10+, `ufw` y acceso a red local.
* **Cliente:** Equipo con cámara web, Python 3.12, CMake/dlib compilado y servidor MySQL (ej. XAMPP) activo para el registro de accesos.
* **Credenciales:** Clave de API de OpenAI y cuenta de correo con contraseña de aplicación para notificaciones SMTP.

### 2. Instalación de Dependencias en Python

```bash
pip install flask langchain langchain-openai langchain-experimental openai opencv-python face-recognition mysql-connector-python
```

### 3. Configuración de Variables de Entorno y Bases de Datos

1. **Variables de entorno:** Configura las credenciales sensibles fuera del código fuente:
   * En el cliente (`.env`):
     ```env
     PASSWORD="tu_contraseña_de_aplicacion_smtp"
     ```
   * En el servidor (`a_env_vars.py` o variable de entorno del sistema):
     ```python
     OPENAI_API_KEY = "sk-..."
     ```

2. **Base de datos MySQL (Auditoría de accesos):**
   ```sql
   CREATE DATABASE reconocimiento_facial;
   USE reconocimiento_facial;

   CREATE TABLE personas (
       id INT AUTO_INCREMENT PRIMARY KEY,
       nombre VARCHAR(255),
       hora_detectada TIMESTAMP DEFAULT CURRENT_TIMESTAMP
   );
   ```

3. **Base de datos SQLite (`tfg.db`):** Asegúrate de contar con el archivo `tfg.db` con las tablas relacionales (`ventas`, `productos`, `clientes`, `paises`, `usuarios`) en el directorio de trabajo del servidor.

### 4. Despliegue como Servicio en el Servidor (Ubuntu)

Crea los archivos de unidad en `/etc/systemd/system/` (por ejemplo, `fastdata-app.service`):

```ini
[Unit]
Description=Flask Application - FastDataAI
After=network.target

[Service]
User=tu_usuario
Group=www-data
WorkingDirectory=/ruta/al/proyecto/servidor_web
ExecStart=/usr/bin/python3 /ruta/al/proyecto/servidor_web/app.py

[Install]
WantedBy=multi-user.target
```

Activa los servicios y configura el firewall:

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now fastdata-app.service
sudo ufw allow 80/tcp
sudo ufw allow 5000:5004/tcp
sudo ufw enable
```

### 5. Ejecución en el Equipo Cliente

1. Añade las fotografías de los usuarios autorizados (nombradas como `NombreUsuario.jpg`) dentro de las carpetas `administradores/` o `usuarios_normales/`.
2. Inicia el agente de escucha en el equipo cliente:
   ```bash
   python app2.py
   ```
3. Accede desde el navegador a `http://<IP_SERVIDOR>:5000` y pulsa **"Acceso a la herramienta"**.

---

## 💡 Ejemplos de Uso del Chatbot

* **Consulta simple:**
  > *"Sácame todas las ventas que tengan la factura 8"*
* **Consulta relacional (JOIN automático):**
  > *"Sácame todas las ventas que tengan la factura 8 y que tengan el producto la consola PlayStation 5"*
* **Búsqueda semántica:**
  > *"Dime los productos que tengan que ver con móviles"*
* **Control de acceso (Usuario Normal):**
  > *"Dime los usuarios de la empresa"* → **Respuesta:** `Acceso restringido a la información de usuarios.`

---

## 🔮 Mejoras Futuras

* **Soporte RAG para Documentos:** Carga de archivos PDF y Word (`PyPDF2`, `python-docx`) para realizar consultas sobre documentación no estructurada.
* **Despliegue Cloud y Alta Disponibilidad:** Migración a AWS o Azure con balanceadores de carga y bases de datos gestionadas (Amazon RDS / Azure SQL).
* **IA Local (*On-Premise*):** Sustitución de la API externa por modelos open-source alojados internamente para garantizar la soberanía total de los datos corporativos.
* **Seguridad Biométrica Avanzada:** Cifrado de imágenes almacenadas en reposo, autenticación de doble factor (2FA) por correo y empaquetado del cliente mediante `PyInstaller`.