# Día 2: Instalación del entorno — WSL, Ubuntu y tu primera terminal

Ayer conocieron el mapa del curso: qué vamos a aprender, con qué herramientas
y por qué esas y no otras. Hoy toca construir el taller donde van a trabajar
los próximos 6 meses. Sin entorno, no hay curso.

Prof. Juan Marcelo Gutiérrez Miranda

**Curso IFCD0112 — Semana 1, Día 2 (Viernes)**
**Objetivo:** Activar WSL en Windows, instalar Ubuntu, abrir la terminal de
Linux y ejecutar los primeros comandos reales. Al terminar el día, cada alumno
tiene Linux funcionando dentro de su Windows.

> Este manual es de consulta. Sigan los pasos con el equipo encendido.
> Cada paso tiene una verificación: si la salida no coincide, levanten la mano.

---

## PASO 0 — De dónde venimos

```
╔══════════════════════════════════════════════════════════════════╗
║              EVOLUCIÓN DE AMTIGRAVITY LAUNCH CONTROL             ║
╠══════════════════════════════════════════════════════════════════╣
║                                                                  ║
║  Día 1   BOE + Propuesta     Conocemos el mapa del curso         ║
║  ► DÍA 2  WSL + Ubuntu       Construimos el taller de trabajo    ║
║                                                                  ║
╚══════════════════════════════════════════════════════════════════╝
```

| Aspecto | Ayer (Día 1) | Hoy (Día 2) |
|---------|-------------|-------------|
| Enfoque | Teoría: qué vamos a hacer | Práctica: montar el entorno |
| Resultado | Saben qué herramientas usaremos | Tienen Linux funcionando en su equipo |
| Proyecto | Conocen Launch Control como concepto | Launch Control tiene un servidor donde vivir |

El proyecto no cambia. Cambia que ahora tiene dónde ejecutarse.

---

## Índice del día

| Parte | Contenido |
|---|---|
| I   | Qué es Linux y por qué lo usamos |
| II  | Qué es WSL — Linux dentro de Windows |
| III | Instalar WSL y Ubuntu paso a paso |
| IV  | Primer contacto con la terminal |
| V   | Los comandos fundamentales: pwd, ls, cd |

---

# PARTE I — QUÉ ES LINUX Y POR QUÉ LO USAMOS

---

## 1. Linux no es una curiosidad — es el estándar

No usamos Linux por capricho. Estos son los datos:

| Métrica | Linux | Windows | Otros |
|---|---|---|---|
| Servidores web (W3Techs 2025) | 96.3% | 3.7% | <0.1% |
| Top 500 supercomputadores | 100% | 0% | 0% |
| Infraestructura cloud (AWS) | ~92% | ~8% | -- |
| Dispositivos Android | 100% (kernel Linux) | -- | -- |
| Contenedores Docker | ~99% | ~1% | -- |

Cuando trabajen como programadores, su código se ejecutará en servidores
Linux. Da igual que desarrollen en Windows o Mac: el destino final es Linux.

Las razones técnicas:

1. **Estabilidad:** un servidor Linux puede funcionar años sin reiniciarse.
2. **Coste:** es software libre. No hay licencias por servidor.
3. **Seguridad:** modelo de permisos robusto, actualizaciones rápidas.
4. **Automatización:** todo se puede controlar desde la terminal y scripts.
5. **Comunidad:** millones de desarrolladores contribuyen al ecosistema.

---

## 2. Qué es Ubuntu

Linux en sí mismo es solo un núcleo (kernel). Para ser usable necesita
programas, herramientas, gestor de paquetes. Una **distribución** empaqueta
todo eso junto.

| Distribución | Uso principal | Nivel |
|---|---|---|
| **Ubuntu** | Escritorio y servidores generalistas | Principiante-Intermedio |
| Debian | Servidores estables | Intermedio |
| CentOS / Rocky | Servidores empresariales | Intermedio-Avanzado |
| Arch | Personalización extrema | Avanzado |
| Alpine | Contenedores Docker (minimalista) | Intermedio |

Usamos **Ubuntu** porque:
- Es la distribución con más documentación y comunidad.
- Lo que aprendan aquí se traduce directamente a cualquier otra distribución.
- Es la que viene integrada en WSL de Microsoft.

---

## 3. La terminal — su herramienta principal

