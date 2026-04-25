# Ejercicio Integrador — Semana 1: Infraestructura de Launch Control

Llevan cinco días de curso. Instalaron Linux, aprendieron a navegar
por el sistema de archivos, crearon estructuras de directorios, movieron
y copiaron archivos, configuraron permisos y escribieron sus primeros
scripts. Todo por separado. Hoy lo juntan.

Prof. Juan Marcelo Gutiérrez Miranda

**Curso IFCD0112 — Ejercicio de consolidación, Semana 1**
**Tiempo estimado:** 4 horas (una jornada completa)
**Requisitos:** Tener Ubuntu funcionando (WSL o VM)

---

# EL CONTEXTO

---

Amtigravity Launch Control es una plataforma que gestiona lanzamientos
espaciales. Piensen en ella como el sistema central que coordina todo:
vehículos, tripulaciones, telemetría, misiones.

El desarrollador anterior renunció sin previo aviso. Dejó el servidor
en un estado lamentable:

- Directorios sin estructura lógica
- Archivos de configuración con permisos abiertos a cualquier usuario
- Scripts que nunca recibieron permiso de ejecución
- Logs mezclados con código fuente
- Sin backups, sin documentación, sin estándares

La dirección técnica les asigna la tarea: **dejar el servidor operativo
en una jornada**. No van a escribir código todavía — eso viene después.
Hoy el trabajo es de infraestructura: organizar, asegurar y automatizar
la base sobre la que se construirá todo lo demás.

Esto no es un simulacro. En el mundo real, cuando entran a trabajar a
una empresa como desarrolladores junior, lo primero que van a encontrar
es un servidor Linux al que acceden por terminal. Sin escritorio, sin
iconos, sin ratón. Solo texto. El que domina la terminal no depende de
nadie para resolver problemas. El que no la domina, espera.

---

## Por qué esto importa (lectura obligatoria)

En cualquier empresa de desarrollo de software:

- Los servidores de producción corren Linux. No Windows, no macOS.
  Linux. Y no tienen interfaz gráfica.
- Los contenedores Docker — que verán en este curso — son Linux
  por dentro. Cada `docker exec` los deja en una terminal.
- Los pipelines de CI/CD (integración continua) ejecutan comandos
  de terminal. Si no saben bash, no pueden debuguear un despliegue
  que falla a las 3 de la mañana.
- Cuando un servidor de producción se cae y hay 10.000 usuarios
  esperando, nadie abre un explorador de archivos. Se conectan por
  SSH y trabajan en terminal.

Lo que hacen hoy no es un ejercicio académico. Es la diferencia entre
ser un desarrollador que puede resolver problemas solo y uno que
necesita que alguien le haga las cosas.

```
╔══════════════════════════════════════════════════════════════════╗
║          DESARROLLADOR QUE DOMINA LA TERMINAL                   ║
╠══════════════════════════════════════════════════════════════════╣
║                                                                  ║
║  → Despliega aplicaciones en servidores remotos                  ║
║  → Diagnostica problemas en producción por SSH                   ║
║  → Automatiza tareas repetitivas con scripts                     ║
║  → Configura permisos y seguridad correctamente                  ║
║  → Trabaja con Docker, Kubernetes, CI/CD sin fricción            ║
║  → No depende de interfaces gráficas que no siempre existen      ║
║                                                                  ║
╠══════════════════════════════════════════════════════════════════╣
║          DESARROLLADOR QUE NO DOMINA LA TERMINAL                 ║
╠══════════════════════════════════════════════════════════════════╣
║                                                                  ║
║  → No puede acceder a servidores de producción                   ║
║  → Espera a que otro le resuelva problemas de infraestructura    ║
║  → Copia y pega comandos sin entender qué hacen                  ║
║  → No puede automatizar nada — todo lo hace a mano               ║
║  → Se bloquea cuando el entorno no tiene GUI                     ║
║                                                                  ║
╚══════════════════════════════════════════════════════════════════╝
```

---

# MISIÓN 1 — DEMOLICIÓN Y RECONOCIMIENTO

**Tiempo:** 30 minutos
**Temas:** navegación, ls, pwd, cd, rutas absolutas y relativas, rm

---

El servidor tiene restos del desarrollador anterior. Antes de construir
hay que limpiar.

## Paso 1: Verificar el estado actual

Abran la terminal. Verifiquen dónde están:

```bash
pwd
```

Deben estar en `/home/TU_USUARIO`. Si no, ejecuten:

```bash
cd ~
```

Verifiquen si existe algún directorio `launch-control` anterior:

```bash
ls -la ~/launch-control 2>/dev/null && echo "EXISTE — hay que limpiarlo" || echo "LIMPIO — no hay nada anterior"
```

> El `2>/dev/null` redirige los errores a la nada. El `&&` ejecuta lo
> siguiente solo si el comando anterior tuvo éxito. El `||` ejecuta lo
> siguiente solo si falló. Van a ver estos operadores constantemente
> en scripts profesionales.

## Paso 2: Si existe, inspeccionar antes de borrar

Nunca borren sin mirar primero. Esto es una regla profesional:

```bash
# Ver qué hay dentro (si existe)
tree ~/launch-control 2>/dev/null || ls -R ~/launch-control 2>/dev/null
```

Si `tree` no está instalado:

```bash
sudo apt install tree -y
```

## Paso 3: Demolición controlada

Una vez que verificaron qué hay dentro y confirmaron que no hay nada
que necesiten conservar:

```bash
rm -ri ~/launch-control
```

> Usan `-ri` y no `-rf`. La `i` pide confirmación archivo por archivo.
> En un servidor de producción, `rm -rf` sin pensar ha destruido
> empresas enteras. Literalmente. Búsquenlo: "rm -rf horror stories".
> La `i` es el cinturón de seguridad.

Si no existe nada anterior, salten al paso 4.

## Paso 4: Limpiar también restos de ejercicios anteriores

```bash
rm -ri ~/mision-espacial 2>/dev/null
rm -ri ~/mision-espacial-backup 2>/dev/null
rm -ri ~/diagnostico 2>/dev/null
rm -ri ~/artemis-iii 2>/dev/null
```

## Verificación de Misión 1

```bash
ls ~ | grep -E "launch|mision|artemis|diagnostico"
```

Si no devuelve nada: está limpio. Si devuelve algo, repitan la
limpieza del directorio que quedó.

---

# MISIÓN 2 — CONSTRUIR LA ESTRUCTURA

**Tiempo:** 45 minutos
**Temas:** mkdir, mkdir -p, touch, echo, >, >>, redirección, cat

---

La dirección técnica entregó el diagrama de la infraestructura que
necesitan. Esta es la estructura oficial del proyecto Launch Control:

```
~/launch-control/
├── config/
│   ├── database.conf
│   ├── api.conf
│   └── secrets/
│       ├── db_password.txt
│       └── api_keys.txt
├── data/
│   ├── vehicles/
│   │   ├── falcon9.dat
│   │   ├── starship.dat
│   │   └── electron.dat
│   ├── crews/
│   │   ├── crew_alpha.dat
│   │   └── crew_beta.dat
│   └── missions/
│       ├── mission_001.log
│       ├── mission_002.log
│       └── mission_003.log
├── logs/
│   ├── system.log
│   ├── access.log
│   └── errors.log
├── scripts/
│   ├── backup.sh
│   ├── deploy.sh
│   ├── health_check.sh
│   └── cleanup.sh
├── docs/
│   ├── README.md
│   ├── CHANGELOG.md
│   └── architecture/
│       └── system_overview.txt
├── tmp/
│   └── .gitkeep
└── .env
```

