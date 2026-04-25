# 23 de Abril 2026 — Red Interna con Cluster de Máquinas Virtuales

**Curso IFCD0112 — Programación con Lenguajes OO y BBDD Relacionales**
**Módulo:** MF0223_3 — Sistemas informáticos | **Jornada:** 5 horas
Prof. Juan Marcelo Gutiérrez Miranda | @TodoEconometria

---

# RECONEXIÓN

---

| Concepto | Lo que hicimos |
|---|---|
| Restauración | Snapshots VirtualBox + Timeshift |
| Estructura | Proyecto Launch Control organizado |
| Sincronización | rsync desde carpeta compartida Windows→Linux |
| Gráficos | X11 forzado, VRAM 256 MB, TearFree |
| IDE | Antigravity instalado y optimizado |

_Tu resumen:_

---

# PREFLIGHT CHECK — Verificación del Entorno

---

## El script de verificación

```bash
# 📁 Ejecutar en la terminal de la VM Desktop
cat > /tmp/preflight_dia23.sh << 'SCRIPT'
#!/bin/bash
# ═══════════════════════════════════════════════════════════
# PREFLIGHT CHECK — Día 23: Red Interna con Cluster de VMs
# Verifica que el entorno está listo antes de empezar
# ═══════════════════════════════════════════════════════════

echo ""
echo "╔══════════════════════════════════════════════════════════════╗"
echo "║  PREFLIGHT CHECK — DÍA 23: RED INTERNA                      ║"
echo "║  $(date '+%Y-%m-%d %H:%M:%S')                                       ║"
echo "╠══════════════════════════════════════════════════════════════╣"

PASS=0
FAIL=0
WARN=0

check_ok() {
    echo "║  [OK]  $1"
    PASS=$((PASS + 1))
}

check_fail() {
    echo "║  [!!]  $1"
    echo "║        FIX: $2"
    FAIL=$((FAIL + 1))
}

check_warn() {
    echo "║  [~~]  $1"
    echo "║        NOTA: $2"
    WARN=$((WARN + 1))
}

echo "║"
echo "║  --- SISTEMA OPERATIVO ---"

# Ubuntu funciona
if [ -f /etc/os-release ]; then
    . /etc/os-release
    check_ok "Sistema: $PRETTY_NAME"
else
    check_fail "No se puede identificar el sistema" "¿Están en Ubuntu?"
fi

# Sesión gráfica
SESSION=$(echo $XDG_SESSION_TYPE 2>/dev/null)
if [ "$SESSION" = "x11" ]; then
    check_ok "Sesión gráfica: X11"
elif [ "$SESSION" = "wayland" ]; then
    check_warn "Sesión gráfica: Wayland" "Funciona, pero X11 es más estable en VBox"
else
    check_warn "Sesión gráfica: $SESSION" "No se pudo detectar"
fi

echo "║"
echo "║  --- VIRTUALBOX GUEST ADDITIONS ---"

# Guest Additions instaladas
if dpkg -l 2>/dev/null | grep -q "virtualbox-guest"; then
    GA_VERSION=$(dpkg -l | grep "virtualbox-guest-utils" | awk '{print $3}' | head -1)
    check_ok "Guest Additions instaladas ($GA_VERSION)"
else
    check_fail "Guest Additions NO instaladas" "sudo apt install virtualbox-guest-utils virtualbox-guest-x11 -y && sudo reboot"
fi

# Módulo de kernel VBox cargado
if lsmod | grep -q "vboxguest"; then
    check_ok "Módulo vboxguest cargado en kernel"
else
    check_fail "Módulo vboxguest NO cargado" "Reiniciar la VM o reinstalar Guest Additions"
fi

echo "║"
echo "║  --- RECURSOS DISPONIBLES ---"

# RAM disponible
RAM_TOTAL=$(free -m | awk '/^Mem:/{print $2}')
RAM_AVAIL=$(free -m | awk '/^Mem:/{print $7}')
if [ "$RAM_AVAIL" -gt 500 ]; then
    check_ok "RAM: ${RAM_AVAIL}MB disponibles de ${RAM_TOTAL}MB total"
else
    check_warn "RAM baja: ${RAM_AVAIL}MB disponibles de ${RAM_TOTAL}MB" "Cerrar aplicaciones innecesarias"
fi

# Disco disponible
DISCO_AVAIL=$(df -BG / | awk 'NR==2{print $4}' | tr -d 'G')
if [ "$DISCO_AVAIL" -gt 10 ]; then
    check_ok "Disco: ${DISCO_AVAIL}GB disponibles en /"
elif [ "$DISCO_AVAIL" -gt 5 ]; then
    check_warn "Disco ajustado: ${DISCO_AVAIL}GB disponibles" "Necesitamos ~5GB para la VM servidor"
else
    check_fail "Disco insuficiente: ${DISCO_AVAIL}GB" "Necesitamos al menos 5GB libres. Limpiar con: sudo apt autoremove -y && sudo apt clean"
fi

echo "║"
echo "║  --- CONECTIVIDAD ---"

# Internet (NAT)
if ping -c 1 -W 3 8.8.8.8 &>/dev/null; then
    check_ok "Conexión a internet (NAT funciona)"
else
    check_warn "Sin conexión a internet" "Verificar que el adaptador NAT está habilitado en VBox"
fi

# DNS
if ping -c 1 -W 3 google.com &>/dev/null; then
    check_ok "Resolución DNS funciona"
else
    check_warn "DNS no resuelve" "Internet por IP funciona, pero DNS no. Verificar /etc/resolv.conf"
fi

echo "║"
echo "║  --- HERRAMIENTAS NECESARIAS ---"

# SSH client
if command -v ssh &>/dev/null; then
    check_ok "SSH cliente instalado"
else
    check_fail "SSH cliente NO instalado" "sudo apt install openssh-client -y"
fi

# SSH server
if dpkg -l 2>/dev/null | grep -q "openssh-server"; then
    check_ok "SSH servidor instalado"
else
    check_warn "SSH servidor NO instalado" "Lo instalaremos durante la clase"
fi

# curl
if command -v curl &>/dev/null; then
    check_ok "curl instalado"
else
    check_fail "curl NO instalado" "sudo apt install curl -y"
fi

# Python3
if command -v python3 &>/dev/null; then
    PY_VER=$(python3 --version 2>&1)
    check_ok "$PY_VER disponible"
else
    check_fail "Python3 NO instalado" "sudo apt install python3 -y"
fi

# net-tools o iproute2
if command -v ip &>/dev/null; then
    check_ok "iproute2 (ip) disponible"
else
    check_fail "iproute2 NO disponible" "sudo apt install iproute2 -y"
fi

# ss
if command -v ss &>/dev/null; then
    check_ok "ss (socket statistics) disponible"
else
    check_warn "ss no disponible" "sudo apt install iproute2 -y"
fi

echo "║"
echo "║  --- ESTRUCTURA LAUNCH CONTROL ---"

for dir in config scripts logs data docs; do
    if [ -d "$HOME/launch-control/$dir" ]; then
        check_ok "launch-control/$dir/"
    else
        check_warn "launch-control/$dir/ no encontrado" "mkdir -p ~/launch-control/$dir"
    fi
done

echo "║"
echo "╠══════════════════════════════════════════════════════════════╣"
echo "║"
echo "║  RESULTADO: $PASS ok | $FAIL errores | $WARN avisos"
echo "║"

if [ $FAIL -eq 0 ]; then
    echo "║  LISTOS PARA EMPEZAR — todos los sistemas operativos"
    echo "║"
    echo "║  Próximo paso: tomar un snapshot antes de configurar redes"
    echo "║  → Desde VirtualBox en Windows: Máquina → Tomar instantánea"
    echo "║  → Nombre sugerido: Pre-redes — 23-04-26"
else
    echo "║  HAY $FAIL ERRORES — corregir antes de continuar"
    echo "║"
    echo "║  Busquen los items marcados [!!] arriba y ejecuten el FIX indicado."
    echo "║  Después vuelvan a ejecutar este script."
fi

echo "║"
echo "╚══════════════════════════════════════════════════════════════╝"
SCRIPT

chmod +x /tmp/preflight_dia23.sh
bash /tmp/preflight_dia23.sh
```

