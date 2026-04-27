# Variables de Entorno, Scripting Bash y Gestión Kanban — APUNTES

**Certificado:** IFCD0112 | **Sesión 13** | Lunes 27 de abril de 2026
**Prof.** Juan Marcelo Gutiérrez Miranda | @TodoEconometria

---

## Reconexión

Verificación rápida:

```bash
ping -c 2 192.168.56.20
ssh alumno@192.168.56.20 hostname
ip addr show | grep 192.168.56
```

---

# BLOQUE I — Variables de Entorno

## ¿Cómo sabe Linux dónde están las cosas?

_(Tus notas aquí)_

```bash
# ¿Cuáles son las carpetas donde busca?
echo $PATH | tr ':' '\n'
```

```bash
# ¿Dónde encontró cada ejecutable?
which python3
which git
which inventado
```

---

## Las variables de entorno: el sistema nervioso de Linux

_(Tus notas aquí)_

```bash
env

echo $HOME
echo $USER
echo $SHELL
echo $HOSTNAME
echo $PWD
echo $LANG
```

### Variables que leen las aplicaciones

| Aplicación | Variable | Para qué |
|---|---|---|
| PostgreSQL | `$PGHOST`, `$PGPORT`, `$PGUSER` | Conexión a la BD |
| Python | `$PYTHONPATH` | Encontrar módulos |
| Git | `$GIT_AUTHOR_NAME`, `$GIT_AUTHOR_EMAIL` | Identidad en commits |
| Docker | `$DOCKER_HOST` | Ubicación del daemon |

### Comparar $HOSTNAME entre máquinas

```bash
# Desde Desktop:
echo "Estoy en: $HOSTNAME"

# Desde Artemis (por SSH):
ssh alumno@192.168.56.20 'echo "Estoy en: $HOSTNAME"'
```

---

## Crear y exportar variables

_(Tus notas aquí — diferencia entre local y exportada)_

```bash
# Variable local — solo esta terminal
MISION="artemis-01"
echo $MISION

# Variable exportada — visible para scripts y procesos hijos
export CLUSTER_IP="192.168.56.20"
export PROYECTO="launch-control"
export CURSO="IFCD0112"

echo "Cluster: $CLUSTER_IP, Proyecto: $PROYECTO"

env | grep CLUSTER
env | grep PROYECTO
```

---

## El truco del PATH personal

_(Tus notas aquí)_

```bash
# Crear directorio para scripts propios
mkdir -p ~/bin

# Script de prueba — saludo del cluster
cat > ~/bin/saludo_cluster.sh << 'EOF'
#!/bin/bash
echo "══════════════════════════════════════"
echo "  Cluster Launch Control"
echo "  Operador: $USER en $(hostname)"
echo "  Fecha: $(date '+%d/%m/%Y %H:%M')"
echo "  Red interna: $(ip -4 addr show | grep 192.168.56 | awk '{print $2}')"
echo "══════════════════════════════════════"
EOF

chmod +x ~/bin/saludo_cluster.sh

# Sin PATH — ruta completa:
~/bin/saludo_cluster.sh

# Añadir ~/bin al PATH:
export PATH="$PATH:$HOME/bin"

# Ahora funciona desde cualquier directorio:
cd /tmp
saludo_cluster.sh
cd ~
```

---

## .bashrc — La memoria permanente del shell

_(Tus notas aquí — cuándo se ejecuta .bashrc vs .bash_profile)_

| Archivo | Cuándo se ejecuta |
|---|---|
| `.bash_profile` | Al iniciar sesión (SSH, tty, `su -`) |
| `.bashrc` | En cada terminal nueva (escritorio, tmux) |
| **Regla Ubuntu** | Editar `.bashrc` (cubre ambos casos) |

```bash
cat ~/.bashrc
nano ~/.bashrc
```

Agregar al final:

