### Adding more objects

Until now we only had one player object, with properties like, `x, y`, `cbox`, `spr`. Now we want to add more objects, also add the same properties to these objects. So let's collect the common properties under a base object:

    function object_init(obj)
    end

    function object_update()
    end

    function object_draw()
        spr(obj.spr,obj.x,obj.y,1,1,obj.flipx,obj.flipy)
    end

    baseobject = {
        x=0,
        y=0,
        init=object_init,
        move=object_move,
        update=object_update,
        draw=object_draw,
        cbox={ x=0,y=0,w=8,h=8 },
        flipx=false,
        flipy=false
    }

The `object_draw` function is the default drawing method. It draws the objects `spr` property to `x y` position.

Remove the `player` method and just make it an object with custom properties, make sure functions like `player_update` are defined first before this definition:

    player = {
        dx=0,
        dy=0,
        cbox={ x=1,y=3,w=6,h=5 },
        // ...more properties
        update=player_update,
        draw=player_draw,
        init=player_init
    }

Our `init_object` method now looks like this:

    function init_object(type,x,y)
       local obj = {}

       merge(obj, baseobject)
       merge(obj, type)

       obj.type=type
       obj.x = x
       obj.y = y

       add(objects, obj)

       obj.init(obj)

       return obj
    end

`merge` is a helper function that merges one table into another, taken from [this comment](https://www.lexaloffle.com/bbs/?pid=51185#p). It will overwrite the existing properties.

    function merge(base, extend)
        for k,v in pairs(extend) do
            base[k] = v
        end
    end

Finally `init_object` in `load_room` will change, let's see how next.

### Springs jump player into air

Add two spring tiles, one for the initial state, and one for the bouncing state. Dont set any flags. Place the spring tile on the map. Because you didn't set the 2 flag it won't be drawn when using the `map` function. Spring is an object and it will have it's own draw function (it will use the default draw function).

Add a new spring object when seen a spring tile (tile 71) in `load_room`. Also note, how we init the player object:

     function load_room()
        for i=1,16 do	
         for j=1,16 do
             local tile = mget(i,j)
             local type
             if tile==1 then
                 type = player
             end
             if tile==71 then
                 type = spring
             end
             if type != null then
                 init_object(type, i*8, j*8)
             end
         end
        end
     end

See `player` is not a method anymore but a template (or a type) to create a player object. Define the `spring` type like the `player` type:

    function spring_update(ispring)

    end

    spring = {
        spr=71,
        update=spring_update
    }

We define the `spr` property to be the spring tile to draw. `spring_update` method will start being called for a spring object, `ispring` parameter is the spring object. Do mind the `spring` variable is globally defined as a type. Next, let's define spring's update logic.

When spring collides with the player, we jump the player into air:

    
    function spring_update(ispring)
       local hit = object_collide(ispring, 
                                  player,
                                  0,
                                  0)
       if hit != nil then
          ispring.spr = 72
          hit.y = ispring.y - 4
          hit.dx *= 0.2
          hit.dy=v0_jump * 2
          hit.djump=1
       end
    end

First we check if the spring is hit with a player using `object_collide`. If there is a hit,
we adjust the player's speed, and replenish the dash.

`object_collide` will check if object collides with a given type of object in this case spring collides with player:

    function object_collide(obj, type)
       local cbox = abs_cbox(obj)

       local other
       for i=1,count(objects) do
          other=objects[i]

          if other != nil and other.type == type and other != obj then

             local cbox2 = abs_cbox(other)
             if box_intersects(cbox, cbox2) then
                return other
             end

          end
       end
       return nil
    end

    function box_intersects(a, b)
       return a.x + a.w > b.x and
          a.y + a.h > b.y and
          a.x < b.x + b.w and
          a.y < b.y + b.h
    end


One problem is the spring doesn't return to it's resting state once the jump is over: ![Spring Stall](pre_spring_stall).

Define a delay timer on spring:

    spring = {
        delay=0,
        // ...
    }

Start the delay timer when spring hits, and set the `spr` back to resting state when the timer runs out:

    if hit != nil then
       //... more 
      ispring.delay = 10
    end

    if ispring.delay > 0 then
        ispring.delay -= 1
        if ispring.delay <= 0 then
            ispring.spr=71
        end
    end

71 is resting state tile, 72 is jump state tile. When the spring is hit we set the `spr` to 72, then we set it back to 71 when the delay timer runs out.
