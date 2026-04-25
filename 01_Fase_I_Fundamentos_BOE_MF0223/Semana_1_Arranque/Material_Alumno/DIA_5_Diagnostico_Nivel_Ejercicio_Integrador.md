# Día 5: Diagnóstico de nivel y ejercicio integrador

Llevan 4 días de curso. Tienen Linux instalado, dominan la terminal básica,
saben crear y mover ficheros, y entienden los permisos. Hoy es el día de
la verdad: vamos a medir qué han retenido realmente, descubrir dónde están
los huecos, y construir algo completo de principio a fin.

Prof. Juan Marcelo Gutiérrez Miranda

**Curso IFCD0112 — Semana 1, Día 5 (Miércoles)**
**Objetivo:** Evaluar el nivel real del grupo a través de ejercicios prácticos
individuales. Consolidar todo lo aprendido en los días 1-4 con un proyecto
integrador completo.

> Este manual tiene dos partes: un diagnóstico individual (sin ayuda) y un
> ejercicio integrador en grupo. Sigan las instrucciones del formador.

---

## PASO 0 — De dónde venimos

```
╔══════════════════════════════════════════════════════════════════╗
║              EVOLUCIÓN DE AMTIGRAVITY LAUNCH CONTROL             ║
╠══════════════════════════════════════════════════════════════════╣
║                                                                  ║
║  Día 1   BOE + Propuesta         Mapa del curso                  ║
║  Día 2   WSL + Ubuntu            Taller montado                  ║
║  Día 3   Terminal + ficheros     Estructura creada               ║
║  Día 4   Permisos + FHS         Seguridad del sistema            ║
║  ► DÍA 5  Diagnóstico + integrador  Medimos nivel real           ║
║                                                                  ║
╚══════════════════════════════════════════════════════════════════╝
```

| Aspecto | Días 2-4 | Hoy (Día 5) |
|---------|----------|-------------|
| Modo | Guiado paso a paso | Autónomo — ustedes solos |
| Objetivo | Aprender herramientas | Demostrar que las dominan |
| Ayuda | Manual + formador + compañeros | Solo ustedes (en el diagnóstico) |

---

## Índice del día

| Parte | Contenido |
|---|---|
| I   | Diagnóstico individual (sin ayuda, sin manual) |
| II  | Corrección colectiva y análisis de errores |
| III | Ejercicio integrador: Misión Artemis |
| IV  | Reto avanzado: automatización con scripts |
| V   | Reflexión y auto-evaluación |

---

# PARTE I — DIAGNÓSTICO INDIVIDUAL

---

## Reglas del diagnóstico

```
╔══════════════════════════════════════════════════════════════╗
║  REGLAS — LEAN ANTES DE EMPEZAR                               ║
╠══════════════════════════════════════════════════════════════╣
║                                                              ║
║  1. Solo ustedes y la terminal. Sin manual. Sin internet.    ║
║  2. No hablar con el compañero. Trabajo 100% individual.     ║
║  3. Duración: 60 minutos exactos.                            ║
║  4. Hay 3 bloques: conceptual, práctico y de permisos.       ║
║  5. No pasa nada si no terminan todo. Esto no es un examen   ║
║     con nota — es un mapa para saber dónde están.            ║
║  6. Hagan lo que sepan. Dejen en blanco lo que no.           ║
║     Inventar respuestas no ayuda a nadie.                    ║
║                                                              ║
╚══════════════════════════════════════════════════════════════╝
```

---

## BLOQUE A — Conceptual (15 minutos, en papel)

Respondan en papel, sin terminal:

### A.1 Completar la tabla

Rellenen la columna "Para qué sirve" de memoria:

| Directorio | Para qué sirve |
|---|---|
| `/home` | |
| `/etc` | |
| `/var/log` | |
| `/tmp` | |
| `/bin` | |
| `/root` | |

### A.2 Verdadero o Falso

Marquen V (verdadero) o F (falso) y corrijan las falsas:

1. En Linux, `Archivo.txt` y `archivo.txt` son el mismo archivo. [ ]
2. El comando `rm` envía los archivos a la papelera de reciclaje. [ ]
3. El operador `>>` sobreescribe el contenido de un archivo. [ ]
4. El símbolo `~` equivale al directorio `/home/alumno`. [ ]
5. El permiso `755` significa que el propietario puede leer, escribir
   y ejecutar, y los demás solo leer y ejecutar. [ ]
6. `sudo` permite ejecutar un comando como administrador. [ ]
7. Los archivos ocultos en Linux empiezan por `_` (guion bajo). [ ]
8. El comando `mkdir -p a/b/c` falla si el directorio `a` no existe. [ ]
9. En el prompt `alumno@ubuntu2204:~$`, el `$` indica que son root. [ ]
10. El comando `chmod 644 archivo.txt` da permiso de ejecución al propietario. [ ]

### A.3 Escribir los comandos

Sin ejecutarlos — escriban el comando que usarían para:

1. Ver en qué directorio estoy:
2. Listar todos los archivos (incluidos ocultos) con detalles:
3. Crear un directorio llamado `proyecto` con subdirectorios `src` y `docs`
   en un solo comando:
4. Copiar el directorio `proyecto` a `proyecto_backup` con todo su contenido:
5. Mover el archivo `notas.txt` al directorio `docs/`:
6. Ver las últimas 10 líneas del archivo `/var/log/syslog`:
7. Contar cuántas líneas tiene un archivo llamado `datos.csv`:
8. Dar permisos 755 al archivo `script.sh`:
9. Crear un archivo `info.txt` con el contenido "Hola mundo" (un solo comando):
10. Añadir la línea "Segunda línea" al final de `info.txt` sin borrar lo anterior:

---

## BLOQUE B — Práctico en terminal (30 minutos)

Abran la terminal. Ejecuten las tareas en orden. Al final de cada tarea,
dejen la terminal visible para que el formador pueda verificar.

### B.1 Estructura de directorios (10 minutos)

Creen esta estructura completa en su home:

```
~/diagnostico/
├── satellites/
│   ├── hubble/
│   │   └── specs.txt          (contenido: "Lanzamiento: 1990")
│   ├── james_webb/
│   │   └── specs.txt          (contenido: "Lanzamiento: 2021")
│   └── iss/
│       └── specs.txt          (contenido: "Lanzamiento: 1998")
├── rockets/
│   ├── saturn_v.txt           (contenido: "Carga LEO: 140000 kg")
│   ├── sls.txt                (contenido: "Carga LEO: 95000 kg")
│   └── falcon_heavy.txt       (contenido: "Carga LEO: 63800 kg")
├── missions/
│   └── manifest.txt           (vacío)
└── README.txt                 (contenido: "Proyecto Diagnóstico - [tu nombre]")
```

**Verificación:** ejecuten `tree ~/diagnostico` y dejen la salida visible.

### B.2 Operaciones con archivos (10 minutos)

Partiendo de la estructura anterior:

1. Copien `saturn_v.txt` a la carpeta `missions/` con el nombre `primary_rocket.txt`
2. Muevan `falcon_heavy.txt` de `rockets/` a `missions/`
3. Creen un archivo `missions/manifest.txt` con estas 3 líneas:
   ```
   Misión: Artemis I
   Fecha: 2025-11-01
   Estado: Planificada
   ```
4. Añadan una cuarta línea al manifest: `Cohete: SLS`
5. Muestren el contenido de `manifest.txt` con números de línea
6. Cuenten cuántos archivos regulares hay en todo `~/diagnostico`

**Verificación:** ejecuten `cat -n ~/diagnostico/missions/manifest.txt` y
`find ~/diagnostico -type f | wc -l`

### B.3 Permisos (10 minutos)

1. Verifiquen los permisos actuales de `rockets/saturn_v.txt`:
   ```bash
   ls -l ~/diagnostico/rockets/saturn_v.txt
   ```
2. Cambien los permisos de `saturn_v.txt` a:
   - Propietario: leer y escribir
   - Grupo: solo leer
   - Otros: nada
   (¿Qué número octal es?)
3. Cambien los permisos de `missions/manifest.txt` a 755.
   ¿Qué significa eso en la práctica? ¿Tiene sentido para un .txt?