En Linux, la **terminal** (también llamada shell, consola o línea de comandos)
es la interfaz principal para interactuar con el sistema. No es un programa
más — es LA forma de trabajar.

```
╔══════════════════════════════════════════════════════════════╗
║  TERMINAL vs INTERFAZ GRÁFICA                                ║
╠══════════════════════════════════════════════════════════════╣
║                                                              ║
║  Crear 1 carpeta:                                            ║
║    Gráfico: Click derecho > Nueva carpeta > Escribir nombre  ║
║    Terminal: mkdir proyecto                                   ║
║                                                              ║
║  Crear 50 carpetas:                                          ║
║    Gráfico: 50 clicks, 50 nombres a mano                    ║
║    Terminal: mkdir modulo_{01..50}                            ║
║                                                              ║
║  Crear 50 carpetas en un servidor remoto:                    ║
║    Gráfico: IMPOSIBLE (los servidores no tienen pantalla)    ║
║    Terminal: ssh servidor && mkdir modulo_{01..50}            ║
║                                                              ║
╚══════════════════════════════════════════════════════════════╝
```

Por qué la terminal y no el escritorio gráfico:

1. **Los servidores no tienen pantalla.** Cuando se conectan a un servidor
   en producción, solo tienen una terminal.
2. **La terminal es más rápida.** Un comando hace lo que con el ratón son
   decenas de clicks.
3. **La terminal es automatizable.** Pueden escribir scripts que ejecuten
   cientos de comandos automáticamente.
4. **La terminal es universal.** Funciona igual en Ubuntu, CentOS, Debian,
   macOS y en cualquier servidor cloud.
5. **La terminal es reproducible.** Pueden copiar y pegar un comando y
   obtener siempre el mismo resultado.

---

# PARTE II — QUÉ ES WSL: LINUX DENTRO DE WINDOWS

---

## 4. WSL — Windows Subsystem for Linux

WSL es una funcionalidad de Windows que permite ejecutar un sistema Linux
**real** directamente dentro de Windows, sin necesidad de una máquina virtual
tradicional.

```
╔══════════════════════════════════════════════════════════════╗
║                    SIN WSL (máquina virtual)                 ║
╠══════════════════════════════════════════════════════════════╣
║                                                              ║
║  Windows 11                                                  ║
║    └── VirtualBox                                            ║
║          └── Ubuntu (máquina virtual completa)               ║
║                └── Terminal                                   ║
║                                                              ║
║  Requiere: descargar ISO de 4.7 GB, crear VM, instalar SO   ║
║  RAM: necesita 4 GB dedicados                                ║
║  Disco: ocupa 10-40 GB                                       ║
║  Arranque: 1-2 minutos                                       ║
║                                                              ║
╠══════════════════════════════════════════════════════════════╣
║                    CON WSL (lo que usamos)                    ║
╠══════════════════════════════════════════════════════════════╣
║                                                              ║
║  Windows 11                                                  ║
║    └── WSL2 (integrado en Windows)                           ║
║          └── Ubuntu (distribución ligera)                     ║
║                └── Terminal                                   ║
║                                                              ║
║  Requiere: un comando de instalación (~400 MB)               ║
║  RAM: comparte con Windows, sin reservar                     ║
║  Disco: ocupa ~1-2 GB                                        ║
║  Arranque: instantáneo (< 2 segundos)                        ║
║                                                              ║
╚══════════════════════════════════════════════════════════════╝
```

### 4.1 Qué es WSL2

WSL tiene dos versiones:
- **WSL1:** traducía las llamadas de Linux a Windows. Rápido pero incompleto.
- **WSL2:** ejecuta un kernel Linux real dentro de una máquina virtual
  ultraligera gestionada por Windows. Es Linux de verdad.

Nosotros usamos **WSL2**. Es la versión por defecto en Windows 11.

### 4.2 Qué pueden hacer con WSL

| Funcionalidad | ¿Funciona en WSL? |
|---|---|
| Terminal Bash completa | Sí |
| Comandos Linux (ls, cd, grep, chmod...) | Sí |
| Instalar programas con apt | Sí |
| Git | Sí |
| Python | Sí |
| PostgreSQL | Sí |
| Docker (con Docker Desktop) | Sí |
| Acceder a ficheros de Windows desde Linux | Sí (`/mnt/c/`) |
| Acceder a ficheros de Linux desde Windows | Sí (desde el Explorador) |
| Escritorio gráfico Linux | No (no lo necesitamos) |

