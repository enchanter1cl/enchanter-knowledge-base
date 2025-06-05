---
icon: bow-arrow
---

# Addressables.LoadAssetsAsync

Beginner Level code version:

```csharp
    IEnumerator Start()
    {
        var handle = Addressables.LoadAssetAsync<Sprite>("character_0");
        yield return handle;

        if (handle.Status == AsyncOperationStatus.Succeeded)
        {
            Sprite mySprite = handle.Result;
            GetComponent<Image>().sprite = mySprite;
        }
    }
```
