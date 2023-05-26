---
title: Tower Defender - Part 3
date: 2023-05-26 18:44:31 +500
categories: [Unity, Tower Defense]
tags: [unity, transform, vector3, quaternion, foreach]
---

# Close enemy detect and shoot

The purpose of the blog is to document my journey in building a simple Tower Defense game.
In this _third_ part I show how to rotate the Tower towards and shoot at the nearest enemy.

The following C# features will be covered:

- Transform
- Vector3
- Quaternion
- GetComponent
- foreach

## Tower Setup

For the Tower I created a cylinder as _base_ and on top a capsule as the fire head that is within an empty game objet _Rotate_. I then saved the Tower as a Prefab, so that it can be used and spawned multiple times going forward.

![Tower Setup](/assets/img/TowerSetup.png){: w="700" h="400" }

## Explain logic for the detect and shoot script:

With the tower being in place, the next step is to create two C# scripts.

- One for detecting and shooting at nearby enemies
- One that controls the bullet movement.

### Above the Start method:

In order to select in the Unity UI what game objects are needed for this script, I created a few SerializeField's. I also defined some variables like the Tower turn speed, fire rate and range.

### The Start method:

Checking 60 times or more per second if an enemy is in range in the Update method would be very CPU consuming. Instead I use a coroutine defined in the Start method to call the **UpdateEnemyInRange** every 2 time per second.

### The Update method:

- Prevent that no resources are wasted if there is no target. The If statement is taking care about it and returns every frame if there is no target.

- Define the position of the target (closed enemy) and tell the Tower to rotate towards the closest enemy in range.

- Call the Shoot() method if a set of conditions is fulfilled. The enemy need to be in range and the tower need to rotate and look at the enemy. Reset the time between shooting.

### Custom function - Shoot:

- Get the pullet component.

- Calls the Seek method to make the bullet follow the target.

- Instantiate the bullet at the position of the Tower fire point.

### Custom function - IEnumerator UpdateEnemyInRange:

- Check two time a second all enemies in range of the tower and define a **target** that is the closest enemy in range.

### The OnDrawGizmosSelected method does the following:

- Draws a a white wire frame sphere around the Tower to visually see the range.

### The complete C# code for the the detect and shoot script

```c#
using System.Collections;
using UnityEngine;

public class TowerMoveAndShoot : MonoBehaviour
{
    [SerializeField] private GameObject projectilePrefab;
    [SerializeField] private Transform partToRotate;
    [SerializeField] private Transform firePoint;
    [SerializeField] private float range = 3f;
    [SerializeField] private float turnSpeed = 8f;
    [SerializeField] private float fireRate = 2f;
    [SerializeField] private string enemyTag = "Enemy";

    private Transform target;
    private float fireCountdown;

    private void Start()
    {
        StartCoroutine(UpdateEnemyInRange());
        fireCountdown = 1f / fireRate;
    }

    private void Update()
    {
        if (target == null)
        {
            return;
        }

        Vector3 dir = target.position - transform.position;
        Quaternion lookRotation = Quaternion.LookRotation(dir);
        Vector3 rotation = Quaternion.Lerp(partToRotate.rotation, lookRotation, Time.deltaTime * turnSpeed).eulerAngles;
        partToRotate.rotation = Quaternion.Euler(0f, rotation.y, 0f);

        if (target != null && fireCountdown <= 0f && Vector3.Dot(partToRotate.forward, (target.position - transform.position).normalized) >= 0.8f)
        {
            Shoot();
            fireCountdown = 1f / fireRate;
        }
        else
        {
            fireCountdown -= Time.deltaTime;
        }
    }

    private void Shoot()
    {
        Projectile projectile = cannonGO.GetComponent<Projectile>();
        projectile?.Seek(target);
        GameObject cannonGO = Instantiate(projectilePrefab, firePoint.position, firePoint.rotation);
    }

    private IEnumerator UpdateEnemyInRange()
    {
        while (true)
        {
            yield return new WaitForSeconds(0.5f);

            GameObject[] enemies = GameObject.FindGameObjectsWithTag(enemyTag);
            float shortestDistance = Mathf.Infinity;
            GameObject nearestEnemy = null;

            foreach (GameObject enemy in enemies)
            {
                float distanceToEnemy = Vector3.Distance(transform.position, enemy.transform.position);
                if (distanceToEnemy < shortestDistance)
                {
                    shortestDistance = distanceToEnemy;
                    nearestEnemy = enemy;
                }
            }

            target = (nearestEnemy != null && shortestDistance <= range) ? nearestEnemy.transform : null;
        }
    }

    private void OnDrawGizmosSelected()
    {
        Gizmos.color = Color.white;
        Gizmos.DrawWireSphere(transform.position, range);
    }
}
```

### Game preview with Towers rotating towards the closet enemy and initiate bullet prefab

![Tower rotate to enemy](/assets/img/TowerSearchTarget.gif){: w="700" h="400" }

## Explain logic for the Bullet script:

In order to make separate the the bullet functionality from the tower, I used a separate script.

### Above the Update method:

- A private variable to store the tower target data.

- A variable to set the bullet move speed.

### Custom function - Seek:

- This method is taking the target information from the Tower script and stores it into the local target variable.

### The Update method:

- First check if we have a target. If not then Destroy the bullet and exit out of the Update method.

- If there is a target the direction of the target will be determined and stored into the _dir_ variable.

- The _IF_ block checks if the the bullet has reached the target and if so calls the custom HitTarget function which destroys the bullet game object.

- If the bullet has not reached the target yet, then the bullet position will be updated every frame to make the bullet move in a constant speed (normalized) towards the target.

### Custom function - HitTarget:

- Destroys the bullet game object when called.

### The complete C# code for the the detect and shoot script

```c#
using UnityEngine;

public class Bullet : MonoBehaviour
{
    private Transform target;
    [SerializeField] private float moveSpeed = 5f;


    public void Seek(Transform _target)
    {
        target = _target;
    }

    // Update is called once per frame
    void Update()
    {
        if (target == null)
        {
            Destroy(gameObject);
            return;
        }

        Vector3 dir = target.position - transform.position;
        float distanceThisFrame = moveSpeed * Time.deltaTime;

        if (dir.magnitude <= distanceThisFrame)
        {
            HitTarget();
            return;
        }
        transform.Translate(dir.normalized * distanceThisFrame, Space.World);

    }

    void HitTarget()
    {
        Destroy(gameObject);
    }
}
```

## Current game preview

![Tower rotate to enemy](/assets/img/TowerRotateAndShoot.gif){: w="700" h="400" }
