# Player Controller

A custom player controller that you can use in any project.

### PlayerController

We will need several variables for this custom player controller which are as follows.

```csharp
[SerializeField] private LayerMask _groundMask;
[SerializeField] private Transform _groundCheck;
[SerializeField] private Animator _animator;

private float _groundCheckRadius = 0.3f;
[SerializeField] private float _speed = 8f;
[SerializeField] private float _maxSpeed = 8f;
[SerializeField] private float _turnSpeed = 1f;
[SerializeField] private float _maxTurnSpeed = 1f;
[SerializeField] private float _jumpForce = 8f;

private Rigidbody _rigidbody;
private Vector3 _direction;
```

The _groundMask is for telling the controller what the ground is, so it knows when its on the ground, which _groundCheck will be where the check will be positioned. _animator is just a reference to the animator on the player, but for this tutorial we wont be using it.

_groundCheckRadius is how far from the _groundCheck it will check for the ground.

_speed, _maxSpeed are used to determine how fast the character moves, _turnSpeed and _maxTurnSpeed is how fast the player can turn. _jumpForce is just how much force is applied when jumping.

_rigidbody is jsut a reference to the Rigidbody and _direction is just to store what direction the character is moving.

```csharp
private void Awake()
{
	_rigidbody = GetComponent<Rigidbody>();
}
```

The first few lines of code will just be assigning the variables to components on the character inside of the awake function.

```csharp
public void OnMove(InputAction.CallbackContext context)
{
	_direction = context.ReadValue<Vector2>().normalized;
}
```

This function is just called from unitys new input system, and when the player presses on a key it sets the _direction to the value of the output of the Vector2 (which in our case is WASD).

```csharp
public void OnJump(InputAction.CallbackContext context)
{
	bool isGrounded = Physics.CheckSphere(_groundCheck.position, _groundCheckRadius, _groundMask);

	if (context.performed && isGrounded)
	{
		_rigidbody.AddForce(Vector3.up * _jumpForce, ForceMode.Impulse);
	}
}
```

The OnJump function is the same as the OnMove function, but it is just called when the player presses space in our case, which will check to see if the player is on the ground or not, and if they are then add a force in the up direction of the Rigidbody. 

We will then need to create the FixedUpdate function that is built into unity to handle all the physics calculations we need to make.

```csharp
bool isRunning = _direction.magnitude > 0.1f;
bool isTurning = _direction.x > 0.1f || _direction.x < -0.1f;
```

There are 2 local bools we need to create just to check if the player is moving or not, which is just checking to see if the _direction is greater then a certain value.

```csharp
if (isRunning)
{
	Vector3 direction = Vector3.forward * _direction.y;
	float maxForce = (Mathf.Max(0.0f, _maxSpeed - _rigidbody.velocity.magnitude) * _rigidbody.mass) / Time.fixedDeltaTime;
	_rigidbody.AddRelativeForce(Mathf.Min(maxForce, _speed) * direction);
}
```

This checks if your running, and then applies a relativeforce to the rigidbody of whichever is lower out of the _speed and maxForce which is then multiplied by the direction. MaxForce is calculated by getting which ever is bigger out of 0 and the _maxSpeed minus the rigidbodys velocity times the mass.

```csharp
if (isTurning)
{
	_rigidbody.AddRelativeTorque(Vector3.up * _turnSpeed * _direction.x, ForceMode.Impulse);
	if (_rigidbody.angularVelocity.magnitude > _maxTurnSpeed)
	{
		_rigidbody.angularVelocity = Vector3.ClampMagnitude(_rigidbody.angularVelocity, _maxTurnSpeed);
	}
}
else
{
	_rigidbody.angularVelocity = Vector3.zero;
}
```

This checks to see if your turning, which if its true will add a torque force to the character in the direction that it receives from the players input. If its larger then the max speed it will clam it to that value as you don’t want to increase in rotation speed as your turning. But if your then let go, and stop turning it sets the turn velocity back to 0.

### Setting up the GameObjects

You will first have to create a Capsule from the 3D Object menu, after you right click in the hierarchy, and name it player.

![Untitled](Images/Player%20Controller/Untitled.png)

We then need to create a empty gameobject as a child by right clicking on the capsule and clicking on create empty, then name it ground check.

![Untitled](Images/Player%20Controller/Untitled%201.png)

---

We then want to add a rigidbody and player input. Then the script we made player controller.

Inside the rigidbody you will need to set a few things.

Drag = 0.5

Angular Drag = 5

Collision Detection = continuous

Constraints: Rotation x, z = true

On the player input component you will need to click create actions and select somewhere in the project to save it. Then it should bring up a window which looks like this.

![Untitled](Images/Player%20Controller/Untitled%202.png)

After we have done those changes we just need to click the save asset button at the top.

Under the player input component we now need to change the behavior to invoke unity events, then click on events and player underneath that.

![Untitled](Images/Player%20Controller/Untitled%203.png)

![Untitled](Images/Player%20Controller/Untitled%204.png)

We need to add one more action to this list and call it jump, then where it says no binding, click on it and then change the path to whatever you want jump to be, so in this case I set it to be space.

![Untitled](Images/Player%20Controller/Untitled%205.png)

We need to assign a function to the move and jump by clicking the plus on each of them, and then dragging the player from the hierarchy into each of them.

![Untitled](Images/Player%20Controller/Untitled%206.png)

Where it says no function we need to click on that, select player controller and then the function we want to call, so in this case it will be the OnMove function we created, and the OnJump function.

![Untitled](Images/Player%20Controller/Untitled%207.png)

---

The next thing we need to set is in the player controller component we added. But before that we need to make sure we have a ground layer created. To do that go to the top right of unity and click on layers > edit layers then in one of the empty slots create a ground layer and a player layer.

![Untitled](Images/Player%20Controller/Untitled%208.png)

The last thing we need to do is select our ground check gameobject in the hierarchy and set its Y position to -1.

![Untitled](Images/Player%20Controller/Untitled%209.png)

Then click back on the player in the hierarchy, and navigate back to the player controller component. Where it says ground mask, click it and select ground from the list. Then were it says ground check, drag the ground check game object that we created earlier into that slot.

![Untitled](Images/Player%20Controller/Untitled%2010.png)
