# DocuSeal

Plataforma de firma electronica open-source (alternativa a DocuSign). Crear, enviar y gestionar firmas de documentos.

## Para que proyectos usarlo

DocuSeal es ideal cuando necesitas firma electronica de documentos sin depender de SaaS externos.

| Tipo de proyecto | Ejemplos |
| --- | --- |
| Contratos digitales | Contratos de servicio, NDAs, acuerdos legales |
| Onboarding de clientes | Formularios de registro con firma, aceptacion de terminos |
| Documentos HR | Contratos laborales, cartas oferta, finiquitos |
| Facturacion | Facturas con conformidad, presupuestos firmados |
| API de firma | Integrar firma electronica en tu app via REST API |

## Levantar

```bash
docker compose up -d
docker compose ps
docker compose logs -f docuseal
```

## Acceder

Abrir [http://localhost:3000](http://localhost:3000) y crear la cuenta de administrador en el primer inicio.

## Datos importantes

| Dato | Valor |
| --- | --- |
| Servicios Compose | `docuseal`, `postgres` |
| Contenedor app | `docuseal-server` |
| Contenedor BD | `docuseal-postgres` |
| Imagen app | `docuseal/docuseal:latest` |
| Imagen BD | `postgres:16-alpine` |
| Puerto app | `3000` |
| BD usuario | `rdev` |
| BD password | `rdev` |
| BD nombre | `docuseal` |
| Volumen app | `docuseal_data` |
| Volumen BD | `postgres_data` |

## Usar Postgres del host (compartido)

Si ya tienes el Postgres compartido corriendo (`postgres-server`) y quieres evitar otro contenedor de BD:

1. Comenta el servicio `postgres` de este compose.
2. Cambia `DATABASE_URL` en `docuseal` a:

   ```yaml
   DATABASE_URL: postgresql://rdev:rdev@host.docker.internal:5432/docuseal
   ```

3. Agrega `extra_hosts` para que funcione en Linux:

   ```yaml
   extra_hosts:
     - "host.docker.internal:host-gateway"
   ```

4. Crea la BD manualmente:

   ```bash
   docker compose exec postgres psql -U rdev -d postgres -c "CREATE DATABASE docuseal;"
   ```

## Configurar SECRET_KEY_BASE

Genera una clave segura y ponla en un archivo `.env`:

```bash
openssl rand -base64 32
```

Luego crear `.env`:

```
SECRET_KEY_BASE=el_valor_generado
```

## Conectarse a la BD

```bash
docker compose exec postgres psql -U rdev -d docuseal
```

## Logs

```bash
docker compose logs -f docuseal
docker compose logs -f postgres
```

## Detener y eliminar

```bash
docker compose stop
docker compose down
```

Para borrar los volumenes (pierde datos):

```bash
docker compose down -v
```
