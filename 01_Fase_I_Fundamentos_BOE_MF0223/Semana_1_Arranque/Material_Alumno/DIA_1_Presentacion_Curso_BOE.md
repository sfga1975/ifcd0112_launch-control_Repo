# Qué van a aprender y por qué estas herramientas
**Certificado:** IFCD0112 -- Programación con lenguajes OO y BBDD Relacionales
**Duración:** 630 horas de clase + 80 horas de prácticas en empresa
**Centro:** Amtigravity | Abril -- Octubre 2026

---

## Antes de empezar: una pregunta honesta

Este curso está regulado por una ficha oficial del Ministerio de Empleo
(la tienen en la carpeta del curso: `IFCD0112_ficha.pdf`). Esa ficha dice
**qué competencias** tienen que adquirir. Pero no dice **con qué herramientas
concretas** hay que hacerlo, porque las herramientas cambian cada pocos años
y las competencias son las que son.

La ficha se publicó en 2013. Desde entonces el mundo de la programación ha
cambiado bastante. Lo que no ha cambiado es lo que tienen que saber hacer
al terminar:

1. Configurar y manejar sistemas informáticos
2. Programar bases de datos relacionales
3. Desarrollar aplicaciones con lenguajes orientados a objetos

Eso es exactamente lo que van a hacer. La diferencia es que lo van a hacer
con las herramientas que se usan hoy en las empresas, no con las de hace
una década.

Este documento explica, módulo a módulo, qué van a aprender y con qué
herramientas. Si en algún momento les preguntan "pero esto cubre lo de la
ficha oficial?", la respuesta es sí, en todos los casos.

---

## El curso de un vistazo

El certificado tiene **3 módulos grandes** (llamados MF). Nosotros los
organizamos en 3 fases, una detrás de otra:

```
FASE I                     FASE II                    FASE III
Sistemas informáticos      Bases de datos             Programación
170 horas (~34 días)       210 horas (~42 días)       250 horas (~50 días)

  Su equipo,                 Donde viven los            Programar
  su terminal,               datos y cómo               aplicaciones
  sus herramientas            preguntarles cosas         de verdad
```

Cada fase los prepara para la siguiente. No pueden programar una base de
datos si no saben moverse por un equipo. No pueden construir una
aplicación si no saben consultar datos. Por eso el orden importa.

---

## FASE I -- Sistemas informáticos (170 horas)

### Lo que dice la ficha oficial

> *Configurar y explotar sistemas informáticos.*

Traducido: saber cómo funciona un equipo por dentro, manejar un sistema
operativo, organizar archivos, y usar las herramientas básicas que necesita
un programador.

### Con qué lo van a aprender

| Lo que van a aprender | Herramienta que usaremos | Qué dice la ficha |
|------------------------|--------------------------|-------------------|
| Cómo funciona un equipo: CPU, memoria, disco, y por qué importa para que sus programas vayan rápido | Su propio portátil + máquinas virtuales | *"Computadores para bases de datos"* (UF1465, 60h) |
| Manejar un sistema operativo de verdad, el que usan los servidores del mundo real | **Linux** (Ubuntu) a través de **WSL** (y más adelante **VirtualBox**) | *"Configurar sistemas operativos"* |
| La terminal: navegar, crear archivos, mover carpetas, automatizar tareas | **Bash** (la línea de comandos de Linux) | *"Intérpretes de comandos, gestión de archivos"* |
| Permisos: quién puede leer, escribir o ejecutar cada archivo | Comandos `chmod`, `chown` en Linux | *"Seguridad del sistema de archivos"* |
| Dónde se guardan los datos y cómo no perderlos | Sistemas de archivos Linux + **Docker** para aislar entornos | *"Sistemas de almacenamiento"* (UF1466, 70h) |
| Control de versiones: guardar cada cambio de tu código y poder volver atrás | **Git** + **GitHub** | *"Herramientas de gestión de cambios"* |
| Trabajo en equipo: cómo 4 personas pueden tocar el mismo código sin pisarse | Ramas, merge y pull requests en **GitHub** | *"Herramientas de control de cambios"* |
| Documentación técnica: escribir instrucciones claras para otros programadores | **Markdown** (el formato que usa GitHub, Stack Overflow, etc.) | *"Elaborar documentación técnica"* (UF1467, 40h) |
| Cómo funciona Internet por dentro: qué pasa cuando abren una web | HTTP, DNS, puertos -- conceptos, no teoría | *"Internet para consulta y generación de documentación"* |

### Qué NO van a hacer (y por qué)

La ficha oficial menciona temas como el procesador 8086, lenguaje ensamblador,
cintas magnéticas o el sistema operativo Solaris. Son temas que tenían sentido
cuando se escribió el programa, pero que hoy no les van a ayudar a encontrar
trabajo como programadores.

No quitamos competencias: las cubrimos con herramientas actuales. Por ejemplo,
la ficha pide que sepan "gestionar cambios" -- nosotros les enseñamos **Git**,
que es lo que usa el 95% de las empresas. La ficha pide que sepan "transferir
ficheros" -- nosotros les enseñamos `git push` y `scp`, no FTP.

