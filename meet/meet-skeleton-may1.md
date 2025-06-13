# MEET SKELETON MAY1

| **Module / System**              | **Responsibility**                                                | **Script(s)**                             | **Folder Structure**   | **Dependencies**                      |
| -------------------------------- | ----------------------------------------------------------------- | ----------------------------------------- | ---------------------- | ------------------------------------- |
| **Game Management**              | Controls game flow, scene loading, pause/restart logic            | GameManager.csSceneLoader.cs              | /Scripts/Core/         | UnityEngine.SceneManagement           |
| **Difficulty & Character DB**    | Difficulty selection; loading character pool; ScriptableObject DB | DifficultyManager.csCharacterDB.cs        | /Scripts/Data/         | Character ScriptableObject assets     |
| **UI Management**                | Main Menu, Settings, No-Ads, dynamic texts/colors                 | UIManager.csSettingsManager.cs            | /Scripts/UI/           | Unity UI, (optional) DOTween          |
| **Card Animation**               | Card flip and collection animations                               | CardFlipController.csCardCollectAnim.cs   | /Scripts/UI/Anim/      | Unity Animator or DOTween             |
| **Character Info (“Remember”)**  | Display info panel; timer bar; trigger “Start Quiz”               | CharacterInfoUI.cs                        | /Scripts/GameFlow/     | UIManager, CharacterDB                |
| **Quiz Mechanic**                | Pull questions; answer validation; per-question timer             | QuizManager.csQuestionController.cs       | /Scripts/Quiz/         | CharacterDB, UIManager                |
| **Score Calculation**            | Implements BrainScore formula, difficulty & speed multipliers     | ScoreCalculator.cs                        | /Scripts/Scoring/      | QuizManager                           |
| **Result & Retry/Collection**    | Result screen UI; Retry vs. Collection button logic; stats update | ResultUIController.csStatsManager.cs      | /Scripts/Results/      | ScoreCalculator, DataPersistence      |
| **Collection & Leaderboard**     | Tracks 100% completions; displays global/friends rankings         | CollectionManager.csLeaderboardManager.cs | /Scripts/Social/       | AnalyticsManager, (future) Server API |
| **Achievement System**           | Tracks milestones; unlocks badges; popup management               | AchievementManager.cs                     | /Scripts/GameFlow/     | DataPersistence                       |
| **Ads & IAP**                    | Rewarded video ads; No-Ads IAP; UnityPurchasing integration       | AdsManager.csIAPManager.cs                | /Scripts/Monetization/ | Unity Ads, Unity Purchasing           |
| **Data Persistence & Analytics** | Saves player data (PlayerPrefs/JSON); logs analytic events        | DataPersistence.csAnalyticsManager.cs     | /Scripts/Core/         | Unity Analytics, System.IO (for JSON) |

#### **Additional Best Practices & Tips**

1. **Folder Structure**
   * Keep /Scripts/ organized into clear subfolders by feature/module.
   * Group animations, UI, data, and core code separately for clarity.
2. **Dependency Injection & Singletons**
   * Expose managers (e.g. GameManager, QuizManager) as singletons or via DI for easy access and testing.
3. **SOLID Principles**
   * **Single Responsibility:** Each script does one thing.
   * **Open/Closed:** Add new difficulties or animations without altering existing code.
4. **Priority Order**
   1. Core (GameManager & SceneLoader)
   2. Data (CharacterDB & DifficultyManager)
   3. UI & Animations (UIManager & CardFlipController)
   4. Gameplay (QuizManager, ScoreCalculator, ResultUIController)
   5. Social & Monetization (LeaderboardManager, AdsManager, IAPManager)
5. **Rough Timeline (MVP)**
   * **Week 1:** Core & Data modules
   * **Week 2:** UI & Animations
   * **Week 3:** Quiz & Scoring
   * **Week 4:** Result, Leaderboard & Achievements
   * **Week 5:** Ads/IAP, Analytics, Testing & Optimization

```csharp
// =========================================
// File: GameManager.cs
// =========================================
using UnityEngine;
using UnityEngine.SceneManagement;

public class GameManager : MonoBehaviour
{
    public static GameManager Instance { get; private set; }

    private void Awake()
    {
        if (Instance == null)
        {
            Instance = this;
            DontDestroyOnLoad(gameObject);
        }
        else
        {
            Destroy(gameObject);
        }
    }

    public void RestartCurrentScene()
    {
        SceneLoader.Load(SceneManager.GetActiveScene().name);
    }
}

```

```csharp
// =========================================
// File: SceneLoader.cs
// =========================================
using UnityEngine.SceneManagement;

public static class SceneLoader
{
    public static void Load(string sceneName)
    {
        SceneManager.LoadScene(sceneName);
    }
}
```