### 4.3 Por qué WSL y no una máquina virtual completa

Para lo que hacemos las primeras semanas (terminal, ficheros, permisos, Git,
scripts), WSL es **idéntico** a un Ubuntu completo. La diferencia es que se
instala en 10 minutos en vez de 2 horas, y no satura la red del aula.

Más adelante en el curso, cuando necesitemos entornos aislados completos
(Docker avanzado, redes entre máquinas), instalaremos VirtualBox. Pero para
empezar, WSL es la opción inteligente.

---

## 5. Cómo se integran Windows y WSL

WSL no es un programa separado que abren y cierran. Es una capa que vive
dentro de Windows:

```
╔══════════════════════════════════════════════════════════════╗
║  WINDOWS 11                                                  ║
║  ├── C:\Users\alumno\          (ficheros Windows)            ║
║  ├── C:\Program Files\         (programas Windows)           ║
║  │                                                           ║
║  └── WSL2                                                    ║
║       └── Ubuntu                                             ║
║            ├── /home/alumno/   (ficheros Linux)              ║
║            ├── /etc/           (configuración Linux)         ║
║            ├── /mnt/c/         (acceso a C:\ de Windows)     ║
║            └── /mnt/d/         (acceso a D:\ si existe)      ║
║                                                              ║
║  Pueden acceder a ficheros de un lado desde el otro:         ║
║  - Desde Linux: /mnt/c/Users/alumno/Documentos               ║
║  - Desde Windows: \\wsl$\Ubuntu\home\alumno                  ║
╚══════════════════════════════════════════════════════════════╝
```

Esto significa que pueden:
- Editar un fichero en Windows y ejecutarlo en Linux
- Crear algo en Linux y abrirlo desde el Explorador de Windows
- Usar el mejor de ambos mundos

---

# PARTE III — INSTALAR WSL Y UBUNTU PASO A PASO

---

## 6. Antes de empezar: abrir PowerShell como administrador

Para instalar WSL necesitamos permisos de administrador.

```
Paso 1:  Pulsar la tecla Windows (o click en el menú Inicio)
Paso 2:  Escribir "PowerShell"
Paso 3:  En el resultado, click derecho > "Ejecutar como administrador"
         (o click en "Ejecutar como administrador" a la derecha)
Paso 4:  Si Windows pregunta "Permitir que esta app haga cambios":
         pulsar "Sí"
```

**Verificación:** se abre una ventana azul/negra con el texto:

```
Windows PowerShell
Copyright (C) Microsoft Corporation. All rights reserved.

PS C:\Windows\System32>
```

El `PS` al principio indica que están en PowerShell (no en cmd ni en Bash).

---

## 7. Verificar la versión de Windows

```powershell
winver
```

Se abre una ventana con la información de versión. Necesitan:
- **Windows 10** versión 2004 o superior, o
- **Windows 11** cualquier versión

Si tienen Windows 11, están bien. Cierren la ventana de winver.

### 7.1 Verificar si WSL ya está disponible

```powershell
wsl --status
```

Posibles resultados:

| Salida | Significado |
|---|---|
| Muestra versión de WSL, kernel, etc. | WSL ya está instalado. Salten al paso 9 |
| "WSL no está instalado" o error | Normal — vamos a instalarlo |
| "No se reconoce wsl" | Windows muy antiguo. Avisar al formador |

---

## 8. Instalar WSL con un solo comando

Este es el comando más importante del día:

```powershell
wsl --install
```

```
╔══════════════════════════════════════════════════════════════╗
║  Qué hace este comando:                                      ║
║                                                              ║
║  1. Habilita la funcionalidad WSL en Windows                 ║
║  2. Habilita la "Plataforma de máquina virtual"              ║
║  3. Descarga el kernel de Linux para WSL2                    ║
║  4. Descarga Ubuntu (la distribución por defecto)            ║
║  5. Configura todo automáticamente                           ║
║                                                              ║
║  Descarga total: ~400 MB (vs 4.7 GB de una ISO completa)    ║
║  Tiempo: 5-15 minutos según la velocidad de internet         ║
╚══════════════════════════════════════════════════════════════╝
```

