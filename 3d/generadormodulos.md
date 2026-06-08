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

## 4. Espesores de tablero

Se leen del **material**: del material de **cascos** para la carcasa y del material de **puertas** para los frentes. Por defecto:

- **Casco (carcasa): `EC = 1,6 cm`**
- **Puertas/frentes: `EP = 1,9 cm`**

(usar estos defaults mientras no haya material asignado).

## 5. Carcasa (caja) estándar

Para un mueble de `ANCHO × ALTO × FONDO`, espesor de casco `EC` (1,6 por defecto):

| Pieza | pos | size |
|-------|-----|------|
| Costado Izq | `[0, 0, 0]` | `[EC, ALTO, FONDO]` |
| Costado Der | `[ANCHO-EC, 0, 0]` | `[EC, ALTO, FONDO]` |
| Suelo | `[EC, 0, 0]` | `[ANCHO-2·EC, EC, FONDO]` |
| Techo | `[EC, ALTO-EC, 0]` | `[ANCHO-2·EC, EC, FONDO]` |
| Trasera | `[EC, EC, FONDO-T]` | `[ANCHO-2·EC, ALTO-2·EC, 0.5]` |

- Suelo y techo van **entre** los costados (de ahí `ANCHO-2·EC`).
- **Trasera** encajada a `T` cm del fondo: **Bajos `T=5` · Altos `T=1.6`**. Espesor trasera `0.5`.

## 6. Frentes (puertas y cajones)

- Espesor `EP` (1,9 por defecto). Cerrados, **sobresalen** por delante → `z` del frente = `-EP`, fondo del box = `EP`.
- Holgura de **0,2 cm** por cada lado → ancho del frente = `ANCHO-0.4`, `pos.x = 0.2`.
- Apilados en Y cubriendo el alto del mueble (las aristas marcan las juntas).

## 7. Cajones — proporciones (cocina)

- Para un **bajo de cajones de 70 cm de alto**: frente pequeño = **14**, frentes grandes = **28**.
- Combinación por defecto: **1 pequeño (arriba) + 2 grandes (abajo)** → 14 + 28 + 28 = 70.
- **Escalado:** si el módulo es más alto, repartir la altura manteniendo la proporción **14 : 28 : 28 ≈ 1 : 2 : 2** (cada frente más alto en proporción).

Ejemplo (60 × 70 × 58):

```js
{ nombre:"Frente cajón grande inferior", pos:[0.2, 0,  -1.9], size:[59.6, 28, 1.9] },
{ nombre:"Frente cajón grande medio",    pos:[0.2, 28, -1.9], size:[59.6, 28, 1.9] },
{ nombre:"Frente cajón pequeño",         pos:[0.2, 56, -1.9], size:[59.6, 14, 1.9] }
```

## 8. Tiradores (asas)

- Tipo **asa**: barra horizontal adelantada de la cara del frente sobre dos patas.
- **Entre centros (distancia entre agujeros) = 12,5 cm** por defecto.
- Centrados en X del mueble; un tirador por frente, a la altura central del frente.
- Generados con `tiradorAsa(centroY, entreCentros)` → devuelve barra + 2 patas (color metal `#8d9196`).
- Patas en los puntos de agujero (`centroX ± entreCentros/2`); barra ~2 cm más larga que la distancia entre centros; adelantada 3,5 cm de la cara.

## 9. Patas

- Por defecto **4 patas redondas regulables**: altura **15,5 cm**, diámetro 5 cm.
- **Centro a 7,5 cm** de cada borde del mueble (frente, trasera y los dos laterales).
- El cuerpo del mueble se **eleva** la altura de la pata: `MODELO.elevacion = 15.5`; todas las piezas que **no** sean patas se suben esa cota. Las patas se marcan `base:true` (no se elevan) y van de `y=0` a `y=alto`.
- Se generan con `patasMueble(ancho, fondo, alto, inset=7.5, diam=5)` (color metal `#9a9da2`).
- Soporte de geometría cilíndrica: pieza con `forma:"cilindro"` (eje Y, diámetro = `size[0]`, altura = `size[1]`).

## 10. Baldas (estantes)

Generadas con `baldasMueble(num, ancho, alto, fondo, ec, traseraZ, fondoBalda)`.

- **Huecos iguales dividiendo el COSTADO completo** (de `y=0` a `y=alto`, no solo el
  interior): se reparte en `num + 1` tramos iguales. Cada línea divisoria es la
  **altura del porta-estante** (mecanización): `yLínea(i) = i · alto/(num+1)`, `i = 1..num`.
  - Ejemplo 1 balda en bajo 70: `yLínea = 70/2 = 35 cm` (mitad justa del costado).
  - Ejemplo 2 baldas: `23,33` y `46,67 cm`.