Las horas que ahorramos en temas obsoletos las invertimos en **Linux**,
**Git** y **Docker**, que son las tres herramientas que van a usar todos
los días de su carrera.

---

## FASE II -- Bases de datos (210 horas)

### Lo que dice la ficha oficial

> *Programar bases de datos relacionales.*

Traducido: diseñar dónde van a vivir los datos de una aplicación, y saber
preguntarles cosas con un lenguaje específico (SQL).

### Con qué lo van a aprender

| Lo que van a aprender | Herramienta que usaremos | Qué dice la ficha |
|------------------------|--------------------------|-------------------|
| Diseñar una base de datos: qué tablas necesitan, cómo se relacionan entre sí | Diagramas E-R con **draw.io** y **dbdiagram.io** | *"Diseño de bases de datos relacionales"* (UF2175, 50h) |
| Normalización: organizar los datos para que no se repitan ni se contradigan | Teoría (1FN, 2FN, 3FN) + práctica real | *"Teoría de la normalización"* |
| Crear tablas, definir columnas, establecer reglas | **SQL** (lenguaje DDL) en **PostgreSQL** | *"Definición y manipulación de datos"* (UF2176, 80h) |
| Consultar datos: filtrar, ordenar, agrupar, combinar tablas | **SQL** (lenguaje DML): SELECT, JOIN, GROUP BY, subconsultas | *"Formulación de consultas de manipulación"* |
| Insertar, actualizar y borrar datos sin romper nada | INSERT, UPDATE, DELETE con transacciones | *"Construcción de consultas de inserción, modificación, borrado"* |
| Hacer que la base de datos haga cosas sola (automatizar) | **PL/pgSQL**: funciones, procedimientos y triggers | *"Desarrollo de programas en el entorno de la BD"* (UF2177, 80h) |
| Optimizar consultas lentas | **EXPLAIN ANALYZE**, índices | *"Optimización de consultas"* |
| Gestionar quién puede ver y modificar cada dato | Roles y permisos en PostgreSQL | *"Privilegios, autorizaciones"* |

### Por qué PostgreSQL y no otro

La ficha dice "bases de datos relacionales" sin especificar cuál. Nosotros
elegimos **PostgreSQL** porque:

- Es **gratuito** y de código abierto (no necesitan licencia)
- Es el que más se usa en startups y empresas tecnológicas
- Es potente: soporta JSON, búsqueda de texto, datos geográficos...
- Lo que aprendan en PostgreSQL se traduce fácilmente a cualquier otro
  (MySQL, SQL Server, Oracle)

Como herramienta gráfica usaremos **DBeaver** (gratuito, funciona con
cualquier base de datos) y como terminal **psql** (viene con PostgreSQL).

---

## FASE III -- Programación orientada a objetos (250 horas)

### Lo que dice la ficha oficial

> *Desarrollar componentes software en lenguajes de programación orientados
> a objetos.*

Traducido: programar aplicaciones de verdad. Esta es la fase más larga y
donde todo lo anterior se junta.

### Con qué lo van a aprender

| Lo que van a aprender | Herramienta que usaremos | Qué dice la ficha |
|------------------------|--------------------------|-------------------|
| Programar: variables, condicionales, bucles, funciones | **Python** en **Antigravity** (IDE de Google) | *"Principios de la programación orientada a objetos"* (UF2404, 90h) |
| POO: clases, objetos, herencia, polimorfismo | Python (soporta todo esto nativamente) | *"Clases, objetos, herencia, polimorfismo"* |
| Estructuras de datos: listas, diccionarios, conjuntos | Colecciones de Python | *"Listas, pilas, colas, árboles, tablas hash"* |
| Manejar errores sin que tu programa explote | try/except/finally en Python | *"Empleo de excepciones"* |
| Escribir tests: comprobar que tu código hace lo que debe | **pytest** | *"Pruebas unitarias, de caja blanca y caja negra"* (UF2406, 80h) |
| Construir una API web: que otros programas puedan hablar con el tuyo | **FastAPI** (framework Python) | *"Modelo de programación web"* (UF2405, 80h) |
| Conectar tu aplicación con la base de datos | **SQLAlchemy** (ORM) + PostgreSQL | *"Interfaz de programación de acceso a datos"* |
| La arquitectura web: cliente, servidor, base de datos | HTTP, REST, JSON | *"Arquitectura multicapa"* |
| Empaquetar tu aplicación para que funcione en cualquier equipo | **Docker** + **Docker Compose** | *"Ciclo de vida del desarrollo"* |
| Trabajar en equipo con metodología | **Agile/Scrum** básico | *"Control de calidad, seguimiento de requisitos"* |

### Por qué Python y no Java o C#

La ficha dice "lenguajes orientados a objetos" sin especificar cuál.
Nosotros elegimos **Python** porque:

- Es el **lenguaje más demandado** en ofertas de empleo en España y Europa
- La sintaxis es limpia: pueden centrarse en aprender a programar, no en
  memorizar punto-y-comas y llaves
