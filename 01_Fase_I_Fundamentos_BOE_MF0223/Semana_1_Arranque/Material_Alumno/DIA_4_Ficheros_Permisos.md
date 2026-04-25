# Día 4: Sistema de ficheros, permisos y lectura de ficheros en Linux

Ayer dominaron la creación, copia y eliminación de archivos y directorios.
Hoy entramos en lo que hace a Linux realmente diferente de Windows: el sistema
de permisos. Quién puede leer, escribir y ejecutar cada archivo es una decisión
explícita, no automática.

Prof. Juan Marcelo Gutiérrez Miranda

**Curso IFCD0112 — Semana 1, Día 4 (Martes)**
**Objetivo:** Entender el sistema de ficheros FHS, dominar permisos (chmod, chown)
y saber leer y editar archivos desde la terminal.

---

## PARTE I -- EL ÁRBOL DE FICHEROS DE LINUX

### 1.1 La filosofía del árbol único

En Windows estamos acostumbrados a trabajar con unidades: C:\, D:\, E:\. Cada
disco, cada partición, cada USB tiene su propia letra. En Linux esto NO existe.

Linux organiza TODO el sistema en un único árbol que nace de un solo punto
llamado **raíz** y representado por la barra `/`. Todo -- absolutamente todo --
cuelga de ahí: el disco duro principal, los USB, los discos externos, las
particiones adicionales... todo se monta dentro de este árbol.

```
Windows:                          Linux:
C:\                               /
C:\Windows                        /etc
C:\Users\alumno                   /home/alumno
D:\  (segundo disco)              /mnt/datos  (montado dentro del árbol)
E:\  (USB)                        /media/alumno/usb  (montado dentro del árbol)
```

La razón histórica es sencilla: Unix (el abuelo de Linux) nació en los años 70
pensando en mainframes con múltiples terminales. Un solo árbol facilitaba la
administración centralizada. Esa filosofía se mantiene hasta hoy y es una de
las razones por las que Linux domina en servidores: un administrador sabe
exactamente dónde buscar cada cosa.

### 1.2 El estándar FHS (Filesystem Hierarchy Standard)

Para que todos los Linux se parezcan entre sí, existe un estándar llamado
**FHS** (Filesystem Hierarchy Standard). Este estándar define qué directorio
va en la raíz y para qué sirve cada uno. Gracias a él, si aprenden dónde están
las cosas en Ubuntu, sabrán encontrarlas también en Debian, Fedora, CentOS o
cualquier otra distribución.

No necesitan memorizar el FHS entero. Lo que necesitan es entender la lógica:
cada directorio tiene un propósito claro y bien definido.

### 1.3 Los directorios principales, uno por uno

A continuación tienen el árbol completo con los directorios más importantes.
Léanlo con calma y luego los exploraremos juntos en la terminal.

```
/
├── bin/         Binarios esenciales del sistema
├── sbin/        Binarios de administración del sistema
├── boot/        Ficheros de arranque (kernel, grub)
├── dev/         Dispositivos de hardware (como ficheros)
├── etc/         Configuración del sistema y de servicios
├── home/        Directorios personales de los usuarios
│   ├── alumno/
│   ├── profesor/
│   └── maria/
├── lib/         Librerías compartidas del sistema
├── media/       Punto de montaje para medios extraíbles
├── mnt/         Punto de montaje temporal para discos
├── opt/         Software de terceros
├── proc/        Sistema de ficheros virtual (procesos)
├── root/        Directorio personal del superusuario
├── run/         Datos de ejecución (PIDs, sockets)
├── srv/         Datos de servicios (web, ftp)
├── sys/         Sistema de ficheros virtual (hardware)
├── tmp/         Ficheros temporales
├── usr/         Programas instalados
│   ├── bin/     Binarios de usuario
│   ├── lib/     Librerías de usuario
│   ├── local/   Software instalado manualmente
│   └── share/   Datos compartidos (documentación, iconos)
└── var/         Datos variables
    ├── log/     Logs del sistema
    ├── cache/   Cachés de aplicaciones
    ├── mail/    Correo local
    └── tmp/     Temporales persistentes
```

### 1.4 Explicación detallada de cada directorio

#### `/` -- La raíz

Es el punto de partida de todo el sistema de ficheros. Solo el usuario root
tiene permiso de escritura aquí. No deben crear ficheros directamente en `/`.

```bash
ls /
```

#### `/bin` -- Binarios esenciales

Contiene los comandos básicos que necesita el sistema para funcionar, incluso
en modo de emergencia: `ls`, `cp`, `mv`, `rm`, `cat`, `echo`, `bash`...

Son los comandos que usan todos los días. Cuando escriben `ls`, el sistema
busca el programa `/bin/ls` y lo ejecuta.

```bash
ls /bin
# Verán cientos de comandos
which ls
# /bin/ls o /usr/bin/ls según la distribución
```

#### `/sbin` -- Binarios de administración

Similar a `/bin`, pero contiene comandos destinados al administrador del
sistema: `fdisk` (gestionar discos), `iptables` (firewall), `reboot`
(reiniciar), `shutdown` (apagar)...

Un usuario normal puede verlos pero no siempre ejecutarlos.

```bash
ls /sbin
which reboot
```

#### `/boot` -- Arranque del sistema

Aquí vive el kernel de Linux (el núcleo del sistema operativo) y los ficheros
que GRUB necesita para arrancar la máquina. Normalmente no tocarán nada aquí.

```bash
ls /boot
# Verán ficheros como vmlinuz (el kernel) e initrd (disco RAM inicial)
```

#### `/dev` -- Dispositivos

En Linux, **todo es un fichero**, incluido el hardware. Cada disco, cada
partición, cada terminal se representa como un fichero dentro de `/dev`.

```bash
ls /dev
# Algunos ejemplos:
# /dev/sda    -> primer disco duro
# /dev/sda1   -> primera partición del primer disco
# /dev/null   -> el agujero negro (descarta todo lo que le envían)
# /dev/zero   -> fuente infinita de ceros
# /dev/tty    -> tu terminal actual
```

`/dev/null` es especialmente útil. Cuando un comando produce salida que no les
interesa, pueden redirigirla ahí:

```bash
comando_ruidoso > /dev/null 2>&1
# Descarta tanto la salida normal como los errores
```

#### `/etc` -- Configuración del sistema

Este es uno de los directorios más importantes. Contiene los ficheros de
configuración de prácticamente todo: el nombre de la máquina, los usuarios,
la red, los servicios instalados...

```bash
ls /etc

# Ficheros que vamos a explorar hoy:
cat /etc/hostname       # Nombre de la máquina
cat /etc/hosts          # Tabla de resolución de nombres local
cat /etc/passwd         # Lista de usuarios del sistema
cat /etc/os-release     # Información de la distribución
```

El fichero `/etc/passwd` NO contiene contraseñas (a pesar del nombre). Es una
lista de todos los usuarios con formato:

```
nombre:x:UID:GID:descripcion:directorio_home:shell
alumno:x:1000:1000:Alumno IFCD0112:/home/alumno:/bin/bash
```

Las contraseñas (cifradas) están en `/etc/shadow`, que solo root puede leer.

#### `/home` -- Directorios personales

