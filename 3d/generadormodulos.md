# Generador de módulos 3D — Reglas

Reglas para construir el objeto `MODELO` del visor 3D (`github-pages/3d/visor.html`).
Todo lo de aquí es para **mobiliario de cocina**. Se irá ampliando a medida que el usuario dicte más reglas.

URL fija del visor (no cambia, se reescribe siempre el mismo archivo):
`https://sazazel66-sys.github.io/despieces-open/3d/visor.html` → corto: `https://tinyurl.com/2dys55jc`

---

## 1. Sistema de coordenadas

- Unidades en **cm**.
- **X = anchura · Y = altura · Z = fondo.**
- Cada pieza:
  - `pos` = esquina **mínima** `[x, y, z]`.
  - `size` = `[ancho, alto, fondo]`.
- El visor centra el mueble en el origen y encuadra la cámara solo (no hay que calcular cámara).

## 2. Formato del objeto `MODELO`

```js
const MODELO = {
  nombre:  "Bajo cajones",                 // título arriba-izquierda
  nota:    "1 cajón pequeño (14) + 2 grandes (28) · 60 × 70 × 58 cm",
  color:   "#caa86b",                      // color tabla por defecto
  canto:   "#5a4a2e",                      // color de aristas
  medidas: [ ["Alto total","70 cm"], ... ],// panel lateral derecho (etiqueta → valor)
  piezas:  [ { nombre, pos:[x,y,z], size:[a,al,f], color? }, ... ]
};
```

- `color` por pieza es opcional; si falta usa `MODELO.color`.
- El panel de **medidas** se muestra siempre a la derecha; incluir todas las cotas importantes.

## 3. Aspecto / colores estándar

Estilo **vista sombreada tipo Rhino** en blancos y grises (no alámbrico, no madera):

| Elemento | Color |
|----------|-------|
| Tabla / carcasa / frentes / trasera | `#d6d8db` (gris claro uniforme) |
| Aristas (canto) | `#3a3d42` (gris oscuro fino) |
| Fondo | degradado vertical `#fdfdfe` (arriba) → `#c7ccd2` (abajo) |

- Material mate: `MeshStandardMaterial` `roughness 0.95`, `metalness 0`.
- Luces neutras y suaves (hemisférica + ambiente + 2 direccionales) para sombreado uniforme.
- Todas las piezas usan el mismo gris; **no** poner `color` por pieza salvo que se quiera resaltar algo. Las juntas (cajones/puertas) se ven por las aristas.
- Cámara por defecto: vista frontal-superior-derecha con los **frentes hacia el espectador** (frentes en `-Z`, cámara en `-Z`).

## 4. Carcasa (caja) estándar

Para un mueble de `ANCHO × ALTO × FONDO`, espesor de tablero **1,9 cm**:

| Pieza | pos | size |
|-------|-----|------|
| Costado Izq | `[0, 0, 0]` | `[1.9, ALTO, FONDO]` |
| Costado Der | `[ANCHO-1.9, 0, 0]` | `[1.9, ALTO, FONDO]` |
| Suelo | `[1.9, 0, 0]` | `[ANCHO-3.8, 1.9, FONDO]` |
| Techo | `[1.9, ALTO-1.9, 0]` | `[ANCHO-3.8, 1.9, FONDO]` |
| Trasera | `[1.9, 1.9, FONDO-T]` | `[ANCHO-3.8, ALTO-3.8, 0.5]` |

- Suelo y techo van **entre** los costados (de ahí `ANCHO-3.8`).
- **Trasera** encajada a `T` cm del fondo: **Bajos `T=5` · Altos `T=1.6`**. Espesor trasera `0.5`.

## 5. Frentes (puertas y cajones)

- Cerrados, **sobresalen 1,9 cm** por delante → `z` del frente = `-1.9`, espesor `1.9`.
- Holgura de **0,2 cm** por cada lado → ancho del frente = `ANCHO-0.4`, `pos.x = 0.2`.
- Apilados en Y cubriendo el alto del mueble (las aristas marcan las juntas).

## 6. Cajones — proporciones (cocina)

- Para un **bajo de cajones de 70 cm de alto**: frente pequeño = **14**, frentes grandes = **28**.
- Combinación por defecto: **1 pequeño (arriba) + 2 grandes (abajo)** → 14 + 28 + 28 = 70.
- **Escalado:** si el módulo es más alto, repartir la altura manteniendo la proporción **14 : 28 : 28 ≈ 1 : 2 : 2** (cada frente más alto en proporción).

Ejemplo (60 × 70 × 58):

```js
{ nombre:"Frente cajón grande inferior", pos:[0.2, 0,  -1.9], size:[59.6, 28, 1.9], color:"#d8b878" },
{ nombre:"Frente cajón grande medio",    pos:[0.2, 28, -1.9], size:[59.6, 28, 1.9], color:"#d8b878" },
{ nombre:"Frente cajón pequeño",         pos:[0.2, 56, -1.9], size:[59.6, 14, 1.9], color:"#d8b878" }
```

## 7. Pendiente (fase 2)

- Mecanizaciones (ranuras, taladros, cajeados, guías).
- Cajones abiertos / interiores de cajón.
- Auto-generar las imágenes PNG del catálogo desde el render.

---

## Flujo de publicación

1. Editar `MODELO` (y `medidas`) en `visor.html`.
2. `git add/commit/push` en `github-pages/` (rama `main`).
3. El enlace **no cambia**; compartir siempre la misma URL.
