# Guía de desarrollo — Facturador

## Estructura del proyecto

```
Facturador/
├── docker-compose.yml       # Orquesta ambos servicios
├── Facturador-back/         # API .NET 10 (ASP.NET Core)
└── facturador-web/          # Frontend Nuxt 3 + Bun
```

## Requisitos previos

- [Docker Desktop](https://www.docker.com/products/docker-desktop/) instalado y corriendo
- No se necesita .NET ni Node/Bun instalados localmente

---

## Levantar el proyecto

### Stack completo (front + back)

```bash
docker compose up
```

Levanta ambos servicios con hot reload activo:
- **API** → http://localhost:8080
- **Frontend** → http://localhost:3000

> La primera vez tarda más porque Docker descarga las imágenes base y restaura dependencias. Las siguientes veces es significativamente más rápido.

---

### Solo el frontend

```bash
docker compose up web
```

Útil cuando estás trabajando solo en el front y corrés el back desde el devcontainer de VS Code.

---

### Solo la API

```bash
docker compose up api
```

Útil para probar o debuggear la API de forma aislada.

---

## Hot reload

| Servicio | Comportamiento |
|----------|---------------|
| Frontend (Nuxt) | HMR completo — los cambios se reflejan en el browser sin recargar |
| API (.NET) | `dotnet watch` — cambios simples aplican sin restart, cambios estructurales reinician el proceso automáticamente en segundos |

Los cambios que hacés en tu editor se sincronizan al contenedor en tiempo real a través de volúmenes de Docker.

---

## Trabajar con Intellisense en el back (devcontainer)

Podés abrir `Facturador-back/` en VS Code con el devcontainer para tener C# Dev Kit e Intellisense completo, **sin conflicto** con Docker Compose.

**Escenario recomendado:**
1. Levantá el stack con `docker compose up`
2. Abrí el devcontainer en VS Code solo para editar código
3. Los cambios que hacés en el devcontainer son detectados por `dotnet watch` en el contenedor del compose automáticamente

**Consideración:** Si Docker Compose ya está corriendo la API en el puerto `8080`, no ejecutes la task "Iniciar API" dentro del devcontainer — habría conflicto de puertos. La task fue desactivada para que no corra automáticamente al abrir el proyecto, pero si la corrés manualmente fallará si el puerto ya está ocupado.

---

## Comandos útiles

### Ver logs en tiempo real

```bash
docker compose logs -f
```

Muestra los logs de todos los servicios. Útil para ver errores de arranque o requests entrantes.

```bash
docker compose logs -f api
docker compose logs -f web
```

Filtra los logs por servicio.

---

### Reconstruir las imágenes

```bash
docker compose build
```

Necesario cuando cambiás el `Dockerfile` o el `docker-compose.yml`. Los cambios en el código fuente no requieren rebuild gracias a los volúmenes.

```bash
docker compose up --build
```

Reconstruye y levanta en un solo comando.

---

### Detener los servicios

```bash
docker compose down
```

Detiene y elimina los contenedores. Los volúmenes y el código fuente no se tocan.

```bash
docker compose stop
```

Solo detiene los contenedores sin eliminarlos. Podés reanudarlos con `docker compose start`.

```bash
docker stop $(docker ps -q)
```

Detiene **todos los contenedores corriendo** en Docker, sin importar el proyecto. Útil cuando tenés múltiples proyectos activos y querés detener todo de una vez.

```bash
docker stop $(docker ps -aq)
```

Igual que el anterior pero incluye también los contenedores ya detenidos. El resultado es el mismo, pero puede mostrar advertencias para los que ya estaban parados. En la mayoría de los casos preferí la versión sin `-a`.

---

### Ver el estado de los contenedores

```bash
docker compose ps
```

Muestra qué servicios están corriendo, su estado y los puertos expuestos.

---

### Entrar al contenedor de la API

```bash
docker compose exec api bash
```

Abre una terminal dentro del contenedor del back. Útil para correr comandos de .NET manualmente, inspeccionar archivos, etc.

```bash
docker compose exec web bash
```

Lo mismo para el contenedor del front.

---

### Limpiar todo (reset completo)

```bash
docker compose down --volumes --rmi local
```

Elimina los contenedores, volúmenes anónimos e imágenes construidas localmente. Útil si algo quedó en un estado inconsistente y querés empezar de cero.

> Después de esto, el próximo `docker compose up` va a tardar como la primera vez.

---

### Ver todos los contenedores del sistema

```bash
docker ps
```

Lista los contenedores actualmente corriendo con su ID, nombre, imagen, puertos y estado.

```bash
docker ps -a
```

Igual pero incluye los contenedores detenidos. Útil para ver si un contenedor crasheó.

---

### Ver imágenes descargadas

```bash
docker images
```

Lista todas las imágenes que Docker tiene guardadas localmente con su tamaño. Útil para saber cuánto espacio están ocupando.

---

### Eliminar contenedores detenidos

```bash
docker rm $(docker ps -aq)
```

Elimina todos los contenedores que están detenidos. No afecta los que están corriendo ni el código fuente.

---

### Eliminar imágenes sin usar

```bash
docker image prune
```

Elimina imágenes "colgadas" (sin tag ni contenedor asociado). Libera espacio en disco.

```bash
docker image prune -a
```

Elimina **todas** las imágenes que no están siendo usadas por ningún contenedor activo. La próxima vez que hagas `docker compose up` las va a volver a descargar.

---

### Limpieza general del sistema

```bash
docker system prune
```

Elimina de una vez: contenedores detenidos, redes sin usar e imágenes colgadas. Pide confirmación antes de ejecutar.

```bash
docker system prune -a --volumes
```

Versión agresiva: elimina todo lo anterior más imágenes no usadas y volúmenes. Útil cuando Docker está ocupando demasiado espacio en disco.

> Después de esto necesitás hacer `docker compose up --build` para reconstruir todo desde cero.

---

### Ver cuánto espacio usa Docker

```bash
docker system df
```

Muestra un resumen del espacio ocupado por imágenes, contenedores y volúmenes. Útil antes de decidir si hacer una limpieza.

---

### Ver los recursos que consume cada contenedor

```bash
docker stats
```

Muestra en tiempo real el uso de CPU, memoria y red de todos los contenedores corriendo. Útil para detectar si algún servicio está consumiendo demasiados recursos.

```bash
docker stats facturador-api facturador-web
```

Filtra solo los contenedores de este proyecto.

---

### Inspeccionar un contenedor

```bash
docker inspect facturador-api
```

Muestra toda la configuración del contenedor en formato JSON: variables de entorno, volúmenes montados, red, puertos, etc. Útil para debuggear problemas de configuración.

---

### Ver los logs de un contenedor específico (sin compose)

```bash
docker logs facturador-api
docker logs facturador-web
```

Muestra los logs del contenedor. Útil cuando el contenedor ya está detenido y `docker compose logs` no funciona.

```bash
docker logs -f --tail 50 facturador-api
```

Sigue los logs en tiempo real mostrando solo las últimas 50 líneas.

---

### Reiniciar un contenedor sin detener el resto

```bash
docker restart facturador-api
```

Reinicia solo ese contenedor. Equivalente a `docker compose restart api` pero usando el nombre del contenedor directamente.

---

## Comunicación entre servicios

Dentro de la red de Docker Compose, los servicios se comunican por nombre:

| Desde | Hacia la API |
|-------|-------------|
| Frontend (server-side) | `http://api:8080` |
| Browser del usuario | `http://localhost:8080` |

Configurá la URL base de la API en `facturador-web/.env`:

```env
NUXT_PUBLIC_API_URL=http://localhost:8080
```

> Usá `localhost` para llamadas desde el browser, y `http://api:8080` para llamadas server-side de Nuxt (dentro de `server/`).

---

## Solución de problemas comunes

### El puerto 8080 o 3000 ya está en uso

Significa que ya hay un proceso usando ese puerto (por ejemplo el devcontainer corriendo la API). Detené el proceso que lo ocupa o cambiá el puerto en `docker-compose.yml`.

```bash
# Ver qué proceso usa el puerto 8080
sudo lsof -i :8080
```

### Los cambios en el código no se reflejan

Verificá que los contenedores estén corriendo:
```bash
docker compose ps
```

Si están corriendo pero los cambios no aparecen, reiniciá el servicio:
```bash
docker compose restart api
docker compose restart web
```

### Se generó un archivo `*.sln` en la raíz o en `Facturador-back/`

Es generado automáticamente por `dotnet` al arrancar. Está ignorado por git (`.gitignore`) así que no afecta al repositorio. Podés borrarlo sin problema.

### Error al arrancar por primera vez

Revisá los logs para ver el error específico:
```bash
docker compose logs api
docker compose logs web
```
