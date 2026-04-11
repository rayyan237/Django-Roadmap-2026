# Chapter 4: Exception Handling — Engineering for Failure

In an ideal world, networks never drop, databases never lock, and users always type numbers into number fields. In the real world, software breaks constantly. 

Amateur code crashes abruptly when it encounters an unexpected state. Professional, enterprise-grade code anticipates failure, intercepts it, and recovers gracefully. This is the art of Exception Handling.

---

## LEVEL 1: THE ABSOLUTE BASICS (Zero Prior Experience)

### 1.1 Syntax Errors vs. Exceptions (Runtime Errors)
* **Syntax Error:** You typed the code wrong (e.g., missing a colon or parenthesis). Python cannot even read the file. You must fix the code.
* **Exception:** The code is written perfectly, but something illegal happened *while it was running* (e.g., trying to divide by zero, or trying to read a file that got deleted).

### 1.2 The `try` and `except` Block
To stop an Exception from crashing your program, you wrap the dangerous code inside a `try` block. If it fails, Python instantly jumps to the `except` block instead of shutting down.

```python
# The Amateur Way (Will crash the whole program if the user types text)
# age = int(input("Enter your age: ")) 

# The Professional Way
try:
    # Python attempts to run this code
    age = int(input("Enter your age: "))
    print(f"You are {age} years old.")
    
except ValueError:
    # If the user typed "twenty", it triggers a ValueError. 
    # Python jumps here and prevents the crash.
    print("Error: Please type numbers, not words.")

print("Program continues running normally...")
```

### 1.3 Catching Specific Exceptions & Multiple Exceptions
You should always catch the *specific* error you are expecting. If multiple different errors require the exact same recovery logic, you can catch them together using a Tuple `()`.

```python
user_database = {"Rayyan": "Admin"}

try:
    target_user = "Sufiyan"
    role = user_database[target_user] # Triggers KeyError
    
    number = int("Not a number") # Triggers ValueError
    
# Catching multiple specific errors in a single line
except (KeyError, ValueError) as e:
    print(f"Data Retrieval Error: Invalid lookup or conversion. Details: {e}")
```

---

## LEVEL 2: INTERMEDIATE MECHANICS (Flow Control & Rules)

A robust backend doesn't just intercept errors; it manages the entire flow of execution surrounding the failure.

### 2.1 The Full Control Flow: `try`, `except`, `else`, `finally`
* **`try`:** The dangerous code.
* **`except`:** Runs ONLY if an error happens.
* **`else`:** Runs ONLY if the `try` block succeeds perfectly (no errors).
* **`finally`:** ALWAYS runs, no matter what. Used for critical cleanup.

```python
def process_transaction(user_id, amount):
    print(f"\n--- Processing Transaction for {user_id} ---")
    
    try:
        if type(amount) != int:
            raise TypeError("Transaction amount must be an integer.")
        new_balance = 1000 / amount 
        
    except TypeError as e:
        print(f"Data Error: {e}")
    except ZeroDivisionError:
        print("Math Error: Cannot divide by zero.")
    else:
        print(f"Success. New ratio is {new_balance}")
    finally:
        print("Cleanup: Closing secure connection.")

process_transaction("User1", "Fifty") # Triggers TypeError
process_transaction("User3", 10)      # Succeeds (Triggers Else)
```

### 2.2 The Exception Hierarchy and the `BaseException` Trap
All exceptions belong to a family tree. `Exception` is the parent of almost all errors. However, `BaseException` sits above everything.

**CRITICAL RULE:** Never use a bare `except:` or `except BaseException:`. Python uses exceptions like `KeyboardInterrupt` (when you press Ctrl+C) to shut down servers. If you catch `BaseException`, you trap the shutdown commands, creating an unkillable "zombie" process.

```python
# The DANGEROUS way (Catches SystemExit and KeyboardInterrupt)
try:
    pass
except: 
    print("Caught everything, including the attempt to close the program.")

# The PROFESSIONAL generic fallback (Ignores system-level shutdowns)
try:
    pass
except Exception as e:
    print(f"Caught a standard application error: {e}")
```

### 2.3 Reraising Errors (Bare `raise`)
Sometimes you want to catch an error, log it to a file, but still let the program crash or let a higher-level function handle it. If you raise a *new* error, you lose the original line number. A bare `raise` throws the *exact same error* back up the chain.

```python
def read_secure_file():
    try:
        with open("secret.txt", "r") as f:
            return f.read()
    except FileNotFoundError:
        print("[AUDIT LOG]: A missing secure file was requested.")
        # We logged the event, but now we push the exact same error upward!
        raise 

try:
    read_secure_file()
except FileNotFoundError:
    print("User facing message: The file is currently unavailable.")
```

### 2.4 The `finally` Return Trap (Silent Bugs)
Because `finally` ALWAYS runs, it possesses absolute authority over the function's output. If you place a `return` statement inside a `finally` block, it will silently annihilate any `return` or `raise` that occurred in the `try` block.

```python
def dangerous_authentication():
    try:
        # A hacker breaks in and we trigger a lockdown
        raise PermissionError("SYSTEM BREACH DETECTED!")
    finally:
        # The developer accidentally returns True during cleanup
        return True 

# The PermissionError is completely swallowed and destroyed.
# The system thinks authentication was successful!
status = dangerous_authentication() 
print(status) # Output: True
```

---

## LEVEL 3: ADVANCED ENGINEERING (Enterprise Architecture)

In massive frameworks, you build your own error ecosystems, enrich errors with metadata, and install global crash monitors.

