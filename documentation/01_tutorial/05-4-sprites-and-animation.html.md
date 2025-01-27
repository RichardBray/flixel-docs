```
title: "4 - Sprites and Animation"
```

So we have a player sprite that moves around the screen. Great! But… we don't want it to just look like a block… so let's add some graphics!

First, you need to have the sprite's graphic. Using your image editor of choice, you can draw pretty much anything you want, and save it as a transparent `.png`. For our player sprite, we're going to have it be made up of 16x16 pixel frames, with two different frames for each of the 4-possible facing directions (left and right are used by the same 2 frames, since we will be flipping them later).

You can draw it yourself, if you want, or use this image, created by my friend Vicky Hedgecock for this tutorial:

![](https://raw.githubusercontent.com/HaxeFlixel/flixel-demos/master/Tutorials/TurnBasedRPG/assets/images/player.png)

Save your file in `assets/images`.

Now we need to actually load the player's graphic into the sprite. So, bring up your `Player` class again.

1. Remove the `makeGraphic()` call from your constructor, and replace it with:

	```haxe
	loadGraphic(AssetPaths.player__png, true, 16, 16);
	```

	This tells your sprite to use the `player.png` graphic, that it's animated, and that each frame is 16x16 pixels. `AssetPaths` is a class generated by a neat [Haxe macro](http://haxe.org/manual/macro.html) which builds its variables from the contents of your `Project.xml`'s assets tag. Macros are a little complicated, but worth taking a look into at some point. In this case, we use `AssetPaths` as an easy way to reference our assets in code. Note that we could also just use a raw string path like `"assets/images/player.png"`.

2. Next, we want to allow the sprite to be flipped based on its `facing` value. This makes it so we only need sprites for one direction (left), and not two (left and right).

	Add the following:

	```haxe
	setFacingFlip(LEFT, false, false);
	setFacingFlip(RIGHT, true, false);
	```

	All we're doing here is saying that we don't want to flip anything when facing left (because our sprite already faces left), and to flip horizontally when facing right. We could do the same for up and down if we wanted.

3. Then, we want to make the hitbox match the graphics. With this more top down perspective, it's good practice for the hitbox to match the feet, so let's set ours to the bottom middle of the sprite:

	```haxe
	setSize(8, 8);
	offset.set(4, 8);
	```

4. Now, we need to define our animations. In our case, we want a unique animation for each direction, except the right-facing one will be a mirrored version of the left-facing animation. We also want a "idle" and "walk" animations, depending on whether the player is moving. So, add:

	```haxe
	animation.add("d_idle", [0]);
	animation.add("lr_idle", [3]);
	animation.add("u_idle", [6]);
	animation.add("d_walk", [0, 1, 0, 2], 6);
	animation.add("lr_walk", [3, 4, 3, 5], 6);
	animation.add("u_walk", [6, 7, 6, 8], 6);
	```

	We're finished with the constructor changes, the final step is to change our `updateMovement()` function to tell the player sprite which way to face. So, modify our section which deals with setting the player's `angle` so that it will also set its `facing`, accordingly:

	```haxe
	if (up || down || left || right)
	{
		var newAngle:Float = 0;
		if (up)
		{
			newAngle = -90;
			if (left)
				newAngle -= 45;
			else if (right)
				newAngle += 45;
			facing = UP;
		}
		else if (down)
		{
			newAngle = 90;
			if (left)
				newAngle += 45;
			else if (right)
				newAngle -= 45;
			facing = DOWN;
		}
		else if (left)
		{
			newAngle = 180;
			facing = LEFT;
		}
		else if (right)
		{
			newAngle = 0;
			facing = RIGHT;
		}

		// determine our velocity based on angle and speed
		velocity.setPolarDegrees(SPEED, newAngle);
	}
	```

	Now we can use this `facing` value to determine which animation to use, but we also need to know whether the player is currently moving or not. Using these values we will determine with animation to use:

	```haxe
	var action = "idle";
	// check if the player is moving, and not walking into walls
	if ((velocity.x != 0 || velocity.y != 0) && touching == NONE)
	{
		action = "walk";
	}

	switch (facing)
	{
		case LEFT, RIGHT:
			animation.play("lr_" + action);
		case UP:
			animation.play("u_" + action);
		case DOWN:
			animation.play("d_" + action);
		case _:
	}
	```

	Every time this function gets called, it will check to see which of the directions the player is pressing, and, based on those, which way the sprite should be facing, and which animation should be playing.
	
	Note: Calling `animation.play` with an animation name matching the current animation will not restart the animation.

5. Save your changes and run your project, and you should see your player sprite animate while being moved, and facing the right direction!

	![](../images/01_tutorial/browser_animated_player.gif)

Next, we'll talk about maps and collision!
