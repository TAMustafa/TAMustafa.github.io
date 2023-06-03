---
title: Tower Defender - Part 5
date: 2023-06-03 18:08:57 +500
categories: [Unity, Tower Defense]
tags: [unity, slider, setActive]
---

# Enemy health bar and damage

The purpose of the blog is to document my journey in building a simple Tower Defense game.
In this _fifth_ part I show how to add a health bar that update it when taking damage.

The following Unity / C# features will be covered:

- [UI Slider](https://docs.unity3d.com/Packages/com.unity.ugui@1.0/manual/script-Slider.html)
- [GameObject.SetActive](https://docs.unity3d.com/2023.2/Documentation/ScriptReference/GameObject.SetActive.html)

## Setup a UI Slider in the Unity Interface

In order to see the damage, a health bar needs to be added to the Enemy. A good way to do this is via UI -> Slider.
I have changed the _Fill_ / _Background_ color and un-ticked the Active mark, so the slider won't be visible at the start of the game but only becomes visible after the first hit.

![UI Slider HealthBar](/assets/img/HealthBar.png){: w="700" h="400" }

## Explain logic for the EnemyHealthController script:

This script controls the UI Slider that is placed above the Enemy prefab and changes it's value based on the damage taken.

### Variables defined at the start

- Total enemy health with a start value of 10. Can be adjusted in the editor.
- A UI field to assign the UI slider too in the editor.

### The InitializeHealthBar method does the following:

- Sets the maximum value of the health slider to the total health of the enemy.
- Sets the slider value to the total health as well. This is used for the start of the game.

### The Update method:

- Calls the custom defined UpdateHealthBarRotation method.

### The UpdateHealthBarRotation method does the following:

- This method assigns the healthBar slider rotation to the same direction as the Camera rotation. This is done so the health Slider keeps in the viewer position, even if the enemy changes direction.

### The TakeDamage method does the following:

- Reduces the total health value after damage is taken.
- Sets the Slider on active (visible) after the first hit.
- If the total health reaches zero or below, the enemy game object will be destroyed.
- Updates the Slider value based on updated total health value.

### The complete C# code for the the tower placement script

```c#
using UnityEngine;
using UnityEngine.UI;

public class EnemyHealthController : MonoBehaviour
{
    [SerializeField] private float totalHealth = 10f;
    [SerializeField] private Slider healthBar;

    private void Start()
    {
        InitializeHealthBar();
    }

    private void InitializeHealthBar()
    {
        healthBar.maxValue = totalHealth;
        healthBar.value = totalHealth;
    }

    private void Update()
    {
        UpdateHealthBarRotation();
    }

    private void UpdateHealthBarRotation()
    {
        healthBar.transform.rotation = Camera.main.transform.rotation;
    }

    public void TakeDamage(float damageAmount)
    {
        totalHealth -= damageAmount;
        healthBar.gameObject.SetActive(true);

        if (totalHealth <= 0)
        {
            totalHealth = 0;
            Destroy(gameObject);
        }

        healthBar.value = totalHealth;
    }
}
```

## Current game progress

![Tower rotate to enemy](/assets/img/EnemyHealthBar.gif){: w="700" h="400" }
