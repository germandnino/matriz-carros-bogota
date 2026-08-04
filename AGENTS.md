# AGENTS.md

Archivo maestro de instrucciones para agentes de IA que trabajen en este repositorio.
Cualquier otro archivo de instrucciones (`CLAUDE.md`, `.cursorrules`, `.github/copilot-instructions.md`) debe apuntar aquí, no duplicar contenido.

---

## 1. Qué es este proyecto

**Aire delgado** — comparador interactivo de 34 carros usados y cero kilómetros para comprar en Bogotá, en el rango de $38 a $72 millones. Corrige la potencia por la altura de 2.600 m, pondera 15 criterios con deslizadores en vivo y calcula costos mensuales reales con tarifas de 2026.

Sitio estático **de un solo archivo**. Sin build, sin dependencias, sin backend, sin telemetría. Todo el cálculo ocurre en el navegador del visitante.

---

## 2. Reglas duras

Estas no se negocian sin autorización explícita del usuario.

1. **Un solo archivo.** Todo el HTML, CSS y JavaScript vive en `index.html`. No dividir en `app.js`, `styles.css` ni módulos. Si una tarea parece exigirlo, pregunta primero.
2. **Cero dependencias en runtime.** No agregar CDN, no agregar `<script src>` externo, no agregar fuentes remotas, no agregar `fetch()` a terceros. El sitio debe funcionar completo abierto con `file://`.
3. **Cero build.** No introducir `package.json`, bundler, transpilador ni framework. `netlify.toml` declara `command = ""` a propósito.
4. **Nada sale del equipo del visitante.** Sin analytics, sin cookies, sin llamadas de red. La única persistencia permitida es `localStorage`.
5. **JavaScript de navegador nativo.** ES2020+ sin transpilar. Sin JSX, sin TypeScript, sin imports.
6. **Español con ortografía completa.** Todo el texto visible, los comentarios y los mensajes de commit van en español con tildes, eñes y signos de apertura (`¿` `¡`). Nunca sustituir por ASCII.
7. **No inventar cifras.** Precios, tasas, consumos y puntajes son calibraciones deliberadas. Si una cifra debe cambiar, el usuario la aporta o autoriza la fuente. Ver §6.

---

## 3. Estructura del repositorio

```
matriz-carros-bogota/
├── index.html        documento completo — HTML + CSS + JS
├── favicon.svg
├── netlify.toml      publicación, cabeceras de seguridad y caché
├── robots.txt
├── .gitignore
├── .gitattributes    fuerza LF en todo el texto
├── README.md         documentación para personas
├── AGENTS.md         este archivo — maestro para agentes
└── CLAUDE.md         puntero a AGENTS.md
```

---

## 4. Mapa de `index.html`

El archivo tiene tres bloques. Los números de línea son aproximados y se corren al editar; usa los nombres de símbolo para ubicarte, no las líneas.

| Rango | Contenido |
|---|---|
| 1–19 | `<head>`, metadatos, favicon inline |
| 20–379 | `<style>` — todo el CSS, variables en `:root` |
| 381–624 | Markup del `<body>`: nav, nueve secciones `<h2>`, footer |
| 626–1780 | `<script>` — bloque único |

### Secciones de la página (los `<h2>`)

1. Los pesos son tuyos — deslizadores de los 15 criterios
2. Tabla general — ranking principal
3. Prueba de robustez — corre la matriz con varias configuraciones
4. Criterio por criterio — desglose por categoría
5. La cuenta mensual — motor de costos
6. La corrección que nadie aplica — altura 2.600 m
7. El nombre del modelo miente — eje de equipamiento
8. Nuevo contra usado
9. Cómo leer esto

### Datos (no lógica) — lo que se edita al actualizar cifras

| Símbolo | Qué es |
|---|---|
| `CATS` | Los 15 criterios: clave, nombre, peso por defecto, textos `hint`/`why` |
| `MONEY` | Cuáles criterios salen del motor de costos y no de juicio experto |
| `CARS` | Los 34 modelos. Un objeto por carro con ficha, puntajes `s`, `bad`, `good` |
| `TRIMS` | Los tres niveles de equipamiento y cómo modifican puntajes y rines |
| `PTS` | Categorías de pico y placa |
| `PRESETS` | Configuraciones de pesos predefinidas |
| `CFG` | Tarifas editables por el visitante: km/mes, gasolina, kWh |
| `FIN` | Parámetros de crédito: inicial, plazo, tasas |
| `ING` | Parámetros de capacidad de compra |

### Constantes de calibración

| Símbolo | Qué es |
|---|---|
| `DERATE` | Pérdida de potencia por altura según aspiración: NA .74, Turbo .94, MHEV .78, EV 1 |
| `RIN` | Costo de un juego de llantas por diámetro de rin |
| `SOAT` | Tarifas SOAT por clase de vehículo y cilindraje |
| `RTM`, `VIDA_LLANTA` | Revisión técnico-mecánica y km de vida de llanta |
| `SMMLV`, `PISO` | Salario mínimo y piso de ingreso |