Cada usuario del sistema tiene su propio directorio dentro de `/home`. Es el
equivalente a `C:\Users\nombre` en Windows.

```bash
ls /home
# alumno  profesor  maria ...

ls -la /home/alumno
# Aquí están sus ficheros: Documentos, Descargas, .bashrc...
```

El símbolo `~` (tilde) es un atajo que siempre apunta a su directorio
personal:

```bash
echo ~
# /home/alumno

cd ~
pwd
# /home/alumno
```

#### `/var` -- Datos variables

Contiene datos que cambian constantemente durante el funcionamiento del
sistema: logs, bases de datos, cachés, colas de correo...

```bash
ls /var
# cache  lib  local  lock  log  mail  opt  run  spool  tmp
```

El subdirectorio más importante para nosotros es `/var/log`:

```bash
ls /var/log
# auth.log    -> intentos de autenticación
# syslog      -> mensajes generales del sistema
# kern.log    -> mensajes del kernel
# dpkg.log    -> historial de paquetes instalados
# apt/        -> logs del gestor de paquetes
```

Muchos de estos ficheros requieren `sudo` para leerlos.

#### `/var/log` -- Logs del sistema

Este directorio merece atención especial. Cuando algo falla en un servidor
Linux, lo primero que hace un administrador es mirar los logs.

```bash
# Ver los últimos mensajes del sistema
sudo tail -20 /var/log/syslog

# Ver los últimos intentos de login
sudo tail -20 /var/log/auth.log

# Ver qué paquetes se instalaron recientemente
tail -20 /var/log/dpkg.log
```

En el mundo real, saber leer logs es una habilidad fundamental.

#### `/tmp` -- Ficheros temporales

Cualquier usuario puede escribir aquí. El contenido se borra al reiniciar
la máquina (en la mayoría de distribuciones).

```bash
ls /tmp
# Verán ficheros temporales de diversas aplicaciones

# Pueden crear ficheros temporales para pruebas:
echo "prueba" > /tmp/mi_prueba.txt
cat /tmp/mi_prueba.txt
# Funcionará hasta el próximo reinicio
```

#### `/usr` -- Programas de usuario

Contiene la mayor parte del software instalado en el sistema. Es como el
`C:\Program Files` de Windows, pero mejor organizado.

```bash
ls /usr
# bin  games  include  lib  local  sbin  share  src

ls /usr/bin | head -30
# Cientos de programas: python3, git, nano, vim...
```

#### `/usr/local` -- Software instalado manualmente

Cuando compilan e instalan software a mano (no a través del gestor de
paquetes), va aquí. Esto evita conflictos con el software del sistema.

```bash
ls /usr/local
# bin  etc  games  include  lib  man  sbin  share  src
```

#### `/opt` -- Software de terceros

Otra ubicación para software de terceros, pero pensada para aplicaciones
completas que vienen con su propio árbol de directorios: Google Chrome,
Visual Studio Code, herramientas de empresa...

```bash
ls /opt
# Puede estar vacío o contener software como google, containerd, etc.
```

#### `/proc` -- Procesos y sistema virtual

No es un directorio real en el disco. Es un sistema de ficheros **virtual**
que el kernel genera en tiempo real. Contiene información sobre cada proceso
y sobre el hardware.

```bash
cat /proc/cpuinfo     # Información del procesador
cat /proc/meminfo     # Información de la memoria
cat /proc/version     # Versión del kernel
ls /proc              # Verán números: cada uno es un proceso (PID)
```

#### `/sys` -- Hardware virtual

Similar a `/proc`, pero organizado de forma más estructurada para representar
dispositivos y sus controladores. Raramente lo usarán directamente.

#### `/mnt` y `/media` -- Puntos de montaje

`/mnt` se usa para montar discos o particiones manualmente.
`/media` se usa para medios extraíbles (USB, CD) que se montan automáticamente.

```bash
ls /mnt
ls /media
# Si conectan un USB, aparecerá en /media/alumno/nombre_usb
```

#### `/root` -- Directorio personal de root

Es el home del superusuario. NO está dentro de `/home` porque `/home` podría
estar en otro disco y no estar disponible en modo de emergencia.

```bash
sudo ls /root
# Normalmente está vacío o casi vacío
```

### 1.5 Dónde van las cosas en el mundo real

Esta es una tabla que les servirá como referencia rápida:

| Necesitan... | Lo encuentran en... |
|---|---|
| Configuración de un servicio | `/etc/nombre_servicio/` |
| Logs de un servicio | `/var/log/` |
| Software instalado con apt | `/usr/bin/`, `/usr/lib/` |
| Software instalado a mano | `/usr/local/bin/` |
| Software de terceros completo | `/opt/` |
| Sus ficheros personales | `/home/su_usuario/` |
| Ficheros temporales | `/tmp/` |
| Información del hardware | `/proc/`, `/sys/` |

### 1.6 Exploración guiada en la terminal

Ejecuten los siguientes comandos en orden. Observen la salida de cada uno
y apunten lo que les llame la atención.

```bash
# 1. Ir a la raíz y ver qué hay
cd /
ls

# 2. Ver con detalle
ls -l

# 3. Ver la configuración del sistema
ls /etc | head -30

# 4. Leer el nombre de la máquina
cat /etc/hostname

# 5. Ver la tabla de hosts
cat /etc/hosts

# 6. Ver los usuarios del sistema
cat /etc/passwd

# 7. Ver información de la distribución
cat /etc/os-release

# 8. Explorar los logs
ls /var/log

# 9. Ver el contenido de /tmp
ls /tmp

# 10. Ver los programas instalados (solo los 20 primeros)
ls /usr/bin | head -20

# 11. Información del procesador (solo las 10 primeras líneas)
head -10 /proc/cpuinfo

# 12. Volver a casa
cd ~
pwd
```

---

## PARTE II -- NAVEGAR EL ÁRBOL: RUTAS ABSOLUTAS Y RELATIVAS

### 2.1 Rutas absolutas

Una ruta absoluta indica la ubicación completa de un fichero o directorio
partiendo desde la raíz `/`. Siempre empieza por `/`.

```bash
# Ejemplos de rutas absolutas:
/home/alumno/Documentos
/etc/hostname
/var/log/syslog
/usr/bin/python3
```

No importa dónde estén en el sistema de ficheros: una ruta absoluta siempre
funciona porque parte desde la raíz.

```bash
# Estén donde estén, esto siempre funciona:
cat /etc/hostname
ls /var/log
```

### 2.2 Rutas relativas

Una ruta relativa indica la ubicación partiendo desde el directorio en el que
están ahora (el directorio de trabajo actual, que pueden ver con `pwd`).

```bash
# Si están en /home/alumno:
pwd
# /home/alumno

ls Documentos
# Equivale a: ls /home/alumno/Documentos

# Si están en /var:
cd /var
ls log
# Equivale a: ls /var/log
```

La diferencia clave: las rutas relativas NO empiezan por `/`.

### 2.3 Atajos de navegación

Linux ofrece varios atajos que hacen la navegación mucho más rápida:

