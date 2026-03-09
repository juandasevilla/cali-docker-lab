# 📖 Guía Completa para Estudiantes - Cali Container City Lab

## 👋 Bienvenido al Laboratorio

Esta guía te llevará paso a paso desde clonar el repositorio hasta tener tu servicio corriendo en Docker y publicado en Docker Hub/GitHub Packages.

---

## 📋 Requisitos Previos

Antes de comenzar, asegúrate de tener instalado:
- [ ] [Docker Desktop](https://www.docker.com/products/docker-desktop/)
- [ ] [Git](https://git-scm.com/downloads)
- [ ] Una cuenta en [Docker Hub](https://hub.docker.com/) o [GitHub](https://github.com/)
- [ ] Un editor de código (VS Code recomendado)

### Verificar instalación
```bash
docker --version
git --version
```

---

## 🚀 PASO 1: Clonar el Repositorio

```bash
# Clonar el repositorio del equipo
git clone <URL_DEL_REPOSITORIO>
cd <nombre-del-repositorio>

# Crear tu rama de trabajo
git checkout -b feature/tu-nombre

# Crear tu carpeta personal
mkdir -p students/tu-nombre
cd students/tu-nombre
```

---

## 🛠️ PASO 2: Crear tu Servicio

Tu servicio debe:
1. ✅ Escuchar en el puerto **8080**
2. ✅ Responder con un mensaje: `Hola, I am <TuNombre> and I live in <Barrio>`
3. ✅ Escribir cada visita en `/var/log/app/visitas.log`

### Elige tu tecnología:

---

### 🐍 Opción A: Python (Flask)

**Archivos a crear en tu carpeta:**

#### `app.py`
```python
from flask import Flask
import os

app = Flask(__name__)
student = os.getenv("STUDENT_NAME", "Anon")
hood = os.getenv("BARRIO", "Unknown")

@app.get("/")
def home():
    msg = f"Hola, I am {student} and I live in {hood}"
    with open("/var/log/app/visitas.log", "a") as f:
        f.write(msg + "\n")
    return msg

@app.get("/health")
def health():
    return {"ok": True}, 200

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=8080)
```

#### `requirements.txt`
```
flask==3.0.3
```

#### `Dockerfile`
```dockerfile
FROM python:3.12-alpine
WORKDIR /app
RUN addgroup -S app && adduser -S app -G app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY app.py .
RUN mkdir -p /var/log/app && chown -R app:app /var/log/app
USER app
EXPOSE 8080
HEALTHCHECK --interval=10s --timeout=3s --retries=5 \
  CMD wget -qO- http://localhost:8080/health >/dev/null || exit 1
CMD ["python", "/app/app.py"]
```

---

### 📦 Opción B: Node.js (Express)

**Archivos a crear en tu carpeta:**

#### `server.js`
```javascript
const express = require('express');
const fs = require('fs');
const app = express();

const student = process.env.STUDENT_NAME || "Anon";
const hood = process.env.BARRIO || "Unknown";

app.get('/', (req, res) => {
    const msg = `Hola, I am ${student} and I live in ${hood}`;
    fs.appendFileSync('/var/log/app/visitas.log', msg + '\n');
    res.type('text/plain').send(msg);
});

app.get('/health', (req, res) => res.json({ ok: true }));

app.listen(8080, () => console.log("Server running on 8080"));
```

#### `package.json`
```json
{
  "name": "cali-service",
  "private": true,
  "version": "1.0.0",
  "dependencies": {
    "express": "^4.19.2"
  }
}
```

#### `Dockerfile`
```dockerfile
FROM node:20-alpine
WORKDIR /app
RUN addgroup -S app && adduser -S app -G app
COPY package.json ./
RUN npm ci --omit=dev
COPY server.js ./
RUN mkdir -p /var/log/app && chown -R app:app /var/log/app
USER app
EXPOSE 8080
HEALTHCHECK --interval=10s --timeout=3s --retries=5 \
  CMD wget -qO- http://localhost:8080/health >/dev/null || exit 1
CMD ["node", "server.js"]
```

---

### 🌐 Opción C: Nginx (HTML Estático)

**Archivos a crear en tu carpeta:**

#### `index.html`
```html
<!DOCTYPE html>
<html>
<head>
    <title>Cali Container City</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            color: white;
            margin: 0;
        }
        .container {
            text-align: center;
            padding: 40px;
            background: rgba(0,0,0,0.3);
            border-radius: 20px;
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>🏠 Hola, I am TU_NOMBRE and I live in TU_BARRIO</h1>
        <p>Bienvenido a Cali Container City!</p>
    </div>
</body>
</html>
```

#### `Dockerfile`
```dockerfile
FROM nginx:alpine
COPY index.html /usr/share/nginx/html/index.html
EXPOSE 80
```

> **Nota:** La opción Nginx es la más simple pero no escribe logs automáticamente. Usa Python o Node.js si quieres el logging completo.

---

## 🏗️ PASO 3: Construir tu Imagen Docker

Desde tu carpeta (`students/tu-nombre/`):

```bash
# Construir la imagen
docker build -t tu-usuario-dockerhub/cali-service:v1 .

# Verificar que se creó
docker images | grep cali-service
```

### Probar localmente primero
```bash
docker run -d --name test-local -p 8080:8080 \
  -e STUDENT_NAME="TuNombre" \
  -e BARRIO="SanAntonio" \
  tu-usuario-dockerhub/cali-service:v1

# Probar
curl http://localhost:8080

# Ver logs
docker logs test-local

# Limpiar
docker stop test-local && docker rm test-local
```

---

## 📤 PASO 4: Publicar tu Imagen

### Opción A: Docker Hub

```bash
# Iniciar sesión en Docker Hub
docker login

# Publicar la imagen
docker push tu-usuario-dockerhub/cali-service:v1
```

### Opción B: GitHub Packages

```bash
# Crear un Personal Access Token (PAT) en GitHub:
# Settings > Developer settings > Personal access tokens > Generate new token
# Permisos necesarios: write:packages, read:packages, delete:packages

# Iniciar sesión
echo "TU_TOKEN" | docker login ghcr.io -u tu-usuario-github --password-stdin

# Etiquetar la imagen para GitHub
docker tag tu-usuario-dockerhub/cali-service:v1 ghcr.io/tu-usuario-github/cali-service:v1

# Publicar
docker push ghcr.io/tu-usuario-github/cali-service:v1
```

---

## 🏃 PASO 5: Ejecutar tu Contenedor en el Barrio

Pregunta al líder cuál es tu **barrio asignado** y **puerto**, luego ejecuta:

```bash
docker run -d --name svc-tunombre \
  --network tu-barrio \
  -e STUDENT_NAME="TuNombre" \
  -e BARRIO="TuBarrio" \
  -v biblioteca-del-pueblo:/var/log/app \
  -p PUERTO_ASIGNADO:8080 \
  tu-usuario-dockerhub/cali-service:v1
```

### Ejemplo completo:
```bash
docker run -d --name svc-ana \
  --network san-antonio \
  -e STUDENT_NAME="Ana" \
  -e BARRIO="San Antonio" \
  -v biblioteca-del-pueblo:/var/log/app \
  -p 8081:8080 \
  ana-docker/cali-service:v1
```

---

## ✅ PASO 6: Verificar que Funciona

### Probar desde el navegador
```
http://localhost:PUERTO_ASIGNADO
```

### Probar desde terminal
```bash
curl http://localhost:PUERTO_ASIGNADO
```

### Probar conectividad entre contenedores del mismo barrio
```bash
docker run --rm --network tu-barrio curlimages/curl -s http://svc-tunombre:8080/
```

### Ver si escribiste en La Biblioteca del Pueblo
```bash
docker run --rm -v biblioteca-del-pueblo:/data alpine sh -c "grep TuNombre /data/visitas.log"
```

---

## 📝 PASO 7: Subir tus Cambios a Git

```bash
# Desde la raíz del repositorio
cd ../..

# Agregar tus archivos
git add students/tu-nombre/

# Hacer commit
git commit -m "feat: agregar servicio de tu-nombre con Python/Node/Nginx"

# Subir tu rama
git push origin feature/tu-nombre
```

### Crear Pull Request
1. Ve a GitHub/GitLab
2. Click en "Compare & Pull Request"
3. Describe brevemente qué tecnología usaste
4. Asigna al líder como reviewer
5. Espera la aprobación

---

## 📋 Checklist Final

Antes de considerar tu trabajo terminado, verifica:

- [ ] Mi carpeta `students/mi-nombre/` existe con todos los archivos
- [ ] Mi Dockerfile construye sin errores
- [ ] Mi imagen está publicada en Docker Hub o GitHub Packages
- [ ] Mi contenedor está corriendo en el barrio asignado
- [ ] Puedo acceder a mi servicio desde `http://localhost:PUERTO`
- [ ] Mi servicio escribe en `/var/log/app/visitas.log`
- [ ] Mis cambios están en un Pull Request
- [ ] Escribí una breve explicación de mi diseño

---

## ❓ Preguntas Frecuentes

### ¿Cómo sé mi puerto asignado?
Pregunta al líder. Cada estudiante tiene un puerto único (8081, 8082, 8083, etc.)

### ¿Cómo sé mi barrio?
El líder asigna los barrios. Los disponibles son: `san-antonio`, `granada`, `ciudad-jardin`, `san-fernando`

### Mi contenedor no inicia, ¿qué hago?
```bash
# Ver los logs del contenedor
docker logs svc-tunombre

# Ver el estado
docker ps -a
```

### ¿Cómo rebuildo mi imagen después de cambios?
```bash
docker build -t tu-usuario-dockerhub/cali-service:v2 .
docker push tu-usuario-dockerhub/cali-service:v2
```

### ¿Cómo reinicio mi contenedor?
```bash
docker stop svc-tunombre
docker rm svc-tunombre
# Luego vuelve a ejecutar el docker run
```

---

## 🆘 ¿Necesitas Ayuda?

1. Revisa el archivo [TROUBLESHOOTING.md](docs/TROUBLESHOOTING.md)
2. Pregunta en el grupo del equipo
3. Contacta al líder

---

**¡Éxitos con tu servicio! 🚀🏙️**