4. Creen un archivo `scripts/test.sh` con este contenido:
   ```
   #!/bin/bash
   echo "Test del diagnóstico"
   ```
   Denle permisos de ejecución (propietario) y ejecútenlo con `./test.sh`.
5. Listen los permisos de TODOS los archivos de `~/diagnostico` recursivamente:
   ```bash
   find ~/diagnostico -type f -exec ls -l {} \;
   ```

**Verificación:** dejen la salida del último comando visible.

---

# PARTE II — CORRECCIÓN COLECTIVA Y ANÁLISIS DE ERRORES

---

## 1. Corrección del Bloque A

*(El formador proyectará las respuestas correctas y las repasaremos juntos.)*

### A.1 Respuestas de la tabla

| Directorio | Para qué sirve |
|---|---|
| `/home` | Carpetas personales de los usuarios |
| `/etc` | Configuración del sistema y servicios |
| `/var/log` | Registros (logs) del sistema |
| `/tmp` | Archivos temporales (se borran al reiniciar) |
| `/bin` | Comandos esenciales del sistema |
| `/root` | Directorio personal del superusuario (root) |

### A.2 Respuestas Verdadero/Falso

1. **F** — Son archivos diferentes. Linux distingue mayúsculas.
2. **F** — `rm` elimina permanentemente. No hay papelera.
3. **F** — `>>` añade al final. `>` es el que sobreescribe.
4. **V** — Correcto (si tu usuario es "alumno").
5. **V** — 7=rwx para owner, 5=r-x para group y others.
6. **V** — Correcto.
7. **F** — Empiezan por `.` (punto), no por guion bajo.
8. **F** — Con `-p`, mkdir crea toda la cadena sin error.
9. **F** — `$` indica usuario normal. `#` indica root.
10. **F** — 644 = rw-r--r--. No da ejecución a nadie.

### A.3 Respuestas de comandos

1. `pwd`
2. `ls -la`
3. `mkdir -p proyecto/{src,docs}`
4. `cp -r proyecto proyecto_backup`
5. `mv notas.txt docs/`
6. `tail -10 /var/log/syslog` (o `tail /var/log/syslog` ya que 10 es el defecto)
7. `wc -l datos.csv`
8. `chmod 755 script.sh`
9. `echo "Hola mundo" > info.txt`
10. `echo "Segunda línea" >> info.txt`

---

## 2. Errores más comunes que han cometido

*(Esta sección se rellena en clase con los errores reales que el formador
detecte al recorrer las mesas. Aquí van los errores típicos:)*

| Error | Por qué ocurre | Solución |
|---|---|---|
| `mkdir proyecto/src/docs` sin -p | Falta -p para crear cadena | `mkdir -p proyecto/{src,docs}` |
| `cp proyecto proyecto_backup` sin -r | cp no copia directorios sin -r | `cp -r proyecto proyecto_backup` |
| Confundir `>` con `>>` | Son parecidos pero hacen lo contrario | `>` sobreescribe, `>>` añade |
| `chmod 755 archivo.txt` para un .txt | 755 da ejecución, un .txt no se ejecuta | Usar 644 para archivos de texto |
| Escribir rutas con espacios sin comillas | Linux interpreta el espacio como separador | Usar comillas o guiones bajos |
| `rm -r` sin verificar antes | Se borra todo sin confirmar | Siempre `ls` antes, o usar `rm -ri` |
| Olvidar `sudo` para `/etc` o `/var` | Esos directorios requieren permisos de root | `sudo comando` |
| Confundir ruta absoluta y relativa | `/home/alumno` vs `alumno` | Absoluta empieza con `/` |

---

# PARTE III — EJERCICIO INTEGRADOR: MISIÓN ARTEMIS

---

## Contexto

La NASA planifica la misión Artemis III: el primer alunizaje tripulado desde
1972. Van a crear la infraestructura de archivos del proyecto como si
fueran el equipo de sistemas de la misión.

Este ejercicio integra TODO lo aprendido en los días 1-4:
- Creación de estructura de directorios (Día 3)
- Redirecciones y creación de archivos (Día 3)
- Permisos y propiedad (Día 4)
- Navegación y verificación (Días 2-3)

