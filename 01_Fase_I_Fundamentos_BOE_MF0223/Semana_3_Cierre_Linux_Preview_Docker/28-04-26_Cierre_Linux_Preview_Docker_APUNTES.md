# Cierre de Linux y Preview de Docker — APUNTES

**Certificado:** IFCD0112 | **Sesión 14** | Martes 28 de abril de 2026
**Prof.** Juan Marcelo Gutiérrez Miranda | @TodoEconometría

---

## Reconexión — ¿Dónde estamos?

_(Tus notas aquí)_

---

# BLOQUE I — El Efecto Wow: Docker en Acción

## Docker puede levantar un servidor web en 15 segundos

_(Tus notas aquí)_

```bash
# Servidor web completo en UN comando
docker run -d --name mi-servidor-web -p 8080:80 nginx
```

Acceder en el navegador: `http://localhost:8080`

```bash
# Base de datos PostgreSQL en UN comando
docker run -d --name mi-postgres \
  -e POSTGRES_PASSWORD=launch2026 \
  -p 5432:5432 \
  postgres:16
```

```bash
# Destruir y recrear desde cero (5 segundos)
docker rm -f mi-postgres
docker run -d --name mi-postgres \
  -e POSTGRES_PASSWORD=launch2026 \
  -p 5432:5432 \
  postgres:16
```

## Servidor web + base de datos juntos

```yaml
# docker-compose.yml
services:
  web:
    image: nginx
    ports:
      - "8080:80"

  db:
    image: postgres:16
    environment:
      POSTGRES_PASSWORD: launch2026
    ports:
      - "5432:5432"
```

```bash
docker compose up -d
```

## ¿Para qué aprendimos Linux entonces?

_(Tus notas aquí)_

---

# BLOQUE II — Cierre de Linux: El Mapa Completo

## Mini-evaluación

### Pregunta 1: ¿Dónde están los archivos de configuración?

_(Tus notas aquí)_

```bash
ls /etc/
```

### Pregunta 2: ¿Qué significa `rwxr-x---`?

_(Tus notas aquí)_

```bash
chmod 750 archivo
```

### Pregunta 3: ¿Proceso consumiendo mucha memoria?

```bash
top
# Presionar M para ordenar por memoria

ps aux --sort=-%mem | head -10

htop
```

### Pregunta 4: VM que no se conecta a otra

```bash
# 1. ¿Tiene IP?
ip addr show

# 2. ¿Llega al otro?
ping 192.168.56.X

# 3. ¿El servicio está escuchando?
ss -tulnp

# 4. ¿El firewall bloquea?
sudo ufw status
```

### Pregunta 5: ¿Alias disponible al abrir terminal?

```bash
echo 'alias ll="ls -la"' >> ~/.bashrc
source ~/.bashrc
```

### Pregunta 6: ¿Conectar a máquina de compañero?

```bash
ssh alumno@192.168.56.101
ssh -p 2222 alumno@192.168.56.101
```

---

## Los 30 comandos que ya dominan

| # | Comando | Qué hace | Ejemplo |
|---|---------|----------|---------|
| 1 | `ls` | Listar archivos | `ls -la /etc/` |
| 2 | `cd` | Cambiar directorio | `cd ~/curso_ifcd0112` |
| 3 | `pwd` | Dónde estoy | `pwd` |
| 4 | `mkdir` | Crear directorio | `mkdir -p proyecto/src` |
| 5 | `cp` | Copiar | `cp archivo.txt backup/` |
| 6 | `mv` | Mover/renombrar | `mv viejo.md nuevo.md` |
| 7 | `rm` | Borrar | `rm -rf directorio/` |
| 8 | `cat` | Ver contenido | `cat /etc/hostname` |
| 9 | `head/tail` | Ver inicio/final | `tail -20 log.txt` |
| 10 | `grep` | Buscar en texto | `grep "error" log.txt` |
| 11 | `chmod` | Cambiar permisos | `chmod 755 script.sh` |
| 12 | `chown` | Cambiar dueño | `sudo chown alumno:alumno archivo` |
| 13 | `ps` | Ver procesos | `ps aux` |
| 14 | `kill` | Terminar proceso | `kill -9 PID` |
| 15 | `top` | Monitor procesos | `top` → `q` para salir |
| 16 | `sudo` | Ejecutar como root | `sudo apt update` |
| 17 | `apt` | Instalar paquetes | `sudo apt install htop` |
| 18 | `ssh` | Conectar remoto | `ssh alumno@192.168.56.101` |
| 19 | `scp` | Copiar por red | `scp archivo usuario@IP:~/` |
| 20 | `ping` | Probar conexión | `ping 192.168.56.1` |
| 21 | `ip addr` | Ver IPs | `ip addr show` |
| 22 | `ss` | Ver puertos | `ss -tulnp` |
| 23 | `echo` | Imprimir texto | `echo $PATH` |
| 24 | `env` | Ver variables | `env \| grep HOME` |
| 25 | `export` | Crear variable | `export MI_VAR="valor"` |
| 26 | `source` | Recargar config | `source ~/.bashrc` |
| 27 | `which` | Encontrar programa | `which python3` |
| 28 | `nano` | Editor de texto | `nano script.sh` |
| 29 | `man` | Manual del comando | `man grep` |
| 30 | `history` | Historial comandos | `history \| grep ssh` |

