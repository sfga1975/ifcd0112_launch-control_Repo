# Apuntes — 21 de Abril 2026: Configuración del Entorno de Desarrollo

**Curso IFCD0112** | **Módulo:** MF0223_3 | Prof. Juan Marcelo Gutiérrez Miranda

Estos apuntes son la referencia rápida de todo lo que se hizo hoy.
Tienen los comandos, rutas y configuraciones exactas. Consulten
este documento cada vez que necesiten repetir o verificar algo.

---

# RECONEXIÓN

---

Ayer trabajamos con **usuarios, grupos, procesos y señales**:

| Tema | Lo esencial |
|------|-------------|
| Usuarios | `useradd`, `usermod`, `passwd`, `/etc/passwd`, `/etc/shadow` |
| Grupos | `groupadd`, `usermod -aG`, `/etc/group` |
| Procesos | `ps aux`, `top`, `htop`, `kill`, `nice`, `renice` |
| Señales | `kill -SIGTERM`, `kill -SIGKILL`, `kill -SIGSTOP`, `kill -SIGCONT` |

Verificación rápida — si esto funciona, están al día:

```bash
id                          # muestra su usuario, uid, grupos
ps aux | wc -l              # cantidad de procesos activos
groups $USER                # grupos a los que pertenecen
```

Si algo falla, revisen los apuntes del día 20-04-26 antes de continuar.

---

# MAPA DEL DÍA

---

Hoy configuramos el entorno completo de trabajo. La secuencia fue:

```
╔═══════════════════════════════════════════════════════════════════╗
║     CONFIGURACIÓN DEL ENTORNO DE DESARROLLO — 21 ABR 2026        ║
╠═══════════════════════════════════════════════════════════════════╣
║                                                                   ║
║  1. Snapshots       Punto de restauración antes de tocar nada     ║
║  2. Carpetas        Estructura de ~/launch-control/ y ~/sync_curso║
║  3. Sincronización  Carpeta compartida VBox → rsync → copia local ║
║  4. Gráficos        X11 forzado + VRAM 256 MB + TearFree         ║
║  5. Antigravity     IDE instalado con flags de optimización       ║
║  6. Autostart       Script + alias para sync automático           ║
║                                                                   ║
║  Resultado: entorno 100% funcional para programar desde mañana   ║
╚═══════════════════════════════════════════════════════════════════╝
```

---

# REFERENCIA RÁPIDA: SNAPSHOTS

---

Los snapshots son fotos del estado completo de la VM.
Si algo se rompe, restauran y vuelven al punto exacto.

## VirtualBox (desde Windows)

| Acción | Cómo |
|--------|------|
| Crear snapshot | Menú Máquina → Tomar instantánea (`Ctrl+Shift+S`) |
| Restaurar | Pestaña Instantáneas → clic derecho → Restaurar |
| Eliminar | Clic derecho → Eliminar (libera espacio) |

## VirtualBox (línea de comandos — desde el host)

```powershell
# Listar snapshots existentes
VBoxManage snapshot "NombreVM" list

# Crear snapshot con descripción
VBoxManage snapshot "NombreVM" take "pre-config" --description "Antes de configurar entorno"

# Restaurar al snapshot
VBoxManage snapshot "NombreVM" restore "pre-config"

# Eliminar snapshot
VBoxManage snapshot "NombreVM" delete "pre-config"
```

## Timeshift (dentro de la VM — nivel sistema)

```bash
# Instalar
sudo apt install timeshift -y

# Crear snapshot manual
sudo timeshift --create --comments "Antes de configurar entorno"

# Listar snapshots
sudo timeshift --list

# Restaurar el más reciente
sudo timeshift --restore

# Eliminar snapshot específico
sudo timeshift --delete --snapshot '2026-04-21_09-00-00'
```

| | Snapshot VirtualBox | Timeshift |
|---|---|---|
| **Desde dónde** | Windows (VirtualBox Manager) | Dentro de Linux |
| **Qué captura** | Todo: disco, RAM, CPU, estado completo | Sistema de archivos (sin /home) |
| **Velocidad** | Instantáneo | Requiere reinicio |
| **Cuándo usarlo** | Antes de cambios grandes | Para experimentar dentro de Linux |

