# Islas de Aether

Un archipiélago de fantasía renderizado **enteramente con caracteres**. Sin texturas, sin mallas 3D, sin WebGL: un `<canvas>` 2D dibujando glifos sobre una rejilla, en **un solo archivo HTML** sin dependencias.

**→ [Entrar al mundo](https://fleremiasflemin20-maker.github.io/islas-de-aether/)**

![Islas de Aether](captura.png)

---

## Qué hay dentro

- **474.000 puntos**, 70 islas repartidas en 6 regiones, generadas proceduralmente desde una semilla.
- **Ciclo día/noche completo**: sol y luna recorren el cielo, la paleta se reescribe cada fotograma, de noche se encienden ventanas, cristales y estrellas.
- **Dragones que reaccionan a tu mirada**: si los enfocas más de un segundo, rompen su órbita y se te plantan delante.
- **Mapa navegable** del archipiélago con viaje rápido entre regiones.
- **60 fps** gracias a culling por chunks y LOD por submuestreo.

## Controles

| Tecla | Acción |
|---|---|
| `W` `A` `S` `D` | Volar |
| `Q` `E` | Bajar / subir |
| arrastrar ratón | Mirar |
| `⇧` | Impulso |
| `M` | Mapa del archipiélago |
| `1`–`6` | Viajar a una región |
| `T` | Adelantar la hora |
| `H` | Cómo funciona el motor |

Si sueltas los controles, entra en *gran tour* automático por las regiones.

## Cómo funciona el motor

1. **Nube de puntos, no polígonos.** El mundo es un array de puntos `{x, y, z, material, brillo}` generados con ruido: discos de isla, quillas de roca colgando al vacío, troncos, copas, agujas de cristal, casas huecas con ventanas, puentes con catenaria.

2. **Proyección en perspectiva.** Cada punto se rota a espacio de cámara (`yaw`, `pitch`) y se divide entre la profundidad: `x/z`, `y/z`. Hay que corregir que la celda de texto es más alta que ancha, o todo sale estirado.

3. **Z-buffer por celda.** Una rejilla paralela guarda la profundidad más cercana vista en cada celda de caracteres. Lo que llega más lejos se descarta. Eso da oclusión real sin ordenar nada.

4. **La rampa de caracteres es el sombreado.** `brillo = luz direccional × niebla por distancia`, y ese número indexa la rampa del material: `. , : ; - = + * # @`. Cada material tiene rampa y color propios, por eso roca, musgo y cristal se leen distinto siendo todo texto.

5. **Chunks + LOD.** Los puntos se ordenan una vez (counting sort) en celdas de 70 unidades. Cada fotograma se descartan de golpe las celdas fuera del cono de visión o más allá de la niebla, y las lejanas se recorren *salteadas* (1 de cada 2, 4 u 8). Se tocan decenas de miles de puntos por fotograma, no 474.000.

6. **Pintado en tiras.** No se dibuja carácter a carácter: cada fila agrupa celdas contiguas del mismo color en una cadena y se pinta de un golpe. De ~10.000 celdas a unos cientos de llamadas a `fillText`.

## Añadir una región

Una región es solo datos. El motor no distingue Aether de Cenizas:

```js
{
  name: "Tu región", sub: "un subtítulo", cx: 0, cz: 0, cy: 0,
  span: 130, n: 12, rMin: 8, rMax: 26,
  seed: 12345,
  flora: "tree",        // "tree" | "dead" | "shroom"
  houses: true, spires: 2,
  glow: { 2: 0.8 },     // materiales que brillan de noche
  skyNight: [6, 8, 14], skyDay: [30, 50, 70],
  mats: [ /* 8 colores RGB: roca, follaje, cristal, luz, agua, dragón, libre, madera */ ]
}
```

Añádela al array `REGIONS` y ya tienes otro sitio del mundo.

## Licencia

MIT.