```bash
# ─── Configuración cluster Launch Control ────────────────────
export CURSO_DIR="$HOME/curso_ifcd0112"
export PATH="$PATH:$HOME/bin"

# IPs del cluster
export ARTEMIS_IP="192.168.56.20"
export DESKTOP_IP="192.168.56.10"

# Alias útiles
alias ll='ls -la --color=auto'
alias ..='cd ..'
alias ...='cd ../..'
alias gs='git status'
alias glog='git log --oneline -10'

# Alias del cluster
alias ping-artemis='ping -c 3 $ARTEMIS_IP'
alias ssh-artemis='ssh alumno@$ARTEMIS_IP'
alias estado='estado_cluster.sh'
```

```bash
source ~/.bashrc

echo $ARTEMIS_IP
ping-artemis
echo $CURSO_DIR
ll
```

---

# BLOQUE II — Scripting Bash

## ¿Por qué automatizar?

_(Tus notas aquí)_

---

## Script 1 — Estado del cluster

```bash
#!/bin/bash
# estado_cluster.sh — Diagnóstico del cluster Launch Control

echo "═══════════════════════════════════════════"
echo "  ESTADO DEL CLUSTER — $(hostname)"
echo "  $(date '+%A %d de %B de %Y, %H:%M')"
echo "═══════════════════════════════════════════"
echo ""

# --- Identidad ---
echo "IDENTIDAD:"
echo "   Operador: $USER"
echo "   Máquina:  $(hostname)"
echo "   Home:     $HOME"
echo ""

# --- Red del cluster ---
echo "RED DEL CLUSTER:"
ip -4 addr show | grep "inet " | grep -v "127.0.0.1" | \
    awk '{printf "   %s → %s\n", $NF, $2}'
echo ""

# --- Ping a Artemis ---
echo "CONECTIVIDAD:"
if ping -c 1 -W 2 192.168.56.20 &>/dev/null; then
    echo "   Artemis (192.168.56.20): ONLINE"
else
    echo "   Artemis (192.168.56.20): OFFLINE"
fi

if ping -c 1 -W 2 192.168.56.10 &>/dev/null; then
    echo "   Desktop (192.168.56.10): ONLINE"
else
    echo "   Desktop (192.168.56.10): OFFLINE"
fi
echo ""

# --- Disco ---
echo "DISCO:"
df -h / | tail -1 | awk '{printf "   Usado: %s de %s (%s)\n", $3, $2, $5}'
echo ""

# --- Memoria ---
echo "MEMORIA:"
free -h | grep Mem | awk '{printf "   Usada: %s de %s\n", $3, $2}'
echo ""

# --- Procesos ---
TOTAL_PROCS=$(ps aux | wc -l)
MIS_PROCS=$(ps aux | grep "^$USER" | wc -l)
echo "PROCESOS:"
echo "   Total: $TOTAL_PROCS | Míos: $MIS_PROCS"
echo ""

# --- Top 3 por RAM ---
echo "TOP 3 PROCESOS (por RAM):"
ps aux --sort=-%mem | head -4 | tail -3 | \
    awk '{printf "   %-20s %s%% RAM (%s)\n", $11, $4, $1}'
echo ""

# --- SSH a Artemis ---
echo "SSH A ARTEMIS:"
if ssh -o ConnectTimeout=3 -o BatchMode=yes alumno@192.168.56.20 'echo OK' 2>/dev/null; then
    echo "   Conexión SSH: OK"
    ARTEMIS_UPTIME=$(ssh -o ConnectTimeout=3 alumno@192.168.56.20 'uptime -p' 2>/dev/null)
    echo "   Uptime Artemis: $ARTEMIS_UPTIME"
else
    echo "   Conexión SSH: FALLO (¿está encendida? ¿clave configurada?)"
fi
echo ""

# --- Últimos archivos del proyecto ---
if [ -d "$HOME/curso_ifcd0112" ]; then
    echo "ÚLTIMOS ARCHIVOS MODIFICADOS (proyecto):"
    find "$HOME/curso_ifcd0112" -type f -printf '%T@ %p\n' 2>/dev/null | \
        sort -rn | head -3 | \
        while read -r ts file; do
            fecha=$(date -d "@${ts%.*}" '+%d/%m %H:%M')
            echo "   $fecha — $(basename "$file")"
        done
    echo ""
fi

echo "═══════════════════════════════════════════"
```

