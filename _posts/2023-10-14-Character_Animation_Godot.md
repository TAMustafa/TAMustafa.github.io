---
title: CharacterBody and Animation in Godot4
date: 2023-10-14 16:55:25 +500
categories: [Godot, Character]
tags: [godot, animation, character, blender, mixamo]
---

# Godot 4 - Characterbody and Animation

The purpose of this blog is to document my journey in becoming a (XR) game developer using the Godot engine.
In this post I show how I setup and animated my Blender model "Rocket Baby" in Godot 4.

Useful links with more detailed information and for future reference:

- [Godot Character Body](https://docs.godotengine.org/en/stable/tutorials/physics/using_character_body_2d.html)
- [Godot Animation Tree](https://docs.godotengine.org/en/stable/tutorials/animation/animation_tree.html)
- [Godot Input and Movement Overview](https://docs.godotengine.org/en/stable/tutorials/2d/2d_movement.html)

## Setup main scene

### Floor and object

The main scene consists of a floor for Rocket Baby to walk on and a simple box object to test the collision and camera arm.

The nodes used for this scene are:

- Static Body 3d with Mesh and Collision.
- [Kenny's Prototype texture](https://godotengine.org/asset-library/asset/781) for the floor and Box.
- The standard environment and light node.

  ![Simple main scene](/assets/img/GodotTBFloor.png){: w="700" h="400" }

## Setup player scene

The majority of the game functionality happens in this scene, since it is used to display, move and animate the Player (Rocket Baby).

### Import Blender model

I included [Mixamo animation](https://www.mixamo.com/#/) clips to the Rocket Baby model in Blender and exported it as a .glb file. Then I dragged the .glb file into the Godot Assets. From there I right clicked on the .glb file and created a **new inherited scene**.

### Setup Character Body

In order to make it easier to move the model, I changed the root node to a **Character Body 3d** node and added a **capsule collider**.

![Player node Setup](/assets/img/GodotRBPlayer.png){: w="700" h="400" }

### Setup Camera and Pivot Point

In the game I like to rotate the camera around Rocket Baby when he is standing still.

In order to implement this, I need a "pivot" point that is at the same location as Rocket Baby for the rotation and then a second point for the placing the camera.

Godot has a very nice node **SpringArm3d** that makes sure that RocketBaby stays in view even if an obstacle is blocking the view. This node is perfect to use place the camera as a child under it.

![Camera setup](/assets/img/GodotRBCamera.png){: w="700" h="400" }

### Setup Animation Tree / Animation nodes

Animations attached to the Model are stored in the **Animation** node but in order to use the Animations and tie it to input actions, an **Animation Tree** node need to be added.

Here you can create a **state machine** or different animation nodes in order to play animations once the provided conditions are met.

![Animation Tree](/assets/img/GodotRBAnimationTree.png){: w="700" h="400" }

## GDScript for movement and animation

The GDScript in the Player node ties all the various elements together. I used the Character Body Template script that is included with the the Node and added code for the camera rotation and animation.

### Variables defined at the start:

- Node references (@onready) for the Armature, Camera and Animation Tree.

- Camera variables to control the sensitivity and initial position.

- Speed and Jump velocity for the player movement speed and jump hight.

- Gravity as defines per Project settings.

### The \_ready method:

This method is called at the start of the game and will cause the mouse cursor to disappear.

### The \_unhandled_input method:

Will listen to all events.

- Close the game preview window when the escape button is pressed.

- Capture and store the Mouse motion values in order to use it to move the camera later on.

### The \_physics_process method:

Updates the state 60 times per second. Physics Process should be used for anything that involves the physics engine, like moving a body that collides with the environment.

- Adding Gravity if the player is in the air (not grounded).

- Handles the player jump / animation if the player is on the ground and the jump button (Space bar) is pressed.

- Capture the input direction as defined in the **Input Manager** and move the player along the X and Z axis by multiplying the velocity with the **SPEED** value provided.

- Smooth the movement when the player changes direction.

- **move_and_slide()** is a method that belongs to the CharacterBody3d node and handles the player movement based on the **velocity** values.

- Camera movement around the player (pivot) by using the mouse motion input variables stored in **spring_pivot** and **spring**.

- Manages the **\*Idle** and **Run** animation based on if the player is standing still (value 0) or moving (value > 0).

## The complete GDScript code

```cs
extends CharacterBody3D

@onready var armature = $Armature
@onready var spring_arm_pivot = $SpringArmPivot
@onready var spring_arm = $SpringArmPivot/SpringArm3D
@onready var animator = $AnimationTree
@onready var playback = animator["parameters/playback"]
var walk_blend := "parameters/Walk/blend_position"


var camera_sensitivity := 0.0020
var spring_pivot = 0.0
var spring = 0.0


const SPEED = 4.0
const JUMP_VELOCITY = 2.5

# TM - Use for Armature Lerp
const LERP_VAL= 0.15

# Get the gravity from the project settings to be synced with RigidBody nodes.
var gravity = ProjectSettings.get_setting("physics/3d/default_gravity")


func _ready():
	# TM - Hide mouse cursor on start
	Input.set_mouse_mode(Input.MOUSE_MODE_CAPTURED)


func _unhandled_input(event):
	# TM - Exit window with ESC button
	if Input.is_action_just_pressed("ui_cancel"):
		get_tree().quit()

	# TM - Capture and store mouse movement values
	if event is InputEventMouseMotion:
		spring_pivot = (-event.relative.x * camera_sensitivity)
		spring = (-event.relative.y * camera_sensitivity)


func _physics_process(delta):
	# Add the gravity.
	if not is_on_floor():
		velocity.y -= gravity * delta

	# Handle Jump and Jump Animation
	if Input.is_action_just_pressed("jump") and is_on_floor():
		velocity.y = JUMP_VELOCITY
		playback.start("Jump")

	# Get the input direction and handle the movement/deceleration.
	var input_dir = Input.get_vector("left", "right", "forward", "backward")
	# TM - Replace player transform.basis with camera basis to look in camera direction.
	var direction = (spring_arm_pivot.basis * Vector3(input_dir.x, 0, input_dir.y)).normalized()
	if direction:
		velocity.x = direction.x * SPEED
		velocity.z = direction.z * SPEED

		#TM - Smooth rotation of RB armature when move direction changes
		armature.rotation.y = lerp_angle(armature.rotation.y, atan2(-velocity.x, -velocity.z), LERP_VAL)

	else:
		velocity.x = 0.0
		velocity.z = 0.0

	move_and_slide()

	# TM - Rotate Camera via mouse movement values
	spring_arm_pivot.rotate_y(spring_pivot)
	spring_arm.rotate_x(spring)
	# Restrict camera movement on the X axis
	spring_arm.rotation.x = clamp(spring_arm.rotation.x, deg_to_rad(-40), deg_to_rad(20))
	spring_pivot = 0.0
	spring = 0.0

	#TM - Animation Idle to Walk
	animator[walk_blend] = lerp(
	animator[walk_blend],
	input_dir.length(),
	delta * 5.0
)
```

## Game preview

![Rocket Baby Preview](/assets/img/GodotRB.gif){: w="700" h="400" }