## Paso 1: Crear todos los directorios de una vez

Un solo comando. La opción `-p` crea toda la cadena de directorios
que no existan:

```bash
mkdir -p ~/launch-control/{config/secrets,data/{vehicles,crews,missions},logs,scripts,docs/architecture,tmp}
```

Verifiquen:

```bash
tree ~/launch-control
```

Salida esperada:

```
/home/TU_USUARIO/launch-control
├── config
│   └── secrets
├── data
│   ├── crews
│   ├── missions
│   └── vehicles
├── docs
│   └── architecture
├── logs
├── scripts
└── tmp

11 directories, 0 files
```

> 11 directorios, 0 archivos. La estructura existe pero está vacía.
> Eso cambia ahora.

## Paso 2: Crear archivos de configuración

```bash
cd ~/launch-control

# Configuración de base de datos
cat > config/database.conf << 'EOF'
# Launch Control — Configuración de Base de Datos
# Entorno: desarrollo
# Última modificación: abril 2026

DB_HOST=localhost
DB_PORT=5432
DB_NAME=launch_control
DB_USER=lc_admin
DB_POOL_SIZE=10
DB_TIMEOUT=30
EOF

# Configuración de la API
cat > config/api.conf << 'EOF'
# Launch Control — Configuración de API REST
# Entorno: desarrollo

API_HOST=0.0.0.0
API_PORT=8080
API_VERSION=v1
API_RATE_LIMIT=100
API_CORS_ORIGINS=http://localhost:3000
DEBUG=true
EOF
```

Verifiquen que el contenido es correcto:

```bash
cat config/database.conf
cat config/api.conf
```

## Paso 3: Crear archivos secretos

Estos archivos contienen credenciales. En un proyecto real NUNCA se
suben a Git (por eso existe `.gitignore`). Los crean así:

```bash
echo "lc_db_password_2026_dev" > config/secrets/db_password.txt
echo "sk-launch-control-api-dev-x8k2m9n4" > config/secrets/api_keys.txt
```

## Paso 4: Crear datos de vehículos

Cada archivo contiene la especificación de un vehículo:

```bash
cat > data/vehicles/falcon9.dat << 'EOF'
VEHICLE_ID=VH-001
NAME=Falcon 9
MANUFACTURER=SpaceX
PAYLOAD_LEO_KG=22800
PAYLOAD_GTO_KG=8300
HEIGHT_M=70
STATUS=operational
FIRST_FLIGHT=2010-06-04
TOTAL_LAUNCHES=300
EOF

cat > data/vehicles/starship.dat << 'EOF'
VEHICLE_ID=VH-002
NAME=Starship
MANUFACTURER=SpaceX
PAYLOAD_LEO_KG=150000
PAYLOAD_GTO_KG=21000
HEIGHT_M=121
STATUS=testing
FIRST_FLIGHT=2023-04-20
TOTAL_LAUNCHES=12
EOF

cat > data/vehicles/electron.dat << 'EOF'
VEHICLE_ID=VH-003
NAME=Electron
MANUFACTURER=Rocket Lab
PAYLOAD_LEO_KG=300
PAYLOAD_GTO_KG=0
HEIGHT_M=18
STATUS=operational
FIRST_FLIGHT=2017-05-25
TOTAL_LAUNCHES=45
EOF
```

## Paso 5: Crear datos de tripulaciones

```bash
cat > data/crews/crew_alpha.dat << 'EOF'
CREW_ID=CR-001
NAME=Alpha
COMMANDER=María Torres
PILOT=Ahmed Nazari
SPECIALIST_1=Kenji Watanabe
SPECIALIST_2=Lucía Mendoza
STATUS=active
MISSION_ASSIGNED=MS-001
CLEARANCE_LEVEL=top_secret
EOF

cat > data/crews/crew_beta.dat << 'EOF'
CREW_ID=CR-002
NAME=Beta
COMMANDER=David Kim
PILOT=Elena Volkov
SPECIALIST_1=Carlos Reyes
SPECIALIST_2=Priya Sharma
STATUS=standby
MISSION_ASSIGNED=none
CLEARANCE_LEVEL=secret
EOF
```

## Paso 6: Crear logs de misiones

```bash
cat > data/missions/mission_001.log << 'EOF'
[2026-03-15 06:00:00] MISSION MS-001 INITIALIZED
[2026-03-15 06:15:00] VEHICLE VH-001 ASSIGNED
[2026-03-15 06:30:00] CREW CR-001 BOARDED
[2026-03-15 07:00:00] PREFLIGHT CHECK: PASSED
[2026-03-15 07:30:00] LAUNCH AUTHORIZED
[2026-03-15 07:31:00] IGNITION SEQUENCE STARTED
[2026-03-15 07:31:05] LIFTOFF CONFIRMED
[2026-03-15 07:33:00] MAX-Q PASSED
[2026-03-15 07:40:00] STAGE SEPARATION CONFIRMED
[2026-03-15 08:15:00] ORBIT INSERTION CONFIRMED
[2026-03-15 08:20:00] MISSION STATUS: NOMINAL
EOF

cat > data/missions/mission_002.log << 'EOF'
[2026-04-01 08:00:00] MISSION MS-002 INITIALIZED
[2026-04-01 08:15:00] VEHICLE VH-002 ASSIGNED
[2026-04-01 08:30:00] CREW: UNMANNED
[2026-04-01 09:00:00] PREFLIGHT CHECK: PASSED
[2026-04-01 09:30:00] LAUNCH AUTHORIZED
[2026-04-01 09:31:00] IGNITION SEQUENCE STARTED
[2026-04-01 09:31:05] LIFTOFF CONFIRMED
[2026-04-01 09:33:00] MAX-Q PASSED
[2026-04-01 09:35:22] ANOMALY DETECTED: HYDRAULIC PRESSURE DROP
[2026-04-01 09:35:30] FLIGHT TERMINATION SYSTEM ACTIVATED
[2026-04-01 09:35:35] MISSION STATUS: ABORTED
EOF

cat > data/missions/mission_003.log << 'EOF'
[2026-04-10 14:00:00] MISSION MS-003 INITIALIZED
[2026-04-10 14:15:00] VEHICLE VH-003 ASSIGNED
[2026-04-10 14:30:00] CREW: UNMANNED
[2026-04-10 14:45:00] PREFLIGHT CHECK: WARNING — WIND SPEED HIGH
[2026-04-10 15:00:00] LAUNCH DELAYED: WEATHER HOLD
[2026-04-10 16:30:00] WEATHER CLEARED
[2026-04-10 17:00:00] LAUNCH AUTHORIZED
[2026-04-10 17:01:00] IGNITION SEQUENCE STARTED
[2026-04-10 17:01:05] LIFTOFF CONFIRMED
[2026-04-10 17:15:00] ORBIT INSERTION CONFIRMED
[2026-04-10 17:20:00] PAYLOAD DEPLOYED
[2026-04-10 17:25:00] MISSION STATUS: SUCCESS
EOF
```

