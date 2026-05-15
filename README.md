# Guia completa de Docker Compose

Esta guia explica Docker Compose desde nivel basico hasta avanzado. Usa el comando moderno `docker compose` de Docker Compose V2, no el comando antiguo `docker-compose`.

Docker Compose sirve para definir y administrar aplicaciones compuestas por uno o varios contenedores usando un archivo YAML, normalmente llamado `docker-compose.yml` o `compose.yml`.

Fuentes oficiales recomendadas:

- [Compose file reference](https://docs.docker.com/reference/compose-file/)
- [Services reference](https://docs.docker.com/reference/compose-file/services/)
- [Compose Specification](https://compose-spec.github.io/compose-spec/spec.html)
- [docker compose up](https://docs.docker.com/reference/cli/docker/compose/up/)
- [docker compose down](https://docs.docker.com/reference/cli/docker/compose/down/)

> Nota importante: en Compose moderno la propiedad `version` esta obsoleta. Puedes omitirla. Docker Compose usa la especificacion mas reciente que soporte tu instalacion.

---

## 1. Que es Docker Compose

Docker ejecuta contenedores individuales. Docker Compose permite describir una aplicacion completa en un solo archivo:

- Servicios: contenedores que forman tu aplicacion, por ejemplo `api`, `postgres`, `redis`.
- Redes: canales de comunicacion entre servicios.
- Volumenes: almacenamiento persistente.
- Variables de entorno: configuracion de cada servicio.
- Healthchecks: pruebas para saber si un servicio esta saludable.
- Dependencias: orden de arranque entre servicios.

Un servicio de Compose no es exactamente un contenedor: es una definicion. Al ejecutar `docker compose up`, Compose crea uno o mas contenedores a partir de esa definicion.

---

## 2. Estructura basica de un archivo Compose

```yaml
name: mi-proyecto

services:
  postgres:
    image: postgres:16.2-alpine
    container_name: postgres-server
    restart: unless-stopped
    environment:
      POSTGRES_USER: rdev
      POSTGRES_PASSWORD: rdev
      POSTGRES_DB: postgres
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data

volumes:
  postgres_data:
```

### Partes principales

| Propiedad | Donde va | Que es | Para que sirve |
| --- | --- | --- | --- |
| `name` | Raiz | Nombre del proyecto Compose | Agrupa contenedores, redes y volumenes bajo un mismo proyecto |
| `services` | Raiz | Lista de servicios | Define los contenedores que se van a ejecutar |
| `image` | Servicio | Imagen Docker | Indica de que imagen se crea el contenedor |
| `container_name` | Servicio | Nombre fijo del contenedor | Facilita identificarlo, aunque limita escalado |
| `restart` | Servicio | Politica de reinicio | Define si Docker debe reiniciar el contenedor automaticamente |
| `environment` | Servicio | Variables de entorno | Configura la app dentro del contenedor |
| `ports` | Servicio | Mapeo de puertos | Expone puertos del contenedor hacia tu maquina |
| `volumes` | Servicio y raiz | Montajes y volumenes | Persiste datos o monta archivos/carpetas |
| `networks` | Servicio y raiz | Redes Docker | Controla como se comunican los servicios |
| `configs` | Servicio y raiz | Archivos de configuracion no sensibles | Monta configuracion sin reconstruir imagenes |
| `secrets` | Servicio y raiz | Datos sensibles | Monta claves, passwords o tokens de forma mas segura |

---

## 3. YAML basico para Compose

Compose usa YAML. Reglas importantes:

- La indentacion importa. Usa espacios, no tabs.
- Los bloques se organizan por niveles.
- Las listas usan `-`.
- Los strings con `:` o valores especiales conviene ponerlos entre comillas.

Ejemplo:

```yaml
ports:
  - "5432:5432"

environment:
  POSTGRES_USER: rdev
  POSTGRES_PASSWORD: rdev
```

Tambien puedes escribir algunas propiedades como lista:

```yaml
environment:
  - POSTGRES_USER=rdev
  - POSTGRES_PASSWORD=rdev
```

La forma de mapa suele ser mas clara.

---

## 4. Ejemplo basico: un solo servicio con Postgres

```yaml
services:
  postgres:
    image: postgres:16.2-alpine
    container_name: postgres-server
    restart: unless-stopped
    environment:
      POSTGRES_USER: rdev
      POSTGRES_PASSWORD: rdev
      POSTGRES_DB: postgres
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data

volumes:
  postgres_data:
```

### Explicacion linea por linea

| Linea | Explicacion |
| --- | --- |
| `services:` | Aqui se declaran los servicios de la aplicacion |
| `postgres:` | Nombre del servicio dentro de Compose |
| `image: postgres:16.2-alpine` | Usa la imagen oficial de Postgres, version ligera Alpine |
| `container_name: postgres-server` | El contenedor se llamara `postgres-server` |
| `restart: unless-stopped` | Reinicia automaticamente salvo que lo detengas manualmente |
| `environment:` | Variables que Postgres usa para inicializarse |
| `ports: "5432:5432"` | Expone el puerto 5432 del contenedor en el puerto 5432 del host |
| `volumes:` | Persiste la data de Postgres fuera del ciclo de vida del contenedor |
| `postgres_data:` | Volumen administrado por Docker |

---

## 5. Ejemplo intermedio: varios servicios

Ejemplo con una API, Postgres y Redis:

```yaml
name: app-ejemplo

services:
  api:
    build:
      context: .
      dockerfile: Dockerfile
    container_name: app-api
    restart: unless-stopped
    depends_on:
      postgres:
        condition: service_healthy
      redis:
        condition: service_started
    environment:
      DATABASE_URL: postgres://rdev:rdev@postgres:5432/app
      REDIS_URL: redis://redis:6379
    ports:
      - "3000:3000"
    networks:
      - backend

  postgres:
    image: postgres:16.2-alpine
    restart: unless-stopped
    environment:
      POSTGRES_USER: rdev
      POSTGRES_PASSWORD: rdev
      POSTGRES_DB: app
    volumes:
      - postgres_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U rdev -d app"]
      interval: 10s
      timeout: 5s
      retries: 5
    networks:
      - backend

  redis:
    image: redis:7.2-alpine
    restart: unless-stopped
    command: "redis-server --appendonly yes"
    volumes:
      - redis_data:/data
    networks:
      - backend

networks:
  backend:

volumes:
  postgres_data:
  redis_data:
```

### Puntos clave

- Los servicios se comunican por el nombre del servicio, por ejemplo `postgres` y `redis`.
- La API no usa `localhost` para conectarse a Postgres. Usa `postgres`, porque ese es el DNS interno de Compose.
- `depends_on` controla orden de arranque. Con `condition: service_healthy`, la API espera a que Postgres pase su healthcheck.
- `volumes` persiste datos aunque se eliminen y creen contenedores nuevos.
- `networks` aisla la comunicacion entre servicios.

---

## 6. Servicios existentes en este repositorio

Este repositorio tiene ejemplos separados para servicios comunes:

| Carpeta | Servicio | Comando para levantarlo |
| --- | --- | --- |
| `postgres/` | Postgres | `cd postgres && docker compose up -d` |
| `mysql/` | MySQL | `cd mysql && docker compose up -d` |
| `redis/` | Redis | `cd redis && docker compose up -d` |
| `mongo-db/` | MongoDB | `cd mongo-db && docker compose up -d` |
| `mqtt/` | MQTT con Eclipse Mosquitto | `cd mqtt && docker compose up -d` |
| `nats/` | NATS con JetStream | `cd nats && docker compose up -d` |
| `sqlserver/` | SQL Server | `cd sqlserver && docker compose up -d` |

Cada carpeta incluye su propio `README.md` con comandos especificos para levantar, conectarse, revisar logs, hacer pruebas y borrar datos.

Ejemplo de operacion:

```bash
cd postgres
docker compose up -d
docker compose ps
docker compose logs -f postgres
```

---

## 7. `restart`: que es y para que sirve

`restart` define que debe hacer Docker cuando un contenedor se detiene. Es util para servicios persistentes como bases de datos, caches, colas, brokers o APIs que deben volver a levantarse si fallan o si reinicias la maquina.

No reemplaza un healthcheck, no corrige errores de configuracion y no actualiza imagenes. Solo decide si Docker debe intentar iniciar de nuevo el contenedor.

| Estado | Que hace | Cuando usarlo |
| --- | --- | --- |
| `"no"` | No reinicia el contenedor automaticamente. Debe ir entre comillas porque `no` puede interpretarse como booleano en YAML. | Pruebas, tareas temporales o contenedores que deben terminar. |
| `always` | Siempre intenta reiniciar el contenedor si se detiene. Tambien vuelve despues de reiniciar Docker. | Servicios criticos que deben volver siempre. |
| `on-failure` | Reinicia solo si el proceso principal termina con codigo distinto de `0`. | Procesos que pueden terminar correctamente y no deben volver en ese caso. |
| `on-failure:3` | Igual que `on-failure`, pero con maximo de reintentos. | Evitar ciclos infinitos de reinicio. |
| `unless-stopped` | Reinicia automaticamente salvo si lo detuviste manualmente con `docker compose stop` o `docker stop`. | Recomendado para la mayoria de servicios persistentes locales. |

Ejemplo:

```yaml
services:
  redis:
    image: redis:7.2-alpine
    restart: unless-stopped
```

Errores comunes:

- Usar `restart` esperando que actualice imagenes. Para actualizar usa `docker compose pull` y luego `docker compose up -d`.
- Usar `always` en servicios de prueba que quieres detener manualmente.
- Escribir `restart: no` sin comillas. Mejor usa `restart: "no"`.
- Confundir `restart` con `healthcheck`. Un contenedor puede estar "arriba" pero no saludable.

---

## 8. Puertos: `ports` vs `expose`

### `ports`

Publica un puerto del contenedor en tu maquina.

```yaml
ports:
  - "8080:80"
```

Formato:

```text
"PUERTO_HOST:PUERTO_CONTENEDOR"
```

Ejemplos:

| Configuracion | Significado |
| --- | --- |
| `"5432:5432"` | El host escucha 5432 y lo envia al 5432 del contenedor |
| `"8080:80"` | El host escucha 8080 y lo envia al 80 del contenedor |
| `"127.0.0.1:5432:5432"` | Solo expone el puerto en localhost |
| `"80"` | Docker elige un puerto aleatorio en el host |

### `expose`

Documenta o expone puertos solo dentro de las redes de Docker, no hacia tu maquina.

```yaml
expose:
  - "3000"
```

Usa `ports` cuando necesitas acceder desde tu navegador, cliente SQL o herramientas externas. Usa `expose` cuando solo otros servicios de Compose deben acceder.

---

## 9. Volumenes: persistencia de datos

Sin volumenes, si eliminas un contenedor puedes perder datos que vivian dentro de el.

### Volumen nombrado

```yaml
services:
  postgres:
    image: postgres:16.2-alpine
    volumes:
      - postgres_data:/var/lib/postgresql/data

volumes:
  postgres_data:
```

Docker administra el volumen. Es lo recomendado para bases de datos.

### Bind mount

```yaml
services:
  app:
    image: node:22-alpine
    volumes:
      - ./src:/app/src
```

Monta una carpeta local dentro del contenedor. Es comun en desarrollo.

### Sintaxis larga

```yaml
volumes:
  - type: volume
    source: postgres_data
    target: /var/lib/postgresql/data
    read_only: false
```

Advertencia: `docker compose down -v` elimina volumenes definidos por Compose. En bases de datos eso puede borrar datos importantes.

---

## 10. Variables de entorno

### Directas en el Compose

```yaml
environment:
  POSTGRES_USER: rdev
  POSTGRES_PASSWORD: rdev
```

### Desde archivo `.env`

Archivo `.env`:

```env
POSTGRES_USER=rdev
POSTGRES_PASSWORD=rdev
```

Compose:

```yaml
environment:
  POSTGRES_USER: ${POSTGRES_USER}
  POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
```

### Con `env_file`

```yaml
env_file:
  - .env
```

Diferencia importante:

- `.env` en la raiz se usa para interpolar valores dentro del YAML.
- `env_file` pasa variables al contenedor.

---

## 11. Redes

Compose crea una red por defecto para el proyecto. Por eso los servicios pueden encontrarse por nombre.

```yaml
services:
  api:
    image: mi-api
    networks:
      - backend

  postgres:
    image: postgres:16.2-alpine
    networks:
      - backend

networks:
  backend:
```

Ejemplo de conexion desde la API:

```text
postgres://rdev:rdev@postgres:5432/app
```

No uses `localhost` para comunicar contenedores entre si. Dentro de un contenedor, `localhost` apunta al mismo contenedor, no a otro servicio.

---

## 12. Healthchecks

Un healthcheck permite saber si un servicio esta realmente listo.

```yaml
healthcheck:
  test: ["CMD-SHELL", "pg_isready -U rdev -d app"]
  interval: 10s
  timeout: 5s
  retries: 5
  start_period: 20s
```

| Campo | Que hace |
| --- | --- |
| `test` | Comando que valida salud |
| `interval` | Cada cuanto se ejecuta |
| `timeout` | Tiempo maximo antes de fallar |
| `retries` | Intentos antes de marcar como unhealthy |
| `start_period` | Tiempo inicial de gracia |
| `disable` | Desactiva healthcheck de la imagen |

Uso con `depends_on`:

```yaml
depends_on:
  postgres:
    condition: service_healthy
```

---

## 13. Comandos para operar un solo servicio

En estos ejemplos el servicio se llama `postgres`.

| Accion | Comando | Que hace |
| --- | --- | --- |
| Levantar | `docker compose up -d postgres` | Crea e inicia solo el servicio `postgres` |
| Detener | `docker compose stop postgres` | Detiene el contenedor sin eliminarlo |
| Iniciar de nuevo | `docker compose start postgres` | Inicia un contenedor detenido |
| Reiniciar | `docker compose restart postgres` | Detiene e inicia el servicio |
| Ver logs | `docker compose logs -f postgres` | Sigue los logs en tiempo real |
| Ver estado | `docker compose ps postgres` | Muestra estado del servicio |
| Ejecutar comando | `docker compose exec postgres sh` | Entra al contenedor en ejecucion |
| Eliminar detenido | `docker compose rm postgres` | Elimina el contenedor detenido del servicio |
| Actualizar imagen | `docker compose pull postgres` | Descarga la version mas reciente de la imagen |
| Recrear actualizado | `docker compose up -d postgres` | Recrea si la imagen o configuracion cambio |
| Forzar recreacion | `docker compose up -d --force-recreate postgres` | Recrea aunque Compose no detecte cambios |

Actualizar un solo servicio:

```bash
docker compose pull postgres
docker compose up -d postgres
docker compose logs -f postgres
```

Bajar un solo servicio:

```bash
docker compose stop postgres
docker compose rm postgres
```

Tambien existe:

```bash
docker compose down postgres
```

Pero `down` esta pensado para bajar el proyecto completo o servicios especificos eliminando contenedores y redes relacionadas segun el caso. Para operar de forma fina un servicio, normalmente es mas claro usar `stop` y `rm`.

---

## 14. Comandos para operar todos los servicios

| Accion | Comando | Que hace |
| --- | --- | --- |
| Levantar todo | `docker compose up -d` | Crea e inicia todos los servicios |
| Levantar viendo logs | `docker compose up` | Inicia y muestra logs en primer plano |
| Detener sin eliminar | `docker compose stop` | Detiene todos los contenedores del proyecto |
| Iniciar detenidos | `docker compose start` | Inicia contenedores detenidos |
| Reiniciar todo | `docker compose restart` | Reinicia todos los servicios |
| Bajar proyecto | `docker compose down` | Detiene y elimina contenedores y redes creadas por `up` |
| Bajar con volumenes | `docker compose down -v` | Tambien elimina volumenes del proyecto |
| Actualizar imagenes | `docker compose pull` | Descarga versiones nuevas de imagenes |
| Recrear actualizado | `docker compose up -d` | Aplica imagenes/configuracion nuevas |
| Ver estado | `docker compose ps` | Lista servicios y contenedores |
| Ver logs | `docker compose logs -f` | Sigue logs de todos los servicios |
| Ver configuracion final | `docker compose config` | Muestra el Compose resuelto y validado |
| Construir imagenes | `docker compose build` | Construye servicios con `build` |
| Construir sin cache | `docker compose build --no-cache` | Reconstruye desde cero |

Actualizar todo:

```bash
docker compose pull
docker compose up -d
docker compose ps
```

Bajar todo sin borrar datos persistentes:

```bash
docker compose down
```

Bajar todo borrando volumenes:

```bash
docker compose down -v
```

> Cuidado: `docker compose down -v` puede eliminar bases de datos, colas persistentes y otros datos guardados en volumenes administrados por Compose.

---

## 15. Propiedades de nivel raiz

| Propiedad | Nivel | Tipo/formato | Que es | Para que se usa | Ejemplo minimo | Notas |
| --- | --- | --- | --- | --- | --- | --- |
| `name` | Basico | String | Nombre del proyecto Compose | Agrupar recursos con un nombre estable | `name: mi-app` | Tambien puedes usar `-p` o `COMPOSE_PROJECT_NAME` |
| `services` | Basico | Mapa | Definicion de servicios | Declarar contenedores de la aplicacion | `services: { app: { image: nginx } }` | Es la seccion principal |
| `networks` | Intermedio | Mapa | Redes reutilizables | Aislar o conectar servicios | `networks: { backend: {} }` | Si no declaras una, Compose crea una default |
| `volumes` | Basico | Mapa | Volumenes nombrados | Persistir datos | `volumes: { db_data: {} }` | No se borran con `down` salvo que uses `-v` |
| `configs` | Avanzado | Mapa | Configuracion montada como archivo | Separar config de imagenes | `configs: { app_conf: { file: ./app.conf } }` | No es para secretos |
| `secrets` | Avanzado | Mapa | Datos sensibles montados como archivo | Manejar passwords/tokens mejor que env vars | `secrets: { db_pass: { file: ./db.pass } }` | En local sigue dependiendo del entorno Docker |
| `include` | Super avanzado | Lista | Incluye otros archivos Compose | Componer proyectos mantenidos aparte | `include: [./compose.db.yml]` | Requiere soporte en tu version de Compose |
| `x-*` | Avanzado | Cualquier YAML | Extensiones ignoradas por Compose | Reutilizar bloques con anchors o documentar | `x-common: &common` | Muy util para evitar duplicacion |
| `version` | Legacy | String | Campo historico del formato | Compatibilidad antigua | `version: "3.8"` | Obsoleto en Compose moderno; mejor omitirlo |

---

## 16. Referencia de propiedades por servicio

Esta tabla resume las propiedades que puede aceptar un servicio en Docker Compose. Algunas dependen de la version de Docker Compose, del sistema operativo, del runtime o de si usas Docker local, Swarm u otra plataforma compatible.

| Propiedad | Nivel | Tipo/formato | Que es | Para que se usa | Ejemplo minimo | Notas |
| --- | --- | --- | --- | --- | --- | --- |
| `annotations` | Avanzado | Mapa o lista | Metadatos tipo anotacion | Integraciones externas o plataformas que leen anotaciones | `annotations: { com.example.foo: bar }` | Similar a labels, pero semantica de anotaciones |
| `attach` | Avanzado | Booleano | Controla si Compose recolecta logs del servicio | Reducir ruido en `docker compose up` | `attach: false` | Por defecto es `true` |
| `build` | Intermedio | String o mapa | Configuracion para construir imagen | Crear imagen desde Dockerfile local | `build: .` | Puede combinarse con `image` para etiquetar |
| `blkio_config` | Super avanzado | Mapa | Control de I/O de bloques | Limitar/priorizar acceso a disco | `blkio_config: { weight: 300 }` | Depende del host Linux |
| `cpu_count` | Avanzado | Numero | Cantidad de CPUs visibles | Limitar CPU en plataformas que lo soportan | `cpu_count: 2` | Mas comun en Windows |
| `cpu_percent` | Avanzado | Numero | Porcentaje de CPU | Limitar CPU en plataformas compatibles | `cpu_percent: 50` | Depende del runtime |
| `cpu_shares` | Avanzado | Numero | Peso relativo de CPU | Prioridad relativa cuando hay contencion | `cpu_shares: 512` | No es limite absoluto |
| `cpu_period` | Super avanzado | Duracion/numero | Periodo CFS | Ajuste fino de CPU en Linux | `cpu_period: 100000` | Usar solo si sabes CFS |
| `cpu_quota` | Super avanzado | Numero | Cuota CFS | Limitar tiempo de CPU por periodo | `cpu_quota: 50000` | Con `cpu_period` define limite |
| `cpu_rt_runtime` | Super avanzado | Duracion/numero | Runtime CPU realtime | Cargas realtime | `cpu_rt_runtime: 400ms` | Requiere kernel/config especial |
| `cpu_rt_period` | Super avanzado | Duracion/numero | Periodo CPU realtime | Cargas realtime | `cpu_rt_period: 1400us` | No suele usarse en desarrollo |
| `cpus` | Intermedio | String/numero | Numero de CPUs asignadas | Limitar CPU de forma simple | `cpus: 1.5` | Practico para local |
| `cpuset` | Avanzado | String | CPUs especificas | Fijar afinidad de CPU | `cpuset: "0,1"` | Depende del host |
| `cap_add` | Avanzado | Lista | Capacidades Linux agregadas | Dar permisos especificos sin `privileged` | `cap_add: [NET_ADMIN]` | Preferible a `privileged` |
| `cap_drop` | Avanzado | Lista | Capacidades Linux removidas | Endurecer seguridad | `cap_drop: [ALL]` | Luego agrega solo lo necesario |
| `cgroup` | Super avanzado | String | Namespace de cgroup | Control de aislamiento cgroup | `cgroup: private` | Soporte variable |
| `cgroup_parent` | Super avanzado | String | Cgroup padre | Integrar con jerarquias externas | `cgroup_parent: m-executor-abcd` | Uso infra avanzado |
| `command` | Basico | String o lista | Sobrescribe CMD de la imagen | Cambiar comando principal | `command: npm run dev` | No cambia ENTRYPOINT |
| `configs` | Avanzado | Lista | Configs montadas en el servicio | Inyectar archivos de config | `configs: [app_conf]` | Deben existir en `configs` raiz |
| `container_name` | Basico | String | Nombre fijo del contenedor | Identificacion manual | `container_name: postgres-server` | Impide escalar multiples replicas del mismo servicio |
| `credential_spec` | Super avanzado | String/mapa | Credenciales gMSA Windows | Integracion con dominios Windows | `credential_spec: { file: my-credential-spec.json }` | Solo Windows containers |
| `depends_on` | Intermedio | Lista o mapa | Dependencias entre servicios | Orden de arranque/parada | `depends_on: [postgres]` | No espera salud salvo con condicion |
| `deploy` | Avanzado | Mapa | Reglas de despliegue | Replicas, recursos, placement | `deploy: { replicas: 2 }` | Muchas opciones aplican mejor en Swarm/plataformas |
| `develop` | Avanzado | Mapa | Configuracion de desarrollo | Watch/sync/rebuild en dev | `develop: { watch: [] }` | Requiere Compose moderno |
| `device_cgroup_rules` | Super avanzado | Lista | Reglas cgroup para devices | Permitir dispositivos por patron | `device_cgroup_rules: ["c 1:3 mr"]` | Linux avanzado |
| `devices` | Avanzado | Lista | Mapea dispositivos del host | USB, serial, GPU u otros devices | `devices: ["/dev/ttyUSB0:/dev/ttyUSB0"]` | Requiere permisos del host |
| `dns` | Intermedio | String/lista | Servidores DNS | Resolver nombres con DNS especifico | `dns: [1.1.1.1]` | Normalmente no hace falta |
| `dns_opt` | Avanzado | Lista | Opciones DNS | Ajustar resolucion DNS | `dns_opt: [use-vc]` | Depende de resolv.conf |
| `dns_search` | Avanzado | Lista | Dominios de busqueda DNS | Resolver nombres cortos | `dns_search: [example.com]` | Uso corporativo |
| `domainname` | Avanzado | String | Dominio del contenedor | Definir FQDN junto a hostname | `domainname: example.com` | Poco comun |
| `driver_opts` | Avanzado | Mapa | Opciones del driver de red por servicio | Ajustes especificos del driver | `driver_opts: { com.docker.network.endpoint.sysctls: net.ipv4.conf.IFNAME.log_martians=1 }` | Avanzado y dependiente de driver |
| `entrypoint` | Intermedio | String/lista | Sobrescribe ENTRYPOINT | Cambiar proceso inicial | `entrypoint: ["sh", "-c"]` | Puede romper imagenes si se usa mal |
| `env_file` | Basico | String/lista/mapa | Archivo de variables | Cargar variables al contenedor | `env_file: .env` | No es lo mismo que interpolacion `.env` |
| `environment` | Basico | Mapa/lista | Variables de entorno | Configurar aplicaciones | `environment: { NODE_ENV: production }` | No ideal para secretos |
| `expose` | Intermedio | Lista | Puertos internos documentados | Exponer solo a redes Docker | `expose: ["3000"]` | No publica al host |
| `extends` | Avanzado | Mapa | Reutiliza otro servicio | Heredar configuracion | `extends: { service: base }` | Puede complicar lectura |
| `external_links` | Legacy | Lista | Enlaces a contenedores externos | Compatibilidad con setups antiguos | `external_links: [redis_1]` | Mejor usar redes externas |
| `extra_hosts` | Intermedio | Lista/mapa | Entradas extra en `/etc/hosts` | Resolver nombres manualmente | `extra_hosts: ["host.docker.internal:host-gateway"]` | Util para acceder al host Linux |
| `gpus` | Avanzado | String/lista | Acceso a GPU | ML, video, computo GPU | `gpus: all` | Requiere runtime/driver NVIDIA u otro soporte |
| `group_add` | Avanzado | Lista | Grupos Linux adicionales | Permisos sobre archivos/devices | `group_add: ["1001"]` | Puede resolver permisos de volumenes |
| `healthcheck` | Intermedio | Mapa | Prueba de salud | Saber si el servicio esta listo | `healthcheck: { test: ["CMD", "curl", "-f", "http://localhost"] }` | Puede usarse con `depends_on` |
| `hostname` | Intermedio | String | Hostname interno | Nombre del host dentro del contenedor | `hostname: db.local` | No reemplaza el nombre de servicio DNS |
| `image` | Basico | String | Imagen Docker | Crear el contenedor desde una imagen | `image: redis:7.2-alpine` | Evita `latest` en produccion |
| `init` | Intermedio | Booleano | Usa init minimo | Manejar procesos zombie/senales | `init: true` | Recomendado para apps con subprocesos |
| `ipc` | Super avanzado | String | Namespace IPC | Compartir/aislar memoria IPC | `ipc: host` | Uso especializado |
| `isolation` | Super avanzado | String | Tecnologia de aislamiento | Windows/Hyper-V/process isolation | `isolation: hyperv` | Principalmente Windows |
| `labels` | Intermedio | Mapa/lista | Metadatos | Integracion con proxies, herramientas, inventario | `labels: { traefik.enable: "true" }` | Muy usado por reverse proxies |
| `label_file` | Avanzado | Lista | Archivo de labels | Cargar labels desde archivo | `label_file: ./labels` | Requiere soporte en Compose moderno |
| `links` | Legacy | Lista | Enlaces entre servicios | Alias antiguos de red | `links: [db]` | Obsoleto en practica; usa redes |
| `logging` | Avanzado | Mapa | Driver/opciones de logs | Rotacion, destino y formato de logs | `logging: { driver: json-file, options: { max-size: "10m" } }` | Importante para evitar logs infinitos |
| `mac_address` | Super avanzado | String | MAC fija | Integraciones de red especificas | `mac_address: "02:42:ac:11:00:02"` | Puede no funcionar en todas las redes |
| `mem_limit` | Intermedio | Bytes/string | Limite maximo de memoria | Evitar que un servicio consuma todo | `mem_limit: 512m` | Muy util en local |
| `mem_reservation` | Avanzado | Bytes/string | Reserva suave de memoria | Planificacion de recursos | `mem_reservation: 256m` | No es limite duro |
| `mem_swappiness` | Super avanzado | Numero 0-100 | Uso relativo de swap | Ajustar comportamiento de memoria | `mem_swappiness: 0` | Linux avanzado |
| `memswap_limit` | Super avanzado | Bytes/string | Limite memoria + swap | Controlar swap total | `memswap_limit: 1g` | Requiere entender memoria Docker |
| `models` | Super avanzado | Lista/mapa | Modelos vinculados al servicio | Integracion con Docker Model Runner | `models: [ai_model]` | Depende de soporte Docker actual |
| `network_mode` | Avanzado | String | Modo de red especial | Usar host, none, service o container | `network_mode: host` | Incompatible con `ports` en algunos modos |
| `networks` | Basico | Lista/mapa | Redes del servicio | Conectar servicios entre si | `networks: [backend]` | Puedes definir aliases/IPs |
| `oom_kill_disable` | Super avanzado | Booleano | Desactiva OOM killer | Evitar kill por memoria | `oom_kill_disable: true` | Riesgoso para el host |
| `oom_score_adj` | Super avanzado | Numero | Ajusta prioridad OOM | Influir que proceso muere primero | `oom_score_adj: -500` | Linux avanzado |
| `pid` | Super avanzado | String | Namespace PID | Compartir procesos con host/servicio | `pid: host` | Reduce aislamiento |
| `pids_limit` | Avanzado | Numero | Limite de procesos | Evitar fork bombs | `pids_limit: 100` | Buena practica de hardening |
| `platform` | Intermedio | String | Plataforma/arquitectura | Forzar amd64/arm64 | `platform: linux/amd64` | Util en Apple Silicon |
| `ports` | Basico | Lista | Publicacion de puertos | Acceso desde host/red externa | `ports: ["8080:80"]` | Usa comillas |
| `post_start` | Super avanzado | Lista | Hook despues de iniciar | Ejecutar comandos posteriores al start | `post_start: [{ command: ./init.sh }]` | Requiere soporte Compose moderno |
| `pre_stop` | Super avanzado | Lista | Hook antes de detener | Limpieza antes del stop | `pre_stop: [{ command: ./cleanup.sh }]` | No reemplaza manejo de senales |
| `privileged` | Avanzado | Booleano | Da privilegios amplios | Casos de bajo nivel o Docker-in-Docker | `privileged: true` | Evitar salvo necesidad real |
| `profiles` | Intermedio | Lista | Activa servicio solo con perfil | Servicios opcionales de dev/test | `profiles: [debug]` | Se activa con `--profile debug` |
| `provider` | Super avanzado | Mapa | Servicio gestionado por proveedor externo | Delegar ciclo de vida a provider | `provider: { type: awesomecloud }` | Soporte depende de Compose |
| `pull_policy` | Intermedio | String | Politica de descarga de imagen | Controlar cuando hacer pull | `pull_policy: always` | Acepta `always`, `missing`, `never`, `build`, `daily`, `weekly` y `every_<duration>` |
| `read_only` | Avanzado | Booleano | Filesystem raiz de solo lectura | Hardening de seguridad | `read_only: true` | Combinar con `tmpfs` para rutas temporales |
| `restart` | Basico | String | Politica de reinicio | Alta disponibilidad local | `restart: unless-stopped` | Ver seccion dedicada |
| `runtime` | Avanzado | String | Runtime OCI | Usar runtime alternativo | `runtime: nvidia` | En muchos casos se prefiere `gpus` |
| `scale` | Intermedio | Numero | Numero de contenedores del servicio | Replicas locales simples | `scale: 3` | No usar con `container_name` fijo |
| `secrets` | Avanzado | Lista | Secretos montados | Pasar datos sensibles como archivos | `secrets: [db_password]` | Deben declararse en raiz |
| `security_opt` | Avanzado | Lista | Opciones de seguridad | AppArmor, SELinux, no-new-privileges | `security_opt: ["no-new-privileges:true"]` | Buena practica de hardening |
| `shm_size` | Intermedio | Bytes/string | Tamano de `/dev/shm` | Browsers, Postgres, apps con shared memory | `shm_size: 1gb` | Comun en Playwright/Chrome |
| `stdin_open` | Basico | Booleano | Mantiene STDIN abierto | Contenedores interactivos | `stdin_open: true` | Similar a `docker run -i` |
| `stop_grace_period` | Intermedio | Duracion | Tiempo para apagado limpio | Dar tiempo antes de SIGKILL | `stop_grace_period: 30s` | Util para DBs y workers |
| `stop_signal` | Intermedio | String | Senal de parada | Cambiar senal enviada al proceso | `stop_signal: SIGINT` | Depende de la app |
| `storage_opt` | Super avanzado | Mapa | Opciones del driver storage | Limitar size u opciones de storage | `storage_opt: { size: "1G" }` | Depende del storage driver |
| `sysctls` | Avanzado | Mapa/lista | Parametros kernel namespaced | Tuning de red/kernel permitido | `sysctls: { net.core.somaxconn: 1024 }` | No todos estan permitidos |
| `tmpfs` | Intermedio | Lista | Montajes temporales en RAM | Archivos temporales no persistentes | `tmpfs: [/tmp]` | Se borra al detener |
| `tty` | Basico | Booleano | Asigna TTY | Apps interactivas | `tty: true` | Similar a `docker run -t` |
| `ulimits` | Avanzado | Mapa | Limites de usuario/proceso | Ajustar archivos abiertos/procesos | `ulimits: { nofile: 65535 }` | Comun en Elasticsearch, brokers |
| `use_api_socket` | Super avanzado | Booleano | Monta socket API de Docker | Permitir que el servicio use Docker API | `use_api_socket: true` | Muy sensible en seguridad |
| `user` | Intermedio | String | Usuario dentro del contenedor | Evitar correr como root | `user: "1000:1000"` | Buena practica |
| `userns_mode` | Avanzado | String | Namespace de usuario | Aislamiento de usuarios | `userns_mode: host` | Depende del daemon |
| `uts` | Super avanzado | String | Namespace UTS | Compartir hostname/domainname | `uts: host` | Reduce aislamiento |
| `volumes` | Basico | Lista | Montajes del servicio | Persistencia o archivos locales | `volumes: ["db:/data"]` | Ver seccion volumenes |
| `volumes_from` | Legacy | Lista | Monta volumenes de otro contenedor | Compatibilidad o patrones antiguos | `volumes_from: [db:ro]` | Mejor declarar volumenes explicitamente |
| `working_dir` | Basico | String | Directorio de trabajo | Cambiar carpeta donde corre el comando | `working_dir: /app` | Equivale a `docker run -w` |

---

## 17. Propiedades frecuentes explicadas con mas detalle

### `image`

```yaml
image: postgres:16.2-alpine
```

Define la imagen usada para crear el contenedor.

Buenas practicas:

- Evita `latest` en entornos importantes.
- Usa versiones fijas: `postgres:16.2-alpine`, `redis:7.2-alpine`.
- Actualiza conscientemente con `docker compose pull`.

### `build`

```yaml
build:
  context: .
  dockerfile: Dockerfile
  args:
    NODE_ENV: production
```

Usa `build` cuando tienes un `Dockerfile` propio.

Campos comunes:

| Campo | Que hace |
| --- | --- |
| `context` | Carpeta enviada al build |
| `dockerfile` | Nombre/ruta del Dockerfile |
| `args` | Argumentos disponibles durante build |
| `target` | Stage especifico de un Dockerfile multi-stage |
| `cache_from` | Fuentes de cache |
| `platforms` | Plataformas a construir |

### `depends_on`

Forma simple:

```yaml
depends_on:
  - postgres
```

Forma con condicion:

```yaml
depends_on:
  postgres:
    condition: service_healthy
```

Condiciones comunes:

| Condicion | Significado |
| --- | --- |
| `service_started` | El servicio dependiente ya inicio |
| `service_healthy` | El servicio dependiente esta healthy |
| `service_completed_successfully` | El servicio termino exitosamente |

### `pull_policy`

Controla cuando Compose descarga la imagen.

```yaml
pull_policy: always
```

Valores aceptados mas utiles:

| Valor | Que hace |
| --- | --- |
| `always` | Siempre intenta descargar antes de crear |
| `missing` | Descarga solo si no existe localmente |
| `never` | Nunca descarga; usa solo imagen local |
| `build` | Prioriza construir imagen |
| `daily` | Intenta descargar si la ultima revision fue hace mas de un dia |
| `weekly` | Intenta descargar si la ultima revision fue hace mas de una semana |
| `every_12h` | Intenta descargar segun la duracion indicada; tambien puede ser `every_30m`, `every_1h`, etc. |

Tambien puedes usar:

```bash
docker compose up -d --pull always
```

### `profiles`

Permite servicios opcionales.

```yaml
services:
  app:
    image: mi-app

  adminer:
    image: adminer
    profiles:
      - tools
    ports:
      - "8080:8080"
```

Ejecutar con perfil:

```bash
docker compose --profile tools up -d
```

### `logging`

Evita que los logs crezcan sin limite:

```yaml
logging:
  driver: json-file
  options:
    max-size: "10m"
    max-file: "3"
```

### `security_opt`, `cap_drop`, `read_only`

Ejemplo de endurecimiento:

```yaml
services:
  app:
    image: mi-app
    read_only: true
    tmpfs:
      - /tmp
    cap_drop:
      - ALL
    security_opt:
      - no-new-privileges:true
```

Esto reduce permisos del contenedor. Puede requerir ajustes si la aplicacion necesita escribir archivos.

---

## 18. Secrets y configs

### Secret

```yaml
services:
  app:
    image: mi-app
    secrets:
      - db_password

secrets:
  db_password:
    file: ./secrets/db_password.txt
```

El secret se monta como archivo dentro del contenedor, normalmente en `/run/secrets/db_password`.

### Config

```yaml
services:
  nginx:
    image: nginx:alpine
    configs:
      - source: nginx_conf
        target: /etc/nginx/nginx.conf

configs:
  nginx_conf:
    file: ./nginx.conf
```

Usa `configs` para configuracion no sensible. Usa `secrets` para datos sensibles.

---

## 19. Multiples archivos Compose

Puedes combinar archivos:

```bash
docker compose -f docker-compose.yml -f docker-compose.dev.yml up -d
```

Ejemplo:

`docker-compose.yml`

```yaml
services:
  app:
    image: mi-app
```

`docker-compose.dev.yml`

```yaml
services:
  app:
    volumes:
      - ./src:/app/src
    environment:
      NODE_ENV: development
```

Compose fusiona los archivos. El segundo puede sobrescribir o ampliar valores del primero.

---

## 20. Escalado

Escalar un servicio:

```bash
docker compose up -d --scale api=3
```

O con propiedad:

```yaml
services:
  api:
    image: mi-api
    scale: 3
```

Importante:

- No uses `container_name` si quieres escalar, porque cada replica necesita nombre unico.
- No puedes mapear el mismo puerto fijo del host para varias replicas.
- Para multiples replicas de una API, normalmente usas un reverse proxy o balanceador.

---

## 21. Ejemplos practicos por servicio

### Postgres

```yaml
services:
  postgres:
    image: postgres:16.2-alpine
    restart: unless-stopped
    environment:
      POSTGRES_USER: rdev
      POSTGRES_PASSWORD: rdev
      POSTGRES_DB: postgres
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data

volumes:
  postgres_data:
```

### MySQL

```yaml
services:
  mysql:
    image: mysql:8.4
    restart: unless-stopped
    environment:
      MYSQL_ROOT_PASSWORD: root
      MYSQL_DATABASE: pruebas
      MYSQL_USER: rdev
      MYSQL_PASSWORD: rdev
    ports:
      - "3306:3306"
    volumes:
      - mysql_data:/var/lib/mysql

volumes:
  mysql_data:
```

### Redis

```yaml
services:
  redis:
    image: redis:7.2-alpine
    restart: unless-stopped
    command: "redis-server --appendonly yes"
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data

volumes:
  redis_data:
```

### MongoDB

```yaml
services:
  mongodb:
    image: mongo:7
    restart: unless-stopped
    environment:
      MONGO_INITDB_ROOT_USERNAME: rdev
      MONGO_INITDB_ROOT_PASSWORD: rdev
    ports:
      - "27017:27017"
    volumes:
      - mongo_data:/data/db

volumes:
  mongo_data:
```

### NATS con JetStream

```yaml
services:
  nats:
    image: nats:2.10-alpine
    restart: unless-stopped
    command: ["--jetstream", "--store_dir=/data", "--http_port", "8222"]
    ports:
      - "4222:4222"
      - "8222:8222"
    volumes:
      - nats_data:/data
    healthcheck:
      test: ["CMD", "wget", "--spider", "-q", "http://localhost:8222/healthz"]
      interval: 10s
      timeout: 5s
      retries: 5

volumes:
  nats_data:
```

### MQTT con Eclipse Mosquitto

```yaml
services:
  mqtt:
    image: eclipse-mosquitto:2
    container_name: mqtt-broker
    restart: unless-stopped
    ports:
      - "1883:1883"
    volumes:
      - ./config/mosquitto.conf:/mosquitto/config/mosquitto.conf:ro
      - ./data:/mosquitto/data
      - ./log:/mosquitto/log
    healthcheck:
      test: ["CMD-SHELL", "mosquitto_sub -h 127.0.0.1 -p 1883 -t '$$SYS/broker/uptime' -C 1 >/dev/null 2>&1"]
      interval: 10s
      timeout: 5s
      retries: 5
      start_period: 5s
```

El archivo `config/mosquitto.conf` define el listener MQTT, persistencia, autosave y logs. Las carpetas `data/` y `log/` pueden crearse automaticamente al ejecutar `docker compose up -d`; tambien puedes crearlas antes con `mkdir -p data log`. Este ejemplo permite conexiones anonimas para desarrollo local; para produccion conviene agregar usuarios, ACLs y TLS.

### SQL Server

```yaml
services:
  sqlserver:
    image: mcr.microsoft.com/mssql/server:2022-latest
    restart: unless-stopped
    environment:
      ACCEPT_EULA: "Y"
      MSSQL_SA_PASSWORD: "Cambia_Esta_Clave_123!"
    ports:
      - "1433:1433"
    volumes:
      - sql_data:/var/opt/mssql

volumes:
  sql_data:
```

> Nota: en SQL Server moderno se recomienda `MSSQL_SA_PASSWORD`. Algunas guias antiguas usan `SA_PASSWORD`.

---

## 22. Flujo recomendado para trabajar

Crear o modificar `docker-compose.yml`:

```bash
docker compose config
```

Levantar:

```bash
docker compose up -d
```

Ver estado:

```bash
docker compose ps
```

Ver logs:

```bash
docker compose logs -f
```

Actualizar:

```bash
docker compose pull
docker compose up -d
```

Detener sin borrar:

```bash
docker compose stop
```

Bajar proyecto:

```bash
docker compose down
```

---

## 23. Conectarse a contenedores y ejecutar comandos

Hay tres formas comunes de trabajar desde la terminal con contenedores:

| Forma | Comando base | Cuando usarla |
| --- | --- | --- |
| Entrar a un contenedor existente | `docker compose exec servicio comando` | Cuando el servicio ya esta corriendo |
| Crear un contenedor temporal del mismo servicio | `docker compose run --rm servicio comando` | Para tareas puntuales sin depender del proceso principal |
| Usar una imagen herramienta | `docker run --rm -it ... imagen comando` | Cuando la imagen del servidor no trae el cliente que necesitas |

### Entrar a una shell

Primero levanta el servicio:

```bash
docker compose up -d
```

Luego entra con `sh`:

```bash
docker compose exec postgres sh
```

Si la imagen tiene Bash:

```bash
docker compose exec nombre_servicio bash
```

Muchas imagenes ligeras, especialmente Alpine, no traen `bash`; por eso `sh` suele funcionar mejor.

### Ejecutar un comando sin entrar

```bash
docker compose exec postgres env
docker compose exec postgres ls -la /var/lib/postgresql/data
```

Con `exec`, el comando corre dentro de un contenedor que ya esta iniciado.

### PostgreSQL con `psql`

En el ejemplo de este repo el servicio se llama `postgres` y las credenciales son `rdev/rdev`.

Entrar al cliente:

```bash
docker compose exec postgres psql -U rdev -d postgres
```

Ejecutar una consulta directa:

```bash
docker compose exec postgres psql -U rdev -d postgres -c "SELECT version();"
```

Listar bases de datos:

```bash
docker compose exec postgres psql -U rdev -d postgres -c "\\l"
```

Hacer backup desde la terminal del host:

```bash
docker compose exec postgres pg_dump -U rdev postgres > backup.sql
```

Restaurar backup:

```bash
docker compose exec -T postgres psql -U rdev -d postgres < backup.sql
```

`-T` desactiva el TTY. Es util cuando rediriges archivos con `<` o `>`.

### MySQL con `mysql`

En el ejemplo de este repo el servicio se llama `mysql`.

Entrar como root:

```bash
docker compose exec mysql mysql -uroot -proot
```

Entrar con el usuario de la app:

```bash
docker compose exec mysql mysql -urdev -prdev pruebas
```

Ejecutar una consulta directa:

```bash
docker compose exec mysql mysql -urdev -prdev pruebas -e "SHOW TABLES;"
```

Hacer backup:

```bash
docker compose exec mysql mysqldump -uroot -proot pruebas > backup.sql
```

Restaurar backup:

```bash
docker compose exec -T mysql mysql -uroot -proot pruebas < backup.sql
```

Nota: en comandos `mysql`, `-p` va pegado al password si lo pasas en la misma linea: `-proot`, no `-p root`.

### MongoDB con `mongosh`

En el ejemplo de este repo el servicio se llama `mongodb`.

Entrar a MongoDB:

```bash
docker compose exec mongodb mongosh -u rdev -p rdev --authenticationDatabase admin
```

Ejecutar un comando directo:

```bash
docker compose exec mongodb mongosh -u rdev -p rdev --authenticationDatabase admin --eval "db.adminCommand({ ping: 1 })"
```

Usar una base de datos concreta:

```bash
docker compose exec mongodb mongosh "mongodb://rdev:rdev@localhost:27017/admin"
```

Backup:

```bash
docker compose exec mongodb mongodump -u rdev -p rdev --authenticationDatabase admin --archive > mongo-backup.archive
```

Restore:

```bash
docker compose exec -T mongodb mongorestore -u rdev -p rdev --authenticationDatabase admin --archive < mongo-backup.archive
```

### Redis con `redis-cli`

En el ejemplo de este repo el servicio se llama `redis`.

Entrar al cliente:

```bash
docker compose exec redis redis-cli
```

Probar conexion:

```bash
docker compose exec redis redis-cli ping
```

Guardar y leer una clave:

```bash
docker compose exec redis redis-cli SET saludo "hola"
docker compose exec redis redis-cli GET saludo
```

### NATS con una imagen herramienta

La imagen del servidor NATS esta pensada para correr el servidor. Para usar herramientas de administracion conviene ejecutar un contenedor temporal con `natsio/nats-box`.

Si tu contenedor se llama `nats-server`, puedes compartir su red con:

```bash
docker run --rm -it --network container:nats-server natsio/nats-box:latest
```

Dentro de `nats-box` puedes ejecutar:

```bash
nats --server nats://127.0.0.1:4222 server check
nats --server nats://127.0.0.1:4222 stream ls
nats --server nats://127.0.0.1:4222 pub pruebas "hola desde nats"
nats --server nats://127.0.0.1:4222 sub pruebas
```

Tambien puedes ejecutar un comando directo sin entrar:

```bash
docker run --rm -it --network container:nats-server natsio/nats-box:latest nats --server nats://127.0.0.1:4222 server check
```

Si prefieres conectarte por la red de Compose, busca el nombre de la red:

```bash
docker network ls
```

Y luego usa algo como:

```bash
docker run --rm -it --network nats_default natsio/nats-box:latest nats --server nats://nats:4222 server check
```

El nombre `nats_default` puede cambiar segun el nombre del proyecto o carpeta.

### MQTT con Mosquitto

En el ejemplo de este repo el servicio se llama `mqtt` y el contenedor se llama `mqtt-broker`.

Ver estado del broker:

```bash
docker compose exec mqtt mosquitto_sub -h 127.0.0.1 -p 1883 -t '$SYS/broker/uptime' -C 1
```

Suscribirse a un topic:

```bash
docker compose exec mqtt mosquitto_sub -h 127.0.0.1 -p 1883 -t sensores/temperatura
```

Publicar un mensaje:

```bash
docker compose exec mqtt mosquitto_pub -h 127.0.0.1 -p 1883 -t sensores/temperatura -m "24.5"
```

Desde tu maquina host, si tienes instaladas las herramientas de Mosquitto:

```bash
mosquitto_sub -h localhost -p 1883 -t sensores/temperatura
mosquitto_pub -h localhost -p 1883 -t sensores/temperatura -m "24.5"
```

### SQL Server con `sqlcmd`

Si la imagen de SQL Server trae `sqlcmd`, puedes probar:

```bash
docker compose exec sqlserver /opt/mssql-tools18/bin/sqlcmd -S localhost -U sa -P "Ron@ld1234" -C -Q "SELECT @@VERSION"
```

Si no esta disponible dentro del servidor, usa una imagen herramienta:

```bash
docker run --rm -it --network container:sqlserver mcr.microsoft.com/mssql-tools /opt/mssql-tools/bin/sqlcmd -S localhost -U sa -P "Ron@ld1234" -Q "SELECT @@VERSION"
```

### Usar clientes desde tu maquina host

Si publicaste puertos con `ports`, tambien puedes conectarte desde herramientas instaladas en tu maquina:

| Servicio | Host | Puerto | Ejemplo |
| --- | --- | --- | --- |
| PostgreSQL | `localhost` | `5432` | `psql -h localhost -p 5432 -U rdev -d postgres` |
| MySQL | `localhost` | `3306` | `mysql -h localhost -P 3306 -urdev -prdev pruebas` |
| MongoDB | `localhost` | `27017` | `mongosh "mongodb://rdev:rdev@localhost:27017/admin"` |
| Redis | `localhost` | `6379` | `redis-cli -h localhost -p 6379 ping` |
| MQTT | `localhost` | `1883` | `mosquitto_sub -h localhost -p 1883 -t '$SYS/broker/uptime' -C 1` |
| NATS | `localhost` | `4222` | `nats --server nats://localhost:4222 server check` |
| SQL Server | `localhost` | `1433` | `sqlcmd -S localhost,1433 -U sa -P "password" -C` |

Si el puerto no esta publicado en `ports`, solo otros contenedores de la misma red de Docker podran acceder.

---

## 24. Diferencia entre detener, bajar, eliminar y actualizar

| Concepto | Comando | Resultado |
| --- | --- | --- |
| Detener | `docker compose stop` | El contenedor queda creado pero apagado |
| Iniciar | `docker compose start` | Enciende un contenedor ya creado |
| Reiniciar | `docker compose restart` | Apaga y prende |
| Bajar | `docker compose down` | Detiene y elimina contenedores/redes del proyecto |
| Eliminar | `docker compose rm` | Elimina contenedores detenidos |
| Actualizar | `docker compose pull && docker compose up -d` | Descarga imagen nueva y recrea si corresponde |
| Reconstruir | `docker compose build && docker compose up -d` | Reconstruye imagen local y recrea |

Regla practica:

- Usa `stop` si solo quieres pausar.
- Usa `down` si quieres limpiar contenedores y redes.
- Usa `down -v` solo si quieres borrar tambien datos persistentes.
- Usa `pull` + `up -d` para actualizar imagenes.
- Usa `build` + `up -d` para actualizar imagenes construidas localmente.

---

## 25. Buenas practicas

- Usa versiones fijas de imagenes en lugar de `latest`.
- Usa volumenes nombrados para bases de datos.
- Usa `.env` para valores que cambian por entorno.
- No subas secretos reales a Git.
- Agrega `healthcheck` a bases de datos y servicios criticos.
- Usa `restart: unless-stopped` para servicios persistentes locales.
- Ejecuta `docker compose config` para validar el YAML.
- Evita `container_name` si necesitas escalar.
- Limita logs con `logging.options.max-size`.
- Evita `privileged: true` salvo que sea estrictamente necesario.
- Usa `read_only`, `cap_drop` y `security_opt` cuando busques endurecer seguridad.

---

## 26. Problemas comunes

### El puerto ya esta ocupado

Error tipico:

```text
bind: address already in use
```

Soluciones:

```bash
docker compose ps
docker ps
```

Cambia el puerto del host:

```yaml
ports:
  - "5433:5432"
```

### Un servicio no conecta con otro

Dentro de Compose usa el nombre del servicio:

```text
postgres:5432
redis:6379
```

No uses `localhost` para otro contenedor.

### Cambie variables y no pasa nada

Recrea el servicio:

```bash
docker compose up -d --force-recreate nombre_servicio
```

### Actualice el tag pero sigue igual

Descarga imagen y recrea:

```bash
docker compose pull
docker compose up -d
```

### Perdi datos

Probablemente ejecutaste:

```bash
docker compose down -v
```

Ese comando elimina volumenes del proyecto. Para bases de datos, antes de usarlo haz backup.

---

## 27. Glosario rapido

| Termino | Significado |
| --- | --- |
| Imagen | Plantilla inmutable para crear contenedores |
| Contenedor | Proceso aislado creado desde una imagen |
| Servicio | Definicion Compose que puede crear uno o mas contenedores |
| Proyecto | Grupo de servicios, redes y volumenes Compose |
| Volumen | Almacenamiento persistente administrado por Docker |
| Bind mount | Carpeta/archivo del host montado en el contenedor |
| Red | Espacio de comunicacion entre contenedores |
| Healthcheck | Comando que determina si un contenedor esta saludable |
| Secret | Dato sensible montado de forma controlada |
| Config | Archivo de configuracion no sensible |

---

## 28. Plantilla recomendada

```yaml
name: nombre-proyecto

services:
  app:
    image: mi-app:1.0.0
    restart: unless-stopped
    environment:
      NODE_ENV: production
    ports:
      - "3000:3000"
    depends_on:
      postgres:
        condition: service_healthy
    networks:
      - backend
    logging:
      driver: json-file
      options:
        max-size: "10m"
        max-file: "3"

  postgres:
    image: postgres:16.2-alpine
    restart: unless-stopped
    environment:
      POSTGRES_USER: rdev
      POSTGRES_PASSWORD: rdev
      POSTGRES_DB: app
    volumes:
      - postgres_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U rdev -d app"]
      interval: 10s
      timeout: 5s
      retries: 5
    networks:
      - backend

networks:
  backend:

volumes:
  postgres_data:
```

Esta plantilla cubre lo esencial para una aplicacion comun: servicio principal, base de datos, red, volumen persistente, logs controlados, reinicio automatico y healthcheck.

---

## 29. Graficos y visualizaciones

Estos diagramas usan Mermaid. GitHub, GitLab y muchos visores de Markdown los renderizan como graficos. Si tu editor no los renderiza, igual puedes leer el codigo como una descripcion textual.

### Arquitectura basica de Docker Compose

```mermaid
flowchart TB
  usuario[Usuario o cliente] --> host[Maquina host]
  host --> compose[Docker Compose]
  compose --> app[Servicio app]
  compose --> db[Servicio postgres]
  compose --> cache[Servicio redis]
  app --> red[Red backend]
  db --> red
  cache --> red
  db --> volumen_db[(Volumen postgres_data)]
  cache --> volumen_cache[(Volumen redis_data)]
```

Idea principal:

- Docker Compose lee el YAML.
- Crea servicios como contenedores.
- Conecta servicios a redes.
- Monta volumenes para persistir datos.

### Red interna entre servicios

```mermaid
flowchart LR
  subgraph host[Maquina host]
    navegador[Navegador o cliente HTTP]
    cliente_db[Cliente SQL local]
  end

  subgraph docker[Proyecto Compose]
    subgraph backend[Red backend]
      api[api:3000]
      postgres[postgres:5432]
      redis[redis:6379]
    end
  end

  navegador -->|localhost:3000| api
  cliente_db -->|localhost:5432 si hay ports| postgres
  api -->|postgres:5432| postgres
  api -->|redis:6379| redis
```

Regla mental:

- Desde tu maquina usas `localhost:PUERTO_PUBLICADO`.
- Desde un contenedor hacia otro usas `nombre_servicio:PUERTO_CONTENEDOR`.
- Dentro de la red de Compose, `postgres` y `redis` funcionan como nombres DNS.

### Diferencia entre `ports` y red interna

```mermaid
flowchart TB
  host[Host]
  internet[Herramientas externas]

  subgraph compose[Docker Compose]
    api[api]
    postgres[postgres]
    redis[redis]
  end

  internet -->|publicado con ports 3000:3000| api
  host -->|publicado con ports 5432:5432| postgres
  api -->|red interna postgres:5432| postgres
  api -->|red interna redis:6379| redis
  internet -. sin ports .-> redis
```

Si un servicio no tiene `ports`, no queda publicado hacia tu maquina. Aun asi, otros servicios de la misma red pueden conectarse usando el nombre del servicio.

### Flujo de datos con volumenes

```mermaid
flowchart LR
  compose[docker compose up -d] --> contenedor[Contenedor postgres]
  contenedor --> ruta[/var/lib/postgresql/data/]
  ruta --> volumen[(postgres_data)]
  volumen --> reinicio[Recrear contenedor]
  reinicio --> datos[Datos siguen existiendo]

  downv[docker compose down -v] --> borrar[Borra volumenes del proyecto]
  borrar --> perdida[Datos pueden perderse]
```

Lectura rapida:

- El contenedor puede eliminarse y volver a crearse.
- El volumen nombrado mantiene la informacion.
- `docker compose down -v` elimina los volumenes del proyecto y puede borrar datos.

### Ciclo de vida de comandos Compose

```mermaid
stateDiagram-v2
  [*] --> NoCreado
  NoCreado --> Corriendo: docker compose up -d
  Corriendo --> Detenido: docker compose stop
  Detenido --> Corriendo: docker compose start
  Corriendo --> Corriendo: docker compose restart
  Detenido --> Eliminado: docker compose rm
  Corriendo --> Eliminado: docker compose down
  Eliminado --> Corriendo: docker compose up -d
```

Atajo:

- `stop` apaga sin eliminar.
- `start` prende algo ya creado.
- `rm` elimina contenedores detenidos.
- `down` baja el proyecto y elimina contenedores/redes.
- `up -d` crea o recrea lo necesario.

### Ejemplo visual: API + PostgreSQL + Redis

```mermaid
flowchart TB
  subgraph proyecto[Proyecto app-ejemplo]
    subgraph backend[network: backend]
      api[api\nbuild: ./\nports: 3000:3000]
      postgres[postgres\nimage: postgres:16.2-alpine\nhealthcheck]
      redis[redis\nimage: redis:7.2-alpine\nAOF enabled]
    end

    pgvol[(postgres_data)]
    redisvol[(redis_data)]
  end

  dev[Desarrollador] -->|http://localhost:3000| api
  api -->|DATABASE_URL postgres:5432| postgres
  api -->|REDIS_URL redis:6379| redis
  postgres --> pgvol
  redis --> redisvol
```

Este grafico representa el ejemplo multi-servicio del README: la API se expone al host, pero la comunicacion con base de datos y cache ocurre por red interna.

### Secuencia de arranque con `depends_on` y `healthcheck`

```mermaid
sequenceDiagram
  participant U as Usuario
  participant C as Docker Compose
  participant DB as postgres
  participant R as redis
  participant API as api

  U->>C: docker compose up -d
  C->>DB: crear e iniciar postgres
  C->>R: crear e iniciar redis
  DB-->>C: healthcheck: starting
  DB-->>C: healthcheck: healthy
  R-->>C: service_started
  C->>API: iniciar api
  API->>DB: conectar a postgres:5432
  API->>R: conectar a redis:6379
```

Este flujo ayuda a entender por que `depends_on` con `condition: service_healthy` es util: evita que la API intente conectarse antes de que la base de datos este lista.

### Mapa rapido de carpetas de este repositorio

```mermaid
mindmap
  root((docker))
    README.md
      Guia Docker Compose
      Comandos
      Propiedades
      Graficos
    postgres
      docker-compose.yml
      postgres:16.2-alpine
    mysql
      docker-compose.yml
      mysql
    redis
      docker-compose.yml
      redis:7.2-alpine
    mongo-db
      docker-compose.yml
      mongo
    mqtt
      docker-compose.yml
      eclipse-mosquitto
    nats
      docker-compose.yml
      nats + JetStream
    sqlserver
      docker-compose.yml
      SQL Server 2022
```

Este mapa muestra que cada carpeta tiene su propio Compose. Para levantar uno, entras a su carpeta y ejecutas `docker compose up -d`.
