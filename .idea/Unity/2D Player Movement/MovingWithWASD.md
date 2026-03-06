# 2D Player Omnidirectional Movement with keyboard 
Simple 2D player movement script (singleplayer)

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
>- _FixedUpdate_ runs at a fixed timestep. It is perfect for running physics proceses, but it can be slower to detect changes in player input.

#### For optimal results, we should use _Update_ to detect the player's desired move direction, and use _FixedUpdate_ to move the player object.
This means that our class will need to know:
>- The Rigidbody2D to move
>- The movement speed (customizable)
>- The movement direction (from user input)

```C#```
    
    using UnityEngine;
    
    public class KeyboardMovement : MonoBehaviour {
    
    private Vector2 movementDirection;
    
    //Set as the Rigidbody2D of the parent player character ([SerializeField] makes private fields customizable in the editor)
    [SerializeField]
    private Rigidbody2D rb;
    
    //Customize this in the editor
    public float moveSpeed; 
    
        //Called every frame
        void Update() {
            /*
                Getting the player's inputs as a 2D vector for the movement direction 
                using Input.GetAxisRaw() for "Horizontal" (x) and "Vertical (y).
                IMPORTANT: movementDirection must be a unit vector (length of 1)
            */
            movementDirection = new Vector2(Input.GetAxisRaw("Horizontal"), Input.GetAxisRaw("Vertical")).normalized;
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
**NOTE: The movement direction vector must be normalized, since Input.GetAxisRaw(...) returns either 1, 0, or -1. Moving diagonally gives a vector with a length of up to  1.414 (√2), increasing speed by up to 41% when moving diagonally**