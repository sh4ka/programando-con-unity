+++
title = "Crear un \"Monster Manual\" en Unity para enemigos tipo RPG"
date = 2025-06-09
+++

## Crear un "Monster Manual" en Unity para enemigos tipo RPG

En este artículo veremos cómo estructurar un sistema de enemigos en Unity inspirado en los juegos de rol clásicos. El objetivo es tener una forma sencilla y organizada de definir criaturas con estadísticas personalizadas, sin repetir código ni depender de configuraciones engorrosas en el editor.

### ✨ Enfoque general

Partimos de una arquitectura basada en estas piezas:

* **`CharacterSheet`**: la hoja de personaje que contiene atributos como fuerza, constitución, inteligencia, etc.
* **`CharacterSheetTemplate`**: una plantilla reutilizable que define los valores base para crear hojas.
* **`MonsterType`**: un `enum` con los nombres de enemigos posibles.
* **`MonsterManual0`**: una clase estática que actúa como "bestia digital", conteniendo una tabla de templates listos para usar.

Esto nos permite, por ejemplo, instanciar un orco con:

```cs
var orcSheet = MonsterManual0.GetTemplate(MonsterType.Orc).CreateSheet();
```

---

### 📒 Clase `CharacterSheet`

Contiene todos los atributos de combate:

```cs
public class CharacterSheet
{
    public int maxHp;
    public int currentHp;

    public int strength;
    public int dexterity;
    public int constitution;
    public int intelligence;
    public int wisdom;
    public int charisma;

    public int armorClass;

    public CharacterSheet(CharacterSheetTemplate template)
    {
        strength     = template.strength;
        dexterity    = template.dexterity;
        constitution = template.constitution;
        intelligence = template.intelligence;
        wisdom       = template.wisdom;
        charisma     = template.charisma;
        armorClass   = template.armorClass;
        maxHp        = template.maxHp;
        currentHp    = maxHp;
    }

    public static int GetModifier(int stat) => Mathf.FloorToInt((stat - 10) / 2f);

    public int StrengthMod     => GetModifier(strength);
    public int DexterityMod    => GetModifier(dexterity);
    public int ConstitutionMod => GetModifier(constitution);
    public int IntelligenceMod => GetModifier(intelligence);
    public int WisdomMod       => GetModifier(wisdom);
    public int CharismaMod     => GetModifier(charisma);
}
```

---

### 📁 Clase `CharacterSheetTemplate`

Es una estructura serializable que define los valores base:

```cs
public class CharacterSheetTemplate
{
    public int strength;
    public int dexterity;
    public int constitution;
    public int intelligence;
    public int wisdom;
    public int charisma;
    public int armorClass;
    public int maxHp;

    public CharacterSheet CreateSheet()
    {
        return new CharacterSheet(this);
    }
}
```

---

### ♟ Enum `MonsterType`

Define los nombres de enemigos que usaremos como clave:

```cs
public enum MonsterType
{
    Goblin,
    Orc,
    Skeleton,
    Troll
    // Puedes agregar más tipos aquí
}
```

---

### 👾 Clase `MonsterManual0`

Una clase estática con un diccionario de templates listos:

```cs
public static class MonsterManual0
{
    private static Dictionary<MonsterType, CharacterSheetTemplate> _templates;

    static MonsterManual0()
    {
        _templates = new Dictionary<MonsterType, CharacterSheetTemplate>
        {
            {
                MonsterType.Goblin,
                new CharacterSheetTemplate
                {
                    strength = 8,
                    dexterity = 14,
                    constitution = 10,
                    intelligence = 8,
                    wisdom = 8,
                    charisma = 6,
                    armorClass = 13,
                    maxHp = 10
                }
            },
            {
                MonsterType.Orc,
                new CharacterSheetTemplate
                {
                    strength = 16,
                    dexterity = 12,
                    constitution = 14,
                    intelligence = 7,
                    wisdom = 10,
                    charisma = 8,
                    armorClass = 13,
                    maxHp = 20
                }
            },
            {
                MonsterType.Skeleton,
                new CharacterSheetTemplate
                {
                    strength = 10,
                    dexterity = 14,
                    constitution = 10,
                    intelligence = 6,
                    wisdom = 8,
                    charisma = 5,
                    armorClass = 13,
                    maxHp = 13
                }
            },
            {
                MonsterType.Troll,
                new CharacterSheetTemplate
                {
                    strength = 18,
                    dexterity = 10,
                    constitution = 16,
                    intelligence = 5,
                    wisdom = 9,
                    charisma = 5,
                    armorClass = 15,
                    maxHp = 40
                }
            }
        };
    }

    public static CharacterSheetTemplate GetTemplate(MonsterType type)
    {
        if (_templates.TryGetValue(type, out var template))
            return template;

        Debug.LogError($"No se encontró el template para el tipo {type}");
        return null;
    }
}
```

---

### ✅ Ventajas del enfoque

* ✔️ Modular: podés cargar datos desde archivos JSON o ScriptableObjects en el futuro
* ✔️ Reutilizable: un solo template para muchas instancias de enemigos
* ✔️ Escalable: fácil de extender con habilidades, loot, inteligencia, etc.

Este sistema sienta la base para integrar enemigos al sistema de combate, IA, o eventos de exploración. En el futuro podrías combinarlo con un generador de encuentros aleatorios o un editor visual para tus monstruos.

¡Nos vemos en el próximo artículo!
