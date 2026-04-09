# Chapter 1: Python Basics — From Zero to Advanced Mechanics

This guide is designed for absolute beginners but scales rapidly to professional-grade mechanics. Whether you have never written a line of code or you want to understand how Python manages memory under the hood, this chapter covers the foundational engine of the language.

*(Note: Data Structures, Object-Oriented Programming, and Exception Handling are covered in their own dedicated chapters).*

---

## LEVEL 1: THE ABSOLUTE BASICS (Zero Prior Experience)

Before we build software, we must understand how to talk to the computer. Python is an "interpreted" language, meaning a program reads your code line by line and translates it into instructions the computer's processor can understand.

### 1.1 The `print()` Function and Strings
The most basic action in programming is making the computer display text on the screen. Text in programming is called a **String** and must always be wrapped in quotes.

```python
# This is a "comment". The computer ignores anything after the '#' symbol.
# We use comments to leave notes for humans reading the code.

print("Hello, World!") 
print('You can use single quotes too.')
```

### 1.2 Primitive Data Types
A "data type" tells the computer what kind of information it is handling. Python has four core primitive data types:

1.  **Strings (`str`):** Text. Always in quotes. `"Skill Swap"`
2.  **Integers (`int`):** Whole numbers. `42`
3.  **Floats (`float`):** Decimal numbers. `3.14`
4.  **Booleans (`bool`):** True or False (must be capitalized). `True`

### 1.3 Variables
A variable is a name tag you create to store data so you can use it later. You assign data to a variable using the equals sign `=`.

```python
user_name = "Zain"
account_age_days = 15
balance = 45.50
is_verified = True

print(user_name) # Output: Zain
```

### 1.4 String Formatting (F-Strings)
If you want to combine variables with text, do not use the `+` symbol (which gets messy). Use **F-strings** (formatted strings). Just put an `f` before the quotes, and wrap your variables in curly braces `{}`.

```python
# The modern, professional way to inject data into text
welcome_message = f"Welcome back, {user_name}! Your balance is ${balance}."
print(welcome_message)
```

---

## LEVEL 2: INTERMEDIATE MECHANICS (Logic and Actions)

Now that we can store data, we need to know how to manipulate it, group it into actions, and make decisions.

### 2.1 Operators, Math, and Casting
Python acts as a powerful calculator. If you receive a number as text (a string), you must "cast" (convert) it before doing math.

```python
x = 10
y = 3

print(x + y)  # Addition: 13
print(x / y)  # Division: 3.333...
print(x // y) # Floor Division (removes the decimal): 3
print(x % y)  # Modulus (returns the remainder): 1

# Casting: Converting a String to an Integer
age_input = "25" 
real_age = int(age_input)
print(real_age + 5) # Output: 30
```

### 2.2 Functions: The Building Blocks of Logic
A function is a reusable block of code. You define it once using `def`, and use it as many times as you want. 
* **`print()`** just shows something on the screen. 
* **`return`** actually hands data back to the program so you can use it in calculations.

```python
# 1. Defining the function (It takes two arguments: price and tax_rate)
def calculate_total(price, tax_rate):
    tax_amount = price * tax_rate
    final_price = price + tax_amount
    # Hand the final number back to whoever called this function
    return final_price 

# 2. Calling the function
# We pass 100 for the price, and 0.05 (5%) for the tax_rate
checkout_amount = calculate_total(100, 0.05)

print(f"Your total today is ${checkout_amount}")
```

### 2.3 Control Flow (`if`, `elif`, `else`)
Programs make decisions using conditions.
* `==` (Equal to)
* `!=` (Not equal to)
* `>` (Greater than), `<` (Less than)

```python
user_age = 18

if user_age >= 18:
    print("Access Granted.")
elif user_age == 17:
    print("Come back next year.")
else:
    print("Access Denied.")
```

### 2.4 Basic Loops (`while` and `for`)
Loops repeat actions so you don't have to write the same code over and over.

```python
# 1. The 'while' loop: Runs as long as a condition is True
countdown = 3
while countdown > 0:
    print(countdown)
    countdown = countdown - 1

# 2. The 'for' loop: Runs a specific number of times using range()
for number in range(3):
    print(f"Sending email #{number}")
```

---

## LEVEL 3: ADVANCED ENGINEERING (Professional Practices)

This section covers how Python manages data under the hood and the modern syntax used by software engineers to write scalable, bug-free applications.

