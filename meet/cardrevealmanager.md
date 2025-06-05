# CardRevealManager

local fields:

```csharp
Character characterToShow;
AsyncOperationHandle currentAvatarLoadHdl;
```

randomly select one person.\
core line: Character original = characterListSO.characters\[randomIndex];

```csharp
bool SelectCharacterForRound()
    {
        if (characterListSO.characters == null || characterListSO.characters.Count == 0)
        {
            Debug.LogError($"CharacterListSO is empty or not assigned.", this);
            return false;
        }
        System.Random rng = new System.Random();
        int randomIndex = rng.Next(characterListSO.characters.Count);
        Character original = characterListSO.characters[randomIndex];

        // create a runtime copy to avoid modifying the SO directly
        characterToShow = new Character(
            original.cname, original.age, original.city, original.profession, original.hobby, original.school, original.favoriteColor, original.avatar_identifier, original.difficulty
        );
        return true;
    }
```

async load this character's sprite avatar:

```csharp
    async Task<bool> LoadSingleAvatarAsync()
    {
        if (characterToShow == null)
        {
            Debug.LogError("Cannot load avatar, characterToShow == null");
            return false;
        }
        ClearAddressables(); //Clear any previous handles
        if (!string.IsNullOrEmpty(characterToShow.avatar_identifier)) 
        {
            string identifier = characterToShow.avatar_identifier;
            currentAvatarLoadHdl = Addressables.LoadAssetAsync<Sprite>(identifier);

            //wait for the load operation to complete
            await handle.Task;

            if (currentAvatarLoadHdl.Status == AsyncOperationStatus.Succeeded) 
            {
                characterToShow.avatar = currentAvatarLoadHdl.Result;
                return true;
            }
            else
            {
                Debug.LogError($"failed to load avatar: {identifier}");
                characterToShow.avatar = null;
                return false;
            }
        }
        else
        {
            Debug.LogWarning($"{characterToShow.cname} has no avatar_identifier");
            characterToShow.avatar = null;
            return true; // Considered success as no avatar was required
        }
    }
```

core:\
AsyncOperationHandle\<Sprite> currentAvtarLoadHdl;\
currentAvatarLoadHdl= Addressables.LoadAssetAsync\<Sprite>(identifier);

## HandleCardClick

param: the cardController instance that be clicked

```csharp
    public void HandleCardClick(CardController clickedCard)
    {
        if (isLoading || !cardsReady || revealedCard != null) return;
        if (characterToShow == null)
        {
            Debug.LogError("characterToShow == null");
            return;
        }
        this.clickedCard = clickedCard;
        clickedCard.Reveal(characterToShow);

        // Optional: Disable ALL cards
        if (cardContainer != null)
        {
            foreach (Transform cardTransform in cardContainer)
            {
                Button btn = cardTransform.GetComponent<Button>();
                if (btn != null)
                {
                    btn.interactable = false;
                }
            }
        }
        else
        {
            Debug.LogError("Card Container missing");
        }
        // Start the delay coroutine before proceeding
        StartCoroutine(ProceedAfterRevealDelay(characterToShow));
    }
```
