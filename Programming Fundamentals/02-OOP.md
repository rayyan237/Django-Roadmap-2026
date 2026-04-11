# Chapter 2: Object-Oriented Programming (OOP) — Zero to Advanced

Procedural programming (writing lists of functions and variables) works for small scripts. But when building massive systems, procedural code becomes an unmanageable mess. 

Object-Oriented Programming (OOP) solves this by organizing code into "Objects" that map directly to real-world entities, managing their own state and behavior.

---

## LEVEL 1: THE ABSOLUTE BASICS (Zero Prior Experience)

### 1.1 What is an Object? What is a Class?
Think of a **Class** as a blueprint for a server. You cannot plug cables into a blueprint. 
Think of an **Object** (also called an **Instance**) as the actual, physical server built from that blueprint. You can build 100 servers from one blueprint; they all share the same structural design, but they have different IP addresses and different data stored inside.

* **Attributes:** The data the object holds (e.g., the server's IP, the user's name).
* **Methods:** The actions the object can perform (e.g., rebooting the server, updating a password).

### 1.2 Writing Your First Class
In Python, we use the `class` keyword. The most important part of a class is the `__init__` method (short for initialize). This is the "constructor"—it runs automatically the exact moment you build a new object.

```python
# 1. Defining the Blueprint (Class names are always Capitalized/PascalCase)
class SystemUser:
    
    # The constructor. 'self' is the most important concept here.
    # 'self' refers to the specific object being created right now.
    def __init__(self, username, role):
        self.username = username  # Assigning the input to the object's attribute
        self.role = role          # Assigning the input to the object's attribute
        self.is_active = True     # A default attribute every user gets automatically
        
    # A method (behavior). Notice it MUST take 'self' as the first parameter.
    def introduce(self):
        print(f"Hi, I am {self.username}. My role is {self.role}.")

# 2. Building the Objects (Instantiation)
# We do not pass anything for 'self'. Python handles that automatically.
user_1 = SystemUser("Rayyan", "Project Manager")
user_2 = SystemUser("Zain", "Lead Developer")

# 3. Interacting with the Objects
print(user_1.username) # Accessing data: Output is "Rayyan"
user_2.introduce()     # Triggering behavior: Output is "Hi, I am Zain..."
```

### 1.3 Instance Attributes vs. Class Attributes
* **Instance Attributes** (attached to `self`) belong to the specific object. 
* **Class Attributes** are declared outside `__init__` and are shared across *all* objects built from that class.

```python
class DatabaseConnection:
    # Class Attribute: Shared by all connections
    max_connections_allowed = 100
    active_connections = 0

    def __init__(self, db_name):
        self.db_name = db_name # Instance Attribute: Unique to this object
        DatabaseConnection.active_connections += 1

conn1 = DatabaseConnection("UsersDB")
conn2 = DatabaseConnection("ProductsDB")

print(DatabaseConnection.active_connections) # Output: 2
```

---

## LEVEL 2: INTERMEDIATE MECHANICS (Architecture & Design)

OOP relies on core design principles: Inheritance, Polymorphism, Encapsulation, and Composition.

### 2.1 Inheritance (Code Reuse)
Inheritance allows a new class (Child) to copy everything from an existing class (Parent) and then add its own unique features.

```python
class BaseUser:
    def __init__(self, username):
        self.username = username

    def login(self):
        print(f"{self.username} has logged in.")

# Child Class inherits from BaseUser
class Admin(BaseUser):
    def __init__(self, username, clearance_level):
        # super() calls the Parent's constructor so we don't rewrite code
        super().__init__(username) 
        self.clearance_level = clearance_level

    def ban_user(self, target):
        print(f"ADMIN: {self.username} banned {target}")

admin_account = Admin("Rayyan", level=5)
admin_account.login() # Inherited from BaseUser
admin_account.ban_user("Spammer99") # Unique to Admin
```

### 2.2 Polymorphism & Duck Typing
In languages like Java, if you want a function to accept different types of objects, they must share a strict Parent class. Python uses **Duck Typing**: *"If it walks like a duck and quacks like a duck, it must be a duck."* Python doesn't care what the object *is*, only what it can *do*.

```python
class PDFDocument:
    def print_data(self):
        print("Printing PDF pages...")

class ImageFile:
    def print_data(self):
        print("Printing Image pixels...")

# This function doesn't care if 'document' is a PDF, an Image, or a User.
# It only cares that the object has a .print_data() method.
def send_to_printer(document):
    document.print_data()

doc1 = PDFDocument()
doc2 = ImageFile()

send_to_printer(doc1) # Works perfectly
send_to_printer(doc2) # Works perfectly
```