- **La balda apoya 0,5 cm por encima** de esa línea → la **cara inferior** de la balda
  está en `yLínea + 0,5`. (`pos.y = yLínea + 0,5`).
- **Espesor** de balda = `EC` (mismo tablero que el casco).
- **Ancho** = `(ancho − 2·EC) − 0,3` (3 mm menos que suelo/techo), **centrada** a lo
  ancho: `pos.x = EC + ((ancho−2·EC) − anchoBalda)/2`.
- **Fondo** por defecto en **bajos = 50 cm**; va **apoyada en la trasera** (su cara
  trasera toca la cara delantera de la trasera): `pos.z = traseraZ − fondoBalda`,
  con `traseraZ = FONDO − T` (`T=5` bajos, `T=1,6` altos).
- La balda **no** es frente: siempre visible aunque se oculte la puerta.

## 11. Visibilidad de la puerta

- Estado `puertaVisible` (bool) + botón `#btnPuerta` ("Puerta: visible" ⇄ "oculta").
- Al ocultar, `poblarGrupo` filtra las piezas `frente:true` y omite sus tiradores
  (`tiradores()`); el resto (carcasa, baldas, patas) se mantiene. Útil para ver el
  interior y la colocación de baldas.

## 12. Bisagras de cazoleta + animación de puerta

Bisagra europea **cazoleta Ø35, codo 0**, malla real de SolidWorks → STL vía **Onshape**
(importar `.SLDASM`+piezas en zip → Export STL binario mm). El STL se **parte en 2** por
el eje X del herraje (umbral −42 mm): **`bisagra_casco.stl`** (pletina/base, X<−42) y
**`bisagra_puerta.stl`** (cazoleta+brazo, X>−42). Ambas se centran por el MISMO punto
`BIS_CENTRO` para conservar su posición relativa.

- **Pletina (casco):** fija al costado → va en `grupo`.
- **Cazoleta+brazo (puerta):** giran con la puerta → van en un sub-grupo `bisP`
  (hijo de `puertaPivot`) **pre-rotado `−ANGULO_ABIERTO`**. Así, al abrir la puerta
  (`puertaPivot → ANGULO_ABIERTO`) la cazoleta coincide con la pletina (hinge montada),
  y la cazoleta sigue metida en la puerta en todo el recorrido. El STL está en pose
  **abierta**, por eso el estado ABIERTO es el de referencia (`M` = `BISAGRA_ROT`+`OFF`
  se afina mirando la puerta abierta). Si faltan los STL → versión paramétrica.

- **Lado de bisagras = opuesto al tirador** (`HINGE_LADO`). Para "puerta izquierda"
  (tirador izq) → bisagras a la **derecha**.
- **2 bisagras** a `CUPS_Y = [10, alto−10]` (10 cm de cada extremo).
- **Cazoleta** Ø3,5 cm, prof 1,2 cm, centro a **2,2 cm del canto** de la puerta
  (`CUP_X`), embebida en la cara interior (z 0..−1,2). Lleva 2 alas con tornillo.
- **Codo** sale de la cazoleta hacia el interior; **pletina + brazo + leva** van
  atornillados a la cara interior del costado (fijos).
- **Solidario a la puerta** (`mov:true`): puerta, tiradores, cazoleta, alas y codo →
  van dentro de `puertaPivot`, restándoles el pivote. **Fijos**: carcasa, baldas,
  patas, pletina, brazo, leva.

**Eje de giro / animación:**
- `puertaPivot` (THREE.Group) pivota sobre `(PIVOT_X, *, PIVOT_Z)` = canto de la puerta
  del lado de bisagras, cara frontal. Es una aproximación de un solo eje (la bisagra
  real es un 4 barras, pero visualmente abre como una puerta normal).
- `ANGULO_ABIERTO` ≈ ±100° (signo según lado). El bucle `animate` suaviza
  `anguloPuerta → anguloObjetivo` (factor 0,14).
- Botón **`#btnAbrir`** ("Abrir/Cerrar puerta"). Si la puerta está oculta, al abrir se
  vuelve a mostrar. `poblarGrupo` recrea `puertaPivot` y reaplica `anguloPuerta`.

## 13. Pendiente (fase 2)

- Mecanizaciones (taladros porta-estantes como agujeros reales en los costados,
  ranuras, cajeados, guías).
- Cajones abiertos / interiores de cajón.
- Auto-generar las imágenes PNG del catálogo desde el render.

---

## Flujo de publicación

1. Editar `MODELO` (y `medidas`) en `visor.html`.
2. `git add/commit/push` en `github-pages/` (rama `main`).
3. El enlace **no cambia**; compartir siempre la misma URL.