| Atajo | Significado | Ejemplo |
|---|---|---|
| `~` | Tu directorio personal (/home/alumno) | `cd ~` |
| `.` | El directorio actual | `ls .` (igual que `ls`) |
| `..` | El directorio padre (un nivel arriba) | `cd ..` |
| `-` | El directorio anterior (donde estabas antes) | `cd -` |

Veamos cada uno en acción:

```bash
# ~ (tilde) -- ir a casa desde cualquier sitio
cd /var/log
pwd
# /var/log

cd ~
pwd
# /home/alumno

# . (punto) -- directorio actual
# Útil sobre todo para ejecutar scripts:
./mi_script.sh
# Significa: ejecuta mi_script.sh que está AQUÍ MISMO

# .. (dos puntos) -- subir un nivel
cd /home/alumno/Documentos
pwd
# /home/alumno/Documentos

cd ..
pwd
# /home/alumno

# Pueden encadenar ..:
cd /home/alumno/Documentos
cd ../..
pwd
# /home

# - (guión) -- volver al directorio anterior
cd /etc
cd /var/log
cd -
pwd
# /etc (vuelve al anterior)
```

### 2.4 Completado con tabulador (Tab completion)

Esta es probablemente la herramienta de productividad más importante que
aprenderán en este curso. Funciona así:

1. Empiezan a escribir un nombre de fichero o directorio
2. Pulsan la tecla **Tab**
3. El shell completa automáticamente si solo hay una opción
4. Si hay varias opciones, pulsan **Tab dos veces** para ver la lista

```bash
# Escriban esto y pulsen Tab:
cd /ho[Tab]
# Se completa a: cd /home/

cd /home/al[Tab]
# Se completa a: cd /home/alumno/

cat /etc/host[Tab]
# Si hay varios ficheros que empiezan por "host", pulsen Tab dos veces:
# hostname  hosts  hosts.allow  hosts.deny

cat /etc/hostn[Tab]
# Se completa a: cat /etc/hostname
```

Regla de oro: **si no están usando Tab constantemente, están escribiendo
demasiado.** Los usuarios experimentados de Linux pulsan Tab cada pocos
caracteres. Esto no solo ahorra tiempo sino que evita errores de escritura.

### 2.5 Historial de comandos

Bash recuerda todos los comandos que han ejecutado. Esto es extremadamente
útil para repetir comandos o buscar algo que hicieron antes.

| Acción | Tecla/Comando |
|---|---|
| Comando anterior | Flecha arriba |
| Comando siguiente | Flecha abajo |
| Ver todo el historial | `history` |
| Buscar en el historial | **Ctrl+R** y escribir |
| Repetir último comando | `!!` |
| Repetir último comando con sudo | `sudo !!` |

```bash
# Ver los últimos 20 comandos
history | tail -20

# Buscar un comando anterior:
# 1. Pulsen Ctrl+R
# 2. Empiecen a escribir (por ejemplo "cat")
# 3. Aparecerá el último comando que contenía "cat"
# 4. Pulsen Enter para ejecutarlo o Ctrl+R para buscar el siguiente

# Repetir el último comando con sudo:
cat /var/log/auth.log
# Permission denied
sudo !!
# Ejecuta: sudo cat /var/log/auth.log
```

### 2.6 Ejercicios de navegación

Realicen los siguientes ejercicios en orden. Después de cada comando,
ejecuten `pwd` para verificar dónde están.

```bash
# 1. Vayan a la raíz
cd /
pwd

# 2. Vayan a /var/log
cd /var/log
pwd

# 3. Suban un nivel (a /var)
cd ..
pwd

# 4. Suban otro nivel (a /)
cd ..
pwd

# 5. Vayan a su home
cd ~
pwd

# 6. Vayan a /etc usando ruta absoluta
cd /etc
pwd

# 7. Vuelvan al directorio anterior (home)
cd -
pwd

# 8. Vayan a /home/alumno/Documentos (si existe) o créenlo
mkdir -p ~/Documentos
cd ~/Documentos
pwd

# 9. Vayan a /tmp con ruta absoluta
cd /tmp
pwd

# 10. Vuelvan a casa
cd ~
pwd
```

---

## PARTE III -- PERMISOS EN LINUX

### 3.1 Por qué existen los permisos

Linux es un sistema **multiusuario**. Esto significa que múltiples personas
pueden usar el mismo equipo, cada una con su propia cuenta. Los permisos
existen para:

1. **Proteger ficheros privados**: que nadie lea tus documentos sin permiso
2. **Proteger el sistema**: que un usuario normal no pueda borrar ficheros
   críticos del sistema operativo
3. **Controlar la ejecución**: que no cualquiera pueda ejecutar cualquier
   programa

Aunque en sus entornos solo hay un usuario, los permisos siguen siendo
fundamentales. Cualquier servidor Linux tiene múltiples usuarios y servicios,
y los permisos son la primera línea de defensa.

### 3.2 Anatomía completa de ls -l

El comando `ls -l` muestra información detallada. Vamos a desgranar CADA
campo de la salida:

```bash
ls -l /home/alumno
```

Ejemplo de salida:

```
drwxr-xr-x  2  alumno  alumno  4096  abr 10 09:15  Documentos
-rw-r--r--  1  alumno  alumno   420  abr  9 18:30  notas.txt
lrwxrwxrwx  1  alumno  alumno    15  abr  8 12:00  enlace -> /tmp/original
```

Desglose campo por campo:

```
d   rwx   r-x   r-x     2     alumno   alumno   4096   abr 10 09:15   Documentos
|   |     |     |       |     |        |        |      |               |
|   |     |     |       |     |        |        |      |               └─ NOMBRE
|   |     |     |       |     |        |        |      └─ FECHA de última
|   |     |     |       |     |        |        |         modificación
|   |     |     |       |     |        |        └─ TAMAÑO en bytes
|   |     |     |       |     |        └─ GRUPO propietario
|   |     |     |       |     └─ USUARIO propietario (dueño)
|   |     |     |       └─ NÚMERO DE ENLACES
|   |     |     └─ Permisos para OTROS (everyone else)
|   |     └─ Permisos para el GRUPO
|   └─ Permisos para el DUEÑO (owner/user)
└─ TIPO de fichero
```

### 3.3 Tipos de fichero

El primer carácter indica de qué tipo es la entrada:

| Carácter | Tipo | Descripción |
|---|---|---|
| `-` | Fichero regular | Un fichero normal (texto, binario, imagen...) |
| `d` | Directorio | Una carpeta |
| `l` | Enlace simbólico | Un acceso directo (como un shortcut de Windows) |
| `b` | Dispositivo de bloque | Un disco o partición (en /dev) |
| `c` | Dispositivo de carácter | Una terminal, un puerto serial (en /dev) |
| `p` | Pipe con nombre | Canal de comunicación entre procesos |
| `s` | Socket | Canal de comunicación de red local |

Los que verán habitualmente son `-`, `d` y `l`. Los demás son más avanzados.

```bash
# Ver ejemplos de cada tipo:
ls -l /home/alumno          # - (ficheros) y d (directorios)
ls -l /dev/sda              # b (dispositivo de bloque)
ls -l /dev/tty              # c (dispositivo de carácter)
```

### 3.4 Significado de rwx para ficheros vs directorios