---

## 3. Fase 1 — Estructura base (20 minutos)

Creen esta estructura completa:

```
~/artemis-iii/
├── crew/
│   ├── commander/
│   │   └── profile.txt
│   ├── pilot/
│   │   └── profile.txt
│   └── specialist/
│       └── profile.txt
├── vehicle/
│   ├── sls/
│   │   ├── stage1.txt
│   │   ├── stage2.txt
│   │   └── boosters.txt
│   ├── orion/
│   │   ├── capsule.txt
│   │   └── service_module.txt
│   └── hls/
│       └── starship.txt
├── mission_control/
│   ├── timeline.txt
│   ├── procedures/
│   │   ├── launch.txt
│   │   ├── orbit.txt
│   │   ├── landing.txt
│   │   └── return.txt
│   └── comms/
│       └── frequencies.txt
├── science/
│   ├── experiments/
│   │   ├── geology.txt
│   │   └── biology.txt
│   └── samples/
│       └── .gitkeep
├── logs/
│   ├── mission.log
│   └── errors.log
├── scripts/
│   ├── preflight_check.sh
│   ├── telemetry.sh
│   └── backup.sh
└── README.md
```

### 3.1 Contenido de los archivos principales

**README.md:**
```
# Artemis III - Infraestructura de Misión
Equipo: [tu nombre]
Fecha: [fecha de hoy]
Estado: Pre-lanzamiento
```

**crew/commander/profile.txt:**
```
Nombre: Reid Wiseman
Rango: Captain, USN
Misiones previas: Expedition 41
Rol: Commander
```

**crew/pilot/profile.txt:**
```
Nombre: Victor Glover
Rango: Captain, USN
Misiones previas: Crew-1
Rol: Pilot
```

**crew/specialist/profile.txt:**
```
Nombre: Christina Koch
Misiones previas: Expedition 59/60
Rol: Mission Specialist
Récord: Vuelo espacial femenino más largo (328 días)
```

**vehicle/sls/stage1.txt:**
```
Nombre: Core Stage
Propulsante: Hidrógeno líquido + Oxígeno líquido
Motores: 4x RS-25
Empuje: 7,440 kN
```

**vehicle/sls/boosters.txt:**
```
Nombre: Solid Rocket Boosters
Cantidad: 2
Propulsante: Sólido (PBAN)
Empuje por unidad: 14,000 kN
Tiempo de combustión: 126 segundos
```

**vehicle/orion/capsule.txt:**
```
Nombre: Orion MPCV
Tripulación máxima: 4
Volumen habitable: 8.95 m3
Escudo térmico: AVCOAT (ablativo)
```

**mission_control/timeline.txt:**
```
T-00:00:00  Ignición
T+00:02:06  Separación boosters
T+00:08:00  Separación Stage 1
T+00:08:30  Encendido Stage 2
T+02:00:00  Inserción orbital
T+04:00:00  Inyección trans-lunar
Día 4       Inserción orbital lunar
Día 6       Descenso a superficie
Día 8       Ascenso y acoplamiento
Día 12      Regreso a Tierra
```

**mission_control/procedures/launch.txt:**
```
PROCEDIMIENTO DE LANZAMIENTO
=============================
1. Verificar telemetría de todos los sistemas
2. Confirmar "GO" de cada estación
3. Autorizar secuencia de cuenta atrás automática
4. T-10s: Activar sistema de supresión de sonido
5. T-6.6s: Ignición motores RS-25
6. T-0s: Ignición boosters / Despegue
7. T+7s: Roll program
8. T+60s: Max-Q (máxima presión dinámica)
```

**logs/mission.log:**
```
[2026-04-15 09:00] Sistema inicializado
[2026-04-15 09:01] Todos los subsistemas: NOMINAL
```

**logs/errors.log:**
(vacío — usar `touch`)

**scripts/preflight_check.sh:**
```bash
#!/bin/bash
echo "=== PREFLIGHT CHECK ==="
echo "Verificando estructura de directorios..."
dirs=$(find ~/artemis-iii -type d | wc -l)
files=$(find ~/artemis-iii -type f | wc -l)
echo "Directorios: $dirs"
echo "Archivos: $files"
echo "Verificando permisos de scripts..."
ls -l ~/artemis-iii/scripts/*.sh
echo "=== CHECK COMPLETO ==="
```