**Regla del día:** Siempre crear snapshot ANTES de instalar software o modificar configuraciones del sistema.

---

# REFERENCIA RÁPIDA: ESTRUCTURA DE CARPETAS

---

```
~/
├── sync_curso/
│   └── mi_proyecto/            ← espejo de la carpeta compartida de Windows
│       ├── subcarpetas...
│       └── archivos...
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

## Crear la estructura

```bash
# Directorio para la sincronización
mkdir -p ~/sync_curso/mi_proyecto

# Estructura del proyecto Launch Control
mkdir -p ~/launch-control/{config,scripts,logs,data,docs}

# Verificar
echo "=== sync_curso ==="
ls ~/sync_curso/
echo ""
echo "=== launch-control ==="
find ~/launch-control -type d | sort
```

## Archivos de configuración base

```bash
# Configuración de base de datos
cat > ~/launch-control/config/database.conf << 'EOF'
# ═══════════════════════════════════════════════════
# Launch Control — Configuración de Base de Datos
# ═══════════════════════════════════════════════════
DB_HOST=localhost
DB_PORT=5432
DB_NAME=launch_control
DB_USER=lc_admin
# DB_PASS se define en .env (nunca en archivos de config)
EOF

# Configuración de red
cat > ~/launch-control/config/network.conf << 'EOF'
# ═══════════════════════════════════════════════════
# Launch Control — Configuración de Red
# ═══════════════════════════════════════════════════
LC_HTTP_PORT=8080
LC_API_PORT=3000
LC_BIND_ADDRESS=0.0.0.0
EOF

# Variables de entorno (archivo sensible)
cat > ~/launch-control/.env << 'EOF'
# ═══════════════════════════════════════════════════
# Launch Control — Variables de Entorno (SENSIBLE)
# ═══════════════════════════════════════════════════
DB_PASS=launch2026_secure
API_KEY=lc-key-demo-12345
SECRET_TOKEN=artemis-7f8e9d
EOF

# Proteger .env — solo el dueño puede leerlo
chmod 600 ~/launch-control/.env

# Verificar permisos
ls -la ~/launch-control/.env
# Debe mostrar: -rw------- (600)
```

**¿Por qué `~/sync_curso/` y no `~/Documentos/`?** Porque el directorio
`Documentos` se llama `Documents` si Ubuntu está en inglés y `Documentos`
si está en español. Usar un directorio propio (`sync_curso`) funciona
igual en cualquier máquina, independiente del idioma del sistema.

---

# REFERENCIA RÁPIDA: SINCRONIZACIÓN

---

## El flujo completo

```
WINDOWS (Host)
  └── Carpeta cualquiera (ej: C:\Users\alumno\mi_proyecto)
        │
        ▼  VirtualBox → Configuración → Carpetas compartidas
        │  (Nombre: "mi_proyecto", Automontar: Sí)
        │
LINUX (VM)
  └── /media/sf_mi_proyecto/          ← montaje automático (root:vboxsf)
        │                               NO se puede escribir aquí directamente
        ▼  rsync -a --delete
        │
  └── ~/sync_curso/mi_proyecto/       ← copia local (TU usuario, permisos completos)
                                        AQUÍ se trabaja
```

## Paso 1 — Configurar la carpeta compartida en VirtualBox

Desde Windows:

| Campo | Valor |
|-------|-------|
| Ruta carpeta | La ruta completa en Windows |
| Nombre carpeta | Un nombre corto sin espacios (ej: `mi_proyecto`) |
| Solo lectura | Sí |
| Automontar | Sí |
| Punto de montaje | Dejar vacío (automático: `/media/sf_<nombre>`) |

## Paso 2 — Permisos de acceso

```bash
# Agregar el usuario al grupo vboxsf
sudo usermod -aG vboxsf $USER

# Cerrar sesión y volver a entrar (o reiniciar)
# Después verificar:
groups
# Debe aparecer "vboxsf" en la lista

# Verificar acceso
ls /media/sf_mi_proyecto/
```

Si dice "Permission denied": cerrar sesión **completa** (logout del sistema, no solo cerrar terminal) y volver a entrar.

## Paso 3 — rsync manual

```bash
# Instalar (normalmente ya viene en Ubuntu 24.04)
sudo apt install rsync -y

