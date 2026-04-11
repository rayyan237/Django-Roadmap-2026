# Chapter 3: Data Structures — Memory, Efficiency, and Organization

If Object-Oriented Programming is how we blueprint our system, Data Structures are how we organize the massive amounts of data flowing through that system. Choosing the right data structure is the difference between a backend that processes millions of requests in milliseconds, and one that crashes from memory overload.

---

## LEVEL 1: THE ABSOLUTE BASICS (Core Collections)

A single variable holds a single piece of data. A data structure holds a *collection* of data. Python has four built-in core structures: Lists, Dictionaries, Tuples, and Sets.

### 1.1 Lists (Dynamic Arrays)
A List is an ordered, changeable (mutable) collection of items. It is defined using square brackets `[]`. You use lists when the **sequence** or **order** of data matters.

```python
# 1. Creating a List
server_logs = ["Booting", "Connecting to DB", "System Ready"]

# 2. Accessing Data (Python uses Zero-Based Indexing)
print(server_logs[0])  # Output: "Booting" (The very first item)
print(server_logs[-1]) # Output: "System Ready" (Negative index counts from the end)

# 3. Modifying Data In-Place
server_logs.append("User Logged In") # Adds to the exact end
server_logs.insert(1, "Loading Configs") # Squeezes it into index 1, pushing others down

# 4. Removing Data
server_logs.remove("Booting") # Removes the exact value
last_log = server_logs.pop()  # Removes and returns the very last item

# 5. Slicing (Grabbing a chunk of the list)
# Syntax: list[start:stop:step] (Stop is exclusive)
print(server_logs[0:2]) 
```

### 1.2 Dictionaries (Hash Maps)
A Dictionary is an unordered collection of **Key-Value pairs**. It is defined using curly braces `{}` with a colon `:` separating the key and value. You use dictionaries when you need to look up data instantly using a specific name or ID.

```python
# 1. Creating a Dictionary
user_profile = {
    "username": "Rayyan",
    "role": "Admin",
    "reputation": 99
}

# 2. Accessing Data
# Amateur way (Will crash with a KeyError if the key doesn't exist)
# print(user_profile["email"]) 

# Professional way (Uses .get() which returns None or a default value if missing)
print(user_profile.get("email", "No email provided")) 

# 3. Adding or Updating Data
user_profile["status"] = "Online"  # Adds a new key-value pair
user_profile["reputation"] = 100   # Updates an existing key

# 4. Iterating through a Dictionary
for key, value in user_profile.items():
    print(f"{key.capitalize()}: {value}")
```

---

## LEVEL 2: INTERMEDIATE MECHANICS (Immutability & Data Flow)

As systems grow, you need strict rules on what data can be altered, how data merges, and how data flows in and out of memory.

### 2.1 Tuples (Read-Only Records)
A Tuple is just like a List, but it is **Immutable** (locked). It is defined using parentheses `()`. Because they cannot change, Python highly optimizes them in memory. 

```python
# 1. Creating a Tuple
database_credentials = ("localhost", 5432, "admin_db", "secure_pass")

# 2. Tuples cannot be changed!
# database_credentials[0] = "192.168.1.1" # THIS WILL CRASH! (TypeError)

# 3. Tuple Unpacking (Highly useful)
host, port, db_name, password = database_credentials
print(f"Connecting to {host} on port {port}...")
```

### 2.2 Sets and Frozensets (Hash Tables for Uniqueness)
A Set is an unordered collection where **every item must be unique**. They are blindingly fast at checking if an item exists.

```python
# 1. High-Speed Lookups and Math
project_a = {"Python", "React", "Docker"}
project_b = {"Python", "AWS", "SQL"}

print(project_a.intersection(project_b)) # Shared: {'Python'}
print(project_a.difference(project_b))   # Unique to A: {'React', 'Docker'}

# 2. Frozenset (Immutable Set)
# Standard sets cannot be used as Dictionary Keys. Frozensets can.
locked_users = frozenset(["Rayyan", "Zain"])
team_assignments = {locked_users: "Team Alpha"} 
print(team_assignments[locked_users])
```

### 2.3 Unpacking and Merging (`*`, `**`, and `|`)
Writing loops to combine data is slow. Python provides unpacking operators to merge data structures instantly.

```python
# 1. Merging Lists with '*' (Unpacking)
frontend = ["React", "Vue"]
backend = ["Python", "Node"]
fullstack = [*frontend, *backend, "Docker"]
print(fullstack) # Output: ['React', 'Vue', 'Python', 'Node', 'Docker']

# 2. Merging Dictionaries with '**' or '|'
default_config = {"theme": "dark", "port": 8000}
user_overrides = {"theme": "light"}

# Python 3.9+ Merge Operator (The pipe '|')
# Data on the right overwrites data on the left.
active_config = default_config | user_overrides
print(active_config) # Output: {'theme': 'light', 'port': 8000}
```

### 2.4 Comprehensions (Pythonic Transformations)
A "Comprehension" is the C-optimized way to build or filter a data structure in a single line.

```python
raw_prices = [10, 22, 50, 100, 105]

# Goal: Add a 20% tax to all prices OVER 30.
# Syntax: [expression for item in iterable if condition]
taxed_prices = [price * 1.2 for price in raw_prices if price > 30]
print(taxed_prices) # Output: [60.0, 120.0, 126.0]
```

---

## LEVEL 3: ADVANCED ENGINEERING (Algorithms & Enterprise Standards)

When building massive enterprise software, basic lists are not enough. You must understand Algorithmic Time Complexity and specialized memory structures.

