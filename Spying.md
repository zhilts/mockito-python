## Introduction ##
True power of mockito comes from two simple concepts: spying and [stubbing](Stubbing.md). Unlike most mocking frameworks, mockito doesn't really have mocks. The name "mock" is used in mockito though, as it is a part of developers vocabulary this days and normally developers call everything that is [Test Double](http://xunitpatterns.com/Test%20Double.html) a "mock".


## Using Spies ##
Spies are created same way [stubs](Stubbing.md) are
```
from mockito import mock, verify
mySpy = mock()
```

Spies let you verify interactions on your doubles
```
mySpy.someMethod()
verify(mySpy).someMethod()
```
Here on the last line we verify that method we expected to be called was actually called. That is what spies are for.
Under the hood mockito simply remembers all invocations and lets you further verify on them. More on verifying in [Verifications](Verifications.md) chapter.

Every valid mockito stub is a spy and vice-versa
```
from mockito import mock, when, verify

rex = mock()
when(rex).bark().thenReturn("wuff")

# Using stubbed method
print rex.bark()

# Verifying spied calls
verify(rex).bark()
```

But normally you'll chose whether you need a spy or stub depending on logic under test. The need to both stub and verify on same test double normally indicates wrong testing approach or bad design of system under test.

## Spying on Real Objects ##
<font color='red'><b>WARNING!</b> </font>Spying on real objects is currently broken and will be fixed in release 0.5.2. Please do not rely on it.

Since 0.5.0 mockito lets you spy on real objects. That is, implementation of methods remains unmodified but you can later verify that interactions happened
```
from mockito import spy, verify

class Dog:
  def bark(self):
    return "wuff"

# Create instance of a class
rex = Dog()

# Create a spy
spiedRex = spy(rex)

# Call methods
print spiedRex.bark() # prints "wuff"

# Verify interactions
verify(spiedRex).bark()
```

Normally you'll create spies in one line
```
rex = spy(Dog())
```

All kinds of verifications available to regular spies are also available to spies on real objects. Proceed to [Verifications](Verifications.md) chapter to read more.