# Prueba manual
rsync -av --delete /media/sf_mi_proyecto/ ~/sync_curso/mi_proyecto/

# Verificar conteo de archivos
echo "Origen:  $(find /media/sf_mi_proyecto/ -type f 2>/dev/null | wc -l)"
echo "Destino: $(find ~/sync_curso/mi_proyecto/ -type f | wc -l)"
```

| Flag | Qué hace |
|------|----------|
| `-a` | Archive: preserva permisos, timestamps, links simbólicos |
| `-v` | Verbose: muestra qué archivos se copian |
| `--delete` | Espejo: borra del destino lo que ya no existe en el origen |

**La barra final `/` importa:** Con barra, rsync copia el **contenido**.
Sin barra, copia el **directorio en sí** (creando un nivel extra).

## Paso 4 — Script de sincronización automática

```bash
cat > ~/sincronizar.sh << 'SCRIPT'
#!/bin/bash
# sincronizar.sh — Espejo de carpeta compartida VirtualBox → copia local

# ─── CAMBIAR ESTE VALOR ────────────────────────────────────
NOMBRE_COMPARTIDO="mi_proyecto"
# ───────────────────────────────────────────────────────────

FUENTE="/media/sf_${NOMBRE_COMPARTIDO}/"
DESTINO="$HOME/sync_curso/${NOMBRE_COMPARTIDO}/"
LOG="$HOME/.sync_curso.log"

if [ ! -d "$FUENTE" ]; then sleep 5; fi

if [ ! -d "$FUENTE" ]; then
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] ERROR: $FUENTE no está montada" >> "$LOG"
    notify-send "Sincronización" "ERROR: carpeta no montada" 2>/dev/null
    exit 1
fi

mkdir -p "$DESTINO"
rsync -a --delete "$FUENTE" "$DESTINO" >> "$LOG" 2>&1

if [ $? -eq 0 ]; then
    ARCHIVOS=$(find "$DESTINO" -type f | wc -l)
    MSG="[$(date '+%Y-%m-%d %H:%M:%S')] Sincronización exitosa — $ARCHIVOS archivos"
    echo "$MSG" >> "$LOG"
    echo "$MSG"
    notify-send "Sincronización" "Completada: $ARCHIVOS archivos" 2>/dev/null
else
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] ERROR en sincronización" >> "$LOG"
    notify-send "Sincronización" "ERROR — revisar ~/.sync_curso.log" 2>/dev/null
    exit 1
fi
SCRIPT

chmod 755 ~/sincronizar.sh
```

Probar:

```bash
bash ~/sincronizar.sh
cat ~/.sync_curso.log
```

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
EOF
```

## Paso 6 — Alias para sincronización manual

```bash
echo 'alias sync-curso="bash ~/sincronizar.sh"' >> ~/.bashrc
source ~/.bashrc

# Probar:
sync-curso
```

## ¿Qué pasa si la ruta de Windows cambia?

Solo hay que actualizar la ruta en VirtualBox → Configuración →
Carpetas compartidas. El script de Linux no necesita cambios porque
trabaja con `/media/sf_mi_proyecto/`, que no depende de la ruta
de Windows.

---

# REFERENCIA RÁPIDA: GRÁFICOS

---

## Verificar servidor gráfico

```bash
echo $XDG_SESSION_TYPE
# x11  → correcto para VirtualBox
# wayland → hay que cambiarlo
```

## Forzar X11

```bash
sudo nano /etc/gdm3/custom.conf
# Buscar: #WaylandEnable=false
# Quitar el #, que quede: WaylandEnable=false
# Guardar: Ctrl+O, Enter, Ctrl+X
sudo reboot
```

## Aumentar VRAM (desde Windows, VM apagada)

```powershell
cd "C:\Program Files\Oracle\VirtualBox"
.\VBoxManage modifyvm "NombreVM" --vram 256
```

En VirtualBox → Configuración → Pantalla:
- Controlador gráfico: **VMSVGA**
- Habilitar aceleración 3D: **Sí**
- Memoria de video: **256 MB**

## TearFree (antiparpadeo)