Esta es una de las cosas más importantes de esta sesión. Los permisos r, w, x
significan cosas DIFERENTES según se apliquen a un fichero o a un directorio.

#### Para FICHEROS:

| Permiso | Significado |
|---|---|
| `r` (read) | Puedes leer el contenido del fichero (cat, less, head...) |
| `w` (write) | Puedes modificar el contenido del fichero |
| `x` (execute) | Puedes ejecutar el fichero como un programa |

#### Para DIRECTORIOS:

| Permiso | Significado |
|---|---|
| `r` (read) | Puedes ver el listado de ficheros dentro (ls) |
| `w` (write) | Puedes crear, borrar y renombrar ficheros dentro |
| `x` (execute) | Puedes entrar en el directorio (cd) |

La diferencia más importante: **`x` en un directorio NO significa ejecutar**.
Significa que puedes entrar con `cd`. Sin permiso `x` en un directorio, no
puedes ni acceder a él ni a nada que contenga, aunque tengas `r`.

```bash
# Demostración práctica:
mkdir /tmp/prueba_permisos
touch /tmp/prueba_permisos/fichero.txt

# Quitar permiso x al directorio:
chmod -x /tmp/prueba_permisos
ls /tmp/prueba_permisos
# Puede que vean los nombres pero no los detalles

cd /tmp/prueba_permisos
# bash: cd: /tmp/prueba_permisos: Permission denied

# Restaurar:
chmod +x /tmp/prueba_permisos
```

### 3.5 Los tres niveles de permisos

Cada fichero tiene permisos definidos para tres categorías de usuarios:

```
rwx   r-x   r-x
 |     |     |
 |     |     └── others (o): todos los demás usuarios del sistema
 |     └──────── group (g): usuarios que pertenecen al grupo propietario
 └────────────── user/owner (u): el dueño del fichero
```

**User (u) -- el dueño:** Normalmente es quien creó el fichero. El dueño
puede cambiar los permisos de sus propios ficheros.

**Group (g) -- el grupo:** Cada usuario pertenece al menos a un grupo. Los
ficheros tienen un grupo asociado. Todos los miembros de ese grupo tienen los
permisos de la columna central.

**Others (o) -- otros:** Todos los usuarios del sistema que no son el dueño
ni pertenecen al grupo.

### 3.6 Analogía: el edificio de apartamentos

Piensen en un edificio de apartamentos:

- **User (dueño)** = el inquilino del piso. Tiene las llaves de su puerta,
  puede decorar su piso, dejar entrar a quien quiera.

- **Group (grupo)** = los vecinos del edificio. Tienen acceso al portal, al
  garaje comunitario, a la azotea. No pueden entrar en tu piso sin tu permiso.

- **Others (otros)** = la gente de la calle. No tienen acceso al portal. Solo
  pueden ver el edificio por fuera.

Los permisos definen exactamente qué puede hacer cada nivel. Un fichero con
permisos `rwxr-x---` sería como:
- Inquilino (user): acceso completo (rwx)
- Vecinos (group): pueden ver y acceder pero no modificar (r-x)
- Calle (others): sin acceso (---)

### 3.7 Ejemplos reales para interpretar

Practiquen leyendo estos permisos:

```
-rw-r--r--  alumno  alumno  notas.txt
```
- Tipo: fichero regular (-)
- Dueño (alumno): puede leer y escribir (rw-)
- Grupo (alumno): solo puede leer (r--)
- Otros: solo pueden leer (r--)
- Interpretación: un fichero que todo el mundo puede leer pero solo el dueño
  puede modificar. Es el permiso por defecto de la mayoría de ficheros.

```
drwxr-xr-x  alumno  alumno  Documentos
```
- Tipo: directorio (d)
- Dueño: acceso completo al directorio (rwx)
- Grupo: puede entrar y ver el contenido (r-x)
- Otros: pueden entrar y ver el contenido (r-x)
- Interpretación: un directorio abierto. El permiso por defecto de la mayoría
  de directorios.

```
-rwx------  alumno  alumno  mi_script.sh
```
- Tipo: fichero regular (-)
- Dueño: acceso completo, puede ejecutar (rwx)
- Grupo: sin acceso (---)
- Otros: sin acceso (---)
- Interpretación: un script ejecutable privado. Solo el dueño puede verlo,
  modificarlo y ejecutarlo.

```
drwx------  alumno  alumno  .ssh
```
- Tipo: directorio (d)
- Dueño: acceso completo (rwx)
- Grupo y otros: sin acceso (---)
- Interpretación: directorio completamente privado. Típico para .ssh que
  contiene claves criptográficas.

```
-rw-------  root  root  /etc/shadow
```
- Solo root puede leer y escribir. Nadie más. Es el fichero de contraseñas.

---

## PARTE IV -- MODIFICAR PERMISOS: chmod

### 4.1 Notación simbólica

La notación simbólica usa letras para indicar a quién, qué acción y qué
permiso:

**A quién:**
| Letra | Significado |
|---|---|
| `u` | user (dueño) |
| `g` | group (grupo) |
| `o` | others (otros) |
| `a` | all (todos: u+g+o) |

**Qué acción:**
| Símbolo | Significado |
|---|---|
| `+` | Añadir permiso |
| `-` | Quitar permiso |
| `=` | Establecer exactamente (quita todo lo anterior y pone esto) |

**Qué permiso:**
| Letra | Significado |
|---|---|
| `r` | read (lectura) |
| `w` | write (escritura) |
| `x` | execute (ejecución) |

Se combinan así: `chmod [quien][accion][permiso] fichero`

#### Ejemplos con notación simbólica

```bash
# Dar permiso de ejecución al dueño
chmod u+x script.sh

# Quitar permiso de escritura al grupo
chmod g-w documento.txt

# Dar lectura a otros
chmod o+r informe.txt

# Dar ejecución a todos
chmod a+x programa

# Quitar todos los permisos a otros
chmod o-rwx secreto.txt

# Establecer permisos exactos para el dueño: solo lectura y escritura
chmod u=rw fichero.txt

# Combinar varios cambios separados por coma:
chmod u+x,g-w,o-r fichero.txt

# Dar rwx al dueño, rx al grupo, nada a otros:
chmod u=rwx,g=rx,o= directorio
```

### 4.2 Notación octal (numérica)

La notación octal usa un número de tres cifras. Cada cifra representa los
permisos de un nivel (user, group, others) y es la suma de:

```
4 = read  (r)
2 = write (w)
1 = execute (x)
0 = sin permiso (-)
```

Para obtener el número de cada nivel, sumen los permisos que quieren dar:

```
rwx = 4 + 2 + 1 = 7
rw- = 4 + 2 + 0 = 6
r-x = 4 + 0 + 1 = 5
r-- = 4 + 0 + 0 = 4
-wx = 0 + 2 + 1 = 3
-w- = 0 + 2 + 0 = 2
--x = 0 + 0 + 1 = 1
--- = 0 + 0 + 0 = 0
```

Y el número completo de chmod usa tres cifras: user + group + others.

### 4.3 Tabla completa de conversiones