```csharp
// =========================================
// File: DifficultyManager.cs
// =========================================
using UnityEngine;

public enum Difficulty { Easy, Medium, Hard }

public class DifficultyManager : MonoBehaviour
{
    public static DifficultyManager Instance { get; private set; }
    public Difficulty CurrentDifficulty { get; private set; }

    private void Awake()
    {
        if (Instance == null) Instance = this;
        else Destroy(gameObject);
        DontDestroyOnLoad(gameObject);
    }

    public void Select(Difficulty difficulty)
    {
        CurrentDifficulty = difficulty;
        SceneLoader.Load("CharacterChoose");
    }
}
```

// =========================================\
// File: CharacterDB.cs\
// =========================================\
using UnityEngine;\
using System.Collections.Generic;

\[CreateAssetMenu(menuName = "Database/CharacterDB")]\
public class CharacterDB : ScriptableObject\
{\
public List characters;

```
public List<CharacterData> GetPool(Difficulty diff)
{
    return characters.FindAll(c => c.difficulty == diff);
}

public static Sprite GetAvatar(int id)
{
    var data = Instance.characters.Find(c => c.id == id);
    return data != null ? data.avatar : null;
}

public static CharacterDB Instance;
private void OnEnable() => Instance = this;
```

}

\[System.Serializable]\
public class CharacterData\
{\
public int id;\
public Difficulty difficulty;\
public Sprite avatar;\
public string characterName;\
public int age;\
public string hometown;\
public string education;\
public string personality;\
public string hobby;\
public string favoriteColor;\
public string strongestTrait;\
public string specialItem;\
public string favoriteFood;\
public List questions;\
}

\[System.Serializable]\
public class QuestionData\
{\
public string question;\
public List options;\
public int correctIndex;\
}

// =========================================\
// File: UIManager.cs\
// =========================================\
using UnityEngine;\
using UnityEngine.UI;

public class UIManager : MonoBehaviour\
{\
public static UIManager Instance { get; private set; }\
private void Awake() { if (Instance==null) Instance=this; else Destroy(gameObject); }

```
public void OnNoAdsClicked() { /* Show IAP */ }
public void OnSettingsClicked() { SceneLoader.Load("Settings"); }
```

}

// =========================================\
// File: SettingsManager.cs\
// =========================================\
using UnityEngine;

public class SettingsManager : MonoBehaviour\
{\
// Volume, language, toggles...\
public void SetVolume(float value) { AudioListener.volume = value; }\
}

// =========================================\
// File: CardFlipController.cs\
// =========================================\
using UnityEngine;\
using UnityEngine.UI;\
using DG.Tweening;

public class CardFlipController : MonoBehaviour\
{\
public RectTransform front, back;\
public Button button;\
private bool isFlipped;

```
private void Start()
{
    button.onClick.AddListener(Flip);
}

public void Setup(CharacterData data)
{
    // assign back visuals
}

void Flip()
{
    if (isFlipped) return;
    isFlipped = true;
    front.DORotate(new Vector3(0,90,0),0.3f).OnComplete(() =>
    {
        front.gameObject.SetActive(false);
        back.gameObject.SetActive(true);
        back.localRotation = new Vector3(0,-90,0);
        back.DORotate(Vector3.zero,0.3f).OnComplete(() =>
        {
            // Animation done, proceed
            CharacterInfoUI.Instance.LoadInfo(/* id */);
        });
    });
}
```

}

// =========================================\
// File: CardCollectAnim.cs\
// =========================================\
using UnityEngine;

public class CardCollectAnim : MonoBehaviour\
{\
public void Collect(Transform target)\
{\
transform.DOMove(target.position,0.5f).OnComplete(() => Destroy(gameObject));\
}\
}

// =========================================\
// File: CharacterInfoUI.cs\
// =========================================\
using UnityEngine;\
using UnityEngine.UI;

public class CharacterInfoUI : MonoBehaviour\
{\
public static CharacterInfoUI Instance;\
public Slider timerBar;\
public Text nameField, ageField, hometownField, educationField;\
// ... other fields\
public Button startQuizButton;\
private float timeRemaining;

```
private void Awake() { Instance=this; }
public void LoadInfo(CharacterData data)
{
    nameField.text = data.characterName;
    ageField.text = data.age.ToString();
    hometownField.text = data.hometown;
    // ... assign all fields
    timeRemaining = QuizManager.Instance.QuestionTime * data.questions.Count;
    startQuizButton.onClick.AddListener(() => QuizManager.Instance.StartQuiz(data));
}

private void Update()
{
    if (timeRemaining > 0)
    {
        timeRemaining -= Time.deltaTime;
        timerBar.value = timeRemaining / (QuizManager.Instance.QuestionTime * QuizManager.Instance.TotalQuestions);
        if (timeRemaining <= 0) startQuizButton.interactable = true;
    }
}
```

}

