## Basic ##
Mockito lets you verify interactions on your test doubles. Simplest and most common form is to call `verify` on double to check if it was called. Example:
```
from mockito import mock, when, verify

# Create mock
list = mock()

# Excersise mock object
list.add(1, "first")
list.clear()

# Verify
verify(list).add(1, "first")
verify(list).clear()
```
In this example verification passes, as methods we verified were called with arguments we verified. If we pass methods that weren't called we will get `VerificationError`
```
verify(list).foo() # raises VerificationError: Wanted but not invoked: foo()
```
Same happens if call arguments differ from those we expected
```
verify(list).add(2, "second") # raises VerificationError: Wanted but not invoked: foo()
```


## Verifying Number of Invocations ##
You can ask mockito to verify exact number of invocations by passing named argument `times` to verify function. For example `verify(foo, times=2).bar()`.

The `times` modifier  tells mockito to verify _exact_ number invocations. In action:
```
from mockito import mock, verify

# Create mock
dummy = mock()

# Excersise the mock
dummy.foo()
dummy.foo()
dummy.bar()
dummy.bar()

# Verifications
verify(dummy, times=2).foo() # passes
verify(dummy, times=3).bar() # raises VerificationError: Wanted times: 3, actual times: 2 
```


There are also `atleast`, `atmost` and `between` modifiers if you want to verify on non-exact amount of interactions. Example below explain it all
```
from mockito import mock, verify

# Create mock
dummy = mock()

# Excersise the mock
dummy.foo()
dummy.foo()

# All verifications below pass
verify(dummy, times=2).foo()
verify(dummy, atleast=1).foo()
verify(dummy, atmost=2).foo()
verify(dummy, between=[1,3]).foo()
```

## Finding Redundant and Unwanted interactions ##
The `verifyNoMoreInteractions` function serves purpose to catch redundant interactions. In example below the `verifyNoMoreInteractions(dummy)` fails as we we said we don't want any more interactions on mock.
```
from mockito import mock, verify, verifyNoMoreInteractions

dummy = mock()

dummy.foo()
dummy.bar()

verify(dummy).foo()
verifyNoMoreInteractions(dummy) # raises VerificationError: Unwanted interaction: bar()
```
This is a useful way to find redundant invocations and make your tests stricter a bit.

You can also verify that no interactions happened on mock with `verifyZeroInteractions`
```
from mockito import mock, verify, verifyZeroInteractions

dummy = mock()

dummy.foo()

verifyZeroInteractions(dummy) # raises VerificationError: Unwanted interaction: foo()
```
The `verifyZeroInteractions(dummy)` failed as we called `foo()` on our mock.

To verify that some method was never called you can pass `0` to times named argument or use mockito's syntactic sugar `never` (_since 0.5.0_)
```
from mockito import mock, verify, never

dummy = mock()

verify(dummy, never).foo()
```

It will fail with `AssertionError` if unwanted interaction happened
```
dummy.foo()

verify(dummy, never).foo() # raises "AssertionError: Unwanted invocation of foo(), times: 1"
```


## In Order Verifications ##
It is also possible(since release 0.4.0) to verify interactions in order
```
from mockito import mock, inorder

dummy = mock()

dummy.first()
dummy.second()

inorder.verify(dummy).first()
inorder.verify(dummy).second()
```

If invocations happened not in order you expected mockito will raise VerificationError
```
dummy = mock()

dummy.second()
dummy.first()

inorder.verify(dummy).first() # raises VerificationError: Wanted first() to be invoked, got second() instead
```

All the verifications modifiers(`times`,  `atleast`, `atmost` and `between`) work same way as with normal verifications. `times` for example:
```
from mockito import mock, inorder

dummy = mock()

dummy.foo()
dummy.foo()

inorder.verify(dummy, times=2).foo()
```

And generate same error messages
```
from mockito import mock, inorder

dummy = mock()

dummy.foo()

inorder.verify(dummy, times=2).foo() # raises VerificationError: Wanted times: 2, actual times: 1
```

Normal verifications can be mixed with inorder once without any conflicts.
```
from mockito import mock, inorder, verify

dummy = mock()

dummy.first()
dummy.second()

# All verifications below pass
verify(dummy).second()
verify(dummy).first()
inorder.verify(dummy).first()
inorder.verify(dummy).second()
```
They do not interfere, however, normally this shouldn't be needed.

## Verifying on Stubbed Calls ##
You can, of-course, make all kinds of verifications (`verify`, `verifyZeroInteractions`, `verifyNoMoreInteractions`) on stubbed calls, however this is normally a signal of bad test design
```
from mockito import mock, when, verify, verifyZeroInteractions, verifyNoMoreInteractions

# Create mocks
dog = mock()
cat = mock()

# Stub method call
when(dog).bark().thenReturn("wuff")

# Execute method
dog.bark()

# All verifications below pass
verify(dog).bark()
verifyNoMoreInteractions(dog)
verifyZeroInteractions(cat)
```

[Argument matchers](Matchers.md) are of very much help in verifications. See [appropriate wiki page](Matchers.md).