Verán algo como:

```
Instalando: Plataforma de máquina virtual
Instalando: Subsistema de Windows para Linux
Descargando: Kernel de WSL
Instalando: Kernel de WSL
Descargando: Ubuntu
Instalando: Ubuntu
La operación solicitada se ha completado. Es necesario reiniciar el sistema.
```

### 8.1 Reiniciar el equipo

**Este paso es OBLIGATORIO.** WSL no funciona sin reiniciar.

```powershell
shutdown /r /t 0
```

O simplemente: Menú Inicio > Reiniciar.

**Esperen** a que Windows arranque completamente.

### 8.2 Qué hacer mientras se descarga / reinicia

La descarga tarda entre 5 y 15 minutos. Mientras esperan:

1. **Lean la Parte I y II** de este manual si no lo han hecho
2. **Anoten** cualquier duda que tengan sobre Linux o la terminal
3. **Si ya terminó la descarga:** reinicien y sigan con el paso siguiente
4. **Si la descarga es muy lenta:** el formador tiene Ubuntu en un medio
   extraíble. Levanten la mano.

### 8.3 Alternativa si la red está saturada

Si con 18 alumnos descargando la red no da, el formador tiene la opción
de instalar Ubuntu desde un archivo local:

```powershell
# El formador les indicará la ruta del archivo
wsl --install -d Ubuntu --web-download
```

O bien importar una imagen pre-descargada:

```powershell
# Solo si el formador se los indica
wsl --import Ubuntu C:\WSL\Ubuntu \\ruta\al\pendrive\ubuntu.tar
```

---

## 9. Primer arranque de Ubuntu

Después del reinicio, pueden pasar dos cosas:

**Opción A:** Se abre automáticamente una ventana de Ubuntu pidiendo crear
un usuario. Vayan al paso 9.1.

**Opción B:** No se abre nada. Ábranlo manualmente:

```
Paso 1:  Pulsar la tecla Windows
Paso 2:  Escribir "Ubuntu"
Paso 3:  Click en "Ubuntu" (icono naranja)
```

### 9.1 Crear su usuario de Linux

La primera vez que abren Ubuntu, les pide crear un usuario:

```
Enter new UNIX username:
```

Escriban: `alumno` y pulsen Enter.

```
New password:
```

Escriban: `alumno2026` y pulsen Enter.

**ATENCIÓN:** al escribir la contraseña **no se ve nada en pantalla**. Ni
asteriscos, ni puntos, ni nada. Es una medida de seguridad de Linux. Escriban
la contraseña y pulsen Enter aunque no vean nada.

```
Retype new password:
```

Escriban otra vez: `alumno2026` y pulsen Enter.

```
passwd: password updated successfully
Installation successful!
```

**Verificación:** ven el prompt de Linux:

```
alumno@NOMBRE-PC:~$
```

Están dentro de Linux. La terminal está lista.

### 9.2 Verificar que es WSL2

```bash
wsl.exe -l -v
```

```
# salida esperada:
  NAME      STATE           VERSION
* Ubuntu    Running         2
```

La columna VERSION debe decir `2`. Si dice `1`, ejecuten desde PowerShell
(no desde Ubuntu):

```powershell
wsl --set-version Ubuntu 2
```

---

## 10. Actualizar Ubuntu

Lo primero que se hace en cualquier instalación Linux fresca es actualizar
los paquetes:

```bash
sudo apt update && sudo apt upgrade -y
```

Les pedirá la contraseña (`alumno2026`). Recuerden: no se ve al escribir.

```
╔══════════════════════════════════════════════════════════════╗
║  Qué hace este comando:                                      ║
║                                                              ║
║  sudo          = ejecutar como administrador                 ║
║  apt update    = actualizar la lista de paquetes disponibles ║
║  &&            = si lo anterior funciona, ejecutar lo que    ║
║                  viene después                               ║
║  apt upgrade   = instalar las actualizaciones pendientes     ║
║  -y            = responder "sí" a todo automáticamente       ║
╚══════════════════════════════════════════════════════════════╝
```

Esto puede tardar 2-5 minutos. Verán muchas líneas de texto desplazándose.
Es normal — está descargando e instalando actualizaciones.

---

## 11. Instalar herramientas básicas

Instalamos algunas herramientas que usaremos durante el curso:

