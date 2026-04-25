# Día 3: Terminal en serio — crear, mover, copiar y destruir

El viernes instalaron Ubuntu y ejecutaron sus primeros comandos.
Hoy la terminal deja de ser una curiosidad y se convierte en su
herramienta principal. Todo lo que hagan en este curso pasa por aquí.

Prof. Juan Marcelo Gutiérrez Miranda

**Curso IFCD0112 — Semana 1, Día 3 (Lunes)**
**Objetivo:** Dominar la creación, copia, movimiento y eliminación de ficheros
y directorios desde la terminal. Construir la estructura de carpetas del
proyecto Launch Control.

> Este manual es de consulta. Sigan los pasos con la terminal Ubuntu abierta y la
> terminal abierta (Ctrl + Alt + T).

---

## PASO 0 — De dónde venimos

```
╔══════════════════════════════════════════════════════════════════╗
║              EVOLUCIÓN DE AMTIGRAVITY LAUNCH CONTROL             ║
╠══════════════════════════════════════════════════════════════════╣
║                                                                  ║
║  Día 1   BOE + Propuesta       Mapa del curso                    ║
║  Día 2   WSL + Ubuntu          Taller montado, pwd/ls/cd         ║
║  ► DÍA 3  Terminal + ficheros  Crear la estructura del proyecto  ║
║                                                                  ║
╚══════════════════════════════════════════════════════════════════╝
```

| Aspecto | Día 2 | Hoy (Día 3) |
|---------|-------|-------------|
| Terminal | 5 comandos básicos | Dominio completo de ficheros |
| Proyecto | No existía físicamente | Tiene estructura de carpetas |
| Nivel | Observar el sistema | Modificar el sistema |

El proyecto no cambia. Cambia que hoy empieza a existir como carpetas reales.

---

## Índice del día

| Parte | Contenido |
|---|---|
| I   | El sistema de archivos de Linux |
| II  | Crear: mkdir, touch, echo y redirecciones |
| III | Copiar, mover y renombrar: cp y mv |
| IV  | Eliminar: rm y rmdir — el poder destructivo |
| V   | Construir Launch Control — estructura del proyecto |

---

# PARTE I — EL SISTEMA DE ARCHIVOS DE LINUX

---

## 1. Todo es un archivo

En Linux, **todo** es un archivo. No es una metáfora — es literal:

| Elemento | En Windows | En Linux |
|---|---|---|
| Un documento de texto | Archivo | Archivo |
| Una carpeta | Carpeta | Archivo (de tipo directorio) |
| El disco duro | Unidad C: | Archivo (`/dev/sda`) |
| La impresora | Dispositivo | Archivo (`/dev/lp0`) |
| La memoria RAM | (no accesible) | Archivo (`/dev/mem`) |
| Un proceso en ejecución | (no accesible) | Archivo (`/proc/1234/status`) |

Esta filosofía simplifica enormemente la administración: si saben leer y
escribir archivos, saben interactuar con cualquier parte del sistema.

---

## 2. Estructura del árbol de directorios

Linux organiza todo en un único árbol que empieza en `/` (raíz). No hay
letras de unidad como en Windows (C:, D:).

```
/                          ← Raíz del sistema (equivale a C:\ en Windows)
├── home/                  ← Carpetas personales de cada usuario
│   └── alumno/            ← TU carpeta personal
│       ├── Descargas/
│       ├── Documentos/
│       ├── Escritorio/
│       └── ...
├── etc/                   ← Configuración del sistema
│   ├── hostname           ← Nombre del equipo
│   ├── passwd             ← Lista de usuarios
│   └── apt/               ← Configuración del gestor de paquetes
├── var/                   ← Datos variables
│   ├── log/               ← Registros del sistema (logs)
│   └── tmp/               ← Temporales que sobreviven al reinicio
├── tmp/                   ← Temporales que se borran al reiniciar
├── usr/                   ← Programas instalados
│   ├── bin/               ← Ejecutables de programas
│   └── lib/               ← Librerías compartidas
├── bin/                   ← Comandos esenciales (ls, cp, mv...)
├── sbin/                  ← Comandos de administración (solo root)
├── dev/                   ← Dispositivos (discos, terminales, USB...)
├── proc/                  ← Información de procesos en ejecución
├── root/                  ← Home del usuario root (administrador)
└── opt/                   ← Software opcional instalado manualmente
```

