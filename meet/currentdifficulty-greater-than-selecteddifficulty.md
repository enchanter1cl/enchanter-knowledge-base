# CurrentDifficulty=> SelectedDifficulty

```csharp
public static DifficultyManager Instance {get; private set;}

private Difficulty selectedDifficulty = Difficulty.Medium;

public Difficulty CurrentDifficulty => selectedDifficulty

public void selectDifficulty(Difficulty difficulty) 
{
	selectedDifficulty = difficulty;
}
```

since it need public set and get, why not just use `public selectDifficulty`??

Reason0: No **Side Effects**\
now it require all changes to go through method selectDifficulty()\
If later you want to log a change, trigger UI updates, or check something before applying the difficulty, you’d have to duplicate that logic everywhere.

classic Java Bean Style:

```java
public class User {
    private String name;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }
}
```

classic Csharp field and property:

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

So can i just use clean getter setter in my Difficulty Manager situation? Just add some validation logic in the setter method.\
You can, but it would be more mess.

```java
    private Difficulty selectedDifficulty;
    public Difficulty SelectedDifficulty
    {
        get { return selectedDifficulty; }
        set
        {
            // ✅ Add validation here
            if (!System.Enum.IsDefined(typeof(Difficulty), value))
            {
                Debug.LogWarning("Invalid difficulty selected.");
                return;
            }
            selectedDifficulty = value;  // ✅ Correct assignment
        }

    }
```