```bash
sudo apt install -y tree nano curl wget git
```

| Herramienta | Para qué sirve |
|---|---|
| `tree` | Visualizar estructura de directorios en árbol |
| `nano` | Editor de texto simple en terminal |
| `curl` | Descargar ficheros y hacer peticiones HTTP |
| `wget` | Descargar ficheros de internet |
| `git` | Control de versiones (lo usaremos mucho) |

**Verificación:**

```bash
tree --version
```

```
# salida esperada:
tree v2.x.x (algún número de versión)
```

```bash
git --version
```

```
# salida esperada:
git version 2.xx.x
```

Si ambos comandos muestran una versión, todo está instalado correctamente.

---

## 12. Cómo abrir y cerrar Ubuntu/WSL

### 12.1 Abrir (3 formas)

**Forma 1 — Desde el menú Inicio:**
```
Tecla Windows > escribir "Ubuntu" > click
```

**Forma 2 — Desde PowerShell o cmd:**
```
wsl
```

**Forma 3 — Desde Windows Terminal (recomendada):**
Si tienen Windows Terminal instalado (viene con Windows 11), aparece Ubuntu
como una opción en la pestaña desplegable (flecha hacia abajo junto al +).

### 12.2 Cerrar

Simplemente escriban:

```bash
exit
```

O cierren la ventana. No hay proceso de apagado — no es una máquina virtual
tradicional.

### 12.3 Verificar si WSL está corriendo

Desde PowerShell:

```powershell
wsl -l -v
```

Muestra el estado (Running / Stopped) de cada distribución instalada.

### 12.4 Apagar WSL completamente

```powershell
wsl --shutdown
```

Libera la RAM y el kernel. La próxima vez que abran Ubuntu, arranca de nuevo
(en 1-2 segundos).

---

## 13. Acceso cruzado entre Windows y Linux

### 13.1 Acceder a ficheros de Windows desde Linux

Su disco C: está montado en `/mnt/c/`:

```bash
ls /mnt/c/Users/
```

```
# salida:
alumno  Default  Public  ...
```

```bash
ls /mnt/c/Users/alumno/Documents/
```

Ven sus documentos de Windows desde Linux.

### 13.2 Acceder a ficheros de Linux desde Windows

Abran el Explorador de archivos de Windows y escriban en la barra de
direcciones:

```
\\wsl$\Ubuntu\home\alumno
```

Ven su home de Linux como si fuera una carpeta de red.

### 13.3 Regla importante sobre dónde guardar ficheros

```
╔══════════════════════════════════════════════════════════════╗
║  REGLA: trabajen siempre dentro de /home/alumno              ║
║                                                              ║
║  NO guarden archivos de trabajo en /mnt/c/                   ║
║  (funciona pero es MUCHO más lento porque cruza              ║
║   el sistema de archivos de Windows)                         ║
║                                                              ║
║  BUENO:  ~/launch-control/                                   ║
║  MALO:   /mnt/c/Users/alumno/launch-control/                 ║
╚══════════════════════════════════════════════════════════════╝
```

---

# PARTE IV — PRIMER CONTACTO CON LA TERMINAL

---

## 14. Abrir la terminal

Si no la tienen abierta ya, abran Ubuntu desde el menú Inicio.

Verán el prompt:

```
alumno@NOMBRE-PC:~$
```

### 14.1 Anatomía del prompt

Cada parte tiene un significado:

```
alumno    @    NOMBRE-PC    :    ~         $
  |       |       |         |    |         |
  |       |       |         |    |         +-- Usuario normal
  |       |       |         |    |             (# = root/admin)
  |       |       |         |    |
  |       |       |         |    +-- Directorio actual: ~ = /home/alumno
  |       |       |         |
  |       |       |         +-- Separador
  |       |       |
  |       |       +-- Nombre de la máquina
  |       |
  |       +-- Separador "en" (at)
  |
  +-- Nombre de usuario
```

El prompt siempre les dice **quién** son, **dónde** están y **en qué máquina**.

En WSL, el nombre de la máquina es el nombre de su equipo de Windows. Es
normal y correcto.

---

## 15. Primer comando: whoami

Antes de hacer nada, verifiquen quién son:

```bash
whoami
```

```
# salida:
alumno
```

Y dónde están:

```bash
hostname
```

```
# salida:
NOMBRE-PC
```

(El nombre será el de su equipo Windows. Cada uno tendrá uno diferente.)

---

# PARTE V — LOS COMANDOS FUNDAMENTALES: PWD, LS, CD

---

## 16. pwd — Print Working Directory

Muestra la ruta completa del directorio en el que están.

```bash
pwd
```

```
# salida:
/home/alumno
```

Úsenlo siempre que no tengan claro dónde están. No cuesta nada y evita
errores.

---

## 17. ls — List

Lista el contenido del directorio actual.

```bash
ls
```

En una instalación fresca de Ubuntu en WSL, puede que no haya nada visible.
Eso es normal — el home empieza vacío.

Con la opción `-la` muestra **todo**, incluidos archivos ocultos y detalles:

```bash
ls -la
```

```
# salida:
total 24
drwxr-xr-x 3 alumno alumno 4096 abr 10 09:30 .
drwxr-xr-x 3 root   root   4096 abr 10 09:15 ..
-rw-r--r-- 1 alumno alumno  220 abr 10 09:15 .bash_logout
-rw-r--r-- 1 alumno alumno 3771 abr 10 09:15 .bashrc
-rw-r--r-- 1 alumno alumno  807 abr 10 09:15 .profile
```

Los archivos que empiezan por `.` (punto) son **ocultos**. Son archivos de
configuración que Linux crea automáticamente.

| Archivo | Para qué sirve |
|---|---|
| `.bashrc` | Configuración de su terminal (colores, atajos, etc.) |
| `.profile` | Se ejecuta al iniciar sesión |
| `.bash_logout` | Se ejecuta al cerrar sesión |

### 17.1 Opciones útiles de ls

| Opción | Qué hace | Ejemplo |
|---|---|---|
| `ls` | Lista básica | `ls` |
| `ls -l` | Lista con detalles (permisos, tamaño, fecha) | `ls -l` |
| `ls -a` | Muestra archivos ocultos | `ls -a` |
| `ls -la` | Detalles + ocultos | `ls -la` |
| `ls -lh` | Detalles con tamaños legibles (KB, MB) | `ls -lh` |
| `ls -R` | Lista recursiva (subdirectorios) | `ls -R` |
| `ls /ruta` | Lista otro directorio | `ls /etc` |

---

## 18. cd — Change Directory

Cambia el directorio actual.

```bash
cd /etc
pwd
```

```
# salida:
/etc
```

```bash
cd ~
pwd
```

```
# salida:
/home/alumno
```

### 18.1 Movimientos básicos

| Comando | A dónde va | Ejemplo |
|---|---|---|
| `cd directorio` | Dentro de un subdirectorio | `cd Documentos` |
| `cd ..` | Un nivel arriba (directorio padre) | `cd ..` |
| `cd ~` o `cd` | Al home del usuario | `cd ~` |
| `cd /` | A la raíz del sistema | `cd /` |
| `cd -` | Al directorio anterior | `cd -` |
| `cd /ruta/completa` | A cualquier sitio (ruta absoluta) | `cd /var/log` |

---

## 19. Rutas absolutas vs rutas relativas

### 19.1 Ruta absoluta

Empieza con `/` (la raíz del sistema). Indica la ubicación completa.

```bash
cd /home/alumno
```

Funciona **desde cualquier sitio**. No importa dónde estén actualmente.

### 19.2 Ruta relativa

NO empieza con `/`. Indica la ubicación respecto al directorio actual.

```bash
cd Documentos
```

Solo funciona si `Documentos` existe dentro del directorio donde están.

### 19.3 Elementos especiales en rutas

| Símbolo | Significado | Ejemplo |
|---|---|---|
| `/` | Raíz del sistema | `cd /` |
| `~` | Home del usuario actual | `cd ~` equivale a `cd /home/alumno` |
| `.` | Directorio actual | `ls .` equivale a `ls` |
| `..` | Directorio padre (un nivel arriba) | `cd ..` |
| `-` | Directorio anterior (donde estaban antes) | `cd -` |

---

## 20. Explorar el sistema de archivos

Ejecuten esta secuencia y observen las salidas:

```bash
cd /
ls
```

```
# salida:
bin  boot  dev  etc  home  lib  lib64  media  mnt  opt  proc  root  run
sbin  snap  srv  sys  tmp  usr  var
```

