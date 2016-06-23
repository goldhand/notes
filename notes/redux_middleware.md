Redux Middleware
================
It provides a third-party extension point between dispatching an action, and the moment it reaches the reducer.


Prerequisite: [currying](https://en.wikipedia.org/wiki/Currying)


React middleware curried function that takes three arguments: `store`, `next`, and `action`

* `store` is referring to the store (you can use `store.getState()`, or `store.dispatch()`)
* `next` is the next middleware function
* `action` is the action that was dispatched and is now being passed along middleware.

The most simple middleware:
```javascript

const myMiddleware = store => next => action => {
  return next(action);
}
```

Visualizing:
```javascript

function(store, next, action) {

  return myMiddleware(store)(next)(action);
}
```

Rather than explaining this... I'm just copying examples from the [docs...](http://redux.js.org/docs/advanced/Middleware.html)

Here is a simple middleware:

```javascript

const logger = store => next => action => {
  console.log('dispatching', action)
  let result = next(action)
  console.log('next state', store.getState())
  return result
}

const crashReporter = store => next => action => {
  try {
    return next(action)
  } catch (err) {
    console.error('Caught an exception!', err)
    Raven.captureException(err, {
      extra: {
        action,
        state: store.getState()
      }
    })
    throw err
  }
}
```

And here is that middleware applied:

```javascript
import { createStore, combineReducers, applyMiddleware } from 'redux'

let todoApp = combineReducers(reducers)
let store = createStore(
  todoApp,
  // applyMiddleware() tells createStore() how to handle middleware
  applyMiddleware(logger, crashReporter)
)
```

A good implementation would be to add a `meta` attribute to actions that should have middleware side effects then listen if the action has that meta attribute.


```javascript
/**
 * Schedules actions with { meta: { raf: true } } to be dispatched inside a rAF loop
 * frame.  Makes `dispatch` return a function to remove the action from the queue in
 * this case.
 */
const rafScheduler = store => next => {
  let queuedActions = []
  let frame = null

  function loop() {
    frame = null
    try {
      if (queuedActions.length) {
        next(queuedActions.shift())
      }
    } finally {
      maybeRaf()
    }
  }

  function maybeRaf() {
    if (queuedActions.length && !frame) {
      frame = requestAnimationFrame(loop)
    }
  }

  return action => {
    if (!action.meta || !action.meta.raf) {
      return next(action)
    }

    queuedActions.push(action)
    maybeRaf()

    return function cancel() {
      queuedActions = queuedActions.filter(a => a !== action)
    }
  }
}
```
