# Cali Docker Lab

Este laboratorio consiste en que cada estudiante agregue su propio servicio al repositorio, lo construya como imagen Docker y lo publique en Docker Hub.

## 1. Clonar el repositorio

```bash
git clone https://github.com/juandasevilla/cali-docker-lab.git
cd cali-docker-lab
```

---

# 2. Crear una nueva rama

Cada estudiante debe trabajar en **su propia rama**.

```bash
git checkout -b feature-tu-nombre
```

Ejemplo:

```bash
git checkout -b feature/{tu nombre}-service
```

---

# 3. Crear tu carpeta de servicio

Dentro del proyecto crea una carpeta con tu nombre.

Ejemplo:

```
cali-docker-lab/
 ├─ juan-sevilla/
 │   ├─ app.py
 │   ├─ requirements.txt
 │   └─ Dockerfile
 ├─ alex-imbacuan/
 │   ├─ app.py
 │   ├─ requirements.txt
 │   └─ Dockerfil
```

---

# 4. Crear el archivo `app.py`

Ejemplo usando Flask:

```python
from flask import Flask
import os

app = Flask(__name__)

student = os.getenv("STUDENT_NAME", "Juan Sevilla")
hood = os.getenv("BARRIO", "Cristobal Colon")

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

---

# 5. Crear `requirements.txt`

```
﻿blinker==1.9.0
click==8.3.1
colorama==0.4.6
Flask==3.1.3
itsdangerous==2.2.0
Jinja2==3.1.6
MarkupSafe==3.0.3
Werkzeug==3.1.6
```

---

# 6. Crear el `Dockerfile`

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

CMD ["python","/app/app.py"]
```

---

# 7. Construir la imagen

Desde la carpeta del proyecto:

```bash
docker build -t TU_USUARIO_DOCKERHUB/cali-service:v1 .
```

Ejemplo:

```bash
docker build -t sevilla85/cali-service:v1 .
```

---

# 8. Subir cambios al repositorio

Agregar los archivos:

```bash
git add .
git commit -m "Add service for <tu nombre>"
git push origin feature/{tu nombre}-service
```

Luego crear un **Pull Request** en GitHub hacia la rama `main` solicitar un aprobador

---

# 9. Subir la imagen a Docker Hub

Iniciar sesión en Docker:

```bash
docker login
```

Subir la imagen:

```bash
docker push TU_USUARIO_DOCKERHUB/cali-service:v1
```

---

# 10. Ejecutar el contenedor

Ejemplo de ejecución:

```bash
docker run -d \
--name svc-tu-nombre \
--network san-antonio \
-e STUDENT_NAME="Tu Nombre" \
-e BARRIO="Tu Barrio" \
-v biblioteca-del-pueblo:/var/log/app \
-p 0:8080 TU_USUARIO_DOCKERHUB/cali-service:v1
```

---

# Resultado esperado

Cada estudiante debe:

* Tener su **carpeta dentro del repositorio**
* Hacer **Pull Request**
* Publicar su **imagen en Docker Hub**
* Ejecutar su **contenedor en la red del laboratorio**