---

## Script de cierre: estado_completo.sh

```bash
#!/bin/bash
# estado_completo.sh — Resumen total del sistema
# Sesión 14 — Cierre de Linux
# IFCD0112 — Amtigravity Launch Control

VERDE="\033[0;32m"
AZUL="\033[0;34m"
AMARILLO="\033[1;33m"
ROJO="\033[0;31m"
RESET="\033[0m"
LINEA="═══════════════════════════════════════════════"

mostrar_cabecera() {
    echo -e "${AZUL}"
    echo "$LINEA"
    echo "   ESTADO DEL SISTEMA — $(hostname)"
    echo "   $(date '+%A %d de %B de %Y, %H:%M')"
    echo "   Usuario: $(whoami)"
    echo "$LINEA"
    echo -e "${RESET}"
}

mostrar_sistema() {
    echo -e "${VERDE}[SISTEMA]${RESET}"
    echo "  Hostname:     $(hostname)"
    echo "  SO:           $(lsb_release -d 2>/dev/null | cut -f2 || cat /etc/os-release | grep PRETTY | cut -d= -f2)"
    echo "  Kernel:       $(uname -r)"
    echo "  Arquitectura: $(uname -m)"
    echo "  Uptime:       $(uptime -p)"
    echo ""
}

mostrar_recursos() {
    echo -e "${VERDE}[RECURSOS]${RESET}"
    CPU_CORES=$(nproc)
    CPU_LOAD=$(cat /proc/loadavg | awk '{print $1}')
    echo "  CPU:      $CPU_CORES cores | Carga: $CPU_LOAD"
    MEM_TOTAL=$(free -h | awk '/Mem:/ {print $2}')
    MEM_USADA=$(free -h | awk '/Mem:/ {print $3}')
    MEM_PORCENT=$(free | awk '/Mem:/ {printf "%.1f", $3/$2*100}')
    echo "  Memoria:  $MEM_USADA / $MEM_TOTAL ($MEM_PORCENT%)"
    DISCO_TOTAL=$(df -h / | awk 'NR==2 {print $2}')
    DISCO_USADO=$(df -h / | awk 'NR==2 {print $3}')
    DISCO_PORCENT=$(df -h / | awk 'NR==2 {print $5}')
    echo "  Disco /:  $DISCO_USADO / $DISCO_TOTAL ($DISCO_PORCENT)"
    echo ""
}

mostrar_red() {
    echo -e "${VERDE}[RED]${RESET}"
    ip -4 addr show | grep -E "^[0-9]|inet " | while read linea; do
        if echo "$linea" | grep -q "^[0-9]"; then
            IFACE=$(echo "$linea" | awk '{print $2}' | tr -d ':')
            echo -e "  Interface: ${AMARILLO}$IFACE${RESET}"
        elif echo "$linea" | grep -q "inet "; then
            IP=$(echo "$linea" | awk '{print $2}')
            echo "    IP: $IP"
        fi
    done
    GATEWAY=$(ip route | grep default | awk '{print $3}' | head -1)
    echo "  Gateway: ${GATEWAY:-No configurado}"
    DNS=$(grep nameserver /etc/resolv.conf 2>/dev/null | head -1 | awk '{print $2}')
    echo "  DNS: ${DNS:-No configurado}"
    echo ""
}

mostrar_servicios() {
    echo -e "${VERDE}[SERVICIOS ACTIVOS]${RESET}"
    if systemctl is-active sshd &>/dev/null || systemctl is-active ssh &>/dev/null; then
        PUERTO_SSH=$(ss -tulnp | grep sshd | awk '{print $5}' | head -1)
        echo -e "  SSH:      ${VERDE}activo${RESET} ($PUERTO_SSH)"
    else
        echo -e "  SSH:      ${ROJO}inactivo${RESET}"
    fi
    for servicio in nginx apache2 postgresql docker; do
        if systemctl is-active "$servicio" &>/dev/null; then
            echo -e "  $servicio: ${VERDE}activo${RESET}"
        fi
    done
    echo ""
}

mostrar_usuarios() {
    echo -e "${VERDE}[USUARIOS CONECTADOS]${RESET}"
    who | while read linea; do
        echo "  $linea"
    done
    TOTAL=$(who | wc -l)
    echo "  Total: $TOTAL usuario(s) conectado(s)"
    echo ""
}

mostrar_procesos_top() {
    echo -e "${VERDE}[TOP 5 PROCESOS POR MEMORIA]${RESET}"
    ps aux --sort=-%mem | head -6 | tail -5 | awk '{printf "  %-10s %5s%% MEM  %s\n", $1, $4, $11}'
    echo ""
}

mostrar_docker() {
    if command -v docker &>/dev/null; then
        echo -e "${VERDE}[DOCKER]${RESET}"
        if docker info &>/dev/null; then
            CONTENEDORES=$(docker ps -q | wc -l)
            IMAGENES=$(docker images -q | wc -l)
            echo "  Estado:       activo"
            echo "  Contenedores: $CONTENEDORES corriendo"
            echo "  Imágenes:     $IMAGENES disponibles"
        else
            echo -e "  Docker instalado pero ${ROJO}no accesible${RESET} (¿permisos?)"
        fi
    else
        echo -e "${AMARILLO}[DOCKER] No instalado${RESET}"
    fi
    echo ""
}

# Ejecución principal
mostrar_cabecera
mostrar_sistema
mostrar_recursos
mostrar_red
mostrar_servicios
mostrar_usuarios
mostrar_procesos_top
mostrar_docker

echo -e "${AZUL}$LINEA${RESET}"
echo "  Generado por: estado_completo.sh"
echo "  Amtigravity Launch Control — IFCD0112"
echo -e "${AZUL}$LINEA${RESET}"
```

