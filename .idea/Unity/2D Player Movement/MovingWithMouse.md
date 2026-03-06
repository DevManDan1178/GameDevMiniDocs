# 2D Player Omnidirectional Movement with Mouse
Simple 2D player movement script (singleplayer) where the player moves towards the most recent mouse right click (without pathfinding) and stops when 'S' is pressed.

## The setup

### - Player object
Add an object for the player, preferably with a Sprite Renderer so you can see it move.

_It can just be an empty game object with a transform and something to see it by._

### - Add a Rigidbody2D to the player object
A Rigidbody is useful for handling physics, like collisions, which will probably be important for your player object.

Set your Rigidbody2D's body type accordingly.
>- _Dynamic_ body type is affected by all physical effects (gravity, forces, collision with all other body types)
>
>- _Kinematic_ body type is unaffected by gravity and forces, but can collide with _Dynamic_ bodies
>
>- _Static_ body type is completely immovable (ignores physics)

In this case, we do not want any gravity to affect our movement, so we can choose a _Dynamic_ body type and set it to zero or just use a _Kinematic_ body type.

## The Script
Add this as a component to the player object.

### - How the script should work.
#### To make the character move, we need to:
>- determine the desired movement direction from the player input.
>
>- apply the movement and its physics to the object.

#### Unity has two relevant continually called _Update_ functions:
>- _Update_ runs once per frame, so the timestep between the last frame is likely different. This makes it good for obtaining the player' input, as we can always detect any changes to it as soon as they happen, but the varying timesteps can lead to unpredictable physics.
>
>
>- _FixedUpdate_ runs at a fixed timestep. It is perfect for running physics processes, but it can be slower to detect changes in player input.

#### For optimal results, we should use _Update_ to detect the player's desired move direction, and use _FixedUpdate_ to move the player object.
This means that our class will need to know:
>- The Rigidbody2D to move
>- The movement speed (customizable)
>- The movement's destination (from user input)
>- The movement direction (from the destination)

```csharp```
    using UnityEngine;
    
    public class MouseMovement : MonoBehaviour {
    
        private Vector2 movementDirection;
        private Vector2 targetPosition;
        
        //Set as the Rigidbody2D of the parent player character ([SerializeField] makes private fields customizable in the editor)
        [SerializeField]
        private Rigidbody2D rb;
        
        //Customize this in the editor
        public float moveSpeed; 
        
        //Called every frame
        void Update() {
            //If the player wants to stop any previous movement, reset the movement and stop checking for other inputs.
            if (Input.GetKey(KeyCode.S)) {
                //Set the target position to its own position so it will avoid moving in future calls
                targetPosition = rb.position;
                movementDirection = Vector2.zero;
                return;
            }
            //If the player did not change the target, avoid overshooting.
            if (!Input.GetMouseButtonDown(1)) {
                
                Vector2 deltaPosition = targetPosition - rb.position;
                
                //If the distance from the targeted position is very small, stop moving (arbitrary threshold of 0.0025 square magnitude chosen)
                movementDirection = deltaPosition.sqrMagnitude <= 0.0025 ? 0 : deltaPosition.normalized;
                return;
            }
            //Set the target to the position of our mouse in the world's coordinates (we can get it from the screen coordinates with our camera)
            Vector3 screenToWorldPoint = Camera.main.ScreenToWorldPoint(Input.mousePosition);

            //Set the target position to the mouse's position in the game world
            targetPosition = new Vector2(screenToWorldPoint.x, screenToWorldPoint.y);

            //Set the movement direction to be moving towards the target position. Normalize it so it remains a unit vector.
            movementDirection = (targetPosition - rb.position).normalized;
        }
        
        //Called at a fixed timestep
        void FixedUpdate() {
            /*
                Apply the previously updated movement direction to the rigidbody
                using Rigidbody.MovePosition (creates a smooth transition movement between frames)
                Moving it by [adding to the position vector] (direction * speed * timestep).
            */
            rb.MovePosition(rb.position + movementDirection * moveSpeed * Time.fixedDeltaTime);
        
        }  
    }

> **Tip:**
> The movement direction vector must be normalized to avoid the positioning of the clicks affecting speed.

_*It is also possible to assign the _Rigidbody rb_ field without the editor through script. This can be done with Unity's _Awake_ method, which is fired when the object instance is loading.*_


    void Awake() {
        rb = GetComponent<Rigidbody2D>();
    }