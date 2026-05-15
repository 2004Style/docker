# PostgreSQL

Base de datos PostgreSQL para desarrollo local.

## Para que proyectos usarlo

PostgreSQL es una base de datos relacional muy completa. Es una buena opcion por defecto para la mayoria de aplicaciones.

| Tipo de proyecto | Ejemplos |
| --- | --- |
| APIs y backends | REST, GraphQL, monolitos, microservicios con datos relacionales |
| SaaS y sistemas administrativos | Usuarios, roles, permisos, facturacion, reportes |
| E-commerce | Productos, ordenes, pagos, inventario |
| Sistemas transaccionales | Datos consistentes, relaciones, constraints, auditoria |
| Analitica moderada | Consultas SQL, vistas, agregaciones, reportes internos |
| Datos geograficos | Apps con mapas usando PostGIS |

Usalo cuando tus datos tienen relaciones claras, necesitas transacciones fuertes o quieres SQL potente. Si necesitas documentos flexibles sin esquema fijo, MongoDB puede ser mas comodo.

## Levantar

```bash
docker compose up -d
docker compose ps
docker compose logs -f postgres
```

## Datos importantes

| Dato | Valor |
| --- | --- |
| Servicio Compose | `postgres` |
| Contenedor | `postgres-server` |
| Imagen | `postgres:16.2-alpine` |
| Puerto | `5432` |
| Base de datos | `postgres` |
| Usuario | `rdev` |
| Password | `rdev` |
| Volumen | `postgres_data` |
| Ruta interna | `/var/lib/postgresql/data` |

## Conectarse

Desde el contenedor:

```bash
docker compose exec postgres psql -U rdev -d postgres
```

Desde tu maquina, si tienes `psql` instalado:

```bash
psql -h localhost -p 5432 -U rdev -d postgres
```

Probar conexion:

```bash
docker compose exec postgres psql -U rdev -d postgres -c "SELECT version();"
```

## Backup y restore

```bash
docker compose exec postgres pg_dump -U rdev postgres > backup.sql
docker compose exec -T postgres psql -U rdev -d postgres < backup.sql
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
