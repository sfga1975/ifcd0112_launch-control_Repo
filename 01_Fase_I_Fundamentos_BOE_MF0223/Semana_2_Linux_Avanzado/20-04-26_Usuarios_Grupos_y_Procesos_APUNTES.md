# Apuntes — 20 de Abril 2026: Usuarios, Grupos y Procesos

**Curso IFCD0112** | **Módulo:** MF0223_3 | Prof. Juan Marcelo Gutiérrez Miranda

Estos apuntes complementan lo visto en clase. Tienen los comandos
esenciales, las tablas de referencia y los conceptos clave.
Úsenlos para repasar y practicar.

---

# BLOQUE 0 — RECONEXIÓN

---

El jueves y viernes armaron la infraestructura completa de Launch Control.
Crearon 11 directorios, 22 archivos, configuraron permisos, escribieron
scripts de automatización. Todo desde la terminal.

```
╔══════════════════════════════════════════════════════════════════╗
║              EVOLUCIÓN DE LAUNCH CONTROL                        ║
╠══════════════════════════════════════════════════════════════════╣
║                                                                  ║
║  Semana 1   Terminal + ficheros + permisos + scripts             ║
║             Ejercicio integrador: infraestructura completa       ║
║                                                                  ║
║  ► HOY      Usuarios + grupos + procesos + servicios             ║
║             Launch Control necesita tripulación y monitoreo      ║
║                                                                  ║
║  Próximo    Redes + SSH: acceso remoto al servidor               ║
║                                                                  ║
╚══════════════════════════════════════════════════════════════════╝
```

Verifiquen que tienen Launch Control montado:

```bash
tree ~/launch-control | head -15
```

Si no lo tienen (faltaron jueves/viernes), lo recreamos rápido al final.

---

# BLOQUE 1 — DIAGNÓSTICO EXPRESS

---

30 minutos. Individual. Sin manual, sin internet, sin compañero.
Esto no tiene nota — es un mapa para saber dónde están.

---

## A. Completar la tabla (de memoria)

| Directorio | Para qué sirve |
|---|---|
| `/home` | |
| `/etc` | |
| `/var/log` | |
| `/tmp` | |
| `/bin` | |
| `/root` | |

---

## B. Verdadero o Falso (corregir las falsas)

1. En Linux, `Archivo.txt` y `archivo.txt` son el mismo archivo. [ ]
2. El comando `rm` envía los archivos a la papelera de reciclaje. [ ]
3. El operador `>>` sobreescribe el contenido de un archivo. [ ]
4. El permiso `755` significa: propietario lee/escribe/ejecuta, el resto lee y ejecuta. [ ]
5. Los archivos ocultos en Linux empiezan por `_` (guion bajo). [ ]
6. `chmod 644 archivo.txt` da permiso de ejecución al propietario. [ ]

---

## C. Escribir los comandos (sin ejecutar)

1. Ver en qué directorio estoy:
2. Listar todos los archivos (incluidos ocultos) con detalles:
3. Crear un directorio `proyecto` con subdirectorios `src` y `docs` en un solo comando:
4. Copiar el directorio `proyecto` a `proyecto_backup` con todo su contenido:
5. Dar permisos 755 al archivo `script.sh`:
6. Crear un archivo `info.txt` con el contenido "Hola mundo" (un solo comando):
7. Añadir la línea "Segunda línea" al final de `info.txt` sin borrar lo anterior:

---

## Corrección colectiva

### A. Respuestas de la tabla

| Directorio | Para qué sirve |
|---|---|
| `/home` | Carpetas personales de los usuarios |
| `/etc` | Configuración del sistema y servicios |
| `/var/log` | Registros (logs) del sistema |
| `/tmp` | Archivos temporales (se borran al reiniciar) |
| `/bin` | Comandos esenciales del sistema |
| `/root` | Directorio personal del superusuario (root) |

### B. Verdadero o Falso

1. **F** — Son archivos diferentes. Linux distingue mayúsculas de minúsculas.
2. **F** — `rm` elimina permanentemente. No hay papelera.
3. **F** — `>>` añade al final. `>` es el que sobreescribe.
4. **V** — 7=rwx para owner, 5=r-x para group y others.
5. **F** — Empiezan por `.` (punto), no por guion bajo.
6. **F** — 644 = rw-r--r--. No da ejecución a nadie.

### C. Respuestas de comandos

