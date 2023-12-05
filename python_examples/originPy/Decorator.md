# Decorator

A Python decorator is a design pattern that allows you to add new functionality to an existing object without modifying its structure. Decorators are typically used to modify or enhance functions and methods in a reusable and concise manner. They are particularly useful for cross-cutting concerns like logging, access control, instrumentation, or simply abstracting away repetitive code.

Here's a basic breakdown of how decorators work in Python:

1. **Functionality Enhancement**: A decorator takes a function as an argument, adds some functionality to it, and then returns a new function. This new function typically includes the original function's behavior plus some added functionality.

2. **Syntax**: In Python, decorators are denoted by the `@` symbol, followed by the decorator name, placed above the definition of the function that is to be decorated.

3. **Creating Decorators**: A decorator itself is usually defined as a function that returns another function. The inner function (the one that is returned) is where the additional functionality is implemented, often including a call to the original function.

Here is an example of a simple decorator:

```python
def my_decorator(func):
    def wrapper():
        print("Something is happening before the function is called.")
        func()
        print("Something is happening after the function is called.")
    return wrapper

@my_decorator
def say_hello():
    print("Hello!")
```

In this example:
- `my_decorator` is the decorator.
- `wrapper` is the new function that adds functionality before and after the `say_hello` function.
- `@my_decorator` is the syntax used to apply the decorator to the `say_hello` function.

When `say_hello()` is called, it actually calls the `wrapper()` function from `my_decorator`, which then manages to execute code both before and after the original `say_hello` function runs, enhancing its behavior without altering its code. 

Decorators are a powerful and flexible tool, often used in web frameworks, logging, and system administration scripts. They provide a clean and readable way to modify function behavior in a DRY (Don't Repeat Yourself) manner.

### Example 1: `@property` ###
```python
    @property
    def n_cvs(self):
        """Number of CVs."""
        return self.out_features
```

The `@property` decorator in Python is used for several reasons, particularly when defining methods in a class that are meant to be accessed like attributes. Here are some key reasons why `@property` might be used in this context:

1. **Readability and Intuitive Interface**: It allows the class to present a simpler, more intuitive interface. In your example, `n_cvs` sounds like a property rather than a method. Using `@property` allows it to be accessed without parentheses (e.g., `instance.n_cvs` instead of `instance.n_cvs()`), which is more readable and aligns with how one typically accesses attributes.
   > **Note**: I think here the aforementioned example refers to 1st case.

2. **Encapsulation and Data Hiding**: It provides a way to control access to class attributes. With properties, you can make an attribute read-only or define custom getter/setter methods. This encapsulation hides the internal representation from the user.

3. **Computed Properties**: If the value of `n_cvs` is derived from other attributes (like `out_features` in this case), using a property ensures that it's always up to date. Whenever `n_cvs` is accessed, it will compute its value based on the current state of `out_features`. This is especially useful if `out_features` can change over the lifetime of the object.

4. **Backward Compatibility**: If an attribute was initially exposed directly but later needed some logic to be applied before access, converting it to a property ensures that the external interface of the class remains the same. This way, code that uses the class doesn't need to be changed.

5. **Validation and Error Checking**: In cases where you also implement a setter with `@property.setter`, it allows for validation or error checking before an attribute is set, ensuring the object always remains in a valid state.
