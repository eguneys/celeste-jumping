### Making a Platformer in Pico8

This is an analysis of Pico8 Celeste platformer game source code. Game includes movement, collisions, jumping, wall sliding, dashing, moving platforms, pickups, linear level progression, and various platformer tricks to make it feel good. It doesn't include music or sound making.

### Hello Platformer

Three main issues I encountered with making a platformer. Physics, collisions, levels, also the camera. Here's two resources about physics I used:

[Simple Physics Based Movement](https://stackoverflow.com/questions/667034/simple-physics-based-movement)
[Building a Better Jump](https://www.youtube.com/watch?v=hG9SzQxaCm8)

These define the parameters of the movements by means of sensible values. First one finds the friction and acceleration by means of maximum velocity and time needed to reach the maximum velocity. Second one finds the gravity and initial velocity by maximum velocity, maximum jump height, maximum horizontal distance taken during the jump. It also discusses about double jumping, and variable height jumping which are basically done by changing the gravity.

Collisions; I mention some of the resources that helped me  in this [question; simple collision resolution in a platformer game](https://gamedev.stackexchange.com/questions/184194/simple-collision-resolution-in-a-platformer-game). But I discarded them when I discovered [Celeste Source Code in C#](https://github.com/NoelFB/Celeste). Celeste separates X and Y axis movement, and moves the objects in small steps until they collide, in which case the collision is detected and resolved before happening. Here are some more resources that talk about Celeste: [Celeste by Game Maker's Toolkit](https://www.youtube.com/watch?v=yorTG9at90g), and [Celeste Physics by Matt Thorson](https://medium.com/@MattThorson/celeste-and-towerfall-physics-d24bd2ae0fc5).

Levels are done using Pico8 map editor which I will mention how Celeste does it in this article.

For camera, I decided to use a horizontal side scroller to add some variety to this research. Here's a good resource about;

[How Camera's work](https://www.youtube.com/watch?v=pdvCO97jOQk).

### Hello Pico8

Here's some resources to get over the basics.

[Pico8 user manual](https://www.lexaloffle.com/pico8_manual.txt)
[Pico8 Shooter tutorial](https://ztiromoritz.github.io/pico-8-shooter/)
[Pico8 Celeste Platformer](https://www.lexaloffle.com/bbs/?tid=2145)

But don't be discouraged by Pico8 the principles applies to other engines as well.

### Jump only on ground and Coyote Jumping

Now the player can jump continously while in the air:
![fail jump grace](pre_jump_fail_grace.gif)

To fix this, we introduce the `grace` timer, as a property on the player object.

   local p = {
      grace=0,
      //...

then, update inside `player_update`:

    if (is_grounded) then
      p.grace = 6
    elseif (p.grace > 0) then
      p.grace -= 1
    end

then add as a guard on our jump code:

     // jump
     if (p.j_buffer > 0) then
        if (p.grace > 0) then
           p.ay = -g_jump
           p.dy = -v0_jump
           p.grace = 0
        end
     end

This timer starts ticking down as soon as we leave the ground, (like running off of a platform or jumping), and allows for 6 frames to be able to jump if not already, of course if we jumped we set it to 0 so we don't jump again.

Play yourself by landing on a higher floor and running off the edge, and try to jump after you have left the floor. Change the timer to 1 to notice the difference.


### Wall Slide

Similar to how we applied friction drag to horizontal movement, we will add vertical friction drag for wall slide.

`v_slide_drag` will control how much to apply vertical friction drag:

     local v_slide_drag = 0

     // wall slide
     if (input_x != 0 and is_solid(p, input_x, 0)) then
        if sgn(p.dy)>0 then
           v_slide_drag = 1
        end
     end

Outer check is checking if there is a wall on the `input_x` direction, and inner check is only allowing vertical drag if the player is going down. If we limit the velocity while sliding and going down, this will allow jumping player to continue to move up as colliding with a wall.

Finally apply the drag after we apply acceleration:

     p.dy += p.ay
     p.dy += -p.dy * x_friction * v_slide_drag


Feel free to suggest a better method, since it uses `x_friction` which is the friction for horizontal drag.