**scripts/telemetry.sh:**
```bash
#!/bin/bash
echo "=== TELEMETRÍA ==="
echo "Fecha: $(date)"
echo "Usuario: $(whoami)"
echo "Hostname: $(hostname)"
echo "RAM disponible:"
free -h | head -2
echo "Disco disponible:"
df -h / | tail -1
echo "=== FIN TELEMETRÍA ==="
```

**scripts/backup.sh:**
```bash
#!/bin/bash
echo "Creando backup de Artemis III..."
cp -r ~/artemis-iii ~/artemis-iii-backup-$(date +%Y%m%d)
echo "Backup completado en ~/artemis-iii-backup-$(date +%Y%m%d)"
```

---

## 4. Fase 2 — Permisos y seguridad (15 minutos)

Apliquen estos permisos a la estructura:

### 4.1 Scripts ejecutables

Todos los archivos `.sh` deben tener permiso de ejecución para el propietario:

```bash
chmod 755 ~/artemis-iii/scripts/*.sh
```

Verifiquen que funcionan:

```bash
bash ~/artemis-iii/scripts/preflight_check.sh
bash ~/artemis-iii/scripts/telemetry.sh
```

### 4.2 Archivos confidenciales

Los perfiles de la tripulación son confidenciales — solo el propietario
puede leerlos:

```bash
chmod 600 ~/artemis-iii/crew/*/profile.txt
```

Verifiquen:

```bash
ls -l ~/artemis-iii/crew/commander/profile.txt
```

```
# salida esperada:
-rw------- 1 alumno alumno ... profile.txt
```

### 4.3 Procedimientos: lectura para todos

Los procedimientos de misión deben ser legibles por cualquier miembro del
equipo:

```bash
chmod 644 ~/artemis-iii/mission_control/procedures/*.txt
```

### 4.4 Logs: solo el propietario escribe, grupo puede leer

```bash
chmod 640 ~/artemis-iii/logs/*.log
```

### 4.5 Verificación completa de permisos

```bash
find ~/artemis-iii -type f -exec ls -l {} \; | head -20
```

---

## 5. Fase 3 — Operaciones de misión (15 minutos)

### 5.1 Añadir entradas al log

```bash
echo "[$(date '+%Y-%m-%d %H:%M')] Estructura de misión creada" >> ~/artemis-iii/logs/mission.log
echo "[$(date '+%Y-%m-%d %H:%M')] Permisos configurados" >> ~/artemis-iii/logs/mission.log
echo "[$(date '+%Y-%m-%d %H:%M')] Preflight check ejecutado" >> ~/artemis-iii/logs/mission.log
```

### 5.2 Crear informe de estado

Creen un archivo `~/artemis-iii/status_report.txt` que contenga:

```
INFORME DE ESTADO - ARTEMIS III
================================
Generado: [la fecha y hora actuales]
Operador: [tu usuario]

TRIPULACIÓN:
- Commander: Reid Wiseman
- Pilot: Victor Glover
- Specialist: Christina Koch

VEHÍCULO:
- Lanzador: SLS Block 1 (4x RS-25 + 2 SRB)
- Cápsula: Orion MPCV
- Módulo lunar: Starship HLS

ESTADO DE DIRECTORIOS:
[pegar aquí la salida de tree ~/artemis-iii]

ESTADO DE ARCHIVOS:
[pegar aquí la salida de find ~/artemis-iii -type f | wc -l] archivos totales

PERMISOS DE SCRIPTS:
[pegar aquí la salida de ls -l ~/artemis-iii/scripts/*.sh]
```

### 5.3 Verificación cruzada entre compañeros

Cuando terminen el informe de estado, intercambien puestos con el compañero
de al lado y verifiquen SU estructura:

**Lista de verificación cruzada:**