```bash
# Crear y ejecutar
nano ~/curso_ifcd0112/scripts/estado_completo.sh
chmod +x ~/curso_ifcd0112/scripts/estado_completo.sh
~/curso_ifcd0112/scripts/estado_completo.sh
```

---

## Función extra: espacio del proyecto

```bash
mostrar_espacio_proyecto() {
    echo -e "${VERDE}[PROYECTO LAUNCH CONTROL]${RESET}"
    PROYECTO="$HOME/curso_ifcd0112"
    if [ -d "$PROYECTO" ]; then
        ARCHIVOS=$(find "$PROYECTO" -type f | wc -l)
        TAMANO=$(du -sh "$PROYECTO" 2>/dev/null | awk '{print $1}')
        TAREAS=$(ls "$PROYECTO/tareas/" 2>/dev/null | wc -l)
        echo "  Directorio: $PROYECTO"
        echo "  Archivos:   $ARCHIVOS"
        echo "  Tamaño:     $TAMANO"
        echo "  Tareas:     $TAREAS entregadas"
    else
        echo -e "  ${ROJO}Directorio del proyecto no encontrado${RESET}"
        echo "  Crear con: mkdir -p $PROYECTO/{scripts,tareas,notas}"
    fi
    echo ""
}
```

---

# BLOQUE III — Docker: Entendiendo la Revolución

## VM vs Contenedor

_(Tus notas aquí)_

### Datos clave

| Aspecto | VM | Contenedor |
|---|---|---|
| Tamaño | 2-10 GB | 50-500 MB |
| Arranque | 1-5 minutos | 1-5 segundos |
| RAM | 1-4 GB reservados | Solo lo necesario |
| Aislamiento | Total (SO completo) | A nivel de proceso |

---

## Vocabulario Docker

| Término | Qué es | Analogía |
|---|---|---|
| **Imagen** | Plantilla de solo lectura | Receta de cocina |
| **Contenedor** | Imagen ejecutándose | Plato cocinándose |
| **Dockerfile** | Instrucciones para construir imagen | Tu receta personalizada |
| **Docker Hub** | Repositorio de imágenes | Biblioteca de recetas |
| **Volumen** | Datos persistentes | La nevera |
| **Puerto** | Acceso desde fuera | Ventanilla de servicio |
| **Compose** | Orquestador de contenedores | Menú del día |

---

## Lo que viene

| Día | Tema |
|---|---|
| 15 | Instalar Docker. Primeros contenedores. |
| 16 | Imágenes y Dockerfile. |
| 17 | Volúmenes y redes Docker. |
| 18 | Docker Compose. Stack completo Launch Control. |

---

## Comparativa Linux → Docker