### Motor de cálculo

| Función | Qué hace |
|---|---|
| `adj(c)` | Aplica el nivel de equipamiento a un carro: ajusta puntajes y rin |
| `costs(c,a)` | Costo mensual: combustible, taller, llantas, SOAT, impuesto, RTM, seguro, depreciación |
| `build()` | Arma las filas y normaliza los criterios monetarios a escala 0–100 |
| `score(s,w)` | Promedio ponderado |
| `ranked(w)` | `build()` + `score()` + orden descendente |
| `robustness()` | Corre la matriz con varias configuraciones y mide consenso y estabilidad |
| `credito(c,co)` | Cuota, intereses y múltiplo del crédito |

### Render y estado

`render()` es el orquestador. Las demás `renderX()` pintan su sección: `renderWeights`, `renderPT`, `renderContext`, `renderRobust`, `renderCosts`, `renderFin`, `renderIng`, `renderCat`, `renderSaved`, `dossier`.

Estado global mutable: `W` (pesos actuales), `BASE` (pesos por defecto), `trim`, `filter`, `PT`, `CFG`, `FIN`, `ING`, `SEL`, `SAVED`.

Persistencia: `localStorage` bajo las claves `matrizCarrosBogota.v1` (configuraciones guardadas, constante `STORE`) y `matrizCarrosBogota.ui` (estado de la interfaz, constante `UIKEY`). Las configuraciones se exportan como base64 vía `enc()`/`dec()` y como JSON.

---

## 5. Convenciones de código

- **CSS**: variables en `:root`, nombres de clase cortos en español o abreviados (`.filt`, `.nav`, `.card`). No hay preprocesador ni utilidades tipo Tailwind.
- **JS**: estilo compacto deliberado — `const` de una línea, arrow functions, template literals. Respétalo; no lo "mejores" reformateando.
- **HTML generado**: se construye con template literals y se inyecta con `innerHTML`. **Todo texto de origen dinámico pasa por `esc()`.** No romper esto.
- **Accesibilidad**: la nav y los botones de filtro usan `aria-pressed` y `aria-label`. Al agregar controles, mantén el patrón.
- **Moneda**: usa `cop()` para pesos y `money()` para rangos de precio en millones. Formato `es-CO`.
- **Comentarios**: escasos y solo donde el porqué no es obvio. Igualar la densidad existente.

---

## 6. Mantenimiento de cifras

Datos que envejecen y dónde tocarlos:

| Dato | Dónde | Frecuencia |
|---|---|---|
| Gasolina corriente, extra, kWh | `CFG` | Cada ajuste de la CREG |
| Tasas de crédito | `FIN` | Trimestral |
| SOAT | `SOAT` | Anual, en enero |
| SMMLV y umbral de impuesto | `SMMLV` y las tasas dentro de `costs()` | Anual, en enero |
| Precios de carros | Campo `p` de cada entrada de `CARS` | Semestral |
| Costo de llantas | `RIN` | Anual |

Los tres primeros también son editables desde la interfaz, así que el visitante puede corregirlos sin tocar código.

**Regla:** al cambiar una cifra estructural, revisa si el texto narrativo de la sección correspondiente sigue siendo cierto. Varios párrafos citan números concretos.

---

## 7. Cómo verificar un cambio

No hay suite de tests. La verificación es manual y obligatoria antes de declarar un cambio terminado:

```bash
python -m http.server 8000
```

Luego abre `http://localhost:8000` y confirma:

1. La consola del navegador no muestra errores.
2. La tabla general renderiza los 34 modelos y reordena al mover un deslizador.
3. La prueba de robustez corre sin excepción.
4. La cuenta mensual muestra cifras plausibles, no `NaN` ni `Infinity`.
5. Guardar, exportar e importar una configuración funciona ida y vuelta.
6. Los tres niveles de equipamiento cambian la tabla.

`file://` también funciona, pero `localStorage` puede quedar bloqueado y el documento lo avisa en amarillo. Para probar persistencia, usa el servidor local.

**No afirmes que algo funciona sin haberlo abierto.**

---

## 8. Git y despliegue

- Rama principal: `main`.
- Mensajes de commit en español, imperativo, sin prefijos convencionales obligatorios. Ejemplo: `Actualiza tarifas de SOAT 2026`.
- `.gitattributes` fuerza LF. No pelear con eso desde Windows.
- Netlify publica desde `main` en cada push. `netlify.toml` ya define publish `.`, command vacío, cabeceras de seguridad y un redirect SPA `/*` → `/index.html` con status 200.
- No hay variables de entorno ni secretos. Si alguna tarea parece necesitarlos, algo se salió del diseño.

---

## 9. Alcance y advertencia

Los puntajes son estimaciones expertas calibradas, no mediciones de banco ni datos de garantía de fabricante. Los costos son cifras de referencia para comparar entre opciones, no cotizaciones. El pie de página y la sección "Cómo leer esto" dicen esto explícitamente: **no lo suavices ni lo elimines.**
