# Redis

Servidor Redis para cache, colas ligeras y almacenamiento clave/valor.

## Para que proyectos usarlo

Redis es ideal cuando necesitas velocidad y datos temporales o semipersistentes.

| Tipo de proyecto | Ejemplos |
| --- | --- |
| Cache | Cache de respuestas, consultas SQL, sesiones, tokens |
| Rate limiting | Limitar requests por usuario/IP/API key |
| Colas simples | Jobs con BullMQ, Celery, RQ, Sidekiq |
| Tiempo real | Pub/Sub, presencia, contadores, ranking |
| Sesiones | Sesiones compartidas entre varias instancias de una app |
| Feature flags o locks | Locks distribuidos, flags temporales, deduplicacion |

Usalo junto a una base de datos principal, no como reemplazo automatico de PostgreSQL/MySQL/MongoDB. Aunque aqui tiene AOF habilitado, Redis normalmente se usa para datos rapidos, cache o coordinacion.

## Levantar

```bash
docker compose up -d
docker compose ps
docker compose logs -f redis
```

## Datos importantes

| Dato | Valor |
| --- | --- |
| Servicio Compose | `redis` |
| Contenedor | `redis-server` |
| Imagen | `redis:7.2-alpine` |
| Puerto | `6379` |
| Persistencia | AOF habilitado |
| Volumen | `redis_data` |
| Ruta interna | `/data` |

## Conectarse

Desde el contenedor:

```bash
docker compose exec redis redis-cli
```

Desde tu maquina, si tienes `redis-cli` instalado:

```bash
redis-cli -h localhost -p 6379 ping
```

Probar comandos:

```bash
docker compose exec redis redis-cli ping
docker compose exec redis redis-cli SET saludo "hola"
docker compose exec redis redis-cli GET saludo
```

## Detener y eliminar

```bash
docker compose stop
docker compose down
```

Para borrar el volumen:

```bash
docker compose down -v
```
