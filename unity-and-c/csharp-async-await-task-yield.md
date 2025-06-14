# Csharp Async Await Task Yield

## 🔹 async and await — Asynchronous Programming

✅ Goal:

Run non-blocking code (like loading a file or waiting for a web request) without freezing your app.

⸻

🔧 async keyword\
• Marks a method as asynchronous.\
• It allows using await inside that method.\
• It usually returns Task, Task\<T>, or void (for **event handlers**).

```csharp
public async Task DoSomethingAsync() 
{
	await Task.Delay(2000); // Imagine there's a mission of HTTP request need 2 seconds to complete. Non-blocking.
	Debug.Log("Done HTTP mission");
}
```

⸻

🔧 await keyword\
• Waits for a Task to finish, but doesn’t block the main thread.\
• Resumes the method after the awaited task completes.

```csharp
await Task.Delay(1000); // Delays for 1 second without freezing; Imitating HTTP request
```

⸻

## 🔹 Task and Task\<T> — Represent Future Results

✅ Goal:

Run methods in the background, or represent operations that complete later.

```csharp
public async Task<int> GetScoreAsync() 
{
	await Task.Delay(1000);//Imitating HTTP request
	return 42;
}
```

```
•	Task: returns nothing
•	Task<T>: returns value T when done
```

⸻

## 🔹 yield — Generator Keyword

✅ Goal:

Create lazy iterators (one item at a time), useful for loops, state machines, or coroutines.

```csharp
public IEnumerable<int> GetNumber() 
{
	yield return 1;
	yield return 2;
	yield return 3;
}
```

```
•	yield return: gives one value at a time
•	yield break: ends the sequence
```

🧠 In Unity:

Unity’s IEnumerator coroutines (used with StartCoroutine) rely on yield:

```csharp
IEnumerator WaitExample()
{
    yield return new WaitForSeconds(2);
    Debug.Log("2 seconds passed");
}
```

⸻

🔄 Summary Table

| Keyword | Purpose                        | Common Return Type       | Example Use             |
| ------- | ------------------------------ | ------------------------ | ----------------------- |
| async   | Marks a method as asynchronous | Task, Task               | async Task LoadData()   |
| await   | Waits for a Task to finish     | N/A                      | await LoadData()        |
| Task    | Represents async operation     | Task, Task               | return Task.Delay(1000) |
| yield   | Generates values lazily        | IEnumerable, IEnumerator | yield return x          |

⸻

🧪 Want to test it?

Here’s a quick Unity example:

```csharp
private async void Start()
{
    Debug.Log("Before wait");
    await Task.Delay(3000); // Doesn't freeze Unity. Imitating this is a http request
    Debug.Log("After 3 seconds");
}
```

And a coroutine version with yield:

```csharp
private void Start()
{
    StartCoroutine(MyCoroutine());
}

private IEnumerator MyCoroutine()
{
    Debug.Log("Before wait");
    yield return new WaitForSeconds(3); //Imitating this is a http request
    Debug.Log("After 3 seconds");
}
```
