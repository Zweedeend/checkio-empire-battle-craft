**In Campaign mode, you can check your strategies on already defeated bases. You will not lose your troops.**

Let's work on some new code for our units. All units in the current craft run the same code which starts when a battle begins. The script you are currently writing will command one craft. So if a craft has 7 units inside, that means 7 copies of this script will be launched.

The commanding principles are based on the 3 main groups of functions.

**Asks** are started with _ask_. Ask functions provide information about the unit you control or environment around it. For example, check out the following code...

```python
from battle import commander
unit_client = commander.Client()
my_info = unit_client.ask_my_info()
print("My ID:{}".format(my_info['id']))
```

... this shows a current unit's ID in the battle console.

**Actions** are started with _do_. The Action function sends a command to a unit. The unit can only hold information about the last command, so every following command will overwrite previous one. For example, check out the following code...

```python
from battle import commander
unit_client = commander.Client()
unit_client.do_move((30, 30))
unit_client.do_move((20, 30))
```

... that code commands units to go to the point (20, 30), but the unit will never get to the point (30, 30).

**Subscriptions** are started with _when_. The subscribe function always has a callback argument. Callback is the function that gets called when a specific event occurs. For example, check out the following code...

```python
from battle import commander
unit_client = commander.Client()

def attack_near_enemy(data):
    unit_client.do_attack(data['id'])

unit_client.when_enemy_in_range(attack_near_enemy)
```

... that commands the unit to attack any enemy that comes into its firing range.

**Prints**. Feel free to use the _print_ function and see every script's output in the right-hand panel for battle replays.


**Your main goal is to destroy the enemy center**.


## Battle Field

The battle field has a size of 40 by 40 tiles, but the half of that is occupied by rocks. The zero coordinates are placed in the top corner. This image should help you to understand how axises are situated:
 
![Map Axises](../media/map.png)

## Items

Units, towers, buildings and other objects on the map are called "items". When you ask for info about specific items, you will receive a dictionary with the item data, or a list of these dictionaries. The item info can contain various fields, so it is better to use the `dict.get` method. An item can have the following keys:

- "id": (int) Unique identifier for the item. All items have this field.
- "player_id": (int) the ownership of the item.
- "role": (str) Describes the role of the item. It can be a `unit`, `tower`, `building`, `center`, or `obstacle`. You can read more below on the different roles.
- "type": (str) Describes the type of the item. It can be a `sentryGun`, `infantryBot` etc.
- "hit_points": (int/float) Defines the durability of the item. If "hit_points" is zero or lower, the item is destroyed.
- "coordinates": (list of two int/float): The item's location coordinates. Units are single point objects.
  For large objects such as buildings, this field contains the coordinates of the center (middle) point.
- "size": (int/float) Units don't have a size. All static objects (buildings, towers etc) are square and the edge length is equal to their "size".
- "speed": (int/float) This is a unit attribute only. It describes how fast the unit may move.
- "damage_per_shot": (int/float) This is a unit/tower attribute which describes how many hit points an attack will take.
- "rate_of_fire": (int/float) This is a unit/tower attribute which describes how many attacks per second the item can take.
- "firing_range": (int/float) This is a unit/tower attribute which describes the maximum distance it can attack.

### Roles

You can use predefined constants instead of string variables.

```python
from battle import ROLE
```

- `unit` - Mobile fighting items, these come from crafts. `ROLE.UNIT`
- `tower` - Stationary fighting items. `ROLE.TOWER`
- `center` - Command Centers, the main building in the game. If they're destroyed, then a battle is over. `ROLE.CENTER`
- `building` - All other stationary buildings. `ROLE.BUILDING`
- `obstacle` - Neutral stationary objects like rocks or plants. `ROLE.OBSTACLE`

## Ask info

- `ask_my_info()` Returns information about the current item.

- `ask_item_info(item_id)` Returns information about the item with `id == item_id` or None.

- `ask_enemy_items()` Returns a list with information on the enemy items.

- `ask_my_items()` Returns a list with information on your items.

- `ask_buildings()` Returns a list with information for all buildings including the Command Center.

- `ask_towers()` Returns a list with information for all towers.

- `ask_center()` Returns information about the Command Center.

- `ask_units()` Returns a list with information for all units.

- `ask_nearest_enemy()` Returns a list with information on all enemies in the current item's firing range.

- `ask_nearest_enemy(role_list)` Returns information about the nearest enemy item with role from `role_list`.

```python
from battle import ROLE
near_tower = unit_client.ask_nearest_enemy([ROLE.TOWER])

```

- `ask_my_range_enemy_items()`  
    Returns a list with information on all enemies in the current items firing range.

- `ask_cur_time()`
    Returns current in-game time. (secs)

## Commands.

- `do_attack(item_id)` Attack the item with `id == item_id`.
    If the target is too far, then the unit will move to the target.

- `do_move(coordinates)` move to the point with the given coordinates. _coordinates_: list/tuple of two int/float.

- `do_moves(steps)` A unit only command.
    Move through the list of coordinates.

```python
unit_client.do_moves([
  [35, 35],
  [35, 20]
])
def do_attack_center(*args):
  unit_client.do_atack(unit_client.ask_center()['id'])

unit_client.when_idle(do_attack_center)

```

### LEVEL 4

for units with level 4 or more.

- `do_message_to_id(message, item_id)` send a message to a unit with `item_id`.

- `do_message_to_craft(message)` send a message to all units from your craft.

- `do_message_to_team(message)` send a message to all units from your team.


## Subscribes.

You can subscribe your units to an event and when this event occurs the _callback_ function
will be called. The callback function will receive input data related to the subscription.
All subscriptions are disposable and removed when triggered.

- `when_in_area(center, radius, callback)` Is triggered when the current unit is in the circle. _center_ describes the coordinates of the center point and _radius_ describes the length of the circle's radius.

- `when_item_in_area(center, radius, callback)` The same as `whenInArea` but gets triggered for any item.

- `when_idle(callback)` Is triggered when the current unit is idle (finishes moving,
  destroys an enemy or doesn't have commands).

- `when_enemy_in_range(callback)` Is triggered when an enemy item is in the current item's
   firing range.

- `when_enemy_out_range(item_id, callback)` Is triggered when the item with _item_id_ is
  out of the current item's firing range.

- `when_item_destroyed(item_id, callback)` Is triggered when the item with _item_id_ is destroyed.

### LEVEL 2

for units with level 2 or more.

- `when_time(secs, callback)` Is triggered at a specific game time. Very useful for the synchronization of units.

### LEVEL 4

for units with level 4 or more.

- `when_message(callback, infinity=True)` Is triggered when a unit gets a message from another unit. The `infinity` argument indicates that you don't need to subscribe on the event again after getting the message and should be used if you want use `when_message` again. The `callback` function gets one argument as a dict with the `message` and `from_id` keys.
