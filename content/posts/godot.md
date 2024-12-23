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

```
[1, 2, 6, 7]
[Vector2(0, 0), Vector2(4, 3), 5, -1.0]
var numbers = [1, 2, 6, 7]
var turtle_path = [
		Vector2(1, 0),
		Vector2(1, 1),
		Vector2(2, 1),
		Vector2(3, 1),
		Vector2(4, 1),
		Vector2(5, 1),
		Vector2(5, 2),
		Vector2(5, 3),
]
```

```
void select_units(cells: Array)
```

```
select_units([
		Vector2(0, 3),
		Vector2(1, 0),
		Vector2(4, 2),
		Vector2(5, 1),
])
```

### Lesson 19 - Looping over arrays

```
var numbers = [0, 1, 2]
for number in numbers:
    print(number)
```

```
for cell in cells:
    if cell in units:
        selected_units.append(cell)
```

In a condition, the in keyword allows you to check if a value exists in an array.  
The array's append() function appends a new value at the end of the array.  

```
void draw_rectangle(length: float, height: float)
void jump(x: float, y: float)
```

```
var rectangle_sizes = [Vector2(200, 120), Vector2(140, 80), Vector2(80, 140), Vector2(200, 140)]

func run():
	for x in rectangle_sizes:
		draw_rectangle(x.x, x.y)
		jump(x.x + 100, 0)
```

### Lesson 20 - Strings

```
String
```

```
"This is a text string."
var my_name = "Robot"
print(my_name)
```

```
for character in "Robot":
    print(character)  # R
                      # o
                      # b
                      # o
                      # t
```

```
var combo = ["jump", "damage", "damage", "jump", "level"]
```

```
combo = ["jab", "jab", "uppercut"]
for x in combo:
	play_animation(x)
```

### Lesson 21 - Functions that return a value

```
round(11.4)  # 11
round(38.5)  # 39
```

The lerp() function, short for linear interpolate, calculates and returns a weighted avarage between two values.  
It takes three arguments: the two values to average and a value between 0.0 and 1.0 to skew the result.  

```
func _process(delta):
    position = lerp(position, get_local_mouse_position(), 2 * delta)
```

```
var cell_size = Vector2(120, 120)

func convert_to_world_coordinates(cell):
    return cell * cell_size + cell_size / 2  # Vector2(1, 1) * Vector2(120, 120) + Vector2(120, 120) / 2

var world_position = convert_to_world_coordinates(Vector2(1, 1))
print(world_position)  # Vector2(180, 180)
```

### Lesson 22 - Appending and popping values from arrays

Queue

```
var waiting_orders = []
var completed_orders = []

func add_order(meal_name):
    waiting_orders.append(meal_name)

func complete_current_order():
    var first_order = waiting_orders.pop_front()
    completed_orders.append(first_order)
```

Stack

```
var crates = ["sword", "gems", "healing heart"]

func use_top_crate():
    var crate = crates.pop_back()
    use(crate)
```

```
var crates = ["sword", "shield", "gems"]

crates.pop_back()
print(crates)  # [sword, shield]
```

```
var crates = ["healing heart", "shield", "gems", "sword"]

func run():
	while crates:
		crates.pop_back()
```

You can check if the crates array still contains values by writing while crates:  

### Lesson 23 - Accessing values in arrays

```
var inventory = [
    "sword",
    "shield",
    "healing heart",
    "gems",
    "healing heart",
    "sword",
]

use_item(inventory[3])  # gems
use_item(inventory[-2])  # healing heart
print(inventory.size())  # 6
```

### Lesson 24 - Creating Dictionaries

```
var inventory = {
    "healing heart": 3,
    "gems": 5,
    "sword": 1,
}

var item_count = inventory["gems"]
inventory["healing heart"] += 1
```

A key could be a string, a number, or even a vector!  
Every key has to be unique.  

### Lesson 25 - Looping over dictionaries

To get the list of keys in the dictionary, you can call its keys() member function.  

```
var inventory = {
    "healing heart": 3,
    "gems": 5,
    "sword": 1,
}

for item_name in inventory.keys():
    print(item_name)  # healing heart
                      # gems
                      # sword
```

Instead, you can directly type the variable name in a for loop after the in keyword. The language understands that you implicitly want to loop over the dictionary's keys.  

```
var inventory = {
    "healing heart": 3,
    "gems": 5,
    "sword": 1,
}

for item_name in inventory:
    var item_count = inventory[item_name]
    print(item_name + ": " + str(item_count))  # healing heart: 3
                                               # gems: 5
                                               # sword: 1
```

```
var unit_cells = {
    Vector2(1, 0): "robot",
    Vector2(2, 2): "turtle",
    Vector2(3, 0): "robot",
}

for cell in unit_cells:
    var unit = unit_cells[cell]
```

### Lesson 26 - Value types

You can get the text representation of a value by calling the str() function (short for "string"). The function returns its argument as a new String.  
You can use this function whenever you want to turn some number or vector into text.  

You can also convert strings into whole numbers or decimal numbers using respectively the int() and float() functions.  

```
print(Vector2(1, 1) * 10)  # (10, 10)
```

```
print(3 / 2)  # 1
print(3.0 / 2.0)  # 1.5
```

### Lesson 27 - Specifying types with type hints

```
var cell_size: Vector2 = Vector2(50.0, 50.0)
```

Type Inference  

```
var cell_size := Vector2(50.0, 50.0)
```

```
var vector: Vector2 = Vector2(1, 1)
var text: String = "Hello, world!"
var whole_number: int = 4
var decimal_number: float = 3.14
```
