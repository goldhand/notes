Javascript Error Handling
=========================


When a javascript statement generates and error, it **throws** an exception.
The interpreter does not proceed to the next statement, it instead checks for
exception handling code. The will return to whatever threw the error if there
is no exception handing. This is repeated for each function on the call stack
until and exception handle is found or the top level function is reached.


ECMA defines 7 types of built-in error objects:

### Error
A generic error, often used for user defined errors.

### RangeError
Numbers that fall outside a specified range.

### ReferenceError
A non-existent variable is accessed. (Often caused by misspelled variable names).

### SyntaxError
A javascript language rule is broken.

### TypeError
When a value is treated as the wrong type. Eg. an object is called as a function.

### URIError

### EvalError


Handling Exceptions
-------------------

Errors are handled in try...catch blocks.

```javascript
try {
  // attempt to execute this code
} catch (exception) {
  // this code handles exceptions
} finally {
  // this code always gets executed (even if something is returned in try/catch)
}
```

Inside the `catch` block, error handling can exist. If there is a catch block,
errors will not be propagated through the call stack and the program will try to
recover.

Use the `instanceof` operator to handle different types of exceptions:

```javascript
try {
  // something breaks
} catch (exception) {
  if (exception instanceof TypeError) {
    // handle type errors
  } else if (exception instanceof RangeError) {
    // handle range errors
  } else {
    // handle everything else
  }
}
```

Alternatively, use the `constructor` method to handle exceptions in a switch
statement:

```javascript
try {
  // something breaks
} catch (exception) {
  switch (exception.constructor) {
    case TypeError:
      // handle type error
    case RangeError:
      // handle range error
    default:
      // everything else
  }
}
```
