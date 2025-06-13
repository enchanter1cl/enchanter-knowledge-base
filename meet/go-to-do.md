---
icon: explosion
---

# GO TO DO

* [ ] This is an unchecked task
* [x] This is a checked task

June 11 2025

*   [x] **Script to Always Start Play Mode from the PersistentManagers scene.**

    Ensures Unity automatically loads `PersistentManagers` scene before entering Play Mode, regardless of the currently open scene.

    For example:

    If you are working in the `CharacterChoose` scene and enter Play Mode, only the `PersistentManagers` and `MainMenu` scenes will be loaded in the Hierarchy.

    When you exit Play Mode, the editor will return to the original scene (`CharacterChoose`) as it was before.
* [x] Solve problem of duplicate event systems
* [x] Navigation bar - mainMenu
* [x] Other scene switch logic
* [x] CharacterChoose - merge mine and berke's code

June 12

* [x] `EditorPlayBootstrapper.cs`explaination note
*   [x] CharacterChoose code merge

    * [x] fix: The event OnFlipped fired but callback didn't invoked. bkz `OnFlipped = null`; \[Problem solved. Bug reason: event invoked by CardRoot gameObject but subscription is in CardController component.]



June 13

* [ ] ScriptableObject containing the list of all characters.

