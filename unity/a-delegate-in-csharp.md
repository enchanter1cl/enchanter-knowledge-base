---
icon: bow-arrow
---

# A Delegate In CSharp

### **🎯 What is a Delegate in C#?**

A **delegate** is a **type that holds a reference to a method** — like a **function pointer**.

> 💡 It lets you **treat methods like variables** — you can pass them around, store them, and call them dynamically.

### **🧠 Analogy: A Delegate is Like a Remote Control**

Imagine a TV remote:

* The **remote** = delegate
* The **TV** = method

You don’t care how the TV works — you just press the button, and it does something. You can switch the remote to control different TVs (i.e., assign different methods).

⸻

## 🧪 Basic Syntax

✅ Step 1: Declare a delegate type

```csharp
public delegate void SaySomething();
```

This says: “Any method I use here must return void and take no parameters.”\
⸻\
✅ Step 2: Create a method to match

```csharp
void Hello()
{
	Debug.Log("Hello");
}
```

⸻

✅ Step 3: Assign the method to the delegate

```csharp
SaySomething sayHello = Hello;
```

SaySomething say = Hello;\
⸻

✅ Step 4: Call the delegate

```csharp
sayHello(); //Output: Hello!
```

⸻

## 🔧 Built-in Shortcut: Action and Func

C# already includes ready-made delegate types:

| Type       | Meaning                       |
| ---------- | ----------------------------- |
| Action     | Delegate with no return value |
| Action\<T> | Delegate with 1 parameter     |
| Func\<T>   | Delegate that returns a value |

## 🎯 What Is an Event in C#?

An event lets one object notify other objects when something happens — like a button being clicked or a sprite finishing loading.\
Think of it like this:\
“Hey, something just happened! Anyone want to respond?"

👶 Beginner Analogy\
Imagine you’re organizing a party. You say:\
“When pizza arrives, everyone clap!”

```
•	“Pizza arrives” = the event
•	“Clap” = the response method (callback)
```

You’re setting up a rule: when Event A happens, call Method B.

### 🔧 Basic Syntax

Here’s how to create, subscribe to, and trigger an event.

✅ Step 1: Define a Delegate (function type)

```csharp
public delegate void MyEventHandler(string message);
```

This says: “I’ll accept any method that takes a string and returns void.”

✅ Step 2: Declare an Event

```csharp
public event MyEventHandler OnSomethigHappened;
```

public event MyEventHandler OnSomethingHappened;

✅ Step 3: Subscribe to the Event

```csharp
OnSomethingHappend += MyMethod
```

✅ Step 4: Raise (Trigger) the Event

```csharp
if (OnSomethingHappened != null)
    OnSomethingHappened("Hello!");
```

### ⚡ Modern Shortcut (No Need for Manual Delegate)

**In real C# code, we often skip the delegate declaration and use built-in types like Action, Action\<T>, or EventHandler.**

🔹 Example with Action\<string>:

```csharp
Using UnityEngine;
//...
public event Action<string> OnClick;

void Start()
{
    OnClick += Hear;
    OnClick?.Invoke("Hi!");
}

void Hear(string msg)
{
    Debug.Log("click say" + msg);
}
```

Summary,

| Term     | Meaning                                           |
| -------- | ------------------------------------------------- |
| event    | A signal that something has occured               |
| delegate | A function signature that subscribers must follow |
| +=       | **Subscribe a method to run when event triggers** |
