# Custom Gravity

Create the object you want to orbit, and create a collider around that object.

### GravityBody

To make custom gravity you first need to disable gravity either on the Rigidbody on the GameObject you are going to be using, or go to Edit > Project Settings > Physics, and then set gravity to 0, which will disable it for every physics body in your game.

Before we start anything, we need to make sure that the GameObejct we are adding this script to has a RigidBody and we can do this before the class starts, you can require a component.

```csharp
[RequireComponent(typeof(Rigidbody))]
```

To get the custom Gravity now, you will need to make a script and name it “GravityBody” and open the script. You will need 4 variables, GravityForce which is a float, GravityVelocity which will be a Vector3, a RigidBody and a list of GravityAreas (Which we will get to later).

```csharp
public float GravityForce { get; private set; }
public Vector3 GravityVelocity { get; private set; }
private Rigidbody _rigidbody;
private List<GravityArea> _gravityAreas;
```

Another variable you will need is GravityDirection which will be a method which will return the type Vector3. The method will get all the current gravity areas you are in, and sort them with their priorities and use the one with the highest and will return this to the variable.

```csharp
public Vector3 GravityDirection
{
	get
	{
		if (_gravityAreas.Count == 0) 
		{
			return Vector3.zero;
		}
		_gravityAreas.Sort((area1, area2) => area1.Priority.CompareTo(area2.Priority));
		return _gravityAreas.Last().GetGravityDirection(this).normalized;
	}
}
```

In your start function you will need to add 2 lines of text which sets your RigidBody to your current RigidBody and you will need to initialise the list of GravityAreas.

```csharp
_rigidbody = transform.GetComponent<Rigidbody>();
_gravityAreas = new List<GravityArea>();
```

We will then need to create a FixedUpdate function, as this should always be used to do physics calculations. Inside of this, we need to calculate a few things, the GravityVelocity and then apply it to the RigidBody, and then rotate the RigidBody to face the correct direction.

```csharp
void FixedUpdate()
{
	GravityVelocity = GravityDirection * (GravityForce * Time.fixedDeltaTime);
	_rigidbody.AddForce(GravityVelocity, ForceMode.Acceleration);

	Quaternion upRotation = Quaternion.FromToRotation(transform.up, -GravityDirection);
	Quaternion newRotation = Quaternion.Slerp(_rigidbody.rotation, upRotation * _rigidbody.rotation, Time.fixedDeltaTime * 3f); ;
	_rigidbody.MoveRotation(newRotation);
}
```

The last thing we need to add is a way to set the gravity and a way to add and remove GravityAreas when you enter and leave a zone

```csharp
public void AddGravityArea(GravityArea gravityArea)
{
	_gravityAreas.Add(gravityArea);
}

public void RemoveGravityArea(GravityArea gravityArea)
{
	_gravityAreas.Remove(gravityArea);
}

public void SetGravity(float SetToGravity)
{
	GravityForce = SetToGravity;
}
```

### GravityArea

Another script we will need to create is called “GravityArea” which will also require the component of Collider.

```csharp
[RequireComponent(typeof(Collider))]
```

You will need 3 variables, one to store the priority of the area, one to set how much gravity to apply, and a public Priority which will get the private one we set earlier.

```csharp
[SerializeField] private int _priority;
public float Gravity = 800;
public int Priority => _priority;
```

In the start method we just need one line of code. Which will get the component of Collider and just make sure the collider on it, is set to true.

```csharp
transform.GetComponent<Collider>().isTrigger = true;
```

This just declares a function that can be used later by other classes that inherit this class of GravityArea.

```csharp
public abstract Vector3 GetGravityDirection(GravityBody _gravityBody);
```

We then need to create a function for OnTriggerEnter which will get fired anytime you enter a new area. We then have a if statement checking if the collider we go into, has a GravityBody, and only continues if it’s true. It then sets the GravityArea to the current one you’r in, and sets the gravity.

```csharp
private void OnTriggerEnter(Collider other)
{
	if (other.TryGetComponent(out GravityBody gravityBody))
	{
		gravityBody.AddGravityArea(this);
		gravityBody.SetGravity(Gravity);
	}
}
```

We also have to do this when leaving a area, but instead of adding we remove the area.

```csharp
private void OnTriggerExit(Collider other)
{
	if (other.TryGetComponent(out GravityBody gravityBody))
	{
		gravityBody.RemoveGravityArea(this);
	}
}
```

### GravityAreaCenter

Used for things like planets and were you want to go around a object.

```csharp
public class GravityAreaCenter : GravityArea
{
	public override Vector3 GetGravityDirection(GravityBody _gravityBody)
	{
		return (transform.position - _gravityBody.transform.position).normalized;
	}
}
```

Calculates the GetGravityDirection by taking the gravitybodys position away from whatever this script is on.

### GravityAreaInversePoint

Where you are attracted to a single point that is defined by the transform

```csharp
public class GravityAreaInversePoint : GravityArea
{
[SerializeField] private Vector3 _center;

	public override Vector3 GetGravityDirection(GravityBody _gravityBody)
	{
		return (_gravityBody.transform.position - _center).normalized;
	}
}
```

Calculates the GetGravityDirection by taking the _center position away from where the gravityBody is. 

### GravityAreaInversePoint

Where you are pushed away from a single point that is defined by the transform

```csharp
public class GravityAreaPoint : GravityArea
{
[SerializeField] private Vector3 _center;

	public override Vector3 GetGravityDirection(GravityBody _gravityBody)
	{
		return (_center - _gravityBody.transform.position).normalized;
	}
}
```

Calculates the GetGravityDirection by taking the gravitybodys position away from the _center position. Meaning it will push the player away from the _center.

### GravityAreaUp

Works like normal gravity where you are attracted down

```csharp
public class GravityAreaUp : GravityArea
{
	public override Vector3 GetGravityDirection(GravityBody _gravityBody)
	{
		return -transform.up;
	}
}
```

Sets the gravity to the opposite of the objects up position.

---

### Setting up the GameObjects

First you need add a sphere game object into the hierarchy the name it Planet and then right click on that object and do the same again, but not is gravity area.

![Untitled](Images/Custom%20Gravity/Untitled.png)

Set the childs transform scale to 1.5 for the X, Y, Z so its slightly larger then the parent, and remove the mesh renderer of the child gameObject as well. Then on the parent set the scale to 10 on the X, Y, Z (Or how ever large you want it to be). So when you select the parent sphere it looks something like this in the scene.

![Untitled](Images/Custom%20Gravity/Untitled%201.png)

On the gameobject Gravity Area add the component GravityAreaCenter to it.

![Untitled](Images/Custom%20Gravity/Untitled%202.png)

Next we just need to create a new gameobject that we will test out this new gravity system on. So we will create a cube the same way we made the sphere and position it so its just above the sphere

![Untitled](Images/Custom%20Gravity/Untitled%203.png)

On the cube we are going to add the gravitybody component that we created earlier, which will interact with the gravity area which we just made, and also add a rigidbody. 

On the rigidbody set the following:

Drag = 0.5

Angular Drag = 5

Collision Detection = Continuous

But for this to work we need to turn off gravity in the project. For that go to Edit > Project Settings > Physics, then make sure gravity is 0, and cloth gravity is also 0 (if you still want gravity but only want certain object to interact with the new gravity you can leave that how it is, but just turn gravity off per rigidbody).

Now when you click play, you should see the cube fall towards the planet, and if you try to move it, it will go back to the planet and realign itself as well.