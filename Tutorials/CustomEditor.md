# Custom Editor

To make a custom editor, you will already need a script with some variables so we can make the custom editor to display something.

### Example Script

```csharp
using UnityEngine;

public class ExampleScript : MonoBehaviour
{
	[SerializeField] private float _testFloat;
	[SerializeField] private Vector3 _testVector3;
	[SerializeField] private string _testString;
}
```

Create a script in unity and just add some variables at the top of the script. They don’t have to do anything, as this is just to show what you could do. Then add it onto any gameobject.

![Untitled](Images/Custom%20Editor/Untitled.png)

Now create another script with the same name then add Editor to the end, so in my example my example was “ExampleScript” so it would be called “ExampleScriptEditor”.

```csharp
using UnityEditor;

[CustomEditor(typeof(ExampleScript))]
public class ExampleScriptEditor : Editor
{
    
}
```

Instead of extending from MonoBehaviour, we will extend this script from Editor and make this a custom editor of type ExampleScript. Which will tell unity, this is a custom editor script, and we want to change how the editor looks for the “ExampleScript”.

Inside of the class we need to have 2 functions one called OnEnable, and one called OnInspectorGUI.

```csharp
private void OnEnable()
{

}

public override void OnInspectorGUI()
{
	serializedObject.Update();

	serializedObject.ApplyModifiedProperties();
}
```

This is the base script you will need for any custom editor for unity. We now need to set some variables in here as well. Which are the type of SerializedProperty. meaning these properties are ones we want to be able to see.

```csharp
SerializedProperty testFloat;
SerializedProperty testVector3;
SerializedProperty testString;
```

In the OnEnable function we need to set these variables to their values in the script. So we need to get the serializedObject and find the property we were looking for then assign it to the variable.

```csharp
testFloat = serializedObject.FindProperty("_testFloat");
testVector3 = serializedObject.FindProperty("_testVector3");
testString = serializedObject.FindProperty("_testString");
```

Inside the OnInspectorGUI function between the update, and apply, you need to now draw these properties to the editor. You can do this by doing EditorGUILayout.PropertyField(), then pass in what you want it to add, for our example, we would write.

```csharp
EditorGUILayout.PropertyField(testFloat);
EditorGUILayout.PropertyField(testVector3);
EditorGUILayout.PropertyField(testString);
```

Then once that.s done, save it and go back to unity, we should see the 3 properties there. But you might be thinking, can’t we do the same with just a script, and you would be right thinking that, but this just separates the editor side from the code, so if you want to test a function, you can add buttons to the editor to do that.

![Untitled](Images/Custom%20Editor/Untitled%201.png)

To do that we need to create a if statement, which will check if the user presses on the button called “Test” and then call the function that is inside of the if statement. In our case it will just print out a log saying “Test”. 

```csharp
if (GUILayout.Button("Test"))
{
	Debug.Log("Test");
}
```

But this could be used to call functions that heal the player, or make it so the player takes damage, so you can test specific aspects of your game when debugging. Then because its on the editor, none of it will be built when you finish your game and  publish it.