```bash
chmod +x ~/bin/estado_cluster.sh
estado
```

---

## Script 2 — Backup del proyecto con fecha

_(Tus notas aquí)_

```bash
#!/bin/bash
# backup_proyecto.sh — Backup comprimido de Launch Control

FECHA=$(date +%Y-%m-%d_%H%M)
ORIGEN="$HOME/curso_ifcd0112"
DESTINO="$HOME/backups"
NOMBRE="launch_control_$FECHA.tar.gz"

mkdir -p "$DESTINO"

if [ ! -d "$ORIGEN" ]; then
    echo "ERROR: No existe $ORIGEN"
    echo "Crealo primero: mkdir -p $ORIGEN"
    exit 1
fi

echo "Creando backup de Launch Control..."
tar -czf "$DESTINO/$NOMBRE" -C "$HOME" "curso_ifcd0112"

if [ $? -eq 0 ]; then
    TAMANO=$(du -sh "$DESTINO/$NOMBRE" | cut -f1)
    echo "Backup creado: $DESTINO/$NOMBRE ($TAMANO)"
else
    echo "ERROR al crear el backup"
    exit 1
fi

echo ""
echo "Últimos backups:"
ls -lht "$DESTINO"/launch_control_*.tar.gz 2>/dev/null | head -5
```

```bash
chmod +x ~/bin/backup_proyecto.sh
mkdir -p ~/curso_ifcd0112
echo "archivo de prueba" > ~/curso_ifcd0112/test.txt
backup_proyecto.sh
```

---

## Estructuras de control en Bash

_(Tus notas aquí — condicionales, bucles, variables especiales)_

### Operadores de comparación

| Archivos | Strings | Números |
|---|---|---|
| `[ -f archivo ]` existe | `[ "$A" = "$B" ]` iguales | `[ $A -eq $B ]` igual |
| `[ -d directorio ]` existe | `[ "$A" != "$B" ]` distintos | `[ $A -lt $B ]` menor |
| `[ ! -f archivo ]` no existe | `[ -z "$A" ]` vacío | `[ $A -gt $B ]` mayor |

### Condicionales

```bash
# ¿Existe el directorio del proyecto?
if [ -d ~/curso_ifcd0112 ]; then
    echo "El proyecto existe"
    echo "Archivos: $(ls ~/curso_ifcd0112 | wc -l)"
else
    echo "El proyecto no existe — creando..."
    mkdir -p ~/curso_ifcd0112
fi

# ¿En qué máquina del cluster estoy?
MAQUINA=$(hostname)
if [ "$MAQUINA" = "artemis" ]; then
    echo "Estás en el SERVIDOR (Artemis)"
    echo "IP esperada: 192.168.56.20"
elif [ "$MAQUINA" = "desktop" ]; then
    echo "Estás en el CLIENTE (Desktop)"
    echo "IP esperada: 192.168.56.10"
else
    echo "Máquina no reconocida: $MAQUINA"
fi

# ¿Artemis está online?
if ping -c 1 -W 2 192.168.56.20 &>/dev/null; then
    echo "Artemis está ONLINE"
else
    echo "ALERTA: Artemis no responde"
fi
```

### Bucles

```bash
# Recorrer archivos del proyecto
for archivo in ~/curso_ifcd0112/*.txt; do
    echo "Archivo: $(basename "$archivo") — $(wc -l < "$archivo") líneas"
done

# Hacer ping a varias máquinas del cluster
for IP in 192.168.56.10 192.168.56.20 192.168.56.1; do
    if ping -c 1 -W 1 "$IP" &>/dev/null; then
        echo "$IP → ONLINE"
    else
        echo "$IP → OFFLINE"
    fi
done
```

### Variables especiales en scripts