1. `pwd`
2. `ls -la`
3. `mkdir -p proyecto/{src,docs}`
4. `cp -r proyecto proyecto_backup`
5. `chmod 755 script.sh`
6. `echo "Hola mundo" > info.txt`
7. `echo "Segunda línea" >> info.txt`

---

## 1. Modelo multiusuario de Linux

Linux está diseñado para múltiples usuarios conectados al mismo
servidor simultáneamente. Cada usuario tiene:

- **UID** — identificador numérico único (el kernel trabaja con números)
- **GID** — grupo primario (los archivos nuevos se crean con este grupo)
- **Home** — directorio personal (`/home/tu_usuario`)
- **Shell** — programa que se ejecuta al abrir terminal (`/bin/bash`)

Regla de UIDs:

| Rango | Tipo |
|---|---|
| 0 | root (superusuario) |
| 1-999 | Usuarios de sistema (servicios) |
| >= 1000 | Usuarios humanos |

---

## 2. Archivos clave del sistema

### /etc/passwd — registro de usuarios

```
alumno:x:1000:1000:Alumno IFCD0112:/home/alumno:/bin/bash
login  │  UID  GID  descripción      home          shell
       └── "x" = contraseña en /etc/shadow
```

```bash
grep alumno /etc/passwd         # Tu entrada
wc -l /etc/passwd               # Total de usuarios
```

### /etc/shadow — hashes de contraseñas

Solo root puede leerlo. Contiene hashes irreversibles, no contraseñas.

```bash
sudo cat /etc/shadow | head -5  # Requiere sudo
ls -l /etc/passwd /etc/shadow   # Comparar permisos
```

### /etc/group — grupos del sistema

```
sudo:x:27:alumno
nombre:pwd:GID:miembros
```

```bash
groups                          # Tus grupos
grep sudo /etc/group            # Miembros de sudo
```

---

## 3. Comandos de gestión de usuarios

| Acción | Comando |
|---|---|
| Ver quién soy | `whoami` |
| Mi UID, GID y grupos | `id` |
| Crear usuario | `sudo adduser nombre` |
| Cambiar contraseña | `sudo passwd nombre` |
| Añadir a grupo | `sudo usermod -aG grupo nombre` |
| Eliminar usuario + home | `sudo deluser --remove-home nombre` |
| Crear grupo | `sudo groupadd nombre` |
| Eliminar grupo | `sudo groupdel nombre` |
| Ver miembros de grupo | `getent group nombre` |
| Quitar de un grupo | `sudo gpasswd -d nombre grupo` |
| Buscar archivos huérfanos | `sudo find / -nouser -ls 2>/dev/null` |

**CRÍTICO:** `usermod -aG` (con `-a`) AÑADE al grupo.
Sin `-a` REEMPLAZA todos los grupos. Nunca olviden la `-a`.

---

## 4. su vs sudo

| | `su` | `sudo` |
|---|---|---|
| Qué hace | Abre sesión como otro usuario | Eleva un solo comando |
| Contraseña | La del usuario destino | La tuya propia |
| Registro | No queda detalle | Todo en /var/log/auth.log |
| Riesgo | Te quedas como root | Solo ese comando |
| En producción | Evitar | Estándar |

```bash
su - nombre          # Sesión completa (pide contraseña del otro)
sudo comando         # Un comando como root (pide tu contraseña)
sudo -l              # Ver qué permisos de sudo tienes
```

---

## 5. Directorio compartido con setgid

Para que un equipo trabaje en el mismo directorio:

```bash
sudo groupadd equipo                           # 1. Crear grupo
sudo mkdir -p /opt/proyecto                    # 2. Crear directorio
sudo chown root:equipo /opt/proyecto           # 3. Asignar grupo
sudo chmod 2770 /opt/proyecto                  # 4. Permisos + setgid
sudo usermod -aG equipo alumno                 # 5. Añadir usuarios
newgrp equipo                                  # 6. Activar sin cerrar sesión
```

El `2` en `2770` es el bit **setgid**: los archivos creados dentro
heredan el grupo del directorio, no el grupo primario del usuario.

---

## 6. Todo es un proceso

Cada programa en ejecución es un proceso con:

| Propiedad | Descripción |
|---|---|
| PID | Identificador único |
| PPID | PID del proceso padre |
| Estado | R(unning), S(leeping), T(stopped), Z(ombie) |
| Usuario | Bajo qué identidad corre |

```bash
echo $$                         # PID de tu terminal
echo $PPID                      # PID del padre
ps aux | wc -l                  # Total de procesos
```

---