### 2.1 Las carpetas que más van a usar

| Directorio | Para qué sirve | Con qué frecuencia |
|---|---|---|
| `~/` (home) | Su espacio personal. Aquí va TODO su trabajo | Siempre |
| `/tmp` | Archivos temporales. Se borran al reiniciar | A veces |
| `/etc` | Configuración. Cuando instalen PostgreSQL, su config está aquí | Fase II en adelante |
| `/var/log` | Logs. Cuando algo falle, aquí buscan el error | Debugging |

---

## 3. Nombres de archivos en Linux — reglas que deben saber

Linux es diferente a Windows en varios aspectos críticos:

| Regla | Windows | Linux |
|---|---|---|
| Mayúsculas/minúsculas | Son lo mismo | Son DIFERENTES |
| Espacios en nombres | Comunes | Legales pero problemáticos |
| Extensión del archivo | Obligatoria (.txt, .doc) | Opcional (es solo parte del nombre) |
| Caracteres prohibidos | \ / : * ? " < > \| | Solo / y null |
| Longitud máxima nombre | 255 caracteres | 255 caracteres |
| Archivos ocultos | Atributo especial | Empiezan por punto |

### 3.1 Las mayúsculas importan

```bash
touch Archivo.txt archivo.txt ARCHIVO.txt
ls
```

```
# salida:
Archivo.txt  archivo.txt  ARCHIVO.txt
```

Son **tres archivos diferentes**. En Windows serían el mismo. Esto es fuente
de errores constantes para principiantes. Acostúmbrense.

### 3.2 Eviten espacios en nombres

```bash
# MALO — funciona pero da problemas
mkdir "Mi Proyecto"

# BUENO — usa guiones o guiones bajos
mkdir mi-proyecto
mkdir mi_proyecto
```

Los espacios requieren comillas o escapar con `\`. Es una molestia innecesaria.
Convención profesional: **guiones bajos o guiones, nunca espacios**.

### 3.3 Archivos ocultos

Cualquier archivo cuyo nombre empiece por `.` (punto) es oculto:

```bash
ls          # NO muestra ocultos
ls -a       # Muestra TODO, incluidos ocultos
ls -la      # Todo con detalles
```

Muchos archivos de configuración son ocultos: `.bashrc`, `.profile`, `.gitconfig`.
No los toquen por ahora — pero sepan que están ahí.

---

# PARTE II — CREAR: MKDIR, TOUCH, ECHO Y REDIRECCIONES

---

## 4. mkdir — crear directorios

```bash
mkdir nombre_directorio
```

Crea un directorio vacío en la ubicación actual.

```bash
cd ~
mkdir practicas
ls
```

```
# salida:
Descargas  Documentos  Escritorio  Imagenes  Musica  Plantillas  practicas  Publico  Videos
```

### 4.1 Crear directorios anidados con -p

Sin `-p`, no pueden crear un directorio dentro de otro que no existe:

```bash
mkdir uno/dos/tres
```

```
# salida:
mkdir: no se puede crear el directorio «uno/dos/tres»: No existe el fichero o el directorio
```

Con `-p` (parents), crea toda la cadena de una vez:

```bash
mkdir -p uno/dos/tres
```

No da error. Crea `uno`, dentro `dos`, dentro `tres`. Todo de un golpe.

**Verificación:**

```bash
ls -R uno
```

```
# salida:
uno:
dos

uno/dos:
tres

