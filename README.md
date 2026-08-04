# Aire delgado

Comparador interactivo de **34 carros usados y cero kilómetros** para comprar en Bogotá, en el rango de $38 a $72 millones.

Es un sitio estático de un solo archivo. No hay build, no hay dependencias que instalar, no hay backend. Todo el cálculo ocurre en el navegador y nada sale del equipo del visitante.

---

## Qué hace

- **Corrige la potencia por altura.** A 2.600 m un motor atmosférico pierde cerca del 26 % de su potencia, un turbo apenas un 6 % y un eléctrico nada. Todo el estudio parte de esa corrección.
- **Quince criterios ponderables** con deslizadores en vivo. Cuatro de ellos —combustible, taller y llantas, costos fijos y depreciación— no son juicio: salen de un motor de costos alimentado con tarifas vigentes de 2026.
- **Eje de nivel de equipamiento** que reconoce que el nombre del modelo no dice qué carro estás comprando.
- **Prueba de robustez**: corre la matriz con varias configuraciones a la vez y muestra qué modelos aguantan arriba sin importar lo que priorices.
- **Financiación y capacidad de compra**: cuota, inicial, intereses, múltiplo y el ingreso familiar que un banco te exigiría.
- **Configuraciones guardadas** en el navegador del visitante, exportables como código o archivo JSON.

---

## Estructura

```
matriz-carros-bogota/
├── index.html        el documento completo (HTML + CSS + JS en un archivo)
├── favicon.svg
├── netlify.toml      publicación, cabeceras de seguridad y caché
├── robots.txt
├── .gitignore
├── .gitattributes
├── README.md         esta guía, para personas
├── AGENTS.md         instrucciones maestras para agentes de IA
└── CLAUDE.md         puntero a AGENTS.md
```

---

## Despliegue

### Opción A — arrastrar y soltar (dos minutos, sin git)

1. Entra a [app.netlify.com/drop](https://app.netlify.com/drop).
2. Arrastra la carpeta completa `matriz-carros-bogota` a la ventana.
3. Listo. Netlify te da una URL del estilo `random-name-123.netlify.app`.

Sirve para probar. La desventaja es que cada actualización toca volver a arrastrar.

### Opción B — GitHub + Netlify (recomendada)

Con esto, cada `git push` republica el sitio solo.

**1. Poner la carpeta en su sitio**

Descomprime el proyecto en `C:\Dev\matriz-carros-bogota`.

**2. Inicializar el repositorio**

Abre PowerShell y ejecuta:

```powershell
cd C:\Dev\matriz-carros-bogota
git init
git add .
git commit -m "Matriz de carros para Bogota: version inicial"
git branch -M main
```

Si es la primera vez que usas git en este equipo, antes configura tu identidad:

```powershell
git config --global user.name "Tu Nombre"
git config --global user.email "tucorreo@ejemplo.com"
```

**3. Crear el repositorio en GitHub**

Crea uno vacío en [github.com/new](https://github.com/new) —sin README, sin .gitignore, sin licencia— y conéctalo:

```powershell
git remote add origin https://github.com/TU-USUARIO/matriz-carros-bogota.git
git push -u origin main
```

**4. Conectar Netlify**

1. En [app.netlify.com](https://app.netlify.com) → **Add new site** → **Import an existing project**.
2. Elige **GitHub** y autoriza el acceso.
3. Selecciona el repositorio.
4. Netlify lee `netlify.toml` y ya sabe qué hacer. Confirma que quede así:
   - **Build command:** vacío
   - **Publish directory:** `.`
5. **Deploy site**.

**5. Cambiar el nombre del sitio**

En **Site configuration → Site details → Change site name** puedes dejarlo en algo como `aire-delgado.netlify.app`.

### Actualizar después

```powershell
cd C:\Dev\matriz-carros-bogota
git add .
git commit -m "Actualizacion de precios y tasas"
git push
```

Netlify republica en menos de un minuto.

---

## Trabajar en local

Basta con abrir `index.html` en el navegador. Funciona directamente desde el disco.

Un detalle: algunos navegadores restringen el almacenamiento local en archivos abiertos con `file://`, y las configuraciones guardadas dejan de persistir. El documento lo detecta y avisa en amarillo. Si te molesta mientras editas, levanta un servidor local:

```powershell
# Con Python instalado
python -m http.server 8000
# Luego abre http://localhost:8000
```

En el sitio publicado no ocurre: el almacenamiento funciona normalmente sobre HTTPS.

---

## Mantenimiento

Las cifras que envejecen más rápido, y dónde tocarlas dentro de `index.html`:

| Dato | Dónde | Con qué frecuencia |
|---|---|---|
| Precio de gasolina y energía | Objeto `CFG` | Cada ajuste de la CREG |
| Tasas de crédito | Objeto `FIN` | Trimestral |
| SOAT | Constante `SOAT` | Anual, en enero |
| Umbral de impuesto y SMMLV | Constantes `SMMLV` y las tasas en `costs()` | Anual, en enero |
| Precios de carros | Campo `p` de cada entrada en `CARS` | Semestral |

Los tres primeros también son editables desde la interfaz sin tocar código, así que el visitante siempre puede corregirlos por su cuenta.

---

## Advertencia

Los puntajes son estimaciones expertas calibradas, no mediciones de banco ni datos de garantía de fabricante. Los costos son cifras de referencia para comparar entre opciones, no cotizaciones. Antes de comprar cualquier vehículo hay que verificar la versión exacta, el avalúo en SIBGA, el historial en el RUNT, la prima real de seguro y hacer un peritaje independiente.
