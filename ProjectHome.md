Mockito-python is a Test Spy Framework based on Java library of the same name.


---

Quick links: [Downloads](http://code.google.com/p/mockito-python/downloads/list) | [Tutorial](Tutorial.md)  | [Bug Tracker](http://bitbucket.org/szczepiq/mockito-python/issues) | [Mailing List](http://groups.google.com/group/mockito-python) | [Source Code](http://bitbucket.org/szczepiq/mockito-python/src)

---

### News ###
_July 26, 2010_: Release 0.5.0. Featuring [spying on real objects](Spying#Spying_on_Real_Objects.md).

_July 2, 2010_: Release 0.4.0. Mockito got [in order verifications](Verifications#In_Order_Verifications.md).

---


## Mockito in 15 minutes ##
### Installing ###
Installing mockito via `easy_install` is easy, just ` $ easy_install mockito `

Proceed to [tutorial](Tutorial.md) for other [installation options](Installing.md).


### Basics ###
Import mockito
```
>>> from mockito import *
```

Creating a stub in mockito is one-line
```
>>> myMock = mock()
```

Now you can use your mock
```
>>> myMock.foo()
>>> myMock.bar()
```

And verify interactions happened
```
>>> verify(myMock).foo()
>>> verify(myMock).bar()
```

Mockito will signal if wanted interactions never happened
```
>>> verify(myMock).baz()
Traceback (most recent call last):
 ...
mockito.verification.VerificationError: 
Wanted but not invoked: baz()
```


### Verifications ###
Finding redundant interactions
```
>>> myMock = mock()
>>> myMock.foo()
>>> myMock.bar()
>>> verify(myMock).foo()
>>> verifyNoMoreInteractions(myMock)
Traceback (most recent call last):
...
mockito.verification.VerificationError: 
Unwanted interaction: bar()
```

Or even say you dont want any interactions at all
```
>>> myMock = mock()
>>> myMock.foo()
>>> verifyZeroInteractions(myMock)
...
mockito.verification.VerificationError: 
Unwanted interaction: foo()
```

More on [verifications](Verifications.md) in [tutorial](Tutorial.md).

### Stubbing ###
Most common usage is to stub a method to return value
```
>>> dog = mock()
>>> when(dog).bark().thenReturn("wuff")
>>> dog.bark()
'wuff'
```

You can stub method to raise exception
```
>>> when(dog).meow().thenRaise(Exception("Who do you think I am?!"))
>>> dog.meow()
Traceback (most recent call last):
...
Exception: Who do you think I am?!
```

Or stub consecutive calls using `.thenReturn()` and `.thenRaise()` as many times you wish and in any order
```
>>> iterator = mock()
>>> when(iterator).next().thenReturn(1).thenReturn(2).thenRaise(Exception("Out Of Numbers"))
>>> iterator.next()
1
>>> iterator.next()
2
>>> iterator.next()
Traceback (most recent call last):
...
Exception: Out Of Numbers
```

However, when stubbing with same parameters multiple times only last stubbing is effective
```
>>> iterator = mock()
>>> when(iterator).next().thenReturn(1)
>>> when(iterator).next().thenReturn(2)
>>> iterator.next()
2
>>> iterator.next()
2
```

When you stub consecutive calls, last stubbing is used for all additional calls
```
>>> dummy = mock()
>>> when(dummy).foo().thenReturn("Hello").thenReturn("Bye")
...
>>> dummy.foo()
'Hello'
>>> dummy.foo()
'Bye'
>>> dummy.foo()
'Bye'
```

When stubbing you can set different behavior based on call arguments
```
>>> dummy = mock()
>>> when(dummy).reply("hi").thenReturn("hello")
>>> when(dummy).reply("bye").thenReturn("good-bye")
>>> dummy.hi()
>>> dummy.reply("hi")
'hello'
>>> dummy.reply("bye")
'good-bye'
```

Same for different amount of arguments.
```
>>> dummy = mock()
>>> when(dummy).foo("arg1", "arg2").thenReturn("called with two arguments")
>>> when(dummy).foo("arg1").thenReturn("called with one argument")
>>> dummy.foo("arg1")
'called with one argument'
>>> dummy.foo("arg1", "arg2")
'called with two arguments'
```
[Argument matchers](Matchers.md) can give you even more freedom. Consult [tutorial](Matchers.md).

You can create mock from existing class
```
>>> class Dog:
...     def bark(self): pass
...     def grin(self): pass
... 
>>> rex = mock(Dog)
```

And stub it's methods
```
>>> when(rex).bark().thenReturn("wuff")
>>> rex.bark()
'wuff'
```

There are two differences between mocks created from real classes and dummy mocks. First is that all methods of mock created from real classes return None by deafult. Following assertion passes
```
>>> assert None == rex.grin()
```

Second is that you can not stub non-existing methods, by default
```
>>> when(rex).meow().thenReturn("meow")
Traceback (most recent call last):
...
mockito.invocation.InvocationError: You tried to stub a method 'meow' the object (__main__.Dog) doesn't have.
```

If that's too strict you can change it
```
>>> rex = mock(Dog, strict=False)
>>> when(rex).meow().thenReturn("meow")
>>> rex.meow()
'meow'
```

Stubbing [instance, class-level and static methods](Stubbing#Instance,_Class-level_and_Static_Methods.md); even [modules](Stubbing#Modules.md) and [existing instances](Stubbing#Instances.md) is also possible. Proceed to [tutorial on stubbing](Stubbing.md) for more info.

This 15 minutes (hopefully less) tutorial should give you enough info to get started and feel taste of mockito. Please consult [tutorial](Tutorial.md), [our mailing list](http://groups.google.com/group/mockito-python), [bug tracker](http://bitbucket.org/szczepiq/mockito-python/issues) and finally [source code](http://bitbucket.org/szczepiq/mockito-python/) for more info.