## Paso 7: Crear logs del sistema

```bash
cat > logs/system.log << 'EOF'
[2026-04-15 00:00:01] SYSTEM BOOT SEQUENCE INITIATED
[2026-04-15 00:00:05] LOADING KERNEL MODULES
[2026-04-15 00:00:12] NETWORK INTERFACES UP
[2026-04-15 00:00:15] DATABASE SERVICE STARTED
[2026-04-15 00:00:18] API SERVICE STARTED ON PORT 8080
[2026-04-15 00:00:20] MONITORING AGENT ACTIVE
[2026-04-15 00:01:00] HEALTH CHECK: ALL SYSTEMS NOMINAL
EOF

cat > logs/access.log << 'EOF'
[2026-04-15 08:30:15] GET /api/v1/vehicles 200 — user:admin — 45ms
[2026-04-15 08:30:22] GET /api/v1/missions 200 — user:admin — 38ms
[2026-04-15 08:31:05] POST /api/v1/missions 201 — user:admin — 120ms
[2026-04-15 08:35:10] GET /api/v1/crews 200 — user:operator — 42ms
[2026-04-15 08:40:00] DELETE /api/v1/missions/999 404 — user:guest — 5ms
[2026-04-15 08:41:15] POST /api/v1/auth/login 401 — user:unknown — 3ms
[2026-04-15 08:41:16] POST /api/v1/auth/login 401 — user:unknown — 2ms
[2026-04-15 08:41:17] POST /api/v1/auth/login 401 — user:unknown — 2ms
EOF

cat > logs/errors.log << 'EOF'
[2026-04-15 08:40:00] ERROR: Resource not found — DELETE /api/v1/missions/999
[2026-04-15 08:41:15] WARN: Failed login attempt from 192.168.1.100
[2026-04-15 08:41:16] WARN: Failed login attempt from 192.168.1.100
[2026-04-15 08:41:17] CRITICAL: Brute force detected from 192.168.1.100 — IP blocked
EOF
```

## Paso 8: Crear documentación

```bash
cat > docs/README.md << 'EOF'
# Amtigravity Launch Control

Plataforma centralizada de gestión de lanzamientos espaciales.

## Componentes
- **API REST** — interfaz de comunicación (puerto 8080)
- **Base de datos** — PostgreSQL 16 (puerto 5432)
- **Sistema de logs** — registro de eventos y auditoría
- **Scripts de operaciones** — automatización de tareas

## Estructura del proyecto
Ejecutar `tree ~/launch-control` para ver la estructura completa.

## Equipo
Proyecto del curso IFCD0112 — Programación con Lenguajes OO y BBDD Relacionales
EOF

cat > docs/CHANGELOG.md << 'EOF'
# Changelog — Launch Control

## [0.1.0] — 2026-04-15
### Creado
- Estructura inicial del proyecto
- Archivos de configuración (database.conf, api.conf)
- Datos de vehículos (Falcon 9, Starship, Electron)
- Datos de tripulaciones (Alpha, Beta)
- Logs de misiones (001, 002, 003)
- Scripts de operaciones (backup, deploy, health_check, cleanup)
- Documentación base (README, CHANGELOG)
EOF

cat > docs/architecture/system_overview.txt << 'EOF'
LAUNCH CONTROL — VISTA GENERAL DEL SISTEMA

  ┌─────────────┐     ┌─────────────┐     ┌─────────────┐
  │   CLIENTE    │────→│   API REST  │────→│  BASE DATOS │
  │  (frontend)  │←────│  (port 8080)│←────│ (PostgreSQL) │
  └─────────────┘     └──────┬──────┘     └─────────────┘
                             │
                     ┌───────┴───────┐
                     │  LOGS / AUDIT │
                     │  (filesystem) │
                     └───────────────┘

Flujo de datos:
1. El cliente envía peticiones HTTP al API
2. El API valida, procesa y consulta la base de datos
3. Cada operación queda registrada en los logs
4. Los scripts de operaciones mantienen el sistema saludable
EOF
```

## Paso 9: Crear el archivo de entorno y el placeholder de tmp

```bash
cat > .env << 'EOF'
# Variables de entorno — Launch Control
# ESTE ARCHIVO NUNCA SE SUBE A GIT
NODE_ENV=development
DATABASE_URL=postgresql://lc_admin:lc_db_password_2026_dev@localhost:5432/launch_control
API_SECRET=jwt-secret-dev-only-change-in-production
LOG_LEVEL=debug
EOF

# .gitkeep es una convención para que Git rastree directorios vacíos
touch tmp/.gitkeep
```

## Paso 10: Crear los scripts (vacíos por ahora — los llenan en Misión 5)

```bash
touch scripts/backup.sh scripts/deploy.sh scripts/health_check.sh scripts/cleanup.sh
```

## Verificación de Misión 2

```bash
tree ~/launch-control
```

Salida esperada:

```
/home/TU_USUARIO/launch-control
├── config
│   ├── api.conf
│   ├── database.conf
│   └── secrets
│       ├── api_keys.txt
│       └── db_password.txt
├── data
│   ├── crews
│   │   ├── crew_alpha.dat
│   │   └── crew_beta.dat
│   ├── missions
│   │   ├── mission_001.log
│   │   ├── mission_002.log
│   │   └── mission_003.log
│   └── vehicles
│       ├── electron.dat
│       ├── falcon9.dat
│       └── starship.dat
├── docs
│   ├── architecture
│   │   └── system_overview.txt
│   ├── CHANGELOG.md
│   └── README.md
├── logs
│   ├── access.log
│   ├── errors.log
│   └── system.log
├── scripts
│   ├── backup.sh
│   ├── cleanup.sh
│   ├── deploy.sh
│   └── health_check.sh
├── tmp
│   └── .gitkeep
└── .env

11 directories, 22 files
```

Cuenten: 11 directorios, 22 archivos. Si tienen menos, les falta algo.

```bash
find ~/launch-control -type f | wc -l
```

Debe dar **22**.

---

# MISIÓN 3 — OPERACIONES DE DATOS

**Tiempo:** 45 minutos
**Temas:** cp, cp -r, mv, rm, wildcards, find, grep, pipes

---

El sistema está montado pero necesitan reorganizar datos y extraer
información. Estas son las operaciones que la dirección técnica
necesita antes del mediodía.

## Tarea 3.1: Backup de datos críticos

Creen un backup completo de la carpeta `data/`:

```bash
cd ~/launch-control
cp -r data/ data_backup/
```

Verifiquen que la copia es idéntica:

```bash
diff <(tree data/) <(tree data_backup/)
```

> `diff` compara dos entradas. Si no devuelve nada, son idénticas.
> La sintaxis `<(comando)` se llama "sustitución de proceso" — ejecuta
> el comando y lo trata como si fuera un archivo. Avanzado, pero útil.

## Tarea 3.2: Reorganizar misiones por estado