Están en la raíz del sistema de archivos de Linux. Cada carpeta tiene un
propósito:

| Directorio | Contenido |
|---|---|
| `/home` | Carpetas personales de los usuarios |
| `/etc` | Archivos de configuración del sistema |
| `/var` | Datos variables (logs, bases de datos, cachés) |
| `/tmp` | Archivos temporales (se borran al reiniciar) |
| `/usr` | Programas instalados por el sistema |
| `/bin` | Comandos esenciales del sistema |
| `/root` | Home del usuario administrador (root) |
| `/mnt` | Puntos de montaje (aquí está `/mnt/c/` con su Windows) |

```bash
cd ~
pwd
```

```
# salida:
/home/alumno
```

Han vuelto a casa.

---

## 21. Crear algo — su primer directorio

Vamos a crear un directorio de prácticas:

```bash
mkdir practicas
ls
```

```
# salida:
practicas
```

```bash
cd practicas
pwd
```

```
# salida:
/home/alumno/practicas
```

Y ahora un archivo:

```bash
echo "Mi primer archivo en Linux" > hola.txt
cat hola.txt
```

```
# salida:
Mi primer archivo en Linux
```

Acaban de crear un archivo de texto desde la terminal. `echo` escribe texto,
`>` lo redirige a un archivo, y `cat` muestra el contenido.

```bash
cd ~
```

---

## 22. Información del sistema

Algunos comandos útiles para saber qué tienen:

```bash
# Qué versión de Ubuntu
lsb_release -a
```

```bash
# Cuántos procesadores
nproc
```

```bash
# Cuánta RAM disponible
free -h
```

```bash
# Espacio en disco
df -h /
```

```bash
# Conexión a internet
ping -c 3 google.com
```

Si el ping funciona (3 paquetes transmitidos, 3 recibidos, 0% packet loss),
tienen conexión a internet desde Linux. Esto es importante para instalar
paquetes más adelante.

---

# EJERCICIO INTEGRADOR — Verifica tu instalación

---

## FASE 1: Pensar (10 minutos, sin computador)

Respondan en papel:

1. Qué diferencia hay entre WSL y una máquina virtual tradicional como
   VirtualBox?

2. Si escriben la contraseña en la terminal y no ven nada en pantalla,
   qué está pasando? ¿Es un error?

3. Si están en la terminal y el prompt dice `alumno@PC-AULA05:~$`,
   qué información pueden extraer?

4. Qué diferencia hay entre una ruta absoluta y una ruta relativa? Den un
   ejemplo de cada una.

5. Si ejecutan `cd /mnt/c/Users/alumno/Documents`, a dónde están accediendo?
   ¿Es un directorio de Linux o de Windows?

---

## FASE 2: Verificar (30 minutos, en el equipo)

Abran Ubuntu y ejecuten cada comando. Comprueben que la salida coincide.

**Verificación 1 — Están en el home:**
```bash
pwd
```
Salida esperada: `/home/alumno`

**Verificación 2 — El sistema reconoce su usuario:**
```bash
whoami
```
Salida esperada: `alumno`

**Verificación 3 — Es WSL2:**
```bash
cat /proc/version
```
Salida esperada: incluye `microsoft` y `WSL2`

**Verificación 4 — Ubuntu está actualizado:**
```bash
lsb_release -d
```
Salida esperada: incluye `Ubuntu`

**Verificación 5 — Tienen conexión a internet:**
```bash
ping -c 3 google.com
```
Salida esperada: 3 paquetes transmitidos, 3 recibidos, 0% packet loss.

**Verificación 6 — tree está instalado:**
```bash
tree --version
```
Salida esperada: `tree v2.x.x`

**Verificación 7 — git está instalado:**
```bash
git --version
```
Salida esperada: `git version 2.xx.x`

**Verificación 8 — Pueden ver Windows desde Linux:**
```bash
ls /mnt/c/Users/
```
Salida esperada: lista de carpetas de usuarios de Windows

**Verificación 9 — Navegar a la raíz y volver:**
```bash
cd /
ls
cd ~
pwd
```
Salida esperada: lista de directorios del sistema, luego `/home/alumno`

