+++
title = "Refactorizando la información de los hexágonos: tipos, enum y mapa centralizado"
date = 2025-06-06
+++

# Refactorizando la información de los hexágonos: tipos, enum y mapa centralizado

En este artículo vamos a revisar cómo hemos mejorado la forma en la que manejamos los datos de cada hexágono en nuestra cuadrícula flat-top. Pasamos de usar strings para identificar el tipo de terreno a una estructura más robusta con enums, y creamos un sistema centralizado para acceder a la información de cualquier tile desde cualquier parte del juego.

## ¿Por qué evitar strings para los tipos de terreno?

Inicialmente teníamos algo como esto en `HexTileInfo`:

```cs
public string tileType; // Ej: "ocean", "grass", "forest"
```

Esto funcionaba, pero tenía varios problemas:

* Mayor propensión a errores tipográficos.
* Comparaciones de strings menos eficientes.
* No hay ayuda de autocompletado en el editor.

La solución fue introducir un `enum` llamado `TileType`:

```cs
public enum TileType
{
    Grass,
    Forest,
    Hill,
    Mountain,
    Ocean,
    Desert
}
```

Y luego actualizar `HexTileInfo`:

```cs
public TileType tileType;
```

Unity lo muestra como un dropdown en el Inspector, y desde código las comparaciones son seguras y rápidas:

```cs
if (tile.tileType == TileType.Ocean)
{
    // Bloquear movimiento
}
```

---

## Centralizando el mapa con HexMapManager

Para evitar tener que hacer raycasts o buscar GameObjects cada vez que queramos saber algo sobre un tile, creamos una clase llamada `HexMapManager` que actúa como fuente de verdad para todos los hexágonos.

```cs
public class HexMapManager : MonoBehaviour
{
    public static HexMapManager Instance { get; private set; }

    private Dictionary<Vector2Int, HexTileInfo> tileMap = new();

    void Awake()
    {
        Instance = this;
    }

    public void RegisterTile(Vector2Int coords, HexTileInfo tile)
    {
        tileMap[coords] = tile;
    }

    public bool TryGetTile(Vector2Int coords, out HexTileInfo tile)
    {
        return tileMap.TryGetValue(coords, out tile);
    }

    public bool IsNavigable(Vector2Int coords)
    {
        return TryGetTile(coords, out var tile) && tile.tileType != TileType.Ocean;
    }
}
```

Durante la generación de la cuadrícula, registramos cada tile:

```cs
Vector2Int coords = new(col, row);
HexMapManager.Instance.RegisterTile(coords, info);
```

Y luego podemos consultar desde cualquier script:

```cs
if (HexMapManager.Instance.IsNavigable(qrCoords))
{
    // Mover al personaje
}
```

---

## Ventajas de esta refactorización

* Acceso rápido y centralizado a la información de los tiles.
* Código más limpio, seguro y legible.
* Mayor facilidad para extender el sistema (por ejemplo, podríamos agregar `GetMovementCost()` según el tipo de tile).

En el próximo artículo vamos a centrarnos en todo lo relacionado con el movimiento del personaje, desde la selección del tile hasta el seguimiento de cámara inteligente.