uno/dos/tres:
```

El flag `-R` (recursive) lista el contenido de todos los subdirectorios.

### 4.2 Crear varios directorios a la vez

```bash
mkdir dir_a dir_b dir_c
```

Crea los tres en el directorio actual.

```bash
mkdir -p proyecto/{src,docs,tests,data}
```

Crea `proyecto` con cuatro subdirectorios dentro. Las llaves `{}` se
expanden automáticamente. Esto se llama **brace expansion** y es muy útil.

```bash
ls proyecto
```

```
# salida:
data  docs  src  tests
```

---

## 5. touch — crear archivos vacíos

```bash
touch nombre_archivo
```

Crea un archivo vacío. Si el archivo ya existe, actualiza su fecha de
modificación sin cambiar el contenido.

```bash
cd ~/practicas
touch notas.txt
ls -l
```

```
# salida:
total 0
-rw-rw-r-- 1 alumno alumno 0 abr 13 09:30 notas.txt
```

El `0` indica que el archivo tiene 0 bytes — está vacío.

### 5.1 Crear varios archivos a la vez

```bash
touch archivo1.txt archivo2.txt archivo3.txt
```

O con brace expansion:

```bash
touch modulo_{01,02,03,04,05}.sql
ls
```

```
# salida:
archivo1.txt  archivo2.txt  archivo3.txt  modulo_01.sql  modulo_02.sql
modulo_03.sql  modulo_04.sql  modulo_05.sql  notas.txt
```

---

## 6. echo — escribir texto

`echo` imprime texto en la pantalla (la "salida estándar"):

```bash
echo "Hola mundo"
```

```
# salida:
Hola mundo
```

Por sí solo no es muy útil. Se vuelve potente con **redirecciones**.

---

## 7. Redirecciones — escribir en archivos

### 7.1 El operador > (sobreescribir)

Redirige la salida de un comando a un archivo. Si el archivo existe, lo
**sobreescribe completamente**:

```bash
echo "Primera línea" > notas.txt
cat notas.txt
```

```
# salida:
Primera línea
```

`cat` (concatenate) muestra el contenido de un archivo. Lo veremos en detalle
más adelante.

### 7.2 El operador >> (añadir)

Añade al final del archivo sin borrar lo que había:

```bash
echo "Segunda línea" >> notas.txt
echo "Tercera línea" >> notas.txt
cat notas.txt
```

```
# salida:
Primera línea
Segunda línea
Tercera línea
```

### 7.3 Diferencia crítica: > vs >>

| Operador | Comportamiento | Peligro |
|---|---|---|
| `>` | Sobreescribe el archivo ENTERO | Si tenía datos, los pierden |
| `>>` | Añade al final | Seguro, no borra nada |

```
╔══════════════════════════════════════════════════════════════╗
║  CUIDADO: echo "texto" > archivo_importante                 ║
║                                                              ║
║  Si ese archivo tenía 500 líneas de trabajo,                 ║
║  ahora tiene 1 línea. Las 500 anteriores: PERDIDAS.         ║
║  No hay papelera de reciclaje en la terminal.                ║
║                                                              ║
║  Regla: si no están SEGUROS, usen >> en vez de >             ║
╚══════════════════════════════════════════════════════════════╝
```

---

## 8. cat — ver contenido de archivos

```bash
cat nombre_archivo
```

Muestra el contenido completo del archivo en la terminal.

```bash
cat notas.txt
```

```
# salida:
Primera línea
Segunda línea
Tercera línea
```

### 8.1 cat con números de línea

```bash
cat -n notas.txt
```

```
# salida:
     1  Primera línea
     2  Segunda línea
     3  Tercera línea
```

### 8.2 Ver solo el principio o el final

Para archivos grandes, no van a querer ver TODO:

```bash
head -5 /etc/passwd     # Muestra las primeras 5 líneas
tail -5 /etc/passwd     # Muestra las últimas 5 líneas
```

### 8.3 less — ver archivos largos con scroll

```bash
less /etc/passwd
```

Abre el archivo en un visor con scroll:
- **Espacio** o **Page Down**: avanzar una página
- **b** o **Page Up**: retroceder una página
- **/** seguido de texto: buscar
- **q**: salir

---

## 9. Contar: wc (word count)

```bash
wc notas.txt
```

```
# salida:
 3  6 45 notas.txt