| # | Verificación | ¿OK? |
|---|---|---|
| 1 | `tree ~/artemis-iii` muestra estructura completa | [ ] |
| 2 | `cat ~/artemis-iii/crew/commander/profile.txt` tiene datos correctos | [ ] |
| 3 | `ls -l ~/artemis-iii/scripts/preflight_check.sh` muestra permiso x | [ ] |
| 4 | `ls -l ~/artemis-iii/crew/commander/profile.txt` muestra 600 (rw-------) | [ ] |
| 5 | `bash ~/artemis-iii/scripts/telemetry.sh` se ejecuta sin errores | [ ] |
| 6 | `cat -n ~/artemis-iii/logs/mission.log` tiene al menos 4 líneas | [ ] |
| 7 | `cat ~/artemis-iii/mission_control/timeline.txt` tiene la secuencia completa | [ ] |
| 8 | `find ~/artemis-iii -type f \| wc -l` muestra al menos 18 archivos | [ ] |

Si encuentran un error en la estructura de su compañero, **no lo
corrijan ustedes** — indíquenle cuál es para que lo corrija él.

La capacidad de verificar el trabajo de otro es una habilidad profesional
fundamental: en el mundo real se llama **code review** y lo harán a diario
cuando trabajen como programadores.

---

### 5.4 Hacer backup

```bash
bash ~/artemis-iii/scripts/backup.sh
```

Verifiquen que el backup existe:

```bash
ls -d ~/artemis-iii-backup-*
```

---

# PARTE IV — RETO AVANZADO: AUTOMATIZACIÓN

---

## 6. Para los que van sobrados

*(Si han terminado todo lo anterior antes de tiempo, intenten estos retos.
No son obligatorios pero demuestran un nivel superior.)*

### Reto 1 — Script de inventario

Creen un script `~/artemis-iii/scripts/inventory.sh` que:
1. Liste todos los directorios del proyecto
2. Cuente cuántos archivos `.txt` hay
3. Cuente cuántos archivos `.sh` hay
4. Muestre el tamaño total del proyecto

Pistas:
```bash
find ~/artemis-iii -type d              # Lista directorios
find ~/artemis-iii -name "*.txt" | wc -l  # Cuenta .txt
du -sh ~/artemis-iii                    # Tamaño total
```

### Reto 2 — Buscar dentro de archivos

El comando `grep` busca texto dentro de archivos. Intenten:

```bash
# Buscar la palabra "Orion" en todos los archivos del proyecto
grep -r "Orion" ~/artemis-iii/

# Buscar archivos que mencionen "RS-25"
grep -rl "RS-25" ~/artemis-iii/

# Contar cuántas líneas contienen "Verificar"
grep -rc "Verificar" ~/artemis-iii/mission_control/
```

`grep` es uno de los comandos más poderosos de Linux. Lo usarán
constantemente el resto del curso.

### Reto 3 — Pipe: conectar comandos

El operador `|` (pipe) envía la salida de un comando a la entrada de otro:

```bash
# Listar archivos y contar cuántos hay
ls -la ~/artemis-iii/ | wc -l

# Ver los procesos que más RAM usan
ps aux | sort -k4 -rn | head -5

# Buscar archivos .sh y mostrar sus permisos
find ~/artemis-iii -name "*.sh" | xargs ls -l
```

---

# PARTE V — REFLEXIÓN Y AUTO-EVALUACIÓN

---

## 7. Auto-evaluación honesta

Respondan en papel. No hay respuestas correctas o incorrectas — esto es
para ustedes:

| Pregunta | Tu respuesta |
|---|---|
| Del 1 al 5, ¿cómo de cómodo me siento con la terminal? | |
| ¿Qué comando me cuesta más recordar? | |
| ¿He necesitado mirar el manual para el diagnóstico? | |
| ¿He podido ayudar a algún compañero esta semana? | |
| ¿Qué es lo que más me ha sorprendido de Linux? | |
| ¿Qué es lo que más me ha costado? | |

## 8. Qué viene la semana que viene

A partir del Día 6, empezamos temas nuevos. Todo lo de esta semana es la
base que necesitan para lo que viene:

| Concepto de esta semana | Dónde lo usarán |
|---|---|
| Terminal y navegación | SIEMPRE. Cada día del curso |
| Crear directorios y archivos | Proyectos, scripts, configuración |
| Permisos | PostgreSQL, Docker, despliegues |
| Redirecciones (> >>) | Logs, scripts de automatización |
| Scripts bash | Automatizar tareas repetitivas |
| Estructura de proyecto | Launch Control evolucionará cada semana |

Si alguien se incorpora la semana que viene, necesita:
1. Instalar WSL + Ubuntu (material del Día 2)
2. Practicar los comandos básicos (Días 3-4)
3. Preguntar a un compañero si algo no queda claro

---

## Tarea con entrega

Subir a Google Classroom:

1. **Foto o scan** del Bloque A del diagnóstico resuelto en papel
2. **Captura de pantalla** de `tree ~/diagnostico` (Bloque B)
3. **Captura de pantalla** de `tree ~/artemis-iii` (Ejercicio integrador)
4. **Captura de pantalla** de la ejecución de `preflight_check.sh`
5. **Captura de pantalla** de `cat ~/artemis-iii/status_report.txt`

**Formato:** 5 imágenes nombradas:
`dia5_1_papel.png`, `dia5_2_diagnostico.png`, `dia5_3_artemis.png`,
`dia5_4_preflight.png`, `dia5_5_status.png`

---

## Tabla de referencia rápida — Todo lo de la semana 1

| Categoría | Comando | Qué hace |
|---|---|---|
| **Navegación** | `pwd` | Dónde estoy |
| | `ls` / `ls -la` | Qué hay aquí |
| | `cd directorio` | Ir a un sitio |
| | `cd ..` | Subir un nivel |
| | `cd ~` | Volver al home |
| | `cd -` | Volver al anterior |
| **Crear** | `mkdir nombre` | Crear directorio |
| | `mkdir -p a/b/c` | Crear cadena |
| | `touch archivo` | Crear archivo vacío |
| | `echo "texto" > arch` | Crear con contenido |
| | `echo "texto" >> arch` | Añadir contenido |
| **Ver** | `cat archivo` | Ver contenido |
| | `cat -n archivo` | Ver con números |
| | `head -N archivo` | Primeras N líneas |
| | `tail -N archivo` | Últimas N líneas |
| | `less archivo` | Ver con scroll |
| | `wc -l archivo` | Contar líneas |
| | `tree directorio` | Árbol visual |
| **Copiar/Mover** | `cp a b` | Copiar archivo |
| | `cp -r dir dir2` | Copiar directorio |
| | `mv a b` | Mover o renombrar |
| **Borrar** | `rm archivo` | Borrar archivo |
| | `rm -r dir` | Borrar directorio |
| | `rm -i archivo` | Borrar con confirmación |
| | `rmdir dir` | Borrar dir vacío |
| **Permisos** | `chmod NNN arch` | Cambiar permisos (octal) |
| | `chmod u+x arch` | Dar ejecución a owner |
| | `chown user:group arch` | Cambiar propietario |
| | `ls -l` | Ver permisos |
| **Sistema** | `whoami` | Quién soy |
| | `hostname` | Nombre de máquina |
| | `free -h` | RAM disponible |
| | `df -h` | Disco disponible |
| | `nproc` | Número de CPUs |
| | `sudo comando` | Ejecutar como admin |
| **Buscar** | `find dir -name "*.txt"` | Buscar por nombre |
| | `find dir -type f` | Buscar archivos |
| | `grep "texto" arch` | Buscar dentro |
| | `grep -r "texto" dir` | Buscar recursivo |
| **Otros** | `history` | Historial de comandos |
| | Tab | Autocompletar |
| | Ctrl+C | Cancelar |
| | Ctrl+L | Limpiar pantalla |
| | `*` / `?` | Wildcards |
| **Permisos octal** | `644` | rw-r--r-- (archivo normal) |
| | `755` | rwxr-xr-x (script/directorio) |
| | `600` | rw------- (confidencial) |
| | `640` | rw-r----- (log) |
| | `700` | rwx------ (privado ejecutable) |

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
>
> **Hash de Certificación:** `4e8d9b1a5f6e7c3d2b1a0f9e8d7c6b5a4f3e2d1c0b9a8f7e6d5c4b3a2f1e0d9c`