| Variable | Significado |
|---|---|
| `$0` | Nombre del script |
| `$1`, `$2` | Primer, segundo argumento |
| `$#` | Número de argumentos |
| `$?` | Código de salida (0 = éxito) |
| `$@` | Todos los argumentos |

```bash
#!/bin/bash
# ping_nodo.sh — Verificar un nodo del cluster
# Uso: ping_nodo.sh <nombre> <ip>

if [ $# -lt 2 ]; then
    echo "Uso: ping_nodo.sh <nombre> <ip>"
    echo "Ejemplo: ping_nodo.sh artemis 192.168.56.20"
    exit 1
fi

NOMBRE="$1"
IP="$2"

echo "Verificando nodo $NOMBRE ($IP)..."
if ping -c 3 -W 2 "$IP" &>/dev/null; then
    echo "  $NOMBRE está ONLINE"
    exit 0
else
    echo "  $NOMBRE está OFFLINE"
    exit 1
fi
```

```bash
chmod +x ~/bin/ping_nodo.sh
ping_nodo.sh artemis 192.168.56.20
ping_nodo.sh desktop 192.168.56.10
echo "Código de salida: $?"
```

---

## Script 3 — Monitor del cluster en tiempo real

_(Tus notas aquí)_

```bash
#!/bin/bash
# monitor_cluster.sh — Monitorizar directorio + estado del cluster
# Uso: monitor_cluster.sh [directorio] [intervalo_segundos]

DIRECTORIO="${1:-$HOME/curso_ifcd0112}"
INTERVALO="${2:-5}"

if [ ! -d "$DIRECTORIO" ]; then
    echo "ERROR: $DIRECTORIO no es un directorio"
    exit 1
fi

echo "Monitor Launch Control — $(hostname)"
echo "Directorio: $DIRECTORIO | Intervalo: ${INTERVALO}s"
echo "Ctrl+C para detener"
echo ""

while true; do
    ARCHIVOS=$(find "$DIRECTORIO" -type f | wc -l)
    TAMANO=$(du -sh "$DIRECTORIO" 2>/dev/null | cut -f1)
    ULTIMO=$(find "$DIRECTORIO" -type f -printf '%T@ %p\n' 2>/dev/null | \
             sort -rn | head -1 | cut -d' ' -f2-)

    if ping -c 1 -W 1 192.168.56.20 &>/dev/null; then
        ARTEMIS="ON"
    else
        ARTEMIS="OFF"
    fi

    echo "[$(date '+%H:%M:%S')] Archivos: $ARCHIVOS | Tamaño: $TAMANO | Artemis: $ARTEMIS | Último: $(basename "$ULTIMO" 2>/dev/null)"
    sleep "$INTERVALO"
done
```

```bash
chmod +x ~/bin/monitor_cluster.sh
monitor_cluster.sh ~/curso_ifcd0112 3
```

---

## Mini-ejercicio — Script de info de red

_(Tus notas aquí)_

```bash
#!/bin/bash
# info_red.sh — Info de red de esta máquina del cluster

echo "═══════════════════════════════════════════"
echo "  INFO RED — $(hostname)"
echo "═══════════════════════════════════════════"
echo ""

echo "INTERFACES DE RED:"
ip -4 addr show | grep -E "inet |^[0-9]" | while read -r linea; do
    if echo "$linea" | grep -q "^[0-9]"; then
        IFACE=$(echo "$linea" | awk -F: '{print $2}' | tr -d ' ')
        echo ""
        echo "  Interfaz: $IFACE"
    fi
    if echo "$linea" | grep -q "inet "; then
        IP=$(echo "$linea" | awk '{print $2}')
        echo "    IP: $IP"
    fi
done
echo ""

echo "GATEWAY:"
ip route | grep default | awk '{printf "   %s via %s\n", $5, $3}'
echo ""

echo "DNS:"
if [ -f /etc/resolv.conf ]; then
    grep "^nameserver" /etc/resolv.conf | awk '{printf "   %s\n", $2}'
fi
echo ""

echo "CONECTIVIDAD:"
if ping -c 1 -W 2 192.168.56.20 &>/dev/null; then
    echo "   Artemis: ONLINE"
else
    echo "   Artemis: OFFLINE"
fi

if ping -c 1 -W 2 8.8.8.8 &>/dev/null; then
    echo "   Internet: OK"
else
    echo "   Internet: Sin conexión"
fi

echo ""
echo "PUERTOS ESCUCHANDO:"
ss -tlnp 2>/dev/null | grep LISTEN | awk '{printf "   %s (%s)\n", $4, $NF}' | head -10

echo ""
echo "═══════════════════════════════════════════"
```