```bash
sudo mkdir -p /etc/X11/xorg.conf.d/

sudo bash -c 'cat << EOF > /etc/X11/xorg.conf.d/20-vmware.conf
Section "Device"
    Identifier "vmware"
    Driver "vmware"
    Option "TearFree" "True"
EndSection
EOF'

sudo reboot
```

## Verificar después del reinicio

```bash
echo $XDG_SESSION_TYPE    # Debe decir: x11
grep -n "Wayland" /etc/gdm3/custom.conf  # WaylandEnable=false (sin #)
```

| Configuración | Valor |
|---------------|-------|
| VRAM | 256 MB |
| Controlador gráfico | VMSVGA |
| Aceleración 3D | Habilitada |
| Servidor gráfico | X11 (no Wayland) |
| TearFree | Configurado |

---

# REFERENCIA RÁPIDA: ANTIGRAVITY IDE

---

## Instalación

```bash
# Dependencias
sudo apt update
sudo apt install curl gpg -y

# Llave de firma del repositorio
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://us-central1-apt.pkg.dev/doc/repo-signing-key.gpg \
  | sudo gpg --dearmor --yes -o /etc/apt/keyrings/antigravity-repo-key.gpg

# Agregar repositorio
echo "deb [signed-by=/etc/apt/keyrings/antigravity-repo-key.gpg] https://us-central1-apt.pkg.dev/projects/antigravity-auto-updater-dev/ antigravity-debian main" \
  | sudo tee /etc/apt/sources.list.d/antigravity.list > /dev/null

# Instalar
sudo apt update
sudo apt install antigravity -y

# Verificar
dpkg -l | grep antigravity
```

## Lanzamiento optimizado para VirtualBox

```bash
google-chrome --ozone-platform=x11 --ignore-gpu-blocklist --enable-gpu-rasterization
```

| Flag | Qué hace |
|------|----------|
| `--ozone-platform=x11` | Fuerza renderizado X11 en lugar de Wayland |
| `--ignore-gpu-blocklist` | Acepta la GPU virtual de VBox |
| `--enable-gpu-rasterization` | Texto e iconos más nítidos usando la GPU |

## Alias permanente

```bash
echo 'alias antigravity="google-chrome --ozone-platform=x11 --ignore-gpu-blocklist --enable-gpu-rasterization"' >> ~/.bashrc
source ~/.bashrc

# Lanzar:
antigravity
```

## Abrir proyecto en el IDE

```bash
antigravity ~/launch-control/
```

Dentro del IDE, abrir terminal integrado (`Ctrl+ñ`) y ejecutar:

```bash
bash scripts/health_check.sh
```

---

# TODOS LOS ALIASES CREADOS

---

```bash
# === NAVEGACIÓN RÁPIDA ===
alias lc='cd ~/launch-control'
alias lcs='cd ~/launch-control/scripts'
alias lcd='cd ~/launch-control/docs'

# === SINCRONIZACIÓN ===
alias sync-curso='bash ~/sincronizar.sh'

# === IDE ===
alias antigravity='google-chrome --ozone-platform=x11 --ignore-gpu-blocklist --enable-gpu-rasterization'

# === SISTEMA ===
alias actualizar='sudo apt update && sudo apt upgrade -y'
alias espacio='df -h | head -5'
alias memoria='free -h'
alias puertos='ss -tulnp'

# === SEGURIDAD ===
alias permisos='stat -c "%a %U:%G %n"'
alias quien='w'
```

Para cargar aliases desde un archivo separado, agregar al final de `~/.bashrc`:

```bash
if [ -f ~/launch-control/scripts/aliases.sh ]; then
    source ~/launch-control/scripts/aliases.sh
fi
```

Después de cualquier cambio:

```bash
source ~/.bashrc
```

---

# TROUBLESHOOTING

---

