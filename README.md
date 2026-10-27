# Flappy Bird · Terminal + Gestor

Un solo proyecto con dos partes que comparten datos:

- **Juego** de Flappy Bird que se juega en la terminal (con `curses`).
- **Gestor** de jugadores y partidas: medallas, monedas, niveles, skins, reportes, JSON, CSV/TXT y bitácora.

Lo que ganas mientras estas jugando (puntaje, monedas, nivel, récord) aparece en el gestor, y las skins que compras en el gestor se ven en el juego.

## Cómo ejecutar

```bash
python main.py                                  # aplicación completa (menú principal)
python -m unittest discover -s tests -v         # pruebas
python -c "from gestor.interfaz import ejecutar; ejecutar()"   # solo el gestor, sin juego
```

Ejecuta los comandos desde esta carpeta. (Las rutas de datos ya no dependen de ella, pero los imports sí.)

**Windows:** `curses` no viene incluido. Instálalo una sola vez:

```bash
pip install -r requirements.txt
```

Si no lo instalas, el juego avisa con un mensaje claro y el gestor sigue funcionando. La terminal debe medir al menos **48 columnas × 24 filas**.

**Gráficos:** el juego usa colores si la terminal tiene al menos 8, y bloques Unicode (`█ ▓ ● ✖`) si la terminal usa UTF-8. Si algo se ve raro (cuadros con `?`), fuerza el modo simple con `FLAPPY_ASCII=1` (en PowerShell: `$env:FLAPPY_ASCII=1`; en CMD: `set FLAPPY_ASCII=1`).

## Controles del juego

| Tecla | Acción |
|---|---|
| `ESPACIO`, `W` o flecha ↑ | Saltar (y comenzar) |
| `P` | Pausa / continuar |
| `Q` o `Esc` | Abandonar (la partida no se registra) |

Qué se ve en pantalla: cielo con nubes, colinas, tuberías con tapa, suelo que avanza, pájaro con ala que aletea, encabezado con puntaje y **MEJOR**, destello al chocar y una pantalla final con el puntaje en número grande, medalla y monedas.

| Skin | Unicode | ASCII |
|---|---|---|
| clásico | `●` amarillo | `O` |
| rojo | `●` rojo | `R` |
| dorado | `◆` amarillo | `$` |
| fantasma | `○` blanco | `~` |

Al chocar el pájaro se convierte en `✖` (`X` en ASCII).

## Estructura

```text
flappy_gestor/
├── main.py              ← punto de entrada
├── aplicacion.py        ← menú principal: une juego y gestor
├── requirements.txt
├── gestor/              ← lógica de negocio y datos
│   ├── config.py        ← rutas con pathlib
│   ├── dificultad.py    ← Dificultad + Facil/Normal/Dificil (herencia, polimorfismo)
│   ├── reglas.py        ← medallas, niveles, categoría, skins
│   ├── validadores.py   ← validación de entradas
│   ├── modelos.py       ← Jugador (propiedades) y Partida (dataclass)
│   ├── repositorio.py   ← colecciones privadas + vistas de solo lectura
│   ├── servicio.py      ← casos de uso
│   ├── almacenamiento.py← JSON: try/except/else/finally
│   ├── exportador.py    ← Exportador, ExportadorCSV, ExportadorTXT
│   ├── bitacora.py      ← archivo de texto .txt
│   ├── reportes.py      ← estadísticas (usa conjuntos)
│   ├── excepciones.py   ← errores propios
│   └── interfaz.py      ← menú del gestor
├── juego/               ← el juego (patrón modelo / vista / controlador)
│   ├── modelo.py        ← Pajaro, Obstaculo, Tuberia, TuberiaMovil, Juego
│   ├── vista.py         ← escena pura (celdas) + colores/temas con curses
│   └── controlador.py   ← teclas + bucle principal
├── tests/               ← 84 pruebas
├── data/                ← flappy.json y bitacora.txt (se crean solos)
├── exports/             ← partidas/ranking en .csv y .txt
└── legado/Flappy_bird_pygame.py   ← el prototipo original con pygame (referencia)
```

Regla de dependencias: `main` → `aplicacion` → (`juego`, `gestor`). El juego usa `gestor.dificultad`, `gestor.excepciones` y `gestor.reglas`; el gestor **no** conoce al juego.

Variables de entorno opcionales: `FLAPPY_DATOS=/otra/carpeta` cambia dónde se guardan `data/` y `exports/`; `FLAPPY_ASCII=1` / `FLAPPY_UNICODE=1` fuerzan el tema de caracteres del juego.

## Mapa de temas de los laboratorios

### 01 · Primeros pasos con Python

| Tema | Dónde verlo |
|---|---|
| Variables, tipos, constantes por convención | `gestor/reglas.py` (`PUNTAJE_MAX`, `SKINS`), `juego/modelo.py` (`GRAVEDAD`, `ANCHO`) |
| Operadores y expresiones | `reglas.py` (`//`, `*`), `modelos.py` (`+=`), `reportes.py` (`/`) |
| Conversiones | `validadores.validar_entero` (`int`), `juego/modelo.py` (`float`, `int`, `round`), `aplicacion.py` (`int`) |
| `input()`, `print()`, f-strings | `interfaz.pedir`, `aplicacion.opcion_jugar` |