// =========================================\
// File: QuizManager.cs\
// =========================================\
using UnityEngine;\
using System.Collections.Generic;

public class QuizManager : MonoBehaviour\
{\
public static QuizManager Instance;\
public float QuestionTime = 10f;\
public int TotalQuestions { get; private set; }\
private List questions;\
private int currentIndex;

```
private void Awake() { Instance=this; }

public void StartQuiz(CharacterData data)
{
    questions = new List<QuestionData>(data.questions);
    TotalQuestions = questions.Count;
    currentIndex = 0;
    ShowQuestion();
}

void ShowQuestion()
{
    var q = questions[currentIndex];
    QuestionController.Instance.Setup(q, currentIndex+1, TotalQuestions);
}

public void RegisterAnswer(int index)
{
    bool correct = questions[currentIndex].correctIndex == index;
    ScoreCalculator.Instance.RecordAnswer(correct, remainingTime);
    currentIndex++;
    if (currentIndex < TotalQuestions) ShowQuestion();
    else FinishQuiz();
}

public void Timeout()
{
    ScoreCalculator.Instance.RecordAnswer(false, 0);
    RegisterAnswer(-1);
}
```

}

// =========================================\
// File: QuestionController.cs\
// =========================================\
using UnityEngine;\
using UnityEngine.UI;

public class QuestionController : MonoBehaviour\
{\
public static QuestionController Instance;\
public Text questionText;\
public Button\[] optionButtons;\
public Text\[] optionTexts;

```
private void Awake() { Instance=this; }

public void Setup(QuestionData data, int index, int total)
{
    questionText.text = $"{data.question}";
    for (int i=0; i<optionButtons.Length; i++)
    {
        optionTexts[i].text = data.options[i];
        int idx = i;
        optionButtons[i].onClick.RemoveAllListeners();
        optionButtons[i].onClick.AddListener(() => QuizManager.Instance.RegisterAnswer(idx));
    }
}
```

}

// =========================================\
// File: ScoreCalculator.cs\
// =========================================\
using UnityEngine;\
using System.Collections.Generic;

public class ScoreCalculator : MonoBehaviour\
{\
public static ScoreCalculator Instance;\
private List timeRatios = new List();\
private int correctCount;

```
private void Awake() { Instance=this; }

public void RecordAnswer(bool correct, float timeLeft)
{
    if (correct) correctCount++;
    timeRatios.Add(timeLeft / QuizManager.Instance.QuestionTime);
}

public float CalculateTotal()
{
    int N = QuizManager.Instance.TotalQuestions;
    float basePoint = 100f / N;
    float mult = (DifficultyManager.Instance.CurrentDifficulty == Difficulty.Easy) ? 1f :
                  DifficultyManager.Instance.CurrentDifficulty == Difficulty.Medium ? 1.2f : 1.5f;
    float total = 0;
    for (int i = 0; i < N; i++)
    {
        if (i < correctCount)
        {
            float speedBonus = 1 + timeRatios[i];
            total += basePoint * mult * speedBonus;
        }
    }
    return total;
}
```

}

// =========================================\
// File: ResultUIController.cs\
// =========================================\
using UnityEngine;\
using UnityEngine.UI;

public class ResultUIController : MonoBehaviour\
{\
public Text brainScoreText;\
public Slider progressBar;\
public Button retryButton, collectionButton, closeButton;

```
private void Start()
{
    float score = ScoreCalculator.Instance.CalculateTotal();
    float max = 100f * ((DifficultyManager.Instance.CurrentDifficulty == Difficulty.Easy) ? 2f :
                         DifficultyManager.Instance.CurrentDifficulty == Difficulty.Medium ? 2.4f : 3f);
    float pct = (score / max) * 100f;
    brainScoreText.text = $"+{Mathf.RoundToInt(score)} BRAIN SCORE";
    progressBar.value = pct/100f;

    bool perfect = pct >= 100f;
    collectionButton.gameObject.SetActive(perfect);
    retryButton.gameObject.SetActive(!perfect);

    retryButton.onClick.AddListener(() => SceneLoader.Load("CharacterChoose"));
    collectionButton.onClick.AddListener(() => SceneLoader.Load("Collection"));
    closeButton.onClick.AddListener(() => SceneLoader.Load("MainMenu"));

    StatsManager.Instance.SaveSession(score, pct, QuizManager.Instance.TotalQuestions);
}
```

}

// =========================================\
// File: StatsManager.cs\
// =========================================\
using UnityEngine;\
using System.Collections.Generic;