### 2.3 Multiple Inheritance & MRO (Method Resolution Order)
Python allows a class to inherit from *multiple* parents at the same time. If both parents have a method with the same name, Python uses the Method Resolution Order (MRO) to decide which one to run—searching from left to right.

```python
class Coder:
    def work(self):
        print("Writing Python code...")

class Reviewer:
    def work(self):
        print("Reviewing pull requests...")

# LeadDeveloper inherits from BOTH. Coder is listed first (Left-to-Right).
class LeadDeveloper(Coder, Reviewer):
    pass

lead = LeadDeveloper()
lead.work() # Outputs: "Writing Python code..." (Because Coder was first)

# You can always check the exact search order using .mro()
print(LeadDeveloper.mro())
```

### 2.4 Mixins (Professional Multiple Inheritance)
Mixins are small classes that contain specific, reusable features. You do not instantiate a Mixin directly; instead, you "mix it in" to other classes using multiple inheritance.

```python
# A Mixin just provides a specific utility
class JSONSerializerMixin:
    def to_json(self):
        import json
        # __dict__ grabs all the object's instance attributes as a dictionary
        return json.dumps(self.__dict__)

# We "mix in" the JSON feature to our User class
class AppUser(JSONSerializerMixin):
    def __init__(self, name, email):
        self.name = name
        self.email = email

user = AppUser("Sufiyan", "sufi@dev.com")
print(user.to_json()) # Output: {"name": "Sufiyan", "email": "sufi@dev.com"}
```

### 2.5 Composition ("Has-A" vs. "Is-A")
Beginners overuse Inheritance ("A Car *is a* Vehicle"). Professionals favor Composition ("A Server *has a* Database"). Instead of inheriting, you put one object *inside* another.

```python
class Engine:
    def start(self):
        print("Engine roaring to life!")

# Car DOES NOT inherit from Engine. It HAS an Engine.
class Car:
    def __init__(self):
        self.engine = Engine() # Storing an object inside another object

    def drive(self):
        self.engine.start()
        print("Car is moving.")

my_car = Car()
my_car.drive()
```

### 2.6 Encapsulation (Access Control)
You do not want other parts of your program accidentally changing sensitive data (like a password hash). In Python, an underscore `_` signals "protected" data, and a double underscore `__` enforces "private" data (via name mangling).

```python
class Wallet:
    def __init__(self, owner, starting_credits):
        self.owner = owner
        self.__credits = starting_credits # Private. Cannot be accessed directly.

    def get_balance(self):
        return self.__credits

wallet = Wallet("Zain", 100)
# print(wallet.__credits) # THIS WILL CRASH! Python hides it.
print(wallet.get_balance()) # Safe, allowed access.
```

---

## LEVEL 3: ADVANCED ENGINEERING (Professional Standards)

This is how senior engineers structure object-oriented code, optimizing for readability, speed, memory, and metaprogramming.

### 3.1 Properties (Getters, Setters, and Deleters)
Python provides the `@property` decorator to make methods *look* and *act* like standard variables. This allows you to trigger complex validation or cleanup logic invisibly.

```python
class ProfileStats:
    def __init__(self):
        self.__trust_score = 0

    # 1. The Getter
    @property
    def trust_score(self):
        return self.__trust_score

    # 2. The Setter: Intercepts changes
    @trust_score.setter
    def trust_score(self, new_value):
        if not isinstance(new_value, int) or not (0 <= new_value <= 100):
            raise ValueError("Score must be an integer between 0 and 100.")
        self.__trust_score = new_value
        print(f"Audit: Score updated to {new_value}")

    # 3. The Deleter: Intercepts the 'del' keyword
    @trust_score.deleter
    def trust_score(self):
        print("Audit: Trust score wiped from database.")
        del self.__trust_score

stats = ProfileStats()
stats.trust_score = 85 # Runs the setter
del stats.trust_score  # Runs the deleter safely
```

### 3.2 The Object Lifecycle (`__new__` vs `__init__`) and Singletons
Beginners think `__init__` creates an object. It doesn't. 
1. `__new__` actually allocates the memory and *creates* the object.
2. `__init__` takes that blank object and *fills* it with data.

By overriding `__new__`, you can create a **Singleton**—a class that refuses to create more than one instance of itself in memory (perfect for Database connection pools).

