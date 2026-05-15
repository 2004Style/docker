# MongoDB

Base de datos MongoDB para desarrollo local.

## Para que proyectos usarlo

MongoDB es una base de datos documental. Guarda documentos JSON/BSON y es comoda cuando el esquema cambia mucho.

| Tipo de proyecto | Ejemplos |
| --- | --- |
| APIs con documentos flexibles | Perfiles, configuraciones, formularios dinamicos |
| Prototipos rapidos | MVPs donde el modelo cambia seguido |
| Catalogos variables | Productos con atributos diferentes por categoria |
| Eventos y logs de aplicacion | Registros semiestructurados, auditorias simples |
| Contenido JSON | CMS, metadata, payloads externos |

Usalo cuando tus datos encajan mejor como documentos completos y no necesitas muchas relaciones complejas. Para transacciones relacionales fuertes, PostgreSQL o MySQL suelen ser mejores.

## Levantar

```bash
docker compose up -d
docker compose ps
docker compose logs -f mongodb
```

## Datos importantes

| Dato | Valor |
| --- | --- |
| Servicio Compose | `mongodb` |
| Contenedor | `mongo-server` |
| Imagen | `mongo:latest` |
| Puerto | `27017` |
| Usuario root | `rdev` |
| Password root | `rdev` |
| Volumen | `mongo_data` |
| Ruta interna | `/data/db` |

## Conectarse

Desde el contenedor:

```bash
docker compose exec mongodb mongosh -u rdev -p rdev --authenticationDatabase admin
```

Desde tu maquina, si tienes `mongosh` instalado:

```bash
mongosh "mongodb://rdev:rdev@localhost:27017/admin"
```

Probar conexion:

```bash
docker compose exec mongodb mongosh -u rdev -p rdev --authenticationDatabase admin --eval "db.adminCommand({ ping: 1 })"
```

## Backup y restore

```bash
docker compose exec mongodb mongodump -u rdev -p rdev --authenticationDatabase admin --archive > mongo-backup.archive
docker compose exec -T mongodb mongorestore -u rdev -p rdev --authenticationDatabase admin --archive < mongo-backup.archive
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