## Interpretación de resultados

| Símbolo | Significado | Acción |
|---|---|---|
| `[OK]` | Todo bien | Nada que hacer |
| `[~~]` | Aviso menor | Funciona pero se puede mejorar |
| `[!!]` | Error crítico | Corregir ANTES de continuar |

## Fixes rápidos

```bash
# 📁 Instalar todo de una vez si falta algo
sudo apt update
sudo apt install -y openssh-client openssh-server curl python3 iproute2 net-tools
```

```bash
# 📁 Limpiar disco si está lleno
sudo apt autoremove -y
sudo apt clean
du -sh /var/cache/apt/archives/ 2>/dev/null
du -sh /tmp/ 2>/dev/null
df -h /
```

## Snapshot antes de empezar

_Tus notas sobre snapshots:_

---

# PARTE I — TEORÍA DE REDES (lo esencial)

---

## 1.1 ¿Qué es una dirección IP?

_Tu explicación:_

```bash
# 📁 Ejecutar en la terminal de la VM Desktop
# Ver las IPs de esta máquina
ip addr show
```

| Tipo | Rango | Ejemplo | Accesible desde internet |
|---|---|---|---|
| **Privada** | 10.x.x.x, 172.16-31.x.x, 192.168.x.x | 192.168.56.10 | No |
| **Pública** | Cualquier otra | 142.250.184.206 (Google) | Sí |