| Linux (lo que hicieron) | Docker (lo que viene) |
|---|---|
| `apt install nginx` | `docker run nginx` |
| Configurar `/etc/nginx/...` | Montar config como volumen |
| `systemctl start nginx` | `docker start mi-nginx` |
| Instalar PostgreSQL manual | `docker run postgres:16` |
| `adduser` para crear usuario | Variable `POSTGRES_USER` |
| Configurar red con netplan | Docker networking (automático) |
| Backup de VM completa | `docker commit` o Dockerfile |
| "Se rompió la VM" | `docker rm` + `docker run` |

---

# BLOQUE IV — Ejercicio: Mi Guía de Supervivencia Linux

```bash
nano ~/curso_ifcd0112/notas/mi_guia_linux.md
```

### Estructura mínima

```markdown
# Mi Guía de Supervivencia Linux
# [Tu nombre] — IFCD0112, abril 2026

## 1. Comandos que uso todos los días
## 2. Cómo me conecto a Artemis
## 3. Si algo no funciona...
## 4. Mis alias favoritos
## 5. Scripts que escribí
## 6. Lo que más me costó y cómo lo resolví
## 7. Lo que quiero aprender de Docker
```

---

## Preparación para mañana: Instalar Docker

```bash
# 1. Actualizar
sudo apt update

# 2. Dependencias
sudo apt install -y ca-certificates curl

# 3. Clave GPG
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg \
  -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# 4. Repositorio
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] \
  https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# 5. Instalar
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io \
  docker-buildx-plugin docker-compose-plugin

# 6. Agregar usuario al grupo docker
sudo usermod -aG docker $USER
newgrp docker

# 7. Verificar
docker --version
docker run hello-world
```

### Troubleshooting

| Problema | Solución |
|---|---|
| `permission denied` | `sudo usermod -aG docker $USER` + cerrar/abrir sesión |
| `Cannot connect to Docker daemon` | `sudo systemctl start docker` |
| `Unable to locate package docker-ce` | Verificar repositorio (paso 4) |
| Error de GPG | Repetir pasos 3 y 4 |
| Sin espacio | `df -h` — necesita 5 GB libres |
| VM lenta | Mínimo 2 GB RAM y 2 CPUs |

### Diagnóstico rápido

```bash
docker --version
sudo systemctl status docker
docker ps
docker pull alpine
docker run --rm alpine echo "Docker funciona!"
```

---

## Tarea 14 — Entrega en Classroom

**Fecha límite:** Miércoles 29 de abril a las 09:00

1. Script `estado_completo.sh` ejecutado (captura)
2. `mi_guia_linux.md` (mínimo 100 líneas, ejemplos reales)
3. Docker instalado (`docker run hello-world` — captura)
4. Tablero Trello actualizado (Linux → DONE, Docker → TO DO)

```bash
mkdir -p ~/curso_ifcd0112/tareas/tarea_14
cp ~/curso_ifcd0112/scripts/estado_completo.sh ~/curso_ifcd0112/tareas/tarea_14/
cp ~/curso_ifcd0112/notas/mi_guia_linux.md ~/curso_ifcd0112/tareas/tarea_14/
```

---

## Tabla de referencia rápida — Docker (preview)

| Comando | Qué hace |
|---|---|
| `docker run` | Crear y ejecutar contenedor |
| `docker ps` | Ver contenedores corriendo |
| `docker ps -a` | Ver todos (incluidos parados) |
| `docker stop` | Parar un contenedor |
| `docker rm` | Eliminar un contenedor |
| `docker images` | Ver imágenes descargadas |
| `docker pull` | Descargar una imagen |
| `docker logs` | Ver salida de un contenedor |
| `docker exec` | Ejecutar comando dentro |
| `docker compose up` | Levantar servicios |
| `docker compose down` | Parar servicios |

---

> **Curso IFCD0112 — Programación con Lenguajes OO y BBDD Relacionales**
> Material complementario de estudio
>
> **Autor:** Prof. Juan Marcelo Gutierrez Miranda
> **Institución:** @TodoEconometria
>
> Este documento es un recurso bibliográfico adicional para los alumnos del curso.
> Módulo: MF0223_3 — Sistemas informáticos
>
> ---
>
> **Propiedad intelectual:** Este material didáctico, su metodología, estructura,
> ejemplos y código base son producción intelectual de Juan Marcelo Gutierrez Miranda.
> El contenido técnico de Python, PostgreSQL, Linux y sus ecosistemas pertenece
> a sus respectivos autores y comunidades; la organización pedagógica, las
> explicaciones, los diagramas y el enfoque metodológico son obra original del autor.
>
> **Hash de Certificación:** `4e8d9b1a5f6e7c3d2b1a0f9e8d7c6b5a4f3e2d1c0b9a8f7e6d5c4b3a2f1e0d9c`
