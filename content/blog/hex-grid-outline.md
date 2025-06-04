+++
title = "Hexágonos Flat-Top en Unity: Crea una Rejilla y Resalta Celdas con LineRenderer"
date = 2025-06-03
+++

# Cómo crear una cuadrícula de hexágonos “flat‐top” en Unity con un contorno procedural que siga el ratón

> **¿Te gustaría aprender a generar en Unity una cuadrícula de hexágonos planos (flat‐top) y resaltar el que esté bajo el cursor con un borde dibujado en tiempo real?**  
> En este artículo paso a paso, en español y con un tono relajado, veremos cómo hacerlo usando un **LineRenderer** en lugar de un sprite para el contorno. Al final tendrás un tablero hexagonal completamente funcional y un highlight (borde) que aparecerá justo donde coloques el ratón.

---

## Índice

1. [Introducción](#introduccion)  
2. [1. Crear un proyecto 2D en Unity](#1-crear-un-proyecto-2d-en-unity)  
3. [2. Organizar carpetas e importar sprites](#2-organizar-carpetas-e-importar-sprites)  
4. [3. Crear el prefab “HexTile”](#3-crear-el-prefab-hexTile)  
5. [4. Generar la cuadrícula flat‐top](#4-generar-la-cuadricula-flat-top)  
6. [5. Dibujar el contorno con LineRenderer](#5-dibujar-el-contorno-con-linerenderer)  
   - [5.1. El concepto del hex flat‐top](#5-1-el-concepto-del-hex-flat-top)  
   - [5.2. Script `HexOutlineController`](#5-2-script-hexoutlinecontroller-cs)  
   - [5.3. Configuración del LineRenderer en el prefab](#5-3-configuracion-del-linerenderer-en-el-prefab)  
7. [6. Capturar el ratón y mostrar/ocultar el contorno](#6-capturar-el-raton-y-mostrar-ocultar-el-contorno)  
   - [6.1. Input System vs. Input Manager](#6-1-input-system-vs-input-manager)  
   - [6.2. Script `MouseHighlighter`](#6-2-script-mousehighlighter-cs)  
8. [7. Resultado final y consejos de ajuste](#7-resultado-final-y-consejos-de-ajuste)  
9. [Conclusiones](#conclusiones)  

---

## Introducción

Las cuadrículas hexagonales son muy populares en juegos de estrategia, rol por turnos o simulación, porque cada celda se conecta con seis vecinas (sin ángulos rectos forzados). En este tutorial veremos:

- **Cómo crear un proyecto Unity 2D** desde cero.  
- **Importar tus sprites de hexágono** (tamaño 96×84 px).  
- **Generar una cuadrícula flat‐top** (colocación “odd‐q offset”).  
- **Dibujar un contorno** (outline) usando un **LineRenderer**, sin necesidad de tener un sprite adicional.  
- **Detectar la posición del ratón** y resaltar en tiempo real el hexágono sobre el que esté el cursor.

¡Manos a la obra!

---

## 1. Crear un proyecto 2D en Unity

1. **Abre Unity Hub**.  
2. Pulsa **New**, elige la plantilla **2D** y ponle de nombre, por ejemplo, `HexGridFlatDemo`.  
3. Selecciona la carpeta de destino y pulsa **Create Project**.  
4. Una vez dentro, asegúrate de que en la ventana **Scene** esté activado el botón **2D** (arriba a la izquierda).  
5. Guarda la escena actual desde **File → Save As…**, crea la carpeta `Assets/Scenes/` y nómbrala `MainScene.unity`.

¡Listo! Ya tienes un proyecto 2D limpio para empezar.

---

## 2. Organizar carpetas e importar sprites

Antes de escribir código, es buena idea mantener todo ordenado. En el panel **Project → Assets**, crea estas carpetas:

```
Assets/
├─ Scenes/
├─ Scripts/
├─ Sprites/
├─ Prefabs/
└─ Materials/   (opcional, para el material del contorno)
```

### 2.1. Tus imágenes PNG

Vas a necesitar **dos** archivos PNG (96×96 px de canvas, 96×84 px de parte visible):

- **HexSprite.png**  
  - El hexágono relleno (verde, por ejemplo). Bounding = 96×84 px.  
- **HexOutlineSprite.png** (opcional, solo para referencia)  
  - La misma figura pero dibujada solo con un contorno. En este tutorial no lo usaremos como contorno final, ¡pero sirve para comparar si deseas ver cómo quedaría con sprite!

Arrástralos dentro de `Assets/Sprites/`.

### 2.2. Configurar cada sprite

1. Selecciona `HexSprite.png`.  
2. En el **Inspector**:  
   - **Texture Type** = **Sprite (2D and UI)**  
   - **Sprite Mode** = **Single**  
   - **Pixels Per Unit** = **100** (con este valor, 96 px = 0.96 unidades, 84 px = 0.84 unidades).  
   - Pulsa **Apply**.  

3. Repite exactamente lo mismo con `HexOutlineSprite.png` (aunque luego no lo usaremos como contorno final, te sirve para comparaciones).


---

### 🧭 ⚠️ Importante sobre el *Pivot*

Cuando haces `Slice` de los tiles (por ejemplo 96×96 px), es **fundamental ajustar el Pivot correctamente** para que el centro visual del hexágono se alinee bien con la cuadrícula.  
En muchos sprites el hex está centrado verticalmente en **solo 84 px**, lo que deja un margen de 6 px arriba y abajo.

🔧 Por eso, el **`Pivot` debe colocarse en coordenadas normalizadas**:

- X = `0.5` (centro horizontal del tile)
- Y = `0.4375` (es decir: `42 / 96`, donde 42 es el centro de la parte útil del hexágono)

> 💡 Unity usa valores entre `0` y `1` para representar el pivot, **no píxeles reales**.  
> Si querés posicionar el centro exacto de un gráfico que mide 84 px de alto dentro de un canvas de 96 px, necesitás calcular:  
> `pivotY = (96 - 84) / 2 + 42 = 42` → `42 / 96 = 0.4375`


4. **Comprobar el tamaño en unidades** (opcional):  
   - Arrastra brevemente `HexSprite.png` a la **Hierarchy** y renómbralo a “HEX_TMP”.  
   - Selecciónalo y, en **SpriteRenderer → Bounds → Size**, verás aproximadamente `X=0.96  Y=0.84`.  
   - Borra “HEX_TMP” una vez verificado.

Desde ahora, sabemos que:

- **`hexWidth  = 0.96` unidades**  
- **`hexHeight = 0.84` unidades**  

Estas cifras las usaremos para generar la cuadrícula y dibujar el contorno.

---

## 3. Crear el prefab “HexTile”

Vamos a crear un prefab que represente cada celda (tile) de la cuadrícula: el sprite + un **PolygonCollider2D** para detectar el ratón.

1. **Crear GameObject base**:  
   - En **Hierarchy → clic derecho → Create Empty** → renómbralo a `HexTilePrefab`.  
   - Ajusta su **Transform.position = (0, 0, 0)**.  

2. **Añadir SpriteRenderer**:  
   - Con `HexTilePrefab` seleccionado, pulsa **Add Component → Sprite Renderer**.  
   - Arrastra **HexSprite.png** al campo **Sprite**.  
   - Deja **Sorting Layer / Order in Layer = Default / 0** (o crea un layer “Tiles” si prefieres).

3. **Añadir PolygonCollider2D**:  
   - Con `HexTilePrefab` activo, pulsa **Add Component → Polygon Collider 2D**.  
   - Unity automáticamente trazará el polígono en torno a los píxeles no transparentes (96×84 px).  

4. **(Opcional) Guardar coordenadas con un script**:  
   - En `Assets/Scripts/`, crea un nuevo script **C#** llamado `HexTileInfo.cs`:  
     ```cs
     using UnityEngine;

     // Guarda las coordenadas q,r (columna, fila) de cada hex
     public class HexTileInfo : MonoBehaviour
     {
         public int q; // columna (x)
         public int r; // fila   (y)
     }
     ```  
   - Guarda y regresa a Unity.  
   - Con `HexTilePrefab` seleccionado, pulsa **Add Component → HexTileInfo** (para poder almacenar la posición de rejilla más adelante).

5. **Etiquetar como “HexTile”**:  
   - Con `HexTilePrefab` seleccionado, en el Inspector haz clic en **Tag → Add Tag…**.  
   - Crea la etiqueta **HexTile** y asígnala a `HexTilePrefab`. Esto permitirá que el script de raycast reconozca cada celda.

6. **Convertir en prefab**:  
   - Arrastra `HexTilePrefab` desde la **Hierarchy** a `Assets/Prefabs/`.  
   - Así se crea `HexTile.prefab`.  
   - Borra “HexTilePrefab” de la **Hierarchy** (solo necesitas el asset).

¡Listo! Ya tienes tu prefab de “HexTile” con sprite y collider.

---

## 4. Generar la cuadrícula flat‐top

Para hexágonos con la parte “plana” arriba y abajo (“flat‐top”), usaremos un **layout odd‐q offset**, donde:

- Cada columna se desplaza verticalmente **media fila** si el índice de columna es impar.  
- La distancia horizontal entre centros es `hexWidth * 0.75`.  
- La distancia vertical entre centros es `hexHeight`.

Recordemos:

- `hexWidth  = 0.96f`  
- `hexHeight = 0.84f`  

Entonces:

- `xOffset = hexWidth * 0.75f = 0.96 * 0.75 = 0.72`  
- `yOffset = hexHeight = 0.84`

### 4.1. Script `HexGridGenerator.cs`

En `Assets/Scripts/`, crea **C# Script → HexGridGenerator.cs** con este contenido:

```cs
using UnityEngine;

public class HexGridGenerator : MonoBehaviour
{
    [Header("Configuración de la cuadrícula")]
    public int gridWidth = 10;       // número de columnas
    public int gridHeight = 8;       // número de filas
    public GameObject hexPrefab;     // arrastra aquí HexTile.prefab

    [Header("Tamaño real del hexágono (unidades)")]
    public float hexWidth  = 0.96f;   // 96 px ÷ 100 PPU
    public float hexHeight = 0.84f;   // 84 px ÷ 100 PPU

    void Start()
    {
        GenerateHexGrid();
    }

    void GenerateHexGrid()
    {
        if (hexPrefab == null)
        {
            Debug.LogError("HexGridGenerator: falta asignar hexPrefab.");
            return;
        }

        // Cálculo de offsets para flat-top (odd-q layout)
        float xOffset = hexWidth * 0.75f; // 0.72
        float yOffset = hexHeight;        // 0.84

        for (int col = 0; col < gridWidth; col++)
        {
            for (int row = 0; row < gridHeight; row++)
            {
                // Posición horizontal
                float xPos = col * xOffset;

                // Posición vertical base
                float yPos = row * yOffset;

                // Desplazar media fila si la columna es impar
                if (col % 2 == 1)
                    yPos += yOffset * 0.5f; // 0.42

                Vector3 spawnPos = new Vector3(xPos, yPos, 0f);
                GameObject hexGO = Instantiate(hexPrefab, spawnPos, Quaternion.identity, this.transform);
                hexGO.name = $"Hex_{col}_{row}";

                // Guardar coordenadas si existe HexTileInfo
                HexTileInfo info = hexGO.GetComponent<HexTileInfo>();
                if (info != null)
                {
                    info.q = col;
                    info.r = row;
                }
            }
        }
    }
}
```

### 4.2. Configurar “HexGridManager”

1. En **Hierarchy**, clic derecho → **Create Empty**, renómbralo a `HexGridManager`.  
2. Con `HexGridManager` seleccionado, pulsa **Add Component → HexGridGenerator**.  
3. En el Inspector de `HexGridGenerator`, asigna:
   - **Grid Width** = 10  
   - **Grid Height** = 8  
   - **Hex Prefab** = `Assets/Prefabs/HexTile.prefab`  
   - **Hex Width** = 0.96  
   - **Hex Height** = 0.84  

4. Pulsa **Play**. Verás en la jerarquía un montón de objetos “Hex_0_0”, “Hex_0_1”, …, “Hex_9_7”, y en la **Scene** la cuadrícula empaquetada sin huecos.

¡Bravo! Ya tienes la base de la cuadrícula flat‐top.

---

## 5. Dibujar el contorno con LineRenderer

En lugar de utilizar un sprite para resaltar el hex seleccionado, vamos a aprovechar el **LineRenderer** para dibujar un polígono de seis lados (hexágono) en tiempo real. Así el contorno será vectorial y escalable.

### 5.1. El concepto del hex flat‐top

- Tu sprite mide en unidades reales:  
  - **Ancho total** = `hexWidth = 0.96`  
  - **Alto total** = `hexHeight = 0.84`  

Para dibujar un hexágono plano, necesitamos conocer el **radio** `r` (distancia desde el centro al punto medio de un lado horizontal) y la **altura vertical** `h` (distancia desde el centro al vértice superior). La fórmula es:

1. `r = hexWidth / 2` → `0.96 / 2 = 0.48` unidades.  
2. `h = r * sqrt(3) / 2` → `0.48 * 0.8660254 ≈ 0.4157` unidades.

Con `r` y `h` podemos calcular los seis vértices en **local‐space**:

```text
0: ( +r,     0 )         → medio lado derecho
1: ( +r/2,  +h )         → esquina superior‐derecha
2: ( -r/2,  +h )         → esquina superior‐izquierda
3: ( -r,     0 )         → medio lado izquierdo
4: ( -r/2,  -h )         → esquina inferior‐izquierda
5: ( +r/2,  -h )         → esquina inferior‐derecha
6: repetir ( +r, 0 )     → para cerrar el loop
```

### 5.2. Script `HexOutlineController.cs`

En `Assets/Scripts/`, crea un archivo **C#** llamado `HexOutlineController.cs` y copia el siguiente código:

```cs
using UnityEngine;

[RequireComponent(typeof(LineRenderer))]
public class HexOutlineController : MonoBehaviour
{
    private LineRenderer lr;

    [Header("Dimensiones del hexágono")]
    [Tooltip("Ancho total del sprite flat-top en unidades (por ejemplo, 0.96)")]
    public float hexWidth = 0.96f;

    [Tooltip("Alto total del sprite flat-top en unidades (por ejemplo, 0.84)")]
    public float hexHeight = 0.84f;

    [Header("Grosor del contorno")]
    [Tooltip("Ancho (en unidades) de la línea que dibuja el contorno.")]
    public float lineWidth = 0.05f;

    void Awake()
    {
        lr = GetComponent<LineRenderer>();

        // Trabajamos en local space y cerramos el loop
        lr.useWorldSpace = false;
        lr.loop = true;

        // Definimos grosor constante desde la variable pública
        lr.startWidth = lineWidth;
        lr.endWidth   = lineWidth;

        // Calculamos r y h para el hex flat-top
        float r = hexWidth * 0.5f;            
        float h = r * Mathf.Sqrt(3f) / 2f;     

        // Creamos los 6 vértices + repetir el primero
        Vector3[] corners = new Vector3[7];
        corners[0] = new Vector3(+r,     0f, 0f);
        corners[1] = new Vector3(+r/2f,  +h, 0f);
        corners[2] = new Vector3(-r/2f,  +h, 0f);
        corners[3] = new Vector3(-r,     0f, 0f);
        corners[4] = new Vector3(-r/2f,  -h, 0f);
        corners[5] = new Vector3(+r/2f,  -h, 0f);
        corners[6] = corners[0];  // cierre del loop

        lr.positionCount = corners.Length;
        lr.SetPositions(corners);

        // Al arrancar, lo ocultamos
        lr.enabled = false;
    }

    /// <summary>
    /// Mueve el contorno al centro del hex en worldPos y lo muestra.
    /// </summary>
    public void ShowAt(Vector3 worldPos)
    {
        // useWorldSpace = false → mover el transform
        transform.position = new Vector3(worldPos.x, worldPos.y, -0.1f);
        lr.enabled = true;
    }

    /// <summary>
    /// Oculta el contorno.
    /// </summary>
    public void Hide()
    {
        lr.enabled = false;
    }
}
```

### 5.3. Configuración del LineRenderer en el prefab

1. Arrastra el GameObject **HexOutline** (vacío) a la **Hierarchy** y renómbralo a `HexOutline`.  
2. Pule **Add Component → Line Renderer**.  
3. En el Inspector del **Line Renderer**:
   - **Material**: crea o asigna un **Material** basado en **Sprites/Default** con color a tu gusto (blanco, amarillo…).  
   - **Loop**: ✔ (marcado).  
   - **Use World Space**: ✘ (desmarcado).  
   - **Sorting Layer**: `Default` (u otro) y **Order in Layer** = `1`.  
4. **Agrega `HexOutlineController`** al mismo GameObject:
   - Haz clic en **Add Component → HexOutlineController**.  
   - En el Inspector podrás editar:
     - **Hex Width** = `0.96`  
     - **Hex Height** = `0.84`  
     - **Line Width** = `0.05`  
5. Convierte `HexOutline` en prefab: arrástralo desde la **Hierarchy** a `Assets/Prefabs/ ⇒ HexOutline.prefab`.  
6. Borra `HexOutline` de la **Hierarchy**.

---

## 6. Capturar el ratón y mostrar/ocultar el contorno

Ahora que ya tenemos la cuadrícula y el prefab del contorno, solo falta hacer el raycast cada fotograma y, si el ratón está sobre un hexágono, mover el contorno al centro de ese tile.

### 6.1. Input System vs. Input Manager

Unity ha “evolucionado” su sistema de entrada. Si ves este error al compilar:

```
InvalidOperationException: You are intentando leer Input usando UnityEngine.Input, 
pero tienes activado el Input System Package en Player Settings.
```

tienes dos opciones:

- **Opción A (rápida)**: volver a la API clásica  
  - **Edit → Project Settings → Player → Other Settings → Active Input Handling** → selecciona **Input Manager (Old)** (o **Both**).  
  - Así `Input.mousePosition` vuelve a funcionar sin tocar el script.

- **Opción B (recomendada si quieres usar Input System)**: migrar tu código al nuevo paquete  
  - Instala el **Input System** desde **Window → Package Manager**.  
  - Reemplaza `Input.mousePosition` por `Mouse.current.position.ReadValue()` y agrega `using UnityEngine.InputSystem;` al script.

En este tutorial mostraremos la **Opción B**, porque es la recomendación actual de Unity.

### 6.2. Script `MouseHighlighter.cs`

En `Assets/Scripts/`, crea un archivo **C#** llamado `MouseHighlighter.cs` y pega este contenido:

```cs
using UnityEngine;
// Necesario para la nueva Input System:
using UnityEngine.InputSystem;

public class MouseHighlighter : MonoBehaviour
{
    [Header("Referencias")]
    public Camera mainCamera;          // Arrastra tu Main Camera aquí
    public GameObject outlinePrefab;   // Arrastra aquí HexOutline.prefab

    private HexOutlineController outlineController;
    private GameObject outlineInstance;

    void Start()
    {
        // Si no asignaste la cámara en el Inspector, toma siempre la principal
        if (mainCamera == null)
            mainCamera = Camera.main;

        if (outlinePrefab == null)
        {
            Debug.LogError("MouseHighlighter: falta asignar outlinePrefab.");
            return;
        }

        // Instanciamos el contorno (solo una vez)
        outlineInstance = Instantiate(outlinePrefab);
        outlineController = outlineInstance.GetComponent<HexOutlineController>();
    }

    void Update()
    {
        // 1) Obtener posición del ratón con la nueva Input System
        Vector2 mouseScreenPos;
        if (Mouse.current != null)
        {
            mouseScreenPos = Mouse.current.position.ReadValue();
        }
        else
        {
            // No hay ratón disponible (ej. en móviles), salimos
            outlineController.Hide();
            return;
        }

        // 2) Convertir de screen (2D) a world (3D)
        Vector3 mouseWorld3D = mainCamera.ScreenToWorldPoint(
            new Vector3(mouseScreenPos.x, mouseScreenPos.y, 0f));
        Vector2 mouseWorld2D = new Vector2(mouseWorld3D.x, mouseWorld3D.y);

        // 3) Raycast 2D puntual en esa posición (Vector2.zero = dirección cero)
        RaycastHit2D hit = Physics2D.Raycast(mouseWorld2D, Vector2.zero);

        if (hit.collider != null && hit.collider.CompareTag("HexTile"))
        {
            // El ratón está sobre un tile válido
            Vector3 hexPos = hit.collider.gameObject.transform.position;
            outlineController.ShowAt(hexPos);
        }
        else
        {
            // Si no está sobre ningún HexTile
            outlineController.Hide();
        }
    }
}
```

### 6.3. Configurar `MouseHighlighterManager`

1. En la **Hierarchy**, clic derecho → **Create Empty** → renómbralo a `MouseHighlighterManager`.  
2. Con él seleccionado, pulsa **Add Component → MouseHighlighter**.  
3. En el Inspector de `MouseHighlighter`:  
   - **Main Camera** = arrastra tu cámara principal (“Main Camera”).  
   - **Outline Prefab** = arrastra `Assets/Prefabs/HexOutline.prefab`.  

¡Y eso es todo! Al pulsar **Play**, Unity:

- Instanciará la cuadrícula bajo `HexGridManager`.  
- Instanciará un único `HexOutline(Clone)` bajo `MouseHighlighterManager`.  
- Cada fotograma hará un Raycast en la posición del ratón; si detecta un `HexTile`, llamará a `ShowAt(...)`, que moverá el contorno y lo mostrará; si no, llamará a `Hide()`, que lo ocultará.

---

## 7. Resultado final y consejos de ajuste

1. Pulsa **Play**. En la escena debería verse tu tablero de hexágonos flat‐top:  
   - Columnas: desplazadas media fila si el índice es impar (odd‐q).  
   - Filas perfectamente empaquetadas sin huecos.  
2. Mueve el ratón sobre distintos hexágonos:  
   - Aparecerá un borde blanco/amarillento perfectamente acoplado al hex bajo el cursor, con grosor `lineWidth` (por defecto 0.05).  
   - Al salir del tile, el contorno se ocultará.  

> **Ajustes que puedes cambiar**:  
> - **`lineWidth`** en `HexOutlineController`: modifica a 0.02 para un borde fino, o 0.1 para algo más grueso.  
> - **Color del contorno**: crea un material diferente (Assets/Materials/ → New Material → Shader “Sprites/Default” → elige color) y asígnalo al LineRenderer.  
> - **Tamaño de la rejilla**: en `HexGridGenerator`, modifica `gridWidth` y `gridHeight` a 20×12 o lo que necesites.  
> - **`hexWidth` / `hexHeight`**: si cambias por otro sprite, actualiza estos valores en el prefab `HexOutline` y en `HexGridGenerator`.  
> - **Input System vs. Input Manager**: si tu proyecto no usa el paquete Input System, basta con cambiar a “Input Manager (Old)” en **Project Settings → Player → Other Settings → Active Input Handling**, y luego tu script original que usaba `Input.mousePosition` funcionará sin problemas.

#### 7.1. Ejemplo final de jerarquía (modo edición)

```
MainScene
├─ HexGridManager             (con HexGridGenerator)
├─ MouseHighlighterManager    (con MouseHighlighter)
└─ Main Camera
```

Durante el Play Mode, se añadirán:

```
MainScene (Play Mode)
├─ HexGridManager
│   ├─ Hex_0_0  (Sprite + Collider + HexTileInfo)
│   ├─ Hex_0_1
│   └─ …       (en total gridWidth×gridHeight)
│   └─ Hex_9_7
├─ MouseHighlighterManager
└─ HexOutline  (instanciado, con LineRenderer + HexOutlineController)
└─ Main Camera
```

---

## Conclusiones

¡Y así hemos terminado! 🎉 Ahora sabes:

- Crear un **proyecto Unity 2D** desde cero.  
- Importar sprites (96×84 px → 0.96×0.84 unidades) y organizarlos en carpetas.  
- Generar una **cuadrícula flat‐top** de hexágonos con **odd‐q offset** usando un script sencillo.  
- Crear un contorno **procedural** con **LineRenderer**, calculando vértices con `r = hexWidth/2` y `h = r * √3 / 2`.  
- Mostrar/ocultar ese contorno en tiempo real según la posición del ratón, usando la **nueva Input System**.

Con este tutorial tienes una base muy sólida para:  
- Hacer clic sobre los hexágonos para selección.  
- Implementar pathfinding (A*, Dijkstra, Flood Fill) sobre la rejilla.  
- Pintar tiles en diferentes colores (cosecha, terreno, unidades).  
- Añadir animaciones a los contornos (por ejemplo, cambiar de color o pulsar).

¡Anímate a explorar más!  
Si tienes dudas, comentarios o quieres sugerir mejoras, deja un mensaje y seguimos aprendiendo juntos.  

¡Feliz codificación hexagonal! 🚀  