## 1.2 ¿Qué es un puerto?

_Tu explicación:_

```bash
# 📁 Ejecutar en la terminal de la VM Desktop
# Ver qué puertos están escuchando en esta máquina
ss -tulnp
```

| Rango | Uso |
|---|---|
| 0-1023 | **Bien conocidos** (SSH=22, HTTP=80, HTTPS=443) — necesitan root |
| 1024-49151 | **Registrados** (PostgreSQL=5432, MySQL=3306) |
| 49152-65535 | **Dinámicos** — los usan las aplicaciones temporalmente |

| Columna `ss` | Significado |
|---|---|
| `State` | LISTEN = esperando conexiones |
| `Local Address:Port` | IP y puerto donde escucha |
| `Process` | Qué programa tiene ese puerto abierto |

## 1.3 ¿Qué es una subred?

_Tu explicación:_

## 1.4 Modos de red en VirtualBox

_Tu explicación:_

| Característica | NAT | Bridge | Host-only |
|---|---|---|---|
| VM accede a internet | Sí | Sí | No |
| Internet accede a la VM | No | Sí | No |
| VMs se ven entre sí | No | Sí | **Sí** |
| Host ve a la VM | No | Sí | **Sí** |
| Seguro para aprender | Sí | No | **Sí** |
| Necesita red externa | Sí | Sí | **No** |

## 1.5 Diagrama de la arquitectura que vamos a montar

_Tu diagrama:_

| Interfaz | Adaptador | Función | IP |
|---|---|---|---|
| `enp0s3` | NAT | Acceso a internet | 10.0.2.15 (DHCP de VBox) |
| `enp0s8` | Host-only | Red interna entre VMs | 192.168.56.10 o .20 |

---

# PARTE II — CONFIGURAR RED HOST-ONLY EN VIRTUALBOX

---

## 2.1 Crear la red host-only

_Pasos desde VirtualBox Manager (Windows):_

| Campo (Adaptador) | Valor |
|---|---|
| Dirección IPv4 | `192.168.56.1` |
| Máscara de red IPv4 | `255.255.255.0` |

| Campo (Servidor DHCP) | Valor |
|---|---|
| Habilitar servidor | **Desmarcar** (usamos IPs estáticas) |

## 2.2 Agregar interfaz host-only a la VM Desktop

_Pasos desde VirtualBox → Configuración → Red:_

**Adaptador 1 — NO TOCAR:**

| Campo | Valor |
|---|---|
| Habilitar | Sí |
| Conectado a | NAT (NO CAMBIAR) |

**Adaptador 2 — CONFIGURAR:**

| Campo | Valor |
|---|---|
| Habilitar adaptador de red | **Sí** (marcar) |
| Conectado a | **Adaptador solo-anfitrión** (Host-only Adapter) |
| Nombre | `VirtualBox Host-Only Ethernet Adapter` |
| Modo promiscuo | Permitir todo (Allow All) |

## 2.3 Configurar IP estática en la VM Desktop

```bash
# 📁 Ejecutar en la terminal de la VM Desktop

# Ver las interfaces de red disponibles
ip link show
```

```bash
# 📁 Ejecutar en la terminal de la VM Desktop

# Ver qué archivo de netplan existe
ls /etc/netplan/
```

```bash
# 📁 Ejecutar en la terminal de la VM Desktop

# Crear configuración para la interfaz host-only
sudo bash -c 'cat > /etc/netplan/60-host-only.yaml << EOF
network:
  version: 2
  ethernets:
    enp0s8:
      addresses:
        - 192.168.56.10/24
      dhcp4: false
EOF'
```

| Línea | Significado |
|---|---|
| `network:` | Inicio de la configuración de red |
| `version: 2` | Versión del formato Netplan |
| `ethernets:` | Sección de interfaces cableadas |
| `enp0s8:` | Nombre de la interfaz host-only |
| `addresses:` | Lista de IPs a asignar |
| `- 192.168.56.10/24` | IP estática + máscara |
| `dhcp4: false` | No pedir IP por DHCP |

