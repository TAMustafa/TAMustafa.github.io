---
title: Learning Unity Part 1 - Player Movement on a NavMesh
date: 2023-04-28 18:17:21 +500
categories: [Unity, Tower Defense]
tags: [unity, navmesh, probuilder, c#]
---

The purpose of the blog is to document my journey in becoming a Unity game developer. In this first part I show how I made the player move towards to the goal using Unity NavMesh.

The following Unity features will be covered:

- New [Navigation System](https://docs.unity3d.com/Packages/com.unity.ai.navigation@1.1/manual/NavInnerWorkings.html)
- [ProBuilder](https://docs.unity3d.com/Packages/com.unity.probuilder@5.0/manual/index.html) for level design.
- Simple and reusable [C# code](https://docs.unity3d.com/2023.2/Documentation/Manual/CreatingAndUsingScripts.html)

## Required Packages

The following packages need to be installed from the Unity Registry.

- NavMesh
- ProBuilder

## Game Platform and NavMesh

Using ProBuilder I created a cube, scaled it down and used it as a platform. Then I added some custom shapes as obstacles for the player.
For the player I used a simple capsule and for the target a sphere.

![Level Overview](/assets/img/Overview.png){: w="700" h="400" }

## Creating a NavMesh

In order to apply AI Navigation needed for the player to find the way to the goal, I selected the platform and added the **NavMeshSurface** component.

![NavMesh Surface](/assets/img/NavMesh.png)

## Creating an Agent

The capsule is used as player and for it to know its way around the NavMesh I added a Nav Mesh Agent to it.

![NavMeshAgent](/assets/img/NavMeshAgent.png)

## Logic to move the player and destroy it once the Target is reached

With the NavMeshSurface and NavMeshAgent applied, the next step is to create a C# script and add it to the player (capsule).

1. **Above the Start Function**

- UnityEngine.AI has been added to the script since it is needed for the Navigation System
- Created private variables for the player, goal, lastPosition and DistanceThreshold to use later in the code.

2. **The Start function does the following:**

- Assign the goal variable to the Goal GameObject (Sphere) and check if the GameObject is available.
- Assign the NavMeshAgent Component to the agent(Capsule).
- Tell the player where it needs to go (Goal Position)
- Assign the player or agent position to the _lastPosition_ variable that will be used later in the _FaceDirection_ function.

3. **The Update function does the following:**

- Calling the two custom functions I created. DestroyOnTarget and FaceDirection.

4. **Custom Function - DestroyOnTarget:**

- The purpose of this function is to destroy the player / agent GameObject once it reaches the Goal.
- The code checks the distance of the player to the goal. If the player is closer then 0.3 units (Threshold) the condition becomes true and the GameObject will be destroyed.

5. **Custom Function - FaceDirection:**

- Assign a new Vector3 variable _Direction_ and assign it the current position of the player / agent.
- If condition checks if the player is moving and if so it will rotate (Z Axis is the front) the player to look in the direction of the movement.

## The complete C# code for the player movement

```c#
using UnityEngine;
using UnityEngine.AI;

public class MoveTo : MonoBehaviour
{
private NavMeshAgent agent;
private GameObject goal;
private Vector3 lastPosition;
[SerializeField] private float ThresholdDistance = 0.3f;

    void Start()
    {
        goal = GameObject.Find("Goal");
        if (goal == null)
        {
            Debug.LogError("Could not find the 'Goal' object.");
            return;
        }

        agent = GetComponent<NavMeshAgent>();
        agent.destination = goal.transform.position;
        lastPosition = transform.position;
    }

    void Update()
    {
        if (agent.velocity.magnitude > 0)
        {
            FaceDirection();
        }
        DestroyOnTarget();
    }

    void DestroyOnTarget()
    {
        if (agent.remainingDistance <= ThresholdDistance)
        {
            Destroy(gameObject);
        }
    }

    void FaceDirection()
    {
        Vector3 direction = transform.position - lastPosition;

        if (direction.magnitude > 0.01f)
        {
            transform.LookAt(transform.position + direction);
            lastPosition = transform.position;
        }
    }
}
```

## Current game preview

![First Run](/assets/img/FirstRun.gif){: w="700" h="400" }