```
Octal   Binario   Permisos   Significado
  0       000       ---       Sin permisos
  1       001       --x       Solo ejecución
  2       010       -w-       Solo escritura
  3       011       -wx       Escritura y ejecución
  4       100       r--       Solo lectura
  5       101       r-x       Lectura y ejecución
  6       110       rw-       Lectura y escritura
  7       111       rwx       Todos los permisos
```

### 4.4 Patrones de permisos más comunes

Estos son los que usarán el 90% del tiempo:

| Octal | Permisos | Uso típico |
|---|---|---|
| `644` | rw-r--r-- | Fichero normal (texto, configuración) |
| `755` | rwxr-xr-x | Script ejecutable o directorio estándar |
| `700` | rwx------ | Directorio o script completamente privado |
| `750` | rwxr-x--- | Directorio compartido con el grupo |
| `600` | rw------- | Fichero privado (claves, contraseñas) |
| `666` | rw-rw-rw- | Fichero de escritura pública (raro, poco seguro) |
| `777` | rwxrwxrwx | Acceso total para todos (PELIGROSO, evitar) |
| `000` | --------- | Sin acceso para nadie (solo root puede acceder) |

```bash
# Ejemplos prácticos:
chmod 644 mi_fichero.txt      # Fichero normal
chmod 755 mi_script.sh        # Script ejecutable
chmod 700 ~/.ssh              # Directorio SSH privado
chmod 600 ~/.ssh/id_rsa       # Clave privada SSH
chmod 750 proyecto/           # Directorio compartido con grupo
```

### 4.5 Conversión rápida de simbólico a octal

Cuando ven permisos como `rwxr-xr--` y necesitan el octal:

```
rwx = 4+2+1 = 7
r-x = 4+0+1 = 5
r-- = 4+0+0 = 4

Resultado: 754
```

Otro ejemplo: `-rw-r-----`

```
rw- = 4+2+0 = 6
r-- = 4+0+0 = 4
--- = 0+0+0 = 0

Resultado: 640
```

Practiquen con estos (anoten el resultado y verifiquen):

```
rwxrwxrwx = ???   (respuesta: 777)
rw-rw-r-- = ???   (respuesta: 664)
rwx------ = ???   (respuesta: 700)
r--r--r-- = ???   (respuesta: 444)
rwxr-x--- = ???   (respuesta: 750)
```

### 4.6 Cuándo usar cada notación

- **Notación simbólica** (`u+x`, `g-w`): cuando quieren cambiar UN permiso
  específico sin tocar los demás. Es más legible y menos propensa a errores.

- **Notación octal** (`755`, `644`): cuando quieren establecer TODOS los
  permisos de golpe. Es lo que encontrarán en la documentación de servidores
  y en los tutoriales de internet.

Ambas son igual de válidas. En la práctica, la octal es más usada en entornos
de servidores porque es más concisa.

### 4.7 Práctica de chmod

```bash
# 1. Crear un fichero de prueba
touch /tmp/prueba_chmod.txt
ls -l /tmp/prueba_chmod.txt
# Verán: -rw-r--r--  (644, permisos por defecto)

# 2. Quitar todos los permisos
chmod 000 /tmp/prueba_chmod.txt
ls -l /tmp/prueba_chmod.txt
# ----------

# 3. Intentar leerlo
cat /tmp/prueba_chmod.txt
# Permission denied

# 4. Dar lectura y escritura al dueño
chmod u+rw /tmp/prueba_chmod.txt
ls -l /tmp/prueba_chmod.txt
# -rw-------

# 5. Añadir lectura al grupo y otros
chmod go+r /tmp/prueba_chmod.txt
ls -l /tmp/prueba_chmod.txt
# -rw-r--r--

# 6. Hacerlo ejecutable para el dueño
chmod u+x /tmp/prueba_chmod.txt
ls -l /tmp/prueba_chmod.txt
# -rwxr--r--

# 7. Establecer exactamente 755
chmod 755 /tmp/prueba_chmod.txt
ls -l /tmp/prueba_chmod.txt
# -rwxr-xr-x

# 8. Volver al estado normal de fichero
chmod 644 /tmp/prueba_chmod.txt
ls -l /tmp/prueba_chmod.txt
# -rw-r--r--

# 9. Practicar con un directorio
mkdir /tmp/dir_prueba
ls -ld /tmp/dir_prueba
# drwxr-xr-x

chmod 700 /tmp/dir_prueba
ls -ld /tmp/dir_prueba
# drwx------

chmod 755 /tmp/dir_prueba
ls -ld /tmp/dir_prueba
# drwxr-xr-x
```

Nota: `ls -ld directorio` muestra los permisos DEL directorio en sí, no de
su contenido. Sin la `-d`, `ls -l` mostraría el contenido interior.

---

## PARTE V -- sudo Y root: PRIVILEGIOS ELEVADOS

### 5.1 Qué es root

En Linux, existe un usuario especial llamado **root** con UID (User ID) 0.
Este usuario es el **superusuario**: puede hacer absolutamente todo en el
sistema sin ninguna restricción de permisos.

```bash
# Ver su UID:
id
# uid=1000(alumno) gid=1000(alumno) grupos=1000(alumno),27(sudo)...

# El UID de root es siempre 0:
id root
# uid=0(root) gid=0(root) grupos=0(root)
```

Root puede:
- Leer, modificar y borrar CUALQUIER fichero del sistema
- Matar cualquier proceso
- Cambiar los permisos de cualquier fichero
- Instalar y desinstalar software
- Modificar la configuración del sistema
- Crear y eliminar usuarios

### 5.2 Por qué NO trabajar como root

Aunque root puede hacerlo todo, es MUY peligroso trabajar como root de forma
habitual:

1. **Un error puede destruir el sistema.** Si como alumno ejecutan
   `rm -rf /`, el sistema dice "permiso denegado" y no pasa nada. Si lo
   hacen como root, borran TODO el sistema operativo instantáneamente.

2. **Sin barrera de seguridad.** No hay nadie que les diga "están seguros?"
   (bueno, algunos comandos sí, pero no todos).

3. **Malware y ataques.** Si un programa malicioso se ejecuta como root,
   tiene acceso total al sistema.

La regla en el mundo profesional es clara: **trabajar siempre como usuario
normal y usar sudo solo cuando sea estrictamente necesario.**

Profundizaremos en esto en el Día 3, cuando hablemos de gestión de usuarios.

### 5.3 El comando sudo

`sudo` significa "superuser do" (hacer como superusuario). Eleva los
privilegios para UN SOLO comando y luego vuelve al nivel normal.

```bash
# Intentar leer un fichero protegido:
cat /var/log/auth.log
# cat: /var/log/auth.log: Permission denied

# Con sudo:
sudo cat /var/log/auth.log
# [pide la contraseña del alumno, no la de root]
# [muestra el contenido del fichero]
```

Detalles importantes sobre sudo:

- **Pide SU contraseña**, no la de root. sudo verifica que su
  usuario tiene permiso para usar sudo (pertenece al grupo sudo).
- **Recuerda la contraseña** durante unos minutos (normalmente 15). No se
  la pedirá otra vez en ese período.
- **Queda registrado en el log.** Cada uso de sudo se registra en
  `/var/log/auth.log`. Esto es importante para auditorías de seguridad.
- **Solo un comando.** Eleva privilegios para ese comando y luego vuelve
  a la normalidad.

