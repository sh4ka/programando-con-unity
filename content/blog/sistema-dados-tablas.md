+++
title = "Sistema de dados y tablas aleatorias en Unity"
date = 2025-06-05
+++

# Sistema de dados y tablas aleatorias en Unity

En muchos juegos de rol o estrategia, es común encontrar tablas de eventos o resultados que se activan mediante una tirada de dados. En este artículo te explico paso a paso cómo implementar un sistema de dados y tablas aleatorias en Unity, utilizando un generador de números aleatorios determinista para obtener resultados reproducibles.

---

## ¿Por qué usar dados y tablas?

El uso de dados permite introducir incertidumbre de forma controlada. Una tabla aleatoria puede modelar desde clima hasta encuentros o recompensas. Por ejemplo:

* Una tabla de 1 a 100 puede representar tipos de eventos.
* Cada fila se activa con un rango de resultados (como 01–10: lluvia).
* La tirada de dados define qué entrada se activa.

---

## Crear la clase `DiceExpression`

Creamos una clase serializable que representa combinaciones del tipo `2d6 + 3`:

```cs
using Unity.Mathematics;

namespace Etrelia.Unity.Core
{
    [System.Serializable]
    public class DiceExpression
    {
        public int diceCount = 1;
        public int diceSides = 6;
        public int modifier = 0;

        public DiceExpression(int count, int sides, int modifier = 0)
        {
            this.diceCount = count;
            this.diceSides = sides;
            this.modifier = modifier;
        }

        public int Roll(ref Random rng)
        {
            int total = 0;
            for (int i = 0; i < diceCount; i++)
            {
                total += rng.NextInt(1, diceSides + 1);
            }
            return total + modifier;
        }

        public override string ToString()
        {
            string mod = modifier == 0 ? "" :
                         modifier > 0 ? $" + {modifier}" : $" - {-modifier}";
            return $"{diceCount}d{diceSides}{mod}";
        }
    }
}
```

---

## Clase `RandomTableEntry` y `RandomTable`

Estas clases permiten definir las tablas con rangos de resultados:

```cs
namespace Etrelia.Unity.Tables
{
    [System.Serializable]
    public class RandomTableEntry
    {
        public int minRoll;
        public int maxRoll;
        public string result;
    }
}
```

```cs
using System.Collections.Generic;
using UnityEngine;
using Unity.Mathematics;
using Etrelia.Unity.Core;

namespace Etrelia.Unity.Tables
{
    [CreateAssetMenu(fileName = "NewRandomTable", menuName = "Tables/RandomTable")]
    public class RandomTable : ScriptableObject
    {
        public string tableName;
        public DiceExpression diceUsed;
        public List<RandomTableEntry> entries;

        public string RollAndGetResult(ref Random rng)
        {
            int roll = diceUsed.Roll(ref rng);
            return GetResult(roll);
        }

        public string GetResult(int roll)
        {
            foreach (var entry in entries)
            {
                if (roll >= entry.minRoll && roll <= entry.maxRoll)
                    return entry.result;
            }
            return $"({roll}) Sin resultado válido";
        }
    }
}
```

Esto te permite crear assets en el editor como tablas aleatorias con sus respectivos dados y rangos.

---

## `GameManager`: el controlador central del juego

Creamos un singleton que mantenga el stream aleatorio y sirva para acceder al sistema:

```cs
using UnityEngine;
using Unity.Mathematics;

namespace Etrelia.Unity.Core
{
    public class GameManager : MonoBehaviour
    {
        public static GameManager Instance { get; private set; }

        [Header("Random Stream")]
        public uint seed = 12345;
        private Random rng;

        void Awake()
        {
            if (Instance != null && Instance != this)
            {
                Destroy(gameObject);
                return;
            }

            Instance = this;
            DontDestroyOnLoad(gameObject);
            rng = new Random(seed);
        }

        public Random GetRNG() => rng;

        public int RollDice(DiceExpression dice) => dice.Roll(ref rng);
    }
}
```

Este objeto debe estar presente en la escena inicial. Lo ideal es guardarlo como prefab y cargarlo desde la primera escena.

---

## Ejemplo de uso

```cs
using UnityEngine;
using Etrelia.Unity.Core;
using Etrelia.Unity.Tables;

public class TestRoll : MonoBehaviour
{
    public RandomTable table;

    void Start()
    {
        var rng = GameManager.Instance.GetRNG();
        string result = table.RollAndGetResult(ref rng);
        Debug.Log($"Resultado obtenido: {result}");
    }
}
```

---

## Conclusión

Con este sistema tienes:

* Dados configurables y reutilizables
* Tablas que dependen de los dados para determinar resultados
* Un GameManager que mantiene el RNG estable para todo el juego

Esto te permite avanzar hacia un sistema de narrativa procedural, encuentros aleatorios, loot controlado y mucho más.

> [Configurar `MouseHighlighterManager`](@/blog/hex-grid-outline.md#6-2-script-mousehighlighter-cs)

¡Ahora puedes usar dados de verdad en tu juego sin perder el control! 🎲