### 3.1 Rich Custom Exceptions (`__init__` overrides)
Using `pass` for a custom error is fine for small scripts. Enterprise errors take arguments (like Error Codes) and format their own output by overriding `__init__` and `__str__`.

```python
class PaymentGatewayError(Exception):
    def __init__(self, error_code, message, transaction_id):
        self.error_code = error_code
        self.message = message
        self.transaction_id = transaction_id
        # Call the parent class constructor
        super().__init__(self.message)

    def __str__(self):
        return f"[Error {self.error_code}] TXN {self.transaction_id}: {self.message}"

def charge_card(amount):
    if amount < 0:
        # Raising our custom, rich exception
        raise PaymentGatewayError(400, "Negative amount requested", "TXN_9912")

try:
    charge_card(-50)
except PaymentGatewayError as e:
    # We can access the specific attributes for our logging system
    print(f"Log to Database: Code {e.error_code} for TXN {e.transaction_id}")
    print(f"User Message: {e}")
```

### 3.2 Exception Chaining (`raise ... from ...`)
If an error happens in your database, and you catch it to raise a custom `AppError`, you lose the original database error trace. Python allows you to "chain" exceptions so the developer sees the entire history.

```python
def query_database():
    try:
        1 / 0 
    except ZeroDivisionError as original_error:
        # Explicitly link our custom error to the root cause
        raise ValueError("Database calculation failed") from original_error

# Output will show: "ZeroDivisionError... The above exception was the direct cause..."
```

### 3.3 Exception Notes (Modern Python 3.11+)
Sometimes an error happens deep in the system, and you want to attach contextual data to it (like the User ID who caused it) *without* changing the original error message.

```python
def process_user_data(user_id):
    try:
        int("Bad Data") # Fails
    except ValueError as e:
        # Adds metadata to the traceback output seamlessly
        e.add_note(f"Failure occurred while processing User ID: {user_id}")
        raise

# The traceback will print the ValueError, and append the custom note at the bottom.
```

### 3.4 Capturing the Traceback (`traceback` module)
When catching generic exceptions in production, `print(e)` only gives you the error message. It does NOT tell you what line number crashed. The `traceback` module extracts the exact file and line number.

```python
import traceback

try:
    int("Not a number") 
except Exception as e:
    print("--- FATAL CRASH LOG ---")
    traceback.print_exc() # Prints the exact file, line number, and function
```

### 3.5 Offensive Programming: Assertions (`assert`)
While `try/except` handles expected failures, `assert` is used to catch *impossible* states during development. It is a way of saying, "If this condition is false, the developer broke the code, crash immediately."

```python
def calculate_discount(price, discount):
    # This should mathematically never happen
    assert 0 <= discount <= 1.0, f"Discount {discount} is out of bounds!"
    return price * (1 - discount)

# calculate_discount(100, 1.5) # CRASHES instantly with an AssertionError
```

### 3.6 Validating Failure (Testing Exceptions)
In CI/CD pipelines, you don't just test that code works; you must mathematically prove that it fails when it is supposed to. You use testing modules (like `unittest` or `pytest`) to "expect" an error.

```python
import unittest

def ban_user(user_id):
    if user_id == 0:
        raise ValueError("System ID 0 cannot be banned.")

class TestModeration(unittest.TestCase):
    def test_ban_system_user_fails(self):
        # This test PASSES only if a ValueError is raised!
        with self.assertRaises(ValueError):
            ban_user(0)

# If you run this file with a test runner, it will show success because the error fired correctly.
```

### 3.7 Strict CI/CD: Elevating Warnings to Exceptions
If a developer uses a deprecated function, Python issues a `Warning` but keeps running. In enterprise DevOps, you want tests to *fail* if old code is used. You can force Python to treat warnings as fatal errors.

```python
import warnings

# This configuration is usually placed at the very top of testing scripts
warnings.simplefilter("error")

def old_function():
    warnings.warn("This is deprecated!", DeprecationWarning)

# Because of simplefilter("error"), this will now crash the application 
# with a DeprecationWarning instead of just printing text.
# old_function() 
```

### 3.8 Exception Groups (Modern Python 3.11+)
In asynchronous code, multiple parallel errors can happen at the exact same time. Python 3.11 introduced `ExceptionGroup` and the `except*` syntax to catch multiple parallel errors.

```python
def run_concurrent_tasks():
    raise ExceptionGroup(
        "Multiple Server Failures",
        [
            ConnectionError("Database timed out"),
            ValueError("Invalid JSON payload"),
        ]
    )

try:
    run_concurrent_tasks()
# The except* allows Python to split the group and handle individual errors!
except* ConnectionError as ce:
    print(f"Network Alert: {ce.exceptions}")
except* ValueError as ve:
    print(f"Data Alert: {ve.exceptions}")
```

### 3.9 The Pinnacle: Global Exception Hooks (`sys.excepthook`)
What happens if your server encounters a fatal exception *outside* of any `try/except` block? Normally, it just dies. By overriding `sys.excepthook`, you can intercept the "uncatchable" crash milliseconds before the program terminates, allowing you to send the crash log to a monitoring server.

```python
import sys

def custom_crash_handler(exc_type, exc_value, exc_traceback):
    """This function runs automatically on ANY unhandled fatal crash."""
    print("!!! GLOBAL INTERCEPT !!!")
    print(f"Uploading crash data to server: {exc_type.__name__}")
    
    # We must still call the original hook so the terminal prints the error
    sys.__excepthook__(exc_type, exc_value, exc_traceback)

# Override the default Python crash handler
sys.excepthook = custom_crash_handler

# raise RuntimeError("The entire system is going down.") # Triggers the global hook!
```
