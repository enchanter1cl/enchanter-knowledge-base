# EditorPlayBootstrapper

This script `EditorPlayBootstrapper.cs` is an **editor-only utility** in Unity that automates how your game enters **Play Mode** in the **Editor**, especially useful for **bootloader patterns** like yours (with PersistentManagers as the entry point). Here’s a **detailed breakdown**:

***

#### **🔹 File Location & Purpose**

* **Path**: Assets/-----Game-----/Scripts/Editor/EditorPlayBootstrapper.cs
* **Purpose**: Ensures that whenever you press **Play** in the Unity Editor, the game **starts from your boot scene (PersistentManagers)**, no matter what scene you were editing before.

***

#### **🔸 Key Concepts in This Script**

**1.**&#x20;

**\[InitializeOnLoad]**

*   This attribute tells Unity:

    > _“Run this class’s static constructor automatically whenever the editor loads.”_

So even before you press Play, this script **hooks into Unity’s lifecycle**.

***

**2.**&#x20;

**EditorApplication.playModeStateChanged += OnPlayModeChanged;**

* Inside the static constructor:

```csharp
static EditorPlayBootstrapper()
{
    EditorApplication.playModeStateChanged += OnPlayModeChanged;
}
```

This line **registers a listener** for Play Mode state changes (e.g., Entering Play Mode, Exiting, etc.).

When such a state change occurs, it runs:

***

#### **🔹 OnPlayModeChanged: How It Works**

```csharp
private static void OnPlayModeChanged(PlayModeStateChange state)
```

This method is the heart of the script.

**✅ When you press**&#x20;

**Play**

&#x20;**in the editor:**

```csharp
if (state == PlayModeStateChange.EnteredPlayMode)
```

*   **Step 1**:

    Store your currently open scene so that you can come back to it later:

```csharp
previousScenePath = SceneManager.GetActiveScene().path;
```

*   **Step 2**:

    Load your **bootloader entry scene**:

```csharp
SceneManager.LoadScene(BootScenePath, LoadSceneMode.Single);
```

* This replaces your current scene with PersistentManagers.
  * This ensures your game **always starts the same way**, like in a real build.
* Optional:

```
//SceneManager.LoadScene(MainScenePath, LoadSceneMode.Additive);
```

* If uncommented, this would **also load MainMenu additively**, useful if you want to test from the main menu directly.

***

**🔁 When you stop playing (back to Edit Mode):**

```csharp
else if (state == PlayModeStateChange.EnteredEditMode)
```

* It **restores** the scene you were working on before pressing Play:

```csharp
EditorSceneManager.OpenScene(previousScenePath, OpenSceneMode.Single);
```

So you don’t lose your editing context — you go back to where you left off.

***

#### **📌 Summary – What It Automates**

| **Action**               | **What Happens**                                                     |
| ------------------------ | -------------------------------------------------------------------- |
| You hit ▶ Play           | Unity **switches to PersistentManagers scene** automatically         |
| You stop Play            | Unity **goes back to your last edited scene**                        |
| Bootload pattern support | Ensures correct loading order in Editor like in real game            |
| Benefit                  | No need to manually switch to PersistentManagers every time you test |

#### **✅ Tips for You**

1. **Make sure PersistentManagers** is marked to **DontDestroyOnLoad** for managers.
2. This script **only affects the Editor**, <mark style="color:red;">not the actual build.</mark>

## why use "initializeOnLoad and static constructor

1. ✅ Why \[InitializeOnLoad] is used
2. ✅ Why the class and constructor are static

### **✅ 1. What does**  **\[InitializeOnLoad]**  **do?**

In Unity **Editor scripts**, \[InitializeOnLoad] makes Unity run the static constructor **as soon as the Unity Editor finishes loading**, or after domain reload (e.g., script recompile, reentering Play Mode).\
This is **Editor-only** behavior and only works in classes inside an Editor folder or marked with #if UNITY\_EDITOR.

### ✅ 2. Why static class and static constructor?

🔹&#x20;

#### **static class EditorPlayBootstrapper**

* A static class:
  * Cannot be instantiated (no new allowed)
  * Can only contain static members
  * Best for utility/helper code
  * Keeps things self-contained and memory-efficient

#### **static EditorPlayBootstrapper()**

This is a **C# static constructor**. Key facts:

* Automatically runs **once** when the class is accessed for the first time — OR immediately on load when combined with \[InitializeOnLoad]
* No parameters allowed
* You **can’t call it manually**; it’s **guaranteed to run once**

**Why it’s perfect here:** You want to hook into:

```csharp
EditorApplication.playModeStateChanged += OnPlayModeChanged;
```

…exactly once, when the Editor loads. This ensures Unity watches for Play mode changes from the start.

### **✅ 3. What C# rules and concepts are involved?**

| **Concept**        | **Description**                                                        |
| ------------------ | ---------------------------------------------------------------------- |
| static class       | Cannot be instantiated; used for utility or singleton-like behavior    |
| static constructor | Auto-executes once per domain load; no args; cannot be manually called |
| delegate           | playModeStateChanged uses a delegate event to register callbacks       |
| +=                 | Adds a method to an event handler list                                 |