- Cubre **todos** los conceptos de POO que pide la ficha: clases, herencia
  (simple y múltiple), polimorfismo, encapsulación, excepciones, hilos
- Tiene un ecosistema enorme: web, datos, automatización, IA
- Lo que aprendan de POO en Python se traduce directamente a Java, C#
  o cualquier otro lenguaje OO

### Nuestro IDE: Google Antigravity

No vamos a usar un editor cualquiera. Su entorno de desarrollo será
**Antigravity**, el IDE de Google basado en VS Code con agentes de IA
(Gemini) integrados.

Qué significa eso en la práctica:

- Tiene todo lo que tiene VS Code (terminal, depurador, extensiones, Git)
- Pero además incluye **agentes de IA** que les pueden ayudar a planificar
  código, explicar errores, generar tests y navegar proyectos grandes
- No es hacer trampa: es aprender a trabajar como se trabaja en 2026.
  Los programadores profesionales ya usan herramientas de IA a diario.
  Mejor que aprendan a usarlas bien desde el principio

El primer día de la Fase III seguirán la documentación oficial de Google
para aprender a manejar Antigravity paso a paso. Es un tutorial guiado
que les explica cada parte del IDE (no es una herramienta online: todo
se ejecuta en su equipo, en local).

Y sí, el nombre no es casualidad. En Python existe un easter egg famoso:
si escriben `import antigravity` en la consola, pasa algo inesperado.
Pruébenlo el primer día.

---

## El proyecto que los va a acompañar todo el curso

A lo largo de las 3 fases van a construir **Amtigravity Launch Control**:
una plataforma de gestión de lanzamientos espaciales. Piensen en un SpaceX
simplificado.

No es un ejercicio de juguete: es una aplicación completa con:

- Una base de datos PostgreSQL con cohetes, tripulaciones, misiones,
  bases de lanzamiento y cargas útiles
- Una API REST con FastAPI para consultar lanzamientos, registrar misiones
  y calcular ventanas de despegue
- Datos reales de la API pública de SpaceX para poblar la base de datos
- Tests automáticos para comprobar que todo funciona
- Despliegue con Docker para que cualquiera pueda ejecutarlo

El proyecto crece con ustedes:

| Fase | Qué añaden al proyecto |
|------|------------------------|
| Fase I | Crean el repositorio Git, el entorno Linux y la estructura de carpetas del proyecto |
| Fase II | Diseñan la base de datos (cohetes, misiones, tripulaciones, bases) y la llenan con datos reales de SpaceX |
| Fase III | Programan la API que consulta esos datos, registra nuevas misiones y calcula ventanas de lanzamiento |

Por qué este proyecto y no una "tienda online" o una "biblioteca":

- **Nadie más lo tiene** en su portfolio -- los diferencia en una entrevista
- **El dominio es rico**: hay fechas, coordenadas, estados (programado /
  en vuelo / completado / fallido), relaciones complejas entre tripulación
  y misiones
- **Los datos son reales**: SpaceX publica una API gratuita con todos sus
  lanzamientos históricos
- **Motiva más**: consultar "cohetes que han aterrizado con éxito" es más
  interesante que "libros prestados en marzo"

Al terminar, tendrán un proyecto real en su GitHub que pueden
enseñar en una entrevista de trabajo. Y no será igual que el de otros
500 candidatos.

---

## Resumen: la ficha oficial vs nuestras herramientas

| La ficha pide | Nosotros usamos |
|---------------|-----------------|
| Configurar sistemas operativos | **Linux** (Ubuntu) + **Docker** |
| Gestionar almacenamiento y archivos | Sistemas de archivos Linux + volúmenes Docker |
| Herramientas de control de cambios | **Git** + **GitHub** |
| Documentar | **Markdown** |
| Diseñar bases de datos | Diagramas E-R + **PostgreSQL** |
| SQL (consultas, definición, manipulación) | SQL real en **PostgreSQL** + **DBeaver** |
| Programar dentro de la base de datos | **PL/pgSQL** (funciones, triggers) |
| Lenguaje orientado a objetos | **Python** |
| Entorno de desarrollo | **Antigravity** (IDE de Google, basado en VS Code + IA) |
| Modelo de programación web | **FastAPI** + **REST** + **JSON** |
| Conexión aplicación-base de datos | **SQLAlchemy** |
| Testing y calidad | **pytest** + linters |
| Despliegue | **Docker Compose** |
| Metodología de desarrollo | **Agile/Scrum** |

Todo gratuito. Todo lo que se usa hoy en la industria. Todo cubierto por
lo que dice la ficha oficial.

---

## Una última cosa

Este curso es intensivo: 5 horas al día, 5 días a la semana, durante 6
meses. Va rápido. Pero cada día construyen encima de lo anterior, y al
final tendrán las herramientas para trabajar como programadores.

No les vamos a enseñar cosas que no vayan a usar. Cada hora de clase
está pensada para que el lunes que empiecen a trabajar, sepan lo que
están haciendo.

Si quieren ver la ficha oficial completa, está en la carpeta del curso:
`00_Administracion_Curso/IFCD0112_ficha.pdf`

---

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