```bash
# Ver quién eres antes de sudo:
whoami
# alumno

# Con sudo:
sudo whoami
# root

# Después de sudo, vuelves a ser tú:
whoami
# alumno
```

### 5.4 Usos comunes de sudo

```bash
# Leer logs del sistema:
sudo cat /var/log/auth.log | head -30
sudo tail -20 /var/log/syslog

# Editar configuración del sistema:
sudo nano /etc/hostname

# Instalar software (lo veremos en detalle más adelante):
sudo apt update
sudo apt install nombre_paquete

# Ver el directorio de root:
sudo ls /root

# Reiniciar un servicio:
sudo systemctl restart nombre_servicio
```

### 5.5 El truco de sudo !!

Uno de los atajos más útiles: cuando ejecutan un comando y falla por falta
de permisos, no necesitan reescribirlo todo. Usen `sudo !!`:

```bash
cat /var/log/auth.log
# Permission denied

sudo !!
# Equivale a: sudo cat /var/log/auth.log
# Funciona correctamente
```

`!!` se expande automáticamente al último comando ejecutado.

---

## PARTE VI -- LEER Y ESCRIBIR FICHEROS

### 6.1 cat -- Mostrar todo el contenido

`cat` (de "concatenate") muestra el contenido completo de un fichero en la
terminal.

```bash
cat /etc/hostname
# ubuntu-alumno

cat /etc/hosts
# 127.0.0.1       localhost
# 127.0.1.1       ubuntu-alumno
```

Útil para ficheros pequeños. Para ficheros grandes, la salida se desborda
y solo verán el final.

```bash
# Mostrar con números de línea:
cat -n /etc/passwd
#   1  root:x:0:0:root:/root:/bin/bash
#   2  daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
#   3  ...

# Concatenar (unir) varios ficheros:
cat fichero1.txt fichero2.txt
# Muestra el contenido de ambos, uno tras otro
```

### 6.2 head -- Ver el principio

`head` muestra las primeras líneas de un fichero (por defecto, 10).

```bash
head /etc/passwd
# Muestra las 10 primeras líneas

# Especificar cuántas líneas:
head -5 /etc/passwd
# Muestra las 5 primeras líneas

head -20 /var/log/syslog
# Las 20 primeras líneas del log del sistema
```

### 6.3 tail -- Ver el final

`tail` muestra las últimas líneas de un fichero (por defecto, 10).

```bash
tail /etc/passwd
# Muestra las 10 últimas líneas

tail -5 /etc/passwd
# Las 5 últimas líneas

# Especialmente útil para logs (lo más reciente está al final):
sudo tail -30 /var/log/syslog
```

`tail` tiene una opción muy potente para monitorizar logs en tiempo real:

```bash
# Seguir un fichero en tiempo real (no lo necesitamos hoy, pero sepan que existe):
sudo tail -f /var/log/syslog
# Muestra nuevas líneas a medida que se añaden
# Pulsar Ctrl+C para salir
```

### 6.4 less -- Lectura paginada

Para ficheros largos, `less` es la herramienta adecuada. Permite moverse
arriba y abajo por el contenido como un lector.

```bash
less /etc/passwd
```

Controles dentro de less:

| Tecla | Acción |
|---|---|
| Espacio | Avanzar una página |
| `b` | Retroceder una página |
| Flecha arriba/abajo | Mover línea a línea |
| `g` | Ir al principio del fichero |
| `G` | Ir al final del fichero |
| `/texto` | Buscar "texto" hacia adelante |
| `n` | Siguiente resultado de búsqueda |
| `N` | Resultado anterior de búsqueda |
| `q` | Salir de less |

```bash
# Ejemplo práctico:
less /etc/passwd
# Dentro de less, escriban /alumno y pulsen Enter
# Buscará la línea que contiene "alumno"
# Pulsen q para salir
```

### 6.5 wc -- Contar líneas, palabras y caracteres

`wc` (word count) cuenta líneas, palabras y caracteres de un fichero.

```bash
wc /etc/passwd
#  35   55  1832  /etc/passwd
#   |    |    |
#   |    |    └── caracteres (bytes)
#   |    └── palabras
#   └── líneas

# Solo líneas:
wc -l /etc/passwd
# 35 /etc/passwd

# Solo palabras:
wc -w /etc/passwd

# Solo caracteres:
wc -c /etc/passwd
```

Especialmente útil combinado con pipes:

```bash
# Cuántos ficheros hay en /etc:
ls /etc | wc -l

# Cuántos usuarios hay en el sistema:
wc -l /etc/passwd
```

### 6.6 nano -- Editor de texto básico

`nano` es un editor de texto sencillo que funciona en la terminal. Es mucho
más fácil que `vim` y perfecto para empezar.

```bash
# Crear o editar un fichero:
nano ~/mi_fichero.txt
```

Dentro de nano:

- Escriban texto normalmente con el teclado
- Las opciones aparecen en la parte inferior con `^` (que significa Ctrl)

| Combinación | Acción |
|---|---|
| Ctrl+O | Guardar el fichero (pide confirmación del nombre) |
| Ctrl+X | Salir de nano (pregunta si quieren guardar si hay cambios) |
| Ctrl+K | Cortar la línea actual |
| Ctrl+U | Pegar la línea cortada |
| Ctrl+W | Buscar texto |
| Ctrl+G | Mostrar la ayuda |

Flujo de trabajo básico:

```bash
# 1. Abrir nano con un fichero nuevo o existente:
nano ~/notas_dia2.txt

# 2. Escribir el texto que quieran

# 3. Guardar: Ctrl+O, luego Enter para confirmar el nombre

# 4. Salir: Ctrl+X
```

Si intentan salir sin guardar, nano les preguntará:
```
Save modified buffer? (y/n)
```
Pulsen `y` para guardar o `n` para descartar los cambios.

### 6.7 echo y redirección: escribir sin editor

Pueden crear y escribir ficheros directamente desde la línea de comandos
usando `echo` con operadores de redirección.

#### Operador `>` -- Sobrescribir

Crea el fichero si no existe. Si ya existe, BORRA su contenido y escribe
lo nuevo.

```bash
echo "Hola, soy alumno" > ~/saludo.txt
cat ~/saludo.txt
# Hola, soy alumno

# Si ejecutan otra vez, SOBRESCRIBE:
echo "Adiós" > ~/saludo.txt
cat ~/saludo.txt
# Adiós
# (el "Hola" ya no está)
```

#### Operador `>>` -- Añadir al final

Añade texto al final del fichero sin borrar lo que ya había. Si el fichero
no existe, lo crea.

```bash
echo "Línea 1" > ~/diario.txt
echo "Línea 2" >> ~/diario.txt
echo "Línea 3" >> ~/diario.txt

cat ~/diario.txt
# Línea 1
# Línea 2
# Línea 3
```

Esto es muy útil para crear ficheros de log o ir acumulando datos:

```bash
# Registrar la hora actual en un fichero de log:
date >> ~/mi_log.txt
echo "He terminado el ejercicio 1" >> ~/mi_log.txt
date >> ~/mi_log.txt
echo "He terminado el ejercicio 2" >> ~/mi_log.txt

cat ~/mi_log.txt
```

