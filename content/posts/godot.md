---
title: "Godot"
date: 2024-08-18T20:06:36+09:00
draft: true
---

# Godot

## Learn to Code From Zero with Godot

[Learn to Code From Zero with Godot](https://gdquest.github.io/learn-gdscript/)  

### Lesson 01 - What Code is Like

```
print("Welcome!")
```

### Lesson 02 - Your First Error

```
func this_code_is_wrong():
    #return
```

### Lesson 03 - We Stand on the Shoulders of Giants

```
show()
hide()
rotate(0.3)
move_local_x(20)
move_local_y(20)
```

```
PI
```

### Lesson 04 - Drawing a Rectangle

```
move_forward(200)
turn_left(45)
turn_right(45)
```

### Lesson 05 - Coding Your First Function

```
func move_and_rotate():
    move_forward(200)
    turn_right(90)

func draw_square(size):
    move_forward(size)
    turn_right(90)
    move_forward(size)
    turn_right(90)
    move_forward(size)
    turn_right(90)
    move_forward(size)
    turn_right(90)
```

```
jump(-100,50)
```

The instructions inside the function must all start with a leading tab character.  

Identifiers also have to start with a letter or an underscore; You can't begin with a number, but you can use numbers after the first character.  
Note that it's fine to use capital letters.  

### Lesson 06 - Your First Function Parameter

```
func move_local_xy(offset_x, offset_y):
    move_local_x(offset_x)
    move_local_y(offset_y)
```

### Lesson 07 - Introduction to Member Variables

```
position
scale
```

```
rotation = 0
position.x = 200
position.y = 250
scale.x = 1.5
scale.y = 1.5
```

### Lesson 08 - Defining Your Own Variables

```
var health = 5
health = health - 1
print(health)
```

```
var health
var health  # Error
```

```
var health = 100
health = "This is some text"
```

Variables are case-sensitive, which means health and Health are technically different variables.  

### Lesson 09 - Adding and Subtracting

```
health -= amount
health = health - amount

health += amount
health = health + amount
```

### Lesson 10 - The Game Loop

```
func _process(delta):
    rotate(0.05)
```

This function can run hundreds of times a second.  
The _process() function takes one parameter: delta.  
If the function exists in your code, and it is called precisely _process, then Godot will automatically run it every frame.  
When Godot draws on the screen, we call that a frame.  
The faster your computer, the more times _process() will run.  
Godot will try and run _process() as quickly as it can.  

### Lesson 11 - Time Delta

It's not just our character that has a _process() function; Almost everything in the game has a _process() function!  
Dozens of times per second, Godot runs every _process() function in the game to update the game world.  
After that, it displays an image of the game world on the screen. We call that image a frame.  
Godot then moves on to calculating the next frame.  

Depending on the game, the computer, and what the game engine needs to calculate, frames take more or less time to display.  
There will always be milliseconds variations frame to frame.  
That is why the _process() function receives a delta parameter.  
Delta represents a time difference. It's the time passed since the previous frame, in seconds.  

```
func _process(delta):
    move_local_x(100 * delta)
```

The robot moves 100 pixels per second.  

### Lesson 12 - Using Variables to Make Code Easier to Read

```
var angular_speed = 4

func _process(delta):
    rotate(angular_speed * delta)

func set_angular_speed(new_angular_speed):
    angular_speed = new_angular_speed
```

```
func _process(delta):
    var angular_speed = 4
    rotate(angular_speed * delta)

func set_angular_speed(new_angular_speed):
    angular_speed = new_angular_speed  # Error
```

### Lesson 13 - Conditions

```
true
false
```

```
func heal(amount):
    health += amount
    if health > 100:
        health = 100
```

```
if health > 100:  # If health is greater than 100
    pass
if health < 50:  # If health is less than 50
    pass
if health == 25:  # If health is equal to 25
    pass
if health != 7:  # If health is not equal to 7
    pass
```

```
func run():
    var speed = 120
    if speed < 100:
        print("You're slow!")
    else:
        print("You're fast!")
```

### Lesson 14 - Multiplying

```
var level = 5
var power = 3

func calculate_damage():
    var damage = power * level
    damage *= 2
```

### Lesson 15 - 2D Vectors

```
func level_up():
    scale.x += 0.2
    scale.y += 0.2
```

In Godot, 2D vectors are a common value type named Vector2,    

```
func level_up():
    scale += Vector2(0.2, 0.2)

func move():
    position += Vector2(20, 20)
```

```
func reset_robot():
    position = Vector2(0, 0)
    scale = Vector2(1.0, 1.0)
```

### Lesson 16 - Introduction to While Loops

```
var number = 0
while number < 4:
    print(number)
    number += 1
```

### Lesson 17 - Introduction to For Loops

```
for number in range(3):
    print(number)
```

Calling range(3) outputs the list of numbers [0, 1, 2],  
This code behaves the same as the previous example:  

```
for element in [0, 1, 2]:
    print(element)
```

### Lesson 18 - Creating arrays

### Lesson 19 - Looping over arrays

### Lesson 20 - Strings

### Lesson 21 - Functions that return a value

### Lesson 22 - Appending and popping values from arrays

### Lesson 23 - Accessing values in arrays

### Lesson 24 - Creating Dictionaries

### Lesson 25 - Looping over dictionaries

### Lesson 26 - Value types

### Lesson 27 - Specifying types with type hints