Las misiones tienen distintos estados. La dirección quiere separarlas:

```bash
mkdir -p data/missions/{completed,failed,delayed}
```

Ahora muevan cada misión a su carpeta correspondiente. Para saber el
estado de cada una, revisen la última línea de cada log:

```bash
tail -1 data/missions/mission_001.log
tail -1 data/missions/mission_002.log
tail -1 data/missions/mission_003.log
```

Con esa información, muevan cada archivo:

```bash
# mission_001: NOMINAL → completed
mv data/missions/mission_001.log data/missions/completed/

# mission_002: ABORTED → failed
mv data/missions/mission_002.log data/missions/failed/

# mission_003: SUCCESS → completed
mv data/missions/mission_003.log data/missions/completed/
```

Verifiquen:

```bash
tree data/missions/
```

```
data/missions/
├── completed
│   ├── mission_001.log
│   └── mission_003.log
├── delayed
└── failed
    └── mission_002.log

3 directories, 3 files
```

## Tarea 3.3: Buscar información con grep

La dirección necesita respuestas rápidas. Usen `grep` para encontrar
la información sin abrir cada archivo manualmente.

**Pregunta 1:** ¿Qué vehículo tiene más capacidad de carga?

```bash
grep "PAYLOAD_LEO_KG" data/vehicles/*.dat
```

```
data/vehicles/electron.dat:PAYLOAD_LEO_KG=300
data/vehicles/falcon9.dat:PAYLOAD_LEO_KG=22800
data/vehicles/starship.dat:PAYLOAD_LEO_KG=150000
```

Starship: 150.000 kg. No hace falta abrir tres archivos.

**Pregunta 2:** ¿Cuántos intentos de login fallidos hubo?

```bash
grep -c "Failed login" logs/errors.log
```

```
2
```

> `-c` cuenta las coincidencias en vez de mostrarlas.

**Pregunta 3:** ¿Desde qué IP atacaron?

```bash
grep "Brute force" logs/errors.log
```

```
[2026-04-15 08:41:17] CRITICAL: Brute force detected from 192.168.1.100 — IP blocked
```

**Pregunta 4:** ¿Qué misiones tuvieron anomalías?

```bash
grep -rl "ANOMALY\|ABORT\|CRITICAL\|WARNING" data/missions/
```

> `-r` busca recursivamente en subdirectorios. `-l` muestra solo
> nombres de archivo, no las líneas completas.

**Pregunta 5:** ¿Quién es el comandante de la tripulación Alpha?

```bash
grep "COMMANDER" data/crews/crew_alpha.dat
```

## Tarea 3.4: Operaciones con wildcards

Cambien la extensión de todos los archivos de vehículos de `.dat` a
`.cfg`:

```bash
cd ~/launch-control
for f in data/vehicles/*.dat; do
    mv "$f" "${f%.dat}.cfg"
done
```

> Esto usa un bucle `for`. La expresión `${f%.dat}` elimina el sufijo
> `.dat` del nombre. Es sintaxis de bash — la van a ver mucho.

Verifiquen:

```bash
ls data/vehicles/
```

```
electron.cfg  falcon9.cfg  starship.cfg
```

Ahora reviertan el cambio (los datos originales usaban `.dat`):

```bash
for f in data/vehicles/*.cfg; do
    mv "$f" "${f%.cfg}.dat"
done
```

## Tarea 3.5: Generar un inventario

Creen un archivo que liste todo el contenido del proyecto:

```bash
cd ~/launch-control
find . -type f | sort > docs/inventory.txt
cat docs/inventory.txt
```

Verifiquen cuántos archivos tienen:

```bash
wc -l docs/inventory.txt
```

Debe dar **23** (los 22 originales más el propio `inventory.txt`).

## Verificación de Misión 3

```bash
# Las misiones están reorganizadas
ls data/missions/completed/ data/missions/failed/

# El inventario existe
test -f docs/inventory.txt && echo "INVENTARIO OK" || echo "FALTA INVENTARIO"

# El backup existe
test -d data_backup && echo "BACKUP OK" || echo "FALTA BACKUP"
```

---

# MISIÓN 4 — SEGURIDAD Y PERMISOS

**Tiempo:** 45 minutos
**Temas:** chmod, notación octal, ls -l, usuarios, grupos, sudo

---

El servidor tiene archivos confidenciales abiertos a todo el mundo.
Un auditor de seguridad va a revisar los permisos. Si los archivos
secretos son legibles por cualquier usuario, la auditoría falla.

## Conceptos clave antes de empezar

Cada archivo en Linux tiene tres niveles de permisos:

```
┌─────────────────────────────────────────────────────────┐
│                    PERMISOS LINUX                        │
├───────────┬───────────┬───────────┬─────────────────────┤
│           │  DUEÑO    │  GRUPO    │  OTROS              │
│           │  (user)   │  (group)  │  (others)           │
├───────────┼───────────┼───────────┼─────────────────────┤
│  Leer (r) │    4      │    4      │    4                │
│  Escribir │    2      │    2      │    2                │
│  Ejecutar │    1      │    1      │    1                │
├───────────┼───────────┼───────────┼─────────────────────┤
│  Total    │  0-7      │  0-7      │  0-7                │
└───────────┴───────────┴───────────┴─────────────────────┘

Ejemplos:
  755 = rwxr-xr-x → dueño: todo, grupo: leer+ejecutar, otros: leer+ejecutar
  644 = rw-r--r-- → dueño: leer+escribir, grupo: leer, otros: leer
  600 = rw------- → dueño: leer+escribir, nadie más
  700 = rwx------ → dueño: todo, nadie más
```

## Paso 1: Ver los permisos actuales

```bash
cd ~/launch-control
ls -la config/secrets/
```

Probablemente ven algo como:

```
-rw-r--r-- 1 usuario usuario  24 abr 15 10:30 db_password.txt
-rw-r--r-- 1 usuario usuario  38 abr 15 10:30 api_keys.txt
```

Eso es `644`: **cualquier usuario del sistema puede leer las
contraseñas**. Eso es un problema de seguridad grave.

## Paso 2: Asegurar archivos secretos

Los archivos con credenciales solo deben ser legibles por su dueño:

```bash
chmod 600 config/secrets/db_password.txt
chmod 600 config/secrets/api_keys.txt
chmod 600 .env
```

Verifiquen:

```bash
ls -la config/secrets/
ls -la .env
```

```
-rw------- 1 usuario usuario  24 abr 15 10:30 db_password.txt
-rw------- 1 usuario usuario  38 abr 15 10:30 api_keys.txt
```

Ahora solo el dueño puede leer y escribir. Nadie más.

## Paso 3: Proteger el directorio de secretos

```bash
chmod 700 config/secrets/
```

> 700 en un directorio significa: solo el dueño puede entrar, listar
> contenido y crear archivos dentro. Nadie más puede ni siquiera ver
> qué archivos hay ahí.

## Paso 4: Configurar permisos correctos para todo el proyecto

Esta es la política de seguridad que la dirección técnica definió:

```
╔═══════════════════════════════════════════════════════════════╗
║              POLÍTICA DE PERMISOS — LAUNCH CONTROL            ║
╠══════════════════════════════════╦════════════╦═══════════════╣
║  Tipo de archivo                 ║  Permiso   ║  Notación     ║
╠══════════════════════════════════╬════════════╬═══════════════╣
║  Secretos (.env, passwords)      ║  600       ║  rw-------    ║
║  Directorio de secretos          ║  700       ║  rwx------    ║
║  Archivos de configuración       ║  640       ║  rw-r-----    ║
║  Datos de tripulaciones          ║  640       ║  rw-r-----    ║
║  Datos de vehículos              ║  644       ║  rw-r--r--    ║
║  Logs del sistema                ║  640       ║  rw-r-----    ║
║  Logs de misiones                ║  644       ║  rw-r--r--    ║
║  Scripts ejecutables             ║  755       ║  rwxr-xr-x    ║
║  Documentación                   ║  644       ║  rw-r--r--    ║
║  Directorios regulares           ║  755       ║  rwxr-xr-x    ║
╚══════════════════════════════════╩════════════╩═══════════════╝
```

Ahora apliquen esa política completa. Esto es lo que tienen que ejecutar:

```bash
cd ~/launch-control

# Configuración: legible por grupo, no por otros
chmod 640 config/database.conf
chmod 640 config/api.conf

# Tripulaciones: datos sensibles, solo dueño y grupo
chmod 640 data/crews/crew_alpha.dat
chmod 640 data/crews/crew_beta.dat

# Vehículos: datos públicos
chmod 644 data/vehicles/*.dat

# Logs del sistema: sensibles
chmod 640 logs/system.log
chmod 640 logs/access.log
chmod 640 logs/errors.log

# Logs de misiones: públicos
find data/missions/ -name "*.log" -exec chmod 644 {} \;

# Documentación: pública
chmod 644 docs/README.md
chmod 644 docs/CHANGELOG.md
chmod 644 docs/architecture/system_overview.txt
chmod 644 docs/inventory.txt

# Scripts: ejecutables
chmod 755 scripts/*.sh
```

> El comando `find ... -exec chmod 644 {} \;` busca todos los `.log`
> dentro de `data/missions/` (incluyendo subdirectorios) y les aplica
> `chmod 644`. El `{}` se reemplaza por cada archivo encontrado.

## Paso 5: Verificar la auditoría completa

Ejecuten este comando que muestra los permisos de todos los archivos:

```bash
find ~/launch-control -type f -exec ls -l {} \; | awk '{print $1, $NF}' | sort
```

> Esto combina `find` (buscar archivos), `ls -l` (ver permisos) y
> `awk` (extraer solo la columna de permisos y el nombre). El `sort`
> los ordena para revisarlos rápido.

Revisen que:
- Los archivos `600` no tengan permisos para grupo ni otros
- Los scripts tengan la `x` de ejecución
- Los archivos de documentación sean legibles por todos

## Verificación de Misión 4

```bash
# Secretos protegidos
stat -c "%a %n" config/secrets/* .env 2>/dev/null

# Scripts ejecutables
stat -c "%a %n" scripts/*.sh

# Configuración correcta
stat -c "%a %n" config/*.conf
```

Salida esperada (los números de la izquierda son los permisos en octal):

```
600 config/secrets/api_keys.txt
600 config/secrets/db_password.txt
600 .env
755 scripts/backup.sh
755 scripts/cleanup.sh
755 scripts/deploy.sh
755 scripts/health_check.sh
640 config/api.conf
640 config/database.conf
```

---

# MISIÓN 5 — AUTOMATIZACIÓN

**Tiempo:** 45 minutos
**Temas:** scripts bash, variables, condicionales, ejecución

---

Los scripts que crearon están vacíos. Es hora de darles vida. Cada
script automatiza una tarea que en una empresa real se ejecuta
diariamente — o cada hora.

## Script 1: health_check.sh

Este script verifica que todos los componentes del proyecto existen
y tienen los permisos correctos.

```bash
cat > ~/launch-control/scripts/health_check.sh << 'SCRIPT'
#!/bin/bash
# ═══════════════════════════════════════════
# Launch Control — Health Check
# Verifica la integridad del proyecto
# ═══════════════════════════════════════════

PROJECT_DIR="$HOME/launch-control"
ERRORS=0
CHECKS=0

echo "╔═══════════════════════════════════════╗"
echo "║   LAUNCH CONTROL — HEALTH CHECK      ║"
echo "║   $(date '+%Y-%m-%d %H:%M:%S')              ║"
echo "╚═══════════════════════════════════════╝"
echo ""

# --- Verificar directorios críticos ---
echo ">> Verificando directorios..."
for DIR in config data logs scripts docs tmp config/secrets data/vehicles data/crews data/missions; do
    CHECKS=$((CHECKS + 1))
    if [ -d "$PROJECT_DIR/$DIR" ]; then
        echo "   [OK] $DIR"
    else
        echo "   [FALLO] $DIR NO EXISTE"
        ERRORS=$((ERRORS + 1))
    fi
done
echo ""

# --- Verificar archivos críticos ---
echo ">> Verificando archivos de configuración..."
for FILE in config/database.conf config/api.conf .env; do
    CHECKS=$((CHECKS + 1))
    if [ -f "$PROJECT_DIR/$FILE" ]; then
        echo "   [OK] $FILE"
    else
        echo "   [FALLO] $FILE NO EXISTE"
        ERRORS=$((ERRORS + 1))
    fi
done
echo ""

# --- Verificar permisos de secretos ---
echo ">> Verificando seguridad de archivos secretos..."
for SECRET in config/secrets/db_password.txt config/secrets/api_keys.txt .env; do
    CHECKS=$((CHECKS + 1))
    if [ -f "$PROJECT_DIR/$SECRET" ]; then
        PERMS=$(stat -c "%a" "$PROJECT_DIR/$SECRET" 2>/dev/null || stat -f "%Lp" "$PROJECT_DIR/$SECRET" 2>/dev/null)
        if [ "$PERMS" = "600" ]; then
            echo "   [OK] $SECRET (permisos: $PERMS)"
        else
            echo "   [ALERTA] $SECRET tiene permisos $PERMS — debería ser 600"
            ERRORS=$((ERRORS + 1))
        fi
    fi
done
echo ""

# --- Verificar que los scripts son ejecutables ---
echo ">> Verificando scripts ejecutables..."
for SCRIPT_FILE in scripts/backup.sh scripts/deploy.sh scripts/health_check.sh scripts/cleanup.sh; do
    CHECKS=$((CHECKS + 1))
    if [ -x "$PROJECT_DIR/$SCRIPT_FILE" ]; then
        echo "   [OK] $SCRIPT_FILE es ejecutable"
    else
        echo "   [ALERTA] $SCRIPT_FILE NO es ejecutable"
        ERRORS=$((ERRORS + 1))
    fi
done
echo ""

# --- Resumen ---
echo "═══════════════════════════════════════"
echo "RESULTADO: $CHECKS verificaciones, $ERRORS errores"
if [ $ERRORS -eq 0 ]; then
    echo "ESTADO: TODOS LOS SISTEMAS OPERATIVOS"
else
    echo "ESTADO: $ERRORS PROBLEMAS DETECTADOS"
fi
echo "═══════════════════════════════════════"
SCRIPT
```

