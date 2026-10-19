# Portainer Service

Despliegue sencillo de **[Portainer Community Edition](https://www.portainer.io/)** mediante Docker Compose, para administrar el daemon de Docker de este host a través de una interfaz web.

## Requisitos

- Docker (20.10 o superior)
- Docker Compose v2 (`docker compose ...`) o docker-compose clásico
- Puertos `9000` y `9443` libres en el host

## Estructura del proyecto

```
.
├── docker-compose.yml   # Definición del servicio Portainer
└── README.md
```

## Variables y recursos

| Recurso        | Valor                         | Descripción                                  |
| -------------- | ----------------------------- | -------------------------------------------- |
| Imagen         | `portainer/portainer-ce:latest` | Portainer Community Edition               |
| Contenedor     | `portainer`                   | Nombre fijo del contenedor                   |
| Puerto HTTPS   | `9443` → `9443`               | Interfaz web sobre TLS (puerto por defecto)   |
| Puerto HTTP    | `9000` → `9000`               | Interfaz web sin TLS (legado/redes internas)  |
| Socket Docker  | `/var/run/docker.sock`        | Otorga acceso al daemon de Docker del host    |
| Volumen        | `portainer_data`              | Persistencia de datos (`/data`)               |

> **Nota de seguridad:** al montar el socket de Docker, el contenedor tiene control total sobre el daemon. No expongas los puertos a internet sin autenticación adicional (reverse proxy + TLS + red firewall).

## Uso

### Levantar el servicio

```bash
docker compose up -d
```

### Verificar el estado

```bash
docker compose ps
docker compose logs -f portainer
```

### Detener el servicio

```bash
docker compose down
```

### Detener y eliminar también los datos

```bash
docker compose down -v
```

### Actualizar a la última versión de la imagen

```bash
docker compose pull
docker compose up -d
```

## Primer acceso

1. Abre en el navegador: `https://localhost:9443` (o `http://localhost:9000` si usas HTTP).
2. En la primera ejecución se solicita crear el usuario administrador.
3. Selecciona el entorno **local** para administrar los contenedores, imágenes, volúmenes y redes de este host.

## Solución de problemas

| Problema | Causa probable | Solución |
| -------- | -------------- | -------- |
| `port is already allocated` | Puerto 9000/9443 en uso | Cambia el lado izquierdo del mapeo en `docker-compose.yml` |
| Página vacía o error de conexión | Contenedor caído | `docker compose logs portainer` y `docker compose up -d` |
| No aparecen contenedores | Socket de Docker no montado | Verifica la línea `volumes` del servicio `portainer` |
| Tras un `pull` sigue igual | Imagen cacheada | `docker compose pull && docker compose up -d` |

## Notas

- `restart: unless-stopped` mantiene el servicio en ejecución tras reinicios del host.
- Los datos (usuarios, entornos, inventarios) se guardan en el volumen `portainer_data`; no se pierden al reconstruir el contenedor.
