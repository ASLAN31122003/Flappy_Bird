https://aslan31122003.github.io/Flappy_Bird/Index.html

Check the guide for step by step scripts: [click here](https://github.com/ASLAN31122003/Flappy_Bird/blob/main/guide.c)

# Flappy Bird Clone — Godot Development Guide

This guide walks through creating a basic Flappy Bird clone in Godot, from setting up the main scene and animating the bird to generating pipes, detecting collisions, tracking scores, and implementing a restart system.

## Table of Contents

1. [Create the Main Scene](#1-create-the-main-scene)
2. [Create the Bird Scene](#2-create-the-bird-scene)
3. [Implement Bird Physics and Movement](#3-implement-bird-physics-and-movement)
4. [Set Up the Main Game Script](#4-set-up-the-main-game-script)
5. [Create the Scrolling Ground](#5-create-the-scrolling-ground)
6. [Create the Pipes](#6-create-the-pipes)
7. [Generate and Scroll Pipes](#7-generate-and-scroll-pipes)
8. [Implement Collision Detection](#8-implement-collision-detection)
9. [Add the Scoring System](#9-add-the-scoring-system)
10. [Create the Game Over Screen](#10-create-the-game-over-screen)
11. [Implement the Restart System](#11-implement-the-restart-system)
12. [Final Result](#12-final-result)

---

## 1. Create the Main Scene

First, create the main scene that will contain the game's background, bird, ground, pipes, score display, and game-over interface.

1. Open Godot and create a new scene.
2. Start with a blank scene and add a basic `Node` as the root.
3. Rename the root node to `Main`.
4. Save the scene inside a folder named `scenes`.

### Add the Background

1. Add a `Sprite2D` node as a child of `Main`.
2. Rename it to `Background`.
3. Drag the `BG` image from the assets folder into the sprite's **Texture** property.
4. Adjust the background's X and Y position to align it with the game window.

### Configure the Window Size

1. Open **Project → Project Settings**.
2. Find the display window settings.
3. Set the viewport width to `864` and the viewport height to `936`.
4. Adjust the background position until it fits the screen correctly.

Run the scene to verify that the background is displayed properly.

---

## 2. Create the Bird Scene

The bird is the primary character controlled by the player.

### Create the Scene Structure

1. Create a new scene with a `CharacterBody2D` root node.
2. Rename the root node to `Bird` (or `Invert` if retaining the original project name).
3. Save the scene inside the `scenes` folder.
4. Add an `AnimatedSprite2D` node as a child.
5. Add a `CollisionShape2D` node as another child.

### Configure the Bird Animation

1. Select `AnimatedSprite2D`.
2. Create a new `SpriteFrames` resource.
3. Add a new animation and rename it from `default` to `fly`.
4. Drag the three bird images from the assets folder into the animation frames.
5. Adjust the animation speed to achieve a natural flying effect.

### Configure the Collision Shape

1. Select the `CollisionShape2D` node.
2. Create a `CapsuleShape2D`.
3. Rotate the collision shape by 90 degrees.
4. Adjust its dimensions to match the bird's body.

The collision shape should cover the bird accurately without extending too far beyond its sprite.

---

## 3. Implement Bird Physics and Movement

Attach a script to the bird's root node. This script will control gravity, falling, rotation, animation, and flapping.

### Define the Variables

The bird script should include variables for:

* **Gravity:** Determines how quickly the bird falls.
* **Maximum velocity:** Limits the bird's falling speed.
* **Flap speed:** Controls the upward movement when the player flaps.
* **Flying flag:** Indicates whether the bird is actively flying.
* **Falling flag:** Indicates whether the bird should fall after a collision.
* **Start position:** Stores the bird's initial coordinates.

Adjust the physics values through testing to obtain the desired gameplay feel.

### Implement the Reset Function

Create a reset function that:

1. Restores the initial flying and falling states.
2. Resets the bird's position to its starting coordinates.
3. Resets its rotation.
4. Restores any other movement variables.

Call this function from `_ready()` so the bird initializes correctly.

### Implement Physics Processing

Use `_physics_process(delta)` to handle movement.

1. If the bird is flying, apply gravity to its vertical velocity.
2. Limit the velocity to the configured maximum.
3. Rotate the bird according to its movement.
4. Play the flying animation.
5. If the bird is falling after a collision, rotate it downward and stop the flying animation.
6. Use `move_and_collide()` or another appropriate `CharacterBody2D` movement method to update its position.
7. If the bird is no longer moving, stop its animation.

Ensure gravity is applied consistently and that terminal velocity is respected.

### Implement the Flap Function

Create a function that sets the bird's vertical velocity to the configured flap speed, applying upward movement.

Test the bird scene independently using **F6** (Run Current Scene). Verify that the bird appears correctly and responds to the movement logic.

Finally, return to the `Main` scene and instantiate the bird scene as a child.

---

## 4. Set Up the Main Game Script

Attach a script to the `Main` node to coordinate the gameplay.

### Define the Game Variables

Include variables for:

* **Game state flags:** Determine whether the game has started or ended.
* **Scroll offset:** Tracks the horizontal scrolling movement.
* **Score:** Tracks the player's progress.
* **Scroll speed:** Controls how quickly the environment moves.
* **Screen size:** Stores the dimensions of the game viewport.
* **Ground height:** Stores the height of the ground.
* **Pipes array:** Keeps track of the pipes currently in the game.
* **Pipe generation settings:** Control pipe spacing, timing, and vertical placement.

Use constants for values that should not change during gameplay.

### Initialize a New Game

Create a `new_game()` function that:

1. Resets the game-state flags.
2. Resets the scrolling offset and score.
3. Resets the bird using its reset function.
4. Clears any existing pipes.
5. Resets the score display.
6. Prepares the game for a new round.

Call this function from `_ready()` after the necessary nodes and references have been initialized.

### Handle Player Input

Create an input handler that responds to mouse clicks.

* If the game is over, ignore normal flap input.
* If the game has not started, use the first click to start it.
* If the game is running and the bird is flying, subsequent clicks make the bird flap.
* Check whether the bird has moved beyond the top of the screen when appropriate.

### Start the Game

Create a `start_game()` function that:

1. Sets the game-running flag.
2. Enables the bird's flying state.
3. Makes the bird flap for the first time.
4. Starts the pipe-generation timer.

Test the game to verify that the bird remains in its starting position until the player clicks, then begins moving under gravity.

---

## 5. Create the Scrolling Ground

The ground gives the game its continuous side-scrolling effect.

### Create the Ground Scene

1. Create a new scene with an `Area2D` root node.
2. Rename the root node to `Ground`.
3. Add a `Sprite2D` child.
4. Add a `CollisionShape2D` child.
5. Assign the ground image to the sprite's **Texture** property.
6. Position the ground at the bottom of the game window.
7. Adjust the sprite's X offset to align the image.
8. Create a `RectangleShape2D` collision shape that covers the ground surface.

### Add Collision Detection

Attach a script to the `Ground` node.

1. Define a custom signal named `hit`.
2. Connect the `body_entered` signal from `Area2D` to a handler.
3. Emit the custom `hit` signal when a body enters the ground.

This signal allows the main game script to respond when the bird touches the ground.

### Instantiate the Ground

Add the ground scene to `Main`.

### Implement Infinite Scrolling

In the main script:

1. Retrieve the viewport dimensions during initialization.
2. Increase the scroll offset in `_process(delta)` while the game is running.
3. When the offset reaches the width of the scrolling image, reset it to zero.
4. Update the ground's horizontal position using the scroll offset.

The ground image should be wide enough to support the intended looping effect. Adjust the offset and image dimensions so the transition between repetitions is seamless.

### Adjust the Draw Order

Select the `Ground` node and set its **Z Index** to `1`.

This ensures that the ground is drawn in front of the pipes where their sprites overlap.

Run the game and verify that the ground scrolls continuously.

---

## 6. Create the Pipes

Pipes form the obstacles that the bird must avoid.

### Create the Pipe Scene

1. Create a new scene with an `Area2D` root node.
2. Rename the root node to `Pipe`.
3. Save it inside the `scenes` folder.
4. Add two `Sprite2D` nodes for the upper and lower pipes.
5. Assign the pipe texture to both sprites.
6. Flip the upper pipe vertically.
7. Adjust the Y positions of the sprites to create the desired gap. For example, use offsets of approximately `380` and `-380`, then refine them to suit the artwork.

### Configure Pipe Collisions

Add collision shapes for the upper and lower pipes.

1. Create a `RectangleShape2D` for each pipe section.
2. Adjust the shape dimensions to match the visible pipe.
3. Position each shape so that it covers the obstacle accurately.
4. Repeat for both upper and lower pipe sections.

Avoid leaving large gaps between the sprite and its collision shape, as this can produce unexpected collisions.

### Add a Hit Signal

Attach a script to the `Pipe` root node.

1. Define a custom `hit` signal.
2. Connect the root `Area2D` node's `body_entered` signal to a handler.
3. Emit the custom signal when a body enters the pipe.

### Add the Score Area

1. Add another `Area2D` as a child of `Pipe`.
2. Rename it to `ScoreArea`.
3. Add a `CollisionShape2D` child.
4. Create a `RectangleShape2D`.
5. Position and resize it to form a narrow detection area just beyond the pipe exit.
6. Define a custom scoring signal and emit it when the bird passes through the area.

The score area must be separate from the obstacle collision shapes so that passing through it awards a point without killing the bird.

---

## 7. Generate and Scroll Pipes

Pipe instances should be created dynamically rather than manually placed in the scene.

### Add a Pipe Timer

1. Return to the `Main` scene.
2. Add a `Timer` node.
3. Rename it to `PipeTimer`.
4. Set its wait time to `1.5` seconds.
5. Enable the timer's one-shot setting only if the implementation explicitly restarts it after every timeout.

### Reference the Pipe Scene

Export a `PackedScene` variable in the main script and assign the pipe scene to it through the Inspector.

This allows the script to instantiate pipes at runtime.

### Calculate the Ground Height

Determine the ground height during initialization. Use this value to calculate the available vertical space for pipe placement.

### Connect the Timer

Connect `PipeTimer`'s `timeout` signal to a `generate_pipes()` function.

### Implement Pipe Generation

Inside `generate_pipes()`:

1. Instantiate a new pipe from the assigned pipe scene.
2. Set its X coordinate to the right edge of the screen plus a small offset.
3. Generate a random Y coordinate within the configured range.
4. Ensure the pipe placement respects the ground height and available screen space.
5. Connect the pipe's `hit` signal to the bird-collision handler.
6. Connect its scoring signal to the scoring handler.
7. Add the pipe to the main scene.
8. Store the pipe reference in the pipes array.

Use a configurable pipe range to control how far the pipes can vary vertically.

### Generate the Initial Pipes

In `new_game()`:

1. Clear the pipe list.
2. Create the initial pipe set.
3. Reset the scrolling offset.

Start the pipe timer from `start_game()` so that automatic generation begins when the player starts the round.

### Scroll the Pipes

Inside `_process(delta)`:

1. Check whether the game is running.
2. Increase the scroll offset.
3. Iterate through the pipes array.
4. Move each pipe to the left using the scrolling offset or an equivalent movement calculation.
5. Remove pipes that have moved completely off-screen.

Run the game to verify that new pipes appear from the right and move toward the bird.

---

## 8. Implement Collision Detection

The game must stop when the bird hits an obstacle or leaves the playable area.

### Check the Top Boundary

Create a `check_top()` function that checks the bird's vertical position.

If the bird moves above the top boundary:

1. Set the appropriate falling state.
2. Stop the game.
3. Prevent further flap input.

Call this function from the appropriate input or movement logic.

### Handle Pipe Collisions

Implement a `bird_hit()` function in the main script.

When the pipe's `hit` signal is emitted:

1. Set the bird's falling state.
2. Stop the game.
3. Stop the pipe timer.
4. Prevent additional scoring or flap input.

### Handle Ground Collisions

Connect the `Ground` scene's `hit` signal to a handler in the main script.

When the bird touches the ground:

1. Disable the bird's flying state.
2. Stop the game if it is still running.
3. Stop the bird's movement as required by the implementation.

Test the game to ensure that hitting a pipe causes the bird to fall and that touching the ground stops the round.

### Stop the Game

Create a `stop_game()` function that:

* Marks the game as no longer running.
* Stops the pipe timer.
* Disables further flap input.
* Updates the bird's movement state.
* Displays the game-over interface.

Keep the game-state flags consistent so that collision handlers do not repeatedly trigger the same game-over logic.

---

## 9. Add the Scoring System

The player earns points by passing through pipe pairs.

### Create the Score Label

1. Open the `Main` scene.
2. Add a `Label` node.
3. Rename it to `ScoreLabel`.
4. Set its initial text to `0`.
5. Position it near the top center of the screen.
6. Increase the font size to approximately `40`.
7. Set its **Z Index** to `1` so it appears above the background and gameplay objects.

### Connect the Score Area

The pipe scene should already contain a `ScoreArea` with its own collision shape and scoring signal.

In `generate_pipes()`:

1. Connect the pipe's scoring signal to a `scored()` function.
2. Ensure each pipe pair awards points only once.

### Implement Scoring

Inside `scored()`:

1. Increase the score by one.
2. Update `ScoreLabel` with the new score.

For example, the label should display `1`, `2`, `3`, and so on as the player passes successive pipe pairs.

### Reset the Score

Inside `new_game()`:

1. Set the score to zero.
2. Update `ScoreLabel` to display `0`.

Run the game and verify that points are awarded only when the bird passes through a pipe pair.

---

## 10. Create the Game Over Screen

The game-over interface allows the player to restart after a collision.

### Create the Game Over Scene

1. Create a new scene with a `CanvasLayer` root node.
2. Add a `Button` child node.
3. Give the nodes meaningful names, such as `GameOverScreen` and `RestartButton`.
4. Save the scene in the `scenes` folder.

### Style the Restart Button

1. Select the restart button.
2. Create a new `StyleBoxTexture` resource for its normal state.
3. Assign the restart button image.
4. Repeat for the hover and pressed states if corresponding images are available.
5. Adjust the size and position to match the intended design.

### Add a Restart Signal

Attach a script to the `CanvasLayer` root node.

1. Define a custom `restart` signal.
2. Connect the button's `pressed` signal to a handler.
3. Emit `restart` when the button is pressed.

### Integrate the Game Over Screen

1. Instantiate the game-over scene inside `Main`.
2. Hide it when `new_game()` is called.
3. Show it when `stop_game()` is called.
4. Connect the `restart` signal to the main script's `new_game()` function.

Run the game and confirm that the restart button appears after the bird dies.

---

## 11. Implement the Restart System

Before finishing, make sure that restarting the game clears all pipes from the previous round.

### Create a Pipe Group

1. Open the pipe scene.
2. Select the root `Pipe` node.
3. Open the **Node** dock and select the **Groups** tab.
4. Create a group named `pipes`.
5. Ensure every instantiated pipe belongs to this group.

### Remove Old Pipes

Inside `new_game()`:

1. Find the nodes belonging to the `pipes` group.
2. Remove them using `queue_free()`.
3. Clear the pipes array.
4. Reset the score and scrolling variables.
5. Reset the bird's position and movement.
6. Hide the game-over screen.
7. Prepare the pipe timer for the next round.

Be aware that `queue_free()` schedules nodes for deletion at the end of the frame. Avoid creating duplicate initial pipes before the previous round's pipes have been properly handled.

### Test the Full Game

Verify the following:

* The bird remains stationary until the first click.
* Clicking starts the game.
* Subsequent clicks make the bird flap.
* Gravity pulls the bird downward.
* The ground scrolls continuously.
* Pipes spawn at different heights.
* The bird dies when it hits a pipe or the ground.
* Moving above the top of the screen ends the round.
* Passing through pipes increases the score.
* The game-over screen appears after death.
* Clicking restart resets the score, bird, pipes, and game state.

---

## 12. Final Result

Congratulations! You now have the structure of a basic Flappy Bird clone built in Godot.

The project includes:

* A scrolling background and ground.
* An animated bird with gravity and flap controls.
* Dynamically generated pipe obstacles.
* Collision detection for pipes, ground, and screen boundaries.
* A scoring system.
* A game-over screen and restart button.
* A reset mechanism for starting new rounds.

**Next steps:** You can improve the game by adding sound effects, background music, difficulty progression, a start screen, high-score persistence, and mobile touch controls.

For the accompanying implementation and project files, see the [Flappy Bird repository](https://github.com/ASLAN31122003/Flappy_Bird) and the [development guide source](https://github.com/ASLAN31122003/Flappy_Bird/blob/main/guide.c).