### 3.1 Memory Allocation and Mutability
When you create a variable, Python allocates space in your computer's RAM. 
* **Immutable (Strings, Integers):** Once created, the data in that exact RAM slot *cannot change*. If you change the variable, Python creates a brand new slot in RAM and moves the variable's name tag to it.
* **Mutable (Lists):** The data in the RAM slot *can* be changed in place.

```python
# Use the id() function to see the exact RAM memory address
score = 100
print(f"Original Memory ID: {id(score)}")

score = score + 1
# Because integers are Immutable, Python created a new '101' in memory.
print(f"New Memory ID:      {id(score)}") # The ID is completely different

# The 'is' operator checks if two things share the exact same memory ID
a = [1, 2]
b = [1, 2]
print(a == b) # True (The values are identical)
print(a is b) # False (They exist in different places in memory)
```

### 3.2 Scope: The LEGB Rule
"Scope" dictates where a variable can be seen or used. Python checks for variables in a strict order: **L**ocal, **E**nclosing, **G**lobal, **B**uilt-in.

```python
# This is a GLOBAL variable. The whole file can see it.
system_status = "Online" 

def check_system():
    # This is a LOCAL variable. It ONLY exists inside this function.
    local_ping = 24 
    print(f"Status: {system_status}, Ping: {local_ping}ms")

check_system()
# print(local_ping) # THIS WILL CRASH! The outside world cannot see local variables.
```

### 3.3 Type Hinting (Python 3.5+)
In massive systems, not knowing what type of data a variable holds causes crashes. Professionals use "Type Hints" to strictly declare what is expected. 

```python
from typing import Optional

# We strictly define that 'name' is a string, 'age' is an int,
# and the function MUST return a boolean (True/False).
def verify_user(name: str, age: int, promo_code: Optional[str] = None) -> bool:
    if age < 18:
        return False
    print(f"Verified {name} with code {promo_code}")
    return True

# If another developer tries to pass a string for the age, 
# their IDE will flag a severe warning before the code runs.
is_valid: bool = verify_user("Sufiyan", 22) 
```

### 3.4 Advanced Control Flow: Structural Pattern Matching
Introduced in Python 3.10, this is a highly advanced way to process complex incoming data (like API requests from a web server).

```python
def handle_server_response(response: dict):
    # 'match' analyzes the shape and contents of the data
    match response:
        case {"status": 200, "user": username}:
            # Matches if status is 200, and saves the user to a variable
            print(f"Login success for: {username}")
            
        case {"status": 403 | 404}: # Matches either 403 or 404
            print("Error: Denied or Not Found.")
            
        case _: # The fallback 'else' equivalent
            print("Unrecognized server response.")

handle_server_response({"status": 200, "user": "Rayyan"})
```

### 3.5 Advanced Control Flow: The Ternary Operator
Professionals condense simple `if/else` statements into a single, optimized line.

```python
system_load = 90

# Standard way
if system_load > 85:
    status = "Critical"
else:
    status = "Stable"

# Advanced Ternary way (Does the exact same thing)
# Syntax: [Result if True] if [Condition] else [Result if False]
status = "Critical" if system_load > 85 else "Stable"
```

### 3.6 Advanced Iteration Protocols
Never use manual counters (`i = 0`) in Python loops. Use the built-in C-optimized iteration tools.

```python
users = ["Zain", "Rayyan"]
roles = ["Admin", "Developer"]

# 1. enumerate(): Gives you the index and the value simultaneously
for index, user in enumerate(users, start=1):
    print(f"{index}. {user}")

# 2. zip(): Combines multiple lists and iterates through them together
for user, role in zip(users, roles):
    print(f"{user} is assigned as {role}")

# 3. The for...else construct
# The 'else' block only triggers if the loop finishes WITHOUT hitting 'break'
target = "Sufiyan"
for user in users:
    if user == target:
        print("User found!")
        break
else:
    print("User was not found in the database.")
```

### 3.7 Imports and The Standard Library
Python comes with a massive "Standard Library" of pre-written code. You use the `import` keyword to bring these tools into your file.

```python
# Importing built-in modules
import math
import random

# Using a function from the math module
root = math.sqrt(64)
print(f"Square root is: {root}")

# Using a function from the random module
lucky_number = random.randint(1, 100)
print(f"Your lucky number is: {lucky_number}")
```
