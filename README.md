# Registro de Miembros de Mesa

Aplicación Flask para registrar miembros de mesa electoral (DNI, región, provincia, distrito y dirección del local de votación). Los datos se guardan en un archivo Excel (`data/miembros_mesa.xlsx`) mediante `openpyxl`.

## Funcionalidades

- Formulario de registro con validación de DNI (8 dígitos) y campos obligatorios.
- Prevención de DNI duplicados.
- Listado de miembros registrados (`/miembros`).
- Descarga del archivo Excel (`/descargar`).
- Endpoint de salud (`/healthz`) para checks de contenedor.

## Estructura del proyecto

```
miembro-mesa/
├── app.py
├── requirements.txt
├── templates/
│   ├── base.html
│   ├── index.html
│   └── miembros.html
├── static/
│   └── style.css
├── data/                 # se crea el .xlsx en tiempo de ejecución
├── Dockerfile             # imagen básica (python:3.12)
├── Dockerfile.optimizado  # imagen Alpine, más liviana
├── Dockerfile.multistage  # build multistage sobre Alpine
└── README.md
```

## Instalación y ejecución local

### Requisitos

- Python 3.10+
- pip

### Pasos

```bash
# 1. Crear entorno virtual
python -m venv venv
# Windows
venv\Scripts\activate
# Linux/Mac
source venv/bin/activate

# 2. Instalar dependencias
pip install -r requirements.txt

# 3. Ejecutar la aplicación
python app.py
```

La aplicación quedará disponible en `http://localhost:5000`.

### Variables de entorno opcionales

| Variable      | Descripción                                   | Valor por defecto |
|---------------|------------------------------------------------|--------------------|
| `PORT`        | Puerto de escucha                              | `5000`             |
| `DATA_DIR`    | Carpeta donde se guarda el Excel               | `./data`           |
| `SECRET_KEY`  | Clave para mensajes flash de Flask             | `dev-secret-key`   |
| `FLASK_DEBUG` | `1` para activar modo debug                    | `0`                |

## Ejecución con Docker

### Imagen básica

```bash
docker build -t miembro-mesa:basico -f Dockerfile .
docker run -p 5000:5000 -v "$(pwd)/data:/app/data" miembro-mesa:basico
```

### Imagen optimizada (Alpine)

```bash
docker build -t miembro-mesa:alpine -f Dockerfile.optimizado .
docker run -p 5000:5000 -v "$(pwd)/data:/app/data" miembro-mesa:alpine
```

### Imagen multistage (Alpine + build separado)

```bash
docker build -t miembro-mesa:multistage -f Dockerfile.multistage .
docker run -p 5000:5000 -v "$(pwd)/data:/app/data" miembro-mesa:multistage
```

> El volumen `-v "$(pwd)/data:/app/data"` persiste el archivo Excel fuera del contenedor. En Windows PowerShell usa `-v "${PWD}/data:/app/data"`.

### Comparación de imágenes

| Dockerfile            | Base              | Servidor          | Uso recomendado                          |
|------------------------|-------------------|--------------------|-------------------------------------------|
| `Dockerfile`            | `python:3.12`     | Flask dev server   | Desarrollo / simplicidad                  |
| `Dockerfile.optimizado` | `python:3.12-alpine` | Gunicorn        | Imagen reducida, un solo stage            |
| `Dockerfile.multistage` | `python:3.12-alpine` (build + runtime) | Gunicorn | Imagen mínima, sin herramientas de compilación en runtime |

## Notas

- El archivo Excel se crea automáticamente en el primer registro si no existe.
- Los datos no se eliminan entre reinicios siempre que se use un volumen para `data/`.
- Para producción, reemplaza `SECRET_KEY` por un valor seguro mediante variable de entorno.
