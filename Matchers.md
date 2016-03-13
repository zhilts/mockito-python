There are only two matchers so far: `any` and `contains`. They allow type and substring matching.

## Usage ##
Argument matchers can be used for both verifications and stubbing. Let's see it in action using simplest `any` matcher. When used without arguments `any` matches any value passed as param.

To use matchers in stubbing simply place them instead of any of arguments
```
from mockito import mock, when, any, verify

# Create mock
dummy = mock()

# When called with any (one) argument return OK
when(dummy).foo(any()).thenReturn("OK")

print dummy.foo(1) # prints "OK"
print dummy.foo("some string") # prints "OK"
```

Same goes for verifications
```
from mockito import mock, any, verify

# Create mock
dummy = mock()

# Excersize mock
dummy.foo(1)
dummy.foo("some string")

# Verify foo was called with whatever argument 2 times
verify(dummy, times=2).foo(any())
```

## Type Matching ##
The `any` matcher allows you to match any argument (`any()`) or argument of specific type(`any(type)`), for example `any(int)`, `any(str)`, `any(MyClass)` and so on with any valid python type
```
from mockito import mock, any, verify

dummy = mock()

dummy.foo(1)
dummy.bar("some string")

# Following verifications will pass
verify(dummy).foo(any(int))
verify(dummy).bar(any(str))
```

Same for stubbing
```
from mockito import mock, when, any

class MyType: pass

dummy = mock()

when(dummy).foo(any(int)).thenReturn("Called with integer argument")
when(dummy).foo(any(str)).thenReturn("Called with string argument")
when(dummy).foo(any(MyType)).thenReturn("Called with MyType argument")

print dummy.foo(321) # prints "Called with integer argument"
print dummy.foo("bar") # prints "Called with string argument"
print dummy.foo(MyType()) # prints "Called with MyType argument"
```

## Substring Matching ##
The `contains` matcher is useful for substring matching. Pass expected substring as argument, like `contains("foo")`. In action:
```
from mockito import mock, when, contains

steve = mock()

when(steve).greet(contains("Jonny")).thenReturn("Hi, Jonny!")
when(steve).greet(contains("Sara")).thenReturn("Hello, Mrs. Parker")

print steve.greet("Jonny Parker") # prints "Hi, Jonny!"
print steve.greet("Mrs. Sara Parker") # prints "Hello, Mrs. Parker"
```

The matcher will surely also work for verifications.
```
from mockito import mock, verify, contains

dummy = mock()

dummy.foo("red blue")

verify(dummy).foo(contains("green")) # raises VerificationError: Wanted but not invoked: foo(<Contains: 'green'>) 
```