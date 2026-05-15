# NATS con JetStream

Servidor NATS con JetStream habilitado para mensajeria, streams y persistencia.

## Para que proyectos usarlo

NATS es muy util para sistemas distribuidos que necesitan mensajeria rapida entre servicios.

| Tipo de proyecto | Ejemplos |
| --- | --- |
| Microservicios | Eventos entre APIs, workers, servicios internos |
| Arquitectura event-driven | Publicar eventos de negocio, reaccionar a cambios |
| Workers y jobs | Procesamiento asincrono, tareas en segundo plano |
| Streaming liviano | JetStream para mensajes persistentes, consumidores y replay |
| Sistemas cloud-native | Servicios pequenos, comunicacion interna, baja latencia |
| IoT backend | Recibir eventos desde gateways y distribuirlos a servicios |

Usalo cuando quieres comunicacion rapida tipo pub/sub, request/reply o streams persistentes. Si solo necesitas cache usa Redis; si necesitas MQTT directo para dispositivos, usa Mosquitto.

## Levantar

```bash
docker compose up -d
docker compose ps
docker compose logs -f nats
```

## Datos importantes

| Dato | Valor |
| --- | --- |
| Servicio Compose | `nats` |
| Contenedor | `nats-server` |
| Imagen | `nats:2.10-alpine` |
| Puerto cliente | `4222` |
| Puerto HTTP monitoreo | `8222` |
| Volumen | `nats_data` |
| Ruta interna | `/data` |

## Probar salud

```bash
curl http://localhost:8222/healthz
```

Si no tienes `curl`, mira el healthcheck:

```bash
docker compose ps
```

## Usar herramientas NATS

La imagen del servidor no siempre trae todas las herramientas CLI. Usa `natsio/nats-box`:

```bash
docker run --rm -it --network container:nats-server natsio/nats-box:latest
```

Dentro de `nats-box`:

```bash
nats --server nats://127.0.0.1:4222 server check
nats --server nats://127.0.0.1:4222 stream ls
nats --server nats://127.0.0.1:4222 pub pruebas "hola"
nats --server nats://127.0.0.1:4222 sub pruebas
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