## 7. Comandos de procesos

| Acción | Comando |
|---|---|
| Listar todos los procesos | `ps aux` |
| Ordenar por CPU | `ps aux --sort=-%cpu \| head -6` |
| Ordenar por memoria | `ps aux --sort=-%mem \| head -6` |
| Buscar un proceso | `ps aux \| grep [n]ombre` |
| Árbol de procesos | `pstree -p` |
| Monitor interactivo | `top` (incluido) / `htop` (instalar) |

### Señales para controlar procesos

| Señal | Número | Efecto | Comando |
|---|---|---|---|
| SIGTERM | 15 | Termina limpiamente | `kill PID` |
| SIGKILL | 9 | Mata inmediatamente | `kill -9 PID` |
| SIGSTOP | 19 | Pausa el proceso | `kill -STOP PID` |
| SIGCONT | 18 | Reanuda el proceso | `kill -CONT PID` |
| SIGINT | 2 | Interrumpe | `Ctrl+C` |
| SIGTSTP | 20 | Pausa desde terminal | `Ctrl+Z` |

**Regla:** siempre `kill PID` primero. Solo `kill -9` si no muere.

### Background y foreground

```bash
comando &             # Lanzar en background
jobs                  # Ver procesos en background
fg %1                 # Traer job 1 al foreground
bg %1                 # Reanudar job 1 en background
Ctrl+Z                # Pausar proceso actual
Ctrl+C                # Matar proceso actual
```

---

## 8. Servicios con systemctl

Un servicio (daemon) corre 24/7 sin terminal. Se gestiona con `systemctl`:

| Acción | Comando |
|---|---|
| Ver estado | `sudo systemctl status servicio` |
| Arrancar | `sudo systemctl start servicio` |
| Parar | `sudo systemctl stop servicio` |
| Reiniciar | `sudo systemctl restart servicio` |
| Habilitar en boot | `sudo systemctl enable servicio` |
| Deshabilitar del boot | `sudo systemctl disable servicio` |
| Listar todos | `sudo systemctl list-units --type=service` |
| Ver fallidos | `sudo systemctl list-units --type=service --state=failed` |

`stop` ≠ `disable`: stop lo para ahora, disable evita que arranque en el boot.

---

## 9. Logs con journalctl

```bash
sudo journalctl -u ssh -n 50    # Últimas 50 líneas de SSH
sudo journalctl -p err          # Solo errores
sudo journalctl -u ssh -f       # Seguir en tiempo real (Ctrl+C para salir)
sudo journalctl --disk-usage    # Espacio que ocupan los logs
sudo journalctl --vacuum-time=7d # Limpiar logs de más de 7 días
```

---

## 10. Diagnóstico de disco

```bash
df -h                           # Vista global: cuánto queda
df -h /                         # Solo la partición principal
du -sh /var/log                 # Tamaño de un directorio
sudo du -h --max-depth=1 / 2>/dev/null | sort -hr | head -10  # Los más grandes
lsblk                           # Estructura del disco
lsblk -f                        # Con tipo de sistema de archivos
```

Si `Use%` de `/` supera el 80%, investigar con `du` qué lo consume.

---

## 11. Auditoría

```bash
sudo cat /var/log/auth.log | grep sudo | tail -10  # Usos de sudo
last | head -10                                     # Últimos logins
who                                                 # Sesiones activas
sudo lastb | head -10                               # Intentos fallidos
```

---

## Resumen visual

```
ANTES DE HOY                         DESPUÉS DE HOY
─────────────                        ──────────────
"Linux tiene usuarios"               Leen /etc/passwd línea a línea
                                     Crean/modifican/eliminan usuarios
                                     Montan directorios de grupo con setgid

"sudo es para ser admin"             Distinguen su de sudo
                                     Leen el log de auditoría

"Los programas se ejecutan"          Todo es un proceso con PID
                                     Matan, pausan y reanudan procesos
                                     Gestionan servicios con systemctl
                                     Leen logs en tiempo real
                                     Diagnostican disco con df/du
```

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
> Módulo: MF0223_3 — Sistemas informáticos
>
> ---
>
> **Propiedad intelectual:** Este material didáctico, su metodología, estructura,
> ejemplos y código base son producción intelectual de Juan Marcelo Gutiérrez Miranda.
> El contenido técnico de Linux y sus ecosistemas pertenece a sus respectivos autores
> y comunidades; la organización pedagógica, las explicaciones, los diagramas y el
> enfoque metodológico son obra original del autor.
