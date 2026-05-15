# MySQL

Base de datos MySQL para desarrollo local.

## Para que proyectos usarlo

MySQL es una base de datos relacional muy usada en aplicaciones web y stacks tradicionales.

| Tipo de proyecto | Ejemplos |
| --- | --- |
| Aplicaciones web | Laravel, WordPress, Drupal, PHP, Java, Node.js |
| Sistemas CRUD | Paneles administrativos, inventarios, ventas, formularios |
| E-commerce | Tiendas, catalogos, ordenes, clientes |
| Apps existentes | Proyectos que ya dependen de MySQL o MariaDB |
| Hosting tradicional | Ambientes compartidos o proveedores con soporte MySQL |

Usalo cuando tu framework, hosting o proyecto ya esta orientado a MySQL. Si empiezas desde cero y necesitas SQL avanzado, PostgreSQL tambien es una excelente opcion.

## Levantar

```bash
docker compose up -d
docker compose ps
docker compose logs -f mysql
```

## Datos importantes

| Dato | Valor |
| --- | --- |
| Servicio Compose | `mysql` |
| Contenedor | `mysql-server` |
| Imagen | `mysql:latest` |
| Puerto | `3306` |
| Root password | `root` |
| Base de datos | `pruebas` |
| Usuario | `rdev` |
| Password | `rdev` |
| Datos locales | `mysql_data/` |

`mysql_data/` es un bind mount local. Docker puede crear la carpeta al levantar el servicio; si quieres crearla antes:

```bash
mkdir -p mysql_data
```

## Conectarse

Como root:

```bash
docker compose exec mysql mysql -uroot -proot
```

Como usuario de la app:

```bash
docker compose exec mysql mysql -urdev -prdev pruebas
```

Desde tu maquina, si tienes cliente MySQL:

```bash
mysql -h localhost -P 3306 -urdev -prdev pruebas
```

## Backup y restore

```bash
docker compose exec mysql mysqldump -uroot -proot pruebas > backup.sql
docker compose exec -T mysql mysql -uroot -proot pruebas < backup.sql
```

## Detener y eliminar

```bash
docker compose stop
docker compose down
```

Para borrar datos locales:

```bash
docker compose down
rm -rf mysql_data
```