public class StatsManager : MonoBehaviour\
{\
public static StatsManager Instance;\
private List pastScores = new List();\
public SessionData LastSession;

```
private void Awake() { Instance=this; }

public void SaveSession(float score, float pct, int qCount)
{
    LastSession = new SessionData { BrainScore=score, AccuracyPercent=pct, QuestionCount=qCount, Difficulty=DifficultyManager.Instance.CurrentDifficulty };
    pastScores.Add(score);
    if (pastScores.Count > 20) pastScores.RemoveAt(0);
}

public SessionData GetLastSession() => LastSession;
public float GetHighest() => pastScores.Count>0 ? Mathf.Max(pastScores.ToArray()) : 0;
public float GetAverage() => pastScores.Count>0 ? Mathf.Sum(pastScores.ToArray())/pastScores.Count : 0;
```

}

public class SessionData\
{\
public float BrainScore;\
public float AccuracyPercent;\
public int QuestionCount;\
public Difficulty Difficulty;\
}

// =========================================\
// File: CollectionManager.cs\
// =========================================\
using UnityEngine;\
using System.Collections.Generic;

public class CollectionManager : MonoBehaviour\
{\
private HashSet collected = new HashSet();

```
public void Unlock(int id)
{
    if (!collected.Contains(id)) collected.Add(id);
}

public List<int> GetCollected() => new List<int>(collected);
```

}

// =========================================\
// File: LeaderboardManager.cs\
// =========================================\
using UnityEngine;\
using System.Collections.Generic;

public class LeaderboardManager : MonoBehaviour\
{\
public void ShowGlobal() { /\* fetch from server _/ }_\
_public void ShowFriends() { /_ fetch friends list \*/ }\
}

// =========================================\
// File: AchievementManager.cs\
// =========================================\
using UnityEngine;

public class AchievementManager : MonoBehaviour\
{\
public void CheckMilestones() { /\* e.g. total meets, perfect plays _/ }_\
_public void Unlock(string key) { /_ show popup \*/ }\
}

// =========================================\
// File: AdsManager.cs\
// =========================================\
using UnityEngine;\
using UnityEngine.Advertisements;

public class AdsManager : MonoBehaviour, IUnityAdsListener\
{\
void Start() { Advertisement.AddListener(this); Advertisement.Initialize(""); }\
public void ShowRewarded() { if (Advertisement.IsReady("rewardedVideo")) Advertisement.Show("rewardedVideo"); }\
public void OnUnityAdsDidFinish(string surfacingId, ShowResult result) { /\* grant reward \*/ }\
public void OnUnityAdsReady(string surfacingId) {}\
public void OnUnityAdsDidError(string message) {}\
public void OnUnityAdsDidStart(string surfacingId) {}\
}

// =========================================\
// File: IAPManager.cs\
// =========================================\
using UnityEngine;\
using UnityEngine.Purchasing;

public class IAPManager : MonoBehaviour, IStoreListener\
{\
private static IStoreController storeController;\
void Start()\
{\
if (storeController == null)\
{\
var builder = ConfigurationBuilder.Instance(StandardPurchasingModule.Instance());\
builder.AddProduct("no\_ads", ProductType.NonConsumable);\
UnityPurchasing.Initialize(this, builder);\
}\
}\
public void BuyNoAds() { storeController.InitiatePurchase("no\_ads"); }\
public void OnInitialized(IStoreController controller, IExtensionProvider extensions)\
{ storeController = controller; }\
public void OnInitializeFailed(InitializationFailureReason error) {}\
public PurchaseProcessingResult ProcessPurchase(PurchaseEventArgs args)\
{\
if (args.purchasedProduct.definition.id == "no\_ads")\
PlayerPrefs.SetInt("NoAds",1);\
return PurchaseProcessingResult.Complete;\
}\
public void OnPurchaseFailed(Product product, PurchaseFailureReason reason) {}\
}

// =========================================\
// File: DataPersistence.cs\
// =========================================\
using UnityEngine;\
using System.IO;

public class DataPersistence : MonoBehaviour\
{\
private string path;\
private void Awake() { path = Path.Combine(Application.persistentDataPath, "save.json"); }\
public void Save(T data)\
{\
var json = JsonUtility.ToJson(data);\
File.WriteAllText(path, json);\
}\
public T Load() where T : new()\
{\
if (!File.Exists(path)) return new T();\
return JsonUtility.FromJson(File.ReadAllText(path));\
}\
}

// =========================================\
// File: AnalyticsManager.cs\
// =========================================\
using UnityEngine.Analytics;\
using System.Collections.Generic;

public class AnalyticsManager : MonoBehaviour\
{\
public void LogEvent(string name, Dictionary\<string, object> parameters = null)\
{\
Analytics.CustomEvent(name, parameters);\
}\
}
