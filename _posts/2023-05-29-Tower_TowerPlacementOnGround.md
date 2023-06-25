---
title: Tower Defender - Part 4
date: 2023-05-29 16:33:28 +500
categories: [Unity, Tower Defense]
tags: [unity, layer, raycast, input system]
---

# Tower placement on defined "Ground" layer

The purpose of the blog is to document my journey in building a simple Tower Defense game.
In this _forth_ part I show how to Place the Tower over a specified area (ground layer).

The following Unity / C# features will be covered:

- [Physics Raycast](https://docs.unity3d.com/2023.2/Documentation/ScriptReference/Physics.Raycast.html)
- [Layermask](https://docs.unity3d.com/2023.2/Documentation/Manual/layermask-set.html)
- [Input System](https://docs.unity3d.com/Packages/com.unity.inputsystem@1.6/manual/index.html)

## Explain logic for the tower placement script

This script will initiate a tower at the position of the mouse cursor if the cursor is above specified Layer (ground).

### Variables defined at the start:

- A prefab UI reference to assign the tower game object that will be initiated at mouse click.
- A layer mask UI reference to assign specify what the ground is.

### The Update method:

- The mouse pointer position is stored in a variable _"ray"_.
- The variable _"RaycastHit"_ is storing information about the object that is hit.
- The IF statement is checking if the mouse is hitting the _"groundLayer"_.
- The following IF statement is checking if there is a Tower at the mouse location and if not it initiates one. (But doesn't place it on the ground yet)
- If the mouse button is clicked on a ground layer, a Tower will be initiated.
  If the mouse is hoovering away from the "Ground" layer the floating Tower will be destroyed.

### Custom Function - PlaceTower:

- This function will initiate a Tower prefab and replaces the hoovering current tower with a new tower.

### Custom Function - DestroyTower:

- This function will destroy the Tower game object.

## The complete C# code for the the tower placement script

```c#
using UnityEngine;
using UnityEngine.InputSystem;

public class TowerPlacement : MonoBehaviour
{
    [SerializeField] private GameObject towerPrefab;
    [SerializeField] private LayerMask groundLayer;

    private GameObject currentTower;

    private void Update()
    {
        HandleTowerPlacement();
    }

    private void HandleTowerPlacement()
    {
        Ray ray = Camera.main.ScreenPointToRay(Mouse.current.position.ReadValue());
        RaycastHit hitInfo;

        if (Physics.Raycast(ray, out hitInfo, Mathf.Infinity, groundLayer))
        {
            if (currentTower == null)
            {
                currentTower = Instantiate(towerPrefab, hitInfo.point, Quaternion.identity);
            }
            else
            {
                currentTower.transform.position = hitInfo.point;
            }

            if (Mouse.current.leftButton.wasPressedThisFrame)
            {
                PlaceTower(hitInfo.point);
            }
        }
        else
        {
            DestroyTower();
        }
    }

    private void PlaceTower(Vector3 position)
    {
        if (currentTower == null)
        {
            return;
        }

        GameObject newTower Instantiate(towerPrefab, position, Quaternion.identity);
        currentTower = newTower;
    }

    private void DestroyTower()
    {
        if (currentTower != null)
        {
            Destroy(currentTower);
            currentTower = null;
        }
    }
}
```

## Game preview

![Tower rotate to enemy](/assets/img/TowerPlacement.gif){: w="700" h="400" }
