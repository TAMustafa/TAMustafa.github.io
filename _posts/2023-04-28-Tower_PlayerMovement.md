---
title: Tower Defender - Part 1
date: 2023-04-28 18:17:21 +500
categories: [Unity, Tower Defense]
tags: [unity, navmesh, probuilder, c#]
---

# Enemy Movement on a NavMesh

The purpose of the blog is to document my journey in building a simple Tower Defense game.
In this _first_ part I show how I made the enemy object move towards to the target using Navigation AI (NavMesh).

The following Unity packages will be covered:

- New [Navigation System](https://docs.unity3d.com/Packages/com.unity.ai.navigation@1.1/manual/NavInnerWorkings.html)
- [ProBuilder](https://docs.unity3d.com/Packages/com.unity.probuilder@5.0/manual/index.html) for level design.
- Simple and reusable [C# code](https://docs.unity3d.com/2023.2/Documentation/Manual/CreatingAndUsingScripts.html)

## Required Packages

The following packages need to be installed from the Unity Registry.

- NavMesh
- ProBuilder

## Game Platform and NavMesh

Using ProBuilder I created a cube, scaled it down and used it as a platform. Then I added some custom shapes as obstacles for the enemy.
For the enemy object I used a simple capsule and for the target a sphere.

![Level Overview](/assets/img/Overview.png){: w="700" h="400" }

## Creating a NavMesh

In order to apply AI Navigation needed for the enemy object to find the way to the target, I selected the platform and added the **NavMeshSurface** component.

![NavMesh Surface](/assets/img/NavMesh.png)

## Creating an Agent

The capsule is used as enemy and for it to know its way around the NavMesh I added a Nav Mesh Agent to it.

![NavMeshAgent](/assets/img/NavMeshAgent.png)

## Explain logic to move the enemy and destroy it once the Target is reached

With the NavMeshSurface and NavMeshAgent applied, the next step is to create a C# script and add it to the enemy (capsule).

### Above the Start Function

- UnityEngine.AI has been added to the script since it is needed for the Navigation System
- Created private variables for the enemy, target, lastPosition and DistanceThreshold to use later in the code.

### The Start function does the following:

- Assign the target variable to the target GameObject (Sphere) and check if the GameObject is available.
- Assign the NavMeshAgent Component to the enemy(capsule).
- Tell the enemy object where it needs to go (target position)
- Assign the enemy position to the _lastPosition_ variable. It will be used later in the _FaceDirection_ function.

### The Update function does the following:

- Calling the two custom functions I created. DestroyOnTarget and FaceDirection.

### Custom Function - DestroyOnTarget:

- The purpose of this function is to destroy the enemy GameObject once it reaches the target.
- The code checks the distance between the enemy and the target. If the enemy is closer then 0.3 units (Threshold), the condition becomes true and the enemy GameObject will be destroyed.

### Custom Function - FaceDirection:

- Assign a new Vector3 variable _Direction_ and assign it the current position of the enemy object.
- The if condition checks if the enemy is moving and if so, it will rotate (Z Axis is the front) the enemy to look in the direction of the movement.

## The complete C# code for the enemy movement

```c#
using UnityEngine;
using UnityEngine.AI;

public class MoveTo : MonoBehaviour
{
private NavMeshAgent agent;
private GameObject target;
private Vector3 lastPosition;
[SerializeField] private float ThresholdDistance = 0.3f;

    void Start()
    {
        target = GameObject.Find("target");
        if (target == null)
        {
            Debug.LogError("Could not find the 'target' object.");
            return;
        }

        agent = GetComponent<NavMeshAgent>();
        agent.destination = target.transform.position;
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
