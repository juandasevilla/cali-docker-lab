# 🏙️ Cali Container City Lab

## 📋 Información del Proyecto
Este es el repositorio del equipo para el laboratorio de Docker "Cali Container City".

**Líder del equipo:** [Tu nombre aquí]  
**Repositorio:** [URL del repositorio aquí]

---

## 🎯 Objetivo del Laboratorio
Cada estudiante diseñará y construirá su propia imagen Docker, la publicará en Docker Hub o GitHub Packages, y ejecutará un contenedor desde esa imagen. Los contenedores vivirán en "barrios" (redes Docker) y registrarán su actividad en el volumen compartido "La Biblioteca del Pueblo".

---

## 📁 Estructura del Repositorio
```
/
├── README.md                    # Este archivo
├── GUIA_ESTUDIANTES.md          # Guía paso a paso para estudiantes
├── .gitignore
├── students/
│   ├── ejemplo-python/          # Ejemplo con Flask
│   │   ├── Dockerfile
│   │   ├── app.py
│   │   └── requirements.txt
│   ├── ejemplo-nodejs/          # Ejemplo con Express
│   │   ├── Dockerfile
│   │   ├── server.js
│   │   └── package.json
│   └── ejemplo-nginx/           # Ejemplo con Nginx
│       ├── Dockerfile
│       └── index.html
└── docs/
    └── TROUBLESHOOTING.md       # Solución de problemas comunes
```

---

## 🛠️ Configuración Inicial (SOLO LÍDER)

### Paso 1: Crear las redes Docker (barrios de Cali)
```bash
docker network create san-antonio
docker network create granada
docker network create ciudad-jardin
docker network create san-fernando
```

### Paso 2: Crear el volumen compartido (La Biblioteca del Pueblo)
```bash
docker volume create biblioteca-del-pueblo
```

### Paso 3: Verificar la configuración
```bash
# Ver redes creadas
docker network ls

# Ver volúmenes creados
docker volume ls
```

### Paso 4: Inicializar el repositorio Git
```bash
#Clonar el repositorio por http 
git clone https://github.com/juandasevilla/cali-docker-lab.git

# O clonar por ssh
git clone git@github.com:juandasevilla/cali-docker-lab.git
```

---

## 👥 Asignación de Barrios

| Estudiante     | Barrio (Red)   | Puerto Host | Tech Stack |
|----------------|----------------|-------------|------------|
| Pedro Parrales | el-troncal     | 8080        | Python     |
| Juan Rojas     | brisas-alamos  | 8081        | Python     |
| Juan Sevilla   | cristobal-colon| 8082        | Python     |
| Alex Silva     | calipso        | 8083        | Python     |
| Alex Imbacuan  | san-nicolas    | 8084        | Python     |

---

## 📊 Comandos para hacer el laboratorio

### Clonar los repositorios de los compañeros
Tener en cuenta cambiar el nombre-imagen y el puerto ir aumentando de 8080 a 8081 y asi, por cada uno
```bash
docker run -d --name nombre-imagen -p 8080:8080 --network nombre-red-creada -v biblioteca-del-pueblo:/var/log/app usuario-companero/cali-service:v1
```

### Intentar acceder a otra imagen que se encuentra en la misma red
```bash
docker exec -it nombre-imagen-1 sh -c "wget -qO- http://nombre-imagen-2:8080/"
```

### Ver logs de La Biblioteca del Pueblo
```bash
docker run --rm -v biblioteca-del-pueblo:/data alpine sh -c "cat /data/visitas.log"
```

### Ver últimas 20 visitas
```bash
docker run --rm -v biblioteca-del-pueblo:/data alpine sh -c "tail -n 20 /data/visitas.log"
```

### Limpiar todo (¡usar con precaución!)
```bash
# Detener todos los contenedores
docker stop $(docker ps -q)

# Eliminar todos los contenedores
docker rm $(docker ps -aq)

# Eliminar las redes
docker network rm san-antonio granada ciudad-jardin san-fernando

# Eliminar el volumen
docker volume rm biblioteca-del-pueblo
```

---

## 🔗 Enlaces Útiles
- [Guía para Estudiantes](GUIA_ESTUDIANTES.md)
- [Solución de Problemas](docs/TROUBLESHOOTING.md)
- [Docker Hub](https://hub.docker.com/)
- [GitHub Packages](https://github.com/features/packages)

---

## 📅 Flujo de Trabajo Git (Trunk-Based Development)

1. Cada estudiante crea una rama desde `main`:
   ```bash
   git checkout main
   git pull origin main
   git checkout -b feature/nombre-estudiante
   ```

2. Trabaja en su carpeta `/students/<nombre>/`

3. Hace commits pequeños y frecuentes:
   ```bash
   git add .
   git commit -m "feat: agregar servicio de <nombre>"
   ```

4. Abre un Pull Request hacia `main`

5. El líder revisa y aprueba el PR

6. Se hace merge a `main`

---

**¡Buena suerte equipo! 🚀**
