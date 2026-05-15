# SQL Server

Base de datos Microsoft SQL Server para desarrollo local.

## Para que proyectos usarlo

SQL Server es una base de datos relacional empresarial, comun en ecosistemas Microsoft.

| Tipo de proyecto | Ejemplos |
| --- | --- |
| Aplicaciones .NET | ASP.NET Core, Entity Framework, sistemas corporativos |
| Empresas con stack Microsoft | Integracion con herramientas Microsoft, BI, reporting |
| Sistemas legacy | Proyectos que ya usan T-SQL, stored procedures o SQL Server |
| ERP/CRM internos | Datos relacionales, auditoria, reportes, procesos de negocio |
| Analitica corporativa | Reporting, integracion con Power BI y herramientas Microsoft |

Usalo cuando el proyecto o cliente trabaja con SQL Server, T-SQL o stack Microsoft. Para proyectos pequenos o portables, PostgreSQL/MySQL pueden ser mas ligeros.

## Levantar

```bash
docker compose up -d
docker compose ps
docker compose logs -f sqlserver
```

## Datos importantes

| Dato | Valor |
| --- | --- |
| Servicio Compose | `sqlserver` |
| Contenedor | `sqlserver` |
| Imagen | `mcr.microsoft.com/mssql/server:2022-latest` |
| Puerto | `1433` |
| Usuario administrador | `sa` |
| Password | `Ron@ld1234` |
| Volumen | `sql_data` |
| Ruta interna | `/var/opt/mssql` |

## Conectarse

Si `sqlcmd` esta dentro del contenedor:

```bash
docker compose exec sqlserver /opt/mssql-tools18/bin/sqlcmd -S localhost -U sa -P "Ron@ld1234" -C -Q "SELECT @@VERSION"
```

Si no esta disponible, usa una imagen herramienta:

```bash
docker run --rm -it --network container:sqlserver mcr.microsoft.com/mssql-tools /opt/mssql-tools/bin/sqlcmd -S localhost -U sa -P "Ron@ld1234" -Q "SELECT @@VERSION"
```

Desde tu maquina, si tienes `sqlcmd`:

```bash
sqlcmd -S localhost,1433 -U sa -P "Ron@ld1234" -C
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
