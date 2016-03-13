## Basics ##
Creating a stub in mockito is as simple as importing one function and calling it
```
from mockito import mock
myMock = mock()
```

That's it. You created a valid stub. For every method you call on it it will return `None` by default
```
from mockito import mock

# Create stub
myMock = mock()

# Call methods
print myMock.foo() # prints "None"
```
You can change this behavior, see next topic.

## Return Values and Exceptions ##
Most common use-case is to tell a stub to return some value. Mockito uses `when` function for this
```
from mockito import mock, when

myMock = mock()

# Stub to return value
when(myMock).foo().thenReturn("bar")

print myMock.foo() # prints "bar"
```

The `.thenReturn()` DSL tells a mock to return value passed in as parameter. Same works for integers, bytes or any other valid python types and variables
```
when(myMock).giveMeInt().thenReturn(42)
when(myMock).giveMeDouble().thenReturn(3.14)
when(myMock).giveMeMoney().thenReturn(Money(1000000))
# And so on...
```

Similar API exists for making stub raise an exception
```
from mockito import mock, when

myMock = mock()

# Stub to raise exception
when(myMock).foo().thenRaise(Exception("intentionally"))

myMock.foo() # raises Exception: intentionally
```
As you saw, to make stubbed method raise an exception simply pass the exception instance to `.thenRaise()`.

Stubbing return values and exceptions in action:
```
from mockito import mock, when

# Creating mock
dog = mock()

# Stubbing calls
when(dog).bark().thenReturn("waff") # stub to return value
when(dog).meow().thenRaise(Exception("Who do you think I am?!")) # stub to raise exception

# Use stub
print dog.bark() # prints "waff"
dog.meow() # raises exception
```

When stubbing you can differentiate calls on arguments
```
from mockito import mock, when

list = mock()

when(list).get(0).thenReturn("first")
when(list).get(1).thenReturn("second")

print list.get(0) # prints "first"
print list.get(1) # prints "second"
```

Also named of-course
```
from mockito import mock, when

users = mock()

when(users).findby(id=6).thenReturn("John May")
when(users).findby(name="John").thenReturn(["John May", "John Smith"])

print users.findby(id=6) # prints "John May"
print users.findby(name="John") # prints "['John May', 'John Smith']"
```

## Consecutive Calls ##
Stubbing consecutive calls is easy with mockito. Chain `.theReturn()` and `.thenRaise()` as many times as you wish and in any order. In action:
```
from mockito import mock, when

iterator = mock()

when(iterator).next().thenReturn(1).thenReturn(2).thenRaise(Exception("Out of numbers"))

print iterator.next() # Prints "1"
print iterator.next() # Prints "2"
print iterator.next() # Raises Exception: Out of numbers
```


## Stubs from Real Classes ##
To create a stub from real class simply pass you class to _mock_ function. Like:
```
dog = mock(Dog)
cat = mock(Cat)
```

In action:
```
from mockito import mock, when

class Cat:
  def meow(self):
    pass

# Generate a stub from a class
cat = mock(Cat)

# Stub method calls
when(cat).meow().thenReturn("meow")

# Use mock
print cat.meow() # prints "meow"
```

The difference in creating stubs from real classes is that if you will try to stub a method class doesn't have Mockito will raise an exception
```
from mockito import mock, when

class Cat:
  def meow(self):
    pass

cat = mock(Cat)

# Stub non-existing method
when(cat).bark().thenReturn("wuff") # raises InvocationError
```

You can change this behavior by setting `strict` flag to `False` when creating mock. For example `cat = mock(Cat, strict=False)`. In action:
```
from mockito import mock, when

class Cat:
  def meow(self):
    pass

cat = mock(Cat, strict=False)

# Stub non-existing method
when(cat).bark().thenReturn("wuff")

print cat.bark() # prints "wuff"
```

If you create a stub from real class return values of all class methods will default to None, until stubbed to do else
```
from mockito import mock, when

class Dog:
  def bark(self):
    return "wuff"
    
dog = mock(Dog)

# Return values default to None
assert None == dog.bark() # passes

# Stub to return other value
when(dog).bark().thenReturn("meow")
assert "meow" == dog.bark() # passes
```

**NOTE:** If you get error message like "InvocationError: You called %s with %s and %s as arguments but we did not expect that." when creating stubs from real classes -- this is due to [issue #8](http://bitbucket.org/szczepiq/mockito-python/issue/8/unexpected-mock-behavior) which is fixed in version 0.3.1. Please upgrade in order to fix that.


## Instance, Class-level and Static Methods ##
In addition to creating stubs from a given class, you can also stub instance methods. Concept is similar to monkey patching. When you stub instance methods, all the instances of that class will have those methods stubbed. In example below both Rex and Sparky will bark identically. **TODO: Explain possible usage options.**

```
from mockito import when

class Dog:
  def bark(self):
    pass

# Stubbing class method
when(Dog).bark().thenReturn("wuff")

# Creating instances 
rex = Dog()
sparky = Dog()

print rex.bark() # prints "wuff"
print sparky.bark() # prints "wuff"
```

No difference for class-level and static methods
```
from mockito import when

class Dog:
  @classmethod
  def bark():
    pass

  @staticmethod
  def grin():
    pass

# Stubbing
when(Dog).bark().thenReturn("wuff")
when(Dog).grin().thenReturn("grrr")

# Using stubbed method
print Dog.bark() # prints "wuff"
print Dog.grin() # prints "grrr"
```


## Modules ##
API for stubbing modules is exactly the same. You can make module functions return values or raise exceptions. Example:
```
from mockito import when

# No wonder that modules have to be imported first
import os.path

# Stub calls
when(os.path).exists("/some/dir").thenReturn(True)
when(os.path).exists("\some]dir").thenRaise(AssertionError("Invalid directory name"))

# Use stubs
assert os.path.exists("/some/dir")
assert os.path.exists("\some]dir") # raises AssertionError: Invalid directory name
```


## Instances ##
Stubbing instances is also possible. You can think of them like partial mocks. Example:
```
from mockito import when, verify

class Dog:
  def bark(self):
    pass
  def grin(self):
    return "grrr"

# Create instance
dog = Dog()

# Stub real instance
when(dog).bark().thenReturn("wuff")

print dog.bark() # prints "wuff"
print dog.grin() # prints "grrr"
```

However you can't verify on methods, that were not stubbed. For example following will pass
```
verify(dog).bark()
```

But verification on non-stubbed method will fail
```
verify(dog).grin() # raises VerificationError: Wanted but not invoked: grin()
```


## Unstubbing ##
There is an `unstab` method, which unstubs all previously registered stubs
```
from mockito import when, unstub

class Dog:
  def bark(self):
    return "wuff"

# Stubbing
when(Dog).bark().thenReturn("meow")

# Create instance of stubbed class
dog = Dog()

# Use stubbed method
print dog.bark() # prints "meow"

# Unstubbing
unstub() # following unstubs ALL stubs!

# Use unstubbed method
print dog.bark() # prints "wuff"
```

`unstab` unregisters _all_ stubs(classes, instances, modules etc). Be careful when using it.