### 6.8 Leer ficheros reales del sistema

Practiquemos leyendo ficheros reales:

```bash
# 1. El nombre de la máquina
cat /etc/hostname

# 2. La tabla de hosts (resolución de nombres local)
cat /etc/hosts

# 3. Información de la distribución
cat /etc/os-release

# 4. La lista de usuarios (con números de línea)
cat -n /etc/passwd

# 5. Los repositorios de software configurados
cat /etc/apt/sources.list

# 6. La configuración de resolución DNS
cat /etc/resolv.conf

# 7. Las primeras líneas del log del sistema
sudo head -20 /var/log/syslog

# 8. Las últimas entradas de autenticación
sudo tail -10 /var/log/auth.log

# 9. Cuántas líneas tiene el fichero de passwords
wc -l /etc/passwd

# 10. Cuántos ficheros de configuración hay en /etc
ls /etc | wc -l
```

---

## PARTE VII -- EJERCICIO GUIADO: EXPLORACIÓN DEL SISTEMA

Este ejercicio les guiará paso a paso por el sistema de ficheros de su
terminal Ubuntu. Ejecuten CADA comando y observen el resultado. Al final del
ejercicio, deberían sentirse cómodos navegando por el sistema.

### Bloque A -- Exploración de la raíz

```bash
# Paso 1: Ir a la raíz
cd /
pwd
# Deben ver: /

# Paso 2: Listar el contenido
ls
# Verán todos los directorios principales: bin boot dev etc home lib...

# Paso 3: Listar con detalle
ls -l
# Observen: todos son directorios (empiezan por d)
# Observen: la mayoría pertenecen a root
```

### Bloque B -- Exploración de /etc

```bash
# Paso 4: Ver cuántos ficheros de configuración hay
ls /etc | wc -l

# Paso 5: Leer el nombre de la máquina
cat /etc/hostname

# Paso 6: Leer la tabla de hosts
cat /etc/hosts
# Observen la línea 127.0.0.1 localhost

# Paso 7: Leer la lista de usuarios
cat /etc/passwd
# Observen:
#   - root es el primer usuario (UID 0)
#   - su usuario tiene UID 1000
#   - hay muchos usuarios de sistema (servicios)

# Paso 8: Contar cuántos usuarios hay
wc -l /etc/passwd

# Paso 9: Ver solo su usuario
grep alumno /etc/passwd

# Paso 10: Información de la distribución
cat /etc/os-release
```

### Bloque C -- Exploración de /var/log

```bash
# Paso 11: Ver qué logs existen
ls /var/log

# Paso 12: Ver los últimos mensajes del sistema
sudo tail -15 /var/log/syslog

# Paso 13: Ver las últimas autenticaciones
sudo tail -10 /var/log/auth.log

# Paso 14: Ver cuántas líneas tiene el syslog
sudo wc -l /var/log/syslog

# Paso 15: Ver el historial de paquetes instalados
tail -20 /var/log/dpkg.log
```

### Bloque D -- Exploración de /tmp y /proc

```bash
# Paso 16: Ver qué hay en /tmp
ls /tmp
# Contenido variable: puede haber ficheros temporales o estar vacío

# Paso 17: Crear un fichero temporal
echo "Esto se borrará al reiniciar" > /tmp/temporal.txt
cat /tmp/temporal.txt

# Paso 18: Ver información del procesador
head -20 /proc/cpuinfo

# Paso 19: Ver información de la memoria
head -5 /proc/meminfo

# Paso 20: Ver la versión del kernel
cat /proc/version
```

### Bloque E -- Trabajar con permisos

```bash
# Paso 21: Crear ficheros de prueba en casa
cd ~
touch publico.txt privado.txt ejecutable.sh

# Paso 22: Ver permisos por defecto
ls -l publico.txt privado.txt ejecutable.sh

# Paso 23: Hacer privado.txt completamente privado
chmod 600 privado.txt
ls -l privado.txt
# -rw-------

# Paso 24: Hacer ejecutable.sh ejecutable
chmod 755 ejecutable.sh
ls -l ejecutable.sh
# -rwxr-xr-x

# Paso 25: Quitar todos los permisos a publico.txt
chmod 000 publico.txt
ls -l publico.txt
# ----------

# Paso 26: Intentar leerlo
cat publico.txt
# Permission denied

# Paso 27: Restaurar permisos
chmod 644 publico.txt
cat publico.txt
# (vacío, porque no hemos escrito nada)
```

### Bloque F -- Escribir y leer con herramientas del día

```bash
# Paso 28: Escribir con echo y redirección
echo "Día 4 del curso IFCD0112" > ~/resumen_dia4.txt
echo "Hoy he aprendido:" >> ~/resumen_dia2.txt
echo "- El árbol de ficheros de Linux" >> ~/resumen_dia2.txt
echo "- Rutas absolutas y relativas" >> ~/resumen_dia2.txt
echo "- Permisos rwx" >> ~/resumen_dia2.txt
echo "- chmod simbólico y octal" >> ~/resumen_dia2.txt
echo "- sudo y root" >> ~/resumen_dia2.txt
echo "- cat, head, tail, less, wc, nano" >> ~/resumen_dia2.txt

# Paso 29: Leer lo que hemos escrito
cat ~/resumen_dia2.txt

# Paso 30: Contar las líneas
wc -l ~/resumen_dia2.txt

# Paso 31: Ver solo las 3 primeras líneas
head -3 ~/resumen_dia2.txt

# Paso 32: Ver solo las 2 últimas líneas
tail -2 ~/resumen_dia2.txt

# Paso 33: Editar con nano (añadir algo más)
nano ~/resumen_dia2.txt
# Añadir una línea nueva al final
# Guardar con Ctrl+O, Enter
# Salir con Ctrl+X

# Paso 34: Verificar el cambio
cat ~/resumen_dia2.txt
```

### Bloque G -- Combinación de todo lo aprendido

```bash
# Paso 35: Crear un directorio de trabajo para la tarea
mkdir -p ~/tareas/dia2
cd ~/tareas/dia2
pwd

# Paso 36: Crear un script sencillo
echo '#!/bin/bash' > info_sistema.sh
echo 'echo "=== Información del sistema ==="' >> info_sistema.sh
echo 'echo "Hostname: $(hostname)"' >> info_sistema.sh
echo 'echo "Usuario: $(whoami)"' >> info_sistema.sh
echo 'echo "Fecha: $(date)"' >> info_sistema.sh
echo 'echo "Directorio: $(pwd)"' >> info_sistema.sh

# Paso 37: Verificar el contenido
cat info_sistema.sh

# Paso 38: Intentar ejecutarlo (sin permisos de ejecución)
./info_sistema.sh
# Permission denied

# Paso 39: Dar permiso de ejecución
chmod u+x info_sistema.sh
ls -l info_sistema.sh

# Paso 40: Ejecutarlo
./info_sistema.sh
# Deben ver la información del sistema
```

---

## TAREA 2 -- Entrega obligatoria (Classroom)

**Fecha límite:** Lunes 13 de Abril a las 09:00

La tarea tiene **5 entregas**. Cada una debe incluir una captura de pantalla
de la terminal mostrando los comandos ejecutados y su salida.