```python
class DatabasePool:
    _instance = None # Class variable to hold the ONE allowed instance

    def __new__(cls, *args, **kwargs):
        # If the instance doesn't exist yet, build it.
        if cls._instance is None:
            print("Allocating memory for the Database Pool...")
            cls._instance = super().__new__(cls)
        # Otherwise, just return the existing one.
        return cls._instance

    def __init__(self):
        # __init__ runs every time, so we must safeguard it
        if not hasattr(self, 'initialized'):
            print("Connecting to database...")
            self.initialized = True

db1 = DatabasePool()
db2 = DatabasePool()

# They are the EXACT SAME object in RAM. We saved massive memory.
print(db1 is db2) # Output: True
```

### 3.3 Custom Class Decorators
Just like you can decorate functions, you can write decorators that wrap around an entire Class to dynamically alter its behavior at load-time without modifying its source code.

```python
# A decorator that automatically adds timestamp data to any class
def add_audit_timestamps(cls):
    from datetime import datetime
    cls.created_at = datetime.now().isoformat()
    return cls

@add_audit_timestamps
class TeamHubProject:
    def __init__(self, title):
        self.title = title

project = TeamHubProject("App Redesign")
# The decorator injected this variable silently!
print(f"Project '{project.title}' initialized at {project.created_at}")
```

### 3.4 Dataclasses (Modern Python 3.7+)
When building classes purely to hold data (like a database row), writing `__init__` is repetitive. `@dataclass` automatically generates the constructor, printing logic, and equality logic for you.

```python
from dataclasses import dataclass

# The @dataclass decorator writes all the boilerplate code behind the scenes
@dataclass
class APIResponse:
    status_code: int
    payload: dict
    endpoint: str
    error_message: str = None # Default value

response = APIResponse(200, {"user": "Rayyan"}, "/api/login")
print(response) 
# Output: APIResponse(status_code=200, payload={'user': 'Rayyan'}, endpoint='/api/login', error_message=None)
```

### 3.5 Memory Optimization with `__slots__`
By default, Python stores an object's attributes in a dynamic dictionary (`__dict__`). This wastes massive amounts of RAM. If you are creating millions of objects, use `__slots__` to lock the class architecture and save up to 50% memory.

```python
class OptimizedUser:
    __slots__ = ['name', 'age']
    
    def __init__(self, name, age):
        self.name = name
        self.age = age

user = OptimizedUser("Sufiyan", 25)
# user.new_attribute = "Test" # THIS WILL CRASH! Slots lock the object shape permanently.
```

### 3.6 Abstract Base Classes (ABCs)
An Abstract Base Class is an interface. It *cannot be instantiated directly*. It forces any child class to build specific methods, ensuring system-wide consistency.

```python
from abc import ABC, abstractmethod

class PaymentGateway(ABC):
    @abstractmethod
    def process_transaction(self, amount):
        pass

class StripeProcessor(PaymentGateway):
    def process_transaction(self, amount):
        print(f"Processing ${amount} via Stripe...")

processor = StripeProcessor()
processor.process_transaction(50.00)
```

### 3.7 Advanced Magic (Dunder) Methods
Double Underscore methods allow your custom objects to interact with standard Python operators seamlessly.

```python
class Task:
    def __init__(self, name, hours):
        self.name = name
        self.hours = hours

    # 1. __str__: Human-readable printing
    def __str__(self):
        return f"[{self.name}: {self.hours}h]"

    # 2. __add__: Controls the '+' operator
    def __add__(self, other):
        return self.hours + other.hours

    # 3. __eq__: Custom Equality
    def __eq__(self, other):
        return self.name == other.name and self.hours == other.hours

    # 4. __call__: Makes the object callable like a function!
    def __call__(self):
        print(f"Executing task: {self.name}...")

task1 = Task("Database Migration", 5)
task3 = Task("API Design", 10)

print(task1 + task3)      # Triggers __add__: 15
task1()                   # Triggers __call__: "Executing task: Database Migration..."
```

### 3.8 Metaclasses (The Absolute Pinnacle)
If an Object is an instance of a Class, a Class is an instance of a **Metaclass**. Metaclasses allow you to intercept the *creation of the blueprint itself*. This is how massive frameworks like Django build their ORMs. 

```python
# A Metaclass that automatically capitalizes all method names
class UppercaseMethodsMeta(type):
    def __new__(cls, name, bases, dct):
        uppercase_dct = {}
        for key, value in dct.items():
            if callable(value) and not key.startswith("__"):
                uppercase_dct[key.upper()] = value
            else:
                uppercase_dct[key] = value
        return super().__new__(cls, name, bases, uppercase_dct)

# Applying the metaclass
class MyClass(metaclass=UppercaseMethodsMeta):
    def say_hello(self):
        print("Hello!")

obj = MyClass()
# We didn't define SAY_HELLO, the metaclass changed the blueprint automatically!
obj.SAY_HELLO() 
```