### 02 · Decisiones, ciclos y colecciones

| Tema | Dónde verlo |
|---|---|
| Comparaciones y operadores lógicos (`and`, `or`, `not`) | `reglas.categoria_para` |
| `if` / `elif` / `else` | `reglas.categoria_para`, `controlador.accion_para`, `aplicacion.elegir_jugador` |
| `for` y `while` | `reglas.medalla_para`; bucles del menú y de `controlador._partida` |
| Listas, tuplas, diccionarios | `_MEDALLAS` (lista de tuplas), `SKINS`, `Jugador.skins` (tupla) |
| **Conjuntos** | `repositorio.verificar_integridad`, `reportes.jugadores_sin_partidas` |
| Estructuras anidadas | El JSON (dict con listas de dicts), `reportes.resumen_por_dificultad` |
| Comprensiones (lista, dict, **set**) | `almacenamiento.cargar`, `reportes.resumen_por_dificultad`, `repositorio.verificar_integridad` |

### 03 · Funciones, módulos y POO

| Tema | Dónde verlo |
|---|---|
| Funciones, parámetros, retorno | `reglas.py`, `validadores.py` (valores por defecto en `reportes.ranking`) |
| **Alcance de variables** (`global`, local) | `interfaz._contar_accion` y `_acciones_en_sesion` |
| Módulos e importaciones | Paquetes `gestor` y `juego` |
| Clases, objetos, `__init__`, `self` | `Jugador`, `Repositorio`, `Servicio`, `Juego`, `Pajaro` |
| **Propiedades** (`@property`, `setter`) | `Jugador.monedas` (valida), `Jugador.nivel` (calculada), `Servicio.repo` |
| **Encapsulamiento** | `Jugador._monedas`, `Repositorio._jugadores` + vistas de solo lectura |
| Colaboración entre clases | `Servicio` ↔ `Repositorio` ↔ `Jugador`; `Juego` ↔ `Pajaro`/`Tuberia` |

### 04 · Excepciones, archivos, persistencia y POO

| Tema | Dónde verlo |
|---|---|
| Validaciones y `raise` | `validadores.py`, `Jugador.monedas` (setter) |
| `try` / `except` | Casi todos los módulos |
| **`else`** | `almacenamiento.cargar`, `bitacora.registrar` |
| **`finally`** | `almacenamiento.guardar` (borra el `.tmp` siempre) |
| **Archivos de texto** | `bitacora.py` (modo `"a"`), `ExportadorTXT` |
| **`pathlib`** | `config.py`, `almacenamiento.py`, `bitacora.py`, `exportador.py` |
| Persistencia JSON | `almacenamiento.py` (escritura atómica + respaldo `.corrupto`) |
| Exportación CSV | `ExportadorCSV` (UTF-8 con BOM para Excel) |
| Herencia | `GestorError` y sus hijas; `Facil/Normal/Dificil`; `Tuberia → TuberiaMovil`; `ExportadorCSV/TXT` |
| **Sobrescritura** | `Dificultad.hueco()` en cada subclase, `Tuberia.bloquea`, `TuberiaMovil.actualizar` (con `super()`), `Jugador.__str__` |
| **Polimorfismo** | `interfaz.op_exportar` (mismo método, dos formatos); `Juego.tick` (cada obstáculo se actualiza a su modo); `reglas.monedas_para` |
| Composición | `Servicio` tiene un `Repositorio` y una `Bitacora`; `Juego` tiene un `Pajaro` y obstáculos |

## Reglas del juego (resumen)

- Cada tubería superada = 1 punto. Medalla: bronce ≥ 10, plata ≥ 20, oro ≥ 40, platino ≥ 100.
- Monedas = `(puntaje // 5) × multiplicador` (fácil 1, normal 2, difícil 3).
- Nivel por monedas acumuladas: 30 → 2, 100 → 3, 250 → 4, 500 → 5.
- Skins: rojo 50, dorado 150, fantasma 300 monedas.
- Dificultad **difícil**: hueco más pequeño, más velocidad y tuberías que suben y bajan.

## Distribución de responsabilidades del equipo

| Integrante | Responsabilidad técnica |
|---|---|
| **Diana Reyes** | Elaboración de historias de usuario, identificación de requerimientos funcionales y definición de criterios de aceptación. |
| **Camilo Zárate** | Documentación de historias de usuario y análisis de las necesidades y funcionalidades del sistema. |
| **Yohan Cuellar** | Desarrollo e implementación del software en Python, programación de funcionalidades y lógica del juego. |
| **Jader Posso** | Implementación del gestor de jugadores y partidas, integración de módulos y persistencia de datos. |

**Proyecto:** Flappy Bird · Terminal + Gestor

**Tecnologías:** Python, curses, JSON, CSV y programación orientada a objetos.
