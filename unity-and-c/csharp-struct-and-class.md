# Struct and Class

🧱 struct vs class – Summary Table

| Feature             | struct                                               | class                               |
| ------------------- | ---------------------------------------------------- | ----------------------------------- |
| Type category       | Value type                                           | Reference type                      |
| Stored in           | Stack (usually)                                      | Heap                                |
| Copied by           | Value (deep copy)                                    | Reference (shallow copy)            |
| Inheritance         | ❌ Cannot inherit or be inherited (except interfaces) | ✅ Full inheritance support          |
| Default constructor | ❌ Cannot define one ✅ Can define freely              |                                     |
| Mutable?            | Can be mutable, but often avoided                    | Fully mutable                       |
| Memory efficient?   | ✅ More efficient for small data                      | 👎 Less efficient for small structs |
| Use cases           | Lightweight data (e.g. Vector2, Color)               | Complex objects, behaviors          |
| Keyword             | struct                                               | class                               |

⸻

🧪 Example of struct

```csharp
public struct Point
{
    public int x;
    public int y;
}

Point a = new Point { x = 1, y = 2 };
Point b = a;
b.x = 99;

Debug.Log(a.x); // 🔹 Outputs 1 (copy was independent)
```

✔️ This is a deep copy — struct behaves like a primitive.

⸻

🧪 Example of class

```csharp
public class Player
{
    public string Name;
}

Player p1 = new Player { Name = "Alice" };
Player p2 = p1;
p2.Name = "Bob";

Debug.Log(p1.Name); // 🔸 Outputs "Bob" (reference changed both)
```

❌ This is a reference copy — changes affect both variables.

⸻

📦 Use struct when:\
• You need small, simple data types (e.g. positions, colors, etc.)
