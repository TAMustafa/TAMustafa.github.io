---
title: Tower Defender - Part 2
date: 2023-05-06 22:04:09 +500
categories: [Unity, Tower Defense]
tags: [unity, coroutine, random, spawn, c#]
---

# Random Object Spawn

The purpose of the blog is to document my journey in building a simple Tower Defense game.
In this _second_ part I show how I made the enemy objects randomly spawn from different areas of the platform.

The following C# features will be covered:

- [Coroutines](https://docs.unity3d.com/2023.2/Documentation/Manual/Coroutines.html) for time between spawns.
- [Random.Range](https://docs.unity3d.com/2023.2/Documentation/ScriptReference/Random.Range.html) for randomize spawning.
- Simple and reusable [C# code](https://docs.unity3d.com/2023.2/Documentation/Manual/CreatingAndUsingScripts.html)

## Spawn Manager

For the SpawnManager I used an **empty game object** and rested it's location to 0. Then I added two empty game objects as children which act as the actual spawning points. For better visibility I added an Icon to them (green diamond).

![Spawn points](/assets/img/SpawnPoints.png){: w="700" h="400" }

## Explain logic to spawn objects randomly

With the SpawnManager and SpawnPoints in place, the next step is to create a C# script and add it to the **SpawnManager** game object.

### Above the Start Function

- System.Collections has been added since I am using **Coroutines** to spread the enemy spawn between frames.
- Add several private variables via a [SerializedField](https://docs.unity3d.com/2023.2/Documentation/ScriptReference/SerializeField.html) to be able to change values directly in the game editor.

### The Start function does the following:

- I like to Spawn Enemy objects every 1-2 seconds. I could use the Update function and an **If statement** to count down to do this but the task would run every frame. In order to reduce the CPU load and be more effective, a coroutine seems to be better to do this kind of tasks.
- **StartCoroutine** is a function and part of the coroutine. It is needed to set the custom function I wrote "SpawnObjects".

### The IEnumerator SpawnObjects function does the following:

- Spawns new enemy prefabs as long the _amountToSpawn_ value is greater zero.
- At every iteration creates a random number that is used to determine at which spawnPoint the enemy prefab is initiated.
- Checks first that we have still objects left to spawn before running the initiate method (Null check).
- Wait the amount specified in _timeBetweenSpawn_ before running again.

## The complete C# code for the Spawn Manager

```c#
using System.Collections;
using UnityEngine;

public class ObjectEnemySpawner : MonoBehaviour
{
    [SerializeField] private GameObject objectToSpawn;
    [SerializeField] private int amountToSpawn = 6;
    [SerializeField] private float timeBetweenSpawn = 1f;
    [SerializeField] private GameObject[] spawnPoints;

    private void Start()
    {
        StartCoroutine(SpawnObjects());
    }

    private IEnumerator SpawnObjects()
    {
        while (amountToSpawn > 0)
        {
            // Spawn an object at a random spawn point
            int randomIndex = Random.Range(0, spawnPoints.Length);
            GameObject spawnPoint = spawnPoints[randomIndex];
            if (objectToSpawn != null)
            {
                Instantiate(objectToSpawn, spawnPoint.transform.position, spawnPoint.transform.rotation);
            }

            // Wait for the specified time before spawning another object
            yield return new WaitForSeconds(timeBetweenSpawn);

            // Decrement the spawn count
            amountToSpawn--;
        }
    }
}
```

## Current game preview

![Spawn Multiple Enemies](https://tamustafa.github.com/assets/img/SpawnEnemyRandom.mp4){: w="700" h="400" }
