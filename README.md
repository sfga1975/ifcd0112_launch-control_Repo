# IFCD0112 — Launch Control

[![Licencia: CC BY-NC-SA 4.0](https://img.shields.io/badge/Licencia-CC%20BY--NC--SA%204.0-lightgrey.svg)](https://creativecommons.org/licenses/by-nc-sa/4.0/)
[![Certificado SEPE](https://img.shields.io/badge/SEPE-IFCD0112-blue.svg)]()
[![Centro](https://img.shields.io/badge/Centro-Amtigravity-orange.svg)]()

Material del alumno para el certificado de profesionalidad **IFCD0112 — Programacion con lenguajes orientados a objetos**.

## Datos del curso

| Campo | Valor |
|-------|-------|
| **Codigo** | IFCD0112 |
| **Nombre** | Programacion con lenguajes orientados a objetos y bases de datos relacionales |
| **Horas** | 710h (590h formativas + 120h practicas) |
| **Centro** | Amtigravity, Barcelona |
| **Convocatoria** | 2026 |
| **Proyecto transversal** | Launch Control |

## Stack tecnologico

| Herramienta | Uso |
|-------------|-----|
| Ubuntu Server 24.04 | Sistema operativo base |
| Docker + Compose | Contenedores y orquestacion |
| Git + GitHub | Control de versiones y colaboracion |
| PostgreSQL + pgAdmin | Bases de datos relacionales |
| Python 3 | Programacion orientada a objetos |
| VirtualBox | Virtualizacion local |

## Fases del curso

### Fase I — Fundamentos (MF0223_3) — 210h
Linux, Docker, Git y metodologias agiles.
Todo corre sobre Ubuntu Server en VirtualBox, sin interfaz grafica.

### Fase II — Bases de Datos (MF0226_3) — 210h
Diseno relacional, SQL desde cero hasta JOINs y subconsultas.
PostgreSQL desplegado en Docker, pgAdmin como cliente.

### Fase III — POO con Python (MF0227_3) — 170h
Programacion orientada a objetos, estructuras de datos, proyecto final.

## Proyecto Launch Control

**Launch Control** es el hilo conductor del curso. Los alumnos construyen paso a paso
un sistema de control de lanzamiento espacial que integra todas las tecnologias:

- **Fase I**: montan la infraestructura (servidores Linux, contenedores Docker, repositorios Git)
- **Fase II**: disenan y pueblan la base de datos (misiones, tripulacion, telemetria)
- **Fase III**: programan la logica de negocio en Python (simulacion, dashboard, API)

Cada sesion aporta una pieza funcional al proyecto. Al finalizar, el alumno tiene
un sistema completo desplegado en contenedores con base de datos y logica de negocio.

## Como usar este material

1. Clona el repositorio:
   ```bash
   git clone https://github.com/TodoEconometria/ifcd0112_launch-control_Repo.git
   ```

2. Navega a la fase y semana que te interese.

3. Cada archivo es una sesion completa con explicaciones, ejemplos y ejercicios.

## Licencia

Este material se distribuye bajo licencia [CC BY-NC-SA 4.0](LICENSE).
Puedes usarlo y adaptarlo para fines educativos, citando la fuente.

---

**Prof. Juan Marcelo Gutierrez Miranda**, 2026