```

Tres números: **líneas**, **palabras**, **bytes**.

| Flag | Qué cuenta | Ejemplo |
|---|---|---|
| `wc -l` | Solo líneas | `wc -l notas.txt` → 3 |
| `wc -w` | Solo palabras | `wc -w notas.txt` → 6 |
| `wc -c` | Solo bytes | `wc -c notas.txt` → 45 |

---

# PARTE III — COPIAR, MOVER Y RENOMBRAR: CP Y MV

---

## 10. cp — copiar archivos

```bash
cp origen destino
```

Copia un archivo de un sitio a otro.

```bash
cp notas.txt notas_copia.txt
ls
```

```
# salida:
archivo1.txt  archivo2.txt  archivo3.txt  modulo_01.sql  modulo_02.sql
modulo_03.sql  modulo_04.sql  modulo_05.sql  notas.txt  notas_copia.txt
```

Ahora hay dos archivos idénticos. El original no se modifica.

### 10.1 Copiar a otro directorio

```bash
cp notas.txt ~/Documentos/
ls ~/Documentos/
```

```
# salida:
notas.txt
```

Si indican un directorio como destino (con `/` al final), el archivo se
copia dentro con el mismo nombre.

### 10.2 Copiar y renombrar a la vez

```bash
cp notas.txt ~/Documentos/apuntes_dia3.txt
```

Copia el archivo y le da un nombre nuevo en el destino.

### 10.3 Copiar directorios con -r

`cp` por defecto solo copia archivos. Para copiar directorios (con todo
su contenido), necesitan `-r` (recursive):

```bash
cp -r uno/ uno_backup/
ls
```

```
# salida:
... uno  uno_backup ...
```

Si olvidan el `-r`:

```bash
cp uno/ otro/
```

```
# salida:
cp: se omite el directorio «uno/»
```

---

## 11. mv — mover y renombrar

`mv` tiene dos funciones: **mover** archivos a otra ubicación y
**renombrar** archivos.

### 11.1 Renombrar

```bash
mv notas_copia.txt respaldo_notas.txt
ls
```

El archivo cambia de nombre. No se crea copia — se renombra directamente.

### 11.2 Mover a otro directorio

```bash
mv respaldo_notas.txt ~/Documentos/
ls ~/Documentos/
```

```
# salida:
apuntes_dia3.txt  notas.txt  respaldo_notas.txt
```

El archivo ya no está en `practicas` — se ha **movido** a `Documentos`.

### 11.3 Mover y renombrar a la vez

```bash
mv ~/Documentos/respaldo_notas.txt ~/Documentos/backup.txt
```

### 11.4 Mover directorios

A diferencia de `cp`, `mv` mueve directorios **sin necesitar -r**:

```bash
mv uno_backup/ ~/Documentos/
```

---

## 12. Diferencia clave: cp vs mv

| Acción | cp | mv |
|---|---|---|
| Archivo original | Se mantiene | Desaparece del origen |
| Requiere -r para directorios | Sí | No |
| Uso típico | Hacer backups, duplicar | Reorganizar, renombrar |

```
╔══════════════════════════════════════════════════════════════╗
║  cp = fotocopiadora (el original queda donde estaba)         ║
║  mv = mudanza (el original se va al destino)                 ║
╚══════════════════════════════════════════════════════════════╝
```

---

# PARTE IV — ELIMINAR: RM Y RMDIR — EL PODER DESTRUCTIVO

---

## 13. rmdir — eliminar directorios vacíos

```bash
mkdir temporal
rmdir temporal
ls
```

`rmdir` solo funciona con directorios **vacíos**. Si tiene contenido:

```bash
rmdir uno
```

```
# salida:
rmdir: fallo al borrar «uno»: El directorio no está vacío
```

Esto es una protección. Para borrar directorios con contenido, necesitan `rm`.

---

## 14. rm — eliminar archivos

```bash
rm archivo1.txt
ls
```

El archivo desaparece. **Sin papelera de reciclaje. Sin confirmación.
Sin vuelta atrás.**

### 14.1 rm con confirmación: -i

```bash
rm -i archivo2.txt
```

```
# salida:
rm: ¿borrar el fichero regular vacío «archivo2.txt»? (s/n)
```

Deben responder `s` (sí) o `n` (no). Es una red de seguridad.

### 14.2 rm recursivo: -r

Para borrar un directorio con todo su contenido:

```bash
rm -r uno
```

Borra `uno`, y todo lo que había dentro (dos, tres, y cualquier archivo).

### 14.3 rm -rf — el comando más peligroso de Linux

```bash
rm -rf directorio
```

`-r` = recursivo (borra todo el contenido).
`-f` = force (no pide confirmación, no muestra errores).

```
╔══════════════════════════════════════════════════════════════╗
║                                                              ║
║  ██████████████████████████████████████████████████████       ║
║  █                                                    █      ║
║  █   rm -rf /    ← NUNCA EJECUTEN ESTO                █      ║
║  █                                                    █      ║
║  █   Borra TODO el sistema. Todo. Incluyendo el       █      ║
║  █   propio comando rm. Linux se autodestruye.        █      ║
║  █                                                    █      ║
║  █   En producción: pierden los datos de todos        █      ║
║  █   los clientes. Los despiden. Los demandan.        █      ║
║  █                                                    █      ║
║  ██████████████████████████████████████████████████████       ║
║                                                              ║
╚══════════════════════════════════════════════════════════════╝
```

### 14.4 Buenas prácticas con rm

1. **Siempre verificar dónde están** antes de borrar: `pwd`
2. **Siempre listar** antes de borrar: `ls` para ver qué hay
3. **Usar -i** si no están seguros: `rm -i archivo`
4. **Nunca usar rm -rf con rutas absolutas** como `/` o `/home`
5. **En duda, no borren** — pregunten antes

---

## 15. Wildcards — patrones para seleccionar archivos

Los wildcards (comodines) permiten seleccionar múltiples archivos a la vez:

| Wildcard | Significado | Ejemplo |
|---|---|---|
| `*` | Cualquier cosa (cero o más caracteres) | `*.txt` = todos los .txt |
| `?` | Exactamente un carácter | `archivo?.txt` = archivo1.txt, archivo2.txt... |
| `[abc]` | Uno de los caracteres indicados | `archivo[123].txt` |
| `[0-9]` | Un rango de caracteres | `modulo_[0-9][0-9].sql` |

### 15.1 Ejemplos prácticos

```bash
cd ~/practicas
ls *.sql
```

```
# salida:
modulo_01.sql  modulo_02.sql  modulo_03.sql  modulo_04.sql  modulo_05.sql
```

```bash
ls modulo_0[1-3].sql
```

```
# salida:
modulo_01.sql  modulo_02.sql  modulo_03.sql
```

```bash
rm *.sql
ls
```

Todos los `.sql` borrados de un golpe. Potente y peligroso.

---

## 16. Historial y atajos de teclado

### 16.1 Historial de comandos

```bash
history
```

Muestra todos los comandos que han ejecutado. Cada uno tiene un número.

```bash
!42          # Ejecuta el comando número 42 del historial
!!           # Ejecuta el último comando
!ls          # Ejecuta el último comando que empezaba por "ls"
```

### 16.2 Atajos de teclado esenciales

| Atajo | Qué hace |
|---|---|
| Tab | Autocompleta nombres de archivos y comandos |
| Tab Tab | Muestra opciones si hay ambigüedad |
| Flecha arriba | Comando anterior |
| Flecha abajo | Comando siguiente |
| Ctrl + C | Cancela el comando actual |
| Ctrl + L | Limpia la pantalla (equivale a `clear`) |
| Ctrl + A | Va al principio de la línea |
| Ctrl + E | Va al final de la línea |
| Ctrl + R | Búsqueda inversa en el historial |
| Ctrl + D | Cierra la terminal (equivale a `exit`) |

### 16.3 Tab — su mejor amigo

El **tabulador** autocompleta. Si escriben `cd Doc` y pulsan Tab, se
completa a `cd Documentos/`. Si hay varias opciones, pulsen Tab dos veces
para verlas todas.

Úsenlo SIEMPRE. Ahorra tiempo y evita errores de escritura.

---

# PARTE V — CONSTRUIR LAUNCH CONTROL

---

## 17. Estructura del proyecto

Vamos a crear la estructura de carpetas del proyecto Amtigravity Launch Control.
Este es el primer paso real del proyecto que los acompañará todo el curso.

```
~/launch-control/
├── docs/                    ← Documentación del proyecto
│   ├── diagramas/           ← Diagramas ER, UML, flujos
│   └── notas/               ← Notas de cada día
├── sql/                     ← Scripts SQL (Fase II)
│   ├── schema/              ← Definición de tablas
│   ├── data/                ← Datos iniciales (INSERT)
│   └── queries/             ← Consultas útiles
├── src/                     ← Código fuente (Fase III)
│   ├── models/              ← Modelos de datos
│   ├── api/                 ← Endpoints de la API
│   └── tests/               ← Tests automatizados
├── scripts/                 ← Scripts de utilidad (bash)
├── README.md                ← Descripción del proyecto
└── .gitignore               ← Archivos que Git debe ignorar
```

### 17.1 Crear toda la estructura

Ejecuten esto en la terminal:

```bash
cd ~
mkdir -p launch-control/{docs/{diagramas,notas},sql/{schema,data,queries},src/{models,api,tests},scripts}
```

**Verificación:**

```bash
ls -R launch-control
```

```
# salida:
launch-control:
docs  scripts  sql  src