Denle permisos y ejecútenlo:

```bash
chmod 755 ~/launch-control/scripts/health_check.sh
~/launch-control/scripts/health_check.sh
```

Si todo está bien, deben ver cero errores. Si hay errores, el script
les dice exactamente qué falta — corrijan y vuelvan a ejecutar.

## Script 2: backup.sh

Este script crea un backup con fecha y hora en el nombre:

```bash
cat > ~/launch-control/scripts/backup.sh << 'SCRIPT'
#!/bin/bash
# ═══════════════════════════════════════════
# Launch Control — Backup automático
# ═══════════════════════════════════════════

PROJECT_DIR="$HOME/launch-control"
BACKUP_NAME="launch-control-backup-$(date '+%Y%m%d_%H%M%S')"
BACKUP_DIR="$HOME/$BACKUP_NAME"

echo "Iniciando backup de Launch Control..."
echo "Origen:  $PROJECT_DIR"
echo "Destino: $BACKUP_DIR"
echo ""

cp -r "$PROJECT_DIR" "$BACKUP_DIR"

if [ $? -eq 0 ]; then
    TOTAL_FILES=$(find "$BACKUP_DIR" -type f | wc -l)
    TOTAL_DIRS=$(find "$BACKUP_DIR" -type d | wc -l)
    echo "Backup completado."
    echo "  Archivos copiados: $TOTAL_FILES"
    echo "  Directorios: $TOTAL_DIRS"
    echo "  Ubicación: $BACKUP_DIR"
else
    echo "ERROR: el backup falló."
    exit 1
fi
SCRIPT

chmod 755 ~/launch-control/scripts/backup.sh
```

Ejecútenlo:

```bash
~/launch-control/scripts/backup.sh
```

Verifiquen que el backup existe:

```bash
ls -d ~/launch-control-backup-*
```

## Script 3: cleanup.sh

Este script elimina archivos temporales y logs antiguos:

```bash
cat > ~/launch-control/scripts/cleanup.sh << 'SCRIPT'
#!/bin/bash
# ═══════════════════════════════════════════
# Launch Control — Limpieza de temporales
# ═══════════════════════════════════════════

PROJECT_DIR="$HOME/launch-control"

echo "Iniciando limpieza de Launch Control..."
echo ""

# Contar archivos en tmp/
TMP_COUNT=$(find "$PROJECT_DIR/tmp" -type f ! -name ".gitkeep" | wc -l)
echo "Archivos temporales encontrados: $TMP_COUNT"

if [ $TMP_COUNT -gt 0 ]; then
    find "$PROJECT_DIR/tmp" -type f ! -name ".gitkeep" -delete
    echo "Temporales eliminados."
else
    echo "No hay temporales que limpiar."
fi

# Mostrar tamaño de logs
echo ""
echo "Tamaño de archivos de log:"
du -sh "$PROJECT_DIR/logs/"* 2>/dev/null
echo ""

TOTAL_SIZE=$(du -sh "$PROJECT_DIR" | awk '{print $1}')
echo "Tamaño total del proyecto: $TOTAL_SIZE"
echo "Limpieza completada."
SCRIPT

chmod 755 ~/launch-control/scripts/cleanup.sh
```

## Script 4: deploy.sh

Este script simula un despliegue — verifica prerrequisitos antes
de "lanzar" la aplicación:

```bash
cat > ~/launch-control/scripts/deploy.sh << 'SCRIPT'
#!/bin/bash
# ═══════════════════════════════════════════
# Launch Control — Deploy (simulación)
# ═══════════════════════════════════════════

PROJECT_DIR="$HOME/launch-control"
DEPLOY_OK=true

echo "╔═══════════════════════════════════════╗"
echo "║   LAUNCH CONTROL — DEPLOY            ║"
echo "║   $(date '+%Y-%m-%d %H:%M:%S')              ║"
echo "╚═══════════════════════════════════════╝"
echo ""

# Pre-flight checks
echo ">> Pre-flight checks..."

# 1. Verificar que existe la configuración
if [ ! -f "$PROJECT_DIR/config/database.conf" ]; then
    echo "   [FALLO] Falta config/database.conf"
    DEPLOY_OK=false
else
    echo "   [OK] Configuración de base de datos"
fi

if [ ! -f "$PROJECT_DIR/config/api.conf" ]; then
    echo "   [FALLO] Falta config/api.conf"
    DEPLOY_OK=false
else
    echo "   [OK] Configuración de API"
fi

# 2. Verificar que .env existe y es seguro
if [ ! -f "$PROJECT_DIR/.env" ]; then
    echo "   [FALLO] Falta .env"
    DEPLOY_OK=false
else
    PERMS=$(stat -c "%a" "$PROJECT_DIR/.env" 2>/dev/null || stat -f "%Lp" "$PROJECT_DIR/.env" 2>/dev/null)
    if [ "$PERMS" = "600" ]; then
        echo "   [OK] .env existe y tiene permisos seguros ($PERMS)"
    else
        echo "   [ALERTA] .env tiene permisos $PERMS — debería ser 600"
        DEPLOY_OK=false
    fi
fi

# 3. Verificar que hay al menos un vehículo registrado
VEHICLES=$(find "$PROJECT_DIR/data/vehicles" -name "*.dat" | wc -l)
if [ "$VEHICLES" -eq 0 ]; then
    echo "   [FALLO] No hay vehículos registrados"
    DEPLOY_OK=false
else
    echo "   [OK] $VEHICLES vehículos registrados"
fi

echo ""

# Decisión de deploy
if [ "$DEPLOY_OK" = true ]; then
    echo ">> Todos los checks pasaron."
    echo ">> Simulando deploy..."
    echo "   Cargando configuración..."
    echo "   Conectando a base de datos..."
    echo "   Iniciando API en puerto 8080..."
    echo "   Registrando en logs..."
    echo "$(date '+[%Y-%m-%d %H:%M:%S]') DEPLOY: Despliegue exitoso" >> "$PROJECT_DIR/logs/system.log"
    echo ""
    echo "DEPLOY COMPLETADO — Launch Control operativo"
else
    echo ">> DEPLOY ABORTADO — corrijan los errores antes de desplegar"
    exit 1
fi
SCRIPT

chmod 755 ~/launch-control/scripts/deploy.sh
```

Ejecuten los cuatro scripts en secuencia:

```bash
echo "=== HEALTH CHECK ===" && ~/launch-control/scripts/health_check.sh
echo ""
echo "=== CLEANUP ===" && ~/launch-control/scripts/cleanup.sh
echo ""
echo "=== BACKUP ===" && ~/launch-control/scripts/backup.sh
echo ""
echo "=== DEPLOY ===" && ~/launch-control/scripts/deploy.sh
```

## Verificación de Misión 5

Los cuatro scripts deben ejecutarse sin errores. Si alguno falla,
lean el mensaje de error — les dice exactamente qué corregir.

```bash
# Verificar que todos los scripts son ejecutables y tienen contenido
for s in ~/launch-control/scripts/*.sh; do
    LINES=$(wc -l < "$s")
    echo "$(basename $s): $LINES líneas, $(test -x "$s" && echo 'ejecutable' || echo 'NO ejecutable')"
done
```

