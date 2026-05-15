# MQTT con Eclipse Mosquitto

Broker MQTT generico para desarrollo local y proyectos que necesiten publicar/suscribirse a mensajes.

## Para que proyectos usarlo

MQTT es ideal cuando necesitas mensajes livianos, constantes y con poco consumo de red.

| Tipo de proyecto | Ejemplos |
| --- | --- |
| IoT | Sensores, medidores, domotica, telemetria, dispositivos conectados |
| Tiempo real liviano | Estados de dispositivos, presencia, comandos remotos |
| Automatizacion | Home Assistant, control de luces, relays, alarmas, actuadores |
| Edge computing | Gateways locales que reciben datos y luego los sincronizan |
| Prototipos hardware | ESP32, Arduino, Raspberry Pi, dispositivos industriales |

Usalo cuando tienes muchos clientes pequenos publicando eventos por topics. Si necesitas colas complejas, reintentos avanzados, streams o comunicacion fuerte entre microservicios, probablemente NATS, RabbitMQ o Kafka encajen mejor.

## Levantar

```bash
docker compose up -d
docker compose ps
docker compose logs -f mqtt
```

## Datos importantes

| Dato | Valor |
| --- | --- |
| Servicio Compose | `mqtt` |
| Contenedor | `mqtt-broker` |
| Imagen | `eclipse-mosquitto:2` |
| Puerto | `1883` |
| Configuracion | `config/mosquitto.conf` |
| Datos | `data/` |
| Logs | `log/` |

`data/` y `log/` se crean automaticamente cuando ejecutas `docker compose up -d` porque son bind mounts. Si quieres dejarlas listas antes de levantar:

```bash
mkdir -p data log
```

Si aparece un error de permisos al escribir persistencia o logs, revisa permisos de esas carpetas.

## Configuracion actual

La configuracion usa:

- `persistence true`: guarda datos persistentes.
- `persistence_location /mosquitto/data/`: carpeta de persistencia dentro del contenedor.
- `autosave_interval 60`: guarda estado cada 60 segundos.
- `log_dest stdout`: permite ver logs con `docker compose logs`.
- `log_dest file /mosquitto/log/mosquitto.log`: guarda logs en la carpeta `log/`.
- `log_type error`, `warning`, `notice`, `information`: muestra niveles utiles de log.
- `allow_anonymous true`: permite conexiones sin usuario/password para desarrollo.
- `listener 1883 0.0.0.0`: escucha MQTT en el puerto `1883`.

Para produccion, no uses `allow_anonymous true`; configura usuarios, ACLs y TLS.

## Probar MQTT

Ver uptime del broker:

```bash
docker compose exec mqtt mosquitto_sub -h 127.0.0.1 -p 1883 -t '$SYS/broker/uptime' -C 1
```

Suscribirte a un topic:

```bash
docker compose exec mqtt mosquitto_sub -h 127.0.0.1 -p 1883 -t sensores/temperatura
```

Publicar un mensaje desde otra terminal:

```bash
docker compose exec mqtt mosquitto_pub -h 127.0.0.1 -p 1883 -t sensores/temperatura -m "24.5"
```

## Detener y eliminar

```bash
docker compose stop
docker compose down
```

Para borrar tambien datos y logs locales:

```bash
docker compose down
rm -rf data log
```