```bash
# 📁 Ejecutar en la terminal de la VM Desktop

# Verificar que el archivo está bien
cat /etc/netplan/60-host-only.yaml

# Aplicar la configuración
sudo netplan apply
```

```bash
# 📁 Si hay error de YAML, verificar indentación
cat -A /etc/netplan/60-host-only.yaml
# Los tabuladores aparecen como ^I — reemplazar por espacios
```

## 2.4 Verificar la configuración

```bash
# 📁 Ejecutar en la terminal de la VM Desktop

# Ver todas las IPs asignadas
ip addr show
```

```bash
# 📁 Ejecutar en la terminal de la VM Desktop

# ¿La interfaz host-only tiene la IP correcta?
ip addr show enp0s8 | grep "inet "
# Debe mostrar: inet 192.168.56.10/24

# ¿Puede alcanzar al host Windows? (192.168.56.1)
ping -c 3 192.168.56.1

# ¿Internet sigue funcionando? (por el adaptador NAT)
ping -c 3 8.8.8.8
```

```bash
# 📁 Si ping 192.168.56.1 falla:

# ¿La interfaz está levantada?
ip link show enp0s8
# Debe decir UP, no DOWN

# Si dice DOWN:
sudo ip link set enp0s8 up
sudo netplan apply

# ¿Hay ruta hacia la subred?
ip route | grep 192.168.56
# Debe mostrar: 192.168.56.0/24 dev enp0s8
```

---

# PARTE III — CREAR VM SERVIDOR "ARTEMIS"

---

## Opciones de creación

| Opción | Ventaja | Desventaja |
|---|---|---|
| **A: Clonar VM Desktop** | Más rápido (10 min) | Pesa más (~20GB), trae escritorio innecesario |
| **B: Ubuntu Server ISO** | Más ligero (512MB RAM, 5GB disco) | Más lento (20-30 min) |

## 3.1 Opción A — Clonar la VM Desktop

_Pasos si elegimos clonar:_

```bash
# 📁 Ejecutar dentro de Artemis (después de arrancar el clon)
sudo hostnamectl set-hostname artemis
# Cerrar sesión y volver a entrar
```

## 3.2 Opción B — Ubuntu Server desde ISO (recomendada)

### Crear la VM en VirtualBox

| Campo | Valor |
|---|---|
| Nombre | `Artemis` |
| Tipo | Linux |
| Versión | Ubuntu (64-bit) |
| ISO Image | Seleccionar el ISO descargado |

| Recurso | Valor | Por qué |
|---|---|---|
| RAM | **512 MB** | Ubuntu Server corre con 512MB |
| CPUs | **1** | Suficiente para ejercicios |

| Campo (Disco) | Valor |
|---|---|
| Crear disco virtual ahora | Sí |
| Tamaño | **5 GB** |
| Tipo | VDI |
| Almacenamiento | Dinámicamente asignado |

### Configurar la red ANTES de instalar

**Adaptador 1:**

| Campo | Valor |
|---|---|
| Habilitar | Sí |
| Conectado a | NAT |

**Adaptador 2:**

| Campo | Valor |
|---|---|
| Habilitar | Sí |
| Conectado a | Adaptador solo-anfitrión |
| Nombre | VirtualBox Host-Only Ethernet Adapter |
| Modo promiscuo | Permitir todo |

### Instalar Ubuntu Server

| Pantalla | Configuración |
|---|---|
| 1. Idioma | English (recomendado) |
| 2. Teclado | Spanish |
| 3. Tipo | Ubuntu Server (minimized) |
| 4. Red | Dejar por defecto |
| 5. Proxy | Vacío |
| 6. Mirror | Predeterminado |
| 7. Disco | Use an entire disk → 5GB |
| 8. Perfil | name: alumno / server: **artemis** / user: alumno / password: elegir |
| 9. SSH | **Install OpenSSH server: SÍ** |
| 10. Snaps | No seleccionar nada |
| 11. Instalación | Esperar → Reboot Now |

```
artemis login: alumno
Password: ********
alumno@artemis:~$
```

## 3.3 Configurar IP estática en Artemis

```bash
# 📁 Ejecutar dentro de la VM Artemis

# Ver las interfaces disponibles
ip link show
```

