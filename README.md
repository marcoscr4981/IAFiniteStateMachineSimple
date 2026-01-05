# IA Avanzada FSM Simple

El script que se ha modificado es _GuardController.cs_.

Los pasos que se dan para resolver el ejercicio es:

1. Cuando se pulsa la tecla **E**, el NPC huye a un punto de escape predeterminado.
2. Mientras el NPC huye, no detecta al jugador.
3. Una vez llega al punto de encuentro, el NPC cambia al modo **Patrol** y ya puede volver a detectar al jugador.

## Script _GuardController.cs_

Obtenemos el _transform_ de un punto de huida (GameObject)

```csharp
[SerializeField] Transform escapePoint;
```

Variable que hace de controlador para anular la detección del jugador cuando el NPC está en modo escape.

```csharp
bool isEscape;
````

### Método Update()

```csharp
void Update()
{
    State tempState = currentState;

    // Detecta si se pulsa la tecla E
    if (Keyboard.current != null && Keyboard.current.eKey.wasPressedThisFrame)
    {
        // Si se pulsa la tecla E:
        //    - La propiedad isEscape cambia a true
        //    - currentState guarda el valor Escape
        //    - Se almacena la última posición que ha tenido el NPC
        isEscape = true;
        currentState = State.Escape;
        lastPlaceSeen = transform.position;
    }
    // Si la propiedad es false
    else if (!isEscape)
    {
        if (ICanSee(player))
        {
            currentState = State.Chase;
            lastPlaceSeen = player.position;
        }
        else
        {
            if (currentState == State.Chase)
            {
                currentState = State.Investigate;
            }
            // Si el NPC no está viendo al jugador y si no está persiguiendo al jugador,
            // como es el caso cuando está huyendo, una vez llegado al punto de escape
            // currentState cambia a Patrullar
            else
            {
                currentState = State.Patrol;
            }
        }
    }

    switch (currentState)
    {
        case State.Patrol:
            Patrol();
            break;
        case State.Investigate:
            Investigate();
            break;
        case State.Chase:
            Chase(player);
            break;
        // Si el valor de currentStat es Escape llama al método Escape()
        case State.Escape:
            Escape();
            break;
    }

    if (tempState != currentState)
    {
        Debug.Log("Guard's state: " + currentState);
    }

    DrawFOVDebug();
}
````

### Método Escape()

```csharp
void Escape()
{
    // Distancia al punto de huida
    float distanceToPoint = Vector3.Distance(transform.position, escapePoint.position);

    // Obtiene la dirección del punto de huida y rota el NPC cara al punto
    Vector3 direction = escapePoint.position - transform.position;
    transform.rotation = Quaternion.Slerp(transform.rotation, Quaternion.LookRotation(direction), Time.deltaTime * chasingRotSpeed);

    // El NPC se mueve cara al punto de huida
    GetComponent<UnityEngine.AI.NavMeshAgent>().SetDestination(escapePoint.position);

    // Si el NPC llega a +2 del punto de huida, cambia la propiedad isEscape a false para
    // que el jugador cambie al estado Patrol
    if (distanceToPoint < GetComponent<UnityEngine.AI.NavMeshAgent>().stoppingDistance + 2)
    {
        isEscape = false;
    }
}
````