```bash
chmod +x ~/bin/info_red.sh
info_red.sh
```

---

# BLOQUE III — Kanban y Lógica Computacional

## ¿Cómo se organiza un equipo de desarrollo real?

_(Tus notas aquí)_

---

## El tablero Kanban

_(Tus notas aquí)_

```
┌────────────────────┬────────────────────┬────────────────────┐
│    TO DO           │   IN PROGRESS      │      DONE          │
├────────────────────┼────────────────────┼────────────────────┤
│                    │                    │                    │
│                    │                    │                    │
└────────────────────┴────────────────────┴────────────────────┘
```

| Regla | Detalle |
|---|---|
| WIP limit | Máximo 1-2 tareas en progreso |
| Regla de 2 días | Si lleva +2 días en progress → dividir o reportar |
| Mover obligatorio | Cuando empiezas → mover. Cuando terminas → mover |
| Herramienta | Trello — tablero del curso |

---

## Lógica computacional — Pensar antes de codificar

_(Tus notas aquí — por qué el pseudocódigo es más importante que el lenguaje)_

### Ejemplo: pseudocódigo de autenticación

```
FUNCIÓN autenticar_usuario(nombre, contraseña):
    usuario = buscar_en_bd(nombre)

    SI usuario NO existe:
        DEVOLVER error("Usuario no encontrado")

    SI contraseña NO coincide con usuario.contraseña_hash:
        DEVOLVER error("Contraseña incorrecta")

    DEVOLVER token_sesion(usuario.id)
FIN FUNCIÓN
```

---

## Diagramas de flujo con Napkin.ai

_(Tus notas aquí)_

| Tipo de diagrama | Cuándo lo usamos |
|---|---|
| Diagrama de flujo | Lógica y algoritmos (HOY) |
| Diagrama E-R | Diseño de bases de datos (Fase II) |
| Diagrama de clases | Arquitectura POO Python (Fase III) |
| Diagrama secuencia | Flujo de llamadas API REST |

---

## Ejercicios de lógica — Pseudocódigo

### Ejercicio A — Clasificar una contraseña

```
- Si está vacía → "Sin contraseña"
- Si tiene menos de 8 caracteres → "Débil"
- Si tiene 8+ y solo letras/números → "Media"
- Si tiene 8+, letras + números + símbolos → "Fuerte"
```

_(Tu pseudocódigo aquí)_

### Ejercicio B — Verificar nodos del cluster

```
Dada una lista de nodos con nombre e IP:
Para cada nodo, hacer ping.
Responde → ONLINE. No responde → OFFLINE + sumar fallo.
Al final: resumen + alerta si hay alguno offline.
```

_(Tu pseudocódigo aquí)_

### Ejercicio C — Validar un email básico

```
Reglas: exactamente un @, no primero ni último, punto después del @, sin espacios
```

_(Tu pseudocódigo aquí)_

### Ejercicio D — Contar palabras únicas

```
"el gato y el perro y el gato" → 4 (el, gato, y, perro)
```

_(Tu pseudocódigo aquí)_

---

## Del pseudocódigo al código — Validar contraseña

_(Tus notas aquí — comparar tu pseudocódigo con el código real)_