launch-control/docs:
diagramas  notas

launch-control/docs/diagramas:

launch-control/docs/notas:

launch-control/sql:
data  queries  schema

launch-control/sql/data:

launch-control/sql/queries:

launch-control/sql/schema:

launch-control/src:
api  models  tests

launch-control/src/models:

launch-control/src/api:

launch-control/src/tests:

launch-control/scripts:
```

### 17.2 Crear los archivos iniciales

```bash
cd ~/launch-control

# README del proyecto
echo "# Amtigravity Launch Control" > README.md
echo "" >> README.md
echo "Plataforma de gestión de lanzamientos espaciales." >> README.md
echo "Proyecto del curso IFCD0112." >> README.md

# Verificar
cat README.md
```

```
# salida:
# Amtigravity Launch Control

Plataforma de gestión de lanzamientos espaciales.
Proyecto del curso IFCD0112.
```

```bash
# Archivo .gitignore (lo usaremos cuando veamos Git)
echo "*.pyc" > .gitignore
echo "__pycache__/" >> .gitignore
echo ".env" >> .gitignore
echo "*.log" >> .gitignore

# Nota del día 3
echo "Día 3: Creada estructura de carpetas del proyecto" > docs/notas/dia03.txt
echo "Fecha: $(date)" >> docs/notas/dia03.txt

