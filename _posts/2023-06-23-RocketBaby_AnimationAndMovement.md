---
title: Character Controller and Animation
date: 2023-06-23 19:48:58 +500
categories: [Unity, Character]
tags: [unity, blender, character controller, animation system, input system]
---

# RocketBaby - Movement and Animation

The purpose of the blog is to document my journey in becoming a Unity game developer.
In this post I show how I got my 3d model "RocketBaby" to move and transition between animations.

The following Unity / C# features will be covered:

- [Character Controller](https://docs.unity3d.com/2023.2/Documentation/Manual/class-CharacterController.html)
- [Input System](https://docs.unity3d.com/Packages/com.unity.inputsystem@1.6/manual/index.html)
- [Animation System](https://docs.unity3d.com/Manual/AnimationOverview.html)

## Export 3d model from Blender

I designed a low poly model "Rocket Baby" in Blender and used the Blender FBX export function in order to import it to Mixamo for the animation.

![Rocket Baby Model](/assets/img/Blender%20RB.png){: w="700" h="400" }

The export settings I used in Blender are as follows:

![FBX Export settings](/assets/img/FBXExport.png){: w="700" h="400" }

## Create Animations using Mixamo.com

After login into Maximo, I uploaded the FBX model, configures the bones as per their setup and chooses a few animations I like for the game.

![Mixamo Animation](/assets/img/MixamoAnimation.gif){: w="700" h="400" }

## Export Mixamo animation files to Unity

Once I found nice animations to use, I exported them as follows. If you only want to export the animation without the full model again, then make sure to choose **Without Skin** in the Skin drop down.

![Mixamo Download Settings](/assets/img/MixamoExportSettings.png){: w="700" h="400" }

## Setup character with animation and conditions

In Unity I dragged the animation files into a Animation folder I created. Under the Animation Tab I selected Humanoid for all animations and assigned the Avatar to the T Pose model (Standard model I imported to Unity).

Then I opened the **Animator** window and dragged the Idle, Running and Jump Animation in. Next I added twoParameters, _isMoving_ and _isJumping_ which are needed to set the logic for the transition between the various animation states.

Finally I connected the Animations with Transition logic (arrows) like "Start the running animation when you are moving and go back to idle when you stand still".

![Mixamo Download Settings](/assets/img/UnityAnimatorSetup.png){: w="700" h="400" }

## Setup the Input System

The great thing with the new input in Unity is that you can define actions with various bindings in one place that allow you to use keyboard, mouse, controllers with using just one code base.

For this game I added a Move and Jump Action. The bindings are the WASD keys for keyboard and left stick for Controller. For jumping I used the Space Bar and X button on controller.

![Input System](/assets/img/InputSystem.png){: w="700" h="400" }

## Create PlayerMovementAndAnimation script

With the animations and and input system setup ready, it is time to create a script that controls all the actions and moves the player and animates it based on the movement / jump.

### Variables defined at the start:

- Declare variables for PlayerInput, Animation and Character Controller to be initiated in the Awake Function.
- Movement and Rotation variables to store the input values which will be used to move and rotate the character.
- Define Gravity values.
- Define variables related to the Jump action.

### The Awake method:

- Initiates the the PlayerInput, Animator and Character Controller.
- Reference to custom callback functions that pass through the input values when the movement or jump buttons are pressed.
- Calling the setupJumpVariables function to declare the Jump Setup at the start of the game.

### Custom Function - OnMovementInput:

This custom Function is called once the movement buttons are pressed. The input values are passed on to this function and the following will be done:

- The _currentMovementInput_ variable gets the Vector2 data assigned.
- Since for moving in Unity only the **X** and **Z** axis are needed, the x and z values will be explicitly stored in the _currentMovement_ variable.
- The _isMovementPressed_ variable will be assigned a true value if the X and Y values are not zero (Player is moving)

### Custom Function - onJumpInput:

- This Function assigns a bool value to the isJumpPressed variable based on the input action.

### Custom Function - setupJumpVariables:

- The conditions about the jump hight and gravity force will be defined here.

### Custom Function - handleJump:

- If the character is on the ground and and the isJumpPressed value true, then the jump animation will be set to true and the currentMovement**Y** value will be set to the velocity value to make the player move up (jump).

- If the player is on the ground and the isJumpPressed button is not pressed then the Animation value will be set back to false.

### Custom Function - handleAnimation:

- This function will assign a _true_ value to the animation if the character is moving and a _false_ value to the animation if the character is not moving.

### Custom Function - handleGravity:

- Applies gravity based on if the character is grounded or not.

### Custom Function - handleRotation:

- Declares a Vector3 variable _positionToLookAt_ to store the same data as the character movement direction and then use these data to make the character look in the same direction the character is moving.

### The OnEnable method:

- Required by the Input System to Enable the Actions of the Input when used.

### The OnDisable method:

- Required by the Input System to Disable the Actions of the Input when **not** used.

### The Fixed Update method:

- Fixed Update is used for physics related updates (every frame) and it calls here the various custom functions that have been defined above.

## The complete C# code for the the PlayerMovementAndAnimation script

```c#
using UnityEngine;
using UnityEngine.InputSystem;

public class PlayerMovementAndAnimation : MonoBehaviour
{
    PlayerInput playerInput;
    CharacterController characterController;
    Animator animator;

    // variables to store player input values
    Vector2 currentMovementInput;
    Vector3 currentMovement;
    bool isMovementPressed;

    // variable to control rotation speed
    float rotationFactorPerFrame = 15.0f;

    // Character Speed
    float moveSpeed = 5.0f;

    //gravity variables
    float gravity = -9.81f;
    float groundGravity = -0.05f;

    // Jump variables
    bool isJumpPressed = false;
    float initialJumpVelocity;
    float maxJumpHight = 0.6f;
    float maxJumpTime = .6f;
    bool isJumping = false;


    void Awake()
    {
        playerInput = new PlayerInput();
        animator = GetComponent<Animator>();
        characterController = GetComponent<CharacterController>();

        // Callback functions that records the Move related Input context
        playerInput.Player.Move.started += onMovementInput;
        playerInput.Player.Move.performed += onMovementInput;
        playerInput.Player.Move.canceled += onMovementInput;

        // Callback function that records the Jump related Input context
        playerInput.Player.Jump.started += onJumpInput;
        playerInput.Player.Jump.canceled += onJumpInput;

        // Call custom function
        setupJumpVariables();
    }

    // Custom function that handles the given callback context
    void onMovementInput(InputAction.CallbackContext context)
    {
        currentMovementInput = context.ReadValue<Vector2>();
        // (re)assign values for correct movement
        currentMovement.x = currentMovementInput.x;
        currentMovement.z = currentMovementInput.y;
        isMovementPressed = currentMovementInput.x != 0 || currentMovementInput.y != 0;
    }

    void onJumpInput(InputAction.CallbackContext context)
    {
        // Assign the bool value from the button input to the variable
        isJumpPressed = context.ReadValueAsButton();

    }

    void setupJumpVariables()
    {
        float timeToApex = maxJumpTime / 2;
        gravity = (-2 * maxJumpHight) / Mathf.Pow(timeToApex, 2);
        initialJumpVelocity = (2 * maxJumpHight) / timeToApex;
    }

    void handleJump()
    {
        if (!isJumping && characterController.isGrounded && isJumpPressed)
        {
            isJumping = true;
            animator.SetBool("isJumping", true);
            animator.SetBool("isMoving", false);
            currentMovement.y = initialJumpVelocity;
            currentMovementInput.y = initialJumpVelocity;
        }
        else if (!isJumpPressed && characterController.isGrounded && isJumping)
        {
            isJumping = false;
        }
    }

    void handleAnimation()
    {
        bool isMoving = animator.GetBool("isMoving");

        if (isMovementPressed && !isMoving)
        {
            animator.SetBool("isMoving", true);
        }
        else if (!isMovementPressed && isMoving)
        {
            animator.SetBool("isMoving", false);
        }
    }

    void handleGravity()
    {
        if (characterController.isGrounded)
        {
            animator.SetBool("isJumping", false);
            currentMovement.y = groundGravity;
            currentMovementInput.y = groundGravity;
        }
        else
        {
            currentMovement.y += gravity * Time.deltaTime;
            currentMovementInput.y += gravity * Time.deltaTime;
        }
    }

    void handleRotation()
    {
        Vector3 positionToLookAt;
        // change rotation values based on character movement
        positionToLookAt.x = currentMovement.x;
        positionToLookAt.y = 0f;
        positionToLookAt.z = currentMovement.z;

        // the current rotation of the character
        Quaternion currentRotation = transform.rotation;

        if (isMovementPressed)
        {
            // new target rotation base on player movement
            Quaternion targetRotation = Quaternion.LookRotation(positionToLookAt);

            // Set Character transform.rotation from previous to current rotation
            transform.rotation = Quaternion.Slerp(currentRotation, targetRotation, rotationFactorPerFrame * Time.deltaTime);
        }
    }

    void OnEnable()
    {
        playerInput.Player.Enable();
    }

    void OnDisable()
    {
        playerInput.Player.Disable();
    }

    void FixedUpdate()
    {
        handleRotation();
        handleAnimation();
        characterController.Move(currentMovement * moveSpeed * Time.deltaTime);

        handleGravity();
        handleJump();
    }

}
```

## Game preview

![Rocket Baby Preview](/assets/img/RBPreview.gif){: w="700" h="400" }
