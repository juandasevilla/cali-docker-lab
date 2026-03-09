# 🔧 Solución de Problemas Comunes

## ❌ Error: "network not found"

**Problema:** Al ejecutar `docker run --network san-antonio`, aparece error de red no encontrada.

**Solución:** El líder debe crear las redes primero:
```bash
docker network create san-antonio
docker network create granada
docker network create ciudad-jardin
docker network create san-fernando
```

---

## ❌ Error: "volume not found"

**Problema:** El volumen `biblioteca-del-pueblo` no existe.

**Solución:** El líder debe crear el volumen:
```bash
docker volume create biblioteca-del-pueblo
```

---

## ❌ Error: "port is already allocated"

**Problema:** El puerto que intentas usar ya está en uso.

**Solución:** 
1. Verifica qué puertos están en uso:
   ```bash
   docker ps
   ```
2. Usa un puerto diferente o detén el contenedor que lo está usando:
   ```bash
   docker stop <nombre-contenedor>
   ```

---

## ❌ Error: "denied: requested access to the resource is denied"

**Problema:** No puedes hacer push a Docker Hub.

**Solución:**
1. Asegúrate de estar logueado:
   ```bash
   docker login
   ```
2. Verifica que el nombre de la imagen incluya tu usuario de Docker Hub:
   ```bash
   docker tag mi-imagen:v1 TU-USUARIO/mi-imagen:v1
   docker push TU-USUARIO/mi-imagen:v1
   ```

---

## ❌ Error: "COPY failed: file not found"

**Problema:** El Dockerfile no encuentra los archivos a copiar.

**Solución:**
1. Ejecuta `docker build` desde la carpeta donde está el Dockerfile
2. Verifica que los archivos (`app.py`, `requirements.txt`, etc.) estén en la misma carpeta

---

## ❌ El contenedor se detiene inmediatamente

**Problema:** El contenedor inicia y se detiene al instante.

**Diagnóstico:**
```bash
docker logs <nombre-contenedor>
```

**Causas comunes:**
- Error de sintaxis en el código
- Dependencias faltantes
- Puerto incorrecto en el código

---

## ❌ No puedo conectarme a otro contenedor en la misma red

**Problema:** `curl http://svc-otro:8080` no funciona.

**Verificaciones:**
1. Ambos contenedores están en la misma red:
   ```bash
   docker network inspect san-antonio
   ```
2. El contenedor destino está corriendo:
   ```bash
   docker ps
   ```
3. Usa el nombre correcto del contenedor (el de `--name`)

---

## ❌ No se escriben logs en La Biblioteca del Pueblo

**Problema:** El archivo `visitas.log` no se actualiza.

**Verificaciones:**
1. El volumen está montado correctamente:
   ```bash
   docker inspect <contenedor> | grep -A 10 "Mounts"
   ```
2. La ruta en el código es `/var/log/app/visitas.log`
3. El usuario tiene permisos de escritura (ver Dockerfile)

**Prueba manual:**
```bash
docker exec -it <contenedor> sh -c "echo 'test' >> /var/log/app/visitas.log"
```

---

## ❌ Error: "pip: not found" o "npm: not found"

**Problema:** Errores al construir la imagen.

**Solución:** Verifica que estás usando la imagen base correcta:
- Para Python: `FROM python:3.12-alpine`
- Para Node.js: `FROM node:20-alpine`

---

## ❌ HEALTHCHECK falla constantemente

**Problema:** El contenedor aparece como "unhealthy".

**Diagnóstico:**
```bash
docker inspect --format='{{json .State.Health}}' <contenedor>
```

**Causas comunes:**
- El endpoint `/health` no existe
- El servicio tarda en iniciar (aumenta `--start-period`)
- `wget` no está instalado en la imagen

---

## 📞 ¿Aún tienes problemas?

1. Copia el error completo
2. Ejecuta estos comandos y comparte la salida:
   ```bash
   docker ps -a
   docker logs <tu-contenedor>
   docker network ls
   docker volume ls
   ```
3. Contacta al líder del equipo