# Verificar
cat docs/notas/dia03.txt
```

```
# salida:
Día 3: Creada estructura de carpetas del proyecto
Fecha: lun 13 abr 2026 10:30:00 CEST
```

### 17.3 Crear scripts placeholder

```bash
# Script de inicio (lo completaremos en Fase II)
echo "#!/bin/bash" > scripts/setup.sh
echo "# Script de configuración de Launch Control" >> scripts/setup.sh
echo "echo 'Setup en construcción...'" >> scripts/setup.sh

# Darle permiso de ejecución (veremos permisos mañana)
chmod +x scripts/setup.sh

# Verificar que funciona
bash scripts/setup.sh
```

```
# salida:
Setup en construcción...
```

---

## 18. El comando tree (bonus)

`tree` muestra la estructura de directorios de forma visual. No viene
instalado por defecto, pero se instala en un segundo:

```bash
sudo apt install tree -y
```

```bash
tree ~/launch-control
```

```
# salida:
/home/alumno/launch-control
├── docs
│   ├── diagramas
│   └── notas
│       └── dia03.txt
├── .gitignore
├── README.md
├── scripts
│   └── setup.sh
├── sql
│   ├── data
│   ├── queries
│   └── schema
└── src
    ├── api
    ├── models
    └── tests

12 directories, 4 files
```

`tree` es su aliado para verificar estructuras de carpetas. Úsenlo
siempre que quieran confirmar que todo está donde debe estar.

---

## 19. Resumen de comandos aprendidos hoy

| Comando | Qué hace | Ejemplo |
|---|---|---|
| `mkdir nombre` | Crea directorio | `mkdir docs` |
| `mkdir -p a/b/c` | Crea directorios anidados | `mkdir -p sql/schema` |
| `touch archivo` | Crea archivo vacío | `touch notas.txt` |
| `echo "texto"` | Imprime texto | `echo "hola"` |
| `>` | Redirige (sobreescribe) | `echo "x" > archivo.txt` |
| `>>` | Redirige (añade) | `echo "y" >> archivo.txt` |
| `cat archivo` | Muestra contenido | `cat README.md` |
| `cat -n archivo` | Muestra con números de línea | `cat -n README.md` |
| `head -N archivo` | Muestra primeras N líneas | `head -5 /etc/passwd` |
| `tail -N archivo` | Muestra últimas N líneas | `tail -5 /etc/passwd` |
| `less archivo` | Visor con scroll | `less /etc/passwd` |
| `wc -l archivo` | Cuenta líneas | `wc -l README.md` |
| `cp origen destino` | Copia archivo | `cp a.txt b.txt` |
| `cp -r dir/ dir2/` | Copia directorio | `cp -r docs/ docs_bk/` |
| `mv origen destino` | Mueve o renombra | `mv old.txt new.txt` |
| `rm archivo` | Elimina archivo | `rm temporal.txt` |
| `rm -r directorio` | Elimina directorio y contenido | `rm -r tests/` |
| `rm -i archivo` | Elimina con confirmación | `rm -i datos.csv` |
| `rmdir directorio` | Elimina directorio vacío | `rmdir vacío/` |
| `*` | Wildcard: cualquier cosa | `ls *.txt` |
| `?` | Wildcard: un carácter | `ls archivo?.txt` |
| `history` | Historial de comandos | `history` |
| `tree` | Muestra árbol de directorios | `tree launch-control` |
| Tab | Autocompleta | Escribir parcial + Tab |

---

# EJERCICIO INTEGRADOR — Misión: Base de Datos de Cohetes

---

## FASE 1: Pensar (10 minutos, sin computador)

Respondan en papel:

1. Qué diferencia hay entre `>` y `>>`? Qué pasa si usan `>` en un archivo
   que ya tenía datos?

2. Si ejecutan `rm -rf ~/launch-control`, qué ocurre? Se puede deshacer?

3. Qué comando usarían para crear esta estructura de un solo golpe?
   ```
   misiones/
   ├── falcon9/
   ├── starship/
   └── electron/
   ```

4. Si están en `/home/alumno/launch-control/sql/schema` y quieren ir a
   `/home/alumno/launch-control/src/models`, escriban el comando usando
   ruta relativa (con `..`).

5. Qué diferencia hay entre `cp` y `mv`? Después de ejecutar cada uno,
   cuántos archivos hay?

---

## FASE 2: Codificar (30 minutos)

Creen esta estructura completa desde cero (sin mirar el manual):

```
~/mision-espacial/
├── cohetes/
│   ├── falcon9/
│   │   ├── especificaciones.txt    (contenido: "Carga: 22800 kg a LEO")
│   │   └── historial.txt           (contenido: "Primer vuelo: 2010")
│   ├── starship/
│   │   ├── especificaciones.txt    (contenido: "Carga: 150000 kg a LEO")
│   │   └── historial.txt           (contenido: "Primer vuelo: 2023")
│   └── electron/
│       ├── especificaciones.txt    (contenido: "Carga: 300 kg a LEO")
│       └── historial.txt           (contenido: "Primer vuelo: 2017")
├── bases/
│   ├── cabo_canaveral.txt          (contenido: "Ubicación: Florida, USA")
│   ├── baikonur.txt                (contenido: "Ubicación: Kazajistán")
│   └── kourou.txt                  (contenido: "Ubicación: Guayana Francesa")
├── misiones/
│   └── log.txt                     (contenido: "Registro de misiones vacío")
└── README.md                       (contenido: "# Base de Datos de Cohetes")
```

**Criterios de verificación:**

```bash
tree ~/mision-espacial
# Debe mostrar la estructura exacta

