# Description

The Addressable Asset System allows the developer to ask for an asset via its address. Once an asset (e.g. a prefab) is marked "addressable", it generates an address which can be called from anywhere. Wherever the asset resides (local or remote), the system will locate it and its dependencies, then return it.

Use ‘Window->Asset Management->Addressables’ to begin working with the system.

Unity's Addressables system is a dynamic asset management solution that provides your users with only the assets they need, when they need them. It allows you to organize your on-demand assets from inside the Unity Editor while you're developing your game, and its runtime API lets you load and unload assets dynamically while users are playing your game.

🔹 Addressables.LoadAssetAsync\<Sprite>(spriteAddress)\
• This tells Unity:\
“Start loading an asset of type Sprite whose address is spriteAddress (a string like "char\_avatar\_01").”\
• LoadAssetAsync\<T>() is generic — you specify what type you want:\
• Sprite, GameObject, AudioClip, etc.\
• It returns an AsyncOperationHandle\<T>, which represents the in-progress loading job.

🔹 .Completed += OnSpriteLoaded;\
• This subscribes a callback function (OnSpriteLoaded) to the Completed event of that async load operation.\
• That means:\
“When loading finishes, run the method OnSpriteLoaded and give it the result.”\
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

Advanced code version:

```csharp
public class TestAddressableLoader : MonoBehaviour
{
    public string spriteAddress = "Assets/Arts/CharacterAvatars/Character 1.png";
    public Image targetImage;

    void Start()
    {
        LoadAvatar();
    }

    void LoadAvatar()
    {
        Addressables.LoadAssetAsync<Sprite>(spriteAddress).Completed += OnSpriteLoaded;
    }

    void OnSpriteLoaded(AsyncOperationHandle<Sprite> handle)
    {
        if (handle.Status == AsyncOperationStatus.Succeeded)
        {
            Sprite loadedSprite = handle.Result;
            targetImage.sprite = loadedSprite;
        }
        else
        {
            Debug.LogError("failed load avatar");
        }
    }
}
```
