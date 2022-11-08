# Basic Dialogue System

### Narration Line

All this script is doing is holding information on who’s speaking, and a array of lines the character is going to say.

```csharp
using UnityEngine;

[CreateAssetMenu(menuName = "Scriptable Objects/Narration/Narration Lines", fileName = "new Narration Lines")]
public class NarrationLine : ScriptableObject
{
    [field: SerializeField] public string Speaker;
    [field: SerializeField] public string[] Dialogue;
}
```

### Narration Set

This just holds how many people are going to be talking, for instance if you had 2 people talking, it would have references to both of the characters.

```csharp
using UnityEngine;

[CreateAssetMenu(menuName = "Scriptable Objects/Narration/Narration Set", fileName = "new Narration Set")]
public class NarrationSet : ScriptableObject
{
    [field: SerializeField] public NarrationLine[] Lines;
}
```

### Narration Manager

All this script is checking is if the player goes into a certain area it checks to see if there is any dialogue to start. If so then it calls the NarrationBody to start the narration.

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class NarraionManager : MonoBehaviour
{
    private void OnTriggerEnter(Collider other)
    {
        if (other.TryGetComponent<NarrationBody>(out NarrationBody body))
        {
            body.StartNarration();
        }
    }
}
```

### Narration Body

In this script is where the speech gets displayed, and where the player can interact with the dialogue.

At the top of the script make sure you have a using of TMPro. As we will be using this later.

```csharp
using TMPro;
```

First you need to make a script called NarrationBody and make 6 variables.

```csharp
public GameObject SpeechBubble;
public TextMeshProUGUI CharacterName;
public TextMeshProUGUI Dialogue;
public NarrationSet Set;

private int _currentSpeaker = 0;
private int _currentDialogue = 0;
```

SpeechBubble is just a reference to the prefab with text for the name of who’s speaking and text for what that character is saying. The 2 TextMeshProUGUI variables are to store a reference to the text in the SpeachBubble prefab, and the NarrationSet is a reference to what dialogue that is going to be displayed.

Inside of the update function all we need to do is check if the player has triggered the speech bubble to appear and if they press space it will go to the next line in the dialogue.

```csharp
if (Input.GetKeyDown(KeyCode.Space) && SpeachBubble.activeInHierarchy)
{
	NextNarration();
}
```

We will then create 3 functions, called StartNarration, NextNarration and EndNarration. These are what will be used to update the text within the text bubble.

```csharp
public void StartNarration()
{

}

public void NextNarration()
{

}

public void EndNarration()
{

}
```

Inside of StartNarration there are only 3 lines of code we will need. One to activate the text bubble able the characters head. One to set the name to the first one speaking and one to display the speech of the character.

```csharp
SpeachBubble.SetActive(true);
CharacterName.text = Set.Lines[_currentSpeaker].Speaker;
Dialogue.text = Set.Lines[_currentSpeaker].Dialogue[_currentDialogue];
```

The next one we will do is the EndDialogue function as all this needs to do is set the current speaker back to 0, as one is speaking, and the same with the dialogue. We then need to disable the GameObject.

```csharp
_currentDialogue = 0;
_currentSpeaker = 0;
SpeachBubble.SetActive(false);
```

The last part we will need it the NextNrration function which will just cycle through all of the dialogues and keep track of which one is next. All this does is increase the _currentDialogue by 1, and checks if it is within the dialogue for that character. If it isn’t then it will go to the next if statement and check if the _currentSpeaker is within the length, and if it is it will show the next dialogue.

If the _currentDialogue is more then there is in the current speaker, it will set the _currentDialogue back to 0 and increate the value of _currentSpeaker by 1, to signify that the next character is ready to speak.

```csharp
_currentDialogue++;
if (Set.Lines[_currentSpeaker].Dialogue.Length <= _currentDialogue)
{
	_currentSpeaker++;
	_currentDialogue = 0;
}

if (Set.Lines.Length <= _currentSpeaker)
{
	EndNarration();
}
CharacterName.text = Set.Lines[_currentSpeaker].Speaker;
Dialogue.text = Set.Lines[_currentSpeaker].Dialogue[_currentDialogue];
```