---

# MISIÓN 6 — INSPECCIÓN FINAL

**Tiempo:** 30 minutos
**Temas:** todo lo anterior consolidado

---

Antes de entregar el servidor a la dirección técnica, necesitan
generar un informe completo del estado.

## Paso 1: Generar el informe

Creen un script que genera el informe automáticamente:

```bash
cat > ~/launch-control/scripts/generate_report.sh << 'SCRIPT'
#!/bin/bash
# ═══════════════════════════════════════════
# Launch Control — Generador de informe
# ═══════════════════════════════════════════

PROJECT_DIR="$HOME/launch-control"
REPORT="$PROJECT_DIR/docs/status_report.txt"

{
echo "═══════════════════════════════════════════════════════"
echo "  LAUNCH CONTROL — INFORME DE ESTADO"
echo "  Generado: $(date '+%Y-%m-%d %H:%M:%S')"
echo "  Generado por: $(whoami)@$(hostname)"
echo "═══════════════════════════════════════════════════════"
echo ""

echo ">> ESTRUCTURA DEL PROYECTO"
tree "$PROJECT_DIR" 2>/dev/null || find "$PROJECT_DIR" -print | sort
echo ""

echo ">> ESTADÍSTICAS"
echo "   Total archivos: $(find "$PROJECT_DIR" -type f | wc -l)"
echo "   Total directorios: $(find "$PROJECT_DIR" -type d | wc -l)"
echo "   Tamaño total: $(du -sh "$PROJECT_DIR" | awk '{print $1}')"
echo ""

echo ">> VEHÍCULOS REGISTRADOS"
for v in "$PROJECT_DIR"/data/vehicles/*.dat; do
    NAME=$(grep "^NAME=" "$v" | cut -d= -f2)
    STATUS=$(grep "^STATUS=" "$v" | cut -d= -f2)
    echo "   - $NAME ($STATUS)"
done
echo ""

echo ">> TRIPULACIONES"
for c in "$PROJECT_DIR"/data/crews/*.dat; do
    NAME=$(grep "^NAME=" "$c" | cut -d= -f2)
    COMMANDER=$(grep "^COMMANDER=" "$c" | cut -d= -f2)
    STATUS=$(grep "^STATUS=" "$c" | cut -d= -f2)
    echo "   - Tripulación $NAME: Comandante $COMMANDER ($STATUS)"
done
echo ""

echo ">> MISIONES"
echo "   Completadas: $(find "$PROJECT_DIR/data/missions/completed" -name "*.log" 2>/dev/null | wc -l)"
echo "   Fallidas: $(find "$PROJECT_DIR/data/missions/failed" -name "*.log" 2>/dev/null | wc -l)"
echo "   Demoradas: $(find "$PROJECT_DIR/data/missions/delayed" -name "*.log" 2>/dev/null | wc -l)"
echo ""

echo ">> SEGURIDAD"
echo "   Archivos con permiso 600 (solo dueño):"
find "$PROJECT_DIR" -type f -perm 600 -exec echo "   - {}" \;
echo ""
echo "   Scripts ejecutables:"
find "$PROJECT_DIR/scripts" -type f -executable -exec echo "   - {}" \;
echo ""

echo ">> ÚLTIMAS ENTRADAS DE LOG"
echo "   System log (últimas 3 líneas):"
tail -3 "$PROJECT_DIR/logs/system.log" | sed 's/^/   /'
echo ""
echo "   Error log (últimas 3 líneas):"
tail -3 "$PROJECT_DIR/logs/errors.log" | sed 's/^/   /'
echo ""

echo "═══════════════════════════════════════════════════════"
echo "  FIN DEL INFORME"
echo "═══════════════════════════════════════════════════════"
} > "$REPORT"

echo "Informe generado en: $REPORT"
echo "Líneas del informe: $(wc -l < "$REPORT")"
SCRIPT

chmod 755 ~/launch-control/scripts/generate_report.sh
```

Ejecútenlo y revisen el informe:

```bash
~/launch-control/scripts/generate_report.sh
cat ~/launch-control/docs/status_report.txt
```

## Paso 2: Verificación final completa

Ejecuten esta secuencia de verificaciones:

```bash
echo "=== VERIFICACIÓN FINAL ==="
echo ""

# 1. Estructura
echo "1. ESTRUCTURA"
tree ~/launch-control | tail -1
echo ""

# 2. Archivos totales
echo "2. ARCHIVOS"
find ~/launch-control -type f | wc -l
echo ""

# 3. Permisos de secretos
echo "3. PERMISOS SECRETOS"
stat -c "%a %n" ~/launch-control/config/secrets/* ~/launch-control/.env
echo ""

# 4. Scripts ejecutables
echo "4. SCRIPTS"
ls -la ~/launch-control/scripts/*.sh | awk '{print $1, $NF}'
echo ""

# 5. Misiones organizadas
echo "5. MISIONES ORGANIZADAS"
tree ~/launch-control/data/missions/
echo ""

# 6. Health check final
echo "6. HEALTH CHECK"
~/launch-control/scripts/health_check.sh
```

---

# RETOS EXTRA (para quienes terminen antes)

---

Estos retos no tienen solución escrita. Usen lo que aprendieron
y el `man` de cada comando si necesitan ayuda.

## Reto 1: Monitor de logs en tiempo real

Escriban un script `scripts/monitor.sh` que:
1. Muestre las últimas 5 líneas de cada archivo `.log` en `logs/`
2. Muestre la hora actual
3. Se ejecute en bucle cada 5 segundos (pista: `while true` + `sleep`)
4. Se pueda detener con Ctrl+C

## Reto 2: Búsqueda inteligente

Sin abrir ningún archivo manualmente, respondan estas preguntas
usando solamente `grep`, `find`, `wc`, `sort` y pipes:

1. ¿Cuántos archivos `.dat` tiene el proyecto?
2. ¿Qué archivo contiene la palabra "CRITICAL"?
3. ¿Cuál es el vehículo con menos lanzamientos?
4. ¿Cuántas líneas tiene el archivo más largo del proyecto?
5. ¿Qué archivos fueron modificados en los últimos 10 minutos?

Pista para la 4:
```bash
find ~/launch-control -type f -exec wc -l {} \; | sort -n | tail -1
```

Pista para la 5:
```bash
find ~/launch-control -type f -mmin -10
```

## Reto 3: Script de nuevo vehículo

Escriban un script `scripts/add_vehicle.sh` que reciba como
parámetros el nombre del vehículo y la carga útil, y cree
automáticamente el archivo `.dat` con el formato correcto.

Uso esperado:
```bash
~/launch-control/scripts/add_vehicle.sh "Ariane 6" 21650
```

Resultado esperado: un archivo `data/vehicles/ariane_6.dat` con
el formato estándar.

## Reto 4: Auditoría de seguridad

Escriban un script `scripts/security_audit.sh` que:
1. Busque todos los archivos con permisos `777` (peligroso) y los reporte
2. Busque archivos que contengan la palabra "password" y verifique que tienen permiso `600`
3. Busque intentos de login fallidos en los logs
4. Genere un informe de seguridad en `docs/security_audit.txt`