```bash
# 📁 Ejecutar dentro de la VM Artemis

# Ver qué archivos de netplan existen
ls /etc/netplan/

# Crear configuración para host-only
sudo bash -c 'cat > /etc/netplan/60-host-only.yaml << EOF
network:
  version: 2
  ethernets:
    enp0s8:
      addresses:
        - 192.168.56.20/24
      dhcp4: false
EOF'

# Aplicar
sudo netplan apply

# Verificar
ip addr show enp0s8
```

## 3.4 Verificar conectividad básica

```bash
# 📁 Ejecutar dentro de la VM Artemis

# ¿Puedo ver al host Windows?
ping -c 3 192.168.56.1

# ¿Puedo ver a la VM Desktop?
ping -c 3 192.168.56.10

# ¿Tengo internet? (por NAT)
ping -c 3 8.8.8.8
```

```bash
# 📁 Ejecutar en la terminal de la VM Desktop

# ¿Desktop puede ver a Artemis?
ping -c 3 192.168.56.20
```

```bash
# 📁 Si no funciona, verificar desde Artemis:

# ¿La interfaz está arriba?
ip link show enp0s8

# ¿Tiene la IP correcta?
ip addr show enp0s8 | grep "inet "

# ¿Hay ruta?
ip route | grep 192.168.56
```

---

# PARTE IV — SNAPSHOT POST-CONFIGURACIÓN

---

## Snapshots de ambas VMs

_Tus notas:_

**VM Desktop:**

| Campo | Valor |
|---|---|
| Nombre | `Post-redes — 23-04-26` |
| Descripción | `Red host-only configurada, IP 192.168.56.10, cluster funcionando` |

**VM Artemis:**

| Campo | Valor |
|---|---|
| Nombre | `Post-instalación — 23-04-26` |
| Descripción | `Ubuntu Server, IP 192.168.56.20, SSH operativo` |

---

# REFERENCIA RÁPIDA

---

## Direcciones IP del cluster

| Máquina | IP host-only | IP NAT | Rol |
|---|---|---|---|
| Windows host | 192.168.56.1 | (su IP real) | Anfitrión |
| VM Desktop | 192.168.56.10 | 10.0.2.15 | Estación de trabajo |
| VM Artemis | 192.168.56.20 | 10.0.2.15 | Servidor |

## Archivos de configuración creados

| Archivo | Máquina | Contenido |
|---|---|---|
| `/etc/netplan/60-host-only.yaml` | Desktop | IP estática 192.168.56.10 |
| `/etc/netplan/60-host-only.yaml` | Artemis | IP estática 192.168.56.20 |

## Comandos clave de hoy

| Comando | Qué hace |
|---|---|
| `ip addr show` | Ver IPs de todas las interfaces |
| `ip link show` | Ver estado de interfaces (UP/DOWN) |
| `ip route` | Ver tabla de rutas |
| `ping -c N ip` | Enviar N pings a una IP |
| `ss -tulnp` | Ver puertos escuchando |
| `sudo netplan apply` | Aplicar configuración de red |
| `hostnamectl` | Ver/cambiar nombre de la máquina |
| `cat -A archivo` | Ver caracteres ocultos (tabs = ^I) |

## Problemas frecuentes

| Problema | Causa probable | Solución |
|---|---|---|
| `ping 192.168.56.x` no responde | Interfaz host-only no configurada | Verificar `ip addr show enp0s8` → `sudo netplan apply` |
| "Network unreachable" | No hay ruta | `ip route` → verificar netplan YAML |
| `netplan apply` da error | Tabs en vez de espacios | `cat -A` y rehacer con espacios |
| enp0s8 no aparece | Adaptador 2 no habilitado | VBox → Red → Adaptador 2 → Habilitar |

---

## DIAGNÓSTICO — ¿Entendí el día?

1. ¿Qué diferencia hay entre una IP privada y una pública?
2. ¿Por qué usamos host-only y no bridge para la red entre VMs?
3. ¿Qué hace `netplan apply`?
4. ¿Cuántas interfaces de red tiene cada VM y para qué sirve cada una?
5. Si `ping 192.168.56.20` no funciona desde Desktop, ¿qué 3 cosas verifico?

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
> El contenido técnico de Linux, VirtualBox, redes TCP/IP y sus ecosistemas pertenece
> a sus respectivos autores y comunidades; la organización pedagógica, las
> explicaciones, los diagramas y el enfoque metodológico son obra original del autor.
>
> **Hash de Certificación:** `4e8d9b1a5f6e7c3d2b1a0f9e8d7c6b5a4f3e2d1c0b9a8f7e6d5c4b3a2f1e0d9c`
