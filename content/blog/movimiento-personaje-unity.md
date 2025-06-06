+++
title = "Movimiento de personaje sobre hexágonos con seguimiento de cámara inteligente"
date = 2025-06-06
+++

# Movimiento de personaje sobre hexágonos con seguimiento de cámara inteligente

En este segundo artículo vamos a ver cómo implementamos el sistema de movimiento para el personaje del jugador dentro de la cuadrícula de hexágonos. También explicamos cómo configuramos una cámara que lo sigue de forma inteligente, y se desacopla automáticamente si el jugador decide moverla manualmente.

---

## Movimiento entre hexágonos adyacentes

Para que el personaje solo pueda moverse a tiles vecinos, usamos una función que calcula los vecinos adyacentes según el sistema odd-q offset:

```cs
public static Vector2Int[] GetNeighbors(Vector2Int coord)
{
    int q = coord.x;
    int r = coord.y;

    Vector2Int[] evenQ = { ... };
    Vector2Int[] oddQ = { ... };

    return (q % 2 == 0) ? evenQ : oddQ;
}
```

Y en `TryMoveTo`, verificamos que el destino sea vecino y que el tile sea navegable:

```cs
foreach (var neighbor in Character.GetNeighbors(currentCoord))
{
    if (neighbor == targetCoord && HexMapManager.Instance.IsNavigable(targetCoord))
    {
        StartCoroutine(MoveToHex(...));
    }
}
```

---

## Movimiento suave con easing

El desplazamiento del personaje no es instantáneo, sino que se hace con una interpolación suave:

```cs
private IEnumerator MoveToHex(Vector3 targetWorldPos, Vector2Int targetHex)
{
    Vector3 start = transform.position;
    Vector3 end = targetWorldPos;

    float elapsed = 0f;
    while (elapsed < moveDuration)
    {
        float t = elapsed / moveDuration;
        float easedT = easing.Evaluate(t);
        transform.position = Vector3.Lerp(start, end, easedT);
        elapsed += Time.deltaTime;
        yield return null;
    }

    transform.position = end;
    character.hexCoordinates = targetHex;
}
```

---

## Cámara inteligente: seguir o no seguir

La cámara está controlada por un script `CameraController` que permite:

* Seguir automáticamente al personaje si está en modo seguimiento.
* Desacoplarse si el jugador la mueve con WASD.
* Recentrarla manualmente con barra espaciadora.

```cs
if (!userOverride && IsPlayerControllingCamera())
{
    DisableFollow();
}

if (followEnabled && followTarget != null)
{
    FollowTarget();
}
```

Y cualquier clase puede pedir a la cámara que recentre al objetivo:

```cs
camController.RecentreOnTarget();
```

---

## Delegando responsabilidades

Una parte importante del refactor fue mover toda la lógica de seguimiento y centrado a `CameraController`. Esto nos permite que `PlayerMovement` se mantenga simple:

```cs
yield return StartCoroutine(MoveSmoothlyTo(...));
camController.RecentreOnTarget();
```

La cámara decide si recentrar, con suavizado o instantáneo, y si reactivar el seguimiento.

---

En conjunto con el artículo anterior, esto nos deja con un sistema sólido, elegante y escalable de movimiento y navegación sobre la cuadrícula de hexágonos.