---

# ENTREGABLES

---

Al terminar, deben poder responder "sí" a cada una de estas preguntas:

| # | Verificación | Comando para comprobar |
|---|---|---|
| 1 | La estructura tiene 11+ directorios y 25+ archivos | `find ~/launch-control -type f \| wc -l` |
| 2 | Los secretos tienen permiso 600 | `stat -c "%a" ~/launch-control/config/secrets/*` |
| 3 | Los scripts son ejecutables (755) | `stat -c "%a" ~/launch-control/scripts/*.sh` |
| 4 | Las misiones están organizadas por estado | `tree ~/launch-control/data/missions/` |
| 5 | El health_check pasa sin errores | `~/launch-control/scripts/health_check.sh` |
| 6 | El deploy se ejecuta correctamente | `~/launch-control/scripts/deploy.sh` |
| 7 | El informe de estado se generó | `cat ~/launch-control/docs/status_report.txt` |
| 8 | El backup existe | `ls -d ~/launch-control-backup-*` |

**Subir a Google Classroom:**
1. Captura de `tree ~/launch-control` (estructura completa)
2. Captura del health_check ejecutándose (sin errores)
3. Captura del deploy ejecutándose (exitoso)
4. El archivo `docs/status_report.txt` (copiar y pegar el contenido)

---

# TABLA DE REFERENCIA — TODOS LOS COMANDOS DE LA SEMANA

| Comando | Qué hace | Ejemplo |
|---|---|---|
| `pwd` | Muestra directorio actual | `pwd` |
| `cd ruta` | Cambia de directorio | `cd ~/launch-control` |
| `cd ..` | Sube un nivel | `cd ..` |
| `cd ~` | Va al home | `cd ~` |
| `ls` | Lista archivos | `ls` |
| `ls -la` | Lista con detalles y ocultos | `ls -la` |
| `mkdir nombre` | Crea directorio | `mkdir docs` |
| `mkdir -p a/b/c` | Crea cadena de directorios | `mkdir -p src/api/tests` |
| `touch archivo` | Crea archivo vacío | `touch notas.txt` |
| `echo "texto"` | Imprime texto | `echo "hola"` |
| `>` | Redirige (sobreescribe) | `echo "x" > file.txt` |
| `>>` | Redirige (añade al final) | `echo "y" >> file.txt` |
| `cat << 'EOF'` | Escribe bloque de texto | `cat > file << 'EOF'` |
| `cat archivo` | Muestra contenido | `cat README.md` |
| `cat -n archivo` | Muestra con números de línea | `cat -n script.sh` |
| `head -N` | Primeras N líneas | `head -5 log.txt` |
| `tail -N` | Últimas N líneas | `tail -10 errors.log` |
| `less archivo` | Visor con scroll | `less /etc/passwd` |
| `wc -l` | Cuenta líneas | `wc -l archivo.txt` |
| `cp origen destino` | Copia archivo | `cp a.txt b.txt` |
| `cp -r dir/ dir2/` | Copia directorio completo | `cp -r data/ backup/` |
| `mv origen destino` | Mueve o renombra | `mv old.txt new.txt` |
| `rm archivo` | Elimina archivo | `rm temporal.txt` |
| `rm -r directorio/` | Elimina directorio y todo dentro | `rm -r tmp/` |
| `rm -ri directorio/` | Elimina con confirmación | `rm -ri backup/` |
| `rmdir directorio` | Elimina directorio vacío | `rmdir empty/` |
| `chmod NNN archivo` | Cambia permisos | `chmod 755 script.sh` |
| `chmod 600` | Solo dueño lee y escribe | `chmod 600 .env` |
| `chmod 644` | Dueño escribe, todos leen | `chmod 644 README.md` |
| `chmod 755` | Dueño todo, otros leen y ejecutan | `chmod 755 deploy.sh` |
| `ls -l` | Ver permisos de archivos | `ls -l scripts/` |
| `stat -c "%a" file` | Ver permiso en octal | `stat -c "%a" .env` |
| `find dir -type f` | Busca archivos | `find . -type f` |
| `find dir -name "*.sh"` | Busca por nombre | `find . -name "*.log"` |
| `find -exec cmd {} \;` | Ejecuta comando por resultado | `find . -name "*.sh" -exec chmod 755 {} \;` |
| `grep "texto" archivo` | Busca texto en archivo | `grep "ERROR" log.txt` |
| `grep -r "texto" dir/` | Busca recursivamente | `grep -r "password" .` |
| `grep -c "texto"` | Cuenta coincidencias | `grep -c "WARN" errors.log` |
| `grep -l "texto" dir/*` | Solo nombres de archivo | `grep -l "ABORT" *.log` |
| `diff a b` | Compara dos archivos | `diff old.conf new.conf` |
| `du -sh directorio` | Tamaño de directorio | `du -sh ~/launch-control` |
| `tree directorio` | Árbol visual | `tree ~/launch-control` |
| `\|` (pipe) | Conecta salida con entrada | `find . \| wc -l` |
| `*` | Wildcard: cualquier texto | `ls *.txt` |
| `?` | Wildcard: un carácter | `ls file?.dat` |
| `$()` | Sustitución de comando | `echo "Hoy: $(date)"` |
| `$HOME` | Variable: directorio home | `echo $HOME` |
| `$?` | Variable: código de salida anterior | `echo $?` |
| `&&` | Ejecuta si el anterior tuvo éxito | `mkdir x && cd x` |
| `\|\|` | Ejecuta si el anterior falló | `test -f x \|\| echo "no existe"` |
| `2>/dev/null` | Descarta errores | `ls /x 2>/dev/null` |
| `history` | Historial de comandos | `history` |
| `Tab` | Autocompleta | escribir parcial + Tab |
| `Ctrl+C` | Cancela el comando actual | — |
| `Ctrl+L` | Limpia la pantalla | — |
| `bash script.sh` | Ejecuta un script | `bash backup.sh` |
| `./script.sh` | Ejecuta (necesita chmod +x) | `./health_check.sh` |
| `#!/bin/bash` | Shebang: indica el intérprete | primera línea del script |

---

## Créditos y referencias

| | |
|---|---|
| **Autor original** | Prof. Juan Marcelo Gutiérrez Miranda |
| **Institución** | @TodoEconometria |

---

> **Curso IFCD0112 — Programación con Lenguajes OO y BBDD Relacionales**
> Material complementario de estudio
>
> **Autor:** Prof. Juan Marcelo Gutiérrez Miranda
> **Institución:** @TodoEconometria
>
> Este documento es un recurso bibliográfico adicional para los alumnos del curso.
> Módulo: MF0223_3 — Sistemas informáticos
>
> ---
>
> **Propiedad intelectual:** Este material didáctico, su metodología, estructura,
> ejemplos y código base son producción intelectual de Juan Marcelo Gutiérrez Miranda.
> El contenido técnico de Python, PostgreSQL, Linux y sus ecosistemas pertenece
> a sus respectivos autores y comunidades; la organización pedagógica, las
> explicaciones, los diagramas y el enfoque metodológico son obra original del autor.
