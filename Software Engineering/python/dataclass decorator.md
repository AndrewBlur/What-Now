The `@dataclass` decorator automatically generates common methods for classes that are primarily used to store data.

Instead of writing boilerplate code like `__init__`, `__repr__`, and `__eq__`, Python generates them for you.

Import it like this:

```python
from dataclasses import dataclass
```

---

## Without `@dataclass`

```python
class Student:
    def __init__(self, name: str, age: int):
        self.name = name
        self.age = age

    def __repr__(self):
        return f"Student(name={self.name!r}, age={self.age!r})"

    def __eq__(self, other):
        return self.name == other.name and self.age == other.age
```

You have to write everything yourself.

---

## With `@dataclass`

```python
from dataclasses import dataclass

@dataclass
class Student:
    name: str
    age: int
```

Python automatically creates:

- `__init__()`
    
- `__repr__()`
    
- `__eq__()`
    

Usage:

```python
s1 = Student("Andrew", 20)

print(s1)
```

Output:

```text
Student(name='Andrew', age=20)
```

---

## Automatically generated `__init__`

This:

```python
@dataclass
class Student:
    name: str
    age: int
```

is roughly equivalent to:

```python
class Student:

    def __init__(self, name, age):
        self.name = name
        self.age = age
```

---

## Automatically generated `__repr__`

```python
s = Student("Andrew", 20)

print(s)
```

Output:

```text
Student(name='Andrew', age=20)
```

Without `@dataclass`, you would typically see something like:

```text
<__main__.Student object at 0x1048ab2d0>
```

---

## Automatically generated `__eq__`

```python
s1 = Student("Andrew", 20)
s2 = Student("Andrew", 20)

print(s1 == s2)
```

Output:

```text
True
```

Without a custom `__eq__`, this would normally be:

```text
False
```

because Python would compare object identities, not their contents.

---

## Default values

```python
from dataclasses import dataclass

@dataclass
class Student:
    name: str
    age: int = 18
```

Now:

```python
s = Student("Andrew")

print(s)
```

Output:

```text
Student(name='Andrew', age=18)
```

---

## Mutable defaults

Avoid this:

```python
@dataclass
class Student:
    subjects: list = []
```

Every instance would share the same list.

Instead use `field(default_factory=...)`:

```python
from dataclasses import dataclass, field

@dataclass
class Student:
    subjects: list = field(default_factory=list)
```

Each instance gets its own list.

---

## `__post_init__`

If you need extra initialization after the generated `__init__` runs:

```python
from dataclasses import dataclass

@dataclass
class Rectangle:
    width: int
    height: int

    def __post_init__(self):
        self.area = self.width * self.height
```

Usage:

```python
r = Rectangle(4, 5)

print(r.area)
```

Output:

```text
20
```

---

## Frozen dataclasses

Make instances immutable:

```python
from dataclasses import dataclass

@dataclass(frozen=True)
class Point:
    x: int
    y: int
```

```python
p = Point(1, 2)

p.x = 5
```

Output:

```text
FrozenInstanceError
```

---

## Ordering

```python
from dataclasses import dataclass

@dataclass(order=True)
class Student:
    marks: int
    name: str
```

Now comparisons work:

```python
s1 = Student(90, "Alice")
s2 = Student(80, "Bob")

print(s1 > s2)
```

Output:

```text
True
```

---

## Common options

```python
@dataclass(
    init=True,      # Generate __init__
    repr=True,      # Generate __repr__
    eq=True,        # Generate __eq__
    order=False,    # Generate comparison methods
    frozen=False    # Make immutable
)
class Student:
    name: str
```

---

## Real-world example

```python
from dataclasses import dataclass

@dataclass
class LLMResponse:
    content: str
    tokens_used: int
    model: str
```

Now you can do:

```python
response = LLMResponse(
    content="Hello!",
    tokens_used=42,
    model="gpt-5.5"
)

print(response)
```

Output:

```text
LLMResponse(content='Hello!', tokens_used=42, model='gpt-5.5')
```

without writing any boilerplate methods.

---

### In short

`@dataclass` is best for classes whose main purpose is to hold data. It automatically generates useful methods like `__init__`, `__repr__`, and `__eq__`, making your code shorter, cleaner, and easier to maintain.