| # | Problema | Diagnóstico | Solución |
|---|----------|-------------|----------|
| 1 | La VM no arranca | Error de VT-x/AMD-V | Activar virtualización en BIOS/UEFI |
| 2 | Pantalla negra al iniciar VM | Servidor gráfico incompatible | Cambiar controlador a VMSVGA, VRAM 256 MB |
| 3 | Carpeta compartida no aparece | No está montada o falta grupo | `sudo usermod -aG vboxsf $USER` + cerrar sesión |
| 4 | rsync dice "permission denied" | Permisos en destino | Verificar con `ls -la` y ajustar con `chmod` |
| 5 | `rsync: command not found` | No está instalado | `sudo apt install rsync` |
| 6 | Antigravity no abre | Paquete no instalado o falta display | `dpkg -l \| grep antigravity` y `echo $DISPLAY` |
| 7 | IDE se ve borroso o lagea | VRAM baja o Wayland activo | VRAM 256 MB + forzar X11 |
| 8 | `source ~/.bashrc` no carga aliases | Error de sintaxis | `bash -n ~/.bashrc` para verificar sintaxis |
| 9 | Timeshift dice "no space" | Disco lleno | `df -h` para verificar, `sudo apt autoremove` |
| 10 | `mount: wrong fs type` | Guest Additions no instaladas | `sudo apt install virtualbox-guest-utils virtualbox-guest-x11` |
| 11 | sincronizar.sh dice "no montada" | Carpeta no configurada o disco no conectado | Verificar VBox → Carpetas compartidas |
| 12 | Archivos sincronizados son de root | rsync ejecutado con sudo | Ejecutar rsync SIN sudo |

---

# AUTOEVALUACIÓN

---

Respondan sin mirar los apuntes. Después verifiquen.

### Preguntas

**1.** ¿Cuál es el comando para crear un snapshot con Timeshift y agregarle un comentario?

**2.** ¿Qué flag de rsync permite simular la sincronización sin copiar nada?

**3.** ¿Cuál es la diferencia entre `rsync -avh` y `rsync -avh --delete`?

**4.** ¿Cómo verifican si están usando X11 o Wayland?

**5.** ¿Qué grupo deben tener para acceder a carpetas compartidas de VirtualBox?

**6.** ¿Por qué el script de sincronización copia de `/media/sf_*` a `~/sync_curso/` y no trabaja directamente en `/media/sf_*`?

**7.** ¿Qué valor de VRAM se recomienda para la VM?

**8.** ¿Qué archivo se edita para forzar X11 en lugar de Wayland?

**9.** ¿Qué variable del script `sincronizar.sh` hay que modificar para que funcione con otra carpeta compartida?

**10.** Si modifican `~/.bashrc`, ¿qué comando ejecutan para que los cambios tomen efecto sin cerrar la terminal?

---

### Respuestas

**1.**
```bash
sudo timeshift --create --comments "Descripción del snapshot"
```

**2.** La flag `-n` (dry-run). Ejemplo: `rsync -avhn origen/ destino/`

**3.** Sin `--delete`: solo copia archivos nuevos o modificados del origen al destino. Con `--delete`: además elimina del destino los archivos que ya no existen en el origen (sincronización espejo).

**4.**
```bash
echo $XDG_SESSION_TYPE
```
Devuelve `x11` o `wayland`.

**5.** El grupo `vboxsf`. Se agrega con: `sudo usermod -aG vboxsf $USER`

**6.** Porque `/media/sf_*` está montada con permisos de `root:vboxsf` y el usuario no puede escribir ahí directamente. La copia en `~/sync_curso/` tiene permisos del usuario y se puede trabajar sin restricciones.

**7.** 256 MB de VRAM.

**8.** `/etc/gdm3/custom.conf` — descomentar la línea `WaylandEnable=false`.

**9.** La variable `NOMBRE_COMPARTIDO` al inicio del script. Cambiar `"mi_proyecto"` por el nombre de la carpeta compartida en VirtualBox.

**10.**
```bash
source ~/.bashrc
```

---

# COMANDOS DE DIAGNÓSTICO

---

| Qué verificar | Comando |
|---|---|
| Grupo del usuario | `groups \| grep vboxsf` |
| Carpeta compartida montada | `mount \| grep sf_` |
| Última sincronización | `tail -5 ~/.sync_curso.log` |
| Snapshots de Timeshift | `sudo timeshift --list` |
| Sesión gráfica | `echo $XDG_SESSION_TYPE` |
| IDE instalado | `dpkg -l \| grep antigravity` |
| Alias disponible | `type antigravity` |
| Autostart | `cat ~/.config/autostart/sincronizar.desktop` |
| Driver gráfico | `grep -i driver /var/log/Xorg.0.log \| head -5` |
| VRAM | `grep -i vram /var/log/Xorg.0.log` |

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