```bash
#!/bin/bash
# validar_password.sh — Del pseudocódigo al código real

if [ $# -eq 0 ]; then
    echo "Uso: validar_password.sh <contraseña>"
    exit 1
fi

PASSWORD="$1"
LONGITUD=${#PASSWORD}

if [ $LONGITUD -eq 0 ]; then
    echo "Sin contraseña"
    exit 1
fi

if [ $LONGITUD -lt 8 ]; then
    echo "DÉBIL — Solo $LONGITUD caracteres (mínimo 8)"
    exit 0
fi

TIENE_LETRAS=0
TIENE_NUMEROS=0
TIENE_SIMBOLOS=0

echo "$PASSWORD" | grep -q '[a-zA-Z]' && TIENE_LETRAS=1
echo "$PASSWORD" | grep -q '[0-9]' && TIENE_NUMEROS=1
echo "$PASSWORD" | grep -q '[^a-zA-Z0-9]' && TIENE_SIMBOLOS=1

if [ $TIENE_LETRAS -eq 1 ] && [ $TIENE_NUMEROS -eq 1 ] && [ $TIENE_SIMBOLOS -eq 1 ]; then
    echo "FUERTE — $LONGITUD caracteres, letras + números + símbolos"
elif [ $TIENE_LETRAS -eq 1 ] && [ $TIENE_NUMEROS -eq 1 ]; then
    echo "MEDIA — $LONGITUD caracteres, letras + números (falta símbolo)"
else
    echo "MEDIA — $LONGITUD caracteres, pero le falta variedad"
fi
```

```bash
chmod +x ~/bin/validar_password.sh
validar_password.sh "abc"
validar_password.sh "abcdefgh"
validar_password.sh "abc12345"
validar_password.sh "Abc123!@"
```

---

## Referencia rápida

### Variables de entorno

| Concepto | Comando |
|---|---|
| Ver todas | `env` o `printenv` |
| Ver una | `echo $VARIABLE` |
| Crear temporal | `VARIABLE="valor"` |
| Exportar | `export VARIABLE="valor"` |
| Añadir al PATH | `export PATH="$PATH:/nueva/ruta"` |
| Hacer permanente | Añadir al `~/.bashrc` |
| Aplicar cambios | `source ~/.bashrc` |
| Encontrar ejecutable | `which programa` |

### Scripting Bash

| Concepto | Sintaxis |
|---|---|
| Shebang | `#!/bin/bash` |
| Hacer ejecutable | `chmod +x script.sh` |
| Variable | `NOMBRE="valor"` |
| Usar variable | `"$NOMBRE"` |
| Salida de comando | `RESULTADO=$(comando)` |
| Condicional | `if [ cond ]; then ... fi` |
| Archivo existe | `[ -f archivo ]` |
| Directorio existe | `[ -d directorio ]` |
| Comparar strings | `[ "$A" = "$B" ]` |
| Comparar números | `[ $A -lt $B ]` |
| Bucle | `for x in lista; do ... done` |
| Código de salida | `$?` (0 = éxito) |

---

## Tarea 13 — Entrega en Classroom

**Fecha límite:** Martes 28 de abril a las 09:00

1. Script `estado_cluster.sh` funcionando + captura
2. Pseudocódigo de los 4 ejercicios de lógica (A, B, C, D) en `.md`
3. Diagrama de flujo de Napkin.ai para Ejercicio A o C (captura)
4. Captura del tablero Trello con al menos 3 tareas organizadas

Entregar en `~/curso_ifcd0112/tareas/tarea_13/`

---

## Autoevaluación

1. ¿Qué comando muestra dónde está un ejecutable en el PATH?

2. ¿Cuál es la diferencia entre `VARIABLE="valor"` y `export VARIABLE="valor"`?

3. ¿Dónde se guardan las variables permanentes en Ubuntu?

4. ¿Qué hace `source ~/.bashrc`?

5. ¿Qué significa `$?` en un script?

6. ¿Qué es el WIP limit en Kanban?

7. ¿Por qué escribimos pseudocódigo antes de codificar?

---

_Prof. Juan Marcelo Gutiérrez Miranda | @TodoEconometria_
_Amtigravity Launch Control — IFCD0112_