**Verificación 10 — Crear y verificar:**
```bash
mkdir prueba_dia2
echo "Funciona" > prueba_dia2/test.txt
cat prueba_dia2/test.txt
rm -r prueba_dia2
```
Salida esperada: `Funciona` y luego la carpeta desaparece.

---

## FASE 3: Demostrar (30 minutos)

**Reto 1 — Navegación:**
Desde el home, vayan a `/var/log`, listen su contenido, y vuelvan al home.
Todo sin usar rutas absolutas excepto la primera.

**Reto 2 — Información del sistema:**
Averigüen cuántos procesadores ve Ubuntu y cuánta RAM tiene asignada:
```bash
nproc
free -h
```

**Reto 3 — Acceso cruzado:**
Desde Linux, naveguen a su carpeta de Documentos de Windows:
```bash
ls /mnt/c/Users/$(cmd.exe /c "echo %USERNAME%" 2>/dev/null | tr -d '\r')/Documents/
```
O más simple si saben su nombre de usuario de Windows:
```bash
ls /mnt/c/Users/SU_USUARIO/Documents/
```

**Reto 4 — Crear estructura:**
Creen un directorio `~/primer_proyecto` con dos subdirectorios `src` y `docs`.
Dentro de `docs`, creen un archivo `notas.txt` con el texto "Día 2 completado".
Verifiquen con `tree ~/primer_proyecto`.

---

## Tarea con entrega

Subir a Google Classroom:

1. **Captura de pantalla** de la terminal mostrando `lsb_release -a`
2. **Captura de pantalla** de la terminal mostrando `ping -c 3 google.com`
3. **Captura de pantalla** de la terminal mostrando `ls /mnt/c/Users/`
4. **Captura de pantalla** de la terminal mostrando `tree ~/primer_proyecto`

**Formato:** 4 imágenes en formato PNG o JPG, nombradas:
`dia2_1_sistema.png`, `dia2_2_red.png`, `dia2_3_windows.png`, `dia2_4_proyecto.png`

**Cómo hacer capturas de pantalla en Windows:**
- `Win + Shift + S` abre la herramienta de recorte
- Seleccionen la zona de la terminal
- Se copia al portapapeles
- Peguen en Paint o directamente en Classroom

---

## Tabla de referencia rápida

| Concepto | Detalle |
|---|---|
| WSL | Windows Subsystem for Linux — Linux dentro de Windows |
| WSL2 | Versión que ejecuta un kernel Linux real |
| Instalar WSL | `wsl --install` (desde PowerShell admin) |
| Abrir Ubuntu | Menú Inicio > Ubuntu, o `wsl` desde PowerShell |
| Cerrar Ubuntu | `exit` o cerrar la ventana |
| Apagar WSL | `wsl --shutdown` desde PowerShell |
| Ver Windows desde Linux | `/mnt/c/` |
| Ver Linux desde Windows | `\\wsl$\Ubuntu\home\alumno` en Explorador |
| `pwd` | Muestra en qué directorio estoy |
| `ls` | Lista el contenido del directorio |
| `ls -la` | Lista todo, incluidos ocultos, con detalles |
| `cd directorio` | Entrar en un directorio |
| `cd ..` | Subir un nivel |
| `cd ~` o `cd` | Volver al home |
| `cd /` | Ir a la raíz del sistema |
| `whoami` | Muestra el usuario actual |
| `hostname` | Muestra el nombre de la máquina |
| `nproc` | Muestra el número de procesadores |
| `free -h` | Muestra la memoria RAM disponible |
| `mkdir nombre` | Crea un directorio |
| `echo "texto" > archivo` | Crea archivo con contenido |
| `cat archivo` | Muestra contenido de archivo |
| `rm -r directorio` | Borra directorio y contenido |
| `sudo apt update` | Actualiza lista de paquetes |
| `sudo apt install paquete` | Instala un programa |
| Contraseña VM | `alumno2026` (no se ve al escribir) |

---

## Créditos y referencias

| | |
|---|---|
| **Autor original** | Prof. Juan Marcelo Gutiérrez Miranda |
| **Institución** | @TodoEconometría |

---

> **Curso IFCD0112 — Programación con Lenguajes OO y BBDD Relacionales**
> Material complementario de estudio
>
> **Autor:** Prof. Juan Marcelo Gutiérrez Miranda
> **Institución:** @TodoEconometría
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
