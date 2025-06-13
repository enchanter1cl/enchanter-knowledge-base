# Csharp Field and Property

> ❌ I**n C# a “property” is not the same as a “field”**. They are **very different concepts**.

#### **📦** **Field**

#### &#x20;**= a variable that holds data inside a class**

It’s the actual **storage** for data.

```csharp
public class Player
{
    public int score; // ← this is a field
}
```

* Has **no logic**, just raw memory.
* Can be public or private.
* If public: other scripts can read/write it directly — not safe.

***

#### **🪟** Property = a public interface to access a field

&#x20;**(or computed value)**

```csharp
public class Player
{
    private int score;  // ← field (backing field)

    public int Score    // ← property
    {
        get { return score; }
        set { score = value; }
    }
}
```

Or in shorthand:

```
public int Score => score;  // read-only property
```

* Looks like a variable when accessed.
* But actually calls hidden getter/setter **methods** behind the scenes.
* Can add validation, logging, computed values, etc.
* Much safer and more flexible.

***

#### **✅ Key Differences**

| **Feature**      | **Field**    | **Property**                        |
| ---------------- | ------------ | ----------------------------------- |
| Holds data?      | ✅ Yes        | ❌ Not necessarily (may be computed) |
| Can add logic?   | ❌ No         | ✅ Yes (in get/set)                  |
| Access syntax    | player.score | player.Score (looks same)           |
| Backed by memory | ✅ Yes        | ❌ Maybe (not always)                |
| Safe to expose?  | ❌ No         | ✅ Yes (controlled)                  |

***

#### **🔧 Example: Why it matters**

```csharp
public class Player
{
    public int health; // anyone can do: player.health = -999;
}
```

vs

```csharp
public class Player
{
    private int health;
    public int Health
    {
        get { return health; }
        set { health = Mathf.Max(0, value); }  // never below 0!
    }
}
```

Now you protect your data — thanks to the property.

***

#### **🧠 Summary:**

> **Field = raw data**

> **Property = controlled access to that data (or a computed value)**

They’re different tools with different purposes. In C#, it’s best to **keep fields private and use properties** for public access.

_So can i just use clean getter and setter in DifficultyManager situation?_

_No_

## Name Convention

🧠 In Unity:

Unity uses m\_ prefix for serialized private fields:

```csharp
[SerializeField]
private int m_Health;

public int Health => m_Health;
```

This is Unity’s convention, not a general C# rule.

⸻

Summary

| Element               | Style                       | Example                         |
| --------------------- | --------------------------- | ------------------------------- |
| Public property       | PascalCase                  | PlayerScore                     |
| Private field         | camelCase                   | playerScore                     |
| Private backing field | \_camelCase or m\_CamelCase | \_playerScore or m\_PlayerScore |

## Omit backing field

🍬 Bonus: Auto-Implemented Property

If you don’t need custom logic, C# lets you skip writing the backing field:

```csharp
public int Score {get; set;} // 👈 auto-property
```

C# creates a hidden backing field behind the scenes.
