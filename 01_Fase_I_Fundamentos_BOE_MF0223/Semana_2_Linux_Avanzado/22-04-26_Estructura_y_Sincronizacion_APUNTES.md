# Apuntes — 22 de Abril 2026: Estructura del Proyecto y Sincronización

**Curso IFCD0112** | **Módulo:** MF0223_3 | Prof. Juan Marcelo Gutiérrez Miranda

---

# RECONEXIÓN

---

```bash
# Verificación rápida (3 comandos)
echo $XDG_SESSION_TYPE
sudo timeshift --list 2>/dev/null | head -3
dpkg -l | grep antigravity
```

---

# CONTEXTO

---

_(Tus notas aquí)_

---

# OBJETIVOS

---

```
╔═══════════════════════════════════════════════════════════════════╗
║  RESULTADOS ESPERADOS — 22 ABR 2026                              ║
╠═══════════════════════════════════════════════════════════════════╣
║                                                                   ║
║  1. ~/launch-control/         Estructura profesional del proyecto ║
║  2. ~/launch-control/.env     Archivo sensible con permisos 600  ║
║  3. health_check.sh           Script que verifica la estructura  ║
║  4. Carpeta compartida VBox   Windows ↔ Linux configurada        ║
║  5. sincronizar.sh            Script de sync automático          ║
║  6. Autostart                 Sync al iniciar sesión             ║
║  7. Snapshot GOLDEN v2        Todo funciona — punto seguro       ║
║                                                                   ║
╚═══════════════════════════════════════════════════════════════════╝
```

---

# BLOQUE 1 — ESTRUCTURA DE CARPETAS

---

## El problema

```bash
# ¿Dónde hay archivos sueltos?
find /home/$USER -maxdepth 2 -name "*.sh" -o -name "*.conf" -o -name "*.txt" 2>/dev/null | head -20

# ¿Qué hay en el escritorio?
ls ~/Escritorio/ 2>/dev/null || ls ~/Desktop/ 2>/dev/null

# ¿Qué hay en /tmp?
ls /tmp/*.conf /tmp/*.sh 2>/dev/null
```

---

## La filosofía: separar responsabilidades

_(Tus notas aquí)_

