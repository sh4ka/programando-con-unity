+++
title = "Tiles por bioma"
date = 2025-06-04
+++

# Cómo generar una cuadrícula hexagonal con tiles basados en bioma

En este tutorial te enseñaremos cómo crear una cuadrícula de hexágonos en Unity donde cada tile se genera automáticamente en función de tres valores simulados: **altura**, **temperatura** y **humedad**. Utilizaremos una **sprite sheet** y un solo prefab para mantener la eficiencia.

---

## Índice

1. [Preparar los assets y carpetas](#1-preparar-los-assets-y-carpetas)
2. [Importar y cortar la sprite sheet](#2-importar-y-cortar-la-sprite-sheet)
3. [Crear el prefab base `HexTile`](#3-crear-el-prefab-base-hextile)
4. [Asignar sprites automáticamente desde Resources](#4-asignar-sprites-automaticamente-desde-resources)
5. [Elegir el bioma según ruido Perlin](#5-elegir-el-bioma-segun-ruido-perlin)
6. [Resultado esperado](#6-resultado-esperado)

---

## 1. Preparar los assets y carpetas

Organiza tus carpetas de esta forma:

```
Assets/
├─ Resources/
│  └─ Sprites/
│     └─ HexTilesheet.png
├─ Scripts/
├─ Prefabs/
└─ Scenes/
```

Tu `HexTilesheet.png` debe contener todos los tiles posibles para los diferentes biomas y variaciones. 

Yo he usado la colección [Pixel Hex Tileset](https://zeshio.itch.io/pixel-hex) y para agrupar las que necesitaba he usado [TexturePacker](https://www.codeandweb.com/texturepacker).
---

## 2. Importar y cortar la sprite sheet

1. Selecciona `HexTilesheet.png` en el panel Project.
2. En el **Inspector**:

    * `Texture Type`: **Sprite (2D and UI)**
    * `Sprite Mode`: **Multiple**
    * `Pixels Per Unit`: 100
3. Pulsa **Apply**, luego entra en **Sprite Editor**.
4. En Sprite Editor:

    * `Slice → Grid by Cell Size`
    * Tamaño: `96 x 96` (o el que uses)
    * Pulsa **Slice**, luego **Apply**

Unity generará automáticamente sub-sprites con nombre `HexTilesheet_0`, `HexTilesheet_1`, etc.

---

## 3. Crear el prefab base `HexTile`

1. En `Hierarchy`, clic derecho → **Create Empty**, renómbralo a `HexTilePrefab`.
2. Añade:

    * `SpriteRenderer`
    * `PolygonCollider2D`
3. Opcional: agrega el script `HexTileInfo.cs` para guardar posición `q,r`.
4. Convierte en prefab arrastrándolo a `Assets/Prefabs/`.

---

## 4. Asignar sprites automáticamente desde Resources

En tu script `HexGridGenerator.cs`, cargá los sprites así:

```cs
private Sprite[] tileSprites;

void Start()
{
    tileSprites = Resources.LoadAll<Sprite>("Sprites/HexTilesheet");
    GenerateHexGrid();
}
```

> Asegurate de que el sprite sheet esté en `Assets/Resources/Sprites/`

Luego, mapearás cada bioma a una lista de índices:

```cs
private Dictionary<string, int[]> biomeSpriteIndices = new Dictionary<string, int[]>
{
    { "ocean",    new[] { 0, 1, 2 } },
    { "mountain", new[] { 3, 4 } },
    { "hill",     new[] { 5, 6, 7, 8 } },
    { "grass",    new[] { 9, 10 } },
    { "forest",   new[] { 11, 12, 13, 14, 15 } }
};
```

---

## 5. Elegir el bioma según ruido Perlin

Para cada celda de la cuadrícula:

```cs
float height = GetPerlin(x, y, heightScale, heightOffset);
float temp = GetPerlin(x, y, tempScale, tempOffset);
float humidity = GetPerlin(x, y, humidityScale, humidityOffset);

string biome = GetBiome(height, humidity, temp);

int[] indices = biomeSpriteIndices[biome];
int spriteIndex = indices[Random.Range(0, indices.Length)];

GameObject tile = Instantiate(hexPrefab, spawnPos, Quaternion.identity, this.transform);
tile.GetComponent<SpriteRenderer>().sprite = tileSprites[spriteIndex];
```

Y tu función `GetBiome` podría ser:

```cs
string GetBiome(float height, float humidity, float temperature)
{
    if (height < 0.3f) return "ocean";
    if (height > 0.8f) return "mountain";
    if (humidity > 0.7f) return "forest";
    if (humidity > 0.4f) return "grass";
    return "hill";
}
```

---

## 6. Resultado esperado

Cuando ejecutes la escena:

* Verás una cuadrícula de hexágonos flat-top
* Cada tile tendrá un sprite diferente según su bioma
* Se usarán variaciones visuales gracias a la sprite sheet
* Todo se hace con **un único prefab** y sin usar múltiples variantes

Esto te da un sistema eficiente y listo para escalar con nuevas condiciones como estaciones, civilizaciones, estructuras, etc.

---

¡Feliz codificación hexagonal! 🚀