### 3.1 Big-O Time Complexity in Python
* **List `.append()`:** `O(1)` (Instant). Adds to the end.
* **List `.insert(0, item)`:** `O(N)` (Slow). If a list has 1 million items and you insert at the front, Python must physically move 1 million items one slot to the right in RAM.
* **Dict / Set Lookup:** `O(1)` (Instant). Hash Tables compute the exact memory address directly.

### 3.2 Stacks (LIFO) vs. Queues (FIFO)
Understanding data flow is critical for algorithms.
* **Stack (Last-In, First-Out):** Like a stack of plates. The last plate you put on top is the first one you take off. Use a standard Python `List` with `.append()` and `.pop()`. Perfect for "Undo" buttons.
* **Queue (First-In, First-Out):** Like a line at a store. The first person in line is the first served. Standard lists are too slow for this. Use `collections.deque`. Perfect for server task processing.

### 3.3 The `collections` Module
Python provides `collections` for high-performance structural needs.

```python
from collections import deque, Counter, defaultdict, namedtuple, ChainMap

# 1. Deque (Double-Ended Queue for FIFO)
task_queue = deque(["Task2", "Task3"])
task_queue.appendleft("Task1")  # Fast O(1) add to front
print(task_queue.pop())         # Fast O(1) remove from back

# 2. Counter (Instant frequency sorting)
server_errors = ["404", "500", "404", "500", "500"]
print(Counter(server_errors).most_common(1)) # Output: [('500', 3)]

# 3. DefaultDict (Prevents KeyErrors)
user_roles = defaultdict(list)
user_roles["Admin"].append("Rayyan") # Automatically creates the list!

# 4. NamedTuple (Read-only classes without dictionary RAM bloat)
ServerConfig = namedtuple("ServerConfig", ["ip", "port"])
main_server = ServerConfig("192.168.1.1", 8080)

# 5. ChainMap (Memory-less Dictionary combination)
# Combines dictionaries for lookups WITHOUT copying data.
settings = ChainMap({"theme": "light"}, {"theme": "dark", "timeout": 30})
print(settings["timeout"]) # Falls back to the second dictionary instantly
```

### 3.4 Advanced Algorithmic Structures (`heapq` and `bisect`)
Sorting massive lists with `.sort()` is computationally heavy (`O(N log N)`). 

```python
import heapq
import bisect

# 1. Heap Queue (Priority Queue)
# Heaps always keep the smallest item at index 0. Perfect for matchmaking algorithms.
tasks = [50, 10, 30]
heapq.heapify(tasks) 
heapq.heappush(tasks, 5) 
print(heapq.heappop(tasks)) # Output: 5 (Instantly removes the smallest item)

# 2. Bisect (Binary Search Insertions)
# Inserts an item into a sorted list of 1 million items in O(log N) time instead of O(N).
leaderboard = [10, 20, 40, 50]
bisect.insort(leaderboard, 30) 
print(leaderboard) # Output: [10, 20, 30, 40, 50]
```

### 3.5 Read-Only Dictionaries (`MappingProxyType`)
If you design a system where developers need to read a configuration dictionary, but you want to absolutely forbid them from changing it, you must lock it.

```python
from types import MappingProxyType

_internal_settings = {"max_retries": 3}
PUBLIC_SETTINGS = MappingProxyType(_internal_settings)

# PUBLIC_SETTINGS["max_retries"] = 5 # THIS WILL CRASH! (TypeError)
```

### 3.6 Generators and `yield` (Massive Memory Optimization)
If you need to process 10 million database records, pulling them all into a List at once will consume Gigabytes of RAM. A **Generator** pauses its execution and "yields" one item at a time, consuming virtually zero RAM.

```python
import sys

# List Comprehension builds all 10 million items in RAM (~80MB)
massive_list = [x for x in range(10_000_000)]

# Generator Expression only calculates the number WHEN you ask for it (~200 bytes!)
massive_generator = (x for x in range(10_000_000))

# Custom Generator Function
def stream_logs():
    yield "Log 1"
    yield "Log 2"

log_streamer = stream_logs()
print(next(log_streamer)) # Output: "Log 1"
```

### 3.7 C-Level Arrays and Memory Views (The Absolute Pinnacle)
Python Lists are bloated because they can hold mixed data types. If you are processing millions of raw numbers or binary data, you must drop down to C-level structures.

```python
import array

# 1. The Array Module ('i' forces it to only accept signed integers)
# Uses exactly half the RAM of a standard Python List.
numeric_array = array.array('i', [1, 2, 3, 4, 5] * 10000)

# 2. MemoryView
# If you need to slice a massive byte-array, standard slicing creates a duplicate copy in RAM.
# memoryview allows you to slice and read the data WITHOUT duplicating the memory.
byte_data = b"Massive binary payload..."
view = memoryview(byte_data)

# Reading a slice of the view uses 0 extra memory allocation.
print(view[0:7].tobytes()) # Output: b'Massive'
```

### 3.8 Custom Iterators (Under the Hood of 'for' Loops)
You can build custom data structures that interact perfectly with Python's standard `for` loops by implementing the Iterator Protocol (`__iter__` and `__next__`).

```python
class SecurityLogReader:
    def __init__(self, logs):
        self.logs = logs
        self.index = 0

    def __iter__(self):
        return self

    def __next__(self):
        if self.index >= len(self.logs):
            raise StopIteration # Tells the 'for' loop to stop gracefully
        
        current_log = self.logs[self.index]
        self.index += 1
        return f"[SECURE]: {current_log}"

database_logs = SecurityLogReader(["Login", "Logout"])
for log in database_logs:
    print(log)
```