| Zona | Contiene | Regla |
|---|---|---|
| **config/** | Archivos de configuración | Parámetros que cambian entre entornos |
| **scripts/** | Scripts ejecutables | Lógica de automatización |
| **logs/** | Registros de ejecución | Se generan solos, se revisan cuando hay problemas |
| **data/** | Datos de la aplicación | Nunca se mezclan con código |
| **docs/** | Documentación | Explicaciones, README, diagramas |
| **.env** | Secretos (contraseñas, API keys) | NUNCA va al repositorio, permisos 600 |

---

## Estructura objetivo

```
~/
├── sync_curso/
│   └── mi_proyecto/            ← espejo carpeta compartida Windows
│
├── launch-control/
│   ├── config/
│   │   ├── database.conf       ← configuración de base de datos
│   │   └── network.conf        ← configuración de red
│   ├── scripts/
│   │   └── health_check.sh     ← verificación del proyecto
│   ├── logs/                   ← registros de ejecución
│   ├── data/                   ← datos de la aplicación
│   ├── docs/                   ← documentación
│   └── .env                    ← variables sensibles (permisos 600)
│
├── sincronizar.sh              ← script de sincronización
└── .sync_curso.log             ← log del script
```

_(Notas: por qué dos directorios, qué hace cada uno)_

---

## Crear la estructura

```bash
# Directorio para la sincronización
mkdir -p ~/sync_curso/mi_proyecto

# Estructura del proyecto Launch Control
mkdir -p ~/launch-control/{config,scripts,logs,data,docs}

# Verificar sync_curso
echo "=== sync_curso ==="
ls ~/sync_curso/

# Verificar launch-control
echo ""
echo "=== launch-control ==="
find ~/launch-control -type d | sort
```

---

## Mover archivos existentes

```bash
# Scripts en el escritorio
mv ~/Escritorio/*.sh ~/launch-control/scripts/ 2>/dev/null
mv ~/Desktop/*.sh ~/launch-control/scripts/ 2>/dev/null

# Configs sueltas en /tmp
mv /tmp/*.conf ~/launch-control/config/ 2>/dev/null

# Verificar qué quedó
echo "=== Archivos en launch-control ==="
find ~/launch-control -type f | sort
```

---

## Archivos de configuración base

### database.conf

```bash
cat > ~/launch-control/config/database.conf << 'EOF'
# ═══════════════════════════════════════════════════
# Launch Control — Configuración de Base de Datos
# ═══════════════════════════════════════════════════
# Estos valores se usan cuando el proyecto conecta
# a PostgreSQL. Por ahora son valores de desarrollo.
#
# La contraseña NO va aquí — va en .env
# ═══════════════════════════════════════════════════
DB_HOST=localhost
DB_PORT=5432
DB_NAME=launch_control
DB_USER=lc_admin
# DB_PASS se define en .env (nunca en archivos de config)
EOF
```

### network.conf

```bash
cat > ~/launch-control/config/network.conf << 'EOF'
# ═══════════════════════════════════════════════════
# Launch Control — Configuración de Red
# ═══════════════════════════════════════════════════
# Puertos donde escuchan los servicios del proyecto.
# Estos valores se ajustan según el entorno.
# ═══════════════════════════════════════════════════
LC_HTTP_PORT=8080
LC_API_PORT=3000
LC_BIND_ADDRESS=0.0.0.0
EOF
```

### Verificar

```bash
echo "=== Archivos de configuración ==="
ls -la ~/launch-control/config/
echo ""
echo "=== Contenido database.conf ==="
cat ~/launch-control/config/database.conf
echo ""
echo "=== Contenido network.conf ==="
cat ~/launch-control/config/network.conf
```

---

## El archivo .env — variables sensibles

_(Notas: por qué se separa, qué pasa si alguien lo ve)_

```bash
cat > ~/launch-control/.env << 'EOF'
# ═══════════════════════════════════════════════════
# Launch Control — Variables de Entorno (SENSIBLE)
# ═══════════════════════════════════════════════════
# Este archivo contiene credenciales.
# NUNCA subirlo a Git.
# NUNCA compartirlo.
# Permisos: 600 (solo el dueño puede leerlo)
# ═══════════════════════════════════════════════════
DB_PASS=launch2026_secure
API_KEY=lc-key-demo-12345
SECRET_TOKEN=artemis-7f8e9d
EOF
```

### Proteger .env

```bash
# Establecer permisos: solo el dueño puede leer y escribir
chmod 600 ~/launch-control/.env

# Verificar
ls -la ~/launch-control/.env
```

_(Notas: qué significa 600 vs 644, por qué importa)_

### Demostración del efecto

```bash
# Ver los permisos actuales
stat -c "%a %U:%G %n" ~/launch-control/.env

# Comparar con database.conf (permisos normales)
stat -c "%a %U:%G %n" ~/launch-control/config/database.conf
```

---

## health_check.sh — el primer script del proyecto

```bash
cat > ~/launch-control/scripts/health_check.sh << 'SCRIPT'
#!/bin/bash
# ═══════════════════════════════════════════════════
# Launch Control — Health Check v2
# Verifica que la estructura del proyecto está completa
# ═══════════════════════════════════════════════════

echo ""
echo "╔══════════════════════════════════════════════╗"
echo "║  LAUNCH CONTROL — HEALTH CHECK              ║"
echo "║  $(date '+%Y-%m-%d %H:%M:%S')                       ║"
echo "╠══════════════════════════════════════════════╣"

PASS=0
FAIL=0

check() {
    local desc="$1"
    local result="$2"
    if [ "$result" = "OK" ]; then
        echo "║  [OK]  $desc"
        PASS=$((PASS + 1))
    else
        echo "║  [!!]  $desc — $result"
        FAIL=$((FAIL + 1))
    fi
}

echo "║                                              ║"
echo "║  --- DIRECTORIOS ---                         ║"

DIRS="config scripts logs data docs"
for dir in $DIRS; do
    if [ -d "$HOME/launch-control/$dir" ]; then
        check "$dir/" "OK"
    else
        check "$dir/" "NO ENCONTRADO"
    fi
done

echo "║                                              ║"
echo "║  --- CONFIGURACIÓN ---                       ║"

for conf in database.conf network.conf; do
    if [ -f "$HOME/launch-control/config/$conf" ]; then
        check "config/$conf" "OK"
    else
        check "config/$conf" "FALTA"
    fi
done

echo "║                                              ║"
echo "║  --- SEGURIDAD ---                           ║"

if [ -f "$HOME/launch-control/.env" ]; then
    PERMS=$(stat -c %a "$HOME/launch-control/.env")
    if [ "$PERMS" = "600" ]; then
        check ".env (permisos: $PERMS)" "OK"
    else
        check ".env permisos" "INSEGURO: $PERMS (debe ser 600)"
    fi
else
    check ".env" "NO ENCONTRADO"
fi

echo "║                                              ║"
echo "║  --- SINCRONIZACIÓN ---                      ║"

if [ -d "$HOME/sync_curso" ]; then
    check "sync_curso/" "OK"
else
    check "sync_curso/" "NO ENCONTRADO"
fi

if [ -f "$HOME/sincronizar.sh" ] && [ -x "$HOME/sincronizar.sh" ]; then
    check "sincronizar.sh (ejecutable)" "OK"
elif [ -f "$HOME/sincronizar.sh" ]; then
    check "sincronizar.sh" "SIN PERMISOS DE EJECUCIÓN"
else
    check "sincronizar.sh" "NO ENCONTRADO (se crea en el Bloque 2)"
fi

echo "║                                              ║"
echo "╠══════════════════════════════════════════════╣"
echo "║  RESULTADO: $PASS aprobados, $FAIL pendientes"

if [ $FAIL -eq 0 ]; then
    echo "║  TODOS LOS SISTEMAS OPERATIVOS              ║"
else
    echo "║  HAY ITEMS PENDIENTES — revisar [!!]         ║"
fi
echo "╚══════════════════════════════════════════════╝"
echo ""
SCRIPT

chmod 755 ~/launch-control/scripts/health_check.sh
```

### Ejecutar

```bash
bash ~/launch-control/scripts/health_check.sh
```

---

# BLOQUE 2 — SINCRONIZACIÓN DE ARCHIVOS

---

## Cómo funciona

_(Notas: por qué no se trabaja directamente en /media/sf_*)_

```
WINDOWS (Host)
  └── Carpeta cualquiera
        │
        ▼  VirtualBox → Configuración → Carpetas compartidas
        │
LINUX (VM)
  └── /media/sf_mi_proyecto/          ← montaje automático (root:vboxsf)
        │                               NO se puede escribir aquí
        ▼  rsync -a --delete
        │
  └── ~/sync_curso/mi_proyecto/       ← copia local (TU usuario)
                                        AQUÍ se trabaja
```

| Componente | Función |
|---|---|
| Carpeta compartida | VBox monta un directorio de Windows en Linux |
| rsync | Copia inteligente que mantiene un espejo exacto |
| `--delete` | Si se borra en origen, se borra en destino |

---

## Paso 1 — Configurar la carpeta compartida en VirtualBox

_(Notas: cómo se configura en VirtualBox)_

| Campo | Valor |
|---|---|
| Ruta carpeta | La ruta completa en Windows |
| Nombre carpeta | Un nombre corto sin espacios |
| Solo lectura | Sí |
| Automontar | Sí |
| Punto de montaje | Vacío (automático: `/media/sf_<nombre>`) |

---

## Paso 2 — Permisos de acceso

```bash
# Añadir el usuario al grupo vboxsf
sudo usermod -aG vboxsf $USER

# Activar sin cerrar sesión (o cerrar sesión y volver a entrar)
newgrp vboxsf

# Verificar que el grupo está activo
groups

# Verificar acceso a la carpeta compartida
ls /media/sf_mi_proyecto/
```

_(Notas: qué hacer si dice Permission denied)_

---

## Paso 3 — rsync

```bash
# Instalar (normalmente ya viene en Ubuntu 24.04)
sudo apt install rsync -y
rsync --version | head -1
```

### Prueba manual

```bash
rsync -av --delete \
  /media/sf_mi_proyecto/ \
  ~/sync_curso/mi_proyecto/
```

| Flag | Qué hace |
|---|---|
| `-a` | Archive: preserva permisos, timestamps, links simbólicos |
| `-v` | Verbose: muestra qué archivos se copian |
| `--delete` | Espejo: borra del destino lo que ya no existe en el origen |

_(Notas: por qué importa la barra final /)_

### Verificar

```bash
# Contar archivos en origen y destino (deben coincidir)
echo "Origen:  $(find /media/sf_mi_proyecto/ -type f 2>/dev/null | wc -l) archivos"
echo "Destino: $(find ~/sync_curso/mi_proyecto/ -type f | wc -l) archivos"

# Verificar que los archivos son del usuario, no de root
ls -la ~/sync_curso/mi_proyecto/ | head -5
```

---

## Paso 4 — Script de sincronización automática

```bash
cat > ~/sincronizar.sh << 'SCRIPT'
#!/bin/bash
# ═══════════════════════════════════════════════════════════
# sincronizar.sh — Sincronización de carpeta compartida
# Espejo unidireccional: VirtualBox shared folder → copia local
#
# QUÉ HACE:
#   1. Espera a que VirtualBox monte la carpeta compartida
#   2. Verifica que la fuente existe
#   3. Copia todo con rsync en modo espejo (--delete)
#   4. Registra el resultado en un log
#
# CONFIGURACIÓN:
#   Cambiar NOMBRE_COMPARTIDO por el nombre de la carpeta
#   compartida configurada en VirtualBox.
# ═══════════════════════════════════════════════════════════

# ─── CAMBIAR ESTE VALOR ────────────────────────────────────
NOMBRE_COMPARTIDO="mi_proyecto"
# ───────────────────────────────────────────────────────────

FUENTE="/media/sf_${NOMBRE_COMPARTIDO}/"
DESTINO="$HOME/sync_curso/${NOMBRE_COMPARTIDO}/"
LOG="$HOME/.sync_curso.log"

# Si se ejecuta al arranque, esperar a que VirtualBox monte
if [ ! -d "$FUENTE" ]; then
    echo "Esperando a que se monte la carpeta compartida..."
    sleep 5
fi

# Verificar que la fuente existe y está montada
if [ ! -d "$FUENTE" ]; then
    MSG="[$(date '+%Y-%m-%d %H:%M:%S')] ERROR: $FUENTE no está montada"
    echo "$MSG" >> "$LOG"
    echo "$MSG"
    echo ""
    echo "  Posibles causas:"
    echo "  - La carpeta compartida no está configurada en VirtualBox"
    echo "  - El disco donde está la carpeta no está conectado"
    echo "  - Verificar en VBox → Configuración → Carpetas compartidas"
    notify-send "Sincronización" "ERROR: carpeta compartida no montada" 2>/dev/null
    exit 1
fi

# Crear directorio destino si no existe
mkdir -p "$DESTINO"

# Sincronizar en modo espejo
rsync -a --delete "$FUENTE" "$DESTINO" >> "$LOG" 2>&1

# Registrar resultado
if [ $? -eq 0 ]; then
    ARCHIVOS=$(find "$DESTINO" -type f | wc -l)
    MSG="[$(date '+%Y-%m-%d %H:%M:%S')] Sincronización exitosa — $ARCHIVOS archivos"
    echo "$MSG" >> "$LOG"
    echo "$MSG"
    notify-send "Sincronización" "Completada: $ARCHIVOS archivos" 2>/dev/null
else
    MSG="[$(date '+%Y-%m-%d %H:%M:%S')] ERROR en sincronización"
    echo "$MSG" >> "$LOG"
    echo "$MSG"
    notify-send "Sincronización" "ERROR — revisar ~/.sync_curso.log" 2>/dev/null
    exit 1
fi
SCRIPT

chmod 755 ~/sincronizar.sh
```

### Probar

```bash
bash ~/sincronizar.sh
cat ~/.sync_curso.log
```

---

## Paso 5 — Ejecución automática al iniciar sesión

```bash
mkdir -p ~/.config/autostart/

cat > ~/.config/autostart/sincronizar.desktop << EOF
[Desktop Entry]
Type=Application
Name=Sincronizar carpeta compartida
Exec=/bin/bash $HOME/sincronizar.sh
Hidden=false
NoDisplay=false
X-GNOME-Autostart-enabled=true
Comment=Sincroniza carpeta compartida de VirtualBox al iniciar sesión
EOF
```

### Verificar

```bash
cat ~/.config/autostart/sincronizar.desktop
# La línea Exec= debe mostrar la ruta completa real
```

---

## Paso 6 — Alias para sincronización bajo demanda

```bash
echo 'alias sync-curso="bash ~/sincronizar.sh"' >> ~/.bashrc
source ~/.bashrc

# Probar:
sync-curso
```

---

## ¿Qué pasa si la ruta de Windows cambia?

_(Notas: dónde se cambia, por qué el script no necesita cambios)_

---

# BLOQUE 3 — VERIFICACIÓN FINAL

---

## Ejecutar health check actualizado

```bash
bash ~/launch-control/scripts/health_check.sh
```

---

## Verificación completa del entorno

```bash
cat > /tmp/verificacion_completa.sh << 'SCRIPT'
#!/bin/bash
# ═══════════════════════════════════════════════════════════
# VERIFICACIÓN COMPLETA — Entorno de Desarrollo
# Verifica: restauración + estructura + sync + gráficos + IDE
# ═══════════════════════════════════════════════════════════

echo ""
echo "╔══════════════════════════════════════════════════════════════╗"
echo "║  VERIFICACIÓN COMPLETA — ENTORNO DE DESARROLLO              ║"
echo "║  $(date '+%Y-%m-%d %H:%M:%S')                                       ║"
echo "╠══════════════════════════════════════════════════════════════╣"

PASS=0
FAIL=0

check() {
    local desc="$1"
    local result="$2"
    if [ "$result" = "OK" ]; then
        echo "  [OK]  $desc"
        PASS=$((PASS + 1))
    else
        echo "  [!!]  $desc — $result"
        FAIL=$((FAIL + 1))
    fi
}

echo ""
echo "  --- RESTAURACIÓN (día 21) ---"
command -v timeshift &>/dev/null && check "Timeshift instalado" "OK" || check "Timeshift instalado" "No encontrado"
sudo timeshift --list 2>/dev/null | grep -q "20" && check "Timeshift tiene snapshots" "OK" || check "Timeshift tiene snapshots" "Sin snapshots"

echo ""
echo "  --- GRÁFICOS (día 21) ---"
SESSION=$(echo $XDG_SESSION_TYPE)
[ "$SESSION" = "x11" ] && check "Sesión X11" "OK" || check "Sesión gráfica" "Es $SESSION (debe ser x11)"
grep -q "^WaylandEnable=false" /etc/gdm3/custom.conf 2>/dev/null && check "Wayland desactivado" "OK" || check "Wayland" "No desactivado"
[ -f "/etc/X11/xorg.conf.d/20-vmware.conf" ] && check "TearFree" "OK" || check "TearFree" "No configurado"

echo ""
echo "  --- IDE (día 21) ---"
dpkg -l 2>/dev/null | grep -q antigravity && check "Antigravity instalado" "OK" || check "Antigravity" "No encontrado"
grep -q "alias antigravity" "$HOME/.bashrc" 2>/dev/null && check "Alias antigravity" "OK" || check "Alias antigravity" "No configurado"

echo ""
echo "  --- ESTRUCTURA (día 22) ---"
for dir in config scripts logs data docs; do
    [ -d "$HOME/launch-control/$dir" ] && check "launch-control/$dir/" "OK" || check "launch-control/$dir/" "No encontrado"
done
for conf in database.conf network.conf; do
    [ -f "$HOME/launch-control/config/$conf" ] && check "config/$conf" "OK" || check "config/$conf" "Falta"
done
if [ -f "$HOME/launch-control/.env" ]; then
    PERMS=$(stat -c %a "$HOME/launch-control/.env")
    [ "$PERMS" = "600" ] && check ".env permisos (600)" "OK" || check ".env permisos" "Tiene $PERMS (debe ser 600)"
else
    check ".env" "No encontrado"
fi
[ -f "$HOME/launch-control/scripts/health_check.sh" ] && check "health_check.sh" "OK" || check "health_check.sh" "No encontrado"

echo ""
echo "  --- SINCRONIZACIÓN (día 22) ---"
[ -d "$HOME/sync_curso" ] && check "sync_curso/" "OK" || check "sync_curso/" "No encontrado"
groups 2>/dev/null | grep -q vboxsf && check "Grupo vboxsf" "OK" || check "Grupo vboxsf" "No está en el grupo"
mount | grep -q "sf_" && check "Carpeta compartida montada" "OK" || check "Carpeta compartida" "No montada"
command -v rsync &>/dev/null && check "rsync instalado" "OK" || check "rsync" "No encontrado"
[ -f "$HOME/sincronizar.sh" ] && [ -x "$HOME/sincronizar.sh" ] && check "sincronizar.sh" "OK" || check "sincronizar.sh" "No encontrado"
[ -f "$HOME/.config/autostart/sincronizar.desktop" ] && check "Autostart" "OK" || check "Autostart" "No configurado"
grep -q "alias sync-curso" "$HOME/.bashrc" 2>/dev/null && check "Alias sync-curso" "OK" || check "Alias sync-curso" "No configurado"

echo ""
echo "╠══════════════════════════════════════════════════════════════╣"
echo "║  RESULTADO: $PASS aprobados, $FAIL con problemas"
if [ $FAIL -eq 0 ]; then
    echo "║  ENTORNO COMPLETO — TOMAR SNAPSHOT GOLDEN v2               ║"
else
    echo "║  REVISAR ITEMS [!!] ANTES DEL SNAPSHOT                     ║"
fi
echo "╚══════════════════════════════════════════════════════════════╝"
echo ""
SCRIPT

chmod +x /tmp/verificacion_completa.sh
bash /tmp/verificacion_completa.sh
```

---

## Snapshot GOLDEN v2

_(Notas: por qué guardar snapshot aquí, cuándo restaurarlo)_

### Desde Linux:

```bash
sudo timeshift --create --comments "GOLDEN v2: Estructura + Sync — 22-04-26"
```

### Desde Windows:

1. Menú **Máquina** → **Tomar instantánea** (`Ctrl+Shift+S`)
2. Nombre: `GOLDEN v2 — Estructura + Sync — 22-04-26`
3. Descripción: `Estructura launch-control, configs, .env, rsync, autostart, health_check`

---

# REFERENCIA RÁPIDA

---

## Comandos de diagnóstico

| Qué verificar | Comando |
|---|---|
| Estructura del proyecto | `find ~/launch-control -type d \| sort` |
| Permisos de .env | `stat -c "%a" ~/launch-control/.env` |
| Health check | `bash ~/launch-control/scripts/health_check.sh` |
| Grupo vboxsf | `groups \| grep vboxsf` |
| Carpeta compartida montada | `mount \| grep sf_` |
| Sincronizar ahora | `sync-curso` |
| Última sincronización | `tail -5 ~/.sync_curso.log` |
| Snapshots Timeshift | `sudo timeshift --list` |
| Sesión gráfica | `echo $XDG_SESSION_TYPE` |
| IDE instalado | `dpkg -l \| grep antigravity` |

## Problemas frecuentes

| Problema | Causa | Solución |
|---|---|---|
| `Permission denied` en `/media/sf_*` | No está en grupo `vboxsf` | `sudo usermod -aG vboxsf $USER` + cerrar sesión |
| Carpeta compartida no aparece | Guest Additions faltan | `sudo apt install virtualbox-guest-utils virtualbox-guest-x11 -y` + reiniciar |
| rsync no encontrado | No instalado | `sudo apt install rsync -y` |
| sincronizar.sh dice "no montada" | Carpeta no configurada | Verificar VBox → Carpetas compartidas |
| .env tiene permisos 644 | chmod no ejecutado | `chmod 600 ~/launch-control/.env` |
| health_check dice [!!] | Falta algún componente | Leer qué item falló y crearlo |
| Autostart no funciona | Ruta incorrecta en .desktop | Verificar `Exec=` en el .desktop |
| Archivos sincronizados son de root | rsync con sudo | Ejecutar rsync SIN sudo |

## Todos los aliases (días 21 + 22)

```bash
# Día 21 — IDE
alias antigravity='google-chrome --ozone-platform=x11 --ignore-gpu-blocklist --enable-gpu-rasterization'

# Día 22 — Sincronización
alias sync-curso='bash ~/sincronizar.sh'
```

## Estructura final del home

```
~/
├── launch-control/             ← Proyecto de trabajo
│   ├── config/
│   │   ├── database.conf
│   │   └── network.conf
│   ├── scripts/
│   │   └── health_check.sh
│   ├── logs/
│   ├── data/
│   ├── docs/
│   └── .env                    ← (permisos 600)
│
├── sync_curso/
│   └── mi_proyecto/            ← Espejo de Windows
│
├── sincronizar.sh
├── .sync_curso.log
│
└── .config/autostart/
    └── sincronizar.desktop
```

---

# AUTOEVALUACIÓN

---

**1.** ¿Qué comando crea varios directorios a la vez usando llaves?

**2.** ¿Qué permisos numéricos debe tener el archivo `.env` y por qué?

**3.** ¿Cuál es la diferencia entre `644` y `600`?

**4.** ¿Qué flag de rsync hace que se borren del destino los archivos que ya no existen en el origen?

**5.** ¿Por qué rsync copia a `~/sync_curso/` en vez de trabajar directamente en `/media/sf_*`?

**6.** ¿Qué variable del script `sincronizar.sh` hay que cambiar para usar otra carpeta compartida?

**7.** ¿Qué archivo se crea en `~/.config/autostart/` para que la sincronización se ejecute al abrir sesión?

**8.** ¿Qué hace el script `health_check.sh`?

**9.** ¿Por qué usamos `~/sync_curso/` en vez de `~/Documentos/`?

**10.** Después de modificar `~/.bashrc`, ¿qué comando ejecutan para activar los cambios sin cerrar la terminal?

---

### Respuestas

**1.** `mkdir -p ~/launch-control/{config,scripts,logs,data,docs}` — `-p` crea padres, `{}` expande a múltiples nombres.

**2.** `600` (rw-------). Contiene contraseñas — solo el dueño debe poder leerlo.

**3.** `644` = todos leen, solo dueño escribe. `600` = solo dueño lee y escribe.

**4.** `--delete`. Sin este flag, rsync solo agrega/actualiza pero no borra.

**5.** Porque `/media/sf_*` tiene permisos `root:vboxsf`. La copia en `~/sync_curso/` tiene permisos propios.

**6.** `NOMBRE_COMPARTIDO` al inicio del script.

**7.** `sincronizar.desktop` con la ruta al script en `Exec=`.

**8.** Verifica estructura completa: directorios, configs, permisos .env, script sync. Muestra [OK]/[!!].

**9.** Porque `Documentos` se llama `Documents` en inglés. `sync_curso` funciona en cualquier idioma.

**10.** `source ~/.bashrc`

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
> El contenido técnico de Linux, VirtualBox, rsync y sus ecosistemas pertenece
> a sus respectivos autores y comunidades; la organización pedagógica, las
> explicaciones, los diagramas y el enfoque metodológico son obra original del autor.
>
> **Hash de Certificación:** `4e8d9b1a5f6e7c3d2b1a0f9e8d7c6b5a4f3e2d1c0b9a8f7e6d5c4b3a2f1e0d9c`
