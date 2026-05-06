# GDD — Dios de Mundos Simulados
> Documento de diseño de juego (en progreso)

---

## Índice

1. [Concepto Central](#1-concepto-central)
2. [Metáfora y Tono](#2-metáfora-y-tono)
3. [Loop del Jugador](#3-loop-del-jugador)
4. [Los Tres Niveles de Actividad](#4-los-tres-niveles-de-actividad)
5. [Ciclo de un Run](#5-ciclo-de-un-run)
6. [Sistema de Recursos](#6-sistema-de-recursos)
7. [Árbol de Habilidades](#7-árbol-de-habilidades)
8. [Tensiones de Diseño](#8-tensiones-de-diseño)
9. [Gramática de Comandos](#9-gramática-de-comandos)
10. [Stack Tecnológico](#10-stack-tecnológico)
11. [Pendientes y Próximos Pasos](#11-pendientes-y-próximos-pasos)

---

## 1. Concepto Central

El jugador es una **entidad divina** que crea, observa y manipula **simulaciones de civilizaciones** a través de comandos de texto. Cada simulación es un *run*. Al terminar un run (por colapso, extracción o ascensión), el jugador obtiene **Datos Primordiales** — el recurso meta que persiste entre simulaciones y permite mejoras permanentes.

El juego es **text-based puro**: sin gráficos, sin UI visual. La interfaz es una terminal. Las acciones del jugador son comandos con verbos, targets y flags.

**Referente principal:** Bitburner (loop de automatización, escalada exponencial, scripting como mecánica central).

---

## 2. Metáfora y Tono

- El jugador nunca ve "el mundo" directamente — lo lee a través de logs, métricas y reportes.
- El tono es frío, analítico, casi científico. La narrativa emerge de los datos.
- La tensión central es: **¿cuánto intervenís?** Intervenir tiene costos. No intervenir tiene riesgos. Aprender cuándo hacer qué *es* el juego.
- El mundo corre solo entre comandos. El jugador reacciona, no controla.

---

## 3. Loop del Jugador

```
Configurar simulación
        ↓
Observar el mundo (scan, inspect, log)
        ↓
Decidir si intervenir
        ↓
Ejecutar comandos (event, influence, tech...)
        ↓
Observar consecuencias (lineales y no lineales)
        ↓
Repetir hasta punto crítico
        ↓
Decidir: extraer ahora o seguir escalando
        ↓
Fin del run → obtener Datos Primordiales
        ↓
Mejorar árbol de habilidades
        ↓
Nueva simulación (con más capacidades)
```

El gancho de progresión sigue el patrón de Bitburner: **tedio → automatización → nuevo tedio de mayor orden → mejor automatización**. Con el tiempo, el jugador puede scriptear secuencias de comandos para automatizar intervenciones rutinarias.

---

## 4. Los Tres Niveles de Actividad

### Nivel 1 — Observación y Diagnóstico
Lo que el jugador hace constantemente, de forma casi pasiva. El mundo corre solo; el jugador lee y evalúa.

```bash
> scan --region all
> inspect --civilization humanos --metric stability
> log --last 10
```

El mundo genera eventos automáticamente: guerras, hambrunas, descubrimientos. El jugador decide si intervenir o dejar correr. **Saber leer la situación antes de actuar es una habilidad central.**

### Nivel 2 — Intervención Directa
Las acciones concretas que modifican el estado del mundo. Acá está el grueso de la toma de decisiones.

```bash
> event --type rain --region sur --duration 3        # evitar una hambruna
> influence --leader "Kael" --trait ambition +2      # empujar una conquista
> tech --unlock agriculture --civ humanos            # acelerar progreso tecnológico
> plague --severity low --region norte               # reducir superpoblación
```

Cada acción:
- Consume **Energía Divina** (recurso limitado y de regeneración lenta)
- Tiene **consecuencias no lineales** (dar lluvia al sur puede causar inundaciones; impulsar a un líder puede desatar una guerra inesperada)

### Nivel 3 — Arquitectura de la Simulación
El nivel más estratégico. Permite configurar condiciones base del mundo antes o durante la simulación. **Desbloqueado progresivamente a través del árbol de habilidades.**

```bash
> config --param fertility --value 0.6
> spawn --faction mercaderes --region centro
> set --law herencia --civ humanos --type primogenitura
```

La diferencia entre el Nivel 2 y el Nivel 3 es la diferencia entre *reaccionar al mundo* y *diseñarlo*.

---

## 5. Ciclo de un Run

### Inicio
- El jugador configura parámetros básicos (limitados por lo que tiene desbloqueado en el árbol)
- Se spawnea una o más civilizaciones iniciales

### Desarrollo (~70% del run)
- El jugador observa, interviene y gestiona recursos
- La civilización progresa a través de **eras**: Tribal → Feudal → Industrial → [eras superiores por definir]
- Cada era introduce eventos más complejos y costosos de gestionar

### Punto Crítico
- La civilización alcanza un umbral (tecnológico, poblacional, filosófico — a definir con precisión)
- El jugador enfrenta una decisión: **extraer ahora** (recompensa segura pero menor) o **seguir escalando** (mayor recompensa, mayor riesgo de colapso)

### Fin del Run — Tres Desenlaces

| Desenlace | Condición | Recompensa |
|---|---|---|
| **Ascensión** | La civilización trasciende por su propia evolución | Máxima — rara y difícil |
| **Extracción** | El jugador termina el run manualmente | Balanceada — la más común |
| **Colapso** | La civilización se destruye | Mínima — pero el jugador aprende qué falló |

---

## 6. Sistema de Recursos

### 6.1 Recursos dentro del run (se resetean al terminar)

| Recurso | Qué representa | Notas |
|---|---|---|
| **Energía Divina** | Capacidad de intervención del jugador | Se regenera lentamente; limita el micromanagement |
| **Influencia** | Cuánto "escucha" el mundo al jugador | Se desgasta si se interviene de forma torpe o repetida |
| **Cohesión Civilizatoria** | Estabilidad interna de la civilización | Baja con guerras, plagas y cambios bruscos |
| **Tiempo de Simulación** | El reloj del run | Recurso pasivo — cada era cuesta más energía gestionar |

La **Energía Divina** siendo limitada es la restricción de diseño más importante: el jugador no puede microgestionar todo y tiene que elegir dónde poner el foco.

### 6.2 Recursos entre runs — Datos Primordiales (persisten)

Los Datos Primordiales se subdividen según el desenlace del run. Esto incentiva **variar la estrategia** en lugar de repetir siempre el mismo patrón.

| Tipo | Cómo se obtiene | Para qué sirve |
|---|---|---|
| **Datos de Colapso** | Run terminado en destrucción | Mejoras defensivas, comandos de estabilización |
| **Datos de Extracción** | Run terminado por el jugador | Mejoras de eficiencia, reducción de costos de comandos |
| **Datos de Ascensión** | Run terminado en trascendencia | Las mejoras más poderosas y raras del árbol |

---

## 7. Árbol de Habilidades

Tres ramas principales, cada una con nodos progresivos. Cada nodo tiene además **niveles de mejora** (no solo se desbloquea, sino que se profundiza).

### Rama: Percepción
Mejora la capacidad del jugador de leer y anticipar el mundo.

```
Ver métricas básicas
    └─ Ver relaciones entre facciones
        └─ Predecir eventos (con probabilidades)
            └─ Ver líneas de tiempo alternativas
```

*Ejemplo de niveles en un nodo:*
- Nivel 1: `inspect` devuelve valores actuales
- Nivel 2: `inspect` devuelve tendencia de los últimos N ticks
- Nivel 3: `inspect` devuelve proyección a futuro con % de confianza

### Rama: Intervención
Amplía el vocabulario de comandos disponibles.

```
Eventos climáticos simples
    └─ Influenciar líderes
        └─ Manipular tecnología
            └─ Reescribir leyes y cultura
                └─ Crear y destruir facciones
```

*Ejemplo de niveles en un nodo (`Eventos climáticos`):*
- Nivel 1: lluvia, sequía
- Nivel 2: tormentas, heladas, olas de calor
- Nivel 3: huracanes dirigidos, corrientes oceánicas, ciclos de glaciación

### Rama: Arquitectura
La rama más poderosa. **Solo se desbloquea con Datos de Ascensión.**

```
Configurar parámetros iniciales del mundo
    └─ Gestionar múltiples civilizaciones simultáneas
        └─ Simular dentro de una simulación (meta-nivel)
```

---

## 8. Tensiones de Diseño

El núcleo del diseño está en que **intervenir tenga consecuencias reales**. Sin esto, el juego se convierte en un panel de control sin interés. Tres mecanismos propuestos:

### 8.1 Dependencia
Si el jugador siempre resuelve los problemas de la civilización (lluvia en cada sequía, alimento en cada hambruna), la civilización nunca desarrolla sus propias soluciones. Se vuelve más estable a corto plazo pero más frágil estructuralmente. El jugador crea civilizaciones dependientes que son más difíciles de llevar a la Ascensión.

### 8.2 Consciencia Divina
Si el jugador interviene de forma demasiado visible o frecuente, la civilización empieza a *percibir* que existe algo superior. Esto tiene consecuencias:
- **Adoración:** baja la Cohesión por fanatismo, altera el desarrollo tecnológico
- **Temor:** cambia la estructura política y cultural de la civilización
- **Negación activa:** facciones que buscan "matar a los dioses" — evento de alto riesgo

### 8.3 Entropía
El mundo tiene una tendencia natural al caos que **aumenta con el tiempo y con la complejidad de la civilización**. Las intervenciones del jugador frenan la entropía localmente, pero nunca la eliminan globalmente. En las eras avanzadas, el jugador se enfrenta a una civilización que genera problemas más rápido de lo que puede resolverlos — esto fuerza la decisión de extraer o arriesgarse al colapso.

---

## 9. Gramática de Comandos

*(Sección a expandir — estructura base definida)*

La gramática sigue el patrón: `verbo --target [valor] --flag [valor]`

### Verbos base (disponibles desde el inicio)

| Verbo | Descripción |
|---|---|
| `scan` | Vista general de una o todas las regiones |
| `inspect` | Métricas detalladas de una civilización, facción o líder |
| `log` | Historial de eventos de la simulación |
| `event` | Disparar un evento en el mundo |
| `influence` | Modificar atributos de un líder o facción |
| `extract` | Terminar el run manualmente y cobrar recompensa |

### Verbos desbloqueables (ejemplos)

| Verbo | Rama | Nivel aprox. |
|---|---|---|
| `tech` | Intervención | Medio |
| `plague` | Intervención | Medio |
| `spawn` | Arquitectura | Alto |
| `config` | Arquitectura | Alto |
| `set --law` | Intervención | Avanzado |
| `simulate` | Arquitectura | Máximo |

### Flags comunes

| Flag | Uso |
|---|---|
| `--region` | Target geográfico |
| `--civ` | Target de civilización |
| `--severity` | Intensidad del efecto (low / mid / high) |
| `--duration` | Duración en ticks |
| `--trait` | Atributo a modificar (usado con `influence`) |

---

## 10. Stack Tecnológico

*(Decisión pendiente — opciones evaluadas)*

| Opción | Pros | Contras |
|---|---|---|
| **Browser + JS puro** | Accesible sin instalación; el lenguaje de scripting del juego puede ser JS real | Menos "nativo" como experiencia de terminal |
| **React + xterm.js** | Terminal real embebida, más control de UI | Más overhead de setup |
| **Node.js + blessed** | Terminal nativa, experiencia cruda | Distribución más compleja |
| **Python + Textual** | Framework TUI moderno y expresivo | Ecosistema distinto al browser |
| **Rust + Ratatui** | Alto rendimiento, binario standalone | Curva de aprendizaje alta |

**Recomendación tentativa:** Browser + JS (o React + xterm.js) para mantener la posibilidad de que el lenguaje de scripting del jugador sea JS real — el mismo patrón que hace a Bitburner tan elegante.

---

## 11. Pendientes y Próximos Pasos

- [ ] Definir nombre del juego y de la entidad que es el jugador
- [ ] Diseñar la gramática de comandos completa (todos los verbos, flags y costos)
- [ ] Definir las eras de civilización y sus umbrales de transición
- [ ] Definir el sistema de entropía con más precisión (¿es un número? ¿una curva?)
- [ ] Diseñar el árbol de habilidades completo con costos en Datos Primordiales
- [ ] Escribir 2-3 "escenas de texto" para testear la voz narrativa del juego
- [ ] Decidir stack tecnológico
- [ ] Definir el sistema de scripting/automatización del jugador

---

*GDD generado como punto de partida. Documento vivo — actualizar a medida que el diseño evoluciona.*
