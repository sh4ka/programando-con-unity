+++
title = "Generación de grid de forma procedural"
date = 2025-06-04
+++

# Cómo combinar contorno interactivo y generación procedural en una cuadrícula hexagonal en Unity

> En este artículo aprenderás a generar una cuadrícula de hexágonos flat-top en Unity, asignar a cada celda valores naturales como altura, humedad y temperatura usando ruido Perlin, y además resaltar con un contorno animado el hexágono bajo el ratón.

---

## Índice

1. [Resumen del objetivo](#resumen-del-objetivo)
2. [Preparar el proyecto y los sprites](#preparar-el-proyecto-y-los-sprites)
3. [Generar la cuadrícula procedural](#generar-la-cuadricula-procedural)
4. [Visualizar valores naturales](#visualizar-valores-naturales)
5. [Agregar contorno interactivo con LineRenderer](#agregar-contorno-interactivo-con-linerenderer)
6. [Resultado final y sugerencias](#resultado-final-y-sugerencias)

---

## Resumen del objetivo

Partiendo desde cero o con una base como la del artículo anterior [Cómo crear una cuadrícula de hexágonos “flat‐top”](@/blog/hex-grid-outline.md), vamos a integrar:

* Un sistema para generar cada tile con valores naturales:

    * **altura**
    * **humedad**
    * **temperatura**
* Una visualización directa de esos valores como color
* Un contorno animado con LineRenderer que sigue el ratón

Todo esto usando **Input System**, **Perlin Noise** y sin shaders personalizados (ideal para empezar sin complicaciones).

---

## Preparar el proyecto y los sprites

Sigue los pasos de los apartados 1 y 2 del artículo anterior [Cómo crear una cuadrícula de hexágonos “flat‐top”](@/blog/hex-grid-outline.md). Asegúrate de:

* Tener los sprites `HexSprite.png` correctamente importados como `Sprite (2D and UI)` con `Pixels Per Unit = 100`
* Crear un prefab `HexTile.prefab` con:

    * SpriteRenderer
    * PolygonCollider2D
    * Script `HexTileInfo` que guarda las coordenadas `q` y `r`

---

## Generar la cuadrícula procedural

El script `HexGridGenerator.cs` debe:

* Instanciar los hexágonos en posición según layout **odd-q offset**
* Calcular para cada uno valores de ruido procedural

### Valores naturales por ruido:

```cs
float nx = col * noiseScale;
float ny = row * noiseScale;

info.height = RoundToStep(Mathf.PerlinNoise(nx + heightOffset.x, ny + heightOffset.y));
info.moisture = RoundToStep(Mathf.PerlinNoise(nx + moistureOffset.x, ny + moistureOffset.y));
info.temperature = RoundToStep(Mathf.PerlinNoise(nx + temperatureOffset.x, ny + temperatureOffset.y));
```

Donde `RoundToStep` es:

```cs
float RoundToStep(float value, float step = 0.1f)
{
    return Mathf.Round(value / step) * step;
}
```

Define un `noiseScale` bajo (ej. 0.2) y offsets separados (0, 100, 200).

---

## Visualizar valores naturales

Dentro del mismo `HexGridGenerator`, al instanciar cada tile puedes usar su `SpriteRenderer` para visualizar los valores. Ejemplo con altura:

```cs
var sr = hexGO.GetComponent<SpriteRenderer>();
if (sr != null)
{
    float h = info.height;
    sr.color = Color.Lerp(Color.black, Color.white, h);
}
```

O visualización combinada en RGB:

```cs
sr.color = new Color(info.height, info.moisture, info.temperature);
```

Esto permite ver el "mapa" directamente desde la vista de escena.

---

## Agregar contorno interactivo con LineRenderer

Usaremos un prefab llamado `HexOutline` que contiene:

* Un **LineRenderer** configurado en modo local
* Un script `HexOutlineController` que calcula los 6 vértices para dibujar el borde

### Resumen de configuración:

* LineRenderer:

    * Loop ✔
    * Use World Space ✘
    * Width: 0.05 (ajustable)
* Script `HexOutlineController.cs`:

```cs
public void ShowAt(Vector3 worldPos)
{
    transform.position = new Vector3(worldPos.x, worldPos.y, -0.1f);
    lr.enabled = true;
}

public void Hide()
{
    lr.enabled = false;
}
```

### Script `MouseHighlighter`

Este script usará raycast 2D con `InputSystem` para mover el contorno si el mouse está sobre un `HexTile`:

```cs
RaycastHit2D hit = Physics2D.Raycast(mouseWorld2D, Vector2.zero);

if (hit.collider != null && hit.collider.CompareTag("HexTile"))
{
    Vector3 hexPos = hit.collider.gameObject.transform.position;
    outlineController.ShowAt(hexPos);
}
else
{
    outlineController.Hide();
}
```

Recuerda asignar la cámara y el prefab del contorno desde el inspector.

---

## Resultado final y sugerencias

Al pulsar **Play**, tendrás:

* Una cuadrícula hexagonal procedural
* Cada tile con color generado por sus valores naturales
* Un contorno suave que sigue el ratón en tiempo real

### Ideas para continuar:

* Añadir **nombres de biomas** según combinaciones de valores (ej. altura > 0.8 y humedad > 0.6 → "Bosque húmedo")
* Agregar una UI flotante que muestre los valores al pasar el ratón
* Usar shaders o materiales avanzados para texturas

---

¿Quieres aprender cómo pasar de estos valores a biomas y decoraciones visuales? ¡Pronto publicaremos una guía sobre generación de biomas y tilesets dinámicos!

¡Feliz desarrollo procedural! 🎲🌍