### Entrega 2.1 -- Exploración del sistema de ficheros

Ejecuten los siguientes comandos y capturen la salida:

```bash
ls /
cat /etc/hostname
cat /etc/os-release
head -5 /etc/passwd
ls /var/log
wc -l /etc/passwd
```

### Entrega 2.2 -- Navegación con rutas absolutas y relativas

Ejecuten esta secuencia completa:

```bash
cd /
pwd
cd /var/log
pwd
cd ..
pwd
cd ~
pwd
cd /etc
pwd
cd -
pwd
```

### Entrega 2.3 -- Diagnóstico del sistema

Ejecuten estos 10 comandos y capturen la salida completa:

```bash
uname -a
whoami
id
pwd
ls -la ~
cat /etc/hostname
echo $PATH
which python3
df -h
free -h
```

### Entrega 2.4 -- Permisos: interpretación y modificación

Ejecuten esta secuencia y capturen la salida:

```bash
cd ~
touch tarea2_permisos.txt
ls -l tarea2_permisos.txt
chmod 000 tarea2_permisos.txt
ls -l tarea2_permisos.txt
chmod 644 tarea2_permisos.txt
ls -l tarea2_permisos.txt
chmod u+x tarea2_permisos.txt
ls -l tarea2_permisos.txt
chmod 755 tarea2_permisos.txt
ls -l tarea2_permisos.txt
chmod 600 tarea2_permisos.txt
ls -l tarea2_permisos.txt
```

### Entrega 2.5 -- Lectura y escritura de ficheros

Ejecuten esta secuencia y capturen la salida:

```bash
echo "Tarea 2 - IFCD0112" > ~/tarea2_fichero.txt
echo "Nombre: [su nombre]" >> ~/tarea2_fichero.txt
echo "Fecha: $(date)" >> ~/tarea2_fichero.txt
echo "Sistema: $(uname -a)" >> ~/tarea2_fichero.txt
cat ~/tarea2_fichero.txt
wc -l ~/tarea2_fichero.txt
head -2 ~/tarea2_fichero.txt
tail -2 ~/tarea2_fichero.txt
ls -l ~/tarea2_fichero.txt
```

---

## Referencia rápida -- Tabla completa del Día 4

### Sistema de ficheros

| Directorio | Contenido |
|---|---|
| `/` | Raíz del sistema |
| `/bin` | Binarios esenciales (ls, cp, mv, cat...) |
| `/sbin` | Binarios de administración (fdisk, reboot...) |
| `/boot` | Kernel y ficheros de arranque |
| `/dev` | Dispositivos como ficheros |
| `/etc` | Configuración del sistema |
| `/home` | Directorios personales de usuarios |
| `/lib` | Librerías del sistema |
| `/media` | Montaje automático de medios extraíbles |
| `/mnt` | Montaje manual de discos |
| `/opt` | Software de terceros |
| `/proc` | Sistema virtual de procesos |
| `/root` | Home del superusuario |
| `/sys` | Sistema virtual de hardware |
| `/tmp` | Ficheros temporales |
| `/usr` | Programas instalados |
| `/usr/local` | Software instalado manualmente |
| `/var` | Datos variables (logs, cachés) |
| `/var/log` | Logs del sistema |

### Navegación

| Comando / Atajo | Función |
|---|---|
| `pwd` | Directorio actual |
| `cd /ruta` | Ir a ruta absoluta |
| `cd directorio` | Ir a ruta relativa |
| `cd ~` | Ir a tu home |
| `cd ..` | Subir un nivel |
| `cd -` | Volver al directorio anterior |
| Tab | Autocompletar |
| Tab Tab | Ver opciones de autocompletado |
| Flecha arriba | Comando anterior |
| `history` | Ver historial de comandos |
| Ctrl+R | Buscar en el historial |
| `!!` | Repetir último comando |
| `sudo !!` | Repetir último comando con sudo |

### Permisos

| Permiso | Valor octal | Sobre fichero | Sobre directorio |
|---|---|---|---|
| `r` | 4 | Leer contenido | Ver listado con ls |
| `w` | 2 | Modificar contenido | Crear/borrar dentro |
| `x` | 1 | Ejecutar como programa | Entrar con cd |
| `-` | 0 | Sin permiso | Sin permiso |

### Patrones de permisos

| Octal | Simbólico | Uso típico |
|---|---|---|
| 644 | rw-r--r-- | Fichero normal |
| 755 | rwxr-xr-x | Script ejecutable / directorio |
| 700 | rwx------ | Privado total |
| 750 | rwxr-x--- | Compartido con grupo |
| 600 | rw------- | Fichero privado (claves) |

### Comandos de chmod

| Comando | Efecto |
|---|---|
| `chmod u+x fichero` | Dar ejecución al dueño |
| `chmod g-w fichero` | Quitar escritura al grupo |
| `chmod o-rwx fichero` | Quitar todo a otros |
| `chmod a+r fichero` | Dar lectura a todos |
| `chmod 644 fichero` | Permisos estándar de fichero |
| `chmod 755 fichero` | Permisos estándar de ejecutable |
| `chmod 700 directorio` | Directorio privado |

### Lectura y escritura de ficheros

| Comando | Función |
|---|---|
| `cat fichero` | Mostrar contenido completo |
| `cat -n fichero` | Mostrar con números de línea |
| `head -N fichero` | Mostrar las primeras N líneas |
| `tail -N fichero` | Mostrar las últimas N líneas |
| `less fichero` | Lectura paginada (q para salir) |
| `wc -l fichero` | Contar líneas |
| `wc -w fichero` | Contar palabras |
| `nano fichero` | Editar con nano |
| `echo "texto" > fichero` | Escribir (sobrescribe) |
| `echo "texto" >> fichero` | Escribir (añade al final) |

### Privilegios

| Comando | Función |
|---|---|
| `whoami` | Tu nombre de usuario |
| `id` | Tu UID, GID y grupos |
| `sudo comando` | Ejecutar con privilegios de root |
| `sudo !!` | Repetir último comando con sudo |

---

## Resumen visual del día

```
ÁRBOL DE FICHEROS                    PERMISOS
=================                    ========

     /  (raíz)                       d rwx r-x r-x
    / \                              | |   |   |
   /   \                             | |   |   └ others
  /     \                            | |   └ group
etc  home  var  ...                  | └ owner
 |    |     |                        └ tipo (d/-/l)
 |  alumno  log
 |
hostname                             OCTAL
passwd                               =====
hosts                                4=r 2=w 1=x
                                     rwx = 7
                                     rw- = 6
NAVEGACIÓN                           r-x = 5
==========                           r-- = 4

Absoluta: /home/alumno/fichero       644 = rw-r--r--
Relativa: Documentos/fichero         755 = rwxr-xr-x
~  = home                            700 = rwx------
.  = aquí
.. = arriba                          LECTURA/ESCRITURA
-  = anterior                        ==================
Tab = autocompletar                  cat   = ver todo
                                     head  = ver principio
                                     tail  = ver final
PRIVILEGIOS                          less  = paginar
===========                          wc    = contar
                                     nano  = editar
root = UID 0 = todopoderoso         >     = sobrescribir
sudo = elevar 1 comando              >>    = añadir
```
