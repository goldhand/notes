Adding a redux reducer to a project
===================================
A short guide to adding a Redux reducer, mid-project, to support a react app.

Adding a reducer should be an easy process and something you should do often. Rather than using state on a single component, maybe consider adding a reducer instead and making the state available to all components.

There are a few different steps in creating a reducer.

1. Map out your state
2. Add your constants
3. Add your actions
4. Write your reducer
5. Implement your reducer in your component


What is a store?
----------------
The store object provides the `getState`, `dispatch` and `subscribe` methods.

```javascript

const createStore = (reducer) => {
  let state;
  let listeners = [];

  const getState = () => state;

  const dispatch = (action) => {
    state = reducer(state, action);
    listeners.forEach(listener => listener());
  };

  const subscribe = (listener) => {
    listeners.push(listener);
    // instead of adding an unsubscribe method, return a function that removes listener
    return () => {
      listeners = listeners.filter(l => l !== listener);
    }
  };

  dispatch({})

  return { getState, dispatch, subscribe };
}