cat ~/mision-espacial/cohetes/falcon9/especificaciones.txt
# Debe decir: Carga: 22800 kg a LEO

cat ~/mision-espacial/bases/cabo_canaveral.txt
# Debe decir: Ubicación: Florida, USA

wc -l ~/mision-espacial/cohetes/starship/historial.txt
# Debe decir: 1
```

**Después de verificar, hagan una copia de seguridad:**

```bash
cp -r ~/mision-espacial ~/mision-espacial-backup
```

**Después del backup, muevan el archivo `log.txt` de misiones a la raíz
del proyecto y renómbrenlo a `registro.txt`:**

```bash
mv ~/mision-espacial/misiones/log.txt ~/mision-espacial/registro.txt
```

---

## FASE 3: Demostrar (30 minutos)

**Reto 1 — Operaciones en cadena:**
Sin salir del directorio `~/mision-espacial`, copien el archivo
`especificaciones.txt` de falcon9 a la carpeta misiones. Renómbrenlo a
`specs_f9.txt`. Verifiquen con `cat`.

**Reto 2 — Limpieza selectiva:**
Borren TODOS los archivos `historial.txt` de los tres cohetes usando
wildcards en un solo comando. Verifiquen con `tree`.

**Reto 3 — Reconstrucción rápida:**
Sin mirar el manual, recreen los archivos `historial.txt` que acaban de
borrar, con el mismo contenido que tenían antes. Tiempo máximo: 3 minutos.

**Reto 4 — Información del proyecto:**
Ejecuten un solo comando que cuente cuántos archivos regulares (no
directorios) tiene todo el proyecto `mision-espacial`. Pista: combinen
`find` y `wc`:

```bash
find ~/mision-espacial -type f | wc -l
```

---

## Tarea con entrega

Subir a Google Classroom:

1. **Captura de pantalla** de `tree ~/launch-control` mostrando la estructura
   completa del proyecto
2. **Captura de pantalla** de `cat ~/launch-control/README.md` mostrando el
   contenido del README
3. **Captura de pantalla** de `tree ~/mision-espacial` mostrando la estructura
   del ejercicio integrador completa
4. **Captura de pantalla** mostrando el resultado de
   `find ~/mision-espacial -type f | wc -l`

**Formato:** 4 imágenes PNG o JPG nombradas:
`dia3_1_launch_control.png`, `dia3_2_readme.png`, `dia3_3_mision.png`,
`dia3_4_conteo.png`

---

## Tabla de referencia rápida

| Concepto | Detalle |
|---|---|
| Todo es un archivo | Linux trata directorios, dispositivos y procesos como archivos |
| Mayúsculas importan | `Archivo.txt` ≠ `archivo.txt` |
| No usar espacios | Usar guiones_bajos o guiones-medios |
| `mkdir -p` | Crea cadena de directorios |
| `touch` | Crea archivo vacío |
| `>` | Sobreescribe archivo |
| `>>` | Añade al final |
| `cat` | Muestra contenido |
| `cp` | Copia (mantiene original) |
| `cp -r` | Copia directorios |
| `mv` | Mueve o renombra |
| `rm` | Elimina sin papelera |
| `rm -r` | Elimina directorios |
| `rm -i` | Elimina con confirmación |
| `*` | Wildcard: todo |
| Tab | Autocompletar |
| `tree` | Visualizar estructura |
| `history` | Ver historial |
| Ctrl+C | Cancelar comando |
| Ctrl+L | Limpiar pantalla |

---

## Commit del día

```bash
cd ~/launch-control
echo "Día 3: estructura de carpetas creada" >> docs/notas/dia03.txt
```

*(Git aún no está instalado. El commit real se hará cuando veamos Git.
Por ahora, documentamos en el archivo de notas.)*

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
